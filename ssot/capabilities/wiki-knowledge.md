---
id: capability.wiki-knowledge
kind: Capability
title: 위키 지식 CRUD·검색
definition: 작업에 딸린 지식 문서를 파일 트리 형태의 위키에서 만들고·읽고·고치고·지우며 버전 이력으로 관리하고, 키워드(FTS5)와 의미(임베딩) 검색을 결합한 하이브리드 검색으로 찾아내는 기능.
purpose: 결정·노트·근거·참고 같은 지식을 작업 옆에 구조화해 쌓아 두고, 표현이 달라도 의미로 다시 찾을 수 있게 해 프로젝트 지식이 휘발되지 않게 한다.
servesPersona:
  - persona.human-operator
  - persona.llm-coding-agent
realizedBy:
  - component.web-dashboard
  - component.cli
  - component.daemon
  - component.embedding-engine
implementedIn:
relatesTo:
  - to: concept.knowledge
    type: mutates
    note: 지식 항목을 생성·수정·삭제하고 버전 이력으로 관리한다
  - to: domain.knowledge-rag
    type: belongs-to
    note: 지식·로컬 RAG 영역이 제공하는 임베딩·하이브리드 검색 위에서 동작한다
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

사람과 에이전트는 작업을 진행하며 쌓이는 지식 — 결정의 배경, 메모, 검증 근거, 참고 자료 — 을 위키에 문서로 남길 수 있다. 위키는 파일 트리처럼 탐색되고, 문서를 고치면 이전 내용이 버전 이력으로 보존된다. 핵심은 검색이다. 정확한 단어를 기억하지 못해도, 의미가 가까운 문서를 함께 찾아주므로 "그때 그 내용"을 다시 끌어낼 수 있다.

가치는 두 갈래다. 첫째, 지식이 작업 옆에 구조화돼 머문다 — 흩어진 메모나 휘발되는 채팅이 아니라 task/unit/plan에 붙은 버전 관리 문서로 남는다. 둘째, 그 지식은 사람의 위키 화면뿐 아니라 에이전트의 RAG 풀에서도 같은 검색 경로로 회수된다 — 사람과 LLM이 같은 지식 저장소를 공유한다.

## 행위

- 지식 생성·수정·삭제: 제목과 본문을 갖춘 지식 항목을 만들고 고치고 지운다.
- 유형 지정: 결정(decision)·노트(note)·근거(evidence)·참고(reference) 등 유형을 부여해 지식의 성격을 구분한다.
- 버전 이력 열람: 문서가 고쳐진 이력을 따라간다.
- 하이브리드 검색: 키워드(FTS5)와 의미(임베딩) 검색을 결합해 지식을 찾는다.
- 가져오기·내보내기: docs/ 같은 외부 문서와 지식을 주고받는다.
- 위키 탐색: 파일 트리 형태로 지식 구조를 둘러본다.

## 시스템 흐름

사람이 위키 화면에서 또는 에이전트가 명령으로 지식 항목을 만들거나 고치면, 데몬은 그 내용을 저장하면서 본문이 있는 경우 자동으로 임베딩해 의미 검색용 벡터를 만든다. 검색을 요청하면 데몬은 키워드 검색과 의미 검색을 함께 돌려 가장 관련 있는 항목을 돌려주며, 이 검색 경로는 웹 위키, CLI 검색 명령, 그리고 에이전트의 RAG 풀이 공통으로 사용한다. 모든 임베딩과 저장은 로컬에서 이뤄져 외부로 데이터가 나가지 않는다.

## 어디에 구현되어 있나

이 기능은 웹 대시보드의 위키 화면(파일 트리 탐색·버전 이력 편집)과 CLI의 지식 명령을 표면으로 하고, 실제 저장·임베딩·하이브리드 검색은 로컬 데몬과 온디바이스 임베딩 엔진이 수행한다. 위키 뷰 컴포넌트와 지식 라우트의 구체 레포 경로는 인벤토리 수준에서 확정되지 않아 별도 노드(웹 대시보드·데몬 검색·지식 엔드포인트)로 추적한다.

## 미확정 (OPEN)
- [ ] OPEN: 위키 뷰 컴포넌트(web/ 소스 트리)와 데몬 지식 라우트(daemon/src/routes/knowledge.rs 추정)의 실제 상대경로 확인 필요
- [ ] OPEN: 버전 이력 보존 방식(전체 스냅샷 vs 델타)과 보존 한도 확인 필요
