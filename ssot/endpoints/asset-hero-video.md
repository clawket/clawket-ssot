---
id: endpoint.asset-hero-video
kind: Endpoint
title: 히어로 데모 비디오 자산(hero.webm/mp4 + poster)
definition: 홈 화면의 데모 영역이 직접 참조하는 제품 데모 비디오(webm·mp4)와 재생 전 정지 프레임(hero-poster.png) 정적 자산이다.
realizedBy:
  - component.landing-site
implementedIn:
  - landing/public/hero.webm
  - landing/public/hero.mp4
  - landing/public/hero-poster.png
  - landing/src/pages/Home.tsx
relatesTo:
  - to: endpoint.route-home
    type: depends-on
    note: 홈 경로가 렌더링하는 화면의 데모 섹션이 이 자산을 직접 참조한다.
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

랜딩 사이트 홈 화면의 데모 섹션에서 자동 재생되는 제품 데모 영상 자산이다. 두 가지 포맷(webm·mp4)으로 제공되어 브라우저가 지원하는 쪽을 고르고, 영상이 로드되기 전에는 정지 프레임 이미지(hero-poster.png)가 표시된다. 영상은 자동 재생·무음·반복·인라인 재생으로 설정되어 방문자가 클릭 없이 제품의 동작을 바로 본다.

대상은 홈에 처음 들어온 방문자로, 제품 흐름을 한눈에 보여주는 첫인상 자산이다.

## 요청 / 응답

세 개의 정적 미디어 파일을 HTTP로 직접 반환한다 — webm 비디오, mp4 비디오, png 포스터 이미지. 홈 화면의 비디오 요소가 포스터로 png를, 소스로 webm·mp4를 순서대로 참조한다. 메타데이터만 선로딩하도록 설정되어 초기 트래픽을 줄인다.

## 권한 / 제약

공개 정적 자산으로 누구나 접근한다. 파일명에 `hero` 토큰이 들어가 사이트의 SPA rewrite 예외에 포함되므로 실제 미디어 파일이 그대로 서빙된다. 영상은 무음 자동 재생을 전제로 하며(브라우저 자동재생 정책상 무음이 필요), 접근성을 위해 설명 레이블이 부여된다.

## provenance

미디어 파일은 `landing/public/hero.webm`·`landing/public/hero.mp4`·`landing/public/hero-poster.png` 이고, 이를 참조하는 비디오 요소는 `landing/src/pages/Home.tsx` 의 데모 섹션에 있다.

## 미확정 (OPEN)
- [ ] OPEN: 영상 컨텐츠의 갱신 주기·생성 파이프라인(캡처 스크립트 연동 여부)은 owner 확인 필요.
