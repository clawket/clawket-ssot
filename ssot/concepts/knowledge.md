---
id: concept.knowledge
kind: Concept
title: Knowledge
definition: "플랜·유닛·태스크에 첨부되는 위키성 문서. 결정(decision)·노트(note)·설계(design)·ADR·참조(reference/link)·스펙(spec) 등 종류(type)를 가지며, 본문이 바뀌면 이전 본문이 버전으로 보존된다. 부모-자식 트리(최대 깊이 8)로 위키처럼 계층화되고, 키워드 검색(FTS5)과 온디바이스 임베딩 기반 의미 검색을 합친 하이브리드 검색의 대상이 되어 MCP를 통해 LLM 에이전트에게 세션 간 컨텍스트로 노출된다. 과거 '아티팩트(artifact)'가 옛 이름이며 식별자 접두사는 그 흔적으로 여전히 ART-다."
relatesTo:
  - to: concept.task
    type: relates-to
    note: 지식은 태스크에 첨부될 수 있다(태스크/유닛/플랜 중 하나 이상 필수)
  - to: concept.plan
    type: relates-to
    note: 지식은 플랜에도 첨부될 수 있다
  - to: concept.unit
    type: relates-to
    note: 지식은 유닛에도 첨부될 수 있다
  - to: endpoint.cli-knowledge
    type: reads
    note: clawket knowledge create/view/update/list/delete/search/import/export 명령이 지식을 다룬다
  - to: domain.knowledge-rag
    type: belongs-to
    note: 지식은 지식·로컬 RAG 영역의 핵심 데이터다
  - to: capability.rag-pull
    type: relates-to
    note: 과거 컨텍스트 RAG 풀의 검색 대상이 지식 항목이다
governedBy: []
governs: []
dependsOn:
  - component.embedding-engine
implementedIn:
  - daemon/src/repo/knowledge.rs
  - daemon/migrations/024_artifacts_to_knowledge.sql
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:concept
---

<!-- 작성 고도: 자연어 우선. 코드 식별자는 provenance/근거에만. -->

## 정의

지식(Knowledge)은 작업에 따라붙는 위키성 문서다. 어떤 결정을 왜 내렸는지(결정), 알아 둘 메모(노트), 설계·ADR, 참고 링크, 스펙 등 다양한 종류의 글을 플랜·유닛·태스크에 첨부해 둔다. 핵심 가치는 두 가지다.

1. **이력 보존**: 글 본문을 고치면 이전 본문이 버전으로 따로 남는다. 즉 지식은 덮어쓰여도 옛 내용을 잃지 않는다.
2. **검색·재사용**: 모든 지식은 검색되며, 특히 의미 기반(임베딩) 검색으로 "비슷한 맥락"을 찾아낼 수 있어, LLM 에이전트가 새 세션을 시작할 때 과거의 결정·맥락을 끌어와(RAG 풀) 잊지 않고 이어 갈 수 있게 한다.

지식은 위키처럼 부모-자식 트리로 계층화된다(최대 깊이 8). 한 지식 문서 아래에 하위 문서를 달아 폴더식 구조를 만들 수 있다.

"아티팩트(artifact)"가 옛 이름이다. 과거에는 결정·설계·와이어프레임·ADR·노트·링크를 아티팩트라 부르고 "범위(scope: reference/working/archived)"라는 분류를 두었으나, 이후 지식(knowledge)으로 이름이 바뀌면서 범위 컬럼이 제거되었고, 보관(archive) 항목은 위키 경로로 내보낸 뒤 삭제되었다. 식별자 접두사는 그 역사적 흔적으로 여전히 ART-를 쓴다(옛 호출부 안정성을 위해 보존).

### 종류(type)

각 지식은 종류를 하나 갖는다 — 결정(decision)·설계(design)·와이어프레임(wireframe)·ADR·노트(note)·링크(link)·스펙(spec) 등. 특히 결정(decision) 종류는 "최근 결정만 모아 보기" 같은 RAG 풀에서 별도로 다뤄진다.

## 엔티티 (DB)

지식은 knowledge 테이블에, 그 버전 이력은 knowledge_versions 테이블에 저장된다. 한 지식 행이 담는 정보:

- 식별자(접두사 ART이 붙은 시간정렬 식별자 — 옛 아티팩트 호환).
- 첨부 대상: 태스크·유닛·플랜 식별자(셋 다 선택이지만 **적어도 하나는 반드시** 있어야 한다 — DB CHECK로 강제. 대상이 삭제되면 지식도 함께 삭제). 무첨부 지식 생성은 거부된다(KNOWLEDGE_NEEDS_ATTACHMENT).
- 종류(type), 제목, 본문, 본문 형식(md/json/yaml, 기본 md).
- 부모 지식 참조(트리 구성용)와 위키 정렬·깊이 값.
- 생성 시각.

