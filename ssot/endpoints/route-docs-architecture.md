---
id: endpoint.route-docs-architecture
kind: Endpoint
title: Architecture 문서 페이지 라우트
definition: 랜딩 사이트의 문서 허브 안에서 "아키텍처" 설명 페이지를 보여주는 중첩 경로(/docs/architecture)다.
realizedBy:
  - component.landing-site
implementedIn:
  - landing/src/App.tsx
  - landing/src/pages/docs/Architecture.tsx
relatesTo:
  - to: endpoint.route-docs
    type: depends-on
    note: 문서 허브(/docs) 레이아웃 라우트의 자식으로 렌더링된다.
  - to: screen.docs-architecture
    type: relates-to
    note: 이 경로가 표시하는 화면이 아키텍처 문서 페이지다.
governedBy: []
impacts: []
consumedBy:
  - screen.docs-architecture
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

랜딩 사이트(공식 소개·문서 웹사이트)의 문서 영역에서 "아키텍처" 설명을 담당하는 페이지 경로다. 방문자가 주소 `/docs/architecture` 로 접근하면 표시된다. 문서 허브(`/docs`)가 공통 레이아웃(사이드바·네비게이션 등)을 제공하는 부모 틀이고, 이 경로는 그 안에 끼워지는 자식 페이지로 동작한다. 즉 단독 페이지가 아니라 문서 허브의 한 갈래다.

대상은 Clawket을 처음 살펴보는 개발자·도입 검토자로, 제품의 내부 구성(플러그인 셸, Rust CLI·데몬, SQLite+벡터 저장, 온디바이스 임베딩, MCP 서버, 로컬 우선 동작)을 한 페이지에서 개관하고 싶을 때 들어온다.

## 요청 / 응답

이 경로는 데이터를 주고받는 API가 아니라 정적 단일 페이지 애플리케이션의 클라이언트 라우트다. 방문자는 경로(주소)만 전달하고, 응답으로 아키텍처 설명 화면 — 시스템 구성도, 계층별 기술 스택 표, XDG 경로 표, 경로 분리 불변식 설명, 로컬 RAG·MCP 연동·프라이버시 설명 — 을 받는다. 페이지 메타(제목·설명·정규 URL·OG URL)도 함께 설정되어 검색·소셜 공유에 쓰인다.

이 경로는 새로고침으로 직접 진입해도 동작하도록 사이트의 SPA rewrite 규칙에서 처리된다(존재하지 않는 정적 파일은 모두 단일 진입 문서로 재작성된다).

## 권한 / 제약

공개 페이지로 누구나 접근할 수 있고 인증이 없다. 부모 문서 허브 레이아웃 안에서만 의미를 가지며 단독 컴포넌트로는 노출되지 않는다. 문서 본문은 영어로 작성된다(랜딩 문서 영어 단일 언어 정책).

## provenance

라우트 정의는 `landing/src/App.tsx` 의 문서 허브 라우트(`/docs`) 하위 중첩 경로(`architecture`)에 선언되어 있고, 표시 컨텐츠는 `landing/src/pages/docs/Architecture.tsx` 가 렌더링한다. 제공 주체는 랜딩 사이트 컴포넌트다.

## 미확정 (OPEN)
- [ ] OPEN: 문서 본문 내용은 현재 코드 기준 정적 텍스트다. 향후 i18n·동적화 여부는 owner 확인 필요.
