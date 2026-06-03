---
id: endpoint.plans-round-diff
kind: Endpoint
title: 데몬 GET /plans/{id}/rounds/{n}/diff
definition: "검증 플랜의 N번째 라운드와 직전 라운드(N-1) 사이에서 시나리오별 QA 판정이 어떻게 바뀌었는지(결함→통과·통과→결함·신규·제거 등) 태스크 단위 델타를 계산해 돌려주는 라운드 비교 엔드포인트."
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/plans.rs]
relatesTo:
  - { to: concept.plan, type: reads, note: "검증 플랜의 라운드(=사이클) 비교 대상" }
  - { to: concept.cycle, type: reads, note: "라운드 번호를 생성 순 사이클로 해석" }
  - { to: concept.task, type: reads, note: "scenario_id·QA 판정을 가진 태스크를 비교" }
  - { to: concept.scenario, type: reads, note: "시나리오 ID 기준으로 라운드 간 매칭" }
  - { to: concept.convergence, type: backed-by, note: "결함/통과 전이 집계가 수렴 판정의 입력" }
governedBy: [invariant.scenario-id-one-to-one]
impacts: []
consumedBy: [endpoint.cli-discover-loop]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

PDD 검증 루프에서 한 라운드를 끝낸 뒤 "이번 라운드가 직전보다 나아졌나, 나빠졌나"를 시나리오 단위로 보여주는 비교 조회다. 한 검증 플랜의 각 라운드는 사이클 하나에 대응하고, 라운드 번호 N은 그 플랜의 사이클을 생성 순으로 나열했을 때 N번째를 가리킨다(첫 라운드가 가장 이른 사이클). 이 엔드포인트는 N번째 라운드와 그 직전(N-1) 라운드의 태스크들을 시나리오 ID로 짝지어, 같은 시나리오의 QA 판정이 어떻게 변했는지 분류한다. 검증 루프 자동화(discover-loop)와 QA 흐름이 라운드 종료 시 회귀/개선 현황을 파악하는 데 쓴다.

## 요청 / 응답

요청에는 플랜 식별자와 라운드 번호 N을 담는다. 응답은 비교한 두 라운드 번호(N-1→N)와 함께, 시나리오들을 다음 갈래로 분류해 돌려준다 — 직전 라운드에서 결함이었다가 이번에 통과로 바뀐 것(개선), 통과였다가 결함으로 바뀐 것(회귀), 두 라운드 모두 통과인 것, 두 라운드 모두 결함인 것, 이번 라운드에 새로 등장한 시나리오, 직전에는 있었으나 이번에 사라진 시나리오. 각 항목은 해당 시나리오 ID와 두 라운드의 태스크 식별자·판정 상태를 가진다. 시나리오 ID가 비어 있는 태스크는 비교에서 제외된다.

## 권한 / 제약

라운드 번호는 2 이상이어야 한다 — 비교할 직전 라운드가 있어야 하기 때문이며, 1 이하를 주면 "직전 라운드와 비교하려면 2 이상이어야 한다"는 잘못된 요청으로 거부된다. 플랜이 N개보다 적은 라운드만 가지면 모자란 쪽은 빈 집합으로 취급해(신규/제거로 분류), 비교 자체는 실패하지 않는다. 시나리오 ID를 가진 태스크만 대상이며, 시나리오와 태스크가 1:1로 대응한다는 전제 위에서 동작한다. 결함/통과 외의 전이(예: 시나리오 오류)는 조용히 버리지 않고 현재 상태 기준으로 통과/결함 갈래에 편입해 노출한다. 로컬 데몬에 붙은 클라이언트만 호출할 수 있는 읽기 전용 조회다.

## provenance

라우터 정의와 핸들러는 `daemon/src/routes/plans.rs`의 `/plans/{id}/rounds/{n}/diff` GET 경로(`round_diff`)에 있으며, 라운드→사이클 해석은 `cycle_for_round`, 시나리오별 태스크 수집은 `round_tasks_for_cycle`, 분류 로직은 `compute_round_diff`가 담당한다. 같은 파일에는 통과→결함만 추려 주는 자매 경로(`round_regression_intent`)도 있다. 제공 주체는 로컬 clawketd 데몬(component.daemon)이다.

## 미확정 (OPEN)
- (없음)
