---
id: endpoint.asset-og-image
kind: Endpoint
title: 소셜 카드 이미지(og-image.png)
definition: 빌드 단계에서 HTML 템플릿을 1200×630 PNG로 렌더링해 생성하는 소셜 미리보기(OG) 이미지 자산으로, 페이지 메타의 og:image로 참조된다.
realizedBy:
  - component.landing-site
implementedIn:
  - landing/public/og-image.png
  - landing/scripts/og/render-og.mjs
relatesTo:
  - to: endpoint.route-home
    type: depends-on
    note: 홈을 비롯한 페이지 메타가 소셜 공유 미리보기로 이 이미지를 가리킨다.
governedBy: []
impacts: []
consumedBy:
  - screen.home
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

링크를 슬랙·메신저·SNS 등에 공유할 때 표시되는 소셜 미리보기 카드 이미지다. 1200×630 크기의 PNG이며, 페이지의 OpenGraph 메타(og:image)가 이 자산을 가리킨다. 손으로 그린 이미지가 아니라 OG 렌더 파이프라인이 HTML 템플릿을 헤드리스 브라우저로 캡처해 생성한 산출물이다.

대상은 공유 링크를 보는 모든 사람으로, 제품의 첫인상을 결정하는 소셜 자산이다.

## 요청 / 응답

HTTP로 단일 정적 PNG 파일을 반환한다. 생성 측면에서는 빌드/스크립트 단계에서 OG 템플릿 HTML을 1200×630 뷰포트의 헤드리스 브라우저로 열어 폰트 로딩 완료 후 화면을 캡처해 PNG로 저장한다. 결과 파일이 공개 자산으로 배포되어 런타임에 정적으로 서빙된다.

## 권한 / 제약

공개 정적 자산으로 누구나 접근한다. 파일명에 `og-image` 토큰이 들어가 사이트의 SPA rewrite 예외에 포함되므로 실제 이미지가 그대로 서빙된다. 이미지 갱신은 코드가 아니라 렌더 스크립트를 다시 실행해 PNG를 재생성하는 방식으로 이뤄진다(런타임 동적 생성 아님).

## provenance

이미지 파일은 `landing/public/og-image.png`, 생성 스크립트는 `landing/scripts/og/render-og.mjs`(짝이 되는 템플릿은 같은 디렉터리의 og-template.html)다.

## 미확정 (OPEN)
- [ ] OPEN: 렌더 스크립트가 CI/빌드에서 자동 실행되는지, 아니면 수동 갱신인지 owner 확인 필요.
