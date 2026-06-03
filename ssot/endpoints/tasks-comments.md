---
id: endpoint.tasks-comments
kind: Endpoint
title: 태스크 코멘트 목록 조회
definition: "GET /tasks/{id}/comments — 한 태스크에 달린 코멘트를 시간순으로 돌려준다(같은 라우터에서 작성·수정·소프트삭제도 함께 제공)."
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/comments.rs]
relatesTo:
  - { to: concept.task-comment, type: reads, note: "태스크에 달린 코멘트를 조회한다" }
  - { to: concept.task, type: reads, note: "코멘트는 태스크에 종속되며 대상 태스크 존재를 먼저 확인한다" }
governedBy: [invariant.comment-task-scoped, invariant.comment-soft-delete, invariant.preserve-history]
impacts: [concept.task-comment]
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
태스크에는 작업 진행 메모·결정·이슈를 코멘트로 붙일 수 있다. 이 동작은 특정 태스크에 달린 코멘트들을 모아 보여 준다. 같은 라우터가 코멘트 작성과 수정, 그리고 삭제까지 한 묶음으로 제공한다 — 다만 삭제는 기록을 실제로 지우는 것이 아니라 "지워진 것으로 표시"하는 방식이다.

## 요청 / 응답
조회는 대상 태스크를 가리키면 그 태스크의 코멘트 목록을 돌려준다. 각 코멘트는 작성자와 본문을 가진다. 태스크를 경로로 직접 지목하는 방식과, 코멘트 컬렉션에 태스크 식별자를 함께 넘겨 조회하는 방식 둘 다 지원한다(후자는 CLI 호환을 위한 경로다).

작성은 작성자와 본문을 받아 새 코멘트를 만든다. 수정은 본문을 새 내용으로 바꾼다. 삭제는 코멘트를 목록에서 사라지게 하는 대신 "삭제됨"으로 표시해 흔적을 남긴다.

## 권한 / 제약
코멘트는 태스크에만 붙는다. 유닛이나 플랜에 코멘트를 달려는 요청은 거부된다(저장 구조가 태스크 코멘트만 지원하기 때문). 컬렉션 방식으로 작성·조회할 때 태스크 식별자는 필수이며, 빠지면 잘못된 요청으로 거절된다. 대상 태스크가 없으면 "찾을 수 없음"으로 응답한다. 삭제는 물리적 삭제가 아니라 소프트 삭제(표시)로 처리해, 취소·삭제된 작업의 이력을 보존한다는 원칙을 따른다.

## provenance
구현은 데몬의 코멘트 라우터(daemon/src/routes/comments.rs)에 있다. 태스크 종속·소프트삭제·이력 보존 규칙은 governedBy가 가리키는 불변식이 정의한다.

## 미확정 (OPEN)
- [ ] OPEN: 코멘트 정렬 기준(생성순/수정순)과 작성자 식별자가 사람·에이전트를 어떻게 구분하는지는 확인 필요.
