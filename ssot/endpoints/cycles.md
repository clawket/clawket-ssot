---
id: endpoint.cycles
kind: Endpoint
title: 사이클 목록 · 조회 · 생성 · 수정 · 삭제 · 사이클 태스크 · 백로그
definition: 사이클(스프린트)의 목록·단건 조회·생성·수정·삭제와, 한 사이클의 편성 태스크 조회 및 미배정 백로그 조회를 다루는 웹 대시보드용 사이클 관리 묶음.
realizedBy: [component.daemon]
implementedIn:
  - web/src/api.ts
  - web/src/components/BacklogView.tsx
relatesTo:
  - { to: concept.cycle, type: mutates, note: "사이클을 만들고 수정·삭제하며 목록·단건을 읽는다" }
  - { to: concept.backlog, type: reads, note: "사이클에 배정되지 않은 태스크 = 백로그를 조회한다" }
  - { to: concept.task, type: reads, note: "한 사이클에 편성된 태스크 목록을 조회한다" }
governedBy: []
impacts: []
consumedBy: [component.web-dashboard, component.web-api-client, screen.web-board, screen.web-backlog]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

타임박스 반복 단위인 사이클(스프린트)을 관리하는 기능 묶음이다. 사이클 목록 보기, 단건 보기, 만들기, 제목·목표·상태 수정, 삭제와 더불어, 한 사이클에 편성된 태스크 목록 보기, 그리고 아직 어떤 사이클에도 배정되지 않은 백로그 태스크 보기를 포함한다. 웹 대시보드의 칸반 보드·백로그 뷰에서 사이클을 구성하고, 백로그에서 사이클로 일거리를 끌어와 일정에 편입하는 흐름의 바탕이 된다.

## 요청 / 응답

목록 조회는 프로젝트·유닛·상태로 범위를 좁혀 사이클들을 돌려준다. 단건 조회는 사이클 하나를 지정해 그 상세를 돌려준다.

생성은 사이클이 속할 프로젝트·유닛과 제목(필요 시 목표·순서)을 보내면 새 사이클이 만들어져 돌아온다. 사이클은 반드시 단일 유닛에 묶인다.

수정은 제목·목표·상태를 바꾼다. 삭제는 사이클을 제거한다.

사이클 태스크 조회는 사이클 하나를 지정하면 그 사이클에 편성된 태스크 목록을 돌려준다. 백로그 조회는 프로젝트를 지정하면 그 프로젝트에서 사이클에 배정되지 않은 태스크들을 돌려준다(보통 생성순 정렬).

## 권한 / 제약

- 목록·단건·사이클 태스크·백로그 조회는 읽기, 생성·수정·삭제는 쓰기다.
- 사이클은 단일 유닛 소속이며, 유닛당 동시에 active일 수 있는 사이클은 하나로 제한된다(사이클 개념의 불변식). 완료된 사이클은 재시작하지 않고 새로 만든다.
- "사이클에 배정되지 않음"이 곧 백로그의 정의다 — 사이클로 옮겨진 태스크는 더 이상 백로그에 나타나지 않는다.
- 로컬 데몬에 붙는 호출이므로 로컬에서만 접근 가능하다. 상태 전이(planning→active→completed) 자체의 규칙은 사이클 개념·관련 활성/완료 엔드포인트가 책임진다.

## provenance

웹 클라이언트(`web/src/api.ts`)가 데몬의 사이클 경로(`/cycles` 목록·생성, `/cycles/{id}` 단건·수정·삭제, `/cycles/{id}/tasks` 편성 태스크, `/backlog` 미배정 태스크)를 감싸고, `web/src/components/BacklogView.tsx`가 백로그·사이클 편성 UI를 제공한다. 사이클 저장·상태 규칙·백로그 산출은 데몬(component.daemon)이 책임진다. 코드가 사라져도 "사이클 CRUD + 사이클별 편성 태스크 조회 + 프로젝트별 미배정 백로그 조회"라는 의미만으로 재구현이 가능해야 한다.

## 미확정 (OPEN)
- [ ] OPEN: 사이클 태스크·백로그의 정렬·취소 태스크 포함 정책을 데몬 태스크 저장소에서 확인해 보강할 것.
