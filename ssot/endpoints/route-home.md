---
id: endpoint.route-home
kind: Endpoint
title: 라우트 / (Home)
definition: "GET / — 랜딩 사이트의 루트 경로. 브라우저가 사이트 최상단에 들어왔을 때 홈(마케팅) 화면을 그려 주는 클라이언트 사이드 라우트다."
realizedBy: [component.spa-router]
implementedIn:
  - landing/src/App.tsx
relatesTo:
  - { to: screen.home, type: relates-to, note: "이 경로에 매칭되면 홈(랜딩) 화면을 렌더한다." }
  - { to: component.landing-site, type: belongs-to, note: "랜딩 사이트 SPA의 진입 경로." }
governedBy: []
impacts: []
consumedBy: [screen.home]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

방문자가 사이트 주소의 최상위(`/`)로 들어왔을 때 보게 되는 첫 화면을 결정하는 경로다. 서버가 매번 페이지를 만들어 보내는 전통적 방식이 아니라, 브라우저에서 동작하는 단일 페이지 앱(SPA)이 현재 주소를 보고 "지금은 홈을 보여줄 차례"라고 판단해 홈 화면을 그려 넣는다.

주로 검색·광고·직접 입력을 통해 처음 들어오는 잠재 사용자가 닿는 진입점이며, 여기서 제품 소개·데모 영상·문서로의 이동이 시작된다.

## 요청 / 응답

- 들어오는 정보: 브라우저가 요청한 경로가 정확히 사이트 루트(`/`)라는 사실 외에 별도 입력은 없다. 별도의 질의어나 식별자를 요구하지 않는다.
- 돌려주는 결과: 홈(랜딩) 화면 한 장. 제품 한 줄 소개·버전 표시·행동 유도(설치/문서 이동) 버튼·짧은 데모 영상·동작 단계·기능 요약·아키텍처 개요·푸터로 구성된 마케팅 페이지다. 화면에 쓰이는 문구는 다국어 사전에서 현재 로케일에 맞게 채워진다.

## 권한 / 제약

누구나 접근 가능한 공개 경로다. 로그인·계정·인증이 필요 없다. 정적 호스팅 환경에서 SPA로 서빙되므로, 다른 모든 경로와 마찬가지로 서버 측 리라이트로 같은 앱 진입점으로 모인 뒤 클라이언트에서 경로가 해석된다(설치 스크립트 같은 정적 자산 경로는 이 리라이트에서 제외된다).

## provenance

라우팅 정의는 `landing/src/App.tsx`의 `BrowserRouter` 라우트 테이블에서 `/`가 홈 화면 컴포넌트에 연결된 것으로 확인된다. 실제 라우트 매칭·렌더는 SPA 라우터 컴포넌트(`component.spa-router`)가 담당하고, 매칭 시 그려지는 화면은 홈 화면(`screen.home`)이다.

## 미확정 (OPEN)
- [ ] OPEN: 홈에서 제공하는 외부 이동 링크(설치·문서·저장소 등)의 최종 목록과 우선순위는 `screen.home`/Home.tsx 정독으로 확정.
