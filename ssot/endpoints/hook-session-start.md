---
id: endpoint.hook-session-start
kind: Endpoint
title: 훅 SessionStart
definition: "세션이 새로 시작되거나 초기화될 때 Claude Code가 호출하는 훅으로, 런타임(CLI·데몬·웹) 설치를 보장하고 데몬을 띄운 뒤 현재 작업 디렉터리의 프로젝트 대시보드와 작업 규칙을 세션 컨텍스트로 주입한다."
realizedBy:
  - component.claude-adapter
  - component.install-gate
implementedIn:
  - clawket/hooks/hooks.json
  - clawket/adapters/claude/session-start.cjs
  - clawket/adapters/shared/claude-hooks.cjs
relatesTo:
  - to: endpoint.cli-dashboard
    type: relates-to
    note: 활성 프로젝트의 작업 요약을 끌어와 세션 컨텍스트의 본문으로 주입한다
  - to: concept.active-project
    type: reads
    note: 현재 작업 디렉터리가 어떤 프로젝트에 속하는지 해석해 등록 여부에 따라 동작을 가른다
  - to: concept.scenario
    type: reads
    note: 진행 중 태스크 중 시나리오가 비어 있는 것을 찾아 위험 경고 줄을 덧붙인다
governedBy:
  - invariant.path-separation-xdg
impacts: []
consumedBy:
  - integration.claude-code-hooks
  - component.install-gate
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

Claude Code 세션이 처음 켜지거나, 대화가 비워지거나(clear), 컨텍스트가 압축될 때(compact) 자동으로 한 번 실행되는 진입점이다. 사람이 직접 부르는 명령이 아니라 Claude Code가 세션 라이프사이클의 시작 시점에 발화시키는 훅이며, Clawket이 그 세션 동안 제대로 동작할 수 있도록 바닥을 까는 부트스트랩 역할을 한다.

세 가지 일을 순서대로 한다. 첫째, 이번 머신에 Clawket 런타임(명령줄 도구·로컬 데몬·웹 번들)이 설치돼 있는지 확인하고 없으면 받아 설치한다(설치 게이트). 둘째, 로컬 데몬이 살아 있도록 보장한다. 셋째, 지금 열린 작업 디렉터리가 등록된 Clawket 프로젝트라면 그 프로젝트의 현재 작업 상황(진행 중인 플랜·사이클·태스크 요약)과 작업 규칙 문서를 세션에 컨텍스트로 밀어 넣어, 에이전트가 "지금 무슨 일을 하던 중이었는지"를 즉시 인지하게 한다.

## 요청 / 응답

이 훅은 세션 시작 신호와 현재 작업 디렉터리만 입력으로 받는다. 응답은 Claude Code가 이해하는 두 갈래의 출력이다 — 하나는 모델에게 보이지 않게 세션에 끼워 넣는 추가 컨텍스트(작업 대시보드 요약 + 규칙 문서), 다른 하나는 사람에게 보이는 짧은 상태 줄(데몬 가동 여부, 웹 대시보드 주소)이다.

분기 동작이 핵심이다. 설치나 데몬 보장이 실패하면 컨텍스트 주입을 포기하고 무엇이 잘못됐는지 알려 주는 경고만 내보낸다 — 망가진 데몬 위에서 작업 요약을 보여 줘 봐야 의미가 없기 때문이다. 현재 디렉터리가 어떤 프로젝트에도 속하지 않으면 아무것도 주입하지 않고 조용히 끝낸다(Clawket이 관리하지 않는 폴더에 작업 규칙을 들이밀지 않는다). 등록된 프로젝트일 때만 작업 요약과 규칙을 주입하며, 이때 진행 중 태스크 가운데 시나리오가 연결되지 않은 것이 있으면 별도 위험 경고 줄을 함께 띄운다.

## 권한 / 제약

세션당 한 번 자동 실행되며 사람의 호출 권한 개념은 없다. 설치 단계가 실패해도 세션 자체를 막지는 않는다 — Clawket 부트스트랩 실패가 Claude Code 기동을 가로막아서는 안 된다는 원칙이다. 등록되지 않은 디렉터리에서는 의도적으로 무동작이며, 데이터·경로는 사용자 홈 아래 표준 위치로 분리해 둔다(경로 분리 불변식).

## provenance

`clawket/hooks/hooks.json`이 SessionStart 이벤트를 matcher `startup|clear|compact`로 어댑터에 연결한다. 얇은 진입점 `clawket/adapters/claude/session-start.cjs`가 공유 핸들러의 세션 시작 루틴(`clawket/adapters/shared/claude-hooks.cjs`의 `runSessionStart`)에 위임한다. 그 루틴에서 설치 보장 → 데몬 보장 → 대시보드 조회·규칙 읽기 → 등록 여부 분기 → 컨텍스트 주입의 흐름과, 시나리오 누락 태스크에 대한 위험 경고 합성을 확인했다.

## 미확정 (OPEN)
- [ ] OPEN: compact 매처로 재실행될 때 이미 주입된 컨텍스트와의 중복 처리·갱신 정책이 어떻게 되는지 추가 정독 필요.
