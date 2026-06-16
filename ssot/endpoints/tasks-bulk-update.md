---
id: endpoint.tasks-bulk-update
kind: Endpoint
title: 데몬 POST /tasks/bulk-update
definition: 여러 태스크를 한 번의 요청으로 같은 필드 변경(예: 상태 일괄 전환)을 적용하는 통로. 보드에서 여러 카드를 한꺼번에 옮기는 식의 일괄 정리에 쓴다.
realizedBy:
  - component.daemon
implementedIn:
  - daemon/src/routes/tasks.rs
relatesTo:
  - to: concept.task
    type: mutates
    note: 지정된 여러 태스크에 동일한 필드 변경을 반복 적용한다
  - to: concept.task-status
    type: mutates
    note: 상태 필드를 함께 보내면 여러 태스크의 상태를 일괄 전환한다
  - to: component.daemon
    type: backed-by
    note: 변경을 적용하고 변경마다 이벤트를 흘려보내는 로컬 데몬
governedBy:
  - invariant.task-patch-vs-structural
impacts:
  - concept.task
  - concept.task-status
consumedBy:
  - endpoint.cli-task
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의
여러 태스크에 똑같은 변경을 한 번에 거는 통로다. 대상 태스크들의 식별자 묶음과, 그들에 공통으로 적용할 변경 내용을 함께 보내면, 데몬이 각 태스크에 같은 변경을 차례로 적용한다. 칸반 보드에서 여러 카드를 한꺼번에 다른 칼럼으로 옮기거나, 한 사이클의 태스크들을 일괄로 정리할 때처럼 "같은 손질을 여러 건에" 하는 상황에 쓴다. 한 건씩 따로 부르는 수고를 줄이는 편의 경로이며, 적용되는 변경의 규칙(어떤 전환이 허용되는지 등)은 단건 변경과 동일하다.

## 요청 / 응답
보내는 정보는 두 부분이다 — 손질할 대상 태스크들의 식별자 목록과, 그 전부에 똑같이 입힐 변경 내용(예: "상태를 done으로", "담당자를 바꿈" 같은 필드 묶음). 변경 내용은 단건 변경에 쓰는 것과 같은 형태의 필드 묶음이라, 단건 변경이 허용하는 것은 일괄에서도 허용된다. 응답으로는 실제로 갱신된 태스크들의 목록을 돌려준다 — 존재하지 않아 갱신되지 못한 식별자는 결과에서 빠진다. 또한 한 태스크가 바뀌면서 상위(유닛·사이클 등)가 연쇄로 영향받을 경우, 그 연쇄 변화도 실시간 이벤트로 함께 알린다.

## 권한 / 제약
데몬이 로컬 루프백에만 바인딩되므로 같은 기기 안에서만 호출된다. 변경은 목록 순서대로 한 건씩 적용되며, 각 건은 단건 변경과 같은 검증·규칙을 거친다 — 즉 일괄이라고 해서 규칙이 느슨해지지 않는다. 목록 중 존재하지 않는 식별자는 조용히 건너뛰고(결과에서 제외) 나머지 처리를 계속한다. 따라서 "전부 성공 아니면 전부 취소"의 한 묶음 거래는 아니며, 일부만 반영될 수 있다는 점에 유의한다.

## provenance
근거는 `daemon/src/routes/tasks.rs`의 라우트 등록(`/tasks/bulk-update`, POST → `bulk_update`)과 핸들러 `bulk_update`다. 요청은 대상 식별자 목록(`ids`)과 공통 변경 필드(`fields`)로 구성되며, 핸들러는 각 식별자에 대해 단건 갱신과 동일한 `tasks::update`를 호출하고, 갱신된 태스크만 결과 벡터에 모은다. 상위 연쇄 변화(`cascade_events`)는 변경마다 데몬 이벤트로 방출된다(주석 FIX-DAEMON-106). 적용·이벤트 방출 주체는 로컬 데몬(component.daemon)이다.

## 미확정 (OPEN)
- [ ] OPEN: 일괄 변경에서 일부 실패 시의 부분 반영 정책이 의도된 설계인지(트랜잭션으로 묶을지)와, `fields`로 허용/금지되는 필드 목록의 정확한 경계는 단건 갱신(`parse_update`) 코드와 함께 owner 확인 필요.
