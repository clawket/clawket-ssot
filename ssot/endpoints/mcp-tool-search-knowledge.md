---
id: endpoint.mcp-tool-search-knowledge
kind: Endpoint
title: MCP 도구 clawket_search_knowledge (구현 정의)
definition: 지식 항목을 하이브리드/키워드/의미 모드로 검색해 제목·유형·300자 스니펫·유사도를 돌려주는 read-only MCP 도구.
realizedBy: [component.mcp-server, component.cli]
implementedIn:
  - cli/src/mcp.rs
relatesTo:
  - { to: concept.knowledge, type: reads, note: "지식 항목을 검색해 요약 형태로 반환한다" }
  - { to: concept.plan, type: reads, note: "plan 필터로 특정 플랜 소속 지식만 좁힌다" }
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

LLM 코딩 에이전트가 작업 도중 "이 프로젝트의 설계 문서·결정사항·스펙이 있나"를 찾을 때 부르는 read-only MCP 도구다. 이전 세션에서 내려진 결정이나 특정 주제의 문서를 의미·키워드를 섞어 탐색하고, 결과를 토큰 절약된 요약 형태로 돌려준다. 같은 검색 기능을 CLI(clawket knowledge search)에서도 쓰지만, 이 도구는 Claude Code의 tool_use 경로로 노출되는 진입점이다.

## 요청 / 응답

요청으로는 검색어, 검색 모드(하이브리드가 기본, 키워드/의미 선택 가능), 결과 상한(기본 10건, 내부 상한으로 캡), 그리고 좁히기용 필터로 지식 유형과 소속 플랜을 줄 수 있다. 응답으로는 각 지식 항목의 식별자·제목·유형·소속 플랜/유닛·본문 앞부분 300자 스니펫·검색 거리(유사도)를 담은 목록이 돌아온다. 본문 전체가 아니라 스니펫만 주어 LLM 컨텍스트 비용을 줄인다.

## 권한 / 제약

데몬이 살아 있어야 동작한다(도구는 데몬의 지식 검색 HTTP API를 호출한다). read-only — 어떤 지식도 만들거나 바꾸지 않는다. 유형·플랜 필터는 데몬이 돌려준 결과를 도구 측에서 한 번 더 걸러 적용하며, 스니펫은 300자로, 결과 수는 내부 최대치로 잘린다. 검색 모드를 지정하지 않으면 하이브리드(키워드+의미)로 동작한다.

## provenance

도구 정의·요청 처리는 cli/src/mcp.rs(123-176행)에 있다. 사람이 읽는 도구 요약은 endpoint.mcp-search-knowledge 에 별도로 정리되어 있으며, 실제 검색은 데몬의 검색 HTTP API(endpoint.daemon-search)에 위임된다. 이 도구는 CLI 바이너리에 내장된 MCP 서버(component.mcp-server)가 노출하는 5개 지식 도구 중 하나다.

## 미확정 (OPEN)
- [ ] OPEN: 결과 수 내부 상한(LIMIT_MAX) 값과 하이브리드 모드의 키워드/의미 가중 방식은 데몬 검색 라우트·임베딩 모듈 정독으로 확인.
