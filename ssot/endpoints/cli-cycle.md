---
id: endpoint.cli-cycle
kind: Endpoint
title: clawket cycle (create/view/list/update/delete/activate/complete/counts)
definition: 단일 유닛에 묶인 타임박스 반복(스프린트)인 사이클을 관리하며, activate 로 planning→active 시작하고, 완료된 사이클은 재시작할 수 없으며, 사이클별 태스크 상태 집계도 제공하는 CLI 명령군.
realizedBy: [component.cli, component.daemon]
implementedIn: [cli/src/main.rs]
relatesTo:
  - { to: concept.cycle, type: mutates, note: "사이클 생성·조회·목록·수정·삭제·활성화·완료" }
  - { to: concept.unit, type: reads, note: "모든 사이클은 정확히 하나의 유닛에 속하며 --unit 이 필수" }
  - { to: concept.task, type: aggregates, note: "counts 가 사이클별 태스크 상태 수(todo/in_progress/done/blocked/cancelled/total)를 집계" }
governedBy: [invariant.start-needs-active-plan-cycle, invariant.one-active-cycle-per-unit, invariant.cycle-unit-required, invariant.cycle-activate-endpoint]
impacts: [domain.work-management]
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
`clawket cycle`은 타임박스 반복(스프린트)인 사이클을 다루는 명령군이다. 사이클은 planning 상태로 시작해 활성화(activate)되면 active가 되고, 그 안의 태스크들이 끝나면 completed로 닫힌다. v3.0부터 모든 사이클은 정확히 하나의 유닛에 속한다. 하위 동작은 생성·조회·목록·수정·삭제, 활성화(planning→active), 완료(active→completed), 그리고 사이클별 태스크 상태 집계(counts)다.

## 요청 / 응답
- 생성: 제목(필수, 예: "Sprint 1")과 소속 프로젝트(필수), 소속 유닛(필수 — 모든 사이클은 한 유닛에 속해야 함), 스프린트 목표, 정렬 순서를 받는다. 갓 만든 사이클은 planning이다.
- 조회/목록: 사이클 ID로 상세를 보거나, 프로젝트·상태(planning/active/completed)로 걸러 목록을 받는다.
- 수정: 제목·목표·상태를 바꾼다.
- 삭제: 사이클을 지우며, 그 사이클에 배정돼 있던 태스크들의 배정을 푼다(태스크 자체는 백로그로 돌아감).
- 활성화: planning 사이클을 active로 올린다 — 태스크를 시작하기 위한 필수 단계다.
- 완료: active 사이클을 completed로 닫는다. 마지막 태스크가 종료 상태(done/cancelled)에 도달하면 자동으로도 완료되므로 이 명령은 조기 종료용이다.
- counts: 한 사이클의 태스크 상태별 개수(todo/in_progress/done/blocked/cancelled와 합계)를 한 번에 돌려준다.

## 권한 / 제약
- 한 유닛에는 동시에 active 사이클이 최대 하나다. 따라서 새 스프린트를 돌리려면 이전 사이클을 닫아야 한다.
- 완료된 사이클은 다시 시작할 수 없다 — 새 사이클을 만들어야 한다. 사이클은 일회성 타임박스이기 때문이다.
- 태스크는 그 태스크가 속한 사이클이 active일 때만 시작될 수 있으므로, activate가 작업 시작 게이트의 한 축(다른 축은 active 플랜)이다.
- 로컬 데몬이 떠 있어야 동작한다.

## provenance
명령 정의와 하위 동작은 CLI의 `cli/src/main.rs`(CycleAction 정의와 처리)에 있다. 생성·활성화·완료·삭제는 로컬 데몬의 사이클 HTTP API를, counts는 데몬의 사이클 카운트 집계 엔드포인트를 호출해 처리하므로 제공 주체는 CLI 바이너리와 데몬이다.

## 미확정 (OPEN)
- [ ] OPEN: 사이클 삭제 시 풀린 태스크가 백로그로 돌아가는 동작과 백로그 개념(concept.backlog)의 연결을 백로그 노드와 교차 확인 필요.
