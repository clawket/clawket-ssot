---
id: endpoint.cycles-diff
kind: Endpoint
title: 사이클 시나리오 diff (GET /cycles/{id}/diff)
definition: 한 사이클의 시나리오 판정 결과를 다른 사이클과 대조해 무엇이 바뀌었는지 돌려주는 조회 통로. 두 사이클 사이에서 합격으로 뒤집힌 것·결함으로 뒤집힌 것·그대로 유지된 것·새로 생긴 것·사라진 것을 분류해 검증 라운드 간 진척을 보여준다.
realizedBy:
  - component.daemon
implementedIn:
  - daemon/src/routes/cycles.rs
relatesTo:
  - to: concept.cycle
    type: reads
    note: 두 사이클(현재·비교 대상)을 시나리오 판정 단위로 대조한다
  - to: concept.scenario
    type: reads
    note: 사이클에 속한 태스크의 시나리오 판정 상태를 비교한다
  - to: concept.task
    type: reads
    note: 사이클의 라운드 태스크 집합으로부터 시나리오 상태를 끌어온다
governedBy:
  - invariant.scenario-id-one-to-one
impacts:
  - domain.pdd-verification
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
검증 라운드를 거듭하며 "이번 사이클에서 무엇이 좋아지고 무엇이 나빠졌나"를 두 사이클을 나란히 놓고 비교해 보여주는 통로다. PDD 검증에서 사이클은 한 검증 라운드의 단위가 되고 각 태스크는 시나리오에 1:1로 매핑되므로, 두 사이클의 시나리오 판정을 맞대면 라운드 간 변화를 가려낼 수 있다. 결함이던 시나리오가 합격으로 돌아섰는지, 멀쩡하던 것이 결함으로 무너졌는지, 새 시나리오가 들어왔는지를 한 번에 본다.

## 요청 / 응답
보내는 정보는 현재 사이클의 식별자와, 선택적으로 비교 대상 사이클의 식별자다. 비교 대상을 주지 않으면 같은 유닛 안에서 이 사이클 바로 앞에 만들어진 사이클을 자동으로 골라 비교한다 — 호출자가 직전 사이클을 미리 찾아둘 필요가 없다. 돌려받는 것은 두 사이클의 식별자와 함께, 시나리오들을 변화 종류로 분류한 묶음이다 — 합격으로 뒤집힌 것, 결함으로 뒤집힌 것, 합격을 유지한 것, 결함을 유지한 것, 새로 생긴 것, 사라진 것. 현재 사이클이 존재하지 않으면 "찾을 수 없음"으로 응답한다.

## 권한 / 제약
데몬이 로컬에만 바인딩되므로 같은 기기 안에서만 조회된다. 비교 대상 자동 선택은 "같은 유닛에 속하고 현재보다 먼저 만들어진 사이클 중 가장 최근 것"이라는 규칙으로 결정된다. 각 사이클의 시나리오 상태는 그 사이클에 속한 라운드 태스크에서 끌어오며, 태스크와 시나리오가 1:1로 대응한다는 규칙(invariant.scenario-id-one-to-one)이 비교의 기준점을 보장한다. 이 통로는 플랜 라운드 diff(`/plans/{id}/rounds/{n}/diff`)와 같은 결과 형태를 사이클 범위로 제공하는 형제 통로다. 읽기 전용 조회이므로 상태를 바꾸지 않는다.

## provenance
근거는 `daemon/src/routes/cycles.rs`의 `/cycles/{id}/diff` GET 라우트(`cycle_diff` 핸들러, US-CLAWKET-PDD-230)다. `against` 비교 대상이 비어 있으면 같은 유닛에서 생성 시각이 더 이른 직전 사이클을 자동 선택하고, 두 사이클의 라운드 태스크 집합을 플랜 쪽 라운드 diff 계산 로직으로 대조해 합격 전환·결함 전환·유지·신규·삭제로 분류한다. 제공 주체는 로컬 데몬(component.daemon), 소비 주체는 라운드 진척을 보여주는 웹 대시보드(component.web-dashboard)다.

## 미확정 (OPEN)
- [ ] OPEN: 시나리오의 "합격/결함" 판정이 태스크의 어떤 필드·상태에서 도출되는지(예: 완료/취소 vs 별도 판정 필드)는 플랜 라운드 태스크·diff 계산 코드를 정독해 확인 필요.
