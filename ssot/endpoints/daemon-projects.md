---
id: endpoint.daemon-projects
kind: Endpoint
title: 프로젝트 관리 API (데몬)
definition: "데몬이 제공하는 프로젝트 CRUD 묶음. 프로젝트 목록 조회, 새 프로젝트 생성, 속성 수정, 삭제를 담당한다. 데스크탑·웹 화면이 프로젝트를 다룰 때 호출하는 단일 창구."
realizedBy:
  - component.daemon
implementedIn:
  - desktop/apps/desktop/src/data/api.ts
relatesTo:
  - to: concept.project
    type: mutates
    note: "프로젝트 엔티티의 생성·수정·삭제·조회를 수행한다."
governedBy:
  - invariant.no-work-without-task
impacts:
  - concept.project
consumedBy:
  - component.renderer
  - component.daemon-client
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

프로젝트는 작업 흐름(계획·단위·사이클·태스크·지식)이 매달리는 최상위 그릇이다. 이 API는 그 프로젝트를 다루는 네 가지 동작을 한곳에 모은 것으로, 데스크탑·웹 화면이 프로젝트를 보여 주거나 새로 만들거나 고치거나 지울 때 데몬에 보내는 요청이다.

- 목록 보기: 등록된 프로젝트 전체를 가져온다.
- 새로 만들기: 이름을 필수로, 설명·키(짧은 식별 접두어)·활성 여부·위키 경로·연결 작업 디렉토리를 곁들여 등록한다.
- 고치기: 위 속성들을 부분적으로 바꾼다(보낸 항목만 반영).
- 지우기: 프로젝트를 삭제한다.

## 요청 / 응답

- 목록 조회: 입력 없음 → 프로젝트들의 목록. 각 항목은 이름·설명·짧은 키·활성 여부·위키 경로 모음·연결된 작업 디렉토리 모음 같은 식별 정보를 담는다.
- 생성: 이름(필수)과 선택 속성(설명, 짧은 키, 활성/비활성, 위키 경로 목록, 작업 디렉토리 목록)을 보낸다 → 만들어진 프로젝트 한 건을 돌려준다. 활성 여부는 켜짐/꺼짐 두 상태로 표현된다.
- 수정: 바꾸고 싶은 속성만 골라 보낸다(생략한 항목은 그대로 유지) → 갱신된 프로젝트 한 건.
- 삭제: 대상 프로젝트를 지정 → "지웠음 + 지운 대상 식별자" 형태의 결과.

## 권한 / 제약

- 데몬에 직접 접속하는 클라이언트(데스크탑 렌더러·웹)가 호출한다. 로컬 접속이며 세션 토큰 인증 위에서 동작한다.
- 수정은 부분 갱신 방식이다 — 일부 속성만 보내 바꾸고 나머지는 보존한다.
- "작업 디렉토리(cwd) 목록"은 어느 폴더에서 이 프로젝트의 작업을 시작하면 자동 연결되는지를 정하는 매핑의 근거가 된다.
- 프로젝트 삭제가 그 아래 매달린 계획·태스크에 어떤 연쇄 효과를 일으키는지(같이 지워지는지, 막히는지)는 데몬 정책 영역이며 아래 OPEN으로 남긴다.

## provenance

- 클라이언트 측 계약(메서드·경로·주고받는 속성): `desktop/apps/desktop/src/data/api.ts` (`listProjects` / `createProject` / `updateProject` / `deleteProject`).
- 제공 컴포넌트: `component.daemon` (서버 측 핸들러).
- 대상 엔티티의 의미: `concept.project`.

## 미확정 (OPEN)
- [ ] OPEN: 프로젝트 삭제 시 하위 계획·단위·사이클·태스크의 처리(연쇄 삭제 vs 거부 vs 보존)를 데몬 핸들러에서 확인.
- [ ] OPEN: "짧은 키" 중복 방지·형식 제약(대문자/길이 등)이 있는지 확인.
