---
id: endpoint.cycles-activate
kind: Endpoint
title: 사이클 활성화 (POST /cycles/{id}/activate)
definition: 계획 중(planning)인 사이클을 진행 중(active)으로 전환하는 전용 통로. 시작 시각이 이때 기록되며, 일반 수정으로는 계획→진행 전환이 막혀 있어 반드시 이 통로를 거친다.
realizedBy:
  - component.daemon
implementedIn:
  - daemon/src/routes/cycles.rs
relatesTo:
  - to: concept.cycle
    type: mutates
    note: 사이클 상태를 진행 중으로 바꾸고 시작 시각을 기록한다
governedBy:
  - invariant.cycle-activate-endpoint
  - invariant.one-active-cycle-per-unit
  - invariant.start-needs-active-plan-cycle
impacts:
  - domain.work-management
consumedBy:
  - component.web-dashboard
  - endpoint.cli-cycle
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의
계획만 세워둔 사이클을 실제로 "지금부터 시작" 상태로 올리는 통로다. 사이클은 계획 중→진행 중→완료의 흐름을 가지는데, 진행 중으로 올리는 일은 이 전용 통로로만 할 수 있다. 이렇게 분리한 이유는 시작 시각을 정확히 남기기 위해서다 — 일반 수정으로 상태만 진행 중으로 바꾸는 것은 막혀 있고, 그 경우 이 활성화 통로를 쓰라고 안내한다. 진행 중 사이클에 배정된 태스크라야 작업을 시작할 수 있으므로, 이 활성화는 그 유닛의 작업을 여는 출발점이다.

## 요청 / 응답
보내는 정보는 대상 사이클의 식별자뿐이다(본문 없음). 돌려받는 것은 진행 중으로 바뀐 사이클의 최신 상태다 — 갱신된 상태와 시작 시각을 포함한다. 대상 사이클이 없으면 "찾을 수 없음"으로 응답한다. 전환이 실제로 일어나면 데몬은 사이클이 갱신되었다는 실시간 이벤트를 흘려보내, 웹 대시보드나 관찰 중인 사용자가 즉시 반영을 본다.

## 권한 / 제약
데몬이 로컬에만 바인딩되므로 같은 기기 안에서만 호출된다. 한 유닛 안에서는 동시에 하나의 사이클만 진행 중일 수 있다는 규칙(invariant.one-active-cycle-per-unit)이 적용되며, 계획→진행 전환은 반드시 이 통로를 거쳐야 한다는 규칙(invariant.cycle-activate-endpoint)이 일반 수정 경로의 우회를 막는다. 진행 중 사이클에 속해야 태스크를 시작할 수 있다는 규칙(invariant.start-needs-active-plan-cycle)과 맞물려, 이 활성화가 작업 시작 자격의 한 전제를 충족시킨다.

## provenance
근거는 `daemon/src/routes/cycles.rs`의 `/cycles/{id}/activate` POST 라우트(`activate` 핸들러)다. 전환 성공 시 `cycle:updated` 이벤트를 emit하고, 대상이 없으면 404로 응답한다. 같은 파일의 일반 수정(`update`) 핸들러는 계획 중 사이클을 진행 중으로 바꾸려는 시도를 거부하고 이 활성화 통로로 안내한다. 제공 주체는 로컬 데몬(component.daemon), 소비 주체는 웹 대시보드(component.web-dashboard)와 `clawket cycle activate`(endpoint.cli-cycle)다.

## 미확정 (OPEN)
- [ ] OPEN: 활성화 시점에 같은 유닛의 기존 진행 중 사이클을 어떻게 처리하는지(자동 종료 vs 거부)는 사이클 저장소의 activate 구현을 정독해 확인 필요.
