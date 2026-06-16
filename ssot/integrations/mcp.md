---
id: integration.mcp
kind: Integration
title: MCP(stdio) 연동
definition: 플러그인이 등록하는 MCP 서버 선언을 통해 Clawket의 임베디드 stdio MCP 서버를 Claude Code에 붙여, 코딩 에이전트가 도구 호출로 read-only 지식·태스크 RAG를 끌어올 수 있게 하는 연동.
purpose: 코딩 에이전트가 세션 사이에 끊긴 의미 컨텍스트(과거 결정·관련 태스크·관련 지식)를 매번 사람에게 다시 설명받지 않고 도구 호출 한 번으로 복원하기 위해 필요하다. 훅이 "쓰기·차단"의 표면이라면 MCP는 "읽기·검색"의 표면이다.
integratesWith:
  - component.mcp-launcher
  - component.mcp-server
  - component.plugin-shell
implementedIn:
  - clawket/.mcp.json
  - clawket/scripts/mcp-launch.cjs
impacts:
  - endpoint.cli-mcp
  - endpoint.mcp-search-knowledge
  - endpoint.mcp-search-tasks
  - endpoint.mcp-find-similar-tasks
  - endpoint.mcp-get-task-context
  - endpoint.mcp-get-recent-decisions
  - persona.llm-coding-agent
relatesTo:
  - to: domain.knowledge-rag
    type: realizes
    note: 지식·로컬 RAG 영역을 Claude Code에 노출하는 채널
  - to: integration.claude-code-hooks
    type: relates-to
    note: 같은 플러그인 셸이 등록하는 또 다른 Claude Code 표면(MCP=읽기, 훅=쓰기·차단)
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

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로 — 무엇을·왜·누가·어떤 규칙·무슨 데이터. 코드(테이블·필드·경로·SQL) 옮겨적기 금지(식별자는 provenance/근거에만). 코드 분기/의도 불명은 판정 말고 OPEN. -->

## 무엇과 연동하나

상대편은 Claude Code의 MCP(Model Context Protocol) 클라이언트다. Clawket은 자신의 RAG 검색 능력을 MCP "도구" 묶음으로 노출하고, Claude Code는 그 도구들을 일반 도구 호출처럼 불러 쓴다.

연결 방식은 표준 입출력(stdio) 기반이다. 별도 포트를 열거나 네트워크 인증을 하는 대신, 플러그인이 등록한 MCP 서버 선언을 따라 Claude Code가 로컬에서 노드 런처를 띄우고, 그 런처가 Clawket CLI의 MCP 서브커맨드를 stdio 서버로 기동한다. 서버는 실행 중인 로컬 데몬의 포트를 자동으로 찾아 데몬의 검색 API에 붙는다.

노출되는 도구는 모두 읽기 전용으로, 지식 검색·태스크 검색·유사 태스크 찾기·태스크 풀컨텍스트 조회·최근 결정 조회의 다섯 가지다. 이들은 데이터를 바꾸지 않으며 오직 컨텍스트를 끌어오기만 한다.

## 구현 위치 (provenance)

MCP 서버를 Claude Code에 등록하는 선언은 플러그인 셸의 MCP 매니페스트(`clawket/.mcp.json`)에 있다. 이 선언은 서버 유형을 stdio로 지정하고, 노드로 실행할 런처 스크립트를 가리키며, 플러그인 루트는 환경 변수로 주입된다.

그 런처(`clawket/scripts/mcp-launch.cjs`, 인벤토리상 `component.mcp-launcher`)가 Clawket CLI의 MCP 서브커맨드(`endpoint.cli-mcp`)를 stdio로 기동하면, CLI에 내장된 MCP 서버(`component.mcp-server`)가 데몬을 자동 발견해 다섯 개 read-only 도구를 노출한다. 패키징·배포 주체는 플러그인 셸(`component.plugin-shell`)이다.

즉 등록(매니페스트) → 런처 → CLI 임베디드 서버 → 데몬 검색의 사슬로 이어지며, 이 노드는 그 첫 두 단계(등록과 런처)를 책임지는 연동이다.

## 불변식

- MCP로 노출되는 도구는 전부 읽기 전용이어야 한다(`invariant.mcp-tools-readonly`). 어떤 도구도 상태를 변경해서는 안 된다 — 쓰기·차단은 훅 쪽 책임이다.
- 응답 크기는 상한이 걸려 있어야 한다(`invariant.mcp-response-cap`). RAG 풀이 컨텍스트 윈도를 폭주시키지 않도록 캡을 둔다.

## 영향 범위

이 연동이 등록되지 않거나 런처가 실패하면 다섯 개 MCP 도구(지식 검색·태스크 검색·유사 태스크·태스크 컨텍스트·최근 결정)가 Claude Code에서 사라진다. 그 결과 코딩 에이전트(`persona.llm-coding-agent`)의 세션 간 의미 컨텍스트 복원이 끊긴다. 데몬이 떠 있지 않으면 서버가 검색 백엔드를 찾지 못하므로, 데몬 가동이 이 연동의 선행 조건이다.

## 미확정 (OPEN)
- [ ] OPEN: 데몬 포트 자동 발견이 실패했을 때(데몬 미가동) MCP 서버가 어떤 응답/에러를 반환하는지 정밀 동작은 endpoint.cli-mcp / component.mcp-server 노드에 위임.
