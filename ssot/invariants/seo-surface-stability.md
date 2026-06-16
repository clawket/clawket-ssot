---
id: invariant.seo-surface-stability
kind: Invariant
title: SEO·소셜 카드 표면 안정성
definition: "검색 노출과 소셜 카드를 결정하는 메타 표면(엔트리 HTML 의 제목·설명·정본 주소·OG·Twitter 세트, 그리고 블로그·튜토리얼 정적 HTML 의 같은 세트)이 빈 값·임시 placeholder·복사-붙여넣기 잔재·잘못된 주소로 회귀하지 않아야 한다."
governs:
  - capability.share-social-card
  - concept.head-config
  - concept.site-origin
  - screen.home
  - screen.blog-post
  - screen.tutorial
governs_note: ""
implementedIn:
  - landing/index.html
  - landing/src/hooks/useHead.ts
  - landing/public/blog/2026-04-mcp-integrated.html
  - landing/public/tutorial/index.html
  - landing/.claude/rules/seo-surface-stability.md
decidedBy:
  - decision.csr-spa-no-ssr
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:invariant"]
---

## 제약

검색 엔진과 소셜 카드가 읽는 "메타 표면"은 항상 채워져 있고 서로 정합해야 한다. 구체적으로 엔트리 HTML 의 제목·설명·정본 주소(canonical)·OpenGraph·Twitter 카드 세트, 그리고 수동으로 만든 블로그·튜토리얼 정적 페이지의 동일한 7개 필드 세트가 대상이다. 이 값들은 빈 문자열이나 "TODO" 같은 임시값으로 남아서는 안 되고, 각 페이지의 정본 주소·OG 주소가 실제로 그 페이지를 가리켜야 한다(다른 페이지를 가리키는 복사-붙여넣기 잔재 금지).

정본 주소가 기준으로 삼는 호스트는 사이트의 공개 production 별칭 한 곳으로 고정되며, 이 호스트가 바뀌면 모든 정본·OG 주소·푸터·마케팅 카피를 한 번에 일괄 치환해야 한다. 새 정적 페이지를 추가할 때도 이 7개 필드 세트를 모두 채우고 각 필드가 새 주소를 가리키는지 확인해야 한다.

## 깨지면 무슨 일이 일어나나

제목·설명이 빈 값이나 placeholder 로 회귀하면 검색 결과의 페이지 요약이 사라진다. 블로그·튜토리얼을 복사-붙여넣기로 만들면서 주소·제목 중 일부만 갱신하면, 소셜 카드가 잘못된 페이지를 가리켜 링크 공유 시 엉뚱한 카드가 렌더된다. 정본 호스트가 바뀌었는데 일괄 치환을 빠뜨리면, 더는 존재하지 않거나 접근이 막힌(인증 보호된) 주소를 정본으로 박아 검색·공유가 깨진다. 이 사이트는 클라이언트 렌더 단일 페이지 앱이라 메타 표면이 곧 production 의 사용자·크롤러가 보는 유일한 표면이므로, 잔재 하나가 곧바로 production 사고로 직결된다.

## 코드에서 어떻게 강제되나

엔트리 HTML 은 크롤러용 비-JS fallback 으로 제목·설명·정본·OG·Twitter 세트를 직접 보유하고, 라우팅된 각 페이지는 런타임에 head 동기화 훅(`landing/src/hooks/useHead.ts`)이 SEO 서브셋을 덮어쓴다. 이 훅은 정본 호스트 상수를 한 곳에서 export 해 모든 정본·OG 주소의 기준 호스트가 된다. 블로그·튜토리얼은 각자 7필드 세트를 가진 정적 HTML 이다. 의도는 작성 규칙 문서(`landing/.claude/rules/seo-surface-stability.md`)가 보존한다.

자동 강제 장치는 없다. 정본 주소와 OG 주소·제목이 서로 일치하는지, 정본 호스트가 실제 production 별칭과 같은지 검사하는 자동 lint/CI 가 없고, 빌드는 빈 제목이어도 통과한다. 따라서 정합성 보존은 작성자의 수동 검증 절차(새 페이지 추가·호스트 변경 시 전수 grep 과 머지 후 소셜 카드 디버거 확인)에 의존한다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy 는 SSR 없이 CSR SPA + 메타 fallback 으로 SEO 를 충족하기로 한 결정에 연결했다. 별도의 "SEO 표면 보존" 전용 결정이 필요하면 추가 연결.
