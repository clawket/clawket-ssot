---
id: domain.context-persistence
kind: Domain
title: 세션 컨텍스트 지속
purpose: "Claude Code 세션이 기본적으로 무상태라서 매번 '어제 어디까지 했지?'를 다시 설명해야 하는 문제를, 작업 상태·이력·계획·의사결정을 로컬에 보존하고 세션 시작 시 자동으로 되살리는 것으로 해결한다."
definition: "한 세션에서 만들어진 작업 맥락(무엇을·왜·어디까지 했는지)을 다음 세션으로 잇는 책임 영역. 시작 시 대시보드를 자동으로 밀어 넣는 push 경로와, 필요할 때 과거 맥락을 의미 검색으로 끌어오는 pull 경로 두 가지로 컨텍스트 연속성을 만든다. 무엇을 어떻게 보존하느냐(저장 메커니즘 자체)는 작업 관리·지식 RAG 도메인의 책임이며, 본 도메인은 그 보존물을 '세션 경계를 넘겨 다시 살아나게' 하는 흐름에 한정한다."
servesPersona: [persona.llm-coding-agent, persona.human-operator]
relatesTo:
  - { to: domain.work-management, type: depends-on, note: "보존·복원되는 맥락의 실체(프로젝트·플랜·태스크 상태)는 작업 관리 도메인이 정의한다" }
  - { to: domain.knowledge-rag, type: depends-on, note: "pull 경로의 의미 검색은 지식·로컬 RAG 도메인의 임베딩·하이브리드 검색을 사용한다" }
governedBy: []
realizedBy: [component.daemon, component.sqlite-storage, component.mcp-server]
impacts: [persona.llm-coding-agent, capability.rag-pull, endpoint.hook-session-start, endpoint.cli-dashboard]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:domain"]
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로 — 무엇을·왜·누가·어떤 규칙·무슨 데이터. 코드 옮겨적기 금지. -->

## 목적

Claude Code는 세션을 새로 열 때마다 백지에서 시작한다. 이전 세션에서 무슨 일을 했는지, 왜 그렇게 결정했는지, 어디까지 진행했는지에 대한 기억이 사라진다. 그 결과 사용자는 매 세션마다 "어제 어디까지 했지?"를 다시 설명해야 하고, 그 설명에만 적지 않은 토큰과 시간이 들어간다.

이 도메인은 그 무상태성을 메우는 것을 책임진다. 작업 상태·변경 이력·계획·과거 의사결정을 로컬에 보존해 두고, 새 세션이 시작될 때 자동으로 되살려 에이전트가 곧장 이어서 일할 수 있게 한다. 핵심 가치는 "세션이 끊겨도 작업 맥락은 끊기지 않는다"이다.

## 경계와 핵심 개념

이 도메인이 다루는 것은 맥락의 **연속성**이다. 두 갈래로 동작한다.

- **Push(자동 주입)**: 세션이 시작되면 그 작업 디렉터리에 해당하는 프로젝트의 작업 요약(활성 플랜·유닛·사이클·진행 중 태스크)을 자동으로 세션 맥락에 밀어 넣는다. 에이전트가 묻지 않아도 "지금 상태"가 먼저 주어진다.
- **Pull(필요 시 조회)**: 에이전트가 작업 도중 과거의 비슷한 작업·결정·지식이 필요할 때, 의미 검색으로 관련 맥락만 골라 끌어온다. 한꺼번에 전부 주입하지 않고 필요한 만큼만 가져오는 것이 push와의 차이다.

경계 밖(이 도메인의 책임이 아닌 것):
- 맥락의 실체가 무엇이고 어떻게 구조화·저장되는지는 작업 관리(`domain.work-management`)와 지식·로컬 RAG(`domain.knowledge-rag`)의 책임이다. 본 도메인은 그 보존물을 세션 경계 너머로 잇는 흐름만 다룬다.
- 누가·언제·무엇을 바꿨는지의 변조 탐지 기록은 거버넌스·감사(`domain.governance-audit`)의 책임이다.

## 기능

- **세션 시작 자동 복원**: 새 세션에서 작업 대시보드를 자동으로 주입한다(`endpoint.hook-session-start` → `endpoint.cli-dashboard`).
- **과거 맥락 RAG 풀**: 의미 검색으로 관련 지식·태스크·결정을 끌어온다(`capability.rag-pull`).

## 시스템 흐름

Push 경로: 세션이 시작되면 시작 훅이 현재 작업 디렉터리를 프로젝트로 해석하고, 데몬에 그 프로젝트의 작업 요약을 요청해 세션 맥락에 주입한다. 에이전트는 별도 질의 없이 "지금 활성 플랜·진행 중 태스크"를 갖고 출발한다.

Pull 경로: 작업 중 과거 맥락이 필요하면 에이전트가 의미 검색 도구를 호출한다. 데몬이 온디바이스 임베딩으로 관련도 높은 항목만 골라 돌려주고, 에이전트는 그 일부만 맥락에 끌어온다. 보존물 자체는 단일 로컬 저장소(`component.sqlite-storage`)에 있고, 데몬이 push·pull 양쪽의 관문 역할을 한다.

## 다른 도메인과의 관계

- 작업 관리(`domain.work-management`)에 의존한다 — 복원되는 맥락(프로젝트·플랜·태스크의 상태)은 작업 관리가 정의·보관하는 데이터다.
- 지식·로컬 RAG(`domain.knowledge-rag`)에 의존한다 — pull 경로의 의미 검색은 그 도메인의 임베딩·하이브리드 검색 위에서 동작한다.

## 미확정 (OPEN)
- [ ] OPEN: push 주입과 pull 조회의 "맥락 양"을 조절하는 정책(과도 주입을 막는 기준)이 명문화되어 있는지 미확인. BLOG.md는 "과도한 컨텍스트 주입이 오히려 방해가 된다"는 고민을 기록하고 있어, 이 경계 규칙이 향후 invariant로 분리될 여지가 있다.
