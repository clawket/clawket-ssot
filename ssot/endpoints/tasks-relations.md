---
id: endpoint.tasks-relations
kind: Endpoint
title: 태스크 관계 목록 조회
definition: "GET /tasks/{id}/relations — 한 태스크가 맺은 태스크 간 관계를 돌려준다(같은 라우터에서 관계 생성·삭제도 제공)."
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/relations.rs]
relatesTo:
  - { to: concept.task-relation, type: reads, note: "태스크가 맺은 관계를 조회한다" }
  - { to: concept.task, type: reads, note: "관계의 양 끝 태스크 존재를 확인한다" }
governedBy: []
impacts: [concept.task-relation]
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
태스크끼리는 "어떤 태스크가 다른 태스크와 연관된다"는 관계를 맺을 수 있다(예: 의존·연관 등 관계 유형으로 구분). 이 동작은 한 태스크가 맺고 있는 관계들을 모아 보여 준다. 같은 라우터가 새 관계를 만들고, 기존 관계를 끊는 일도 함께 제공한다.

## 요청 / 응답
조회는 대상 태스크를 가리키면 그 태스크가 출발점이 되는 관계 목록을 돌려준다. 관계 생성은 출발 태스크와 도착 태스크, 그리고 관계 유형을 받아 새 관계를 만든다. 관계 삭제는 관계 자체를 식별자로 지목해 끊고, 무엇을 지웠는지 확인 정보를 돌려준다.

## 권한 / 제약
관계를 만들 때는 출발 태스크와 도착 태스크가 모두 실제로 존재해야 하며, 어느 한쪽이라도 없으면 "찾을 수 없음"으로 응답한다(이 오류 메시지는 사용자 로케일에 맞춰 현지화된다). 조회 시에도 대상 태스크가 없으면 같은 방식으로 응답한다.

## provenance
구현은 데몬의 관계 라우터(daemon/src/routes/relations.rs)에 있다. 관계 데이터 자체는 task-relation 개념이 보관하며, 라우터는 양 끝 태스크의 존재를 확인한 뒤 관계를 생성·조회·삭제한다.

## 미확정 (OPEN)
- [ ] OPEN: 관계 유형의 허용 어휘(의존/차단/연관 등)와 양방향 표현 여부, 도착 태스크 기준 역조회 제공 여부는 확인 필요.
