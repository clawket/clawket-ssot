---
id: invariant.scenario-amend-intent-only
kind: Invariant
title: 시나리오 수정의 유일한 정당 사유는 의도 불일치다
definition: "시나리오를 분할·재정의·삭제하는 유일하게 정당한 사유는 의도 불일치(intent mismatch)이며, 시간·비용·복잡도·코드 영향 크기를 사유로 한 시나리오 약화는 거부된다."
governs:
  - concept.scenario
  - domain.pdd-verification
  - capability.author-scenarios
implementedIn:
  - clawket/skills/clawket-verify-loop/SKILL.md
  - clawket/skills/clawket-scenario-refine/SKILL.md
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
  - to: capability.defect-fix
    type: relates-to
    note: 코드 영향이 크면 시나리오를 약화하지 말고 별도 결함수정 플랜으로 모든 경로를 덮는다
  - to: concept.convergence
    type: relates-to
    note: 시나리오 약화로 수렴을 앞당기는 우회를 막아 수렴의 의미를 지킨다
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지. -->

## 제약

검증 라운드에서 시나리오 자체가 잘못됐다고 판정된 행(scenario_error)을 처리할 때, 시나리오를 손대는(분할·의도 재정의·삭제) 유일하게 받아들여지는 이유는 **의도 불일치**다. 즉 시나리오가 두 가정을 한데 섞고 있거나(원자적이지 않음), 기대 결과가 제품 비전과 모순되거나, 해당 시나리오가 다음 큰 범위로 미뤄진 경우다.

반대로 "시간이 없어서", "토큰을 아끼려고", "복잡해서", "코드 영향이 커서" 같은 사유로 시나리오를 합치거나 약하게 고치는 것은 거부된다. 이런 사유가 감지되면 정제 작업을 멈추고 사용자 확인을 요청한다. 코드 영향이 크다면 시나리오를 약화하는 대신, 영향받는 모든 코드 경로를 덮는 별도 결함수정 플랜을 등록해야 한다.

## 깨지면 무슨 일이 일어나나

만약 시간·비용을 이유로 시나리오를 약화할 수 있게 두면, 검증 통과는 "코드가 의도를 만족함"이 아니라 "시나리오를 통과하기 쉽게 깎았음"을 뜻하게 된다. 수렴(결함=0·시나리오오류=0)이 품질의 증거가 아니라 회피의 결과로 전락한다. 이 제약은 일정·비용 트레이드오프를 시나리오의 진실성과 분리해, 검증이 제품 완성도를 지키는 장치로 남도록 한다.

## 코드에서 어떻게 강제되나

코드의 자동 차단이 아니라 작업 절차(스킬 문서)로 강제된다. 시나리오 정제 스킬은 (1) 수정 사유를 먼저 검증해 의도 불일치만 통과시키고 시간·비용·복잡도·코드영향 사유는 거부하며, (2) 의심스러운 수정은 사용자 확인을 받고, (3) 코드 영향이 크면 별도 수정 플랜으로 우회하도록 명시한다. 검증 루프 스킬도 "시나리오의 유일한 정당 수정 사유는 의도 불일치이며 시간·비용·복잡도·코드영향 크기는 거부"라고 못 박는다. 자기점검 항목에도 "수정 사유가 의도 불일치인지" 확인이 들어 있다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요 — 시나리오 약화 금지 정책을 정한 결정 기록 노드가 SSOT에 아직 없음
- [ ] OPEN: 자동 차단(훅) 없이 절차 규칙으로만 강제됨 — 코드 수준 가드 도입 여부 미확인
