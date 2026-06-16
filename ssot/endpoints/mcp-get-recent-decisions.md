---
id: endpoint.mcp-get-recent-decisions
kind: Endpoint
title: MCP clawket_get_recent_decisions
definition: 결정(type=decision) 지식 항목을 최신순으로 추려 돌려주는, 타입 기반 필터형 read-only MCP 도구.
realizedBy:
  - component.mcp-server
  - component.daemon
implementedIn:
  - cli/src/mcp.rs
relatesTo:
  - to: concept.knowledge
    type: reads
    note: 타입이 decision 인 지식 항목만 골라 최신순으로 돌려준다
  - to: endpoint.daemon-search
    type: depends-on
    note: 지식 목록 조회를 데몬 HTTP API에 위임한다
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

LLM 코딩 에이전트가 세션을 시작하며 "지난 회차에 어떤 결정이 있었지"를 빠르게 훑을 때 쓰는 도구다. 자연어 검색이 아니라 "결정"으로 분류된 지식 항목만 추려 최신순으로 돌려준다. 특정 주제를 키워드로 찾고 싶을 때는 지식 검색 도구(`clawket_search_knowledge`)를 쓰고, 이 도구는 "최근 결정 흐름을 시간순으로 보기"에 쓴다. 조회만 하고 아무것도 바꾸지 않는다.

## 요청 / 응답

요청에는 가져올 개수(1~30개, 기본 10개)를 담고, 특정 플랜에 속한 결정만 보도록 범위를 좁히거나, 특정 시점(Unix 밀리초) 이후에 만들어진 결정만 보도록 제한할 수 있다.

응답은 결정 지식 항목 목록을 최신순으로 정렬해 돌려준다. 항목마다 식별자, 제목, 소속 플랜·유닛, 생성 시각, 그리고 본문 앞부분을 추린 스니펫(약 500자까지 — 결정 내용을 충분히 담기 위해 일반 검색보다 길게)을 담는다.

## 권한 / 제약

읽기 전용이며 데몬이 떠 있어야 동작한다. 데몬에 닿지 못하면 구조화된 오류를 돌려준다. 가져올 개수는 최대 30개로 제한된다. 시점 필터는 과거 epoch 초로 들어오던 값을 현재의 ISO 8601 생성 시각과 견주어 비교한다. 정렬은 ISO 8601 문자열이 사전순=시간순이라는 점을 이용해 최신이 위로 오도록 한다.

## provenance

도구 정의와 필터·정렬·매핑 로직은 `cli/src/mcp.rs`의 `clawket_get_recent_decisions`에 있다. 지식 목록 조회는 데몬 HTTP API(데몬의 지식 라우트, `endpoint.daemon-search`)에 위임하고, CLI 내장 MCP stdio 서버(`component.mcp-server`)가 노출한다.

## 미확정 (OPEN)
- [ ] OPEN: 데몬의 `/knowledge?type=decision` 응답이 plan_id/unit_id/created_at 을 항상 포함하는지, plan_id 필터가 데몬단에서 지원되는지 데몬 라우트 정독으로 확인.
