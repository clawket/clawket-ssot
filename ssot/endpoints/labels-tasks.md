---
id: endpoint.labels-tasks
kind: Endpoint
title: 라벨별 태스크 조회·라벨 부착/해제 (데몬 GET /labels/{label}/tasks 외)
definition: 특정 라벨이 붙은 태스크 목록을 조회하고, 태스크에 라벨을 붙이거나 떼는 데몬 엔드포인트.
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/labels.rs]
relatesTo:
  - { to: concept.task, type: reads, note: "특정 라벨이 붙은 태스크 집합 조회" }
  - { to: concept.task, type: mutates, note: "태스크에 라벨 부착/해제" }
governedBy: []
impacts: [concept.task]
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
태스크에 자유 분류 태그(라벨)를 붙이고 떼는 동작과, 특정 라벨이 달린 태스크들을 한꺼번에 찾아오는 조회를 제공한다. 라벨은 플랜·유닛·사이클 같은 구조적 묶음과 별개로, 가로지르는 주제(예: 특정 기능 영역, 긴급도 표시 등)로 태스크를 골라내고 싶을 때 쓰는 가벼운 분류 수단이다.

## 요청 / 응답
- 조회 요청은 라벨 이름을 경로에 담아 보내고, 응답으로 그 라벨이 붙은 태스크 목록을 돌려준다.
- 부착 요청은 대상 태스크와 붙일 라벨 이름을 보내고, 응답으로 라벨이 반영된 태스크 한 건을 돌려준다.
- 해제 요청은 대상 태스크와 뗄 라벨을 경로에 담아 보내고, 라벨이 빠진 태스크를 돌려준다.

## 권한 / 제약
로컬 데몬 내부 API로 외부 인증은 없다. 라벨은 자유 문자열로 사전 정의된 어휘 제약이 코드 수준에 보이지 않는다(부착 시 검증 없이 받음). 조회는 해당 라벨이 매핑된 태스크 식별자를 모아 각 태스크 본문을 채워 반환한다.

## provenance
- 라우터·핸들러: `daemon/src/routes/labels.rs`의 `POST /tasks/{id}/labels`(부착), `DELETE /tasks/{id}/labels/{label}`(해제), `GET /labels/{label}/tasks`(라벨별 조회). 조회는 task_labels 매핑 테이블에서 태스크 식별자를 골라 태스크를 채운다.
- 제공 컴포넌트: clawketd 데몬(component.daemon).

## 미확정 (OPEN)
- [ ] OPEN: 라벨 어휘의 사전 정의/검증 여부와, 이 엔드포인트를 호출하는 표면(CLI 하위명령·웹 뷰)이 어디인지 확인(현재 consumedBy 미상).
