---
id: endpoint.tasks-get
kind: Endpoint
title: 데몬 GET /tasks/{id}
definition: 식별자로 단일 태스크 하나를 조회하는 데몬 HTTP 엔드포인트.
realizedBy: [component.daemon]
implementedIn:
  - daemon/src/routes/tasks.rs
relatesTo:
  - { to: concept.task, type: reads, note: "지정한 식별자의 태스크 단건을 반환한다" }
governedBy: []
impacts: []
consumedBy: [endpoint.cli-task, component.web-dashboard]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

특정 태스크 하나의 상세를 가져오는 조회 기능이다. 태스크 상세 화면을 열거나, CLI에서 한 건을 들여다볼 때, 또는 다른 작업이 시작 전에 대상 태스크의 현재 상태를 확인할 때 쓰인다.

## 요청 / 응답

요청에는 조회할 태스크의 식별자 하나만 담는다. 응답은 그 태스크의 단건 정보로, 제목·본문·상태·담당·상위 태스크 관계·소속 유닛/사이클 등 태스크가 가진 속성을 담는다. 해당 식별자의 태스크가 없으면 "찾을 수 없음"으로 응답한다.

## 권한 / 제약

로컬 데몬에 한정된 읽기 전용 조회다. 식별자에 사람이 읽는 티켓 번호(CK-N)나 내부 ULID 어느 쪽을 주는지에 따른 해석은 저장소 계층의 조회 규칙을 따른다(아래 OPEN).

## provenance

`daemon/src/routes/tasks.rs`의 `/tasks/{id}` GET 라우트가 태스크 저장소의 단건 조회를 호출하고, 없으면 404로 응답한다. 제공 주체는 clawketd 데몬(component.daemon)이며 CLI task 명령과 웹 대시보드가 소비한다.

## 미확정 (OPEN)
- [ ] OPEN: 식별자로 ULID와 티켓 번호(CK-N)를 모두 받는지, 실행 봉투 등 부가 정보가 단건 응답에 함께 실리는지 저장소·직렬화 계층에서 확인해 보강할 것.
