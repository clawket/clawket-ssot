---
id: concept.tsv-evidence
kind: Concept
title: TSV 근거(7-field)
definition: "검증 라운드에서 시나리오 한 건을 코드에 대조한 판정을 한 행으로 적은 근거 기록. 한 행은 일곱 칸 — 어느 시나리오인지·판정(통과/결함/시나리오오류)·왜 그렇게 봤는지(Given→When→Then 코드 추적)·근거 위치(file:line)·어느 모델 등급으로 봤는지·어느 배치인지·등급을 올렸다면 그 사유 — 으로 구성된다. 서브에이전트가 산출하며, 곧바로 반영하지 않고 배치별로 모아두었다가 한꺼번에 태스크로 동기화한다."
relatesTo:
  - to: concept.scenario
    type: relates-to
    note: 각 근거 행은 시나리오 한 건(scenario_id)에 대한 판정이다
  - to: concept.tier
    type: relates-to
    note: 어느 모델 등급으로 판정했는지(tier_used)와 등급 상향 사유(escalation_reason)를 행에 담는다
  - to: concept.knowledge
    type: belongs-to
    note: 모인 근거 행들은 라운드 증거 지식(type=evidence) 항목으로 보관된다
  - to: persona.qa-reasoner-subagent
    type: backed-by
    note: 배치당 1개로 스폰되는 QA 추론 서브에이전트가 근거 행을 생산한다
  - to: concept.convergence
    type: relates-to
    note: 근거 행의 판정 분포(결함·시나리오오류 개수)가 라운드 종료 시 수렴 판정의 입력이 된다
governedBy: []
governs: []
dependsOn: []
implementedIn:
  - clawket/skills/clawket-verify-loop/SKILL.md
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:concept
---

<!-- 작성 고도: 자연어 우선. 코드 식별자는 provenance/근거에만. -->

## 정의

TSV 근거는 검증 라운드의 실제 산출물이다. 서브에이전트가 시나리오 묶음을 받아 코드와 하나씩 대조하면서, 시나리오 한 건당 판정 한 행을 탭으로 구분된 표(TSV) 형태로 적는다. 이 표가 "코드가 시나리오대로 동작하는가"에 대한 행 단위 증거가 된다.

한 행은 일곱 칸을 가진다.

- **어느 시나리오인가(scenario_id)**: `US-<도메인>-<번호>` 형식. 시나리오를 가리키지 않는 행은 거부된다.
- **판정(status)**: 셋 중 하나 — 통과(pass) / 결함(defect) / 시나리오오류(scenario_error). "통과"는 코드가 시나리오대로 동작함, "결함"은 코드가 잘못됨(고쳐야 함), "시나리오오류"는 시나리오 자체가 잘못됨(시나리오를 고쳐야 함)을 뜻한다.
- **왜 그렇게 봤나(reasoning)**: Given→When→Then 을 코드로 따라간 추적 설명. 모든 행에 필수.
- **근거 위치(evidence)**: 판정의 근거가 된 코드 위치(file:line). 모든 행에 필수다 — 통과 행에도 적어 회귀(regression) 기준선을 남긴다.
- **어느 등급으로 봤나(tier_used)**: 판정에 사용한 모델 등급(haiku / sonnet / opus). 항상 채운다.
- **어느 배치인가(batch_id)**: 같은 서브에이전트 디스패치에서 나온 행들을 묶는 식별자(`BATCH-<26자 ULID>`). 모든 행에 필수.
- **등급 상향 사유(escalation_reason)**: 최상위 등급(opus)으로 판정했을 때 그 사유. opus 행에는 필수.

핵심 규율은 **"판정과 반영의 분리"**다. 서브에이전트의 출력은 즉시 시스템에 반영하지 않고, 배치별로 근거(TSV)에 누적한다. 모든 배치의 TSV가 도착한 뒤에야 한 번에 태스크로 옮겨 적는다(동기화). 그리고 이 동기화 단계에는 어떠한 추론도 끼어들면 안 된다 — 판정은 이미 TSV에서 끝났고, 동기화는 그 결과를 그대로 옮길 뿐이다.

