---
id: capability.task-questions
kind: Capability
title: 작업 질문 만들기·답하기
definition: 데스크탑 앱의 작업 상세 화면에서 그 작업에 대해 명확화·결정·블로커 종류의 질문을 만들고, 열린 질문에 답을 적어 닫는다. 답이 달린 질문은 답변과 함께 읽기 전용으로 보이고, 답이 없는 질문은 답변 입력칸을 연다.
purpose: 작업을 진행하다 막히거나 사람의 결정이 필요할 때, 그 요청을 작업 옆에 질문으로 남기고 답을 받아 진행을 이어가게 한다.
servesPersona:
  - persona.human-operator
realizedBy:
  - component.desktop-app
implementedIn:
  - desktop/apps/desktop/src/shell/TaskQuestionsPanel.tsx
  - desktop/apps/desktop/src/data/api.ts
relatesTo:
  - to: concept.question
    type: relates-to
    note: 작업에 매인 질문을 생성·조회·답변한다
  - to: concept.task
    type: relates-to
    note: 질문은 특정 작업에 붙는다
  - to: endpoint.cli-question
    type: relates-to
    note: 같은 질문 데이터를 CLI 질문 명령과 공유한다
impacts:
  - concept.question
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:capability
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로. 코드 옮겨적기 금지. -->

## 사용자가 할 수 있는 일

사람(또는 에이전트)이 작업을 진행하다 사람의 판단이 필요한 지점을 질문으로 남기고, 다른 사람이 그 질문에 답할 수 있다. 질문에는 세 종류가 있다 — 모호함을 풀기 위한 **명확화(clarification)**, 갈림길에서의 **결정(decision)**, 진행을 막는 **블로커(blocker)**. 답이 달리지 않은 질문은 "열림"으로 표시되며 답변 입력칸이 열려 있고, 답이 달리면 "답변됨"으로 바뀌어 답변 내용과 함께 읽기 전용으로 굳는다.

## 행위

- 작업 상세를 열면 그 작업의 질문 목록을 불러와 보여준다(열린 질문이 없으면 "열린 질문 없음").
- 질문을 만든다 — 종류(명확화/결정/블로커)를 고르고 본문을 적어 등록한다. 종류를 따로 고르지 않으면 명확화가 기본이다.
- 열린 질문에 답을 적어 닫는다. 답을 보내면 그 질문은 답변된 상태로 바뀌고 답변자·답변 시각과 함께 표시된다.
- 질문을 만들거나 답을 단 뒤에는 목록을 다시 불러와 최신 상태를 반영한다.

## 시스템 흐름

작업 상세 → 질문 패널이 그 작업의 질문 목록을 읽어 종류 배지와 함께 렌더 → 사람이 질문을 만들면 데몬에 등록(종류·출처 기본값은 서버가 채움) 후 재조회 → 열린 질문에 답을 보내면 데몬이 질문을 답변 상태로 갱신 → 재조회로 답변 블록이 화면에 반영. 답변은 서버 상태를 바꾸므로 성공 후 목록을 다시 읽는다.

## 어디에 구현되어 있나

데스크탑 앱의 작업 질문 패널(`TaskQuestionsPanel.tsx`)이 질문 목록·종류 선택·생성·답변 UI와 재조회 흐름을 담당한다. 조회·생성·답변 호출은 데스크탑 데이터 계층(`data/api.ts`)을 거쳐 데몬으로 간다.

## 미확정 (OPEN)
- [ ] OPEN: 질문을 삭제하거나 답변을 수정하는 경로가 데스크탑 화면에 있는지 미확인(이 패널은 생성·답변만 노출).
- [ ] OPEN: 블로커 질문이 작업을 자동으로 blocked 상태로 전환하는 등 라이프사이클과 연동되는지 미확인.
