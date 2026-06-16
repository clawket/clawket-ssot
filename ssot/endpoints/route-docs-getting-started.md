---
id: endpoint.route-docs-getting-started
kind: Endpoint
title: 라우트 /docs/getting-started
definition: "GET /docs/getting-started — 문서 레이아웃 안에 '시작하기' 문서 페이지를 끼워 렌더하는 중첩 클라이언트 사이드 라우트. 플러그인 설치 → 프로젝트 등록 → 첫 태스크를 evidence로 마감하는 온보딩 흐름을 안내한다."
realizedBy: [component.spa-router]
implementedIn:
  - landing/src/App.tsx
relatesTo:
  - { to: endpoint.route-docs, type: depends-on, note: "/docs 부모 라우트의 중첩 자식. 부모 레이아웃 본문에 끼워 렌더된다." }
  - { to: component.landing-site, type: belongs-to, note: "랜딩 사이트 SPA 문서 영역의 한 페이지." }
governedBy: []
impacts: []
consumedBy: [screen.docs-hub]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

문서 영역의 "시작하기(Getting Started)" 페이지를 보여 주는 경로다. `/docs` 레이아웃(사이드바·헤더)을 그대로 두른 채, 본문 자리에 온보딩 안내를 채워 넣는다.

처음 도입하는 사용자가 따라가는 흐름을 순서대로 설명한다: 사전 준비물 → 플러그인 설치 → 프로젝트 등록 → 첫 태스크 만들기 → 완료 시 근거(evidence) 제출 → 그 사이 훅이 무엇을 했는지 요약. SaaS 계정·API 키·필수 클라우드 동기화가 없고 모든 상태가 로컬에 저장된다는 점, 첫 세션에 바이너리가 자동 설치된다는 점을 함께 안내한다.

## 요청 / 응답

- 들어오는 정보: 현재 경로가 `/docs/getting-started`라는 사실. 별도 질의어·식별자는 없다.
- 돌려주는 결과: 문서 레이아웃 안에 렌더된 "시작하기" 문서 본문. 설치 명령, 프로젝트 등록 방법, Project → Plan → Unit → Cycle → Task 순서로 첫 태스크를 만들고 시작하는 예시, 완료 시 evidence를 함께 제출하는 방법, 그리고 세션 동안 작동한 훅들의 요약을 담는다. 이 페이지는 자신의 제목·설명 등 메타 정보를 스스로 설정한다.

## 권한 / 제약

공개 경로로 인증이 필요 없다. `/docs` 부모 라우트의 중첩 자식이므로 부모가 깔아 둔 레이아웃 안에서만 렌더되며, 부모 없이 단독으로 동작하지 않는다.

## provenance

라우팅은 `landing/src/App.tsx`에서 `/docs` 아래 `getting-started` 자식 라우트로 선언된 것에서 확인된다. 페이지 본문(설치·등록·첫 태스크·evidence 흐름)은 해당 문서 페이지 컴포넌트(GettingStarted)에 있다. 매칭·렌더는 SPA 라우터(`component.spa-router`)가, 레이아웃은 문서 허브(`screen.docs-hub`)가 제공한다.

## 미확정 (OPEN)
- [ ] 없음 — 라우트 위치(중첩 자식)와 페이지 목적이 코드에서 명확히 확인됨.
