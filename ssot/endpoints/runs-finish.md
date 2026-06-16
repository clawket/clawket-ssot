---
id: endpoint.runs-finish
kind: Endpoint
title: 데몬 POST /runs/{id}/finish (실행 종료 기록)
definition: "진행 중인 실행 기록을 종료 상태로 마감하면서 결과와 노트, 종료 시각을 함께 남기는 동작이다."
realizedBy: [component.daemon]
implementedIn: ["daemon/src/routes/runs.rs", "daemon/src/repo/runs.rs"]
relatesTo:
  - { to: concept.run, type: mutates, note: "실행을 종료 상태로 마감하고 결과·노트·종료 시각을 기록한다" }
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
열려 있던 실행 기록을 닫고 그 결과를 남기는 마감 동작이다. 에이전트가 한 태스크에 대한 실행을 마쳤을 때, 무엇으로 끝났는지(성공/실패 등 결과)와 부가 메모를 남기며 실행을 종료 처리한다. 실행이 종료되어야 같은 태스크에 다음 실행을 새로 열 수 있다(미종료 실행이 남아 있으면 새 실행 생성이 거부되므로).

## 요청 / 응답
요청에는 실행 기록의 식별자와, 그 실행의 결과(필수)와 선택적인 노트를 보낸다. 마감 시점의 종료 시각은 데몬이 현재 시각으로 기록하고, 상태를 "종료됨"으로 바꾼다. 응답으로는 갱신된 실행 기록 한 건을 돌려주며, 실시간 이벤트("실행 갱신됨, 종료됨"에 결과 포함)가 함께 발행된다.

## 권한 / 제약
- 종료 시각·결과·노트를 채우고 상태를 "종료됨"으로 바꾸는 갱신이다.
- 지정한 식별자에 해당하는 실행이 없으면 찾을 수 없음으로 응답한다.

## provenance
데몬 실행 라우트(`daemon/src/routes/runs.rs`)의 종료 핸들러가 진입점이고, 종료 시각·결과·노트 기록과 상태 전환은 실행 저장소 계층(`daemon/src/repo/runs.rs`)의 마감 로직에 들어 있다. 제공 주체는 로컬 데몬이며, CLI의 실행 명령이 이 경로를 소비한다. 한 태스크에 미종료 실행이 하나만 존재하도록 하는 불변식은 실행 생성 측(목록·생성 엔드포인트)에서 강제되며, 그 전제 조건이 바로 이 종료 동작이다.

## 미확정 (OPEN)
- 없음.
