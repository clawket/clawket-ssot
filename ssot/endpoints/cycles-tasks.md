---
id: endpoint.cycles-tasks
kind: Endpoint
title: 데몬 GET /cycles/{id}/tasks
definition: 특정 사이클에 편성된 태스크 목록을 반환하는 데몬 HTTP 조회 엔드포인트.
realizedBy: [component.daemon]
implementedIn:
  - daemon/src/routes/cycles.rs
relatesTo:
  - { to: concept.cycle, type: reads, note: "사이클 식별자로 그 사이클에 묶인 태스크를 조회한다" }
  - { to: concept.task, type: reads, note: "사이클에 편성된 태스크 집합을 돌려준다" }
governedBy: []
impacts: []
consumedBy: [endpoint.cli-cycle, component.web-dashboard]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

하나의 사이클(스프린트)에 현재 편성되어 있는 태스크가 무엇인지 알고 싶을 때 쓰는 조회 기능이다. 사이클을 가리키는 식별자를 경로에 담아 호출하면, 그 사이클에 배정된 태스크 전부를 한 번에 돌려준다. 사람 운영자가 칸반·타임라인 화면에서 특정 사이클의 진행 상황을 펼쳐 볼 때, 그리고 CLI에서 사이클 단위로 작업을 점검할 때의 바탕이 된다.

## 요청 / 응답

요청에는 조회 대상 사이클을 지정하는 식별자 하나만 필요하다. 응답으로는 그 사이클에 속한 태스크들의 목록이 돌아오며, 각 태스크 항목은 제목·상태(할 일/진행 중/완료/차단/취소)·담당·상위 태스크 관계 등 태스크가 가진 일반 속성을 담는다. 사이클에 배정된 태스크가 하나도 없으면 빈 목록이 돌아온다.

## 권한 / 제약

로컬 데몬에 한정된 조회로, 읽기 전용이라 상태를 바꾸지 않는다. 조회 범위는 "이 사이클에 배정된 태스크"로 한정된다 — 사이클에 아직 배정되지 않은 백로그 태스크는 여기에 포함되지 않는다. 존재하지 않는 사이클 식별자를 주면 빈 결과로 취급된다(태스크가 없는 사이클과 같은 모양).

## provenance

`daemon/src/routes/cycles.rs`의 `/cycles/{id}/tasks` 라우트가 사이클 식별자를 필터로 삼아 태스크 저장소를 조회한다. 제공 주체는 clawketd 데몬(component.daemon)이며, CLI의 cycle 명령과 웹 대시보드가 이 조회를 소비한다.

## 미확정 (OPEN)
- [ ] OPEN: 정렬 순서·취소된 태스크 포함 여부 등 목록 구성 정책을 저장소 계층(`repo::tasks::list`)에서 확인해 자연어로 보강할 것.
