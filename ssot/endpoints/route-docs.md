---
id: endpoint.route-docs
kind: Endpoint
title: 라우트 /docs (허브)
definition: "GET /docs — 문서 영역의 레이아웃과 허브(개요) 화면을 그려 주는 클라이언트 사이드 라우트. 사이드바 내비게이션을 두르고, 정확히 /docs일 때는 개요를, 하위 경로일 때는 해당 문서 페이지를 안쪽에 끼워 보여준다."
realizedBy: [component.spa-router]
implementedIn:
  - landing/src/App.tsx
  - landing/src/pages/Docs.tsx
relatesTo:
  - { to: screen.docs-hub, type: relates-to, note: "문서 레이아웃 + 허브(개요) 화면을 렌더한다." }
  - { to: endpoint.route-docs-getting-started, type: depends-on, note: "하위 중첩 경로. 이 레이아웃 안쪽에 끼워 렌더된다." }
  - { to: endpoint.route-docs-workflow, type: depends-on, note: "하위 중첩 경로." }
  - { to: endpoint.route-docs-cli, type: depends-on, note: "하위 중첩 경로." }
  - { to: component.landing-site, type: belongs-to, note: "랜딩 사이트 SPA의 문서 영역 진입 경로." }
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

문서 영역 전체의 골격을 담당하는 경로다. 좌측 사이드바 내비게이션(개요 + 4개 섹션 링크)과 본문 자리를 둔 레이아웃을 깔고, 그 본문 자리에 무엇을 넣을지를 현재 주소로 판단한다.

- 주소가 정확히 `/docs`(혹은 `/docs/`)이면 본문 자리에 문서 개요(허브)를 직접 그린다. 허브에는 제품 한 줄 정의, 각 문서 섹션으로 가는 카드, 그리고 진실의 출처(원본 저장소·핵심 문서 링크) 안내가 담긴다.
- 주소가 하위 경로이면(예: `/docs/workflow`) 같은 레이아웃을 유지한 채 본문 자리에 해당 하위 문서 페이지를 끼워 넣는다.

문서를 찾아 들어온 사용자가 설치·워크플로우·CLI·아키텍처 사이를 사이드바로 오가는 허브 역할을 한다.

## 요청 / 응답

- 들어오는 정보: 현재 브라우저 경로. `/docs`인지 하위 경로인지에 따라 본문에 그릴 대상이 달라진다. 별도 질의어·식별자는 없다.
- 돌려주는 결과: 사이드바를 두른 문서 레이아웃. `/docs`에서는 개요(섹션 카드 + 출처 안내), 하위 경로에서는 그 문서 페이지가 본문에 채워진다. 페이지 제목·설명 등 메타 정보는 허브일 때와 하위 페이지일 때를 구분해 설정한다(하위 페이지는 자체적으로 메타를 덮어쓴다).

## 권한 / 제약

공개 경로로 인증이 필요 없다. 정적 SPA로 서빙되며, 하위 문서 경로들은 이 경로의 중첩 자식으로 동작하므로 사이드바·헤더 같은 공통 레이아웃을 공유한다.

## provenance

라우팅은 `landing/src/App.tsx`에서 `/docs`가 부모 라우트로 선언되고 그 아래에 getting-started·workflow·cli·architecture가 자식으로 중첩된 구조에서 확인된다. 레이아웃·허브 내용·사이드바 내비게이션 정의와 "정확히 /docs면 허브, 아니면 자식 렌더" 분기는 `landing/src/pages/Docs.tsx`에 있다. 매칭·렌더는 SPA 라우터(`component.spa-router`)가, 화면은 문서 허브(`screen.docs-hub`)가 담당한다.

## 미확정 (OPEN)
- [ ] 없음 — 라우트 구조·허브 구성·중첩 관계가 코드에서 명확히 확인됨.
