---
id: integration.model-tier-routing
kind: Integration
title: Claude 모델 티어 라우팅
definition: Clawket v3가 Claude 모델 계열만을 대상으로, 각 태스크에 붙은 tier 라벨(low/med/high)에 맞는 등급의 모델을 쓰도록 라우팅하는 연동. 현재 세대에서 등급 미달(다운그레이드)은 경고 수준(advisory)이며 강제 차단은 다음 세대로 예정되어 있다.
purpose: 작업의 난이도에 모델 등급을 맞춰 품질과 비용을 함께 맞추기 위해 필요하다 — 단순 작업에 고가 모델을 낭비하지 않고, 어려운 작업에 저가 모델을 써서 품질이 무너지지 않도록 태스크별 등급 요구와 실제 모델을 대조한다.
integratesWith:
  - component.claude-adapter
  - component.daemon
implementedIn:
  - clawket/adapters/shared/claude-hooks.cjs
impacts:
  - concept.tier
  - endpoint.hook-pre-tool-use
  - persona.llm-coding-agent
relatesTo:
  - to: concept.tier
    type: depends-on
    note: 태스크에 부여된 tier 라벨(low/med/high)이 라우팅 판정의 입력
  - to: integration.claude-code-hooks
    type: relates-to
    note: 도구 사용 직전 훅에서 티어 적합성을 검사한다
governedBy:
  - invariant.claude-only-vendor
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:integration
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로 — 무엇을·왜·누가·어떤 규칙·무슨 데이터. 코드(테이블·필드·경로·SQL) 옮겨적기 금지(식별자는 provenance/근거에만). 코드 분기/의도 불명은 판정 말고 OPEN. -->

## 무엇과 연동하나

상대편은 Claude 모델 계열이다. Clawket v3는 의도적으로 Claude 한 벤더만을 대상으로 한다(벤더 무관 어댑터 계층은 다음 세대로 예정). 각 태스크는 세 등급 중 하나의 tier 라벨을 단다 — low(Haiku급)·med(Sonnet급)·high(Opus급). 에이전트 스포너는 그 태스크를 등급을 충족하거나 그 이상인 모델로 보낸다.

판정의 입력은 두 가지다. 하나는 태스크에 붙은 tier(데몬이 부여), 다른 하나는 현재 실행 중인 모델 식별자(Claude Code가 환경 변수로 알려주며, 별도 재정의 환경 변수로도 덮어쓸 수 있다). 모델 식별자는 이름에 담긴 단서(opus·sonnet·haiku와 세대 번호)로 등급이 분류된다.

규칙은 단순하다. low 태스크는 어떤 모델이든 허용한다. med 태스크는 적어도 med 이상(저가 모델 불가)을 요구한다. high 태스크는 high 등급만 허용한다. 등급이 맞지 않으면 거부 메시지로 "더 높은 등급 모델로 바꾸거나, 태스크 tier를 낮춰라"는 안내를 내고, 외부 하네스가 읽을 수 있도록 "지원되지 않는 환경"을 뜻하는 종료 코드를 함께 신호한다. 통과한 경우에는 실제로 인정된 등급을 후속 처리(라우팅·감사)가 읽을 수 있도록 환경 채널로 노출한다.

여기서 "다운그레이드는 경고(advisory)"라는 표현은, 현재 세대(v3)에서는 등급 미달에 대한 전면 차단이 아직 약하게(경고 위주로) 걸려 있고, 강제 차단은 벤더 무관 어댑터 계층과 함께 다음 세대로 예정되어 있다는 의미다.

## 구현 위치 (provenance)

티어 적합성 검사 로직은 공유 핸들러(`clawket/adapters/shared/claude-hooks.cjs`)의 도구 사용 직전 처리 안에 있다. 여기서 태스크 tier와 현재 모델 식별자를 읽어 등급을 분류하고, 불일치 시 구조화된 거부를 내며, 통과 시 인정 등급을 환경 채널로 surface 한다.

태스크에 tier를 부여하는 쪽은 데몬(`component.daemon`)이고, 검사를 수행하는 쪽은 Claude 어댑터(`component.claude-adapter`)다. tier 개념 자체의 정의는 별도 개념 노드(`concept.tier`)에 있다.

벤더를 Claude로 한정한다는 정책 근거는 제품 README의 벤더 정책·티어 라우팅 절에 명시되어 있다.

## 불변식

- v3는 Claude 모델 계열만을 대상으로 한다(`invariant.claude-only-vendor`). 다른 벤더 모델은 이 세대의 라우팅 대상이 아니다.
- 등급 미달에 대한 현재 처리는 advisory(경고 위주)이며, 강제 차단은 다음 세대로 예정된 변경이다 — 따라서 이 노드는 "현재 동작"과 "예정"을 구분해 기술한다.

## 영향 범위

라우팅 판정은 도구 사용 직전 훅(`endpoint.hook-pre-tool-use`) 위에서 작동하며, 입력이 되는 것은 tier 개념(`concept.tier`)이다. 직접 영향을 받는 주체는 코딩 에이전트(`persona.llm-coding-agent`)로, 어떤 모델로 작업할 수 있는지가 이 라우팅에 좌우된다. 벤더 한정 정책이 풀려 다중 벤더를 지원하게 되면(다음 세대), 이 연동의 분류·강제 규칙이 함께 재설계 대상이 된다.

## 미확정 (OPEN)
- [ ] OPEN: "advisory(경고)"와 "hard-block(강제 차단)"의 현재 경계 — 어떤 등급 조합에서 실제로 거부까지 가고 어떤 경우 통과시키며 경고만 남기는지의 정밀 매트릭스는 endpoint.hook-pre-tool-use 노드에서 확정 필요.
