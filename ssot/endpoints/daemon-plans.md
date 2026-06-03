---
id: endpoint.daemon-plans
kind: Endpoint
title: 계획 관리 API (데몬)
definition: "데몬이 제공하는 계획(Plan) CRUD와 승인. 목록·단건 조회, 생성, 수정, 삭제에 더해 '승인' 동작으로 계획을 active 상태로 올린다. 승인은 프로젝트당 단 하나의 active 계획만 허용한다."
realizedBy:
  - component.daemon
implementedIn:
  - desktop/apps/desktop/src/data/api.ts
relatesTo:
  - to: concept.plan
    type: mutates
    note: "계획 엔티티의 조회·생성·수정·삭제·승인을 수행한다."
  - to: concept.project
    type: reads
    note: "계획은 프로젝트에 속하며, 목록을 프로젝트 단위로 걸러 조회한다."
governedBy:
  - invariant.one-active-plan-per-project
  - invariant.start-needs-active-plan-cycle
impacts:
  - concept.plan
  - concept.project
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

계획은 프로젝트 안에서 "이번에 무엇을 할지"를 묶는 작업 흐름의 두 번째 층이다. 계획이 승인(active)되지 않으면 그 아래의 작업을 시작할 수 없으므로, 승인 동작이 이 API의 핵심이다.

- 목록·단건 조회: 프로젝트별로 계획들을 보거나 한 건을 자세히 본다.
- 생성: 프로젝트와 제목을 필수로, 설명·출처·출처 경로를 곁들여 초안(draft) 계획을 만든다.
- 수정: 제목·설명·상태를 바꾼다. 상태는 초안·활성·완료 세 단계.
- 승인: 초안 계획을 활성으로 올린다. 같은 프로젝트에 이미 활성 계획이 있으면 그쪽이 물러나고 이 계획이 활성이 된다(프로젝트당 활성 1개 규칙). 이미 활성인 계획에 다시 호출하면 아무 일도 일어나지 않는다.
- 완료: 활성 계획을 완료로 바꾼다. 완료는 되돌릴 수 없으며, 다시 일하려면 새 계획을 만들어야 한다. (완료에는 별도 경로가 없고 상태 수정으로 표현된다.)
- 삭제: 계획을 지운다.

## 요청 / 응답

- 목록: 프로젝트 범위(선택)를 지정 → 그 프로젝트의 계획 목록. 범위를 안 주면 전체.
- 단건: 계획을 지정 → 계획 한 건(제목·설명·상태·출처 등).
- 생성: 프로젝트와 제목(필수) + 설명·출처·출처 경로(선택) → 만들어진 계획(초안 상태로 시작).
- 수정: 바꿀 항목만(제목/설명/상태) → 갱신된 계획.
- 승인: 대상 계획 지정(본문 없음) → 활성으로 전이된 계획. 다른 활성 계획이 있었다면 함께 물러난다.
- 삭제: 대상 지정 → "지웠음 + 지운 대상 식별자".

## 권한 / 제약

- "프로젝트당 활성 계획은 하나"가 강제된다. 승인은 이 규칙을 데몬이 보장하는 지점이다.
- 활성 계획이 없으면 그 아래 사이클·태스크를 시작할 수 없다(작업 시작의 전제 조건).
- 완료는 비가역. 완료된 계획은 재활성화 불가 — 새 계획 생성이 정답.
- 승인은 멱등이다(이미 활성인 계획에 다시 호출해도 안전).
- 데몬 직접 접속 클라이언트가 호출하며 세션 토큰 인증 위에서 동작한다.

## provenance

- 클라이언트 측 계약: `desktop/apps/desktop/src/data/api.ts` (`listPlans` / `getPlan` / `createPlan` / `updatePlan` / `approvePlan` / `completePlan` / `deletePlan`).
- 제공 컴포넌트: `component.daemon`.
- 강제 규칙: `invariant.one-active-plan-per-project`(활성 1개), `invariant.start-needs-active-plan-cycle`(작업 시작의 전제).
- 대상 엔티티: `concept.plan`.

## 미확정 (OPEN)
- [ ] OPEN: 계획 삭제가 하위 단위·사이클·태스크에 미치는 연쇄 효과를 데몬 핸들러에서 확인.
- [ ] OPEN: "출처(source)"가 가질 수 있는 값(플랜 모드 산출물·수동 등)의 정의 위치 확인.
