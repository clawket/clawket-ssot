---
id: capability.task-comments-questions
kind: Capability
title: 태스크 코멘트·질문 관리
definition: 태스크 상세에서 코멘트를 작성·삭제하고, 사람의 명확화·결정을 요청하는 질문과 그 답변을 함께 보는 기능이다.
servesPersona:
  - persona.builder
  - persona.human-operator
realizedBy:
  - component.web
  - component.web-api-client
implementedIn:
  - web/src/components/task-detail/TaskComments.tsx
  - web/src/components/task-detail/TaskSections.tsx
  - web/src/api.ts
relatesTo:
  - to: concept.comment
    type: mutates
    note: 태스크에 묶인 코멘트를 작성·삭제한다 — 취소된 작업의 맥락을 보존하는 수단으로도 쓰인다.
  - to: concept.question
    type: reads
    note: 사람 명확화·결정 요청 질문과 그 답변을 표시한다.
  - to: concept.task
    type: belongs-to
    note: 코멘트·질문은 태스크 상세 표면의 일부다.
impacts:
  - domain.task-management
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:capability
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로. -->

## 사용자가 할 수 있는 일
사용자는 태스크 상세 화면에서 그 태스크에 코멘트를 남겨 작업 맥락·결정·메모를 기록하고, 필요 없어진 코멘트를 지운다. 코멘트는 파괴적 삭제 대신 히스토리를 보존하는 수단으로도 쓰인다. 같은 화면에서 그 태스크에 달린 질문(명확화·결정·블로커 요청)과 답변(누가 무엇이라 답했는지)을 함께 확인한다.

## 행위
- 코멘트 목록을 작성자·작성시각과 함께 보여주고, 작성자/본문을 입력해 새 코멘트를 추가한다.
- 개별 코멘트를 삭제한다.
- 작성자·본문이 비어 있으면 추가 버튼이 비활성화되고, 전송 중에는 중복 전송을 막는다.
- 질문 섹션에서 각 질문의 종류(kind), 질문자, 본문, 그리고 답변이 있으면 답변자와 답변 내용을 표시한다(현재 웹 표면은 질문 읽기 중심).

## 시스템 흐름
태스크 상세가 열리면 코멘트·질문 데이터가 상태로 들어온다. 코멘트 추가/삭제는 데몬 API(웹 API 클라이언트 경유)로 보내고, 성공 시 화면의 코멘트 목록을 갱신한다. 질문 섹션은 받은 질문 목록을 그대로 렌더링하며, 답변이 채워진 질문은 답변자·답변을 강조해 보여준다.

## 어디에 구현되어 있나
코멘트 작성·삭제 UI는 웹 앱의 태스크 코멘트 컴포넌트(web/src/components/task-detail/TaskComments.tsx)에, 질문 표시는 태스크 섹션 컴포넌트(web/src/components/task-detail/TaskSections.tsx)의 질문 섹션에 있고, 서버 호출은 데몬 API 래퍼(web/src/api.ts)를 통한다.

## 미확정 (OPEN)
- [ ] OPEN: 코멘트/질문을 서빙하는 데몬 엔드포인트 노드와의 mutates/reads 엣지 정렬(web 스코프 전용 노드 미확인 — daemon.comments/daemon.questions 는 desktop 스코프)
- [ ] OPEN: 웹에서 질문 생성·답변(쓰기)을 직접 지원하는지 여부(현재 코드상 질문 섹션은 읽기 표시 중심)
