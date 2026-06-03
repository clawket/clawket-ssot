---
id: invariant.convergence-two-rounds
kind: Invariant
title: 수렴은 결함=0·시나리오오류=0이 2라운드 연속일 때만 인정된다
definition: "검증 루프는 한 라운드에서 결함=0이고 시나리오오류=0인 것만으로는 종료하지 않으며, 두 라운드 연속으로 둘 다 0일 때만 수렴(종료)으로 판정한다."
governs:
  - concept.convergence
  - concept.tsv-evidence
  - domain.pdd-verification
  - capability.run-verification-loop
implementedIn:
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
  - to: endpoint.cli-discover-loop
    type: relates-to
    note: converged 판정과 라운드 카운트를 담당하는 데몬 명령
  - to: concept.scenario
    type: relates-to
    note: 수렴 판정의 집계 단위가 시나리오별 판정 행
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지. -->

## 제약

검증 루프는 코드와 시나리오를 함께 진화시키며 라운드를 반복한다. 한 라운드가 끝날 때 결함 건수와 시나리오오류 건수를 센다. 이 둘이 모두 0이 된다고 해서 곧바로 "끝"으로 보지 않는다. **직전 라운드와 이번 라운드, 연속 두 번** 모두 결함=0이고 시나리오오류=0일 때에 한해 수렴으로 판정하고 루프를 종료한다. 한 라운드만 0이고 그 직전 라운드에 결함이나 시나리오오류가 있었다면, 다음 라운드를 한 번 더 돌려 0이 유지되는지 확인한다.

## 깨지면 무슨 일이 일어나나

한 라운드만 0인 것으로 종료해 버리면, 우연히 그 라운드의 배치가 결함을 놓쳤거나(주의력 희석 등) 직전 수정이 새 결함을 심어 놓은 상태(회귀)를 종료로 오인할 수 있다. 두 라운드 연속 0을 요구하는 것은 "한 번의 0은 운일 수 있다"는 가정 아래 안정성을 한 번 더 확인하는 안전장치다. 이 안전장치가 빠지면 미완성 코드가 "검증 통과"로 봉인되어 결함이 사용자에게 흘러간다.

## 코드에서 어떻게 강제되나

라운드 종료 시 데몬의 discover-loop 상태 조회가 결함·시나리오오류 건수를 집계하고(작업의 qa_status 또는 status를 묶어 카운트하여 부분적 칼럼 드리프트가 수치를 왜곡하지 않게 한다), converged 판정 명령이 두 라운드 연속 0 조건을 확인해 충족 시에만 종료(exit 0)를 반환한다. 종료 시 플랜 본문에 수렴 기록(직전 두 라운드의 각 0 카운트가 증거)이 자동 첨부된다. 한 라운드만 0이고 직전이 0이 아니면 다음 라운드(R+1)를 예약한다. 이 절차는 검증 루프 스킬 문서에 작업 규칙으로 정의돼 있다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요 — "2라운드 연속" 임계값을 정한 결정 기록 노드가 SSOT에 아직 없음
