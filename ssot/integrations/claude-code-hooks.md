---
id: integration.claude-code-hooks
kind: Integration
title: Claude Code 훅 연동
definition: Claude Code가 세션·도구 실행·서브에이전트 시점마다 발화하는 표준 훅 이벤트를 Clawket 전용 핸들러에 연결해, 세션 컨텍스트 주입·변경 작업 차단·플랜 동기화를 수행하는 연동.
purpose: Claude Code가 작업하는 매 시점(세션 시작, 프롬프트 제출, 도구 사용 직전/직후, 서브에이전트 시작/종료)에 Clawket이 끼어들어 "활성 태스크 없이는 변경 불가", "세션 컨텍스트 자동 주입", "Plan Mode 출력의 Clawket 플랜 등록 유도" 같은 가드레일과 컨텍스트 보강을 작동시키기 위해 필요하다.
integratesWith:
  - component.claude-adapter
  - component.plugin-shell
implementedIn:
  - clawket/hooks/hooks.json
  - clawket/adapters/claude/session-start.cjs
  - clawket/adapters/claude/user-prompt-submit.cjs
  - clawket/adapters/claude/pre-tool-use.cjs
  - clawket/adapters/claude/post-tool-use.cjs
  - clawket/adapters/claude/plan-sync.cjs
  - clawket/adapters/claude/subagent-start.cjs
  - clawket/adapters/claude/subagent-stop.cjs
  - clawket/adapters/shared/claude-hooks.cjs
impacts:
  - endpoint.hook-session-start
  - endpoint.hook-user-prompt-submit
  - endpoint.hook-pre-tool-use
  - endpoint.hook-post-tool-use
  - endpoint.hook-plan-sync
  - endpoint.hook-subagent-start
  - endpoint.hook-subagent-stop
  - persona.llm-coding-agent
relatesTo:
  - to: domain.hook-enforcement
    type: realizes
    note: 훅 강제(가드레일) 영역을 Claude Code 훅 이벤트 위에서 실제로 구현하는 연동
  - to: integration.mcp
    type: relates-to
    note: 같은 플러그인 셸이 등록하는 또 다른 Claude Code 표면(훅=쓰기·차단, MCP=읽기 RAG)
governedBy:
  - invariant.active-task-required
  - invariant.no-work-without-task
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:integration
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로 — 무엇을·왜·누가·어떤 규칙·무슨 데이터. 코드(테이블·필드·경로·SQL) 옮겨적기 금지(식별자는 provenance/근거에만). 코드 분기/의도 불명은 판정 말고 OPEN. -->

## 무엇과 연동하나

상대편은 Claude Code 그 자체다. Claude Code는 사용자와 코딩 에이전트가 작업하는 동안 정해진 시점마다 "훅 이벤트"를 발화한다. Clawket은 이 이벤트들을 자신의 핸들러로 받아 작업 관리 규칙을 끼워 넣는다.

연결되는 시점은 여섯 개의 표준 이벤트와, 그 중 하나에 더 좁게 건 하나의 매처로 구성된다.

- 세션이 시작/초기화/압축될 때 — 데몬 보장과 함께 활성 프로젝트의 작업 대시보드와 규칙을 컨텍스트로 주입한다.
- 사용자가 프롬프트를 제출할 때 — 지금 진행 중인 태스크의 컨텍스트를 끼워 넣고, 활성 태스크가 없으면 경고한다.
- 도구를 쓰기 직전(특히 코드 변경·서브에이전트 생성·메시지 전송 계열) — 활성 태스크가 없으면 변경 작업을 막고, 파괴적 명령과 검증 안티패턴, 모델 티어 적합성을 검사한다.
- 파일을 편집/생성한 직후 — 그 변경을 활성 태스크에 자동 기록한다.
- 서브에이전트가 시작/종료할 때 — 서브에이전트를 배정 태스크에 묶고, 종료 시 결과를 요약하고 조건이 맞으면 태스크를 자동 완료한다.
- Plan Mode를 빠져나가는 순간(도구 사용 직후 이벤트의 좁은 매처) — Plan Mode가 만든 계획 출력을 Clawket 플랜으로 등록하도록 유도한다.

프로토콜은 Claude Code의 훅 규약을 그대로 따른다. 각 이벤트는 노드 스크립트를 명령으로 실행하는 형태이며, 핸들러가 표준 출력으로 돌려주는 결정(허용/차단/컨텍스트 추가)을 Claude Code가 해석한다. 인증 같은 별도 절차는 없고, 모든 동작은 로컬 데몬을 통해서만 일어난다.

## 구현 위치 (provenance)

훅을 어느 이벤트에 어떤 핸들러로 걸지는 플러그인 셸의 훅 매니페스트(`clawket/hooks/hooks.json`)가 선언한다. 이 매니페스트는 이벤트별로 실행할 노드 스크립트의 경로를 가리키며, 플러그인 루트는 환경 변수로 주입되거나 기본 캐시 경로로 폴백된다.

각 이벤트의 얇은 진입점은 `clawket/adapters/claude/` 아래의 개별 `.cjs` 파일들(세션 시작·프롬프트 제출·도구 사용 전/후·플랜 동기화·서브에이전트 시작/종료)이고, 실제 강제 로직과 데몬 연결·설치 게이트 같은 무거운 공통 처리는 모두 공유 핸들러(`clawket/adapters/shared/claude-hooks.cjs`)에 모여 있다. 즉 진입점은 이벤트별로 나뉘어 있지만 판단 로직의 단일 집결지는 공유 핸들러 한 곳이다.

이 연동을 사용하는 내부 컴포넌트는 Claude 어댑터(`component.claude-adapter`)이며, 그것을 패키징해 배포하는 것은 플러그인 셸(`component.plugin-shell`)이다.

매니페스트의 스키마 버전 표기상 이 구성은 v3 세대다 — 표준 6개 이벤트에 더해, 도구 사용 직후 이벤트에 Plan Mode 종료만 따로 잡는 매처를 추가로 둔 형태다.

## 불변식

- 활성 태스크가 지정되어 있지 않으면 변경 계열 도구 호출이 도구 사용 직전 시점에서 차단된다(`invariant.active-task-required`, `invariant.no-work-without-task`). 이 연동의 존재 이유 자체가 이 가드를 작동시키기 위한 것이다.
- 차단/허용 판정은 항상 로컬에서, 데몬을 경유해 이뤄진다. 외부로 나가는 호출은 없다(이 노드의 상위 영역인 훅 강제가 로컬 우선 전제 위에서 동작).

## 영향 범위

이 연동이 끊기거나 잘못 등록되면 위 일곱 개 훅 엔드포인트(세션 시작·프롬프트 제출·도구 사용 전/후·플랜 동기화·서브에이전트 시작/종료)가 모두 작동하지 않는다. 그 결과 코딩 에이전트(`persona.llm-coding-agent`)에 대한 가드레일(활성 태스크 강제, 컨텍스트 자동 주입, 플랜 동기화 유도)이 사라진다.

## 미확정 (OPEN)
- [ ] OPEN: 각 훅의 타임아웃·실패 시 동작(차단 vs 통과)이 이벤트별로 어떻게 다른지 본문 수준의 정밀 기술은 각 endpoint.hook.* 노드에 위임 — 본 연동 노드는 연결 구조까지만 확정.
