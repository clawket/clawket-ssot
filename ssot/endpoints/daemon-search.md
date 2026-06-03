---
id: endpoint.daemon-search
kind: Endpoint
title: 데몬 검색 HTTP API (FTS5/KNN/hybrid)
definition: 태스크와 지식을 대상으로 키워드 검색·의미 검색·둘을 합친 하이브리드 검색을 제공하는 데몬 검색 통로. 웹 위키, CLI search 명령, MCP 도구가 모두 이 통로를 공유한다.
realizedBy:
  - component.daemon
implementedIn:
  - clawket/README.md
relatesTo:
  - to: concept.task
    type: reads
    note: 태스크를 검색 대상으로 조회한다
  - to: concept.knowledge
    type: reads
    note: 지식 항목을 검색 대상으로 조회한다
  - to: component.embedding-engine
    type: depends-on
    note: 의미 검색에 온디바이스 임베딩 벡터를 사용한다
  - to: component.sqlite-storage
    type: backed-by
    note: FTS5 인덱스와 vec0 벡터 테이블이 저장소에 있다
governedBy: []
impacts:
  - domain.knowledge-rag
consumedBy:
  - component.web-dashboard
  - endpoint.cli-knowledge
  - endpoint.cli-task
  - component.mcp-server
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의
"이런 내용을 찾고 싶다"는 요청에 가장 관련 있는 태스크·지식을 돌려주는 검색 통로다. 세 가지 방식으로 찾을 수 있다 — 단어가 그대로 들어맞는지로 찾는 키워드 검색, 단어가 달라도 의미가 가까운지로 찾는 의미 검색, 그리고 두 방식의 장점을 합친 하이브리드 검색이다. 이 통로 하나를 웹 대시보드의 위키 검색, 터미널의 검색 명령, 그리고 LLM이 과거 맥락을 끌어오는 MCP 도구가 공통으로 쓴다 — 즉 사람이 보는 화면과 에이전트가 보는 검색이 같은 결과 기반을 공유한다.

## 요청 / 응답
보내는 정보는 찾고 싶은 질의어와 검색 방식(키워드/의미/하이브리드), 그리고 대상(태스크인지 지식인지)이다. 돌려받는 것은 관련도 순으로 정렬된 항목 목록이다 — 각 항목은 해당 태스크 또는 지식의 본문과 그것을 식별할 수 있는 정보를 가지며, 의미·하이브리드 검색의 경우 "얼마나 가까운가"라는 관련도까지 포함한다. 사람은 이를 위키 검색 결과로 보고, 에이전트는 세션 간 컨텍스트 복원의 재료로 받는다.

## 권한 / 제약
데몬이 로컬 루프백에만 바인딩되므로 같은 기기 안에서만 검색한다. 의미·하이브리드 검색은 온디바이스 임베딩(component.embedding-engine)으로 만든 벡터를 사용하며, 검색 인덱스는 저장소(component.sqlite-storage)의 키워드 인덱스(FTS5)와 벡터 테이블(vec0)에 보관된다. 임베딩과 검색이 모두 로컬에서 이뤄지므로 질의나 본문이 외부로 나가지 않는다. 새로 만들거나 고친 항목은 임베딩이 갱신되어야 의미 검색에 반영된다.

## provenance
근거는 `clawket/README.md`로, 데몬이 키워드(FTS5)·의미(vec0에 대한 KNN)·하이브리드 검색 HTTP 엔드포인트를 노출하고 "같은 엔드포인트를 웹 위키·CLI search 서브커맨드·MCP 서버가 재사용한다"고 명시한다. 벡터 저장은 sqlite-vec의 vec0 가상 테이블, 임베딩은 온디바이스 다국어 모델을 쓴다. 제공 주체는 로컬 데몬(component.daemon), 소비 주체는 웹 대시보드(component.web-dashboard)·`clawket knowledge`/`clawket task`의 search(endpoint.cli-knowledge, endpoint.cli-task)·MCP 서버(component.mcp-server)다.

## 미확정 (OPEN)
- [ ] OPEN: 하이브리드 검색의 키워드·의미 점수 결합 방식(가중치·랭킹 규칙)과 검색 엔드포인트의 정확한 경로·페이지네이션·기본 검색 방식은 데몬 라우터·검색 코드를 정독해 확인 필요.
