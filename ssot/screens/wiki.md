---
id: screen.wiki
kind: Screen
title: Wiki 화면 (지식 위키 / 파일 트리)
purpose: "사람 운영자가 프로젝트 지식 항목과 파일시스템 마크다운 문서를 트리로 탐색·검색하고, 우측 리더로 읽으며, 지식 항목을 생성·편집·삭제한다."
servesPersona: [persona.human-operator]
realizedBy: [component.web-dashboard, component.desktop-app]
implementedIn:
  - desktop/apps/desktop/src/views/WikiView.tsx
  - desktop/apps/desktop/src/views/ViewShell.tsx
  - web/src/components/WikiView.tsx
consumesApi: [endpoint.daemon-search, endpoint.daemon-events]
relatesTo:
  - { to: concept.knowledge, type: reads, note: "SQLite 지식 항목을 트리로 보여주고 CRUD한다" }
  - { to: capability.wiki-knowledge, type: realizes, note: "위키 지식 CRUD·검색 능력을 구현하는 화면" }
  - { to: endpoint.daemon-search, type: calls, note: "검색창 입력으로 지식 하이브리드 검색을 호출한다" }
impacts: [concept.knowledge]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:screen"]
---

## 화면 목적

Wiki 화면은 프로젝트의 지식을 한 곳에서 읽고 관리하는 위키다. 좌측에는 두 종류의 탐색 트리가 있다 — 데몬이 보관하는 지식 항목(SQLite knowledge) 트리와, 프로젝트 디렉터리의 마크다운 파일(.md) 트리. 가운데 위에는 검색창이 있고, 우측에는 선택한 문서를 마크다운으로 렌더링하는 리더가 있다. 지식 항목은 이 화면에서 생성·편집·삭제할 수 있다.

## UI 요소 / 입력 필드

- 지식 트리: parent_id로 부모-자식 관계를 만들고 wiki 인덱스·제목순으로 정렬한 지식 항목을 펼쳐 보여준다. 각 항목에는 편집·삭제 버튼이 있다.
- 파일 트리: 파일들을 wiki 루트별로 묶고, 경로를 폴더/파일 트리로 재구성해(폴더 먼저, 이름 자연정렬) 보여준다.
- 검색창: 입력하면 디바운스 후 지식 검색을 수행하고, 매칭된 지식과 그 조상 항목만 트리에 남기며 파일 트리도 제목·이름·경로로 필터링한다.
- 우측 리더: 선택한 지식/파일의 본문을 GFM 마크다운으로 렌더링하고 제목 헤더를 보여준다.
- 모달: 지식 생성 모달(제목·type·부모·내용 입력), 지식 편집 모달(제목·부모·내용), 지식 삭제 확인 모달.

## 표시 데이터 / 호출 API

화면은 지식 항목 목록과 파일 목록을 받아 두 트리를 만든다. 파일 본문은 선택 시 데몬에서 해당 파일을 받아 렌더링한다. 검색은 데몬의 검색 API(키워드 FTS5 + 의미 벡터 하이브리드)를 호출해 활성 프로젝트 범위에서 지식을 찾는다. 지식 생성·수정·삭제는 데몬의 지식 변경 동작을 호출하며, 지식은 버전 관리되고 임베딩되어 이후 검색·MCP 노출 대상이 된다. 데이터는 실시간 이벤트 스트림(SSE)을 구독하는 공통 데이터 계층을 통해 갱신된다.

## 상태 / 엣지케이스

- 로딩/에러 상태: 데이터·리더 각각의 에러 메시지를 표시한다.
- 빈 상태: 표시할 문서가 없으면 빈 안내를 보여준다.
- 검색 진행 중/검색 에러: 검색 상태와 오류를 별도로 표시한다.
- 검색어가 비면 전체 트리로 복귀한다.
- 모달 제출 실패: 모달 안에 오류 메시지를 표시한다.

## 미확정 (OPEN)
- [ ] OPEN: 파일 트리가 읽는 wiki_root 집합의 출처(어느 디렉터리를 스캔하는지)와 데스크탑/웹 간 차이를 확정할 것
- [ ] OPEN: 지식 type 어휘(decision/note/evidence/reference 등) 선택지가 화면에서 어떻게 고정/제약되는지 정독해 명시할 것
