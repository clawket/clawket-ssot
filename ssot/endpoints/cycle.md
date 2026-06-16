---
id: endpoint.cycle
kind: Endpoint
title: clawket cycle (사이클 라이프사이클·태스크 카운트)
definition: "clawket cycle — 단일 유닛에 묶인 사이클(스프린트)을 만들고 보고 수정·삭제하며, activate 로 시작하고 complete 로 종료하고, 사이클별 태스크 상태 개수를 집계하는 CLI 명령 그룹."
realizedBy: [component.cli, component.daemon]
implementedIn: [cli/src/main.rs]
relatesTo:
  - { to: concept.cycle, type: mutates, note: "사이클 엔티티를 생성·수정·삭제하고 planning→active→completed 를 전이한다" }
  - { to: concept.unit, type: reads, note: "모든 사이클은 정확히 하나의 유닛에 속한다(--unit 필수)" }
  - { to: concept.task, type: reads, note: "counts 는 사이클에 속한 태스크의 상태별 개수를 집계한다" }
governedBy: [invariant.one-active-cycle, invariant.cycle-unit-required, invariant.cycle-activate-endpoint, invariant.start-needs-active-plan-cycle, invariant.terminal-auto-cascade]
impacts: [domain.work-management]
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

`clawket cycle` 은 단일 유닛에 묶인 **타임박스 반복(스프린트)** 인 사이클을 다루는 명령 그룹이다. 사이클은 `planning → active → completed` 를 거치며, **active 사이클에 배정된 태스크만 시작**할 수 있고, 한 유닛에는 active 사이클이 동시에 하나만 존재한다. 완료된 사이클은 재시작할 수 없다.

제공하는 동작:
- **만들기(create)**: 제목·소속 프로젝트·소속 유닛(필수)으로 사이클을 만든다. 목표와 정렬 순서를 줄 수 있다. `planning` 으로 시작한다.
- **보기/목록(view/list)**: 단일 사이클 상세, 또는 프로젝트·상태로 거른 목록을 본다.
- **수정(update)**: 제목·목표·상태를 바꾼다. (완료된 사이클은 되살릴 수 없으므로 새로 만든다.)
- **삭제(delete)**: 사이클을 지우고, 그에 배정됐던 태스크는 모두 미할당(백로그)으로 돌린다.
- **활성화(activate)**: planning 사이클을 active 로 올린다 — 태스크 시작의 전제 조건.
- **완료(complete)**: active 사이클을 completed 로 닫는다. 마지막 태스크가 종료되면 자동 완료되므로 이 명령은 주로 조기 종료용이다.
- **개수(counts)**: 한 사이클의 태스크를 상태별(todo/in_progress/done/blocked/cancelled/total)로 집계한다 — 데몬의 단일 SQL 집계로 서빙.

## 요청 / 응답

- **만들 때** 보내는 정보: 제목(필수), 소속 프로젝트(필수), 소속 유닛(필수), 목표, 정렬 순서.
- **목록**은 프로젝트·상태 필터로 추린 사이클들을 반환하며, 각 항목은 제목·상태·목표·소속 유닛을 담는다.
- **counts** 는 사이클 ID 하나를 받아 상태별 개수와 총합을 반환한다 — 대시보드/스크립트가 진행도를 한눈에 보거나, 후반 태스크의 주의 분산을 감지할 때 쓴다.

## 권한 / 제약

- 모든 사이클은 정확히 하나의 유닛에 속한다 — create 시 `--unit` 필수(`invariant.cycle-unit-required`).
- 한 유닛에는 active 사이클이 동시에 하나뿐 — `invariant.one-active-cycle`.
- planning→active 전이는 전용 activate 경로로 한다 — `invariant.cycle-activate-endpoint`.
- 태스크 시작에는 active 사이클이 필수 — `invariant.start-needs-active-plan-cycle`.
- 완료된 사이클은 재시작 불가 — 새 사이클을 만든다.
- 마지막 비종료 태스크가 종료되면 사이클이 자동으로 completed 로 cascade 된다 — `invariant.terminal-auto-cascade`.

## provenance

- 명령 정의·핸들러: `cli/src/main.rs`(Commands::Cycle, CycleAction). counts 는 데몬의 사이클별 카운트 집계 엔드포인트로 위임.
- 제공 컴포넌트: `component.cli` + `component.daemon`.
- 재구현 시 보존해야 할 의미: 사이클-유닛 1:1 소속, 유닛당 단일 active, activate 게이트, 완료 후 재시작 금지, delete 시 태스크 미할당화, counts 의 상태별 집계 의미.

## 미확정 (OPEN)

- [ ] OPEN: update 로 직접 status=active 를 줄 때 activate 와 같은 단일성 검증이 도는지(우회 가능 여부)는 데몬 라우트 확인 필요.
