---
id: endpoint.units-counts
kind: Endpoint
title: 유닛별 태스크 상태 카운트 (GET /units/{id}/counts)
definition: 한 유닛에 속한 태스크들을 상태별로 집계한 개수를 돌려주는 데몬 조회 엔드포인트.
realizedBy: [component.daemon]
implementedIn:
  - daemon/src/routes/units.rs
relatesTo:
  - { to: concept.unit, type: reads, note: "대상 유닛을 식별·조회한다" }
  - { to: concept.task, type: aggregates, note: "유닛에 속한 태스크를 상태별로 집계한다" }
governedBy: []
impacts: []
consumedBy: [component.web-dashboard]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

한 유닛 안에 들어 있는 태스크들이 각 진행 상태별로 몇 개씩인지를 한눈에 보여 주는 집계 조회 입구다. 대시보드에서 유닛의 진척을 막대나 숫자로 표시하거나, 유닛이 얼마나 끝나가는지 파악할 때 쓴다. 태스크를 일일이 받아 세지 않고 데이터베이스 차원에서 합산해 돌려주므로 가볍다.

## 요청 / 응답

보내는 정보는 집계할 유닛의 식별자 하나다. 돌려주는 정보는 그 유닛이 어떤 유닛인지(식별자·제목)와 함께, 그 안 태스크들을 상태별로 합산한 개수 — 할 일(todo), 진행 중(in_progress), 완료(done), 차단됨(blocked), 취소됨(cancelled), 그리고 전체 합계 — 다. 해당 식별자의 유닛이 없으면 "찾을 수 없음"으로 응답한다.

## 권한 / 제약

로컬 데몬 경계 안에서 조회용으로 동작하며 별도 외부 인증은 없다. 읽기 전용이라 데이터를 변경하지 않는다. 상태 구분(todo/in_progress/done/blocked/cancelled)은 태스크 라이프사이클의 상태 어휘를 그대로 따른다. 존재하지 않는 식별자는 "찾을 수 없음"으로 처리된다.

## provenance

데몬의 유닛 라우터(`daemon/src/routes/units.rs`)가 `/units/{id}/counts` 경로의 조회 동작으로 제공하며, 상태별 합산은 데이터베이스 집계 질의로 한 번에 계산한다. 제공 주체는 clawketd 데몬이고, 주 호출 주체는 웹 대시보드다.

## 미확정 (OPEN)
- [ ] OPEN: 대시보드 외에 CLI나 다른 표면이 이 카운트를 소비하는지 확인해 consumedBy를 보강.
