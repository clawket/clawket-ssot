---
id: integration.mcp-claude-code
kind: Integration
title: Claude Code ↔ MCP·훅 어댑터 연동
purpose: "LLM 코딩 에이전트(Claude Code)가 Clawket의 과거 컨텍스트를 의미 검색으로 끌어오고, 훅을 통해 작업 가드레일을 받도록 — CLI가 MCP 서버와 훅 어댑터를 표준 규약으로 노출한다."
definition: "clawket CLI가 MCP stdio 서버를 띄워 Claude Code에 읽기 전용 RAG 도구를 제공하고, 함께 배포한 Claude Code 훅 어댑터(.cjs)들이 세션·작업 가드레일을 건다. doctor가 이 두 접점의 정상 등록 여부를 진단한다."
integratesWith:
  - component.cli
  - component.mcp-server
  - component.claude-adapter
  - component.mcp-launcher
consumedBy:
  - persona.llm-coding-agent
impacts:
  - endpoint.cli-mcp
  - endpoint.cli-doctor
  - endpoint.mcp-search-knowledge
  - endpoint.mcp-search-tasks
  - endpoint.mcp-find-similar-tasks
  - endpoint.mcp-get-task-context
  - endpoint.mcp-get-recent-decisions
relatesTo:
  - to: integration.mcp
    type: relates-to
    note: "플러그인 쪽 .mcp.json 등록과 짝을 이루는, CLI 쪽 MCP 서버 구현 관점이다."
  - to: domain.knowledge-rag
    type: belongs-to
    note: "에이전트가 세션 간 의미 컨텍스트를 복원하는 RAG 도메인의 외부 노출 통로다."
governedBy:
  - invariant.mcp-tools-readonly
  - invariant.mcp-response-cap
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:integration
---

## 무엇과 연동하나

상대는 LLM 코딩 에이전트인 Claude Code다. Clawket은 Claude Code에 두 가지 접점을 노출한다.

첫째는 MCP(Model Context Protocol) stdio 서버다. clawket CLI 안에 내장된 이 서버는 표준입출력으로 Claude Code와 대화하며, 다섯 가지 읽기 전용 RAG 도구를 제공한다: 지식 검색, 태스크 검색, 유사 태스크 찾기, 태스크 전체 컨텍스트 가져오기, 최근 결정 가져오기. 이 도구들로 에이전트는 이전 세션에서 쌓인 의미 컨텍스트를 다시 끌어올 수 있다. 이름 그대로 전부 읽기만 하며 어떤 데이터도 바꾸지 않는다.

둘째는 Claude Code 훅 어댑터다. 플러그인은 Claude Code의 훅 이벤트마다 도는 .cjs 진입점들을 함께 배포해, 세션 컨텍스트 주입과 작업 가드레일을 건다(상세 동작은 별도 훅 연동 노드 참조).

## 구현 위치 (provenance)

MCP 서버의 자기소개와 도구 목록 선언은 `cli/src/mcp.rs`에 있다. 서버는 시작 정보에 "Clawket 읽기 전용 RAG 도구, clawketd 실행 필요"와 다섯 도구 이름을 명시하고, 시작 전 데몬을 먼저 보장한 뒤 표준입출력으로 서비스를 연다. v3에서 실행·봉투 계열 쓰기 도구들은 제거되어, 노출 도구는 읽기 전용으로 좁혀졌다.

Claude Code가 이 서버를 어떻게 띄우는지는 플러그인 쪽 `.mcp.json`과 노드 런처(`component.mcp-launcher`)가 정하고, 런처가 CLI의 `mcp` 서브커맨드를 stdio로 기동한다.

doctor 명령은 두 접점을 진단한다. `cli/src/doctor.rs`의 MCP 섹션은 `clawket mcp` 서브커맨드 도달 가능성과 도구 목록을 점검하고, Hooks 섹션은 플러그인의 `adapters/claude/` 아래 일곱 개 훅 핸들러 파일(세션 시작, 프롬프트 제출, 도구 전/후, 플랜 동기화, 서브에이전트 시작/종료)이 존재하는지 확인한 뒤, 각 Claude Code 훅 이벤트별로 선언된 핸들러 경로와 마지막 발화 시각까지 읽어 보여 준다.

이 연동을 소비하는 주체는 LLM 코딩 에이전트(`persona.llm-coding-agent`)이고, 내부적으로는 CLI·MCP 서버·Claude 어댑터·MCP 런처가 함께 구성한다.

## 불변식

- MCP가 노출하는 다섯 도구는 모두 읽기 전용이어야 한다(`invariant.mcp-tools-readonly`). 에이전트가 MCP 경유로 상태를 바꾸는 일은 없으며, 쓰기는 CLI 서브커맨드로만 한다.
- MCP 도구 응답 크기에는 상한이 걸려 있다(`invariant.mcp-response-cap`). Claude Code가 큰 도구 결과를 조용히 잘라버리기 전에, 깔끔한 "잘림" 안내로 끊어 준다.
- MCP 서버는 데몬이 떠 있어야 동작한다 — 시작 시 데몬을 먼저 보장한다.

## 영향 범위

이 연동의 동작은 MCP 서버 기동 명령(`endpoint.cli-mcp`), 진단 명령(`endpoint.cli-doctor`), 그리고 다섯 개 MCP 도구 엔드포인트 전부에 직접 닿는다. 도구가 바뀌거나 읽기 전용 원칙이 깨지면 에이전트의 컨텍스트 복원 경로와 가드레일 신뢰가 동시에 흔들린다.

## 미확정 (OPEN)
- [ ] OPEN: 훅 어댑터 연동을 이 노드와 `integration.claude-hooks` 중 어느 쪽이 주(主)로 다룰지 경계 확정
- [ ] OPEN: MCP 응답 상한의 정확한 정책(임계치·잘림 메시지 형식)을 불변식 노드에서 확정
