---
id: endpoint.comments-post
kind: Endpoint
title: 코멘트 작성·조회 (데몬 POST·GET /tasks/{id}/comments · /comments)
definition: 특정 태스크에 코멘트를 새로 달거나, 태스크에 달린 코멘트 목록을 조회하는 데몬 엔드포인트.
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/comments.rs]
relatesTo:
  - { to: concept.comment, type: mutates, note: "태스크에 코멘트 생성" }
  - { to: concept.comment, type: reads, note: "태스크별 코멘트 목록 조회" }
  - { to: concept.task, type: reads, note: "코멘트는 태스크에만 붙으며 대상 태스크 존재를 확인" }
governedBy: []
impacts: [concept.comment]
consumedBy: [endpoint.cli-comment]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
태스크에 작업 메모·기록을 남기는 코멘트를 새로 작성하거나, 한 태스크에 달린 코멘트들을 시간순으로 조회한다. 진행 상황 기록, 결정의 배경 메모, 취소된 작업의 사연 남기기 등에 쓰인다. 같은 기능이 두 가지 경로 모양으로 제공되는데, 하나는 URL에 태스크를 박는 형태(`/tasks/{id}/comments`), 다른 하나는 컬렉션 경로에 대상 태스크를 본문/쿼리로 싣는 형태(`/comments`)다. 후자는 CLI 호환을 위해 추가된 것으로, CLI가 태스크 식별자를 본문에 담아 보내던 방식을 받아주기 위함이다.

## 요청 / 응답
- 작성 요청은 작성자와 코멘트 본문, 그리고 대상 태스크 식별자를 보낸다(URL 또는 본문 중 하나에). 응답은 방금 만든 코멘트 한 건을 돌려준다.
- 조회 요청은 대상 태스크 식별자만 보내고, 응답으로 그 태스크의 코멘트 목록을 돌려준다.
- 코멘트는 오직 태스크에만 붙을 수 있다 — 유닛이나 플랜을 대상으로 한 코멘트 작성 요청은 "지원하지 않음"으로 거절한다. 대상 태스크가 없으면 "찾을 수 없음", 대상 식별자가 빠지면 "필수 항목 누락"으로 응답한다.

## 권한 / 제약
로컬 데몬 내부 API로 외부 인증은 없고 CLI·웹이 호출한다. 데이터 모델상 코멘트 저장 대상은 태스크뿐이라(유닛/플랜용 테이블 없음), 유닛·플랜 코멘트 시도는 의도적으로 막는다. 대상 태스크 식별자는 작성·조회 모두에서 필수다.

## provenance
- 라우터·핸들러: `daemon/src/routes/comments.rs`의 `GET·POST /tasks/{id}/comments`(태스크 경로형)와 `GET·POST /comments`(컬렉션형, CLI 호환). 컬렉션형은 unit_id/plan_id가 오면 거부하고 task_id를 강제한다.
- 제공 컴포넌트: clawketd 데몬(component.daemon).

## 미확정 (OPEN)
- [ ] OPEN: 코멘트 작성 시 작성자(author) 값의 출처·검증 규칙(에이전트명/사람 식별 등) 확인.
