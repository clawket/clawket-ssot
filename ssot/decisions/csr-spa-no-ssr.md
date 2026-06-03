---
id: decision.csr-spa-no-ssr
kind: Decision
title: 랜딩은 SSR 없이 CSR SPA로 두고 SEO는 정적 fallback + per-route 헤드로 충족
purpose: 공개 랜딩 페이지를 SSR(서버 렌더링) 프레임워크로 만들지, 단일 HTML 엔트리의 클라이언트 렌더 SPA로 만들지
definition: 랜딩을 Vite + React Router 기반의 단일 HTML 엔트리 CSR SPA로 운영하고, 검색·소셜 크롤러를 위한 SEO는 정적 index.html의 비-JS 메타 fallback과 라우트별 런타임 헤드 갱신(useHead)으로 충족한다.
relatesTo:
  - to: component.landing-site
    type: governs
    note: 랜딩 사이트의 렌더링·라우팅 방식을 이 결정이 규정한다.
  - to: invariant.seo-surface-stability
    type: relates-to
    note: SEO 표면을 두 곳(정적 fallback + per-route 헤드)으로 나눈 구조가 이 불변식의 보존 대상이다.
  - to: endpoint.route-home
    type: relates-to
    note: 홈 라우트가 클라이언트 라우팅으로 렌더되는 대표 경로.
  - to: endpoint.route-docs
    type: relates-to
    note: docs 레이아웃이 동일한 CSR SPA + per-route 헤드 패턴을 따른다.
  - to: integration.vercel
    type: relates-to
    note: Vercel의 SPA rewrites(정적 자산 외 전부 index.html fallback)가 이 결정의 배포 측 전제다.
supersedes: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:decision"]
---

> Decision은 append-only다. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 `supersedes`로 잇는다.

## 맥락 (Context)

Clawket의 공개 랜딩 페이지는 마케팅·온보딩 표면이라 검색 엔진과 소셜 카드(OpenGraph·Twitter) 노출이 중요하다. 일반적으로 SEO가 중요한 사이트는 서버 사이드 렌더링(SSR)이나 정적 사이트 생성(SSG)을 채택해, 자바스크립트를 실행하지 않는 크롤러도 완성된 HTML과 메타 태그를 받도록 한다.

그러나 이 랜딩은 홈 한 페이지와 docs 4페이지로 이루어진 소규모 다국어(20 로케일) 사이트이며, 제품 정본(기능 설명·아키텍처)은 별도 wrapper 레포의 README/문서가 가진다. 이 규모에서 SSR 프레임워크를 도입하면 빌드·배포 파이프라인과 런타임이 무거워진다.

## 결정 (Decision)

랜딩을 단일 HTML 엔트리(index.html)에서 클라이언트가 렌더하는 React Router 기반 SPA로 둔다. SEO는 다음 두 축으로 나눠 충족한다.

- 정적 `index.html`이 자바스크립트 없이도 읽히는 기본값으로 `<title>`·`meta description`·canonical 링크·OpenGraph 세트·Twitter 카드 세트를 보유한다. 이것이 크롤러용 비-JS fallback이다.
- 라우팅된 각 페이지는 런타임에 `useHead` 훅이 해당 페이지의 title·description·canonical·og:url·og:title·og:description·twitter title/description를 덮어쓴다. 모든 canonical/og:url의 정본 호스트는 한 곳(`SITE_ORIGIN` 상수)에서 나온다.

즉 "정적 fallback이 항상 유효한 기본값을 제공하고, 클라이언트 라우팅이 페이지별로 그 일부를 갱신한다"는 이중 구조다.

## 근거와 결과 (Consequences)

- 소규모 사이트 규모에 SSR/SSG 프레임워크는 과한 인프라다. CSR SPA는 빌드를 정적 자산 산출(dist/)로 단순하게 유지하고, Vercel은 SPA rewrites로 정적 자산 외 모든 경로를 index.html로 fallback해 클라이언트 라우팅에 넘긴다.
- 대가로 SEO 표면이 정적 fallback과 런타임 헤드 갱신 두 곳에 흩어진다. 그래서 두 곳의 메타·canonical·OG/Twitter URL을 깨뜨리지 않는 것이 별도 불변식(SEO 표면 안정성)으로 관리된다. 새 라우트를 추가할 때는 Vercel rewrites 제외 목록이 그 경로를 가로채지 않는지 확인해야 한다.
- sitemap.xml·robots.txt·구조화 데이터(JSON-LD)는 현재 부재이며 별도 계획으로 다룬다(OPEN — 본 결정의 범위 밖).
