---
id: domain.pdd-verification
kind: Domain
title: PDD 검증·수렴 루프
purpose: "코드와 시나리오를 한 번에 완성하려 하지 않고, 라운드를 반복하며 둘을 함께 진화시켜 의도와 동작이 어긋난 지점을 결함 또는 시나리오 오류로 분리·해소해 안정적인 수렴에 이르게 한다."
definition: "시나리오 작성 → 플랜 설계 → 서브에이전트 배치 검증 → TSV 근거 수집·동기화 → 3-way 수렴 판정으로 한 라운드를 돌리고, 결함=0이며 시나리오 오류=0이 2라운드 연속일 때 루프를 종료하는 영역. 검증 단위는 원자적 시나리오이며 각 시나리오에 태스크가 1:1로 매핑된다. 코드도 시나리오도 신성하지 않아 의도와 동작이 벌어지면 어느 쪽이든 고친다 — 단 시나리오 수정의 유일한 정당 사유는 의도 불일치다. 임베딩·검색이나 훅 차단 메커니즘 자체는 이 영역 밖이다."
servesPersona:
  - persona.qa-reasoner-subagent
  - persona.llm-coding-agent
  - persona.human-operator
relatesTo:
  - to: domain.work-management
    type: depends-on
    note: "각 시나리오가 태스크 1:1로 매핑되고 라운드가 플랜·사이클·유닛 위에서 돌아가, 작업 관리 영역의 워크플로우 엔티티를 검증의 골격으로 쓴다."
  - to: domain.hook-enforcement
    type: depends-on
    note: "시나리오 ID 추적·배치 한도·근거 필수·동기화 순수성 같은 검증 규율을 훅 강제 영역이 작업 시점에 막아 준다."
  - to: domain.knowledge-rag
    type: feeds
    note: "시나리오·TSV 근거·라운드 결과·감사 로그를 지식 항목으로 남기고, 다음 라운드가 이를 다시 검색해 참조한다."
governedBy:
  - invariant.scenario-id-one-to-one
  - invariant.batch-size-30
  - invariant.evidence-on-tsv
  - invariant.sync-purity
  - invariant.convergence-two-rounds
  - invariant.scenario-amend-intent-only
  - invariant.scenario-snapshot-only
realizedBy:
  - component.daemon
  - component.skills
impacts:
  - capability.run-verification-loop
  - capability.author-scenarios
  - capability.defect-fix
  - capability.discover-loop-run
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:domain"]
---

## 목적

큰 변경을 한 번에 "완성"으로 선언하는 대신, 시나리오(의도)와 코드(동작)를 라운드마다 나란히 검증해 어긋난 곳을 드러낸다. 어긋남은 두 종류다 — 코드가 잘못된 결함이거나, 시나리오가 잘못된 시나리오 오류. 둘을 분리해 각각 다른 후속 작업(결함 수정 / 시나리오 정제)으로 보내고, 두 종류 모두 0이 두 라운드 연속 유지될 때 비로소 수렴으로 본다. 시간·비용·복잡도·코드 변경 크기는 시나리오를 고칠 사유로 인정하지 않는다 — 유일한 정당 사유는 의도 불일치다.

## 경계와 핵심 개념

포함:
- 시나리오 — As-a/I-want/So-that + Given/When/Then 형식의 원자적 사용자 시나리오. `US-<도메인>-<번호>` ID를 가지며 검증의 단위이고 태스크가 1:1로 붙는다.
- TSV 근거(7필드) — 행마다 시나리오 ID·판정(pass/defect/scenario_error)·추론·근거(파일:줄)·사용 티어·배치 ID·에스컬레이션 사유. 결함/시나리오 오류 행은 물론 통과 행에도 근거가 필수다(회귀 기준선 확보).
- 배치 검증 — 한 서브에이전트가 한 유닛의 시나리오 묶음(최대 30개)을 코드에 대조한다. 30을 넘으면 주의력 희석이 시작되므로 금지된다.
- 벌크 동기화 — 모든 TSV가 도착한 뒤 판정을 상태로 전사한다(pass→done / defect→blocked / scenario_error→cancelled). 이 단계에는 추론이 끼면 안 된다 — 판정은 TSV가 이미 내렸고, 동기화는 옮겨 적기만 한다.
- 3-way 수렴 판정 — 라운드 종료 시 결함≥1, 시나리오 오류≥1, 또는 둘 다 0의 세 갈래로 분기한다.

