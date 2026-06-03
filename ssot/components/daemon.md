---
id: component.daemon
kind: SystemComponent
title: clawketd 데몬 (Rust: axum+rusqlite)
definition: 유닉스 도메인 소켓과 루프백 TCP(기본 포트 19400)로 HTTP API·SSE 이벤트 버스를 제공하고 SQLite를 단독 소유하며, 지식·태스크 자동 임베딩과 시작 시 누락 임베딩 백필을 수행하는 로컬 데몬.
purpose: 모든 작업 상태(프로젝트·플랜·유닛·사이클·태스크·지식·실행기록·감사로그)의 단일 소유자이자 진실 저장소로서, CLI·웹·MCP·훅 어댑터가 공유하는 상태를 한곳에서 일관되게 다루기 위해 존재한다. 로컬에서만 동작(루프백+유닉스소켓)하며 외부로 트래픽을 내보내지 않는다는 로컬 우선 원칙을 물리적으로 보장하는 컴포넌트다. 검색을 위해 온디바이스 임베딩과 하이브리드 검색을 데몬 내부에 둬, 어떤 데이터도 외부 벡터 DB로 나가지 않게 한다.
realizedBy:
  - domain.work-management
  - domain.knowledge-rag
implementedIn:
  - daemon/src
  - daemon/src/main.rs
  - daemon/migrations
dependsOn:
  - component.sqlite-storage
  - component.embedding-engine
consumesApi: []
providesApi:
  - endpoint.daemon-health
  - endpoint.daemon-events
  - endpoint.daemon-activity
  - endpoint.daemon-plans-import-strict
  - endpoint.daemon-search
integratesWith:
  - integration.github-releases
impacts:
  - component.cli
  - component.web-dashboard
  - component.mcp-server
  - persona.llm-coding-agent
  - persona.human-operator
  - domain.knowledge-rag
relatesTo:
  - to: component.sqlite-storage
    type: depends-on
    note: 데몬이 SQLite + sqlite-vec 저장소를 단독 소유·소비한다
  - to: component.embedding-engine
    type: contains
    note: 자동 임베딩·시작 시 백필은 데몬이 온디바이스 임베딩 엔진을 구동해 수행
governedBy:
  - invariant.local-only-no-egress
  - invariant.path-separation-xdg
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:systemcomponent
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로 — 무엇을·왜·누가·어떤 규칙·무슨 데이터. 코드(테이블·필드·경로·SQL) 옮겨적기 금지(식별자는 provenance/근거에만). 코드 분기/의도 불명은 판정 말고 OPEN. -->

## 책임

Clawket의 상태 서버다. 작업관리에 관한 모든 상태 — 프로젝트·플랜·유닛·사이클·태스크·지식·실행기록·코멘트·질문·감사로그 — 를 SQLite에 단독으로 소유하고, 그 위에 읽기·쓰기 HTTP API를 올린다. 상태를 데몬이 단독 소유하기 때문에, CLI·웹·MCP·훅 어댑터가 동시에 같은 진실을 보고 일관된 갱신을 한다.

검색·RAG 책임도 데몬 안에 있다. 지식 항목과 태스크는 생성·수정 시점에 자동으로 임베딩되고, 데몬 시작 시 누락된 임베딩이 있으면 백필한다. 키워드 검색·의미 검색·하이브리드 검색을 모두 데몬이 제공하므로, 외부 벡터 DB로 어떤 데이터도 내보내지 않는다.

태스크가 done/cancelled로 끝나면, 그 부모 유닛·플랜·사이클의 자식이 모두 종료 상태일 때 데몬이 완료를 자동 cascade한다 — 이를 위한 별도 훅이 필요 없다.

## 경계와 의존

데몬은 로컬에서만 듣는다. 유닉스 도메인 소켓과 루프백 TCP(기본 포트 19400, 사용 중이면 자동 증가)에만 바인딩하고, 바깥으로 나가는 네트워크 요청을 만들지 않는다. 저장은 SQLite + sqlite-vec 저장소(`component.sqlite-storage`)에 위임하며, 임베딩 계산은 온디바이스 임베딩 엔진(`component.embedding-engine`)을 구동해 수행한다. 데몬 바이너리 자체는 GitHub Releases에서 받아 설치된다(`integration.github-releases`).

## 통신 패턴

CLI·웹·MCP가 데몬의 HTTP API를 호출해 상태를 읽고 쓴다(`endpoint.daemon.*`로 정렬). 실시간 변경은 SSE 이벤트 버스로 푸시되어 웹 대시보드와 watch가 구독한다. 훅 어댑터는 헬스 핑·활동 기록·플랜 strict 검증을 데몬에 보내고, 데몬이 떠 있지 않으면 어댑터의 강제 검사는 우아하게 건너뛴다. 검색은 키워드(FTS5)·의미(벡터 KNN)·하이브리드를 같은 엔드포인트군으로 제공하고, 웹 위키·CLI search·MCP가 이를 공용한다.

## 하위 서브패키지 (책임 단위)

- 상태 저장소 소유(`component.sqlite-storage`): 모든 작업 상태와 벡터 가상 테이블의 단일 보관소.
- 임베딩·검색(`component.embedding-engine` + 검색 라우트): 자동 임베딩·백필·하이브리드 검색.
- HTTP/SSE 표면: 헬스·활동·플랜 strict 검증·검색·이벤트 스트림 엔드포인트.
- 마이그레이션: 시작 시 보류 스키마 변경 자동 적용.

## 미확정 (OPEN)
- [ ] OPEN: 포트 충돌 시 자동 증가의 상한·재시도 정책, 그리고 완료 자동 cascade의 정확한 조건 경계는 endpoint.daemon.* 및 concept.cycle/plan 노드에 위임.
