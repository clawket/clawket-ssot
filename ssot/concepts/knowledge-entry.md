---
id: concept.knowledge-entry
kind: Concept
title: 지식 항목(Knowledge Entry)
definition: "task/unit/plan에 첨부되는 위키 성격의 문서. doc · decision · reference · note · spec 중 하나의 유형을 가지며, 마크다운 본문을 담고, 부모 항목으로 위키 트리를 이루며, 임베딩되어 의미·키워드 하이브리드 검색으로 다시 찾을 수 있다."
relatesTo:
  - { to: concept.knowledge, type: mirrors, note: "인벤토리 상 concept.knowledge와 같은 위키 지식 개념을 CLI 구현 관점에서 본 것." }
  - { to: concept.task, type: reads, note: "지식 항목은 task에 첨부될 수 있다." }
  - { to: concept.plan, type: reads, note: "지식 항목은 plan에 첨부될 수 있다." }
  - { to: concept.unit, type: reads, note: "지식 항목은 unit에 첨부될 수 있다." }
governedBy:
  - invariant.mcp-tools-readonly
implementedIn:
  - cli/src/main.rs
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:concept"]
---

## 정의
지식 항목은 작업과 결정의 맥락을 글로 남기는 위키 문서다. 각 항목은 다음을 갖는다.

- **유형(type)**: doc(문서) · decision(결정) · reference(참고) · note(메모) · spec(명세) 중 하나. 결정 유형은 "최근 결정" 조회 같은 특수 흐름에서 따로 모인다.
- **본문**: 마크다운(기본) 형식의 내용. 텍스트·코드 형식도 가능.
- **부모 관계(parent)**: 다른 지식 항목을 부모로 지정해 계층적 위키 트리(폴더 같은 구조)를 이룬다.
- **첨부 대상**: task·unit·plan 중 최소 하나에 붙는다 — 떠다니는 문서가 아니라 항상 작업 맥락에 결속된다.

지식 항목의 핵심 가치는 "다시 찾을 수 있음"이다. 생성·갱신되면 온디바이스로 임베딩되어, 키워드 검색과 의미 검색을 결합한 하이브리드 검색으로 세션을 넘어 다시 끌어올 수 있다. LLM 에이전트는 이를 읽기 전용 MCP 도구로 조회해 과거 컨텍스트를 복원한다.

## 엔티티 (DB)
지식 항목은 제목·유형·본문·본문형식·부모 ID·첨부 대상(task/unit/plan)을 보유한다. 부모 ID로 트리를 형성하며, 본문은 임베딩되어 벡터 검색 대상에 포함된다. 결정(decision) 유형은 최신순 결정 조회에 쓰인다.

## API 표면
- 생성: 제목·유형을 필수로, 본문·본문형식·부모·첨부 대상을 옵션으로 받아 항목을 만든다(task/unit/plan 중 최소 하나에 첨부).
- 조회/갱신/목록/삭제: 단건 보기, 제목·본문·유형 갱신, 목록, 삭제.
- 검색: 키워드(FTS5)·의미(벡터)·하이브리드 모드로 검색.
- 가져오기/내보내기: 작업 디렉터리의 docs/ 와 양방향 동기화(특정 plan/unit 한정 가능, dry-run 미리보기 지원).
- 위키 트리: 부모/자식 관계를 재귀적으로 펼쳐 평면 목록 또는 들여쓴 아웃라인으로 렌더링(서브트리 루트·plan 필터 지원).
- LLM 노출: 읽기 전용 MCP 도구(지식 검색, 최근 결정 조회 등)로 조회만 가능하며 변경은 불가하다.

## 불변식
- MCP를 통한 지식 접근은 읽기 전용이다 — 에이전트는 MCP로 지식을 검색·조회만 할 수 있고 변경은 CLI/데몬 경로로만 가능하다(invariant.mcp-tools-readonly).

## 구현 위치 (provenance)
- CLI 지식 서브커맨드 정의에서 유형(doc/decision/reference/note/spec)·본문형식·부모·첨부 대상·가져오기/내보내기·위키 트리 옵션이 선언되어 있다(`cli/src/main.rs`의 KnowledgeAction 정의 부근).
- 데이터 모델·저장소·라우트는 인벤토리에 함께 기록되어 있다(models / repo/knowledge / routes/knowledge).

## 미확정 (OPEN)
- [ ] OPEN: 지식 항목의 버전 관리(이력 보존) 동작의 정확한 범위는 CLI 옵션만으로는 불완전 — 저장소 계층 추가 추적 필요.
