---
id: endpoint.mcp-search-knowledge
kind: Endpoint
title: MCP clawket_search_knowledge
definition: LLM 에이전트가 프로젝트의 설계 문서·결정사항·스펙을 의미·키워드·하이브리드로 검색해 과거 컨텍스트를 끌어오는 read-only MCP 도구.
realizedBy:
  - component.mcp-server
  - component.daemon
implementedIn:
  - cli/src/mcp.rs
relatesTo:
  - to: concept.knowledge
    type: reads
    note: 위키성 지식 항목을 검색 대상으로 삼아 제목·타입·스니펫·유사도를 돌려준다
  - to: endpoint.daemon-search
    type: depends-on
    note: 실제 검색은 데몬의 지식 검색 HTTP API에 위임한다
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

LLM 코딩 에이전트가 "이전 세션에서 어떤 결정을 내렸지", "이 주제에 대한 설계 문서가 있나" 같은 질문에 답하기 위해, 프로젝트에 쌓아 둔 지식 문서(설계·결정·스펙·노트 등)를 검색하는 도구다. 사람이 키워드를 직접 입력하는 일반 검색창이 아니라, Claude Code 같은 에이전트가 도구 호출(tool_use)로 부르는 MCP 도구다. 검색만 하고 어떤 것도 바꾸지 않는다(read-only).

세 가지 검색 방식을 고를 수 있다. 키워드 일치(정확한 단어를 찾을 때), 의미 검색(표현이 달라도 뜻이 비슷한 문서를 찾을 때), 그리고 둘을 합친 하이브리드(기본값)다. 의미 검색은 온디바이스 임베딩으로 이루어지므로 검색어가 외부로 나가지 않는다.

## 요청 / 응답

요청에는 검색어(자연어 또는 키워드), 검색 방식(키워드/의미/하이브리드 중 하나, 기본 하이브리드), 가져올 개수(1~30개, 기본 10개)를 담는다. 추가로 지식 타입(결정·설계·아키텍처·스펙·노트·문서·참고 등)으로 좁히거나 특정 플랜에 속한 지식만 보도록 범위를 제한할 수 있다.

응답은 검색에 걸린 지식 항목 목록이다. 항목마다 식별자, 제목, 타입, 소속 플랜·유닛, 본문 앞부분을 추린 짧은 스니펫(약 300자까지), 그리고 검색어와 얼마나 가까운지를 나타내는 유사도 값을 돌려준다. 본문 전체가 아니라 요약 스니펫만 주는 이유는 에이전트가 한 번에 받는 양을 적정하게 유지하기 위해서다. 결과 전체가 너무 커지면(약 100KB 초과) 잘린 부분을 알리는 안내와 함께 앞부분만 돌려주므로, 그럴 때는 검색 범위를 좁히거나 개수를 줄여 다시 부르면 된다.

## 권한 / 제약

읽기 전용 도구라 데이터를 만들거나 고치거나 지우지 않는다. 동작하려면 로컬 데몬(clawketd)이 떠 있어야 하며, 데몬에 닿지 못하면 오류를 구조화된 형태로 돌려준다(중단되지 않는다). 가져올 개수는 최대 30개로 제한된다. 타입 필터와 플랜 범위 제한은 데몬이 돌려준 결과를 도구 쪽에서 한 번 더 걸러 적용한다.

## provenance

도구 정의와 데몬 호출은 `cli/src/mcp.rs`의 `clawket_search_knowledge`에 있다. 실제 검색 연산은 데몬의 지식 검색 HTTP API에 위임하며(`endpoint.daemon-search`), 이 도구는 CLI 바이너리에 내장된 MCP stdio 서버(`component.mcp-server`)가 노출한다. 데이터 저장·임베딩은 데몬(`component.daemon`)이 소유한다.

## 미확정 (OPEN)
- [ ] OPEN: 데몬의 `/knowledge/search` 응답이 plan_id/unit_id/_distance 필드를 항상 포함하는지(타입 필터·플랜 필터가 데몬단에서도 지원되는지) 데몬 라우트 정독으로 확인.
