---
id: endpoint.runs-get
kind: Endpoint
title: 데몬 GET /runs/{id} (단일 실행 기록 조회)
definition: "식별자로 지정한 실행 기록 한 건의 전체 내용을 돌려주는 읽기 전용 조회로, 없으면 찾을 수 없음으로 응답한다."
realizedBy: [component.daemon]
implementedIn: ["daemon/src/routes/runs.rs", "daemon/src/repo/runs.rs"]
relatesTo:
  - { to: concept.run, type: reads, note: "지정한 실행 기록 한 건을 읽어 반환한다" }
  - { to: concept.execution-envelope, type: reads, note: "그 실행이 시작될 때 동결해 둔 봉투 식별자와 스냅샷을 함께 돌려준다" }
governedBy: []
impacts: []
consumedBy: [endpoint.cli-run, endpoint.cli-replay, component.web-dashboard]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
실행 기록 하나를 그 식별자로 콕 집어 전체 내용을 읽어오는 조회 기능이다. 특정 실행이 어떻게 시작되고 끝났는지, 어떤 봉투(실행 계약)로 돌아갔는지 자세히 들여다볼 때 쓰인다. 목록 조회로 후보를 좁힌 뒤 상세를 펼쳐 보는 흐름의 마지막 단계에 해당한다.

## 요청 / 응답
요청은 실행 기록의 식별자 하나뿐이다. 응답으로는 그 실행의 식별자, 매여 있는 태스크, 세션, 실행 주체(에이전트), 시작·종료 시각, 결과와 노트, 현재 상태(대기/진행 중/종료됨), 그리고 시작 시점에 동결된 실행 봉투 식별자와 봉투 스냅샷을 함께 돌려준다. 봉투 스냅샷이 있으면 해당 실행이 어떤 의도·성공 기준 아래 돌았는지 사후에 그대로 재구성할 수 있다.

## 권한 / 제약
- 읽기 전용 조회다. 이 경로로는 실행 기록을 바꾸지 않는다.
- 지정한 식별자에 해당하는 실행이 없으면 찾을 수 없음으로 응답한다.

## provenance
데몬 실행 라우트(`daemon/src/routes/runs.rs`)의 단건 조회 핸들러가 진입점이고, 실제 조회와 봉투 스냅샷 복원은 실행 저장소 계층(`daemon/src/repo/runs.rs`)이 담당한다. 제공 주체는 로컬 데몬이며, CLI의 실행·리플레이 명령과 웹 대시보드가 이 경로를 소비한다.

## 미확정 (OPEN)
- 없음.
