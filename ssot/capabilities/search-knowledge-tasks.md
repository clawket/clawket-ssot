---
id: capability.search-knowledge-tasks
kind: Capability
title: 지식·태스크 검색(키워드/의미/하이브리드)
purpose: "쌓인 지식 항목과 태스크를 키워드·의미·하이브리드 방식으로 검색하고, 유사 태스크 찾기·최근 결정 보기 같은 단축 조회로 과거 맥락을 빠르게 되살린다."
servesPersona:
  - persona.llm-coding-agent
  - persona.human-operator
realizedBy:
  - component.cli
  - component.daemon
  - component.embedding-engine
implementedIn:
  - cli/src/main.rs
relatesTo:
  - { to: domain.knowledge-rag, type: relates-to, note: "이 기능이 속한 지식·로컬 RAG 도메인" }
  - { to: concept.knowledge, type: relates-to, note: "검색 대상이 되는 지식 항목" }
  - { to: concept.task, type: relates-to, note: "제목·본문이 검색·임베딩되는 태스크" }
  - { to: endpoint.daemon-search, type: calls, note: "FTS5/벡터 KNN/하이브리드 검색 실제 수행 지점" }
  - { to: endpoint.cli-task, type: calls, note: "태스크 검색 하위 명령(task search)" }
  - { to: endpoint.cli-knowledge, type: calls, note: "지식 검색 하위 명령(knowledge search)" }
  - { to: endpoint.cli-rag-aliases, type: calls, note: "find-similar/get-recent-decisions 단축 명령" }
impacts:
  - concept.knowledge
  - concept.task
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:capability
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로. -->

## 사용자가 할 수 있는 일

작업이 쌓이면 "전에 이 결정을 왜 내렸지", "비슷한 일을 한 적이 있었나"를 찾아야 한다. 이 기능은 그 탐색을 세 가지 방식으로 제공한다 — 정확한 단어로 찾는 키워드 검색, 표현이 달라도 뜻이 가까우면 찾아주는 의미 검색, 둘을 합친 하이브리드. 검색 대상은 지식 항목과 태스크 두 종류이며, 자주 쓰는 묶음("이 태스크와 비슷한 것", "최근 결정")은 별도 단축 명령으로 한 번에 꺼낼 수 있다. 핵심 가치는 세션이 끊겨도 과거의 의미적 맥락을 다시 끌어와 같은 실수를 반복하지 않는 것이다.

## 행위

- **태스크 검색**: 제목·본문을 대상으로 키워드(keyword)·의미(semantic)·하이브리드(hybrid) 중 한 모드로 검색한다(기본 keyword, 결과 수 제한 가능).
- **지식 검색**: 지식 항목을 같은 세 모드로 검색한다.
- **유사 태스크 찾기**: 질의에 의미적으로 가까운 태스크를 벡터 검색으로 찾는다(태스크 의미 검색의 최상위 별칭).
- **최근 결정 보기**: 결정(decision) 유형 지식 항목을 최신순으로 가져온다.

## 시스템 흐름

CLI의 검색 하위 명령(`endpoint.cli-task`의 search, `endpoint.cli-knowledge`의 search)과 RAG 단축 명령(`endpoint.cli-rag-aliases`)은 모두 데몬의 검색 API(`endpoint.daemon-search`)로 모인다. 데몬은 키워드는 SQLite 전문검색(FTS5), 의미는 온디바이스 임베딩으로 만든 벡터 이웃(KNN), 하이브리드는 둘의 결합으로 처리한다. 의미·하이브리드 검색은 임베딩 엔진(`component.embedding-engine`)이 미리 임베딩해 둔 결과에 의존한다.

## 어디에 구현되어 있나

CLI 명령 선언은 `cli/src/main.rs`에 있다. 태스크 검색(search 모드: keyword|semantic|hybrid)과 RAG 별칭(find-similar, get-recent-decisions 등)이 여기서 정의되고, 실제 검색 연산은 데몬 HTTP API로 위임된다. 같은 read-only 검색 표면이 MCP 도구로도 LLM에 노출된다.

## 미확정 (OPEN)
- [ ] OPEN: 하이브리드 모드의 점수 결합 방식(가중치/리랭킹) 데몬 측 코드 확인 필요
