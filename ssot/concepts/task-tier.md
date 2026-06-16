---
id: concept.task-tier
kind: Concept
title: 태스크 tier / 실제 실행 tier
definition: "태스크가 요구하는 모델 등급(선언 tier)과 에이전트가 실제로 실행한 등급(tier_used)을 함께 기록하는 한 쌍의 속성. 둘이 다르면 그 차이를 설명하는 escalation_reason 이 반드시 따라붙어야 한다."
relatesTo:
  - { to: concept.tier, type: relates-to, note: "tier(low/med/high) 개념을 태스크 한 건에 실제로 적용·기록하는 형태" }
  - { to: concept.task, type: belongs-to, note: "태스크 레코드에 붙는 속성들" }
governedBy: []
implementedIn:
  - daemon/migrations/011_tier_column.sql
  - daemon/migrations/021_tier_used.sql
  - daemon/src/models.rs
  - daemon/src/routes/tasks.rs
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:concept"]
---

<!-- 작성 고도: 자연어 우선. 식별자는 provenance/근거에만. -->

## 정의

태스크에는 두 종류의 모델 등급이 기록된다.

- **선언 tier** — 이 태스크가 "원래 어느 등급의 모델로 처리되어야 하는가"를 나타내는 기본값이다. 보통 실행 봉투나 정책에서 정해져 내려오며, 값은 낮음(low)·중간(med)·높음(high) 세 단계 중 하나다. 비워둘 수도 있다(미지정).
- **실제 실행 tier(tier_used)** — 에이전트가 그 태스크를 "실제로 어느 등급의 모델로 처리했는가"이다. 작업을 진행하는 에이전트가 태스크를 갱신할 때 직접 적어 넣는다.

핵심 규칙은 둘이 일치해야 한다는 것이 아니라, **둘이 다를 때는 왜 달랐는지를 기록하도록 강제한다**는 점이다. 선언 tier 와 실제 실행 tier 가 어긋나는 순간, 그 차이를 설명하는 짧은 사유(escalation_reason)가 같이 제출되지 않으면 갱신 자체가 거절된다. 이렇게 하면 "중간 등급으로 충분하다고 봤는데 실제로는 높은 등급을 써야 했다" 같은 등급 상향(에스컬레이션)이 침묵 속에 묻히지 않고, 나중에 등급 추정이 얼마나 정확했는지·어떤 작업이 자꾸 상향되는지를 돌아볼 수 있는 근거가 남는다.

이 개념은 등급 그 자체(낮음/중간/높음의 의미)인 tier 를 태스크 한 건의 실제 이력으로 구체화한 것이며, 태스크 레코드의 일부로 보관된다.

## 엔티티 (DB)

태스크 레코드에 세 개의 관련 항목이 함께 보관된다.

- **선언 tier**: 낮음·중간·높음 중 하나만 허용되며, 그 외 값은 저장 단계에서 막힌다. 기본은 "지정 안 됨"이다.
- **실제 실행 tier**: 에이전트가 갱신 시 적는 값. 자유롭게 비울 수 있다.
- **에스컬레이션 사유**: 실제 실행 tier 가 선언 tier 와 다를 때 채워지는 설명 문구. 평소에는 비어 있다.

선언 tier 컬럼은 비교적 이른 시점(스키마 11)에 도입되었고, 실제 실행 tier 와 에스컬레이션 사유는 이후(스키마 21)에 함께 추가되었다.

## API 표면

태스크 갱신 경로에서 이 한 쌍이 관리된다. 갱신 요청에 실제 실행 tier 가 담겨 있으면, 데몬은 그 값을 같은 요청 안의 선언 tier(없으면 현재 저장된 선언 tier)와 비교한다. 둘이 다른데 에스컬레이션 사유가 비어 있으면, 요청은 "에스컬레이션 사유 필요"라는 코드와 함께 거절된다. 사유가 채워져 있으면 실제 실행 tier 와 사유가 함께 태스크에 반영된다. 태스크 등급 분포·에스컬레이션 비율은 진단 명령에서 점검 대상이 된다.

## 불변식

- 실제 실행 tier 가 선언 tier 와 다르면, 비어 있지 않은 에스컬레이션 사유가 같은 갱신에 반드시 동반되어야 한다(아니면 거절). 이 규칙은 별도 invariant 노드로 분리되어 있지 않고 갱신 검증에 직접 박혀 있다.
- 선언 tier 는 낮음·중간·높음 외의 값을 가질 수 없다.

## 구현 위치 (provenance)

- 선언 tier 컬럼 정의와 허용값 제약: `daemon/migrations/011_tier_column.sql`
- 실제 실행 tier·에스컬레이션 사유 컬럼: `daemon/migrations/021_tier_used.sql`
- 태스크 모델의 세 필드 정의: `daemon/src/models.rs`
- 갱신 시 "다르면 사유 필수" 검증: `daemon/src/routes/tasks.rs`

## 미확정 (OPEN)
- [ ] OPEN: 선언 tier 가 비어 있을 때(미지정) 실제 실행 tier 와의 비교가 빈 문자열 기준으로 이뤄지는데, 이 경우의 의도된 동작(미지정 = 모든 등급 허용인지, 사유 면제인지)이 정책으로 명문화되어 있는지 미확인.
- [ ] OPEN: 등급 상향이 현재 경고(advisory)인지 차단인지의 최종 정책은 모델 티어 라우팅 쪽에 걸려 있어 본 노드 범위 밖.
