---
id: endpoint.runs-start
kind: Endpoint
title: 데몬 POST /runs/{id}/start (실행 시작 표시)
definition: "대기 상태로 만들어 둔 실행 기록을 진행 중으로 전환하는 동작으로, 대기 상태일 때만 전환되고 그 외 상태는 그대로 둔다."
realizedBy: [component.daemon]
implementedIn: ["daemon/src/routes/runs.rs", "daemon/src/repo/runs.rs"]
relatesTo:
  - { to: concept.run, type: mutates, note: "대기 상태의 실행을 진행 중으로 전환한다" }
governedBy: []
impacts: []
consumedBy: [endpoint.cli-run]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
이미 "대기" 상태로 예약해 둔 실행 기록을 실제로 "진행 중"으로 바꾸는 동작이다. 실행을 두 단계로 나누는 흐름 — 먼저 봉투 스냅샷만 받아 두는 대기 단계, 그다음 실제로 일을 시작하는 단계 — 에서 두 번째 단계에 해당한다. 실행을 생성할 때 곧바로 "진행 중"으로 열었다면 이 동작은 필요 없다.

## 요청 / 응답
요청은 전환할 실행 기록의 식별자 하나뿐이며 본문은 없다. 응답으로는 갱신된 실행 기록 한 건을 돌려준다. 전환에 성공하면 실시간 이벤트("실행 갱신됨, 진행 중")가 함께 발행된다.

## 권한 / 제약
- 상태 전환은 그 실행이 "대기" 상태일 때만 일어난다. 이미 진행 중이거나 종료된 실행에 대해 호출하면 상태는 바뀌지 않는다(조용히 현 상태 유지). 즉 대기→진행 중 한 방향 전환만 의미를 가진다.
- 지정한 식별자에 해당하는 실행이 없으면 찾을 수 없음으로 응답한다.

## provenance
데몬 실행 라우트(`daemon/src/routes/runs.rs`)의 시작 핸들러가 진입점이고, "대기 상태일 때만 진행 중으로 전환"이라는 조건부 갱신은 실행 저장소 계층(`daemon/src/repo/runs.rs`)의 시작 표시 로직에 들어 있다. 제공 주체는 로컬 데몬이며, CLI의 실행 명령이 이 경로를 소비한다.

## 미확정 (OPEN)
- 없음.
