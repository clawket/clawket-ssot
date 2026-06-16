---
id: integration.prometheus-metrics
kind: Integration
title: Prometheus 메트릭 노출(/metrics)
definition: 데몬을 메트릭 옵션과 함께 켜면 운영 지표를 Prometheus 표준 텍스트 포맷으로 /metrics 경로에 노출하는 연동이다. 기본은 꺼져 있고 플래그 또는 환경변수로 켠다.
purpose: 외부 모니터링 도구가 표준 방식으로 데몬의 상태(스키마 버전·가동 시간·태스크/플랜 수·벡터 검색 활성 여부)를 긁어갈 수 있게 하기 위함 — 로컬 데몬을 관측 가능하게 만든다.
integratesWith:
  - component.daemon
  - component.clawketd
implementedIn:
  - daemon/src/routes/metrics.rs
  - daemon/src/config.rs
impacts:
  - endpoint.metrics
relatesTo:
  - to: endpoint.metrics
    type: realizes
    note: 이 연동이 노출하는 HTTP 표면이 /metrics 엔드포인트다
  - to: component.daemon
    type: belongs-to
    note: 데몬이 옵션으로 켤 때만 라우터에 /metrics가 붙는다
governedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:integration
---

## 무엇과 연동하나

연동 상대는 Prometheus 계열 모니터링 도구다. 이들은 정해진 주소를 주기적으로 긁어(scrape) 텍스트로 받은 지표를 수집하는데, 이 연동은 그 표준 텍스트 포맷(exposition format)으로 데몬의 운영 지표를 `/metrics` 경로에 내놓는다. 응답은 Prometheus가 기대하는 콘텐츠 타입(text/plain, 버전 0.0.4)으로 내려가고 마지막에 줄바꿈을 붙인다.

이 표면은 기본적으로 꺼져 있다. 데몬을 메트릭 옵션과 함께 시작하거나 메트릭 환경변수를 켰을 때만 `/metrics` 라우트가 활성화된다. 즉 관측이 필요한 운영 환경에서만 의도적으로 노출하고, 평소에는 표면을 닫아 둔다.

노출하는 지표는 사람이 운영 상태를 한눈에 볼 만한 기본 수치들이다 — 현재 DB 스키마 버전, 데몬이 켜진 뒤 흐른 시간, 의미 검색(벡터)이 활성인지 여부, 그리고 태스크와 플랜의 개수를 상태별로 쪼갠 값이다. 인증은 두지 않는다(로컬 데몬 대상).

## 구현 위치 (provenance)

메트릭 라우터와 지표 산출 로직은 `daemon/src/routes/metrics.rs`에 있고, 메트릭을 켜는 옵션(플래그/환경변수)은 `daemon/src/config.rs`에서 다룬다. 데몬(component.daemon, component.clawketd)이 시작 시 그 옵션이 켜져 있을 때만 라우터에 `/metrics`를 붙인다. 본문만으로 재현하려면 이렇게 기억하면 된다 — "옵션이 켜졌을 때만 /metrics를 열고, 스키마 버전·가동 시간·벡터 활성·태스크/플랜 상태별 개수를 Prometheus 텍스트 포맷으로 내놓는다." 이 HTTP 표면 자체가 SSOT의 메트릭 엔드포인트(endpoint.metrics)다.

## 불변식

이 연동 자체에 걸린 별도의 깨면 안 되는 제약은 현재 SSOT에 명시된 것이 없다. 다만 설계상 "기본 꺼짐, 옵션으로만 켜짐"이 일관되게 지켜지며, 이는 불필요한 표면 노출을 줄이는 안전 기본값이다.

## 영향 범위

이 연동이 닿는 것은 메트릭 엔드포인트(endpoint.metrics)다. 노출하는 지표 항목·이름·포맷이 바뀌면 외부 모니터링 대시보드·알림 규칙이 함께 영향을 받으므로, 지표 이름은 안정적으로 유지하는 것이 바람직하다.

## 미확정 (OPEN)
- [ ] OPEN: /metrics를 켰을 때 바인딩 주소가 로컬 전용으로 제한되는지(외부에서 긁을 수 있게 되는지)와 그 보안 함의 확인 필요
