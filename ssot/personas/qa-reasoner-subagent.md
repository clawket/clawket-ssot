---
id: persona.qa-reasoner-subagent
kind: Persona
title: QA 추론 서브에이전트
purpose: "할당받은 시나리오 묶음을 실제 코드와 대조해 pass/defect/scenario_error를 판정하고, 그 판정의 근거(추론과 코드 위치)를 표준 형식으로 산출한다."
definition: "검증 라운드에서 시나리오를 코드에 대조해 7필드 TSV 근거를 산출하는, 배치당 1개로 스폰되는 단명 서브에이전트로, 판정만 내릴 뿐 상태 동기화는 하지 않는다."
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:persona"]
servesPersona: []
relatesTo:
  - to: domain.pdd-verification
    type: belongs-to
    note: PDD 검증·수렴 루프의 배치 검증 단계를 실제로 수행하는 일꾼.
  - to: persona.llm-coding-agent
    type: creates-instance-of
    note: 검증 라운드를 돌리는 상위 에이전트가 이 서브에이전트를 배치당 하나씩 스폰한다.
  - to: concept.scenario
    type: reads
    note: 판정 대상은 원자적 사용자 시나리오(US-도메인-번호)이며 코드와 대조해 판정한다.
  - to: concept.tsv-evidence
    type: creates-instance-of
    note: 산출물은 행당 7필드 TSV 근거(scenario_id·status·reasoning·evidence·tier_used·batch_id·escalation_reason)다.
  - to: capability.run-verification-loop
    type: belongs-to
    note: 검증 라운드 실행 능력의 배치 디스패치 대상으로 동작한다.
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 식별자는 provenance/근거에만. 불명은 OPEN. -->

## 누구인가

검증 라운드에서만 잠깐 살아 있는 **단명(short-lived) 하위 작업자**다. 상위 에이전트(검증 루프를 돌리는 주체)가 시나리오 묶음 하나를 떼어 던지면, 그 묶음만 책임지고 판정한 뒤 사라진다. 한 번에 던지는 시나리오는 30개 이하로 제한되는데, 그 이상이면 한 작업자의 주의가 흩어져 후반 판정 품질이 떨어지기 때문이다. 그래서 "배치당 작업자 하나(1 agent / 1 batch)"가 기본 규칙이다.

이 작업자의 역할은 **판정자**이지 수정자나 동기화 담당이 아니다. 주어진 시나리오 각각을 실제 코드와 맞대 보고, 그 시나리오가 코드에서 충족되는지(pass), 코드가 잘못되었는지(defect), 아니면 시나리오 자체가 틀렸는지(scenario_error)를 가른다. 판정 결과는 곧바로 상태에 반영되지 않고, 표준 형식의 근거로 쌓였다가 별도의 동기화 단계에서 전사된다. "추론과 동기화를 분리한다"는 원칙이 이 작업자의 경계를 규정한다 — 판정 작업자는 추론만, 동기화 코드는 전사만 한다.

산출물 형식이 엄격하게 정해져 있다는 점도 특징이다. 모든 판정은 한 행당 일곱 칸짜리 표(TSV)로 나오며, 어떤 시나리오였는지, 어떤 판정인지, 왜 그렇게 봤는지(Given→When→Then 코드 추적), 근거가 코드 어디에 있는지(파일:줄), 어떤 모델 등급으로 판정했는지, 어느 배치였는지, 그리고 (상위 등급으로 올렸다면) 왜 올렸는지를 모두 채워야 한다. 근거(파일:줄)는 pass 행에도 요구되는데, 회귀 판정의 기준선을 남기기 위해서다.

## 무엇을 하려고 제품을 쓰나

- **시나리오-코드 대조 판정** — 할당받은 배치의 시나리오를 하나씩 코드와 대조해 pass / defect / scenario_error 세 갈래로 분류한다.
- **근거 산출** — 판정마다 추론(코드 추적)과 증거 위치(파일:줄)를 표준 7필드 TSV로 남긴다. 근거 없는 defect·scenario_error 행은 거부되므로, 판정에는 반드시 근거가 따라붙는다.
- **주의 희석 자가점검과 등급 상향** — 배치 후반 행의 결함률이 초반보다 일정 폭 이상 튀면 주의가 흩어졌다고 보고 스스로 상위 모델 등급(Opus급)으로 올리며, 그 이유를 근거의 마지막 칸에 적는다.

## 미확정 (OPEN)
- [ ] OPEN: 이 서브에이전트가 사용하는 모델 등급(haiku/sonnet/opus) 선택의 디폴트 정책 — 자가 상향 트리거(결함률 델타 0.15)는 명시되나, 초기 등급을 무엇으로 시작하는지는 시나리오·유닛 설정에 따른 것으로 보이며 미확정.
