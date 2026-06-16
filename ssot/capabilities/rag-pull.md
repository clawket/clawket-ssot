---
id: capability.rag-pull
kind: Capability
title: 과거 컨텍스트 RAG 풀
definition: LLM 에이전트가 새 세션에서 필요할 때, 읽기 전용 MCP 도구로 과거의 지식·태스크·결정을 의미 기반으로 검색해 끌어와 컨텍스트를 복원하는 기능. 세션 시작 시의 자동 주입(push)을 보완하는, 에이전트가 스스로 당겨오는(pull) 경로다.
purpose: 세션이 끊겨도 "지난번에 어디까지 했는지", "예전에 이 설계를 왜 그렇게 결정했는지"를 외부 벡터 DB 없이 로컬에서 복원해, 매 세션이 백지에서 시작하는 문제를 없앤다.
servesPersona:
  - persona.llm-coding-agent
realizedBy:
  - component.mcp-server
  - component.daemon
  - component.embedding-engine
implementedIn:
relatesTo:
  - to: concept.knowledge
    type: reads
    note: 지식 항목을 의미·키워드·하이브리드로 검색해 끌어온다
  - to: concept.task
    type: reads
    note: 과거 태스크와 유사 태스크의 이웃·전체 컨텍스트를 조회한다
  - to: domain.knowledge-rag
    type: belongs-to
    note: 지식·로컬 RAG 영역이 제공하는 검색 위에서 동작한다
impacts:
  - concept.knowledge
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:capability
---

## 사용자가 할 수 있는 일

LLM 에이전트는 새 세션을 시작할 때 자동으로 프로젝트 대시보드와 규칙을 주입받지만, 그것만으로는 부족한 순간이 있다 — "예전에 인증 재시도 정책을 어떻게 결정했더라?", "이 주제와 비슷하게 다뤘던 태스크가 있었나?" 같은 질문이다. 이 기능은 에이전트가 그런 과거 맥락을 필요할 때 스스로 끌어올 수 있게 한다. 키워드뿐 아니라 의미가 가까운 항목까지 찾아주므로, 표현이 달라도 같은 주제의 과거 지식·결정을 회수한다.

핵심 가치는 세션을 가로지르는 기억이다. 그리고 그 기억은 어디로도 나가지 않는다 — 검색도 임베딩도 로컬 데몬 안에서 일어나, 외부 벡터 서비스로 데이터를 보내지 않는다. 자동 주입이 "들이미는" 컨텍스트라면, 이 기능은 에이전트가 판단해 "당겨오는" 컨텍스트다.

## 행위

- 지식 의미 검색: 위키성 지식 항목을 의미/키워드/하이브리드로 검색한다.
- 태스크 의미 검색: 과거 태스크를 같은 방식으로 검색한다.
- 유사 태스크 탐색: 시드 태스크와 의미가 가까운 이웃을 찾고, 그 코멘트에서 결정·이슈를 추출해 본다.
- 태스크 전체 컨텍스트 조회: 특정 태스크와 그에 딸린 지식·관계·코멘트·활동 이력을 한 묶음으로 받는다.
- 최근 결정 회수: 결정(decision) 유형의 지식 항목을 최신순으로 받아 과거 의사결정을 복원한다.

## 시스템 흐름

에이전트가 과거 맥락이 필요하다고 판단하면, MCP 도구 호출로 데몬에 검색을 요청한다. 데몬은 키워드 검색과 온디바이스 임베딩 기반 의미 검색을 결합해 가장 관련 있는 지식·태스크를 골라 돌려주고, 에이전트는 그 결과를 현재 작업의 컨텍스트로 받아들인다. 모든 임베딩과 검색은 로컬에서 수행되며, 이 도구들은 읽기 전용이라 어떤 상태도 바꾸지 않는다.

## 어디에 구현되어 있나

이 기능은 CLI 바이너리에 내장된 MCP stdio 서버가 노출하는 다섯 개의 읽기 전용 도구로 실현되며, 실제 검색은 로컬 데몬의 하이브리드 검색(키워드+의미)과 온디바이스 임베딩 엔진이 수행한다. 같은 검색 경로는 CLI의 RAG 별칭 명령과 웹 위키에서도 공유된다. 도구·검색의 구체 레포 경로는 인벤토리 수준에서 확정되지 않아 별도 노드(MCP 서버·데몬 검색)로 추적한다.

## 미확정 (OPEN)
- [ ] OPEN: 다섯 MCP 도구 핸들러의 clawket 레포 기준 실제 상대경로(CLI src/mcp.rs 추정) 확인 필요
- [ ] OPEN: pull(MCP 도구)과 동등한 CLI 별칭(find-similar / get-task-context / get-recent-decisions)의 관계 — 같은 데몬 경로를 공유하는지 명시 필요
