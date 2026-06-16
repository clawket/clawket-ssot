---
id: endpoint.discover-dispatch-plan
kind: Endpoint
title: 디스패치 매니페스트 조회 (배치 크기·배치 ID 메타데이터)
definition: GET /discover-loop/dispatch-plan — 플랜의 각 유닛에 붙은 시나리오 수를 세어, 배치 크기에 맞춰 몇 개 배치로 나눌지·각 배치 크기·배치 ID 목록을 담은 디스패치 계획을 돌려준다.
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/discover.rs]
relatesTo:
  - { to: concept.plan, type: reads, note: "플랜과 그 유닛들을 읽는다" }
  - { to: concept.knowledge, type: reads, note: "각 유닛에 첨부된 시나리오 지식 항목에서 시나리오 수를 센다" }
  - { to: concept.scenario, type: reads, note: "시나리오 ID 패턴이 들어간 줄 수로 시나리오 개수를 추산한다" }
  - { to: domain.pdd-verification, type: realizes, note: "서브에이전트 배치 디스패치 계획 산출 단계를 구현한다" }
governedBy: []
impacts: [concept.scenario]
consumedBy: [endpoint.cli-discover-loop]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

한 라운드의 시나리오들을 QA 추론 서브에이전트들에게 어떻게 나눠 맡길지 계획표를 뽑아 주는 조회용 명령이다. 플랜을 가리키고 배치 크기(서브에이전트 한 명이 한 번에 추론할 시나리오 최대 수)를 주면, 플랜의 각 유닛마다 거기 붙은 시나리오가 몇 개인지 세고, 그 수를 배치 크기로 나눠 몇 개의 배치가 필요한지, 각 배치에 시나리오를 몇 개씩 담을지, 그리고 각 배치에 미리 발급한 배치 식별자들을 묶어 돌려준다. 검증 루프 에이전트는 이 계획표를 받아 그대로 서브에이전트를 스폰하고, 발급된 배치 ID를 산출 근거(TSV)의 배치 열에 박는다. 데이터를 바꾸지 않는 읽기 전용 메타데이터 조회다.

## 요청 / 응답

보내는 정보:
- 대상 플랜
- (선택) 배치 크기 — 서브에이전트 한 명이 한 번에 추론할 시나리오 최대 수. 생략하면 30.

돌려주는 정보(디스패치 매니페스트):
- 플랜과 적용된 배치 크기
- 배치 크기가 권장 상한(30)을 넘었는지 알리는 위반 플래그
- 유닛별 정보 묶음 — 유닛 식별·제목, 그 유닛의 시나리오 지식 항목, 시나리오 개수, 필요한 배치 수, 미리 발급한 배치 ID 목록, 그리고 배치별로 담을 시나리오 수(앞쪽 배치는 가득, 마지막 배치는 나머지 — 예: 시나리오 35개·배치 크기 30이면 [30, 5]). 서브에이전트가 시나리오 목록을 결정적으로 쪼갤 수 있도록 이 분배표를 함께 준다.

## 권한 / 제약

- 배치 크기는 1 이상 200 이하여야 한다(범위를 벗어나면 거절). 권장 상한은 30.
- 시나리오가 자동 분할로 처리 가능한 경우(예: 35개를 30+5로 쪼개는 것)는 위반이 아니다. 위반 플래그는 "요청한 배치 크기 자체가 권장 상한 30을 넘을 때"만 켜진다 — 즉 호출자가 안티패턴을 요구했다는 신호이지, 시나리오 수가 많다는 신호가 아니다.
- 시나리오 수는 유닛에 첫 번째로 붙은 지식 항목 내용에서 시나리오 ID 패턴이 들어간 줄 수로 추산한다(휴리스틱). 붙은 지식이 없으면 0개로 보고 배치 1개로 처리한다.
- 읽기 전용 — DB 상태를 변경하지 않는다.

## provenance

데몬의 `daemon/src/routes/discover.rs`(dispatch_plan 핸들러, GET `/discover-loop/dispatch-plan`)가 제공한다. CLI `clawket discover-loop dispatch-plan` 이 사용자 표면이다. 발급되는 배치 ID 형식(BATCH-로 시작하는 ULID)과 시나리오 ID 패턴(US-로 시작하는 식별자)은 TSV 근거 검증 엔드포인트가 받는 형식과 짝을 이루므로 함께 유지되어야 한다.

## 미확정 (OPEN)
- [ ] OPEN: 유닛에 지식 항목이 여러 개 붙은 경우 "첫 번째"만 시나리오 출처로 삼는 정책이 의도된 것인지(여러 항목 합산이 아닌 이유) 확인 필요.
