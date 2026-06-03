---
id: invariant.evidence-on-tsv
kind: Invariant
title: TSV 근거 행에는 file:line 증거가 반드시 있어야 한다
definition: "검증 라운드의 TSV 판정 행, 특히 defect·scenario_error 행은 file:line 형태의 증거가 없으면 동기화에서 거부된다."
governs:
  - concept.tsv-evidence
  - concept.scenario
  - domain.pdd-verification
  - capability.run-verification-loop
implementedIn:
  - clawket/adapters/shared/claude-hooks.cjs
  - clawket/skills/clawket-verify-loop/SKILL.md
decidedBy: []
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:invariant
relatesTo:
  - to: endpoint.hook-subagent-stop
    type: relates-to
    note: 태스크 done/blocked 전환 시 증거 유무를 검사하는 훅
  - to: endpoint.cli-discover-loop
    type: relates-to
    note: sync 단계에서 증거 없는 행을 거부하는 데몬 명령
  - to: persona.qa-reasoner-subagent
    type: backed-by
    note: 7필드 TSV를 산출하는 주체가 증거 칸을 채워야 한다
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지. -->

## 제약

검증 라운드에서 QA 추론 서브에이전트가 내놓는 판정은 시나리오 한 건마다 한 행의 근거(TSV)로 기록된다. 이 근거 행에는 "왜 그렇게 판정했는가"를 코드의 어느 위치에서 확인했는지를 가리키는 증거가 반드시 들어가야 한다. 증거는 파일과 줄 번호를 함께 적는 형태(예: 어떤 소스 파일의 몇 번째 줄)다.

특히 결함(defect)으로 판정한 행과 시나리오 자체가 잘못됐다고 판정한 행(scenario_error)은 증거가 비어 있으면 받아들이지 않는다. 통과(pass)로 판정한 행에도 증거를 요구하는데, 이는 다음 라운드에서 회귀(전에 통과하던 것이 깨짐)를 잡아낼 기준선을 남기기 위해서다.

증거 칸의 내용에도 최소·최대 길이 기준이 있다. 너무 짧으면(자리채우기용 placeholder로 의심되면) 거부되고, 4KiB를 넘기는 장황한 내용도 거부된다 — 간결한 file:line 참조나 요약으로 압축해야 한다.

## 깨지면 무슨 일이 일어나나

증거 없이도 판정이 통과된다면, "결함이다"라는 주장이 코드 어디에도 근거를 두지 않은 채 작업 큐에 들어가게 된다. 그러면 결함 수정 담당이 무엇을 고쳐야 하는지 추적할 수 없고, 통과 판정도 다음 라운드에서 회귀를 비교할 기준이 사라진다. 결국 검증 루프의 수렴 판정(결함=0, 시나리오오류=0)이 근거 없는 숫자가 되어, "수렴했다"는 결론 자체를 신뢰할 수 없게 된다.

## 코드에서 어떻게 강제되나

두 지점에서 강제된다. 첫째, 태스크 상태 전환(특히 done·blocked·cancelled로 갈 때) 시점에 어댑터의 훅(X8 evidence 체크)이 증거 칸이 비었는지, 너무 짧거나 너무 긴지 검사해 부적합하면 전환을 막는다. 둘째, TSV를 일괄 동기화하는 데몬 명령이 스키마 수준에서 증거 없는 defect·scenario_error 행을 거부한다. 검증 루프 스킬 문서도 "모든 행에 file:line 증거 필수, defect·scenario_error 행은 증거 없으면 행 자체 거부"를 작업 규칙으로 못 박는다.

데몬에 연결할 수 없는 상황에서는 이 훅 검사를 건너뛴다(경고만 남김) — 즉 강제는 데몬이 살아 있을 때 성립한다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요 — 이 규칙이 도입된 결정 기록(PDD A 시리즈) 노드가 SSOT에 아직 없음
