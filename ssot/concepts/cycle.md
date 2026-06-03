---
id: concept.cycle
kind: Concept
title: Cycle
definition: "단일 유닛에 묶인 타임박스 반복(스프린트). planning → active → completed를 거치며, 유닛당 active 사이클은 하나뿐이고 완료된 사이클은 재시작할 수 없다(새로 만든다)."
relatesTo:
  - { to: concept.unit, type: belongs-to, note: "사이클은 단일 유닛에 묶인다." }
  - { to: concept.task, type: contains, note: "태스크는 시작하려면 active 사이클에 배정되어야 한다." }
  - { to: concept.backlog, type: relates-to, note: "사이클에 배정되지 않은 태스크가 백로그를 이루며, 사이클로 드래그하면 일정에 편입된다." }
  - { to: endpoint.cli-cycle, type: mutates, note: "create/activate/complete 등으로 관리된다." }
  - { to: endpoint.daemon-events, type: relates-to, note: "사이클 변경이 SSE 이벤트로 푸시된다." }
governedBy:
  - invariant.one-active-cycle-per-unit
  - invariant.cycle-unit-required
  - invariant.cycle-required-for-tasks
  - invariant.start-needs-active-plan-cycle
governs: []
dependsOn: []
implementedIn:
  - daemon/src/models.rs
  - daemon/src/repo/cycles.rs
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:concept"]
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지. -->

## 정의

Cycle(사이클)은 타임박스된 반복 단위, 즉 스프린트에 해당한다. 사람이 보기엔 "이번 주기에 실제로 굴릴 작업 묶음"이다. 사이클은 **단일 유닛에 묶이며**, 그 유닛의 태스크 중 이번 주기에 다룰 것을 담는다.

사이클은 세 상태를 가진다.

- **planning(계획)**: 사이클이 만들어졌지만 아직 시작되지 않은 상태.
- **active(활성)**: activate를 거쳐 시작된 상태. 태스크는 active 사이클에 배정되어 있어야 시작할 수 있다.
- **completed(완료)**: 주기가 끝난 상태.

핵심 규칙은 두 가지다. 첫째, **유닛당 동시에 active인 사이클은 하나뿐**이다 — 같은 유닛에 이미 active 사이클이 있는데 다른 사이클을 active로 만들려 하면 거부되거나, activate 시 기존 active 사이클을 자동으로 완료 처리해 직렬화한다. 둘째, **완료된 사이클은 재시작할 수 없다** — 다시 돌리려면 새 사이클을 만들어야 한다. 또한 사이클은 시작 시각·종료 시각을 보존해 주기의 기간을 추적한다.

## 엔티티 (DB)

사이클 한 건은 소속 프로젝트와 소속 유닛, 순서(idx), 제목·목표, 생성 시각, 시작 시각(started_at), 종료 시각(ended_at), 그리고 현재 상태(planning/active/completed)를 보존한다.

## API 표면

사이클은 `clawket cycle` 명령군으로 생성·조회·목록·수정·삭제하고, `activate`로 시작(같은 유닛의 기존 active 사이클은 자동 완료 처리), `complete`로 종료한다. 사이클 상태 변경은 데몬의 SSE 이벤트 스트림으로 푸시되어 웹 대시보드와 watch가 실시간으로 반영한다.

## 불변식

- **유닛당 active 사이클 하나**: 같은 유닛에서 동시에 active일 수 있는 사이클은 하나뿐이며, 새 사이클 activate 시 기존 active 사이클은 자동 완료된다(`invariant.one-active-cycle-per-unit`).
- **유닛 소속 필수**: 사이클은 단일 유닛에 묶여야 한다(`invariant.cycle-unit-required`).
- **태스크에 사이클 필요**: 태스크는 사이클에 배정되어야 하며, 사이클 없이는 일정에 편입되지 않는다(`invariant.cycle-required-for-tasks`).
- **시작 전제**: 태스크 시작에는 active 플랜과 active 사이클이 모두 필요하다(`invariant.start-needs-active-plan-cycle`).
- 완료된 사이클은 재시작 불가 — 새로 만든다.

## 구현 위치 (provenance)

사이클 엔티티의 형상은 `daemon/src/models.rs`, 상태 전이·activate 시 동일 유닛 직렬화·완료 후 재시작 금지 가드는 `daemon/src/repo/cycles.rs`에 있다. 사이클을 단일 유닛에 묶는 unit_id 도입 결정도 이 영역에서 추적된다.

## 미확정 (OPEN)
- [ ] OPEN: planning 사이클의 순서 제약(이미 planning 사이클이 있을 때 idx 순서 강제)의 정확한 사용자 영향 검증 필요.
