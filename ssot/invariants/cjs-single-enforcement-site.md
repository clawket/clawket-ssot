---
id: invariant.cjs-single-enforcement-site
kind: Invariant
title: 각 게이트는 정확히 한 곳에서만 강제된다
definition: "cjs 훅 핸들러는 '도구 호출을 진행해도 되는가'를 결정하고 데몬은 '상태 변이를 영속시켜도 되는가'를 결정하며, 각 게이트(활성 태스크·파괴적 명령·PDD 검사·티어·플랜 strict·evidence·DB 불변식)는 둘 중 정확히 한 곳에서만 강제되어 중복이 없다."
governs:
  - domain.hook-enforcement
  - component.claude-adapter
  - component.daemon
  - endpoint.hook-pre-tool-use
  - endpoint.hook-post-tool-use
  - endpoint.daemon-plans-import-strict
  - concept.evidence-required
governs_note: "cjs 층과 데몬 층은 중복이 아니라 상보적이다 — 전자는 호출 허용 여부를, 후자는 변이 영속 여부를 책임진다."
implementedIn:
  - clawket/docs/HOOK_ENFORCEMENT.md
  - clawket/adapters/shared/claude-hooks.cjs
decidedBy: []
crossesBoundary: true
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:invariant
---

## 제약

Clawket의 가드레일은 두 층에서 강제되지만, 각 규칙은 두 층 중 정확히 한 곳에서만 검사된다 — 절대 양쪽에서 중복으로 검사하지 않는다. 역할 분담의 기준은 단순하다.

- **cjs 훅 핸들러**는 "이 도구 호출을 진행시켜도 되는가"를 결정한다. 여기서 강제되는 것은 활성 태스크 게이트(작업 컨텍스트 없이 변경 도구 사용 차단), 파괴적 명령 차단, PDD 검사(시나리오 ID·배치 크기·근거·동기화 추론), 티어 게이트(검사 결과를 기록), 그리고 Plan Mode 종료 시 strict 플랜 검증이다. cjs는 이 판단을 위해 데몬에 읽기 전용 질의만 던진다.
- **데몬**은 "이 상태 변이를 영속시켜도 되는가"를 결정한다. 여기서 강제되는 것은 태스크 완료에 근거(evidence)를 요구하는 규칙, 그리고 외래키·"하나만 활성"·유닛 정합성 같은 DB 차원의 불변식이다.

핵심은 두 층이 중복이 아니라 상보적이라는 점이다. 같은 규칙을 양쪽이 따로 검사하지 않으므로, 한 규칙의 정의가 바뀔 때 고쳐야 할 자리도 한 곳뿐이다.

## 깨지면 무슨 일이 일어나나

한 게이트가 두 곳에서 강제되기 시작하면 두 정의가 서서히 어긋나(drift) 한쪽은 막고 다른 쪽은 통과시키는 모순이 생긴다. 그러면 "왜 같은 동작이 어떤 경로에서는 차단되고 어떤 경로에서는 허용되는가"를 추적하기 어려워지고, 규칙을 바꿀 때 한 곳만 고쳐 나머지가 낡은 채 남는 사고가 난다. 단일 강제 지점(single enforcement site) 원칙은 이런 모순과 유지보수 사고를 구조적으로 막기 위한 것이다.

## 코드에서 어떻게 강제되나

이 역할 분담은 강제 레이아웃 문서(HOOK_ENFORCEMENT.md)에 게이트별 "어디서 강제되는가 / 데몬의 역할"이 표로 명시되어 있고, 실제로 cjs 공유 핸들러가 호출 허용 판단(활성 태스크·파괴적 명령·PDD·티어·플랜 strict)을 모두 한 파일 안의 핸들러들에서 수행하며, 데몬은 evidence 요구와 DB 제약을 별도로 떠맡는다. 이 배치는 회귀 테스트들로 고정되어 있어, 강제 지점이 의도치 않게 데몬으로 넘어가거나 중복되는 변경을 잡아낸다. 한때 단일 MCP 도구로 강제를 일원화하는 대안이 검토되었으나, 게이트가 이미 각자 한 집을 가지고 있어 중복 문제가 실재하지 않는다는 판단으로 채택되지 않았다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(cjs/데몬 단일 강제 지점 분담을 정한 근거 결정) 연결 필요
