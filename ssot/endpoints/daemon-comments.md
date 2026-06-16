---
id: endpoint.daemon-comments
kind: Endpoint
title: 데몬 코멘트 API (/tasks/:id/comments, /comments/:id)
definition: "작업에 달리는 코멘트를 다루는 데몬 엔드포인트. 작업별 코멘트 목록·생성은 작업 경로 아래에, 수정·삭제는 코멘트 경로(/comments/:id) 아래에 있으며, 삭제는 행을 지우지 않고 본문에 [DELETED] 표식을 붙이는 소프트 삭제다."
realizedBy: [component.daemon]
implementedIn:
  - desktop/apps/desktop/src/data/api.ts
relatesTo:
  - { to: concept.comment, type: mutates, note: "코멘트의 생성·수정·소프트 삭제를 다룬다" }
  - { to: concept.task-comment, type: mutates, note: "코멘트는 작업에만 붙으며 task_comments에 저장된다" }
  - { to: concept.task, type: reads, note: "코멘트는 항상 특정 작업에 종속된다" }
governedBy:
  - invariant.comment-task-scoped
  - invariant.comment-soft-delete
impacts: [concept.comment]
consumedBy: [component.web-dashboard]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

작업에 달리는 코멘트를 읽고 쓰는 데몬 API다. 코멘트는 작업 진행 중의 메모·결정 기록 수단이자, 작업을 파괴적으로 지우는 대신 "취소" 처리할 때 그 사유와 맥락을 남겨 히스토리를 보존하는 수단으로도 쓰인다.

경로가 둘로 나뉜다 — 특정 작업의 코멘트를 나열·추가할 때는 그 작업 경로 아래(/tasks/:id/comments)를 쓰고, 이미 만들어진 개별 코멘트를 수정·삭제할 때는 코멘트 자체 경로(/comments/:id)를 쓴다.

## 요청 / 응답

- **목록**: 한 작업에 달린 코멘트들을 시간순으로 돌려준다. 각 항목은 본문·작성자·작성 시각을 담는다.
- **생성**: 본문을 보내 코멘트를 단다. 작성자를 명시하지 않으면 기본 작성자(에이전트 측 "main")로 귀속된다.
- **수정**: 개별 코멘트의 본문을 고친다.
- **삭제(소프트)**: 행을 실제로 지우지 않고 본문 앞에 삭제 표식을 붙여 둔다. 스레드 연속성을 위해 항목 자체는 남고, 화면 렌더러가 표식을 감지해 회색 처리한다.

## 권한 / 제약

- 코멘트는 작업에만 붙는다. 데몬은 유닛·플랜에 대한 코멘트 요청을 거부한다 — 저장 스키마가 작업 코멘트만 담기 때문이다(governedBy: 코멘트 작업 스코프 불변식).
- 삭제는 항상 소프트 삭제다. 데이터를 물리적으로 제거하지 않고 표식만 부여해 히스토리를 보존한다(governedBy: 코멘트 소프트 삭제 불변식).
- 로컬 데몬 특성상 호출 주체는 같은 머신의 사람·에이전트로 한정된다.

## provenance

- implementedIn: 데스크탑 데몬 클라이언트가 작업별 목록/생성과 /comments/:id 수정/삭제의 경로 분리, 소프트 삭제 표식, 기본 작성자 규약을 주석으로 명시한다.
- realizedBy: clawketd 데몬이 코멘트 저장소를 소유하고 이 API를 제공한다.

## 미확정 (OPEN)
- [ ] OPEN: 소프트 삭제된 코멘트를 복구하거나 영구 제거하는 경로의 존재 여부.
