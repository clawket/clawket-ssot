---
id: invariant.scenario-id-one-to-one
kind: Invariant
title: 모든 태스크는 정확히 하나의 시나리오에 매핑된다
definition: 검증 흐름의 모든 태스크는 US-<DOMAIN>-<NNN> 형식의 시나리오 ID 하나에 정확히 1:1로 매핑되어야 하며, strict 모드에서는 누락이나 형식 위반이 차단된다.
governs:
  - concept.task
  - concept.scenario
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

검증 워크플로우에 들어가는 태스크는 저마다 정확히 하나의 사용자 시나리오에 연결되어야 한다. 그 연결은 `US-<DOMAIN>-<NNN>` 형식의 시나리오 식별자(예: 도메인 코드와 일련번호로 이뤄진 ID)로 표현한다. 태스크에 시나리오 ID가 비어 있어서도 안 되고, 형식에 맞지 않는 ID가 붙어서도 안 된다.

이 "정확히 하나" 규칙은 검증의 단위를 명확히 하기 위한 것이다. 한 태스크가 어떤 시나리오를 검증하는지가 1:1로 정해져 있어야, 검증 결과(통과·결함·시나리오 오류)를 그 시나리오에 정확히 귀속시킬 수 있다.

## 깨지면 무슨 일이 일어나나

태스크에 시나리오 ID가 없으면, 그 태스크가 무엇을 확인하기 위한 것인지 검증 시스템이 알 수 없어 결과를 어디에도 귀속시키지 못한다. 형식이 어긋난 ID는 자동 집계·동기화 과정에서 잘못된 시나리오에 붙거나 누락되어, 검증 결과가 오염된다. 한 태스크가 여러 시나리오를 뭉뚱그려 다루면 "이 시나리오는 통과했는가"라는 질문에 깨끗이 답할 수 없게 되어, 수렴 판정 자체가 흐려진다.

규칙이 작동하면, 시나리오 ID가 없거나 형식이 틀린 태스크는 strict 모드에서 작업이 차단되어 먼저 ID를 바로잡도록 강제된다.

## 코드에서 어떻게 강제되나

Claude Code 훅의 공유 핸들러(`adapters/shared/claude-hooks.cjs`)에 있는 scenario_id 검사(PDD X3)가 이를 강제한다. 이 검사는 태스크에 시나리오 ID가 비었는지(NULL/누락), 그리고 ID가 있을 때 `US-<DOMAIN>-<NNN>` 형식 패턴에 맞는지를 본다. 형식이 틀리거나 ID가 없으면 "모든 태스크는 정확히 1개 시나리오에 매핑되어야 한다"는 안내와 함께 바로잡는 명령을 제시한다.

차단 여부는 강제 모드에 달려 있다. 이 X3 검사의 기본 모드는 경고(warn)로 — 기존 작업의 점진적 마이그레이션을 고려한 호환 설정 — 위반을 알리되 막지는 않는다. 그러나 검증 루프 스킬(`skills/clawket-verify-loop/SKILL.md`)이 구동하는 PDD 검증 흐름에서는 strict 모드로 운용되어, 누락·형식 위반이 실제로 차단된다. 또한 데몬이 떠 있지 않으면 검사는 우아하게 건너뛴다. 훅(에이전트 측)과 검증 루프(워크플로우)에 걸쳐 작동하므로 경계를 가로지른다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(이 규칙이 도입된 결정 기록) 연결 필요.
