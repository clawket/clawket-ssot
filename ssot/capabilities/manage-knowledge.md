---
id: capability.manage-knowledge
kind: Capability
title: 위키 지식 관리
definition: 데스크탑 앱의 Wiki 화면에서 지식 엔트리를 만들고(타입·부모·마크다운 본문) 고치고 지운다. 트리로 묶인 지식 항목과 프로젝트의 .md 파일을 함께 탐색하고, 제목·내용 검색(의미 기반 하이브리드)으로 찾아 본문을 마크다운으로 읽는다.
purpose: 결정·메모 등 세션을 넘어 보존할 지식을 사람이 화면에서 직접 만들고 정리하며, 트리와 검색으로 다시 찾아 읽을 수 있게 한다.
servesPersona:
  - persona.human-operator
realizedBy:
  - component.desktop-app
implementedIn:
  - desktop/apps/desktop/src/views/WikiView.tsx
  - desktop/apps/desktop/src/data/api.ts
relatesTo:
  - to: concept.knowledge
    type: relates-to
    note: 지식 엔트리를 생성·수정·삭제하고 트리로 표시한다
  - to: endpoint.daemon-search
    type: reads
    note: 지식 검색에 데몬의 하이브리드 검색을 쓴다
  - to: capability.wiki-knowledge
    type: relates-to
    note: 위키 지식 CRUD·검색의 데스크탑 표면이다
  - to: capability.web-review
    type: relates-to
    note: 웹 대시보드 Wiki 뷰와 같은 화면을 데스크탑에서 제공한다
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

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로. 코드 옮겨적기 금지. -->

## 사용자가 할 수 있는 일

사람이 결정·메모 같은, 작업이 끝난 뒤에도 남겨야 할 지식을 화면에서 직접 만들고 정리할 수 있다. Wiki 화면은 두 종류의 내용을 한 트리에서 함께 보여준다 — Clawket이 보관하는 **지식 엔트리**와, 프로젝트 폴더에 실제로 존재하는 **.md 파일**이다. 지식 엔트리는 부모 밑에 매달아 계층으로 정리할 수 있고, 제목·내용으로 검색하면 의미 기반(하이브리드) 검색으로 관련 항목을 추려 트리에서 그 항목과 상위 경로만 남겨 보여준다. 고른 항목의 본문은 오른쪽에 마크다운으로 렌더되어 읽기 좋다.

## 행위

- 지식 엔트리를 만든다 — 제목(필수), 타입(wiki/decision/note), 부모 항목, 마크다운 본문을 입력해 생성한다. 만든 직후 그 항목을 선택해 보여준다.
- 지식 엔트리를 고친다 — 제목·본문·부모를 바꾼다. 부모는 "그대로 둠 / 부모 비움 / 다른 부모로" 세 가지로 다룬다.
- 지식 엔트리를 지운다 — 확인 창을 거쳐 삭제하며, 보존 정책에 따라 부드러운 삭제일 수 있다고 안내한다. 지운 항목이 선택돼 있었다면 선택을 해제한다.
- 지식과 파일을 트리로 함께 탐색하고, 검색어로 의미 기반 검색을 해 결과만 추려 본다.
- 고른 지식 또는 파일의 본문을 마크다운으로 읽는다(파일 본문은 그때그때 불러온다).

## 시스템 흐름

Wiki 화면 → 메모리에 들고 있는 지식 목록을 트리로, 프로젝트 .md 파일을 또 다른 트리로 구성 → 검색어 입력 시 데몬의 하이브리드 지식 검색을 호출해 일치 항목만 추려 표시 → 생성/수정/삭제는 데스크탑 데이터 계층을 통해 데몬에 반영하고, 변경 이벤트로 목록이 갱신 → 선택한 항목·파일 본문을 마크다운으로 렌더.

## 어디에 구현되어 있나

데스크탑 앱의 Wiki 뷰(`WikiView.tsx`)가 지식·파일 트리 구성, 생성·수정·삭제 모달, 디바운스된 하이브리드 검색, 마크다운 리더를 모두 담는다. 생성·수정·삭제·검색·파일읽기 호출은 데스크탑 데이터 계층(`data/api.ts`)을 거쳐 데몬으로 간다.

## 미확정 (OPEN)
- [ ] OPEN: 삭제 시 "보존 정책에 따라 소프트삭제일 수 있다"는 안내의 실제 정책(어떤 조건에 영구/소프트 삭제인지) 미확인 — 데몬 측 보존 규칙 확인 필요.
- [ ] OPEN: 생성 모달의 타입 선택지(wiki/decision/note)가 전체 지식 타입을 다 노출하는지(evidence/reference 등 누락 여부) 미확인.
