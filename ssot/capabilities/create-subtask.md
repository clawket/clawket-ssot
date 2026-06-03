---
id: capability.create-subtask
kind: Capability
title: 하위작업 만들기
definition: 데스크탑 앱에서 어떤 작업의 상세 화면을 열고, 그 아래에 더 잘게 쪼갠 하위작업을 손으로 만든다. 하위작업은 부모가 속한 묶음(유닛)과 진행 주기(사이클)를 기본으로 물려받고, 사람이 우선순위나 담당자를 따로 적으면 그 부분만 바꿔 만든다.
purpose: 큰 작업을 사람이 직접 손으로 더 작은 실행 단위로 나누되, 부모가 가진 분류(어느 묶음·어느 주기에 속하는지)를 다시 입력하지 않고 그대로 이어받게 한다.
servesPersona:
  - persona.human-operator
realizedBy:
  - component.desktop-app
implementedIn:
  - desktop/apps/desktop/src/shell/SubtaskCreateModal.tsx
  - desktop/apps/desktop/src/data/api.ts
relatesTo:
  - to: concept.task
    type: relates-to
    note: 부모 작업 아래에 새 작업(하위작업)을 만든다
  - to: capability.manage-task-lifecycle
    type: relates-to
    note: 만들어진 하위작업은 일반 작업과 같은 라이프사이클을 따른다
impacts:
  - concept.task
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

사람이 큰 작업을 더 작은 실행 단위로 직접 쪼갤 수 있다. 작업 상세 화면에서 "하위작업 추가"를 열면, 새 하위작업의 제목과 내용을 적어 부모 작업 밑에 매단다. 이때 핵심 가치는 **상속**이다 — 하위작업이 어느 묶음·어느 진행 주기에 속하는지는 부모에게서 자동으로 물려받으므로, 사람이 그 분류를 매번 다시 고를 필요가 없다. 다만 우선순위와 담당자는 부모 값이 미리 채워진 상태에서 필요하면 바꿀 수 있다.

이 화면은 사람이 직접 손으로 나누는 통로다. 시스템이 success_criteria로부터 하위작업을 제안하는 자동 분해와는 별개의, 수동 경로다.

## 행위

- 부모 작업의 상세에서 하위작업 생성 창을 연다(작업 식별 정보로 부모를 표시).
- 제목(필수)·내용(선택)·우선순위·담당자를 입력한다. 우선순위는 부모 값으로 시작하고, 담당자 칸을 비워 두면 부모의 담당자를 그대로 물려받는다.
- 제목이 비어 있으면 만들 수 없다(생성 버튼 비활성).
- 사람이 따로 입력해 부모와 달라진 값(우선순위·담당자·내용)만 함께 보내 하위작업을 만든다 — 바꾸지 않은 값은 부모에게서 상속된다.
- 생성에 실패하면 화면에 오류를 보여주고 창을 닫지 않는다.

## 시스템 흐름

작업 상세 → 하위작업 생성 창 → 입력값 검증(제목 필수) → 데스크탑 데이터 계층을 통해 데몬에 "부모 아래 새 작업" 생성 요청 → 데몬이 부모의 묶음·주기를 상속하고(명시 입력이 있으면 덮어씀) 작업을 만든다 → 성공 시 창을 닫고, 실패 시 오류 메시지를 표시한다.

## 어디에 구현되어 있나

데스크탑 앱의 하위작업 생성 모달(`SubtaskCreateModal.tsx`)이 입력 UI와 상속/덮어쓰기 규칙(부모 값과 다른 항목만 전송)을 담당한다. 실제 생성 호출은 데스크탑 데이터 계층(`data/api.ts`)이 데몬으로 위임한다.

## 미확정 (OPEN)
- [ ] OPEN: 상속되는 분류가 묶음·주기로 한정되는지, 그 외 필드(예: tier)도 부모에서 상속되는지 데몬 측 동작 미확인.
- [ ] OPEN: 하위작업 깊이 제한(몇 단계까지 중첩 가능한지) 여부 미확인.
