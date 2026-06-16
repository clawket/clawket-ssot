---
id: component.mcp-server
kind: SystemComponent
title: 임베디드 MCP stdio 서버
definition: CLI 바이너리 안에 함께 들어 있는 MCP(stdio) 서버로, Claude Code가 필요할 때 Clawket의 지식·태스크 컨텍스트를 직접 끌어올 수 있게 5개의 읽기 전용 도구를 노출한다. 데몬이 쓰는 포트를 캐시 파일에서 스스로 찾아 데몬 HTTP API를 호출하며, 어떤 데이터도 변경하지 않는다.
purpose: "LLM 에이전트(Claude Code)가 세션 도중 능동적으로 과거 작업 맥락과 지식을 의미 검색해 끌어오게 한다. SessionStart의 푸시형 컨텍스트 주입을 보완하는 풀(pull)형 통로다."
realizedBy:
  - capability.rag-pull
  - domain.knowledge-rag
implementedIn:
  - cli/src/mcp.rs
dependsOn:
  - component.cli
  - component.daemon
consumesApi:
  - endpoint.daemon-search
providesApi:
  - endpoint.mcp-search-knowledge
  - endpoint.mcp-search-tasks
  - endpoint.mcp-find-similar-tasks
  - endpoint.mcp-get-task-context
  - endpoint.mcp-get-recent-decisions
integratesWith:
  - integration.mcp
impacts:
  - persona.llm-coding-agent
  - capability.rag-pull
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:systemcomponent"]
relatesTo:
  - to: component.cli
    type: belongs-to
    note: 별도 바이너리가 아니라 clawket CLI 바이너리에 내장되어 `clawket mcp` 서브커맨드로 기동된다.
  - to: component.mcp-launcher
    type: depends-on
    note: 플러그인 .mcp.json이 가리키는 노드 런처가 이 서버를 stdio 자식 프로세스로 띄운다.
  - to: integration.mcp
    type: realizes
    note: 이 서버가 곧 MCP(stdio) 연동의 실체다.
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 분기/의도 불명은 OPEN. -->

## 책임

이 구성 요소는 Claude Code가 대화 도중 "필요할 때 직접" Clawket의 지식과 태스크 맥락을 끌어오게 해 주는 다리다. Clawket은 두 가지 방식으로 컨텍스트를 에이전트에 전달한다. 하나는 세션이 시작될 때 자동으로 밀어 넣는 방식(푸시)이고, 다른 하나가 바로 이 MCP 서버를 통해 에이전트가 스스로 끌어오는 방식(풀)이다. 풀 방식 덕분에 에이전트는 지금 작업과 의미가 가까운 과거 결정·태스크·지식을 그때그때 검색해 가져올 수 있다.

서버가 노출하는 도구는 다섯 가지이며 모두 읽기 전용이다 — 어떤 데이터도 만들거나 바꾸지 않는다. 지식 검색, 태스크 검색, 유사 태스크 찾기(코멘트에서 결정·이슈 추출 포함), 태스크의 전체 맥락(연관 지식·관계·코멘트·활동 이력) 조회, 최근 결정(decision) 지식 조회가 그것이다. 이 "읽기 전용" 성격은 에이전트가 RAG 도구로 상태를 망가뜨릴 수 없게 하는 안전선이다.

## 경계와 의존

이 서버는 독립 바이너리가 아니라 CLI(`component.cli`) 바이너리 안에 내장되어 있고, `clawket mcp` 명령으로 stdio 모드로 기동된다. 실제 데이터는 데몬(`component.daemon`)이 쥐고 있으므로, 서버는 데몬의 검색 HTTP API(`endpoint.daemon-search`)를 호출해 결과를 가져온다. 데몬이 떠 있지 않으면 서버는 먼저 데몬을 자동 기동하려 시도하고, 그래도 실패하면 개별 도구 호출이 구조화된 오류로 응답한다(프로세스가 죽지 않는다).

데몬의 포트는 고정값이 아니라 캐시 디렉터리의 포트 파일에서 자동으로 찾아낸다. 플러그인 쪽에서는 `.mcp.json`이 MCP 런처(`component.mcp-launcher`)를 가리키고, 그 런처가 이 서버를 stdio 자식으로 띄워 Claude Code에 연결한다.

## 통신 패턴

바깥(Claude Code) 방향으로는 표준 입출력(stdio) 기반 MCP 프로토콜로 통신한다 — 도구 목록을 알리고, 도구 호출을 받아 결과를 돌려준다. 안쪽(데몬) 방향으로는 루프백 HTTP로 데몬의 검색·조회 API를 호출한다. 즉 이 서버 자체는 데이터를 들고 있지 않고, MCP 요청을 데몬 HTTP 호출로 번역해 주는 얇은 어댑터다. 서버가 자기소개에 명시하듯, 동작에는 데몬 실행이 전제된다.

## 하위 서브패키지 (책임 단위)

- MCP 핸들러·라우터: 다섯 개 도구의 입력 스키마를 정의하고 호출을 분배한다.
- 데몬 클라이언트: 루프백 HTTP로 데몬의 검색·조회 API를 호출하고 결과를 정리한다.
- 데몬 자동 기동 글루: 서버 시작 시 데몬이 없으면 띄우려 시도한다.
- 결과 정리: 스니펫 길이·반환 개수 상한 등 토큰 효율을 위한 출력 다듬기.

## 미확정 (OPEN)
- [ ] OPEN: 검색 모드(keyword/semantic/hybrid)별로 데몬의 어떤 검색 경로가 호출되는지의 1:1 대응은 데몬 검색 라우트 측에서 추가 확인 필요.
