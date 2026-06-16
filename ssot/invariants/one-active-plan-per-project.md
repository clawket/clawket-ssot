---
id: invariant.one-active-plan-per-project
kind: Invariant
title: 프로젝트당 활성 플랜은 하나, 작업 전 승인 필수
definition: "플랜은 초안 상태로 만들어진 뒤 승인(approve)을 거쳐야 활성이 되며, 활성 플랜이 있어야 그 프로젝트에서 태스크를 시작할 수 있다."
governs:
  - concept.plan
  - concept.project
  - domain.hook-enforcement
implementedIn:
  - daemon/src/repo/plans.rs
decidedBy:
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:invariant
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지. -->

## 제약

플랜은 곧바로 일할 수 있는 상태로 태어나지 않는다. 먼저 초안(draft)으로 만들어지고, 명시적인 승인(approve) 절차를 통과해야 비로소 활성(active) 상태가 된다. 그리고 활성 플랜이 존재해야만 그 프로젝트의 태스크를 진행 중으로 시작할 수 있다. 일이 끝난 플랜은 완료(completed)로 닫힌다.

승인은 단순한 상태 변경이 아니라 "이 의도를 실제로 추진하기로 결정했다"는 게이트다. 승인 시점이 기록으로 남아 계획이 언제 확정되었는지 추적된다.

## 깨지면 무슨 일이 일어나나

승인 게이트가 없으면 검토·합의가 끝나지 않은 초안 계획이 곧바로 실행에 들어갈 수 있다. 그러면 "승인된 의도가 작업의 단일 진실"이라는 원칙이 깨지고, 어떤 계획이 실제로 추진 승인을 받은 것인지 구분할 수 없게 된다. 작업이 합의 없이 진행되어 되돌리기 비용이 커진다.

## 코드에서 어떻게 강제되나

데몬 측 플랜 저장소 로직이 플랜 상태를 draft·active·completed 세 값으로만 제한하고, 그 외 값은 거부한다. 승인 동작(approve)은 상태를 active로 바꾸면서 승인 시각을 함께 기록하는 단일 경로로 구현되어, 활성 전환이 항상 승인 기록을 남기도록 강제한다(`daemon/src/repo/plans.rs`의 상태 검증과 approve 경로). 그리고 태스크 시작 시점에는 훅 게이트가 "활성 플랜이 하나라도 있는가"를 별도로 확인한다(관련 불변식 `invariant.start-needs-active-plan-cycle`).

"프로젝트당 동시에 하나의 active 플랜만"이라는 단일성 자체가 데이터베이스 제약으로 강하게 강제되는지는 현재 확인 범위에서 단정하기 어렵다 — 상태 어휘와 승인 게이트는 코드로 확인되나, 동일 프로젝트 내 활성 플랜 개수를 1개로 막는 카운트 검사는 이 추적에서 확정하지 못했다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요
- [ ] OPEN: "프로젝트당 활성 플랜 1개" 단일성을 강제하는 카운트/유니크 제약의 존재 여부 미확정 (상태 어휘·승인 게이트만 코드 확인)
