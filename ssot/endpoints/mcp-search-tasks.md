---
id: endpoint.mcp-search-tasks
kind: Endpoint
title: MCP clawket_search_tasks
definition: LLM 에이전트가 과거·현재 태스크(작업 티켓)를 의미·키워드·하이브리드로 검색해 비슷한 작업이나 관련 티켓을 찾는 read-only MCP 도구.
realizedBy:
  - component.mcp-server
  - component.daemon
implementedIn:
  - cli/src/mcp.rs
relatesTo:
  - to: concept.task
    type: reads
    note: 작업 티켓을 검색 대상으로 삼아 티켓번호·제목·상태·우선순위·소속유닛·유사도를 돌려준다
  - to: endpoint.daemon-search
    type: depends-on
    note: 실제 검색은 데몬의 태스크 검색 HTTP API에 위임한다
  - to: concept.ticket-number
    type: reads
    note: 결과에 사람이 읽는 티켓번호(CK-N)를 함께 담는다
  - to: domain.knowledge-rag
    type: belongs-to
governedBy:
  - invariant.mcp-tools-readonly
  - invariant.mcp-response-cap
impacts:
  - capability.rag-pull
consumedBy:
  - persona.llm-coding-agent
  - integration.mcp
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

LLM 코딩 에이전트가 "예전에 비슷한 작업을 한 적 있나", "이 일과 관련된 티켓이 있나"를 확인하기 위해 작업 티켓(태스크)을 검색하는 도구다. 새 작업을 시작하기 전에 중복·선례를 찾거나 맥락을 잇는 데 쓴다. 검색만 하고 어떤 것도 바꾸지 않는다.

더 풍부한 결과(코멘트에서 결정·이슈 패턴까지 뽑아 주는 것)가 필요하면 비슷한-태스크-찾기 도구(`clawket_find_similar_tasks`)를 쓰고, 이 도구는 가벼운 목록 검색에 쓴다.

## 요청 / 응답

요청에는 검색어, 검색 방식(키워드/의미/하이브리드, 기본 하이브리드), 가져올 개수(1~30개, 기본 10개)를 담고, 태스크 상태(할 일·진행 중·완료·취소됨·차단됨)로 좁힐 수 있다.

응답은 검색에 걸린 태스크의 요약 목록이다. 항목마다 식별자, 사람이 읽는 티켓번호(CK-N), 제목, 상태, 우선순위, 작업 유형, 소속 유닛, 그리고 검색어와의 유사도를 돌려준다. 결과가 너무 커지면 잘림 안내와 함께 앞부분만 돌려주므로 범위를 좁혀 다시 부르면 된다.

## 권한 / 제약

읽기 전용이며 데몬이 떠 있어야 동작한다. 데몬에 닿지 못하면 구조화된 오류를 돌려준다. 가져올 개수는 최대 30개로 제한된다. 상태 필터는 데몬이 돌려준 결과를 도구 쪽에서 한 번 더 걸러 적용한다.

## provenance

도구 정의와 데몬 호출은 `cli/src/mcp.rs`의 `clawket_search_tasks`에 있다. 검색 연산 자체는 데몬의 태스크 검색 HTTP API(`endpoint.daemon-search`)에 위임하고, CLI 내장 MCP stdio 서버(`component.mcp-server`)가 노출한다.

## 미확정 (OPEN)
- [ ] OPEN: 데몬의 `/tasks/search` 응답에 우선순위·유형·유닛·_distance 가 항상 포함되는지, 상태 필터가 데몬단에서도 지원되는지 데몬 라우트 정독으로 확인.
