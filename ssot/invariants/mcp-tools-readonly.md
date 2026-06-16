---
id: invariant.mcp-tools-readonly
kind: Invariant
title: MCP 도구는 전부 읽기 전용
definition: CLI에 내장된 MCP 서버가 노출하는 다섯 개의 도구는 모두 데몬에서 데이터를 읽기만 하며, 어떤 상태도 변경하지 않는다.
governs:
  - component.mcp-server
  - capability.rag-pull
governedBy: []
implementedIn:
  - cli/src/mcp.rs
decidedBy: []
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:invariant
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로. -->

## 제약
Clawket이 Claude Code에 제공하는 다섯 개의 MCP 도구 — 지식 검색, 태스크 검색, 유사 태스크 찾기, 태스크 컨텍스트 조회, 최근 결정 조회 — 는 전부 **읽기 전용**이다. 이들은 데몬에서 데이터를 가져오기만 할 뿐, 프로젝트·플랜·태스크·지식 등 어떤 상태도 생성·수정·삭제하지 않는다.

상태를 바꾸는 작업(태스크 시작/완료, 지식 작성 등)은 MCP 경로가 아니라 CLI 서브커맨드를 통해서만 일어나야 한다. MCP는 어디까지나 과거 컨텍스트를 의미 검색으로 끌어오는(RAG 풀) 통로다.

## 깨지면 무슨 일이 일어나나
MCP 도구가 쓰기를 할 수 있게 되면, 변경 작업이 CLI를 거치지 않고 곧장 데이터를 바꾸는 우회로가 생긴다. 그러면 활성 태스크 가드·라이프사이클 게이트·감사 로그 같은 워크플로우 안전장치를 LLM이 우회할 수 있어, "활성 태스크 없이는 변경 불가"라는 핵심 강제가 무너진다. read-only 보장은 MCP를 안전한 조회 채널로 한정해, 모든 변경이 추적되는 정규 경로(CLI + 훅)를 거치도록 만든다.

## 코드에서 어떻게 강제되나
내장 MCP 서버는 자신의 안내문에서 다섯 도구를 나열하고 "모두 읽기 전용 — 변경 없음"임을 명시한다(`cli/src/mcp.rs`의 서버 정보 안내문). 노출되는 도구 집합 자체가 검색·조회 계열만으로 구성되어 있고, 변경 작업은 도구로 제공되지 않는다 — 변경이 필요하면 별도 CLI 서브커맨드를 쓰라고 안내한다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(MCP read-only 정책의 근거 결정) 연결 필요
