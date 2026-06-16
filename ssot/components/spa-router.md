---
id: component.spa-router
kind: SystemComponent
title: 클라이언트 라우터 (App.tsx)
definition: 브라우저 안에서 화면을 전환하는 클라이언트 라우팅 셸로, 홈(/)과 문서 허브(/docs) 및 그 아래 4개 중첩 문서 페이지를 매핑한다.
purpose: 서버 왕복 없이 URL 에 맞는 화면을 즉시 보여 주어, 정적 SPA 가 여러 페이지를 가진 사이트처럼 동작하게 한다.
realizedBy:
  - capability.read-docs
implementedIn:
  - landing/src/App.tsx
dependsOn:
  - component.landing-site
consumesApi: []
providesApi:
  - endpoint.route-home
  - endpoint.route-docs
  - endpoint.route-docs-getting-started
  - endpoint.route-docs-workflow
  - endpoint.route-docs-cli
  - endpoint.route-docs-architecture
integratesWith:
  - integration.vercel
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:systemcomponent"]
relatesTo:
  - to: screen.home
    type: reads
    note: 루트 경로가 홈 화면을 렌더한다.
  - to: screen.docs-hub
    type: reads
    note: /docs 경로가 문서 허브 레이아웃을 렌더하고 그 안에 자식 페이지를 끼워 넣는다.
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지. -->

## 책임

이 컴포넌트는 사이트의 길찾기 역할을 한다. 방문자가 보는 주소(URL)를 읽어, 그에 맞는 화면을 골라 띄운다. 새 페이지를 서버에서 다시 받아 오는 전통적 방식이 아니라, 브라우저 안에서 화면만 바꿔 끼우는 방식이라 전환이 즉각적이다.

다루는 경로는 둘로 나뉜다. 첫째는 홈(루트 경로)이고, 둘째는 문서 허브(/docs)와 그 아래 4개 문서 페이지(시작하기, 워크플로우, CLI 레퍼런스, 아키텍처)다. 문서 영역은 공통 레이아웃(사이드바 + 본문 영역)을 두고 그 안에 각 문서 페이지를 갈아 끼우는 중첩 구조다.

블로그 포스트와 튜토리얼처럼 수동으로 작성된 정적 HTML 페이지는 이 라우터가 다루지 않는다. 그 경로들은 SPA 의 화면 전환 대상에서 빠지고, 호스팅이 해당 파일을 직접 서빙한다.

## 경계와 의존

라우터는 사이트 셸 안에서 동작하며, 화면 자체(홈·문서 페이지들)는 별도의 화면 컴포넌트가 그린다. 라우터는 어떤 화면을 언제 보여 줄지만 결정한다.

브라우저 주소창 방식의 라우팅을 쓰기 때문에, 방문자가 깊은 경로로 직접 들어오거나 새로고침을 해도 화면이 깨지지 않으려면 호스팅 쪽 fallback 규칙이 필요하다. 정적 자산·블로그·튜토리얼·확장자 있는 경로를 제외한 나머지 요청을 모두 단일 진입점으로 돌려보내는 규칙이 배포 설정에 정의되어 있으며, 새 경로를 추가할 때는 이 규칙이 그 경로를 가로채는지 반드시 확인해야 한다.

## 통신 패턴

자체 네트워크 통신은 없다. 브라우저 히스토리를 조작해 화면을 바꾸고, 실제 페이지 요청은 호스팅의 fallback 규칙과 협력한다.

## 하위 서브패키지 (책임 단위)

라우트 정의 자체가 본체다. 별도 하위 모듈로 나뉘지 않고, 각 경로를 해당 화면 컴포넌트에 연결하는 매핑 테이블 한 곳으로 구성된다.

## 미확정 (OPEN)
- [ ] OPEN: 없음
