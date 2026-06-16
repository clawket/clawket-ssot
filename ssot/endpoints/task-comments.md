---
id: endpoint.task-comments
kind: Endpoint
title: 태스크 코멘트 조회 · 생성 · 삭제
definition: 한 태스크에 달린 코멘트를 조회하고, 새 코멘트를 작성하며, 코멘트를 삭제하는 웹 대시보드용 코멘트 기능.
realizedBy: [component.daemon]
implementedIn:
  - web/src/api.ts
relatesTo:
  - { to: concept.comment, type: mutates, note: "태스크에 코멘트를 달거나 지운다" }
  - { to: concept.task, type: reads, note: "코멘트는 특정 태스크에 부속된다" }
governedBy: []
impacts: []
consumedBy: [component.web-dashboard, component.web-api-client, screen.task-detail-drawer]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

특정 태스크에 사람·에이전트가 남긴 코멘트를 다루는 기능이다. 한 태스크의 코멘트 전부를 불러오고, 작성자와 내용을 담아 새 코멘트를 달고, 기존 코멘트를 지운다. 웹 대시보드에서 태스크 상세를 열었을 때 그 태스크에 오간 메모·논의·결정 흔적을 보여 주고 추가하는 데 쓰인다.

## 요청 / 응답

조회는 대상 태스크를 지정하면 그 태스크에 달린 코멘트 목록을 돌려준다. 각 코멘트 항목은 작성자·내용·작성 시각 등 코멘트가 가진 일반 속성을 담는다.

생성은 대상 태스크와 함께 누가 썼는지(작성자)와 내용을 보내면 새 코멘트가 만들어져 돌아온다.

삭제는 지울 코멘트를 지정해 제거한다.

## 권한 / 제약

- 조회는 읽기, 생성·삭제는 쓰기 작업이다.
- 코멘트는 단순한 메모를 넘어, 취소된 작업의 사유나 결정 맥락을 보존하는 히스토리 수단으로도 쓰인다 — 파괴적 삭제 대신 코멘트로 흔적을 남기는 운영 관행과 맞물린다.
- 로컬 데몬에 붙는 호출이므로 로컬에서만 접근 가능하다.

## provenance

웹 클라이언트(`web/src/api.ts`)가 데몬의 코멘트 경로(`/tasks/{id}/comments` 조회·생성, `/comments/{id}` 삭제)를 감싼다. 코멘트 저장·조회·삭제는 데몬(component.daemon)이 책임진다. 코드가 사라져도 "태스크별 코멘트 목록 조회 + 작성자·내용 기반 생성 + 삭제"라는 의미만으로 재구현이 가능해야 한다.

## 미확정 (OPEN)
- [ ] OPEN: 코멘트 정렬 순서(작성순/역순)와 감사 로그 연동 여부를 데몬 코멘트 저장소에서 확인해 보강할 것.
