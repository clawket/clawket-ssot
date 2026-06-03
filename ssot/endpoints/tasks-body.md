---
id: endpoint.tasks-body
kind: Endpoint
title: 데몬 POST /tasks/{id}/body
definition: 한 태스크의 본문에 내용을 덧붙이는 통로. 기존 설명을 통째로 갈아끼우지 않고 진행 메모·발견 사항을 누적해 작업 흐름을 보존한다.
realizedBy:
  - component.daemon
implementedIn:
  - daemon/src/routes/tasks.rs
relatesTo:
  - to: concept.task
    type: mutates
    note: 태스크 본문에 텍스트를 덧붙인다
  - to: component.daemon
    type: backed-by
    note: 덧붙임을 적용하는 로컬 데몬
governedBy:
  - invariant.preserve-history
impacts:
  - concept.task
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
한 태스크의 본문(설명·메모)에 내용을 "덧붙이는" 통로다. 기존 본문을 통째로 새 내용으로 바꾸는 것이 아니라, 뒤에 이어 붙여 작업 진행 중에 알게 된 것, 시도한 것, 다음에 할 것 같은 흐름을 누적한다. 작업을 진행하며 발견 사항을 차곡차곡 기록해 두면 나중에 "어떤 경로로 여기까지 왔나"를 되짚을 수 있다. 본문 전체를 교체하는 일반 수정과는 의도가 다르다 — 이쪽은 히스토리를 지우지 않고 쌓는 쪽이다.

## 요청 / 응답
보내는 정보는 본문 끝에 이어 붙일 텍스트 한 덩어리다. 응답으로는 덧붙임이 반영된 태스크 전체를 돌려주어, 호출 측이 갱신된 본문을 바로 확인할 수 있다. 대상 태스크가 없으면 "찾을 수 없음"으로 응답한다.

## 권한 / 제약
데몬이 로컬 루프백에만 바인딩되므로 같은 기기 안에서만 호출된다. 이 통로는 기존 본문을 보존하고 뒤에 더하는 누적 방식이라, 과거 기록을 덮어 지우지 않는다(히스토리 보존 성격). 본문 전체를 갈아끼우려면 일반 태스크 수정 통로를 쓴다 — 두 경로의 역할이 분리되어 있다.

## provenance
근거는 `daemon/src/routes/tasks.rs`의 라우트 등록(`/tasks/{id}/body`, POST → `append_body`)과 핸들러 `append_body`다. 핸들러는 경로의 태스크 식별자와 요청 본문의 텍스트(`text`)를 받아 `tasks::append_body`를 호출하고, 결과 태스크를 돌려주거나 없으면 404로 응답한다. 적용 주체는 로컬 데몬(component.daemon)이다.

## 미확정 (OPEN)
- [ ] OPEN: 덧붙임의 구분 표기(타임스탬프·작성자·구분선 삽입 여부)와, 덧붙임이 본문 검색 인덱스·임베딩 재계산을 트리거하는지는 `tasks::append_body` 본체와 임베딩 스케줄 코드를 정독해 확인 필요.
