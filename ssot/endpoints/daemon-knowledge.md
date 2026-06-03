---
id: endpoint.daemon-knowledge
kind: Endpoint
title: 데몬 지식 API (/knowledge, /knowledge/search, /knowledge/import)
definition: "위키성 지식 항목을 다루는 데몬 엔드포인트. 표준 CRUD에 더해 키워드·의미·하이브리드 검색(GET /knowledge/search)과 문서 일괄 가져오기(POST /knowledge/import)를 제공한다."
realizedBy: [component.daemon]
implementedIn:
  - desktop/apps/desktop/src/data/api.ts
relatesTo:
  - { to: concept.knowledge, type: mutates, note: "지식 항목의 생성·수정·삭제·검색·가져오기를 다룬다" }
  - { to: concept.knowledge-entry, type: mutates, note: "데몬이 소유하는 지식 항목 레코드를 다룬다" }
  - { to: capability.wiki-knowledge, type: realizes, note: "위키 지식 CRUD·검색 능력을 이 API가 실현한다" }
  - { to: endpoint.daemon-search, type: backed-by, note: "검색은 데몬 공용 검색(FTS5/KNN/하이브리드) 위에서 지식 대상으로 노출된다" }
governedBy: []
impacts: [concept.knowledge, domain.knowledge-rag]
consumedBy: [component.web-dashboard]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

위키성 지식 항목(결정 기록·노트·근거·참조 등)을 읽고 쓰는 데몬 API다. 지식은 작업·유닛·플랜에 첨부되어 버전 관리되고, 생성·수정 시 자동으로 임베딩되어 의미 검색 대상이 된다. 이 API는 세션 간 의미 컨텍스트를 복원하는 로컬 RAG의 쓰기·검색 표면이다.

표준 CRUD(목록·조회·생성·수정·삭제) 위에 두 가지를 더한다 — 키워드·의미·하이브리드를 아우르는 검색과, 외부 문서를 한꺼번에 끌어오는 일괄 가져오기다.

## 요청 / 응답

- **목록/조회**: 지식 항목을 돌려준다. 프로젝트·플랜·유닛·작업·유형으로 범위를 좁힐 수 있다. 각 항목은 제목·유형·본문·소속·계층(부모) 등을 담는다.
- **생성**: 유형과 제목을 필수로, 본문·본문 형식·소속(작업/유닛/플랜)·부모를 선택적으로 보낸다.
- **수정**: 바꿀 값만 보내며, 부모 재지정은 "빼면 그대로 / 빈 값이면 부모 해제 / 값이면 설정"의 3상태 규칙을 따른다.
- **삭제**: 결과로 실제 삭제인지 소프트 삭제인지를 함께 알린다.
- **검색**: 검색어와 함께 키워드(전문검색)·의미(벡터 근접)·하이브리드 중 한 방식으로 묻는다. 결과는 각 적중 항목에 점수(키워드 점수·벡터 점수·하이브리드 점수)와 잘림 여부를 붙여 돌려주고, 반환 개수·한도·전체 잘림 여부 같은 메타도 함께 준다.
- **가져오기**: 외부 문서들을 일괄로 끌어온다. 결과는 가져온 수·건너뛴 수와, 각각의 항목 목록·건너뛴 사유를 알린다. 미리보기(dry-run) 여부도 함께 표시된다.

## 권한 / 제약

- 검색의 의미·하이브리드 모드는 항목이 임베딩되어 있어야 동작한다 — 임베딩은 데몬이 생성·수정 시 자동 수행하고 시작 시 백필한다.
- 임베딩은 온디바이스로 계산되며 외부로 데이터가 나가지 않는다(로컬 우선 동작).
- 로컬 데몬 특성상 호출 주체는 같은 머신의 사람·에이전트로 한정된다.

## provenance

- implementedIn: 데스크탑 데몬 클라이언트가 CRUD·검색(점수 포함 응답·하이브리드 모드)·가져오기(가져온/건너뛴 집계·dry-run)의 요청·응답 의미를 타입과 주석으로 노출한다.
- realizedBy: clawketd 데몬이 지식 저장소와 임베딩·벡터 검색을 소유하고 이 API를 제공한다.

## 미확정 (OPEN)
- [ ] OPEN: 검색의 하이브리드 점수 결합 공식(키워드·벡터 가중)과 기본 검색 모드.
- [ ] OPEN: 가져오기가 어떤 소스 형식(예: docs/ 내 마크다운)을 대상으로 하며 중복·갱신을 어떻게 판정하는지.
