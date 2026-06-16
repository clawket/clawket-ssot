---
id: platform.clawket
kind: Platform
title: Clawket
definition: LLM 코딩 에이전트를 위한 구조화된 작업 계약 도구. Jira+Confluence를 대체하는 로컬 우선 상태 계층으로, 프로젝트·플랜·유닛·태스크·지식·실행 이력을 로컬 SQLite와 경량 데몬으로 세션 간 영속화하고, 훅 가드레일로 "활성 태스크 없이는 코드 수정 불가"를 강제한다.
purpose: LLM 코딩 에이전트가 세션을 넘나들어도 "어디까지 했는지"를 잃지 않게 하고, 모든 변경이 명시적 작업에 묶여 추적·복원 가능하도록 만든다.
value: 활성 태스크가 없으면 코드 변경 자체가 차단되므로, 에이전트의 모든 행동이 계획·근거·이력에 자동으로 연결된다. 컨텍스트는 사라지지 않고 다음 세션에서 의미 검색으로 복원된다.
servesPersona:
  - persona.llm-coding-agent
  - persona.human-operator
  - persona.qa-reasoner-subagent
governs:
  - domain.work-management
  - domain.knowledge-rag
  - domain.hook-enforcement
  - domain.pdd-verification
relatesTo:
  - to: platform.claude-code-plugin
    type: realizes
    note: Clawket는 Claude Code 플러그인 형태로 배포·활성화된다.
  - to: platform.local-first
    type: relates-to
    note: 모든 데이터가 로컬에 머무는 로컬 우선 원칙이 Clawket 동작 전반을 제약한다.
confidence: inferred
lastVerified: 2026-06-04
lifecycle: active
owner: TBD
tags:
  - status:active
  - type:platform
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 식별자는 provenance/근거에만. -->

## 무엇인가

Clawket은 LLM 코딩 에이전트(주로 Claude Code)를 위한 **구조화된 작업 계약 도구**다. 사람이 쓰던 Jira(작업 추적)와 Confluence(지식 문서)의 역할을, 에이전트가 직접 읽고 쓸 수 있는 단일 상태 계층으로 합쳐 대체한다.

핵심 발상은 단순하다. 에이전트가 파일을 고치거나 명령을 실행하기 직전에 "지금 진행 중인 활성 태스크가 있는가?"를 묻고, 없으면 그 변경 자체를 막는다. 이 한 줄의 제약 덕분에 — 무엇을 고쳤는지, 누가(어떤 서브에이전트가) 했는지, 어떤 계획의 일부인지가 전부 자동으로 기록되고 연결된다. 칸반 보드·타임라인·위키 같은 나머지 기능은 본질적으로 그 기록을 사람에게 보여 주는 뷰일 뿐이다.

작업은 **프로젝트 → 플랜 → 유닛 → 사이클 → 태스크** 계층으로 구조화된다. 플랜은 승인(approve)되어야 active가 되고, active 플랜의 태스크만 시작할 수 있다. 태스크는 직접 다루는 유일한 원자적 단위이며, done으로 끝내려면 근거(evidence)가 있어야 한다.

상태는 모두 로컬 SQLite 파일에 저장되고, 경량 데몬이 이를 단일 writer로 소유한다. 그 위에 온디바이스 임베딩 기반의 로컬 RAG(키워드 + 의미 하이브리드 검색)가 얹혀, 지난 세션의 결정·태스크·지식을 다음 세션에서 의미 검색으로 끌어올 수 있다.

## 누구를 위한 것인가

- **LLM 코딩 에이전트** (`persona.llm-coding-agent`): CLI와 MCP로 같은 상태를 읽고 쓰는 1차 사용자. 활성 태스크 없이는 변경 도구가 훅에 의해 차단되므로, 작업 전에 스스로 태스크를 만들고 진행 상태를 갱신하게 된다.
- **사람 운영자** (`persona.human-operator`): 웹 대시보드(Summary·Board·Backlog·Timeline·Wiki)로 에이전트와 동일한 상태를 보고, 칸반 드래그앤드롭으로 상태를 바꾸거나 백로그를 사이클에 배정하며 작업을 감독한다.
- **QA 추론 서브에이전트** (`persona.qa-reasoner-subagent`): 검증 라운드에서 시나리오를 코드에 대조해 근거를 산출하는, 배치 단위로 스폰되는 검증 전용 주체.

