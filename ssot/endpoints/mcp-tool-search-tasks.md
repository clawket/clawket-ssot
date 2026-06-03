---
id: endpoint.mcp-tool-search-tasks
kind: Endpoint
title: MCP 도구 clawket_search_tasks (구현 정의)
definition: 태스크를 하이브리드/키워드/의미 모드로 검색해 티켓번호·제목·상태·우선순위·유사도를 돌려주는 read-only MCP 도구.
realizedBy: [component.mcp-server, component.cli]
implementedIn:
  - cli/src/mcp.rs
relatesTo:
  - { to: concept.task, type: reads, note: "태스크를 검색해 요약(티켓번호·제목·상태·우선순위)으로 반환한다" }
  - { to: concept.ticket-number, type: reads, note: "결과에 사람이 읽는 티켓 번호(CK-xxx)를 포함한다" }
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

LLM 코딩 에이전트가 "과거에 비슷한 작업을 했나, 관련 티켓이 있나"를 확인할 때 부르는 read-only MCP 도구다. 작업 티켓(태스크)을 의미·키워드를 섞어 검색해 요약 목록으로 돌려준다. 더 풍부한 패턴·결정 추출이 필요하면 자매 도구인 clawket_find_similar_tasks(endpoint.mcp-tool-find-similar-tasks)를 쓰도록 안내한다.

## 요청 / 응답

요청으로는 검색어, 검색 모드(하이브리드 기본, 키워드/의미 선택), 결과 상한(기본 10건, 내부 상한으로 캡), 그리고 상태 필터(예: in_progress)를 줄 수 있다. 응답으로는 각 태스크의 사람이 읽는 티켓 번호(CK-xxx)·제목·상태·우선순위·소속 유닛·검색 거리(유사도)를 담은 요약 목록이 돌아온다. 본문 전체가 아니라 요약만 주어 컨텍스트 비용을 줄인다.

## 권한 / 제약

데몬이 살아 있어야 동작한다(도구는 데몬의 태스크 검색 HTTP API를 호출한다). read-only — 태스크를 만들거나 바꾸지 않는다. 상태 필터는 데몬이 돌려준 결과를 도구 측에서 한 번 더 걸러 적용한다. 검색 모드를 지정하지 않으면 하이브리드로 동작하며 결과 수는 내부 최대치로 잘린다.

## provenance

도구 정의·요청 처리는 cli/src/mcp.rs(178-213행)에 있다. 사람이 읽는 도구 요약은 endpoint.mcp-search-tasks 에 별도로 정리되어 있으며, 실제 검색은 데몬의 검색 HTTP API(endpoint.daemon-search)에 위임된다. CLI 바이너리에 내장된 MCP 서버(component.mcp-server)가 노출하는 지식 도구군의 하나다.

## 미확정 (OPEN)
- [ ] OPEN: 결과 요약(task_summary)에 포함되는 정확한 필드 집합과 하이브리드 모드 가중 방식은 데몬 검색 라우트 정독으로 확인.
