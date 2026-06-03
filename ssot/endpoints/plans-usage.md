---
id: endpoint.plans-usage
kind: Endpoint
title: 데몬 GET /plans/{id}/usage
definition: "한 플랜에 속한 모든 태스크 실행의 토큰 사용량·비용을 모델별로 묶어 집계하고 플랜 전체 합계까지 돌려주는 사용량 조회 엔드포인트."
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/usage.rs]
relatesTo:
  - { to: concept.plan, type: reads, note: "플랜 단위로 사용량을 집계하는 대상" }
  - { to: concept.run, type: reads, note: "실행(run)별로 기록된 사용량을 합산" }
  - { to: concept.execution-envelope, type: backed-by, note: "예산 기준은 봉투의 token_budget 에서 비롯" }
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

한 플랜에서 그동안 얼마나 많은 토큰과 비용이 쓰였는지를 모델 종류별로 묶어 보여주는 집계 조회다. 개별 태스크 실행마다 기록되는 사용량(입력·출력 토큰, 캐시 읽기/쓰기 토큰, 비용)을 플랜 전체 범위에서 모아, 사용한 모델별로 소계를 내고 플랜 전체 합계까지 함께 돌려준다. 비용·토큰 소비를 최소화하려는 운영 관점에서 "이 로드맵 전체에 얼마가 들었나"를 확인하는 데 쓰이며, 웹 대시보드의 사용량/비용 요약 화면이 소비한다.

## 요청 / 응답

요청에는 집계 대상 플랜의 식별자만 담는다. 응답은 모델별 집계 목록과 플랜 전체 합계 두 부분으로 구성된다. 모델별 집계의 각 항목은 어떤 모델인지와 그 모델로 쓴 입력·출력 토큰, 캐시 토큰, 비용, 실행 횟수를 가진다. 전체 합계는 그 항목들을 모두 더한 값(총 입력/출력 토큰, 캐시 토큰, 총비용, 총 실행 수)이다. 식별자에 해당하는 플랜이 없으면 "찾을 수 없음"으로 응답한다.

## 권한 / 제약

로컬 데몬에 붙은 클라이언트만 호출할 수 있는 읽기 전용 집계다. 이 플랜 단위 조회 자체는 예산 초과 여부를 판정하거나 차단하지 않는다 — 예산 게이트(token_budget 기준 초과 시 차단)는 태스크 단위 사용량/프리플라이트 경로에서 다루며, 여기서는 단순히 누적 소비량만 합산해 보여준다. 사용량 기록은 실행 시점에 적용되던 예산 스냅샷을 보존하므로, 나중에 예산을 바꿔도 과거 기록이 다시 쓰이지 않는다.

## provenance

라우터 정의와 핸들러는 `daemon/src/routes/usage.rs`의 `/plans/{id}/usage` GET 경로(`list_plan_usage`)에 있으며, 모델별 집계는 사용량 저장소(`usage::aggregate_by_plan`)를 거쳐 합계를 합산한다. 같은 파일에 태스크 단위 사용량 기록/조회와 예산 프리플라이트(초과 시 429) 경로가 함께 정의되어 있다. 제공 주체는 로컬 clawketd 데몬(component.daemon)이다.

## 미확정 (OPEN)
- (없음)
