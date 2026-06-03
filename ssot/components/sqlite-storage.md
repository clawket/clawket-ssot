---
id: component.sqlite-storage
kind: SystemComponent
title: SQLite + sqlite-vec 저장소
definition: Clawket의 모든 작업 상태와 의미 검색용 벡터를 사용자 컴퓨터 안 단일 SQLite 데이터베이스 파일에 담는 단일 진실 저장소다. 프로젝트·플랜·유닛·사이클·태스크·지식·실행기록·감사로그 같은 모든 데이터가 여기 모여 있고, 의미 검색을 위한 임베딩 벡터는 sqlite-vec 확장의 가상 테이블로 같은 파일 안에 함께 보관된다.
purpose: "Clawket이 다루는 모든 데이터(작업 관리 상태 + 지식 + 검색용 벡터)를 한 곳에 영속화하는 로컬 저장 계층. 외부 데이터베이스나 클라우드 없이 사용자 기기 안에서만 진실을 보관한다."
realizedBy:
  - domain.work-management
  - domain.knowledge-rag
  - platform.local-first
implementedIn:
  - daemon/src/db.rs
dependsOn: []
consumesApi: []
providesApi: []
integratesWith: []
impacts:
  - component.daemon
  - component.embedding-engine
  - domain.knowledge-rag
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:systemcomponent"]
relatesTo:
  - to: component.daemon
    type: depends-on
    note: 데몬만이 이 데이터베이스 파일을 직접 열고 읽고 쓴다. 다른 모든 구성 요소는 데몬을 거쳐 간접 접근한다.
  - to: component.embedding-engine
    type: backed-by
    note: 임베딩 엔진이 만든 384차원 벡터가 이 저장소의 벡터 가상 테이블에 적재된다.
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 분기/의도 불명은 OPEN. -->

## 책임

이 구성 요소는 Clawket이 다루는 데이터의 "단 하나의 진실 원천"이다. 프로젝트, 플랜, 유닛, 사이클, 태스크, 백로그, 지식 문서, 실행 기록, 코멘트, 질문, 감사 로그까지 — 사용자와 LLM 에이전트가 만들어 내는 모든 작업 상태가 사용자 컴퓨터 안의 단일 SQLite 데이터베이스 파일 하나에 모인다. 이 파일은 사용자 홈 디렉터리 아래 데이터 경로(`~/.local/share/clawket/db.sqlite`, 환경변수로 위치 변경 가능)에 놓인다.

저장소는 두 가지 성격의 데이터를 함께 품는다. 첫째는 평범한 관계형 작업 데이터(누가 어떤 태스크를 언제 어떤 상태로 바꿨는가)이고, 둘째는 의미 검색을 위한 벡터 데이터다. 벡터는 sqlite-vec 확장이 제공하는 특수한 가상 테이블에 저장되어, 태스크와 지식 문서를 "비슷한 의미"로 찾을 수 있게 한다. 태스크용 벡터 묶음과 지식용 벡터 묶음이 각각 따로 관리되며, 둘 다 384개 숫자로 이루어진 임베딩을 담는다.

## 경계와 의존

이 저장소는 수동적인 데이터 보관 계층이라 스스로 외부와 통신하지 않는다. 오직 데몬(`component.daemon`)만이 이 파일을 열어 직접 읽고 쓴다. CLI, MCP 서버, 웹 대시보드 등 다른 모든 구성 요소는 데몬의 HTTP API를 거쳐 간접적으로만 이 데이터에 닿는다. 이 단일 통로 구조 덕분에 동시 접근과 일관성이 데몬 한 곳에서 관리된다.

벡터 데이터는 임베딩 엔진(`component.embedding-engine`)에 의존한다. 태스크나 지식이 만들어지거나 바뀔 때 엔진이 텍스트를 384차원 벡터로 변환하고, 그 결과가 이 저장소의 벡터 가상 테이블에 적재된다. 벡터 갱신은 단순 덮어쓰기가 아니라 "기존 행 삭제 후 새 행 삽입" 방식으로 처리되는데, sqlite-vec 가상 테이블이 일반적인 갱신·삽입(INSERT OR REPLACE)을 지원하지 않기 때문이다.

## 통신 패턴

저장소 자체는 네트워크 통신을 하지 않는다. 디스크에 있는 단일 파일로 존재하며, 데몬 프로세스가 파일을 직접 여닫는다. 스키마는 데몬이 시작될 때 마이그레이션 절차로 최신 형태로 맞춰지고, 누락된 태스크 벡터가 있으면 시작 시 백필(빠진 부분 채우기)로 보강된다. 어떤 데이터도 이 기기 밖으로 나가지 않는다 — 이것이 Clawket의 로컬 우선 원칙을 떠받치는 물리적 근거다.

## 하위 서브패키지 (책임 단위)

- 관계형 작업 데이터 영역: 프로젝트~감사 로그까지의 일반 테이블 군.
- 벡터 가상 테이블 영역: sqlite-vec(`vec0`) 기반의 태스크용·지식용 임베딩 테이블 군. 의미 검색의 물리적 토대.
- 전문 검색 인덱스: 키워드 검색(FTS5)을 위한 인덱스 — 의미 검색과 합쳐 하이브리드 검색을 구성한다.

## 미확정 (OPEN)
- [ ] OPEN: 데이터베이스 파일의 백업·복원 시 벡터 가상 테이블이 함께 정상 이관되는지의 구체 동작은 백업 구성 요소 측에서 별도 확인 필요.
- [ ] OPEN: 전체 테이블 목록·스키마 세부는 마이그레이션 디렉터리에서 추가 추적이 필요(이 노드는 책임·경계 수준으로만 기술).
