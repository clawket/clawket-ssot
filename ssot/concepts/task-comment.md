---
id: concept.task-comment
kind: Concept
title: TaskComment (작업 코멘트)
definition: 작업(Task)에만 달 수 있는 코멘트로, 삭제 시 행을 지우는 대신 본문 앞에 [DELETED] 표식을 붙이는 소프트 삭제로 처리해 스레드의 연속성을 유지한다.
relatesTo:
  - to: concept.task
    type: belongs-to
    note: 코멘트는 항상 한 작업에 종속되며 작업 외 엔티티에는 달 수 없다.
  - to: concept.comment
    type: sibling-of
    note: 데몬의 일반 코멘트 개념을 데스크탑에서 작업 스코프로 좁혀 다룬 표현이다.
governedBy:
  - invariant.comment-task-scoped
  - invariant.comment-soft-delete
dependsOn:
  - concept.task
implementedIn:
  - desktop/apps/desktop/src/data/types.ts
  - desktop/apps/desktop/src/data/api.ts
  - desktop/apps/desktop/src/shell/TaskCommentsPanel.tsx
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:concept
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지(식별자는 provenance에만). 분기/의도 불명은 OPEN. -->

## 정의

TaskComment는 하나의 작업에 달린 댓글이다. 작성자, 본문, 작성 시각을 가지며 항상 특정 작업에 종속된다. 계획(Plan)이나 단위(Unit)에는 코멘트를 달 수 없다 — 데몬이 작업 외 대상의 코멘트를 거부하기 때문에, 코멘트는 사실상 작업 전용 수단이다.

가장 두드러진 특징은 **소프트 삭제**다. 코멘트를 지워도 해당 기록 자체를 데이터베이스에서 없애지 않고, 본문 맨 앞에 `[DELETED]` 표식을 붙여 "지워진 코멘트"로 표시만 바꾼다. 데스크탑 화면은 이 표식을 감지해 해당 코멘트를 흐리게(회색) 처리한다. 이렇게 하는 이유는 대화 스레드의 연속성을 보존하기 위해서다 — 중간 코멘트가 통째로 사라지면 앞뒤 맥락이 끊기지만, 표식만 남기면 "여기에 무언가 있었고 지워졌다"는 흐름이 유지된다. 이는 Clawket이 파괴적 삭제를 피하고 히스토리를 보존하려는 전반적 설계 방향과 일치한다.

작업 상세 패널의 코멘트 영역에서 사람이 코멘트를 작성하고 소프트 삭제할 수 있다.

## 엔티티 (DB)

작업 코멘트는 작업에 묶인 별도 기록으로 저장된다. 각 코멘트는 자신이 속한 작업, 작성자, 본문, 작성 시각을 갖는다. 삭제는 기록을 제거하는 연산이 아니라 본문 텍스트를 변형(앞에 표식 추가)하는 갱신 연산이라는 점이 데이터 모델상의 핵심이다. 따라서 "삭제된 코멘트"도 데이터로는 그대로 남아 있고, 단지 표식으로 식별된다.

## API 표면

- 코멘트 목록 조회·생성: 해당 작업의 코멘트 컬렉션을 통해 이뤄진다.
- 코멘트 수정·삭제: 개별 코멘트를 대상으로 하며, 삭제는 소프트 삭제(표식 부여)로 처리된다.

## 불변식

- **작업 스코프 전용**: 코멘트는 작업에만 달 수 있고, 단위·계획 코멘트는 데몬이 거부한다.
- **소프트 삭제 보존**: 삭제는 기록을 제거하지 않고 본문에 `[DELETED]` 표식을 붙여 스레드 연속성을 유지하며, 화면은 표식을 감지해 회색 처리한다.

## 구현 위치 (provenance)

- 코멘트 구조 정의와 소프트 삭제 표식 설명: `desktop/apps/desktop/src/data/types.ts`
- 코멘트 목록/생성/수정/삭제를 데몬으로 라우팅하는 API 글루: `desktop/apps/desktop/src/data/api.ts`
- 작업 상세의 코멘트 작성·삭제 UI: `desktop/apps/desktop/src/shell/TaskCommentsPanel.tsx`

## 미확정 (OPEN)
- [ ] OPEN: 소프트 삭제된 코멘트를 다시 복원하는 경로가 있는지(표식 제거 동작 존재 여부) 확인 필요.
