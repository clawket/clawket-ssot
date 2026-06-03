---
id: screen.web-wiki
kind: Screen
title: Wiki 화면 (지식·문서)
definition: 지식 항목 트리와 프로젝트 파일 트리를 좌측에 두고 우측에서 문서 본문을 렌더링하며, 지식 생성·삭제·버전 이력·하이브리드 검색을 제공하는 위키 화면.
purpose: 사람 운영자가 프로젝트 지식과 문서 파일을 트리로 탐색하고 생성·검색하며 버전 이력을 확인한다
servesPersona: [persona.human-operator]
realizedBy: [component.web-dashboard]
implementedIn:
  - web/src/components/WikiView.tsx
consumesApi:
  - endpoint.daemon-knowledge
  - endpoint.daemon-wiki
  - endpoint.daemon-search
  - endpoint.daemon-plans
  - endpoint.daemon-units
  - endpoint.daemon-projects
relatesTo:
  - { to: capability.wiki-knowledge, type: realizes, note: 위키 지식 CRUD·하이브리드 검색의 사용자 표면 }
  - { to: concept.knowledge, type: reads, note: 버전 관리되는 지식 항목을 트리로 표시·생성·삭제 }
  - { to: domain.knowledge-rag, type: reads, note: FTS5+벡터 하이브리드 검색을 위키에서 노출 }
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:screen"]
---

## 화면 목적

Wiki 화면은 프로젝트에 축적된 지식 항목과 프로젝트 폴더의 문서 파일을 한 화면에서 탐색·열람·관리하는 위키다. 좌측 사이드바의 파일 트리에서 문서를 고르면 우측에서 본문을 렌더링하고, 지식 항목은 새로 만들거나 삭제하고 버전 이력을 되짚을 수 있다. 상단 검색창으로는 의미 기반과 키워드를 결합한 하이브리드 검색을 수행해 원하는 문서를 빠르게 찾는다.

## UI 요소 / 입력 필드

- **사이드바 — Wiki 헤더**: "+ New" 버튼으로 새 지식 항목을 만든다.
- **검색창**: 입력하면 디바운스 후 서버 하이브리드 검색을 호출하고, 검색어가 제목/내용에 강조 표시된다.
- **Knowledge 트리**: 지식 항목을 부모-자식 계층 트리로 보여준다. 각 노드는 포맷 아이콘(마크다운/JSON/기타)·제목을 표시하고 펼침/접힘과 하위 문서 추가(+)를 지원한다. 들여쓰기는 데몬이 제공하는 깊이 값을 우선 사용한다.
- **Project files 트리**: 프로젝트 작업 디렉터리의 문서 파일들을 위키 루트별로 묶어 폴더 트리로 보여준다.
- **본문 영역**: 선택한 지식 항목 또는 파일의 제목·타입·포맷·날짜와 본문을 렌더링한다. 마크다운은 GFM(테이블 등)으로 렌더링하고, code/json/yaml 포맷은 경량 토큰 하이라이트로 표시한다.
- **History / Delete**: 선택한 지식 항목에 대해 버전 이력 패널(버전 번호·시각·작성자·복원)을 토글하고, 삭제할 수 있다.
- **New Knowledge Entry 모달**: 제목·타입(document/decision/wireframe/api_spec/schema)·포맷(markdown/json/yaml)·내용을 입력해 지식 항목을 생성한다.

## 표시 데이터 / 호출 API

진입 시 프로젝트의 플랜·프로젝트 정보를 불러오고, 각 플랜과 그 하위 유닛에 첨부된 지식 항목을 모아 중복 제거 후 트리로 구성한다. 프로젝트 작업 디렉터리가 있으면 그 폴더의 위키 파일 목록을 함께 적재한다. 검색어를 입력하면 데몬의 하이브리드(BM25+벡터) 검색을 호출하고, 데몬이 검색을 지원하지 않는 경우 클라이언트 측 부분일치 필터로 폴백한다. 지식 생성은 첫 플랜에 첨부하는 방식으로 만들고, 버전 이력·파일 본문도 데몬에서 조회한다. 데몬의 지식·위키 파일·검색·플랜·유닛·프로젝트 조회 및 지식 생성/삭제 API를 사용한다.

## 상태 / 엣지케이스

- **로딩 중**: "Loading wiki..." 표시.
- **검색 중 / 결과 없음**: 검색 진행 중에는 진행 안내, 결과가 없으면 "No results for …"를 표시.
- **문서 없음**: 지식·파일이 모두 없으면 "No documents yet" 안내.
- **미선택**: 본문 영역에 "Select a document from the sidebar" 안내.
- 파일 적재가 실패하면 파일 목록을 비워 화면 자체는 계속 동작한다.
- 버전 복원은 현재 동작이 본문 일부를 로그로 남기는 수준으로 보여 완전 적용 흐름은 미확정.

## 미확정 (OPEN)
- [ ] OPEN: 버전 복원(onRestore)이 실제 지식 본문을 갱신하는지, UI 데모 수준인지 확정
- [ ] OPEN: 지식 트리의 부모-자식 깊이 산정(데몬 wiki_depth) 규칙을 자연어로 정리
