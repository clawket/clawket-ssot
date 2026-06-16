---
id: endpoint.daemon-units
kind: Endpoint
title: 단위 관리 API (데몬)
definition: "데몬이 제공하는 단위(Unit) CRUD. 계획별 목록·단건 조회, 생성, 수정, 삭제를 담당한다. 단위는 상태가 없는 순수 묶음 엔티티이며, 수정 시 '목표' 값은 그대로 둠/지움/설정의 세 가지 의도를 구분해 보낸다."
realizedBy:
  - component.daemon
implementedIn:
  - desktop/apps/desktop/src/data/api.ts
relatesTo:
  - to: concept.unit
    type: mutates
    note: "단위 엔티티의 조회·생성·수정·삭제를 수행한다."
  - to: concept.plan
    type: reads
    note: "단위는 계획에 속하며, 목록을 계획 단위로 걸러 조회한다."
governedBy:
  - invariant.unit-no-status
impacts:
  - concept.unit
  - concept.plan
consumedBy:
  - component.renderer
  - component.daemon-client
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

단위는 계획 아래에서 관련된 작업을 묶기 위한 그룹이다. 계획·사이클·태스크와 달리 단위에는 진행 상태가 없다 — 순수하게 "묶음" 역할만 한다. 따라서 이 API에는 승인·활성화 같은 상태 전이가 없고, 만들고·고치고·지우고·보는 네 동작만 있다.

- 목록·단건 조회: 계획별로 단위들을 보거나 한 건을 자세히 본다.
- 생성: 계획과 제목을 필수로, 목표·정렬 순서·실행 방식을 곁들여 만든다.
- 수정: 제목·목표·실행 방식을 바꾼다.
- 삭제: 단위를 지운다.

## 요청 / 응답

- 목록: 계획 범위(선택)를 지정 → 그 계획의 단위 목록. 범위를 안 주면 전체.
- 단건: 단위 지정 → 단위 한 건.
- 생성: 계획과 제목(필수) + 목표·정렬 순서·실행 방식(선택) → 만들어진 단위.
- 수정: 바꿀 항목만 → 갱신된 단위. 이때 "목표"는 세 가지 의도를 구분해 보낸다 — 항목을 안 보내면 "그대로 둠", 빈값(없음)으로 보내면 "목표를 지움", 문자열로 보내면 "그 값으로 설정". (이 세 상태 인코딩은 "안 바꿈"과 "비움"을 헷갈리지 않게 하려는 약속이다.) "실행 방식"은 그런 비움 구분 없이 값을 줄 때만 바뀐다.
- 삭제: 대상 지정 → "지웠음 + 지운 대상 식별자".

## 권한 / 제약

- 단위는 상태가 없다(순수 묶음). 승인·활성화 흐름이 없다.
- 수정의 "목표" 값은 그대로/비움/설정 세 의도를 명확히 구분한다 — UI가 "목표를 지우는 것"과 "건드리지 않는 것"을 다르게 다룰 수 있게 한다.
- 데몬 직접 접속 클라이언트가 호출하며 세션 토큰 인증 위에서 동작한다.

## provenance

- 클라이언트 측 계약: `desktop/apps/desktop/src/data/api.ts` (`listUnits` / `getUnit` / `createUnit` / `updateUnit` / `deleteUnit`).
- 제공 컴포넌트: `component.daemon`.
- 강제 규칙: `invariant.unit-no-status`(단위는 상태 없는 순수 묶음).
- 대상 엔티티: `concept.unit`.

## 미확정 (OPEN)
- [ ] OPEN: "실행 방식(execution_mode)"이 가질 수 있는 값과 그 의미(예: 병렬/순차 등)를 데몬 측에서 확인.
- [ ] OPEN: 단위 삭제가 하위 사이클·태스크에 미치는 연쇄 효과 확인.