## 엔티티 (DB)

TSV 근거는 자체 테이블이 아니라 두 형태로 데이터에 남는다.

- **모인 근거 묶음**: 라운드별로 모인 행들은 위키 지식(type=evidence, 제목 "Round R evidence — <도메인>") 항목 본문에 보관된다(→ concept.knowledge).
- **태스크로 동기화된 판정**: 동기화 시 데몬이 판정을 태스크 상태로 매핑한다 — 통과→done, 결함→blocked, 시나리오오류→cancelled. 시나리오오류 행의 보정 제안(scenario_amendment)·등급 상향 사유(escalation_reason)도 함께 태스크에 반영되고, 다시 시도할 행들의 큐 지식과 라운드 증거 지식이 한 트랜잭션으로 기록된다.

행의 일곱 칸 중 scenario_id·status·tier_used·batch_id·escalation_reason 은 검증 태스크의 대응 필드(scenario_id·상태·tier/tier_used·batch_id 등)로 이어진다.

## API 표면

- **스키마 검증·동기화**: 데몬의 검증 동기화 경로(discover-loop sync)가 7칸 형식을 검사하고 판정→상태 매핑·증거 지식 작성·재시도 큐 생성을 한 트랜잭션으로 수행한다. 결함·시나리오오류 행에 근거 위치(file:line)가 없으면 행이 거부된다.
- **에이전트측 강제**: 데몬은 스키마 수준 가드만 하고, 운영상 안티패턴(시나리오ID 없는 태스크, 배치 30 초과, 동기화 단계에 추론 섞기)은 에이전트측 훅과 코드리뷰가 막는다.

## 불변식

- **시나리오ID 필수**: scenario_id 없는 행/태스크는 거부된다.
- **근거 위치 필수**: 모든 행에 evidence(file:line)가 있어야 한다. 특히 결함·시나리오오류 행에 근거가 없으면 그 행은 거부된다(통과 행도 회귀 기준선을 위해 근거를 남긴다).
- **배치ID 필수**: 모든 행에 batch_id 가 있어야 한다.
- **상향 사유 조건부 필수**: 최상위 등급(opus)으로 판정한 행에는 escalation_reason 이 있어야 한다.
- **판정 ≠ 반영**: 서브에이전트 출력은 즉시 반영하지 않고 배치별로 누적한 뒤 일괄 동기화한다. 동기화 핸들러 안에서는 어떤 추론도 수행하지 않는다(판정은 TSV에서 끝났고 동기화는 전사뿐). 추론이 섞이면 중단된다.
- **배치 크기 한도**: 한 배치는 시나리오 30개 이하. 30을 넘으면 디스패치가 거부된다(주의 분산 방지).

## 구현 위치 (provenance)

- 7칸 스키마 정의·각 칸의 필수/조건부 규칙·"판정 ≠ 반영" 규율·배치 30 한도: `clawket/skills/clawket-verify-loop/SKILL.md`(TSV evidence schema / Bulk sync 절).
- 판정→태스크 상태 매핑(pass→done / defect→blocked / scenario_error→cancelled)과 동기화 트랜잭션도 같은 문서에 기술됨(데몬 discover-loop sync 위임).

## 미확정 (OPEN)
- [ ] OPEN: 데몬의 verify-tsv / sync 엔드포인트가 7칸 검증을 어느 정밀도까지(정규식·필수칸·배치한도) 강제하는지 데몬 소스 추적 미완 — 스킬 문서상 "스키마 수준 가드"라고만 확인됨.
- [ ] OPEN: tier_used 의 허용값이 스킬 문서에서는 haiku/sonnet/opus(모델명)인데 태스크 tier 개념은 low/med/high(등급)다. 두 어휘의 변환 지점(어느 단계에서 모델명↔등급이 매핑되는지) 추적 미완(→ concept.tier 와 교차).
