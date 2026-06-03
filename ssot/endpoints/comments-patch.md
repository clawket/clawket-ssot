---
id: endpoint.comments-patch
kind: Endpoint
title: 코멘트 수정 / 소프트삭제 (데몬 PATCH·DELETE /comments/{id})
definition: 이미 달린 코멘트의 본문을 고쳐 쓰거나, 코멘트를 지우되 기록은 남기는 소프트삭제를 수행하는 데몬 엔드포인트.
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/comments.rs]
relatesTo:
  - { to: concept.comment, type: mutates, note: "코멘트 본문 수정 및 [DELETED] 표식 소프트삭제" }
  - { to: invariant.preserve-history, type: backed-by, note: "삭제가 행 제거가 아니라 표식 처리라 히스토리가 보존된다" }
governedBy: [invariant.preserve-history]
impacts: [concept.comment]
consumedBy: [endpoint.cli-comment]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
태스크에 이미 달려 있는 코멘트 하나를 대상으로, 그 본문을 새 내용으로 고쳐 쓰거나(수정) 또는 그 코멘트를 더 이상 보이지 않게 지우는(삭제) 동작을 제공한다. 사람이 웹 대시보드에서 코멘트를 편집할 때, 또는 에이전트가 `clawket comment update/delete` 명령을 쓸 때 이 엔드포인트로 흘러든다. 코멘트는 작업의 진행 메모이자 취소된 작업의 사연을 남기는 수단이므로, 이 엔드포인트의 핵심 성격은 "지워도 흔적은 남긴다"는 데 있다.

## 요청 / 응답
- 수정 요청은 대상 코멘트를 가리키는 식별자와 함께 새로 쓸 본문 한 가지를 보낸다. 응답으로는 갱신된 코멘트 한 건(작성자·본문 등)을 돌려준다.
- 삭제 요청은 대상 코멘트 식별자만 보낸다. 이때 실제로 행을 제거하는 것이 아니라 본문 앞에 "삭제됨" 표식을 붙이는 소프트삭제로 처리되며, 응답은 삭제 처리됐다는 확인(대상 식별자 포함)을 돌려준다.
- 존재하지 않는 코멘트를 가리키면 "찾을 수 없음"으로 응답한다.

## 권한 / 제약
로컬 데몬에만 노출되는 내부 API라 외부 인증 개념은 없고, 같은 기기의 CLI·웹 대시보드가 호출 주체다. 핵심 제약은 삭제가 파괴적이지 않다는 점 — 삭제 요청이 와도 코멘트 본문을 물리적으로 지우지 않고 표식만 붙여, 취소·삭제된 작업의 맥락을 사후에 추적할 수 있게 한다(히스토리 보존 불변식).

## provenance
- 라우터·핸들러: `daemon/src/routes/comments.rs`의 `PATCH /comments/{id}`(본문 수정)와 `DELETE /comments/{id}`(소프트삭제) 경로. 삭제는 저장소 계층의 soft_delete를 호출해 본문에 [DELETED] 접두를 붙인다.
- 제공 컴포넌트: clawketd 데몬(component.daemon).

## 미확정 (OPEN)
- [ ] OPEN: 소프트삭제된 코멘트가 목록 조회에서 어떻게 노출/필터되는지(완전 숨김 vs 표식 노출) 저장소 계층 정독으로 확인.
