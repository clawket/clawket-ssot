---
id: capability.read-docs
kind: Capability
title: 문서 읽기
definition: 방문자가 문서 허브와 사이드바를 통해 시작하기·워크플로우·CLI 레퍼런스·아키텍처 네 개 섹션을 둘러보며 Clawket의 설치·작업 흐름·명령·구조를 읽을 수 있다.
purpose: 검색으로 유입된 방문자가 제품의 사용법과 구조를 한 곳에서 파악하도록 정돈된 문서 경로를 제공한다.
servesPersona:
  - persona.human-operator
  - persona.llm-coding-agent
realizedBy:
  - component.landing-site
implementedIn:
  - landing/src/pages/Docs.tsx
  - landing/src/App.tsx
  - landing/src/pages/docs/GettingStarted.tsx
  - landing/src/pages/docs/Workflow.tsx
  - landing/src/pages/docs/CLI.tsx
  - landing/src/pages/docs/Architecture.tsx
relatesTo:
  - to: endpoint.route-docs
    type: relates-to
    note: 문서 허브가 열리는 라우트
  - to: endpoint.route-docs-getting-started
    type: relates-to
    note: 사이드바 섹션 — 시작하기
  - to: endpoint.route-docs-workflow
    type: relates-to
    note: 사이드바 섹션 — 워크플로우
  - to: endpoint.route-docs-cli
    type: relates-to
    note: 사이드바 섹션 — CLI 레퍼런스
  - to: endpoint.route-docs-architecture
    type: relates-to
    note: 사이드바 섹션 — 아키텍처
  - to: screen.docs-hub
    type: relates-to
    note: 문서 허브 화면이 이 기능의 진입 화면이다
impacts:
  - endpoint.route-docs
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:capability
---

## 사용자가 할 수 있는 일

방문자는 문서 영역(`/docs`)에 들어가 좌측 사이드바로 네 개 섹션을 오갈 수 있다 — 시작하기(설치·프로젝트 등록·근거와 함께 첫 태스크 닫기), 워크플로우(프로젝트→플랜→유닛→사이클→태스크의 분해·계약·실행 루프), CLI 레퍼런스(주요 명령과 `--help` 사용 습관), 아키텍처(CLI·데몬·SQLite·벡터 검색·온디바이스 임베딩·MCP 구성). 문서 허브 화면은 각 섹션의 요약 카드와, 제품의 정본 소스(플러그인 저장소와 네 개 구성요소 저장소)로 향하는 링크를 함께 제공한다.

## 행위

- 읽기(허브): 문서 허브에서 제품 개요와 네 섹션 요약, 정본 소스 링크를 본다.
- 탐색: 사이드바에서 현재 위치가 강조된 채로 섹션 간을 이동한다.
- 읽기(섹션): 선택한 섹션의 본문을 읽는다.

이 기능은 정적 문서 페이지만 사용하며 데몬 API나 CLI를 호출하지 않는다.

## 시스템 흐름

방문자가 `/docs`로 들어오면 사이드바와 본문 레이아웃이 그려지고, 본문에는 허브(개요·섹션 카드·소스 링크)가 표시된다. 사이드바에서 섹션을 고르면 하위 라우트(시작하기/워크플로우/CLI/아키텍처)가 본문 영역에 그려지고, 각 페이지는 자신에 맞는 제목·설명·정규 주소 등 head 정보를 설정한다. 라우팅은 홈과 문서 허브·하위 섹션을 한 앱에서 처리한다.

## 어디에 구현되어 있나

- `landing/src/App.tsx` — `/`, `/docs`, 그리고 네 하위 문서 라우트 정의.
- `landing/src/pages/Docs.tsx` — 문서 영역 레이아웃, 사이드바, 허브 화면(요약 카드·소스 링크).
- `landing/src/pages/docs/GettingStarted.tsx`, `Workflow.tsx`, `CLI.tsx`, `Architecture.tsx` — 네 섹션 본문.

이 기능은 랜딩 사이트(`component.landing-site`)에서 실현된다.

## 미확정 (OPEN)
- [ ] 없음 — 네 섹션과 허브 라우팅이 코드로 확인됨.
