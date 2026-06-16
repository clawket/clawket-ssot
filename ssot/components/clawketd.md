---
id: component.clawketd
kind: SystemComponent
title: clawketd 데몬 프로세스
definition: Clawket 의 백그라운드 서버 프로세스. 하나의 HTTP 라우터를 로컬 TCP(기본 127.0.0.1:19400)와 Unix 소켓 두 경로로 동시에 노출하고, 모든 작업 데이터를 단일 SQLite 파일에 보관하며, 지식·태스크 임베딩 백필을 시작 시 비동기로 돌린다.
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:systemcomponent"]
realizedBy: []
implementedIn:
  - daemon/src/main.rs
  - daemon/Cargo.toml
  - daemon/src/config.rs
dependsOn:
  - component.http-router
  - component.db-store
  - component.repo-layer
  - component.embeddings
consumesApi: []
providesApi:
  - endpoint.daemon-health
  - endpoint.daemon-events
  - endpoint.daemon-search
integratesWith: []
impacts:
  - domain.work-management
  - domain.knowledge-rag
relatesTo:
  - to: component.daemon
    type: relates-to
    note: 인벤토리의 component.daemon(배포·바이너리 관점의 데몬)을 프로세스 내부 구조 관점에서 구체화한 노드.
  - to: component.sqlite-storage
    type: depends-on
    note: 모든 상태를 단일 SQLite 파일에 보관한다.
---

## 책임

clawketd 는 Clawket 의 상주 서버다. 사용자가 직접 띄우는 화면이 아니라, CLI·웹 대시보드·LLM 에이전트가 공통으로 바라보는 "하나의 진실"을 지키는 뒷단 프로세스다. 책임은 세 갈래로 나뉜다.

첫째, **요청 받기**. 같은 작업 데이터에 두 종류의 클라이언트가 접근한다 — 같은 기기에서 직접 붙는 CLI/스킬(빠르고 신뢰되는 경로)과, 브라우저에서 붙는 웹 대시보드(네트워크 경유 경로)다. clawketd 는 동일한 기능 표면을 이 두 경로 모두에 동시에 열어 둔다.

둘째, **데이터 지키기**. 프로젝트·플랜·유닛·사이클·태스크·지식 등 모든 상태는 이 프로세스가 소유한 단일 로컬 데이터베이스 파일에 들어간다. 프로세스가 부팅할 때 데이터베이스 구조를 현재 바이너리가 이해하는 최신 형태까지 단계적으로 끌어올린다(마이그레이션). 이미 최신이면 아무 일도 하지 않는다.

셋째, **의미 검색 준비**. 시작 직후, 아직 벡터(임베딩)가 없는 지식·태스크를 찾아 비동기로 채워 넣는 백필 작업을 돌린다. 이 덕분에 키워드뿐 아니라 의미 기반 검색이 가능해진다.

## 경계와 의존

clawketd 는 자기 안에서 모든 일을 하지 않고 역할별 내부 부품에 위임한다. 들어온 요청을 알맞은 처리 담당에게 나눠 주는 일은 라우터(`component.http-router`)가, 실제 데이터 읽기·쓰기는 저장소 계층(`component.repo-layer`)과 그 아래 데이터베이스 부팅·마이그레이션(`component.db-store`)이, 의미 벡터 생성은 임베딩 엔진(`component.embeddings`)이 맡는다.

이 노드는 배포·바이너리 관점의 데몬(`component.daemon`)을 프로세스 내부 구조 관점에서 더 깊게 본 것이며, 상태 보관처로 단일 SQLite 저장소(`component.sqlite-storage`)에 의존한다.

## 통신 패턴

요청은 두 리스너로 받는다. **Unix 소켓** 경로는 같은 기기·같은 사용자만 접근할 수 있고(소켓 파일 권한을 소유자 전용으로 제한), 로컬에서 직접 붙는 신뢰된 통로라 추가 인증을 거치지 않는다. **TCP** 경로는 기본적으로 루프백(127.0.0.1 등 자기 자신)에만 묶이고, 외부 주소로 열려는 시도는 명시적 옵트인이 없는 한 거부한다 — Clawket 은 본래 로컬 전용 데이터를 다루기 때문이다. TCP 경로에는 매 재시작마다 새로 발급되는 토큰 기반 인증이 붙는다.

두 경로 모두 동일한 처리 표면을 공유하되, 데이터베이스 구조를 끌어올리는 동안 들어온 변경 요청은 일시적으로 "지금 마이그레이션 중"이라며 잠시 막는다(읽기는 통과). 종료 신호를 받으면 진행 중 요청에 짧은 유예를 준 뒤 강제 종료해, 오래 살아 있는 연결 하나 때문에 종료가 무한정 걸리는 상황을 막는다.

지식·태스크가 새로 생기거나 바뀌면 의미 벡터가 다시 만들어지고, 변경 사건은 실시간 이벤트 스트림으로 흘러 웹 대시보드와 관찰(watch) 클라이언트에 전달된다.

## 하위 서브패키지 (책임 단위)

- **요청 분배(라우터)** — `component.http-router`: 27개 영역별 처리 묶음을 하나의 요청 표면으로 합친다.
- **데이터베이스 부팅·마이그레이션** — `component.db-store`: 단일 SQLite 파일을 열고 구조를 최신까지 단계적으로 끌어올린다.
- **저장소 계층** — `component.repo-layer`: 엔티티별 데이터 읽기·쓰기를 캡슐화한다.
- **임베딩 엔진** — `component.embeddings`: 작업·지식 텍스트를 의미 벡터로 만든다.

## 미확정 (OPEN)
- [ ] OPEN: 기본 포트(19400) 점유 시의 폴백 포트 선택 규칙 세부는 별도 확인 필요.
- [ ] OPEN: 종료 시 강제 종료 전 유예 시간의 정확한 값은 환경에 따라 다를 수 있어 미확정.
