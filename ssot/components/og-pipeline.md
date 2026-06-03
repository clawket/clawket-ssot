---
id: component.og-pipeline
kind: SystemComponent
title: OG 이미지 렌더 파이프라인 (scripts/og)
definition: 디자인된 HTML 템플릿을 Playwright 로 1200×630 크기로 캡처해 소셜 카드용 OG 이미지를 생성하는 보조 스크립트다.
purpose: 링크를 소셜 미디어·메신저에 공유할 때 깨지지 않는 일관된 미리보기 카드가 뜨도록 정적 OG 이미지를 만들어 둔다.
realizedBy:
  - capability.share-social-card
implementedIn:
  - landing/scripts/og/render-og.mjs
  - landing/scripts/og/og-template.html
  - landing/scripts/og/landing-smoke.mjs
dependsOn:
  - component.landing-site
consumesApi: []
providesApi:
  - endpoint.asset-og-image
integratesWith:
  - integration.playwright
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:systemcomponent"]
relatesTo:
  - to: capability.share-social-card
    type: reads
    note: 각 라우트가 설정하는 OG 메타데이터가 가리키는 카드 이미지를 이 파이프라인이 만든다.
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지. -->

## 책임

이 파이프라인은 소셜 공유용 미리보기 카드 이미지를 만든다. 디자인이 들어간 HTML 템플릿을 헤드리스 브라우저로 열어, 소셜 카드 표준 비율인 가로 1200·세로 630 픽셀로 화면을 캡처해 PNG 이미지로 저장한다. 글꼴이 다 로드된 뒤에 찍어 글자가 깨지지 않도록 한다.

이렇게 만든 이미지는 사이트의 정적 자산으로 들어가고, 각 페이지가 공유될 때 미리보기 카드로 사용된다. 페이지 쪽에서는 라우트마다 카드 제목·설명·이미지 주소 같은 메타데이터를 설정하는데, 그 이미지 주소가 가리키는 실체가 바로 이 파이프라인의 산출물이다.

함께 들어 있는 스모크 점검 스크립트는 별도 책임을 진다. 빌드된 사이트를 미리보기로 띄워, 홈의 데모 영상 요소가 포스터와 함께 제대로 붙었는지 자동 확인한다.

## 경계와 의존

OG 이미지 생성도 스모크 점검도 평소 빌드 경로의 일부가 아니다. 필요할 때 수동으로 돌리는 보조 도구다. 브라우저 자동화 도구는 형제 캡처 파이프라인의 설치본을 공유해 중복 설치를 피한다.

이 파이프라인이 만드는 산출물은 사이트가 정적으로 서빙하는 자산일 뿐, 동적 API 가 아니다. 따라서 빌드 결과물에 미리 포함된 채로 배포된다.

## 통신 패턴

헤드리스 브라우저가 로컬 템플릿 파일을 열어 캡처하고, 스모크 점검은 로컬 미리보기 서버에 접속해 화면을 검사한다. 외부 네트워크 통신은 없다.

## 하위 서브패키지 (책임 단위)

세 부분이다. OG 이미지의 모양을 정의하는 HTML 템플릿, 그 템플릿을 캡처해 이미지로 저장하는 렌더 스크립트, 그리고 빌드된 사이트의 데모 영상 요소가 정상인지 확인하는 스모크 점검 스크립트다.

## 미확정 (OPEN)
- [ ] OPEN: 라우트별 개별 OG 이미지 분기 여부 — 현재 단일 카드 이미지로 추정.
