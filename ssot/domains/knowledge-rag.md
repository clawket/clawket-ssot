---
id: domain.knowledge-rag
kind: Domain
title: 지식·로컬 RAG
purpose: "세션을 넘어 사라지지 않는 프로젝트 지식을 보존하고, 의미 기반 검색으로 과거 맥락을 다시 끌어와 LLM 코딩 에이전트가 매번 처음부터 시작하지 않게 한다."
definition: "위키성 지식 항목을 버전 관리하고, 태스크와 지식을 온디바이스 임베딩으로 색인해 키워드(FTS5)와 의미(벡터) 검색을 결합한 하이브리드 검색을 제공하는 영역. 검색·임베딩·저장 전부가 로컬 데몬 안에서 일어나며, 어떤 데이터도 외부 벡터 DB로 나가지 않는다. 모델 라우팅·훅 차단·검증 판정은 이 영역에 포함되지 않는다."
servesPersona:
  - persona.llm-coding-agent
  - persona.human-operator
relatesTo:
  - to: domain.work-management
    type: depends-on
    note: "지식 항목은 task/unit/plan에 첨부되고, 태스크 자체도 임베딩 대상이라 작업 관리 영역의 엔티티 위에서 동작한다."
  - to: domain.pdd-verification
    type: feeds
    note: "검증 루프가 산출하는 시나리오·TSV 근거·라운드 결과를 지식 항목으로 저장하고, 다음 라운드가 이를 다시 검색해 참조한다."
governedBy:
  - invariant.local-only-no-egress
  - invariant.scenario-snapshot-only
realizedBy:
  - component.daemon
  - component.embedding-engine
  - component.sqlite-storage
  - component.mcp-server
impacts:
  - capability.rag-pull
  - capability.wiki-knowledge
  - capability.session-restore
  - capability.search-knowledge-tasks
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:domain"]
---

## 목적

LLM 코딩 에이전트는 세션이 끊기면 직전까지 쌓은 맥락을 잃는다. 이 영역은 그 맥락을 로컬에 영구 보존하고, 새 세션에서 "예전에 비슷한 걸 했었다"를 다시 찾아낼 수 있게 한다. 의사결정 기록, 메모, 검증 근거, 참고 문서 같은 위키성 지식을 작업 엔티티에 붙여 두고, 필요할 때 의미가 가까운 항목을 검색해 끌어온다. 핵심 약속은 이 모든 일이 사용자 기기 안에서만 일어난다는 점이다 — 외부 벡터 데이터베이스나 클라우드 임베딩 서비스에 의존하지 않는다.

## 경계와 핵심 개념

포함:
- 지식 항목(Knowledge) — task/unit/plan에 첨부되는 버전 관리 문서. `decision`/`note`/`evidence`/`reference` 같은 유형을 가진다.
- 임베딩 — 지식 항목은 생성/수정 시(본문이 있을 때), 태스크는 생성/수정 시 자동으로 384차원 벡터로 변환된다. 다국어(50개 이상 언어) 모델을 쓰므로 한국어·일본어·영어가 섞인 항목도 같은 공간에서 비교된다. 데몬 시작 시 비어 있는 임베딩은 백필된다.
- 하이브리드 검색 — 키워드 검색(전문 인덱스)과 의미 검색(벡터 근접 이웃)을 결합한다. 같은 검색 경로를 웹 위키, CLI 검색 명령, MCP 서버가 공유한다.

제외:
- 작업 상태 전이·라이프사이클 규칙(작업 관리 영역).
- 검증 라운드의 수렴 판정 로직(PDD 검증 영역) — 단, 그 산출물을 지식으로 저장하는 것은 이 영역.
- 훅 기반 차단·가드레일(훅 강제 영역).

## 기능

- 위키 지식 CRUD와 버전 관리, 그리고 하이브리드 검색(`capability.wiki-knowledge`, `capability.search-knowledge-tasks`).
- MCP 도구를 통한 과거 컨텍스트 풀 — 에이전트가 필요할 때 지식·태스크·최근 결정을 의미 검색으로 당겨온다(`capability.rag-pull`).
- 세션 복원 — 새 세션에서 직전 작업 맥락을 다시 확보한다(`capability.session-restore`).

이 기능들은 데몬(검색·자동 임베딩·백필), 임베딩 엔진(온디바이스 벡터화), SQLite 저장소(지식·벡터 보관), MCP 서버(읽기 전용 노출)로 실현된다.

## 시스템 흐름

지식 항목을 만들거나 고치면 데몬이 본문을 받아 저장하고, 본문이 있을 때 그 자리에서 임베딩을 계산해 벡터로 함께 보관한다. 태스크도 마찬가지로 생성·수정 시 임베딩된다. 데몬이 켜질 때는 아직 임베딩이 없는 항목을 찾아 백필한다.

검색 요청이 들어오면 데몬은 키워드 일치(전문 검색)와 의미 근접(벡터 최근접 이웃)을 함께 평가하는 하이브리드 결과를 돌려준다. 이 검색 경로는 한 곳에 구현되어 웹 위키 화면, CLI 검색 명령, 그리고 에이전트가 호출하는 MCP 도구가 모두 같은 결과를 받는다. 임베딩 모델·벡터·검색이 전부 데몬 프로세스 안에서 돌기 때문에 검색 질의나 항목 내용이 기기 밖으로 나가지 않는다.

## 다른 도메인과의 관계

- 작업 관리 영역에 의존한다(`depends-on`) — 지식은 작업 엔티티에 매달리고 태스크 자체가 색인 대상이라, 작업 관리의 구조 없이는 성립하지 않는다.
- PDD 검증 영역에 자료를 공급한다(`feeds`) — 시나리오·검증 근거·라운드 결과가 지식으로 쌓이고, 이어지는 라운드와 세션이 그것을 다시 검색해 참조한다.

## 미확정 (OPEN)
- [ ] OPEN: 지식 항목 유형(type) 전체 목록과 각 유형의 의미가 코드/문서에서 완전히 열거되는지 미검증.
- [ ] OPEN: 하이브리드 검색에서 키워드 점수와 의미 점수를 결합하는 가중 방식의 정확한 규칙 미확정.
