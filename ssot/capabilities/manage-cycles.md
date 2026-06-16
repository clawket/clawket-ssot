---
id: capability.manage-cycles
kind: Capability
title: 사이클 생성·편집
definition: 유닛에 묶이는 사이클(스프린트)을 만들고 제목·목표를 고치며, 생성 시 어느 유닛에 속할지 고르는 기능. 변경은 데몬에 즉시 반영되고 위반 시 사유가 화면에 표시된다.
purpose: 사람 운영자가 작업을 묶을 타임박스 단위(스프린트)를 정의하고 그 목표를 명시하게 한다.
servesPersona:
  - persona.human-operator
realizedBy:
  - component.web-dashboard
implementedIn:
  - web/src/components/CycleCreateModal.tsx
  - web/src/components/CycleEditModal.tsx
relatesTo:
  - to: concept.cycle
    type: mutates
    note: 사이클을 생성하고 제목·목표를 편집하는 직접 대상
  - to: concept.unit
    type: reads
    note: 사이클은 반드시 하나의 유닛에 묶이며 생성 시 유닛을 고른다
impacts:
  - concept.cycle
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:capability"]
---

## 사용자가 할 수 있는 일

사람 운영자는 사이클(스프린트)을 새로 만들고 제목과 목표를 적을 수 있다. 사이클은 반드시 하나의 유닛에 속하므로, 생성 시 소속 유닛을 고른다(이미 특정 유닛 맥락에서 열렸으면 그 유닛으로 고정된다). 이미 만든 사이클의 제목·목표는 편집 모달에서 고친다. 목적은 작업을 묶을 타임박스 단위를 정의하고 그 사이클이 무엇으로 수렴해야 하는지(목표)를 분명히 남기는 것이다.

## 행위

- 생성 모달에서 제목(필수)·목표(선택)를 입력하고, 유닛이 미지정이면 프로젝트의 유닛 목록에서 하나를 고른다. 유닛이 지정된 채 열리면 그 유닛으로 고정 표시된다.
- 유닛 목록은 active 플랜의 유닛을 우선해 기본 선택한다.
- 편집 모달에서 기존 사이클의 제목·목표를 고친다. 바뀐 필드만 갱신 요청에 담는다.
- 제목이 비어 있으면 저장할 수 없고, 변경 사항이 없으면 편집 저장이 비활성화된다.
- 성공/실패는 토스트로 알리며, 실패 사유 메시지를 모달 안에도 표시한다.

## 시스템 흐름

생성 모달은 유닛 미지정 시 프로젝트의 플랜·유닛을 읽어 선택지를 만든다. 제출하면 선택한 유닛 아래에 사이클을 만드는 요청이 데몬으로 가고, 성공하면 호출한 화면(보드 등)에 새 사이클이 반영된다. 편집은 바뀐 제목·목표만 담은 갱신 요청으로 처리된다. 사이클 상태 전이(시작·종료)나 "유닛당 active 사이클 하나" 같은 규칙은 데몬이 강제하며, 위반 시 반환된 메시지가 모달과 토스트로 드러난다.

## 어디에 구현되어 있나

생성은 `web/src/components/CycleCreateModal.tsx`, 편집은 `web/src/components/CycleEditModal.tsx` 가 담당한다. 두 모달 모두 웹 대시보드가 데몬 API를 통해 사이클을 만들고 갱신한다. 이 모달들은 보드 화면 등에서 호출된다.

## 미확정 (OPEN)
- [ ] OPEN: 사이클 상태(planning/active/completed) 전환이 이 모달이 아닌 보드·백로그 화면에서 처리되는 분담 경계가 SSOT 상 명시되어야 함
