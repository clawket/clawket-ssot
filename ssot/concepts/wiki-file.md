---
id: concept.wiki-file
kind: Concept
title: WikiFile (파일 위키)
definition: 프로젝트 작업 디렉터리의 지정된 위키 경로 아래에 실제로 존재하는 .md/.mdx 디스크 파일로, SQLite에 저장되는 지식(Knowledge)과는 구별되는 별개의 위키 표면이다.
relatesTo:
  - to: concept.knowledge
    type: complements
    note: 위키 화면에서 함께 보이지만 저장 위치(디스크 vs SQLite)가 다른 별도 표면이다.
  - to: screen.wiki
    type: relates-to
    note: Wiki 화면의 파일 트리가 이 파일들을 경로 기반으로 표시한다.
governedBy: []
dependsOn:
  - concept.project
implementedIn:
  - desktop/apps/desktop/src/data/types.ts
  - desktop/apps/desktop/src/views/WikiView.tsx
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:concept
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지(식별자는 provenance에만). 분기/의도 불명은 OPEN. -->

## 정의

WikiFile은 프로젝트가 위키로 지정한 폴더 아래에 실제 디스크 파일로 존재하는 마크다운 문서(.md/.mdx)를 가리킨다. 프로젝트 설정에는 "위키 경로" 목록이 있어, 그 경로(예: `docs` 폴더) 아래에 놓인 마크다운 파일들이 위키 표면으로 끌어올려진다.

핵심은 WikiFile이 데이터베이스에 저장되는 지식 엔트리(Knowledge)와는 **다른 갈래**라는 점이다. Knowledge는 데몬의 SQLite에 저장되고 버전·타입·임베딩을 갖는 구조화된 항목인 반면, WikiFile은 그저 디스크에 놓인 파일이다. 데스크탑의 Wiki 화면은 이 둘을 한 화면에 나란히 보여주되, 트리에서 서로 구별되는 별개의 표면으로 다룬다 — SQLite 지식은 타입·부모 관계 기반 트리로, 디스크 파일은 폴더 경로 기반 트리로 표시된다.

각 파일은 작업 디렉터리 기준 상대 경로, 확장자를 뗀 이름, 파일 첫머리의 `# 제목` 헤딩(없으면 파일명), 크기, 수정 시각, 그리고 어느 위키 경로에서 온 파일인지를 나타내는 출처 정보를 갖는다.

중요한 동작상의 비대칭이 하나 있다. 파일 트리에서 디스크 파일을 **읽는** 것은 파일 위키 표면으로 직접 처리되지만, 위키 항목의 생성·수정·삭제는 디스크 파일을 직접 건드리지 않고 모두 지식(Knowledge) 경로로 흐른다. 즉 디스크 위키 파일은 사실상 읽기 중심 표면이다.

## 엔티티 (DB)

WikiFile은 데이터베이스 테이블이 아니다. 디스크 위의 실제 파일을 데몬이 훑어 메타데이터로 변환해 내려주는 표현일 뿐이다. 파일 목록은 위키 트리 조회 시 파일별로 경로·이름·표시 제목·크기·수정 시각·출처 위키 경로를 담아 반환되고, 파일 내용 조회는 본문과 포맷을 추가로 싣는다. 따라서 WikiFile의 "단일 진실"은 SQLite가 아니라 디스크 파일 그 자체다.

## API 표면

- 디스크 파일 목록: 위키 파일 목록 조회로 가져온다.
- 개별 파일 내용: 위키 파일 내용 조회로 가져온다.
- 생성/수정/삭제: WikiFile 자체에는 쓰기 경로가 없고, 위키 항목을 만들거나 고치는 동작은 지식 CRUD 엔드포인트로 흐른다.

## 불변식

- WikiFile의 출처는 항상 프로젝트의 위키 경로 설정 아래에 있는 파일이어야 한다 — 위키 경로 밖의 파일은 이 표면에 올라오지 않는다.
- WikiFile은 SQLite 지식과 별개의 표면이며, 둘이 한 화면에 함께 보여도 저장소·식별 체계가 섞이지 않는다.
- 생성·수정·삭제는 디스크 파일을 직접 쓰는 대신 지식 경로로 위임된다.

## 구현 위치 (provenance)

- WikiFile 메타데이터·내용의 구조 정의: `desktop/apps/desktop/src/data/types.ts`
- 파일 트리·검색·마크다운 리더로 파일을 표시하는 화면: `desktop/apps/desktop/src/views/WikiView.tsx`

## 미확정 (OPEN)
- [ ] OPEN: 디스크 위키 파일에 대한 직접 편집·삭제가 향후 지원될 계획인지(현재는 지식 경로 위임이 의도된 동작인지) 확인 필요.
