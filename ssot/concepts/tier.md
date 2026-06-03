---
id: concept.tier
kind: Concept
title: Tier(low/med/high)
definition: "태스크가 요구하는 모델 등급. 세 단계로 low(가벼운 Haiku급)·med(중간 Sonnet급)·high(고성능 Opus급)이며, 태스크마다 한 등급이 붙는다. 에이전트 스포너는 각 태스크를 그 등급을 충족하거나 넘는 모델로 라우팅한다. 현재(v3) 등급보다 낮은 모델을 쓰려 하면 막지 않고 경고만 한다(advisory)."
relatesTo:
  - to: concept.task
    type: relates-to
    note: 등급은 태스크에 붙는 속성으로, 태스크가 어느 수준의 모델을 요구하는지 나타낸다
  - to: integration.model-tier-routing
    type: relates-to
    note: 태스크 등급에 맞는 Claude 모델로 라우팅하는 연동이 이 개념을 소비한다
  - to: concept.tsv-evidence
    type: relates-to
    note: 검증 근거 행의 tier_used 칸이 어느 등급으로 판정했는지를 기록한다
governedBy: []
governs: []
dependsOn: []
implementedIn:
  - clawket/adapters/shared/claude-hooks.cjs
  - desktop/packages/ui/src/components/TierMark/TierMark.tsx
  - clawket/README.md
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:concept
---

<!-- 작성 고도: 자연어 우선. 코드 식별자는 provenance/근거에만. -->

## 정의

Tier(등급)는 "이 태스크는 얼마나 똑똑한 모델이 처리해야 하는가"를 세 단계로 표시하는 속성이다.

- **low**: 가벼운 작업 — Haiku급 모델로 충분.
- **med**: 중간 난도 — Sonnet급.
- **high**: 고난도 — Opus급(또는 그에 준하는 최신 고성능 모델).

태스크마다 등급 하나가 붙고, 에이전트 스포너는 각 태스크를 그 등급을 충족하거나 넘어서는 모델로 보낸다. 등급은 "최소 요구 수준"이라 상위 등급 모델은 하위 등급 작업을 처리할 수 있다(상위는 하위의 상위집합).

현재 v3에서 등급 강제는 **경고 수준(advisory)**이다. 즉 등급보다 낮은 모델로 작업하려 해도 차단하지 않고 경고만 띄운다. 문서상 v4 이상에서 하드 강제(hard-enforced)로 바뀔 예정이며, 그때 벤더 중립 어댑터 계층도 함께 도입된다(v3은 Claude 모델 계열만 대상).

검증 흐름의 근거 행에서는 등급이 "어느 수준으로 판정했나"를 기록하는 데도 쓰인다(→ concept.tsv-evidence). 다만 근거 행은 등급(low/med/high)이 아니라 모델명(haiku/sonnet/opus) 어휘를 쓰며, 두 어휘의 매핑은 아래 OPEN으로 남긴다.

## 엔티티 (DB)

등급은 태스크의 속성으로 보관된다(태스크가 가진 tier 필드, 값은 low/med/high). 검증 흐름에서는 판정에 실제 사용한 등급을 별도로 남기기도 한다(tier_used).

- 태스크 생성·수정 시 등급을 지정·변경할 수 있고, 태스크 목록을 등급으로 필터할 수 있다.
- 등급 필터는 표준 표기 low/med/high 외에 정책 문서에서 쓰는 별칭 G1/G2/G3(대소문자 무관)도 받는다 — G1=low, G2=med, G3=high에 대응한다.

## API 표면

- **등급 부여·변경**: 태스크 생성/수정에서 등급을 지정한다(태스크 명령의 `--tier low|med|high`). 등급 게이트가 모델 불일치를 알리면 모델을 올리거나 태스크 등급을 낮추라고 안내한다.
- **등급 필터 조회**: 태스크 목록을 등급으로 필터(low/med/high, 또는 별칭 G1/G2/G3).
- **등급 게이트(훅)**: 작업 도구 사용 직전 훅이 현재 모델과 태스크 등급을 비교한다. high 태스크는 Opus급 또는 최신 고성능(sonnet-4-5+/claude-4+) 모델을, med 태스크는 최소 Sonnet급 또는 haiku-3-5+를 요구한다. 불일치 시 v3에서는 경고/안내만 하고, 받아들인 등급을 후속 모델 라우팅·감사 로그가 읽을 수 있도록 환경값으로 노출한다.

## 불변식

- **세 단계 폐집합**: 등급 값은 low / med / high 셋뿐(별칭 G1/G2/G3 포함). 그 외 값은 받지 않는다.
- **상위는 하위의 상위집합**: high 모델은 med 작업을, med 모델은 low 작업을 처리할 수 있다(등급은 최소 요구 수준).
- **v3 경고 강제**: v3에서 등급-모델 불일치는 차단이 아니라 경고다. 하드 강제는 v4+ 예정.
- **Claude 계열 한정(v3)**: v3의 등급 라우팅 대상은 Claude 모델 계열뿐이다.

## 구현 위치 (provenance)

- 등급-모델 비교 게이트(high/med 판정 규칙, v3 경고·env 노출): `clawket/adapters/shared/claude-hooks.cjs`(Tier mismatch gate 부근).
- 등급 표기(low/med/high)와 v3=advisory / v4=hard-enforced 주석, UI 배지: `desktop/packages/ui/src/components/TierMark/TierMark.tsx`.
- 벤더 정책·등급 정의(low=Haiku급 / med=Sonnet급 / high=Opus급)와 v3 advisory→v4 hard-enforce 로드맵: `README.md`(Vendor Policy & Tier Routing 절).

## 미확정 (OPEN)
- [ ] OPEN: 등급(low/med/high)과 검증 근거의 모델명 어휘(haiku/sonnet/opus)가 매핑되는 정확한 지점·규칙 추적 미완(→ concept.tsv-evidence 와 교차).
- [ ] OPEN: 별칭 G1/G2/G3 가 태스크 필터 외에 어디까지(정책 문서·라우팅) 통용되는지 범위 확인 미완.
