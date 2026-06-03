---
id: invariant.sync-purity
kind: Invariant
title: 벌크 동기화는 추론 없이 TSV가 정한 status만 전사한다
definition: "TSV→DB 벌크 동기화 단계는 TSV가 이미 결정한 status를 옮겨 적기만 해야 하며, 동기화 코드 안에 판정·분기 추론이 섞이면 차단된다(기본 strict)."
governs:
  - concept.tsv-evidence
  - domain.pdd-verification
  - capability.run-verification-loop
  - endpoint.cli-discover-loop
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
  - to: endpoint.hook-pre-tool-use
    type: relates-to
    note: Bash 도구 호출에서 동기화 코드의 추론 혼입을 검사하는 훅
  - to: persona.qa-reasoner-subagent
    type: relates-to
    note: 판정 추론은 서브에이전트 디스패치 단계에서만, 동기화 단계와 분리
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지. -->

## 제약

검증 루프는 두 단계로 깔끔히 나뉘어야 한다. 먼저 QA 추론 서브에이전트가 시나리오를 코드에 대조해 통과·결함·시나리오오류 중 하나로 판정하고, 그 결과를 TSV 근거로 남긴다. 그다음 동기화 단계가 이 TSV를 받아 작업 상태로 옮겨 적는다(통과→완료, 결함→차단, 시나리오오류→취소로 대응).

이 제약은 두 번째 단계, 즉 동기화 단계에 추론이 끼어들어선 안 된다는 것이다. 동기화는 "TSV가 이미 내린 결론을 그대로 전사"만 한다. 동기화 코드 안에서 새로 무언가를 판단하거나(예: 신호를 보고 상태를 if/else로 가르거나, 다시 추론·분류·평가하거나), 동기화가 진행 중인 와중에 새 추론 에이전트를 띄우는 행위는 모두 단계 혼합으로 본다.

기본 동작은 strict(차단)이며, 환경 설정으로 warn(경고만)이나 off(검사 안 함)로 바꿀 수 있다. 이 검사는 동기화 블록이 명시적으로 표시된 구간 안에서만 작동하므로, 평범한 다중 업데이트 스크립트나 무관한 명령은 오탐되지 않는다.

## 깨지면 무슨 일이 일어나나

동기화 단계가 판정까지 겸하면, "코드 대조 근거"와 "상태 결정"이 한 곳에 뒤섞여 어느 판정이 증거에 기반했는지 추적이 불가능해진다. 같은 TSV를 다시 동기화해도 결과가 달라질 수 있고(전사가 멱등하지 않게 됨), 증거 불변식(invariant.evidence-on-tsv)이 무력화된다. 결국 판정 책임 소재가 흐려져 검증 루프 전체의 신뢰성이 무너진다. 이는 단일 책임 분리(판정 ↔ 전사) 원칙을 깨는 것이다.

## 코드에서 어떻게 강제되나

어댑터의 훅(X9 sync-reasoning 체크)이 Bash 도구 호출을 가로채, 동기화 컨텍스트가 켜진 구간에서 동기화 패턴(일괄 업데이트·스레드풀 등)과 추론 흔적(reasoning/decide/classify 같은 어휘, 또는 신호를 상태값으로 가르는 조건 분기)이 함께 나타나면 strict 모드에서 차단한다. 동기화 진행 중 새 에이전트 디스패치를 시도하면 추론 어휘 없이도 차단된다. 데몬에 연결할 수 없으면 검사를 건너뛴다. 검증 루프 스킬 문서도 "동기화 핸들러 안에 추론이 있으면 중단", "데몬의 일괄 동기화는 상태 매핑 외 분기가 없다"를 작업 규칙으로 명시한다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요 — sync-purity 3-mode 토글(strict/warn/off) 도입 결정(PDD A8) 노드가 SSOT에 아직 없음
