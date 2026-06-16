---
id: component.landing-site
kind: SystemComponent
title: Clawket 랜딩 사이트 (@clawket/landing)
definition: Clawket 의 공개 마케팅·문서 사이트. Vite + React 19 단일 페이지 앱(SPA)으로 빌드되어 Vercel 에 정적 배포되는, 본체와 분리된 독립 git 서브레포다.
purpose: 잠재 사용자에게 Clawket 이 무엇인지·왜 쓰는지·어떻게 설치하는지를 보여 주고, 실제 대시보드 데모 영상과 문서로 첫 접점을 만든다.
realizedBy:
  - capability.watch-demo
  - capability.read-docs
  - capability.copy-install-command
  - capability.switch-locale
  - capability.share-social-card
implementedIn:
  - landing/
  - landing/src/main.tsx
  - landing/package.json
dependsOn:
  - component.spa-router
  - component.i18n
consumesApi:
  - integration.compliance-evals-feed
providesApi: []
integratesWith:
  - integration.vercel
  - integration.i18n-sync-ci
impacts:
  - persona.human-operator
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:systemcomponent"]
relatesTo:
  - to: component.web-dashboard
    type: reads
    note: 홈의 hero 데모 영상이 실제 웹 대시보드 6개 화면을 녹화해 만든 것이라, 랜딩이 보여 주는 제품 표면은 곧 웹 대시보드다.
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지. -->

## 책임

이 컴포넌트는 Clawket 의 "바깥쪽 얼굴"이다. 제품을 처음 만나는 방문자가 보는 마케팅 홈 화면, 문서 허브, 설치 안내, 그리고 실제 대시보드 동작을 보여 주는 데모 영상까지를 한 사이트에 담는다. 제품 본체(CLI·데몬·웹 대시보드)와는 코드도 배포도 분리되어 있으며, 별도의 독립 git 저장소로 관리된다.

핵심 책임은 세 가지다. 첫째, 제품 가치를 전달하는 랜딩 홈과 4개 섹션 문서를 렌더링한다. 둘째, 방문자가 곧바로 설치 명령을 실행할 수 있도록 진입 경로를 제공한다. 셋째, 20개 언어로 카피를 현지화해 전 세계 방문자가 모국어로 읽을 수 있게 한다.

이 사이트는 제품의 기능 정의나 아키텍처 정본을 스스로 들고 있지 않다. 그 정본은 본체 저장소의 README·문서이며, 랜딩은 그것을 사용자 친화적으로 옮겨 보여 주는 표면일 뿐이다. 기능 주장은 본체 정본과 동기화되어야 한다.

## 경계와 의존

런타임은 브라우저에서 도는 정적 SPA다. 서버 사이드 렌더링 없이 단일 HTML 진입점을 띄우고, 그 안에서 클라이언트 라우터가 화면을 전환한다. 화면 전환과 라우팅은 별도 라우팅 셸 컴포넌트가 맡고, 화면에 표시되는 모든 카피와 언어 전환은 자체 i18n 엔진이 공급한다.

외부 데이터 의존은 매우 얕다. 컴플라이언스 배지 위젯이 외부 평가 피드를 가져오도록 설계되어 있으나 현재 어느 화면에도 연결되어 있지 않다. 그 외에는 자기 자신이 들고 있는 정적 자산(데모 영상, OG 이미지, 설치 스크립트)만으로 동작한다.

배포는 Vercel 이 담당한다. 기본 브랜치에 푸시가 들어오면 Vercel 이 빌드해 정적 산출물을 발행하고, 응답 헤더와 SPA fallback 규칙은 저장소의 배포 설정 파일이 정본이다.

## 통신 패턴

방문자 브라우저 ↔ 정적 호스팅(Vercel) 사이의 HTTP 요청·응답이 전부다. 동적 백엔드 API 호출은 없다. 데모 영상은 미리 녹화된 파일을 직접 내려받아 재생하고, 소셜 카드용 메타데이터는 각 라우트가 런타임에 문서 헤드를 갱신해 채운다.

## 하위 서브패키지 (책임 단위)

라우팅 셸(클라이언트 라우터), 자체 i18n 엔진, 그리고 빌드 산출물에 들어가지 않는 보조 파이프라인 두 개(데모 영상 캡처, OG 이미지 렌더)로 구성된다. 보조 파이프라인은 사이트가 동작하는 데 필요한 정적 자산을 만들어 두는 도구일 뿐, 빌드·런타임 경로의 일부가 아니다.

## 미확정 (OPEN)
- [ ] OPEN: 커스텀 도메인 없이 Vercel 할당 alias 로 서빙 중 — 정식 도메인 확정 여부 미정.
