---
id: screen.docs-architecture
kind: Screen
title: 아키텍처 문서 화면
purpose: 방문자가 Clawket이 어떤 구조로 동작하는지 — 어떤 부품이 어떻게 맞물리고, 데이터가 어디에 저장되며, 무엇이 기기 밖으로 나가지 않는지 — 를 한 페이지에서 이해하도록 돕는다.
servesPersona:
  - persona.human-operator
realizedBy:
  - component.landing-site
implementedIn:
  - landing/src/pages/docs/Architecture.tsx
consumesApi: []
relatesTo:
  - to: platform.local-first
    type: relates-to
    note: 데몬이 루프백·유닉스소켓에만 바인딩하고 임베딩도 온디바이스라 외부 전송이 없음을 화면이 설명한다.
  - to: domain.knowledge-rag
    type: relates-to
    note: vec_tasks/vec_knowledge 두 벡터 테이블과 키워드·의미·하이브리드 검색이 데몬 안에 있음을 설명한다.
  - to: component.cli
    type: relates-to
  - to: component.daemon
    type: relates-to
  - to: component.sqlite-storage
    type: relates-to
  - to: component.embedding-engine
    type: relates-to
  - to: component.mcp-server
    type: relates-to
  - to: component.web-dashboard
    type: relates-to
  - to: component.claude-adapter
    type: relates-to
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:screen
---

## 화면 목적
랜딩 사이트의 문서 영역 중 "아키텍처" 페이지다. 처음 Clawket을 접한 사람이 제품의 전체 그림을 잡도록, 시스템이 어떤 계층으로 나뉘고 각 계층이 무슨 역할을 하며 데이터가 어디에 어떻게 저장되는지를 자연어 설명·표·다이어그램으로 풀어준다. 핵심 메시지는 "로컬 우선 — 기본 설정에서는 어떤 데이터도 당신의 기기를 떠나지 않는다"이다.

읽는 주체는 도구를 평가·도입하려는 운영자/개발자다. 따라서 설치 방법이 아니라 "이게 내 데이터를 어떻게 다루는가, 믿을 만한 구조인가"라는 의사결정 질문에 답하도록 구성되어 있다.

## UI 요소 / 입력 필드
입력 요소가 없는 순수 설명(읽기 전용) 화면이다. 다음 정보 블록으로 구성된다.

- 상단 요약: 플러그인 셸(훅) + Rust CLI(운영 표면·내장 MCP) + Rust 데몬(SQLite·벡터 인덱스·HTTP/SSE) + React 웹 대시보드가 같은 데몬 API를 공유하는 한 문단 개요.
- 시스템 다이어그램: Claude Code → 플러그인 훅 / .mcp.json 자식 프로세스 → 내장 MCP(stdio) → 데몬(유닉스소켓·TCP 루프백·SSE·자동 임베딩·시작 시 백필) → SQLite+sqlite-vec로 이어지는 데이터 흐름을 아스키 도식으로 보여준다.
- 컴포넌트 표: 계층(CLI·데몬·저장소·임베딩·MCP·웹·어댑터)별 기술 스택 요약.
- XDG 경로 표: 데이터·캐시·설정·로그가 각각 어느 디렉터리에 살고 어떤 환경변수로 위치를 바꾸는지.
- 경로 분리 불변식 설명: 사용자 데이터가 플러그인 디렉터리 밖에 있어야 하는 이유(플러그인 재설치가 그 트리를 지우므로)와 데몬 런타임 가드·doctor 진단의 2중 방어.
- 로컬 RAG·MCP 통합·프라이버시 섹션.

## 표시 데이터 / 호출 API
이 화면은 정적 콘텐츠만 렌더링하며 데몬 API를 호출하지 않는다(consumesApi 없음). 표시하는 정보의 의미는 다음과 같다.

- 부품 구성: CLI는 MCP를 내장한 단일 Rust 바이너리, 데몬은 axum+rusqlite 기반, 저장소는 SQLite+sqlite-vec, 임베딩은 candle-core로 다국어 모델을 온디바이스 실행, 웹은 React 19 SPA, 어댑터는 Claude Code 플러그인.
- 데이터 위치: SQLite DB·벡터 인덱스는 데이터 디렉터리에, 소켓·런타임 상태는 캐시 디렉터리에, 설정은 설정 디렉터리에, 로그·감사 추적은 상태 디렉터리에 저장된다.
- RAG 동작: 태스크는 생성·수정마다 임베딩되고 시작 시 누락분이 백필되며, 지식은 본문이 있을 때 임베딩된다. 키워드·의미·하이브리드 3가지 검색이 웹 위키·CLI·MCP가 공유하는 동일 엔드포인트로 노출된다.
- MCP: CLI에 내장된 stdio 서버가 5개의 읽기 전용 도구를 노출하며 데몬 상태를 변경하지 않는다.
- 프라이버시: 데몬은 루프백(기본 포트 19400, 사용 중이면 자동 증가)과 유닉스소켓에만 바인딩하고 외부 네트워크 요청을 하지 않으며, 원격 텔레메트리가 없고 유일한 관측 데이터인 로컬 감사 로그도 기기 밖으로 전송되지 않는다.

## 상태 / 엣지케이스
정적 페이지라 로딩·빈·에러 상태나 권한 분기가 없다. 라우팅 측면에서, 이 페이지는 랜딩 SPA의 일부(`/docs/architecture`)로 클라이언트 라우팅을 통해 표시되며, 별도 정적 HTML로 분리 서빙되는 블로그·튜토리얼과 달리 SPA 인덱스 fallback 경로 안에서 렌더링된다.

## 미확정 (OPEN)
- [ ] OPEN: 문서 허브(screen.docs-hub)에서 이 페이지로의 내비게이션 구조·연관 문서 링크를 자연어로 보강할 것.
