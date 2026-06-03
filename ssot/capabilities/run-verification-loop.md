---
id: capability.run-verification-loop
kind: Capability
title: 검증 라운드 실행
definition: 한 라운드(Round R) 동안 시나리오를 코드에 대조해 검증하는 전 과정을 끝까지 돌리는 기능. 시나리오 묶음을 배치로 나눠 서브에이전트에 디스패치하고, 7필드 TSV 근거를 모아 일괄 동기화하며, 결함·시나리오오류·수렴의 3-way 판정으로 다음 라운드를 잡거나 루프를 닫는다.
purpose: 코드와 시나리오를 함께 진화시켜, 검증이 사람의 직관이 아니라 추적 가능한 근거(scenario_id·status·이유·file:line)에 기반하게 하고, 결함과 시나리오 오류가 모두 사라질 때까지 라운드를 자동으로 이어 수렴을 객관적으로 판정한다.
servesPersona:
  - persona.llm-coding-agent
  - persona.qa-reasoner-subagent
realizedBy:
  - component.cli
  - component.daemon
  - component.skills
implementedIn:
  - clawket/skills/clawket-verify-loop/SKILL.md
relatesTo:
  - to: endpoint.skill-verify-loop
    type: realizes
    note: 이 검증 라운드 러너 스킬의 절차를 그대로 실현한다
  - to: endpoint.cli-discover-loop
    type: calls
    note: 라운드 플랜·배치 매니페스트·TSV 검증·벌크 동기화·수렴 판정을 데몬 명령으로 수행한다
  - to: concept.scenario
    type: evaluates
    note: 시나리오를 코드에 대조해 통과/결함/시나리오오류를 판정한다
  - to: concept.tsv-evidence
    type: mutates
    note: 판정 결과를 배치별 7필드 TSV 근거로 누적·동기화한다
  - to: concept.convergence
    type: evaluates
    note: 라운드 종료 시 3-way 수렴 판정을 산출한다
  - to: concept.tier
    type: reads
    note: 주의 희석 의심 시 상위 티어(Opus)로 에스컬레이션하고 그 사유를 기록한다
  - to: concept.task
    type: mutates
    note: 동기화 시 판정 상태를 태스크 상태(done/blocked/cancelled)로 매핑한다
  - to: capability.author-scenarios
    type: leads-to
    note: scenario_error 행은 시나리오 정제 기능으로 넘긴다
  - to: capability.defect-fix
    type: leads-to
    note: defect 행은 결함 수정 기능으로 넘긴다
impacts:
  - concept.task
  - concept.scenario
  - domain.pdd-verification
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:capability
---

## 사용자가 할 수 있는 일

검증을 한 번의 판정이 아니라 라운드의 반복으로 돌린다. 한 라운드는 "지금 시나리오가 현재 코드에서 실제로 성립하는가"를 시나리오 하나하나에 대해 따져, 통과(pass)·결함(defect)·시나리오오류(scenario_error)로 가른다. 결함이면 코드가 틀린 것이고, 시나리오오류면 시나리오가 의도와 어긋난 것이다 — 코드도 시나리오도 신성하지 않으며, 의도와 동작이 갈리면 어느 쪽이든 고친다(단, 시나리오를 고칠 수 있는 유일한 정당 사유는 "의도 불일치"뿐이고, 시간·비용·복잡도·코드 변경량은 사유가 되지 못한다).

핵심 가치는 객관적 종료 조건이다. 라운드가 "충분해 보인다"가 아니라, 결함=0이고 시나리오오류=0인 상태가 2라운드 연속일 때만 수렴으로 닫힌다. 그리고 모든 판정에는 file:line 단위 근거가 강제되므로, "왜 통과/결함이라 했는지"가 항상 추적된다.

## 행위

- 라운드 진입 점검: active 플랜·active 사이클이 있는지, 각 유닛의 시나리오 지식이 "현재 의도만" 담고 있는지(히스토리 누수 없음) 확인한다.
- 배치 디스패치: 유닛별 시나리오를 30개 이하 배치로 쪼개, 배치당 서브에이전트 1개를 스폰한다(30 초과는 주의 희석으로 거부).
- 7필드 TSV 근거 수집: 시나리오마다 scenario_id·status·이유·근거(file:line)·사용 티어·batch_id·에스컬레이션 사유를 한 행으로 산출한다. 근거가 없는 결함/시나리오오류 행은 거부된다.
- 벌크 동기화: 모든 배치 TSV가 도착하면 추론 없이 일괄 전사한다(pass→done, defect→blocked, scenario_error→cancelled로 매핑). 대규모 라운드는 배치별 스트리밍 동기화를 쓴다.
- 3-way 수렴 판정: 라운드 종료 시 결함·시나리오오류 수를 세어 분기한다.
- 다음 라운드 예약 또는 종료: 수렴 조건을 못 채우면 다음 라운드를 잡고, 채우면 플랜 본문에 수렴 기록을 남기고 루프를 닫는다.

## 시스템 흐름

라운드가 시작되면 active 플랜과 사이클 존재를 확인한 뒤, 데몬에 배치 매니페스트를 요청해 유닛·배치 단위로 시나리오를 나눈다. 배치마다 QA 추론 서브에이전트가 하나씩 스폰되어 시나리오를 Given→When→Then 순으로 코드에 추적하고, 그 판정을 즉시 동기화하지 않고 배치별 TSV 근거로 누적한다. 후반 배치에서 결함율이 급증하면(주의 희석) 서브에이전트가 스스로 상위 티어로 에스컬레이션하고 사유를 7번째 칼럼에 적는다. 모든 TSV가 도착하면 데몬이 한 트랜잭션으로 상태 매핑·시나리오 수정 추출·에스컬레이션 사유 저장·재시도 큐 생성·라운드 근거 지식 작성을 처리한다. 이후 데몬이 결함·시나리오오류 수를 세어 판정을 돌려주면, 결함은 결함 수정 기능으로·시나리오오류는 시나리오 정제 기능으로 넘기고, 둘 다 0이고 2라운드 연속이면 수렴으로 닫는다. 직전 라운드보다 결함이 늘면 회귀로 보아 즉시 우선순위 조사 태스크를 등록하고 사람 확인을 청한다.

## 어디에 구현되어 있나

이 기능의 절차는 검증 라운드 러너 스킬(`clawket/skills/clawket-verify-loop/SKILL.md`)에 정의되어 있고, 결정적 상태 처리(라운드 플랜·사이클·유닛 자동 생성, 배치 매니페스트, TSV 스키마 검증, 벌크 전사, 수렴·라운드 조회)는 데몬의 discover-loop 명령군이 소유한다. 스킬은 LLM 판단 영역(시나리오 추론·에스컬레이션·수렴 기록 문구)을, 데몬은 결정적 상태를 맡아 책임을 분담한다.

## 미확정 (OPEN)
- [ ] OPEN: 배치 디스패치 시 서브에이전트 스폰의 정확한 도구 경로(Agent/Task 도구)와 SubagentStart 훅의 scenario_id·배치크기 강제의 연동 지점 명시 필요
- [ ] OPEN: clawket-verify-batch(운영 패턴) 스킬과 본 러너의 책임 경계 — 디스패치+동기화 위임의 정확한 분기점 확인 필요
