---
id: concept.knowledge-artifact
kind: Concept
title: 지식 아티팩트(웹 관점)
definition: "task/unit/plan 에 매달리는 위키성 콘텐츠를 웹 위키 화면이 다루는 표상으로, 데몬 마이그레이션 이후 경로가 /knowledge 로 통일되었으나 웹 코드의 타입·함수 식별자는 Artifact 표기를 잠정 유지한다."
relatesTo:
  - to: concept.knowledge
    type: relates-to
    note: 데몬·CLI가 정의하는 지식 엔티티를 웹 위키 화면이 그대로 투영한다 — Artifact 는 동일 엔티티의 레거시 코드 표기다.
  - to: concept.task
    type: relates-to
    note: 지식 아티팩트는 task/unit/plan 중 하나에 첨부될 수 있다.
  - to: endpoint.daemon-search
    type: relates-to
    note: 위키 검색은 데몬의 BM25+벡터 하이브리드 검색 엔드포인트를 사용한다.
governedBy: []
governs: []
dependsOn:
  - concept.knowledge
consumesApi:
  - endpoint.daemon-search
implementedIn:
  - web/src/types.ts
  - web/src/api.ts
  - web/src/components/WikiView.tsx
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:concept"]
---

<!-- 작성 고도: 자연어. 코드 옮겨적기 금지. 분기/의도 불명은 OPEN. -->

## 정의

지식 아티팩트(웹 관점)는 웹 위키 화면이 다루는 **위키성 콘텐츠**다 — 결정·노트·근거·참조 같은 문서 항목으로, task·unit·plan 중 하나에 첨부되거나 독립 위키 문서로 존재한다. 이 엔티티의 진실은 데몬·CLI가 정의하는 지식(`concept.knowledge`)에 있고, 이 노드는 그 위키 표면을 가리킨다.

명칭에 **중요한 역사적 단절**이 있다. 데몬 마이그레이션(migration-024) 이후 이 엔티티의 정식 HTTP 경로는 `/knowledge` 로 통일되었고, 데몬은 더 이상 `/artifacts` 별칭을 제공하지 않는다. 그러나 웹 코드의 타입·함수 식별자는 여전히 `Artifact` 표기(예: 목록·생성·수정·삭제 함수, 검색 응답 타입)를 **잠정적으로 유지**한다 — 모든 호출은 실제로는 `/knowledge` 경로를 향한다. 즉 "Artifact" 는 코드 안에서만 살아남은 옛 이름이고, 사람이 보는 화면과 데몬 경로는 모두 "지식(knowledge)" 이다.

위키 화면은 첨부물들을 **파일 트리**로 구성해 보여준다. 각 항목은 제목·콘텐츠·콘텐츠 형식을 가지며, 트리 정렬을 위한 형제 순서 힌트와 깊이 힌트를 함께 둔다(부모 연결이 불완전한 고아 항목의 들여쓰기를 깊이 힌트로 보정한다). 콘텐츠는 마크다운으로 렌더링된다.

## 엔티티 (DB)

저장은 데몬 책임이다. 웹 관점의 아티팩트 한 건은 식별자, 매달린 대상(task/unit/plan 중 하나, 모두 없으면 독립), 타입, 제목, 콘텐츠와 콘텐츠 형식, 생성 시각, 그리고 위키 트리용 순서·깊이 힌트로 구성된다. 버전은 별도 모양(아티팩트 식별자·버전 번호·콘텐츠·형식·생성 시각·생성자)으로 다루어진다 — 지식은 버전 관리된다.

## API 표면

- 지식 목록 조회: task/unit/plan/타입으로 필터해 `/knowledge` 에서 받는다.
- 지식 단건 조회 / 생성 / 수정 / 삭제: 모두 `/knowledge` 경로를 향한다(코드 함수명은 Artifact 표기).
- 지식 검색: 서버측 BM25 + 벡터 하이브리드 검색(`/knowledge/search`). 기본 모드는 하이브리드이며, sqlite-vec 이 없으면 키워드 전용으로 폴백한다. 오래된 데몬이라 이 경로가 없으면(404/501) 클라이언트측 필터로 폴백한다.

## 불변식

- 정식 HTTP 경로는 `/knowledge` 하나다 — `/artifacts` 별칭은 데몬에서 제거되었다.
- 웹 코드의 `Artifact*` 식별자는 레거시 표기이며 모든 호출은 `/knowledge` 를 향한다(표기와 경로의 분리).
- 첨부 대상은 task/unit/plan 중 최대 하나이며, 셋 다 없으면 독립 위키 문서다.

## 구현 위치 (provenance)

- `web/src/types.ts` — 아티팩트와 아티팩트 버전의 데이터 모양, 위키 트리용 순서·깊이 힌트 선언.
- `web/src/api.ts` — `/knowledge` 경로 CRUD·검색 호출과 "마이그레이션-024 로 artifact→knowledge 개명, Artifact 식별자는 레거시 유지" 주석.
- `web/src/components/WikiView.tsx` — 첨부물을 파일 트리로 구성·마크다운 렌더링하는 위키 화면.

## 미확정 (OPEN)
- [ ] OPEN: 웹 코드의 `Artifact` 표기를 `Knowledge` 로 정식 개명할 계획·시점은 미정.
