---
id: endpoint.tasks-stats
kind: Endpoint
title: 데몬 GET /tasks/stats
definition: 검증 배치 단위로 태스크의 검증 결과를 집계하는 데몬 HTTP 엔드포인트.
realizedBy: [component.daemon]
implementedIn:
  - daemon/src/routes/tasks.rs
relatesTo:
  - { to: concept.batch, type: reads, note: "한 검증 배치에 속한 태스크들의 결과를 합산한다" }
  - { to: concept.task, type: reads, note: "태스크의 검증 결과(통과/결함/시나리오오류)를 집계 대상으로 삼는다" }
governedBy: []
impacts: []
consumedBy: [endpoint.cli-discover-loop, endpoint.cli-task]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

하나의 검증 배치에 속한 태스크들의 검증 결과를 한눈에 합산해 주는 집계 기능이다. PDD 검증 루프에서 한 라운드(배치)가 끝났을 때 "이번 배치에서 몇 건이 통과·결함·시나리오오류였는지"를 즉시 확인하는 데 쓰이며, 3-way 수렴 판정의 입력이 된다.

## 요청 / 응답

요청에는 집계 대상 검증 배치를 가리키는 배치 식별자가 필수로 들어간다. 응답은 그 배치의 총 태스크 수와 함께, 검증 결과별 건수 — 통과(pass)·결함(defect)·시나리오오류(scenario_error) — 를 합산한 수치를 돌려준다.

## 권한 / 제약

로컬 데몬에 한정된 읽기 전용 집계다. 배치 식별자는 정해진 형식(Crockford base32 ULID, 26자)을 만족해야 하며, 형식이 어긋나면 DB 조회 전에 잘못된 요청으로 거절된다 — 생성·수정 경로와 동일한 검증을 통과해야 한다.

## provenance

`daemon/src/routes/tasks.rs`의 `/tasks/stats` 라우트가 배치 식별자 형식을 먼저 검증한 뒤 태스크 저장소의 배치별 집계 질의를 호출한다. 제공 주체는 clawketd 데몬(component.daemon)이며, 검증 루프 자동화 명령(discover-loop)과 CLI task 명령이 소비한다.

## 미확정 (OPEN)
- [ ] OPEN: 취소된 태스크의 집계 포함 여부 등 합산 범위 정책을 저장소 집계 질의에서 확인해 보강할 것.
