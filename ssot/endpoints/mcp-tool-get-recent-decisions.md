---
id: endpoint.mcp-tool-get-recent-decisions
kind: Endpoint
title: MCP 도구 clawket_get_recent_decisions (구현)
definition: "최근에 기록된 결정(type=decision 지식 항목)을 최신순으로 목록 조회하는 읽기 전용 MCP 도구. 세션 시작 시 \"지난 회차에 어떤 결정이 있었지?\"를 키워드 없이 타입 기반으로 확인하는 용도다."
realizedBy: [component.mcp-server]
implementedIn: ["cli/src/mcp.rs"]
relatesTo:
  - { to: concept.knowledge, type: reads, note: "유형이 decision 인 지식 항목만 걸러 최신순으로 읽어 반환한다" }
  - { to: concept.plan, type: reads, note: "plan_id 를 주면 특정 플랜에 속한 결정으로 한정한다" }
  - { to: endpoint.mcp-get-recent-decisions, type: relates-to, note: "동일 도구의 README 기준 인벤토리 노드 — 본 노드는 mcp.rs 구현 기준 상세" }
  - { to: endpoint.mcp-search-knowledge, type: relates-to, note: "키워드 의미 탐색은 search-knowledge, 타입 기반 결정 목록은 본 도구로 역할 분담" }
governedBy: []
impacts: [capability.rag-pull]
consumedBy: [persona.llm-coding-agent]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
프로젝트에서 그동안 내려진 "결정"만 골라 최신순으로 보여주는 읽기 전용 도구다. 지식 항목 중 유형이 결정(decision)으로 표시된 것만 추려 준다. 세션을 새로 시작한 LLM 에이전트가 "지난 회차에서 무엇이 결정됐는지"를 한눈에 훑어 맥락을 복원하는 데 쓴다. 자연어 키워드로 찾는 검색이 아니라, 결정이라는 유형으로 필터링하는 목록 조회라는 점이 핵심이다. 특정 주제를 키워드로 뒤지고 싶으면 지식 검색 도구를 쓰고, "최근 결정 흐름"을 보고 싶으면 이 도구를 쓴다.

## 요청 / 응답
요청에는 세 가지 선택 항목을 줄 수 있다: (1) 특정 플랜에 속한 결정만 보고 싶을 때 그 플랜 식별자, (2) 몇 개까지 받을지 개수, (3) 특정 시점 이후에 생성된 결정만 보고 싶을 때의 기준 시각. 개수를 주지 않으면 기본값이 적용되고 상한이 있다(상한을 넘겨 요청해도 상한선으로 잘린다).

응답은 결정 항목들의 목록이며, 각 항목은 제목·소속 플랜·소속 유닛·생성 시각, 그리고 내용 앞부분 요약(스니펫)을 담는다. 목록은 생성 시각 기준 최신이 먼저 오도록 정렬되어 있고, 요청한 개수만큼 잘려 나온다. 본문 전체가 아니라 결정 미리보기 분량만 주므로 세부가 필요하면 해당 지식 항목을 따로 조회해야 한다.

## 권한 / 제약
읽기 전용으로 상태를 바꾸지 않으며, 동작하려면 로컬 데몬이 떠 있어야 한다. 반환 개수에는 상한이 걸려 있어 한 번에 무한정 끌어오지 못한다. 시점 필터는 기준 시각 이후에 만들어진 결정만 남기도록 동작한다. 모든 데이터는 로컬에 머문다.

## provenance
구현은 CLI 바이너리에 내장된 MCP 서버 안에 있다(`cli/src/mcp.rs`의 `clawket_get_recent_decisions`). 데몬의 지식 조회 경로를 유형=decision 으로 필터해 호출하고, 받은 목록을 시점 필터·최신순 정렬·개수 제한을 거쳐 제목·플랜·유닛·시각·요약 형태로 축약해 반환한다. 제공 컴포넌트는 임베디드 MCP stdio 서버(component.mcp-server)다.

## 미확정 (OPEN)
- [ ] OPEN: 시점 필터(기준 시각) 인자의 단위·해석이 호출 측 기대와 일관적인지 owner 확인 필요 — 인자 설명과 실제 비교 처리(ISO 8601 created_at 대비)의 정합성이 코드 주석상 과거 epoch 호환 처리로 남아 있어 의미가 모호하다.
