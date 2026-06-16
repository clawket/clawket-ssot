---
id: invariant.one-active-cycle-per-unit
kind: Invariant
title: 사이클은 정확히 한 유닛에 속하고 유닛당 활성 사이클은 최대 하나
definition: "모든 사이클은 정확히 하나의 유닛에 묶이며, 한 유닛은 동시에 활성 사이클을 하나만 가질 수 있고, 완료된 사이클은 다시 시작할 수 없다."
governs:
  - concept.cycle
  - concept.unit
decidedBy:
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
implementedIn:
  - daemon/src/repo/cycles.rs
tags:
  - status:active
  - type:invariant
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지. -->

## 제약

사이클(타임박스 반복, 스프린트에 해당)은 항상 정확히 하나의 유닛에 속한다. 유닛 없이 떠 있는 사이클은 만들 수 없다. 또한 한 유닛 안에서는 같은 시점에 활성(active) 사이클이 최대 하나만 존재할 수 있다 — 같은 유닛의 사이클들은 동시에 굴리지 않고 순차로 진행한다. 그리고 한 번 완료(completed)된 사이클은 다시 active로 되돌려 재시작할 수 없으며, 이어서 일하려면 새 사이클을 만들어야 한다.

## 깨지면 무슨 일이 일어나나

한 유닛에 활성 사이클이 둘 이상이면 같은 작업 그룹 안에서 어느 스프린트가 "지금 진행 중"인지 불명확해지고, 태스크가 어느 사이클로 흘러가야 하는지 모호해진다. 완료된 사이클을 되살릴 수 있게 두면 이미 닫힌 스프린트의 일정·범위가 사후에 흔들려 과거 기록이 진실성을 잃는다. 그래서 재시작을 막고 새 사이클 생성을 유도한다 — 닫힌 것은 닫힌 채로 보존된다.

## 코드에서 어떻게 강제되나

데몬 측 사이클 저장소 로직이 세 가지를 모두 강제한다. 첫째, 사이클 생성 시 유닛 식별자를 필수로 요구하고 비어 있으면 거부한다(PDD A4: Cycle ⊂ Unit). 둘째, 사이클을 active로 바꾸려 할 때 같은 유닛에 이미 다른 활성 사이클이 있으면 거부하며, 기존 활성 사이클을 먼저 완료하라고 안내한다(같은 유닛 사이클의 직렬화). 셋째, 이미 completed 상태인 사이클을 다른 상태로 되돌리려는 변경을 거부하고 새 사이클을 만들라고 안내한다(`daemon/src/repo/cycles.rs`의 create·update 검증). 또한 사이클을 완료로 닫을 때는 처리되지 않은 todo 잔여 태스크가 없는지 확인한다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요 — PDD A4/A8 축리(Cycle⊂Unit, 동일 유닛 직렬화)를 정한 결정 노드 미확인
