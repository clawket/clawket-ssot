---
id: endpoint.plans-counts
kind: Endpoint
title: 데몬 GET /plans/{id}/counts
definition: "한 플랜에 속한 유닛별로 태스크를 상태(할일·진행중·완료·차단·취소)별로 세고 플랜 전체 합계까지 집계해 돌려주는 진행률 엔드포인트."
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/plans.rs]
relatesTo:
  - { to: concept.plan, type: reads, note: "플랜 단위 진행 집계 대상" }
  - { to: concept.unit, type: reads, note: "유닛별로 그룹핑해 카운트" }
  - { to: concept.task, type: reads, note: "태스크를 상태별로 카운트" }
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

플랜 하나의 진행 상황을 한눈에 보여주기 위한 집계 조회다. 플랜에 속한 각 유닛을 순서대로 나열하고, 유닛마다 그 안의 태스크를 상태별(할 일·진행 중·완료·차단·취소)로 세어 준다. 동시에 플랜 전체 차원의 합계(상태별 총합과 전체 태스크 수)도 함께 돌려준다. 웹 대시보드가 플랜 보드의 진행률 막대나 유닛별 요약을 그릴 때 이 한 번의 호출로 필요한 숫자를 모두 얻는다. 유닛은 정의된 순서(idx)대로 정렬되어, 표시 순서가 일관되게 유지된다.

## 요청 / 응답

요청에는 집계 대상 플랜의 식별자만 담는다. 응답은 플랜 식별자, 유닛 목록(각 항목은 유닛 식별자·제목과 그 유닛의 상태별 태스크 수·총합), 그리고 플랜 전체의 상태별 합계와 전체 태스크 수로 구성된다. 태스크가 한 건도 없는 빈 플랜이라도 유닛들은 모두 0으로 채운 형태로 돌려주어, 화면이 유닛 구조 자체는 그릴 수 있게 한다. 식별자에 해당하는 플랜이 없으면 "찾을 수 없음"으로 응답한다.

## 권한 / 제약

로컬 데몬에 붙은 클라이언트만 호출할 수 있는 읽기 전용 집계다. 카운트는 단일 집계 질의로 한 번에 계산해(유닛마다 따로 묻는 비효율을 피해) 응답하며, 상태를 바꾸지 않는다. 빈 플랜에 대한 0-채움 응답 형태가 유일한 특수 케이스다.

## provenance

라우터 정의와 핸들러는 `daemon/src/routes/plans.rs`의 `/plans/{id}/counts` GET 경로(`counts`)에 있으며, 유닛별 상태 합산은 단일 SQL 집계로 수행하고 빈 플랜은 유닛 저장소(`units::list`)로 보강한다. 응답 형태는 모델 계층의 PlanCounts/UnitCounts 구조에 대응한다. 제공 주체는 로컬 clawketd 데몬(component.daemon)이다.

## 미확정 (OPEN)
- (없음)