이들은 모두 같은 단일 상태를 공유한다 — 에이전트가 CLI로 쓴 것을 사람이 웹에서 보고, 사람이 웹에서 옮긴 것을 에이전트가 다음 세션에서 읽는다.

## 서비스 영역

Clawket이 다루는 영역은 네 갈래로 나뉜다(각 영역의 상세 규칙·동작은 해당 도메인 노드에 위임한다).

- **작업 관리** (`domain.work-management`): 프로젝트·플랜·유닛·사이클·태스크 워크플로우. 승인·활성화·완료·취소·차단 같은 상태 전이와 그 순서 규칙을 담당한다.
- **지식·로컬 RAG** (`domain.knowledge-rag`): 위키성 지식 항목의 버전 관리와, 온디바이스 임베딩을 이용한 키워드+의미 하이브리드 검색. 세션 간 의미 컨텍스트 복원의 토대다.
- **훅 강제(가드레일)** (`domain.hook-enforcement`): Claude Code 훅 이벤트마다 활성 태스크·플랜·사이클 상태와 안티패턴·파괴적 명령을 검사해 작업을 허용/차단한다. "태스크 없으면 변경 불가" 규칙이 실제로 강제되는 지점이다.
- **PDD 검증·수렴 루프** (`domain.pdd-verification`): 시나리오 작성 → 플랜 설계 → 배치 검증 → 근거 동기화 → 수렴 판정으로 코드와 시나리오를 함께 진화시키는 검증 방법론.

## 접근 매트릭스 (역할 → 서비스)

| 역할 | 작업 관리 | 지식·RAG | 훅 강제 | PDD 검증 |
|---|---|---|---|---|
| LLM 코딩 에이전트 | 생성·전이·완료(CLI) | 작성·검색·MCP 풀 | 강제 대상(차단·주입을 받음) | 라운드 실행·결함 수정·시나리오 정제 |
| 사람 운영자 | 검토·드래그앤드롭·승인 | 위키 열람·편집 | (강제 대상 아님, 프로젝트 enabled 토글로 켜고 끔) | 라운드 결과 검토 |
| QA 추론 서브에이전트 | (배정된 검증 태스크 한정) | 근거용 코드·지식 조회 | 배치 크기·근거 체크의 적용 대상 | 시나리오↔코드 대조, 근거 산출 |

## 기술 스택 (제품 맥락)

Clawket은 Claude Code 플러그인(`platform.claude-code-plugin`)으로 배포되지만, 플러그인 자체는 훅·프롬프트·스킬·MCP 등록만 담은 얇은 셸이다. 실제 런타임은 별도 바이너리로 분리되어 있다.

- **CLI** (`component.cli`): 단일 정적 Rust 바이너리. 작업 관리 명령 전부와, CLI에 내장된 MCP stdio 서버를 제공한다.
- **데몬** (`component.daemon`): Rust(axum+rusqlite) 로컬 데몬. 유닉스 소켓과 TCP로 HTTP API·SSE를 제공하고 SQLite를 단일 writer로 소유하며, 지식·태스크 임베딩을 자동 수행한다.
- **저장소** (`component.sqlite-storage`): SQLite + sqlite-vec. 모든 상태와 벡터 인덱스를 담는 단일 진실 저장소.
- **웹 대시보드** (`component.web-dashboard`): React 기반 SPA. 데몬 API·SSE에 붙어 5개 뷰를 렌더링한다.
- **연동**: Claude Code 훅(`integration.claude-code-hooks`)과 MCP stdio(`integration.mcp`)로 에이전트와 결합한다.

스택은 전부 로컬에서 동작하며, 임베딩 모델조차 온디바이스로 실행되어 어떤 데이터도 외부로 나가지 않는다(로컬 우선 원칙, `platform.local-first`).

## 미확정 (OPEN)

- [ ] OPEN: owner 미지정(현재 TBD).
- [ ] OPEN: 벤더 정책상 v3은 Claude 계열만 대상이며 티어 다운그레이드는 현재 경고(advisory)에 그친다 — v4+에서의 하드 강제·벤더 무관 어댑터 도입 시점은 SSOT에 미확정.
