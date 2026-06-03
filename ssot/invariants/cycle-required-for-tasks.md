---
id: invariant.cycle-required-for-tasks
kind: Invariant
title: 태스크는 활성 사이클 없이 시작·생성될 수 없다
definition: 태스크를 생성하려면 소속 사이클이 반드시 지정되어야 하고, 태스크를 시작하려면 그 사이클이 활성 상태여야 한다 — 둘 다 어기면 데몬이 거부한다.
governs:
  - concept.task
  - concept.cycle
  - domain.work-management
implementedIn:
  - daemon/src/routes/tasks.rs
  - daemon/src/repo/tasks.rs
  - cli/src/main.rs
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

<!-- 작성 고도: 자연어 우선. 식별자는 provenance/근거에만. -->

## 제약

태스크는 직접 다루는 유일한 원자적 작업 단위지만, 일정 체계 밖에서 떠다닐 수 없다. 두 가지 시점에서 사이클이 강제된다. 첫째, 태스크를 만들 때 어느 사이클에 속하는지 반드시 지정해야 한다(사이클 식별자 누락 시 생성 거부). 둘째, 태스크를 실제로 시작(진행 중으로 전환)하려면 그 사이클이 활성 상태여야 한다. 사이클이 배정되지 않았거나 사이클이 아직 활성이 아니면 시작이 막힌다.

과거에는 활성 사이클을 자동 추론해 채워주던 동작이 있었으나 현재는 제거되어, 호출자가 명시적으로 사이클을 지정해야 한다.

## 깨지면 무슨 일이 일어나나

사이클에 묶이지 않은 태스크가 시작될 수 있다면, 타임박스·진행 추적·완료 cascade(사이클 내 모든 태스크가 종결되면 사이클·플랜이 자동 완료되는 흐름)의 기준점이 사라진다. 어떤 태스크가 어느 스프린트에 속하는지 알 수 없어 백로그와 진행 중 작업의 경계가 무너지고, 보드·타임라인의 일정 표현이 불가능해진다. 또 비활성 사이클의 태스크가 시작되면 아직 시작하지 않기로 한 반복의 작업이 새어 나간다.

## 코드에서 어떻게 강제되나

태스크 생성 경로에서 사이클 식별자가 비어 있으면 거부한다(`daemon/src/routes/tasks.rs`, MISSING_CYCLE_ID — 사이클을 활성화하고 명시적으로 지정하라는 안내 포함). 태스크 시작(진행 중 전환) 경로에서는 태스크에 사이클이 붙어 있지 않으면 MISSING_CYCLE_ID로, 붙어 있더라도 그 사이클이 활성이 아니면 잘못된 전환으로 거부한다(`daemon/src/repo/tasks.rs`). CLI 명령 정의에도 사이클 식별자가 필수임이 명시되어 있다(`cli/src/main.rs`).

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요 — 사이클 자동 추론 제거(API-TASK-001)와 사이클 필수화 결정을 decision 노드로 연결.
