---
id: concept.unit
kind: Concept
title: Unit
definition: "플랜 안에서 태스크를 묶는 순수 그룹핑 엔티티. 상태도 승인도 없으며, 병렬(parallel) 실행 모드의 유닛에 속한 태스크는 여러 에이전트가 동시에 실행할 수 있다."
relatesTo:
  - { to: concept.plan, type: belongs-to, note: "유닛은 한 플랜에 속한다." }
  - { to: concept.task, type: contains, note: "유닛은 태스크를 묶는 직접 컨테이너다." }
  - { to: concept.cycle, type: contains, note: "사이클은 단일 유닛에 묶이며 유닛당 active 사이클은 하나뿐이다." }
  - { to: domain.work-management, type: belongs-to, note: "작업 관리 계층의 그룹핑 단계." }
  - { to: endpoint.cli-unit, type: mutates, note: "create/view/list/update/delete로 관리된다." }
governedBy:
  - invariant.unit-no-status
  - invariant.cycle-unit-required
  - invariant.one-active-cycle-per-unit
governs: []
dependsOn: []
implementedIn:
  - daemon/src/models.rs
  - daemon/src/repo/units.rs
  - daemon/migrations/013_drop_unit_status_approval.sql
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:concept"]
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지. -->

## 정의

Unit(유닛)은 플랜 안에서 태스크를 묶는 **순수 그룹핑** 단위다. 사람이 보기엔 "이 플랜 안에서 함께 묶이는 작업 묶음(서브 영역)"에 해당한다. 핵심은 유닛에는 **상태도 승인도 없다**는 점이다 — 유닛은 진행 중/완료 같은 라이프사이클을 갖지 않고, 오로지 태스크를 분류·그룹핑하는 역할만 한다. (초기 설계에는 유닛 상태·승인이 있었으나 제거되어, 지금은 순수 그룹핑 엔티티다.)

유닛은 **실행 모드(execution_mode)** 를 가진다. 이 모드가 "병렬(parallel)"이면, 그 유닛에 속한 태스크들은 여러 LLM 에이전트가 동시에 실행할 수 있다 — 즉 유닛은 동시 실행의 단위를 표현한다. 유닛은 또한 플랜 안에서의 순서(idx)와 목표(goal)를 가진다.

사이클(Cycle)은 단일 유닛에 묶이므로, 유닛은 사이클의 상위 컨테이너이기도 하다. 한 유닛에는 동시에 active인 사이클이 하나만 존재할 수 있다.

## 엔티티 (DB)

유닛 한 건은 소속 플랜, 플랜 내 순서(idx), 제목, 목표, 실행 모드(직렬/병렬 등), 생성 시각을 보존한다. 과거에 존재했던 상태·승인 컬럼은 마이그레이션으로 제거되어 현재 유닛에는 라이프사이클 필드가 없다.

## API 표면

유닛은 `clawket unit` 명령군으로 생성·조회·목록·수정·삭제한다. 유닛 자체에는 승인이나 상태 전이 명령이 없다(상태가 없으므로).

## 불변식

- **유닛은 상태가 없다**: 유닛은 라이프사이클·승인을 갖지 않는 순수 그룹핑 엔티티다(`invariant.unit-no-status`).
- **사이클은 유닛 소속 필수**: 사이클은 단일 유닛에 묶여야 한다(`invariant.cycle-unit-required`).
- **유닛당 active 사이클 하나**: 한 유닛 안에서 동시에 active일 수 있는 사이클은 하나뿐이다(`invariant.one-active-cycle-per-unit`).

## 구현 위치 (provenance)

유닛 엔티티의 형상은 `daemon/src/models.rs`, 영속 로직은 `daemon/src/repo/units.rs`에 있다. 유닛에서 상태·승인 컬럼을 제거한 결정은 마이그레이션 `daemon/migrations/013_drop_unit_status_approval.sql`에 기록되어 있다.

## 미확정 (OPEN)
- [ ] OPEN: execution_mode가 가질 수 있는 값의 전체 집합(직렬/병렬 외)과 각 의미 검증 필요.