버전 행은 본문이 바뀔 때마다 직전 본문을 보존한다 — 지식 본문을 새 내용으로 수정하면, 바뀌기 직전 본문이 버전 한 건으로 떨어져 나가 보관된다.

검색을 위한 보조 구조:
- 전문검색(FTS5) 가상 테이블(knowledge_fts): 제목·본문에 대한 키워드 검색. 검색어 끝에 와일드카드를 붙여 접두사 매칭을 한다.
- 벡터 테이블(vec_knowledge): 온디바이스 임베딩 엔진이 만든 다국어 임베딩(384차원)을 저장해, 의미 거리 기반 최근접 검색(KNN)을 가능하게 한다.

## API 표면

데몬 HTTP 표면 + CLI(clawket knowledge):

- 생성: 대상(태스크/유닛/플랜 중 하나 이상)·종류·제목·본문·부모를 받아 만든다. 자기 자신을 부모로 두거나(SELF_PARENT) 트리 깊이 8을 넘기면(WIKI_DEPTH_EXCEEDED) 거부.
- 조회/목록: 단건 조회와 위키 트리 조회(부모 기준 또는 플랜 기준, 깊이 8까지).
- 수정: 제목·본문·형식·부모를 바꾼다. 본문이 실제로 달라지면 이전 본문이 자동으로 버전화된다. 부모를 바꿀 때 순환(WIKI_CYCLE)이 생기면 거부.
- 삭제: 행을 지우되, 그 직속 자식들은 부모 없는 루트로 끌어올려 위키 트리가 끊기지 않게 한다(클라이언트에는 soft 처리로 보고).
- 검색: 키워드(FTS5)·의미(벡터 KNN)·둘을 합친 하이브리드. 같은 검색 표면을 웹 위키·CLI search·MCP가 공유한다.
- 가져오기/내보내기: 문서 디렉터리(docs/)와 주고받는다.

지식은 MCP read-only 도구(지식 검색·최근 결정 조회 등)를 통해 LLM 에이전트에게 노출된다 — 이것이 세션 간 컨텍스트 복원(RAG 풀)의 통로다.

## 불변식

- **첨부 필수**: 모든 지식은 태스크·유닛·플랜 중 적어도 하나에 첨부돼야 한다(DB CHECK). 무첨부 생성은 거부.
- **종속 삭제**: 첨부 대상이 삭제되면 지식도 함께 삭제된다.
- **버전 보존**: 본문이 바뀌면 직전 본문이 버전으로 남는다 — 옛 내용은 사라지지 않는다.
- **위키 트리 제약**: 자기 자신을 부모로 둘 수 없고, 트리 깊이는 최대 8이며, 부모 변경 시 순환이 생기면 거부된다. 부모가 삭제되면 자식은 루트로 승격돼 트리 도달성이 유지된다.

## 구현 위치 (provenance)

- 지식 생성(첨부 필수·트리 깊이·자기부모 검사)·수정(본문 변경 시 버전 생성·순환 검사)·삭제(자식 루트 승격)·위키 트리·키워드 검색·임베딩 저장/검색: `daemon/src/repo/knowledge.rs`.
- 아티팩트→지식 이름 변경, 범위 컬럼 제거, 보관 항목 사전 내보내기·삭제, knowledge/knowledge_versions/knowledge_fts 재구성, ART- 식별자 호환: `daemon/migrations/024_artifacts_to_knowledge.sql`.

## 미확정 (OPEN)
- [ ] OPEN: 종류(type) 값이 DB CHECK로 폐집합 강제되는지(주석에 후보 나열은 있으나 CHECK 제약 여부 미확정) 추가 확인 필요.
- [ ] OPEN: 임베딩 자동 생성 시점(생성/수정 시 동기 vs 데몬 시작 시 백필)과 하이브리드 검색의 점수 결합 방식(FTS5 rank + 벡터 distance 가중) 정확한 로직 — embeddings 엔진·검색 라우트 측 교차 추적 필요.
- [ ] OPEN: 노드 소스의 웹 컴포넌트(WikiView 등)·`apps/desktop/src/data/types.ts`는 웹/데스크탑 레포 기준 경로 — 위키 화면 측 표현은 별도 화면(Screen) 노드에서 다룰 영역.
