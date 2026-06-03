---
id: concept.site-origin
kind: Concept
title: SITE_ORIGIN (정본 호스트)
definition: 랜딩 사이트의 모든 정본 주소(canonical)와 소셜 공유 주소(og:url)를 만들 때 기준이 되는 단 하나의 프로덕션 호스트 상수.
relatesTo:
  - to: concept.head-config
    type: relates-to
    note: HeadConfig 의 canonical·og:url 호스트가 이 상수에서 나온다.
  - to: integration.vercel
    type: relates-to
    note: 값은 Vercel 프로덕션 별칭 도메인을 가리킨다.
  - to: component.landing-site
    type: belongs-to
    note: 랜딩 사이트가 자기를 가리키는 정본 호스트 정의다.
governedBy: []
governs: []
dependsOn: []
implementedIn:
  - landing/src/hooks/useHead.ts
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:concept"]
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 식별자는 provenance/근거에만. -->

## 정의

SITE_ORIGIN 은 랜딩 사이트가 "이 사이트의 진짜 주소는 여기"라고 선언할 때 쓰는 단 하나의 기준 호스트다. 사이트의 모든 [HeadConfig](head-config.md) 가 정본 주소(canonical)와 소셜 공유 주소(og:url)를 만들 때, 경로 앞에 붙이는 호스트 부분으로 이 상수를 사용한다. 예를 들어 홈은 이 호스트 + "/", 문서 허브는 이 호스트 + "/docs" 식으로 페이지별 정본 주소가 구성된다.

값은 Clawket 랜딩의 프로덕션 별칭 도메인(Vercel 에 배포된 `landing-seungwoo321s-projects.vercel.app`)을 가리킨다 — [integration.vercel](../integrations/vercel.md) 참조. 이렇게 호스트를 한 곳에 상수로 모아 두는 이유는 정본화(canonicalization)다. 같은 콘텐츠가 미리보기 배포·여러 별칭 도메인 등 여러 주소에서 보일 수 있을 때, 검색엔진과 소셜 카드가 항상 "대표 주소 하나"를 가리키게 해 중복 색인·잘못된 공유 미리보기를 막는다.

이웃 개념: [concept.head-config](head-config.md)(per-route 메타)가 canonical·og:url 의 호스트로 이 값을 쓰며(HeadConfig 가 이 상수에 의존), [component.landing-site](../components/landing-site.md) 랜딩 사이트가 자기를 가리키는 정본 호스트 정의다.

## 엔티티 (DB)

데이터베이스 엔티티가 아니다. 단일 문자열 상수이며 프런트엔드 소스에 박혀 빌드 시 번들에 포함된다. 런타임에 외부에서 받아오지 않는다.

## API 표면

서버 API 가 없다. 이 상수는 정본·소셜 주소 문자열을 조립하는 데만 쓰이며 자체적으로 어떤 호출도 만들지 않는다.

## 불변식

- 사이트 전역에서 정본·소셜 공유 주소의 호스트는 이 단 하나의 상수에서만 나온다(페이지마다 호스트를 따로 정하지 않는다 — single source of truth).
- 값은 프로덕션 별칭 도메인 하나를 가리키며, 미리보기·임시 배포 주소를 정본으로 노출하지 않는다.

## 구현 위치 (provenance)

정본 호스트 상수와 그것을 사용해 canonical·og:url 을 조립하는 head 동기화 로직은 함께 `landing/src/hooks/useHead.ts` 에 있다. 코드가 사라져도, "프로덕션 별칭 도메인 한 개를 모든 정본·소셜 주소의 호스트 기준으로 쓴다"는 규칙만으로 동작을 재현할 수 있다.

## 미확정 (OPEN)
- [ ] OPEN: 향후 커스텀 도메인 연결 시 이 상수 값의 갱신 주체·절차가 명시되지 않음(owner TBD).
