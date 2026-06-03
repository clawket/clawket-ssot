---
id: endpoint.metrics
kind: Endpoint
title: 데몬 GET /metrics (Prometheus)
definition: 데몬의 기본 운영 지표를 Prometheus 텍스트 포맷으로 노출한다(옵트인으로만 켜짐).
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/metrics.rs]
relatesTo:
  - { to: concept.task, type: reads, note: "상태별 태스크 개수를 집계해 지표로 노출한다" }
  - { to: concept.plan, type: reads, note: "상태별 플랜 개수를 집계해 지표로 노출한다" }
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
데몬의 건강·작업 현황을 외부 모니터링 도구(Prometheus 등)가 긁어 갈 수 있도록 표준 텍스트 형식으로 내보내는 관측용 엔드포인트다. 운영자가 대시보드/알림을 붙이고 싶을 때 사용한다. 기본적으로는 꺼져 있고, 데몬을 옵트인 옵션으로 띄웠을 때만 활성화된다.

## 요청 / 응답
- 요청: 별도 입력 없이 호출한다.
- 응답: Prometheus 노출 형식의 평문(지표 이름·설명·타입·값)이다. 담기는 지표는 다음과 같다.
  - 현재 DB 스키마 버전
  - 데몬이 시작된 뒤 경과 시간
  - 벡터 검색(sqlite-vec) 활성 여부(켜짐 1 / 꺼짐 0)
  - 상태별 태스크 개수
  - 상태별 플랜 개수

## 권한 / 제약
- 옵트인 기능이다. 데몬을 메트릭 활성 옵션(플래그 또는 환경 변수)으로 기동했을 때만 이 엔드포인트가 붙는다. 기본 동작에서는 노출되지 않는다.
- 읽기 전용 관측 엔드포인트로 상태를 바꾸지 않는다. 로컬 데몬에 붙은 수집기가 호출한다.
- 노출 내용은 운영 지표(개수·버전·플래그)에 한정되며 태스크 본문 같은 내용 데이터는 담지 않는다.

## provenance
- 라우터/핸들러: `daemon/src/routes/metrics.rs`
- 제공 컴포넌트: 로컬 데몬(`component.daemon`)

## 미확정 (OPEN)
- [ ] OPEN: 향후 추가될 지표(에스컬레이션율·임베딩 큐 등)의 범위와 라벨 규약을 owner가 관측 정책으로 확정할 필요가 있는지 확인.