제외:
- 작업 상태 머신의 일반 규칙(작업 관리 영역), 임베딩·하이브리드 검색의 메커니즘(지식·RAG 영역), 훅 차단의 구현(훅 강제 영역) — 이 영역은 그것들을 사용·산출할 뿐 소유하지 않는다.

## 기능

- 검증 라운드 실행 — 배치 디스패치·근거 수집·벌크 동기화·수렴 판정으로 한 라운드를 끝까지 돈다(`capability.run-verification-loop`, `capability.discover-loop-run`).
- 시나리오 작성·정제 — 도메인별 원자 시나리오를 쓰고, 시나리오 오류 행에 대해 분할/의도 재정의/삭제를 결정한다(`capability.author-scenarios`).
- 결함 수정 — 결함 행마다 결함 해결 플랜의 해당 라운드 유닛 아래 수정 태스크를 등록하고 코드를 고친다(`capability.defect-fix`).

실현 주체는 결정적 상태를 소유하는 데몬(디스패치 매니페스트·TSV 스키마 검증·벌크 전사·수렴/회귀 카운트)과, LLM 판단 영역을 담당하는 번들 스킬들이다.

## 시스템 흐름

라운드 진입 시 active 플랜·사이클을 확인하고, 각 유닛의 시나리오 지식이 현재 의도만 담고 있는지(이력 혼입 없음) 점검한다. 이어 유닛별로 시나리오를 30개 이하 배치로 쪼개 서브에이전트를 하나씩 띄운다. 각 서브에이전트는 시나리오를 코드에 대조해 7필드 TSV 근거를 산출하되, 그 결과는 즉시 동기화하지 않고 배치별 근거로 누적된다.

모든 TSV가 도착하면 별도 단계에서 벌크 동기화한다 — 추론 없이 판정만 상태로 옮긴다. 데몬은 상태 매핑, 시나리오 수정 사유 추출, 에스컬레이션 사유 영속, 재시도 큐 지식 생성, 라운드 근거 지식 작성을 한 트랜잭션으로 처리한다.

라운드 종료 시 데몬이 결함·시나리오 오류를 세어 3-way로 분기한다. 결함이 있으면 결함 수정 흐름으로, 시나리오 오류가 있으면 시나리오 정제 흐름으로 보낸다(수정 태스크가 완료되면 원래 QA 태스크가 자동 해제된다). 둘 다 0이고 그 상태가 2라운드 연속이면 수렴으로 종료하고 플랜 본문에 수렴 증거가 자동 추가된다 — 아니면 다음 라운드를 예약한다. 직전 라운드보다 결함이 늘면 회귀로 보고 우선순위 조사 태스크를 즉시 등록한다.

## 다른 도메인과의 관계

- 작업 관리 영역에 의존한다(`depends-on`) — 시나리오↔태스크 1:1 매핑과 플랜·사이클·유닛 골격 위에서 라운드가 돈다.
- 훅 강제 영역에 의존한다(`depends-on`) — 시나리오 추적·배치 한도·근거 필수·동기화 순수성을 작업 시점에 막아 준다.
- 지식·RAG 영역에 자료를 공급한다(`feeds`) — 시나리오·근거·라운드 결과·감사 로그를 지식으로 남기고 후속 라운드가 검색해 참조한다.

## 미확정 (OPEN)
- [ ] OPEN: 주의력 희석 자가 점검의 임계값(후반 배치 결함률 델타 기준)이 정책으로 고정인지 휴리스틱인지 미확정.
- [ ] OPEN: 스트리밍 동기화(대규모 라운드)와 단일 호출 동기화 사이의 선택 기준이 규칙으로 정해지는지 미검증.
