---
id: invariant.batch-size-30
kind: Invariant
title: 검증 배치는 시나리오 30개를 넘을 수 없다
definition: 검증 서브에이전트 한 번의 배치에 시나리오를 30개 초과로 담으면 주의력 희석을 막기 위해 디스패치가 거부된다(기본 strict).
governs:
  - concept.scenario
  - capability.run-verification-loop
  - domain.pdd-verification
implementedIn:
  - clawket/adapters/shared/claude-hooks.cjs
  - clawket/skills/clawket-verify-loop/SKILL.md
decidedBy:
crossesBoundary: true
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:invariant
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 식별자는 provenance/근거에만. -->

## 제약

검증 라운드에서 하나의 서브에이전트(시나리오를 코드에 대조해 판정하는 일꾼)에게 한 번에 맡기는 시나리오는 30개를 넘을 수 없다. 정확히 30개까지는 통과하고, 31개부터는 너무 많은 것으로 보고 그 배치를 거부한다.

이 상한선의 목적은 "주의력 희석"을 막는 것이다. 한 에이전트가 한꺼번에 너무 많은 시나리오를 검증하려 하면, 각 시나리오에 충분한 추론을 들이지 못해 판정 품질이 떨어진다. 30개라는 천장은 한 배치의 검증 깊이를 보장하기 위한 경계다.

## 깨지면 무슨 일이 일어나나

상한이 없다면 한 에이전트가 수백 개 시나리오를 한 프롬프트에 욱여넣을 수 있고, 그 경우 뒤쪽 시나리오는 형식적으로만 "통과"로 처리되거나 근거 없이 판정되어 검증 신뢰도가 무너진다. 결함을 놓치거나 거짓 통과가 섞여, 수렴 판정 자체가 의미를 잃는다.

규칙이 작동하면, 30개를 넘는 배치는 디스패치 단계에서 거부되고, 시스템은 어디서 어떻게 배치를 쪼개야 할지(어느 시나리오 ID 구간으로 나눌지)까지 안내해 에이전트가 적정 크기로 다시 나누도록 유도한다.

## 코드에서 어떻게 강제되나

Claude Code 훅의 공유 핸들러(`adapters/shared/claude-hooks.cjs`)에 있는 배치 크기 검사(PDD X7)가 강제한다. 서브에이전트에게 전달되는 프롬프트에서 `US-<DOMAIN>-<NNN>` 형식의 시나리오 ID를 추출해 중복을 제거하고(같은 ID 반복은 한 번으로 셈) 고유 개수를 센다. 여러 도메인이 섞이면 합산하고, 시나리오 ID가 아닌 다른 텍스트는 무시한다. 고유 개수가 30 이하면 통과, 31 이상이면 차단한다.

이 X7 검사의 기본 모드는 strict(자동 강제 차단)이다 — 경고만 하던 과거 동작이 30개 천장을 강제하지 못한다는 검증 결과를 반영해 기본값을 강화했다. 데몬이 떠 있지 않으면 검사는 우아하게 건너뛴다. 검증 루프 스킬(`skills/clawket-verify-loop/SKILL.md`)이 배치 디스패치 시 이 규칙을 전제로 작업을 분할한다. 훅(에이전트 디스패치 시점)과 검증 워크플로우에 걸쳐 작동하므로 경계를 가로지른다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(이 규칙이 도입된 결정 기록) 연결 필요. 코드 주석상 PDD A8 / US-CKT-HOOK 맥락으로 추정되나 결정 노드 미확정.
