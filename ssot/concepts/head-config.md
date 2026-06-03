---
id: concept.head-config
kind: Concept
title: HeadConfig (per-route 메타)
definition: 한 라우트(페이지)가 가져야 할 SEO·소셜 공유 메타데이터 묶음. 페이지 제목, 설명, 정본 주소(canonical), 공유 카드 주소(og:url)로 구성된다.
relatesTo:
  - to: concept.site-origin
    type: depends-on
    note: canonical·og:url 의 호스트 기준으로 SITE_ORIGIN 상수를 쓴다.
  - to: capability.share-social-card
    type: relates-to
    note: 소셜 카드 공유 기능이 이 메타 묶음을 라우트별로 설정한다.
  - to: component.spa-router
    type: relates-to
    note: 각 라우트 컴포넌트가 자기 메타를 설정해 SPA 내비게이션 시 head 를 갱신한다.
governedBy: []
governs: []
dependsOn:
  - concept.site-origin
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

HeadConfig 는 랜딩 사이트의 한 페이지(라우트)가 검색엔진과 소셜 공유에 대해 자기를 어떻게 소개할지를 담은 메타데이터 묶음이다. 네 가지 값으로 이뤄진다: (1) 페이지 제목, (2) 설명 문구, (3) 정본 주소(canonical — "이 페이지의 진짜 대표 URL은 여기"라고 검색엔진에 알리는 값), (4) 소셜 공유 카드 주소(og:url).

랜딩은 단일 페이지 앱(SPA)이라 페이지 간 이동이 브라우저 새 요청 없이 일어난다. 그래서 각 라우트 컴포넌트가 화면에 그려질 때 자기 HeadConfig 를 문서의 head 영역에 직접 반영한다. 구체적으로는 제목을 바꾸고, 설명·정본 링크·소셜 카드 관련 메타(og 제목·설명·주소, 트위터 제목·설명)를 해당 라우트 값으로 덮어쓴다. 라우트가 바뀌면 이 갱신이 다시 일어나 head 가 현재 페이지에 맞춰진다.

여기에는 의도적인 역할 분담이 있다: 자바스크립트를 실행하지 않는 크롤러를 위해, 빌드 산출물의 정적 HTML 파일은 기본 제목·설명·정본·소셜 메타 블록을 그대로 갖고 있다(비-JS 폴백). HeadConfig 는 그 위에서 "라우트별로 의미 있는 부분집합"만 런타임에 덮어쓰는 역할이다. 즉 정적 폴백은 항상 존재하고, HeadConfig 는 라우팅된 페이지에 맞춰 그 일부를 갱신한다.

이웃 개념: canonical·og:url 의 호스트 부분은 [concept.site-origin](site-origin.md)(정본 호스트 상수)에 의존한다. [capability.share-social-card](../capabilities/share-social-card.md)(소셜 카드 공유)가 이 메타 묶음을 라우트별로 설정하는 기능이고, [component.spa-router](../components/spa-router.md)(클라이언트 라우터)의 각 페이지가 자기 메타를 직접 반영한다.

## 엔티티 (DB)

데이터베이스 엔티티가 아니다. HeadConfig 는 각 라우트 컴포넌트가 코드에서 구성해 넘기는 값이며, 영속 저장되지 않는다. 브라우저 문서의 head 라는 DOM 영역에 런타임에 반영될 뿐이다.

## API 표면

서버 API 가 없다. head 갱신은 전부 클라이언트 측 DOM 조작이며, 외부 호출을 발생시키지 않는다.

## 불변식

- canonical·og:url 의 호스트는 항상 정본 호스트 상수([SITE_ORIGIN](site-origin.md))를 기준으로 구성된다 — 페이지가 임의의 호스트를 정본으로 선언하지 않는다.
- 정적 HTML 의 기본 메타 블록(비-JS 폴백)은 항상 존재해야 하며, HeadConfig 는 그것을 대체하지 않고 라우트별 부분집합만 덮어쓴다.
- 라우트가 바뀔 때마다 head 가 현재 페이지의 메타로 다시 동기화된다(이전 페이지 메타가 남지 않는다).

## 구현 위치 (provenance)

HeadConfig 의 형태(제목·설명·canonical·og:url 네 값)와 이를 문서 head 에 반영하는 동기화 로직은 `landing/src/hooks/useHead.ts` 에 있다. 각 라우트 컴포넌트(홈·문서 허브·문서 하위 페이지 등)는 이 훅을 호출해 자기 메타를 설정한다. 코드가 사라져도, "네 값 / SPA 라우트마다 head 부분집합 덮어쓰기 / 정적 폴백 유지 / 호스트는 SITE_ORIGIN 기준" 규칙만으로 동작을 재현할 수 있다.

## 미확정 (OPEN)
- [ ] OPEN: 라우트별 제목·설명 문구가 i18n 사전과 연동되는지, 영어 고정인지는 화면 노드(screen.*)에서 확정 필요.
