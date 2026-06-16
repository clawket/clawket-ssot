---
id: endpoint.relations-delete
kind: Endpoint
title: 태스크 관계 삭제·작성·조회 (데몬 DELETE /relations/{id} 외)
definition: 태스크 사이의 관계(의존 등)를 만들고, 한 태스크의 관계를 조회하고, 관계 하나를 삭제하는 데몬 엔드포인트.
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/relations.rs]
relatesTo:
  - { to: concept.task-relation, type: mutates, note: "태스크 간 관계 생성/삭제" }
  - { to: concept.task-relation, type: reads, note: "한 태스크의 관계 목록 조회" }
  - { to: concept.task, type: reads, note: "관계 양 끝 태스크의 존재 확인" }
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
한 태스크와 다른 태스크 사이의 관계(예: 의존·연관 등 관계 유형을 붙인 연결)를 등록하고, 특정 태스크가 가진 관계들을 조회하고, 더 이상 필요 없는 관계 하나를 끊는 동작을 제공한다. 태스크 트리(부모-자식 분해)와 달리, 관계는 구조 계층을 가로질러 태스크끼리 임의로 연결을 매는 수단이다.

## 요청 / 응답
- 생성 요청은 출발 태스크(경로)와 대상 태스크 식별자, 그리고 관계 유형을 보낸다. 응답은 방금 만든 관계 한 건을 돌려준다.
- 조회 요청은 한 태스크 식별자를 보내고, 그 태스크가 얽힌 관계 목록을 돌려준다.
- 삭제 요청은 관계 식별자를 보내고, 끊었다는 확인(대상 식별자 포함)을 돌려준다.
- 관계의 양 끝 태스크 중 어느 하나라도 존재하지 않으면 "찾을 수 없음"으로 응답하며, 이 메시지는 로케일에 맞게 현지화된다.

## 권한 / 제약
로컬 데몬 내부 API로 외부 인증은 없다. 관계를 만들 때는 출발·대상 태스크가 모두 실재해야 하며, 없으면 거절한다. 오류 메시지는 환경 로케일(언어 설정)에 따라 번역되어 반환된다(요청 헤더 기반 로케일 협상은 아직 미완성).

## provenance
- 라우터·핸들러: `daemon/src/routes/relations.rs`의 `GET·POST /tasks/{id}/relations`(조회·생성)와 `DELETE /relations/{id}`(삭제). 생성 시 출발·대상 태스크 존재를 확인하고, 미존재 시 현지화된 TASK_NOT_FOUND를 반환한다.
- 제공 컴포넌트: clawketd 데몬(component.daemon).

## 미확정 (OPEN)
- [ ] OPEN: 허용되는 관계 유형(relation_type)의 어휘 집합과 중복/순환 관계 방지 규칙, 호출 표면(CLI/웹) 확인.
