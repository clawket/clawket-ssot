---
id: endpoint.cycles-counts
kind: Endpoint
title: 사이클 태스크 상태 카운트 (GET /cycles/{id}/counts)
definition: 한 사이클에 속한 태스크를 상태별로 집계해 돌려주는 조회 통로. 할 일·진행 중·완료·차단·취소 각각의 개수와 전체 개수를 한 번에 받아 사이클 진행도를 화면에 그린다.
realizedBy:
  - component.daemon
implementedIn:
  - daemon/src/routes/cycles.rs
relatesTo:
  - to: concept.cycle
    type: reads
    note: 대상 사이클을 식별하고 제목을 함께 반환한다
  - to: concept.task
    type: reads
    note: 사이클에 속한 태스크를 상태별로 집계한다
governedBy: []
impacts:
  - domain.work-management
consumedBy:
  - component.web-dashboard
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의
"이 사이클이 얼마나 진행됐나"를 한눈에 보여주는 숫자를 돌려주는 통로다. 사이클에 묶인 태스크를 상태별로 세어 — 할 일, 진행 중, 완료, 차단, 취소 — 각각의 개수와 총 개수를 합산해서 준다. 태스크 목록을 모두 받아와 화면에서 일일이 세지 않고, 집계만 가볍게 받아 진행도 막대나 요약 배지를 그리는 데 쓴다.

## 요청 / 응답
보내는 정보는 대상 사이클의 식별자뿐이다. 돌려받는 것은 그 사이클을 가리키는 식별 정보(식별자와 제목)와 함께, 상태별 태스크 개수 다섯 가지(할 일·진행 중·완료·차단·취소)와 전체 개수다. 대상 사이클이 존재하지 않으면 "찾을 수 없음"으로 응답한다.

## 권한 / 제약
데몬이 로컬에만 바인딩되므로 같은 기기 안에서만 조회된다. 집계는 저장소에서 한 번의 합산 질의로 계산되어, 태스크가 많아도 목록 전체를 옮기지 않고 숫자만 가볍게 돌려주도록 설계됐다. 읽기 전용 조회이므로 상태를 바꾸지 않는다.

## provenance
근거는 `daemon/src/routes/cycles.rs`의 `/cycles/{id}/counts` GET 라우트(`counts` 핸들러)다. 먼저 사이클 존재를 확인한 뒤, 해당 사이클의 태스크를 상태별 합산 질의로 한 번에 집계해 사이클 식별자·제목과 다섯 상태 개수·총계를 함께 반환한다. 대상이 없으면 404로 응답한다. 제공 주체는 로컬 데몬(component.daemon), 소비 주체는 진행도 표시가 필요한 웹 대시보드(component.web-dashboard)다.

## 미확정 (OPEN)
- [ ] OPEN: 이 집계가 사이클 직속 태스크만 세는지, 하위 분해 태스크까지 포함하는지는 태스크의 사이클 귀속 규칙과 집계 질의 범위를 함께 확인 필요.
