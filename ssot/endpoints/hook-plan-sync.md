---
id: endpoint.hook-plan-sync
kind: Endpoint
title: 훅 PostToolUse:ExitPlanMode (plan-sync)
definition: "Plan Mode 종료를 가로채는 훅으로, 산출된 플랜 마크다운을 strict 포맷으로 검증한 뒤 통과하면 자동으로 Clawket 플랜으로 등록하고, 어긋나면 위반 위치를 짚어 등록을 막는다."
realizedBy:
  - component.claude-adapter
implementedIn:
  - clawket/hooks/hooks.json
  - clawket/adapters/claude/plan-sync.cjs
  - clawket/adapters/shared/claude-hooks.cjs
relatesTo:
  - to: endpoint.daemon-plans-import-strict
    type: relates-to
    note: 플랜 마크다운의 strict 포맷 적합성을 데몬에 위임해 검증한다
  - to: concept.plan
    type: mutates
    note: 검증을 통과한 Plan Mode 출력을 Clawket 플랜으로 자동 생성한다
  - to: endpoint.cli-plan
    type: relates-to
    note: 파싱된 플랜 본문으로 플랜 생성 명령을 호출해 등록한다
  - to: concept.active-project
    type: reads
    note: 현재 디렉터리의 프로젝트를 해석해 어느 프로젝트에 플랜을 매달지 정한다
governedBy:
  - invariant.db-is-sot-plan-markdown-view
impacts: []
consumedBy:
  - integration.claude-code-hooks
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

Claude Code의 Plan Mode(계획 수립 모드)를 빠져나가는 순간을 가로채는 훅이다. 사람이 모델과 함께 세운 계획이 자유 형식 마크다운으로 흘러가 버리지 않도록, 그 출력을 Clawket의 정식 플랜으로 끌어들이는 다리 역할을 한다. 즉 "계획을 세웠다"와 "그 계획이 작업 관리 시스템에 등록됐다" 사이의 간극을 자동으로 메운다.

핵심은 두 단계다. 먼저 Plan Mode가 내놓은 마크다운이 Clawket이 요구하는 엄격한(strict) 플랜 형식에 맞는지 검증하고, 통과하면 그 자리에서 곧바로 Clawket 플랜으로 등록한다. 형식이 어긋나면 어디가 왜 틀렸는지(줄·열·위반 종류·고치는 힌트)를 짚어 주고 등록을 막아, 사람이 마크다운을 고치거나 이 프로젝트의 강제를 일시적으로 끄도록 안내한다.

## 요청 / 응답

입력은 Plan Mode가 산출한 플랜 마크다운이다. 이 마크다운은 두 경로로 들어올 수 있다 — 종료 도구 입력에 직접 실려 오거나(권장 경로), 그게 없으면 최근에 작성된 플랜 파일에서 읽어 온다. 출력은 검증 결과에 따라 갈린다.

- 형식 위반이면: 위반 위치·종류·고침 힌트를 담은 안내를 내보내고 작업을 막는다(block). 안내에는 두 선택지가 제시된다 — 마크다운을 형식에 맞게 고치거나, 이 프로젝트에서 Clawket 강제를 잠시 끄거나(이때도 파괴 명령 가드는 유지).
- 형식 통과이면: 검증기가 돌려준 파싱 결과(제목·본문·유닛 구성)로 현재 디렉터리의 프로젝트 아래에 플랜을 자동 생성하고, 등록된 플랜 식별자와 유닛 요약을 안내로 돌려준다. 데몬이 응답하지 않는 등 자동 등록 경로가 막히면 수동 등록을 유도하는 안내로 떨어진다.

## 권한 / 제약

Plan Mode 종료 시점에만 자동 실행된다. 비활성 프로젝트는 이 strict 게이트를 통째로 우회한다(나머지 훅 표면과 동일한 일관성). 검증 자체의 단일 진실은 데몬의 strict 임포트 검사이며, 이 훅은 그 판정을 받아 사람에게 전달하고 통과 시 등록까지 자동화하는 역할만 한다. 위반 안내의 문구(줄·열·힌트)는 사람이 틀린 줄을 찾는 계약이므로 잘리거나 가려지지 않게 그대로 전달한다.

## provenance

`clawket/hooks/hooks.json`이 PostToolUse를 matcher `ExitPlanMode`로 별도 어댑터에 연결한다. `clawket/adapters/claude/plan-sync.cjs`가 공유 핸들러(`clawket/adapters/shared/claude-hooks.cjs`의 `runPlanSync`)에 위임한다. 그 루틴에서 인라인/파일 두 경로의 마크다운 수집, 데몬 strict 검증(`validateStrictPlan`), 위반 시 block + 안내(`strictGuideMessage`), 통과 시 프로젝트 해석 후 플랜 자동 생성의 흐름을 확인했다.

## 미확정 (OPEN)
- [ ] OPEN: 자동 생성된 플랜이 draft로 등록되는지 active로 등록되는지(승인 절차와의 관계)와, 동일 Plan Mode 출력이 반복 등록될 때의 중복 방지 정책은 추가 정독 필요.
