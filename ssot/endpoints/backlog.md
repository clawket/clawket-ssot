---
id: endpoint.backlog
kind: Endpoint
title: 데몬 GET /backlog — 사이클 미배정 태스크 조회
definition: 특정 프로젝트에서 아직 어떤 사이클에도 배정되지 않은(대기 중) 태스크 목록을 조회하는 엔드포인트.
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/backlog.rs]
relatesTo:
  - { to: concept.backlog, type: reads, note: "사이클에 묶이지 않은 태스크 집합 = 백로그를 그대로 반환" }
  - { to: concept.task, type: reads, note: "백로그 항목은 cycle 미배정 상태의 태스크" }
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

한 프로젝트 안에서 "아직 일정(사이클)에 편입되지 않은 태스크"만 추려서 보여 주는 조회 엔드포인트다. 즉 백로그를 그대로 반환한다. 운영자가 웹 대시보드의 백로그 뷰에서 대기 중인 일거리를 보고, 그중 일부를 사이클로 끌어와 일정에 편입할 때 그 출발점이 되는 데이터다.

## 요청 / 응답

요청에는 어떤 프로젝트의 백로그를 볼지 프로젝트 식별 값을 준다(사람이 읽는 참조든 내부 ID든 받아 해석한다).

응답은 그 프로젝트에서 사이클에 배정되지 않은 태스크들의 목록이며, 만들어진 순서대로 정렬돼 돌아온다. 각 항목은 태스크의 전체 정보를 담는다.

## 권한 / 제약

- 프로젝트 식별 값은 필수다. 받은 값은 내부 프로젝트로 해석(resolve)한 뒤 조회한다 — 따라서 사람이 읽는 참조도 받아들인다.
- "사이클에 배정되지 않았다"는 조건이 곧 백로그의 정의다. 사이클로 옮겨진 태스크는 더 이상 백로그에 나타나지 않는다.
- 로컬 데몬에 직접 붙는 호출이므로 로컬에서만 접근 가능하다. 단순 조회이므로 데이터를 바꾸지 않는다.

## provenance

백로그 라우터(`daemon/src/routes/backlog.rs`)가 `/backlog` 경로의 조회 핸들러를 제공한다. 내부적으로는 태스크가 속한 유닛·플랜을 거쳐 프로젝트에 연결되고, 그중 사이클이 비어 있는 태스크를 생성순으로 모은다. 데이터는 로컬 SQLite 저장소(`component.daemon`이 소유)에서 읽는다. 코드가 사라져도 "프로젝트별 사이클 미배정 태스크 목록"이라는 의미만으로 재구현이 가능해야 한다.

## 미확정 (OPEN)
- [ ] OPEN: 백로그 정렬·페이징 정책이 생성순 고정 외에 추가 옵션을 가질지는 웹 대시보드 백로그 뷰 요구사항과 함께 확정할 것.
