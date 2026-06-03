---
id: endpoint.tasks-labels
kind: Endpoint
title: 태스크 라벨 추가/제거
definition: "POST /tasks/{id}/labels · DELETE /tasks/{id}/labels/{label} — 태스크에 라벨을 붙이거나 떼고, 라벨로 태스크를 역조회한다."
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/labels.rs]
relatesTo:
  - { to: concept.task, type: mutates, note: "태스크에 라벨을 추가하거나 제거한다" }
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
태스크에 자유로운 꼬리표(라벨)를 붙여 분류·필터에 쓰는 기능이다. 이 동작 묶음으로 태스크에 라벨을 하나 붙이고, 더 이상 필요 없는 라벨을 떼며, 반대로 특정 라벨이 붙은 태스크들을 한 번에 모아 볼 수 있다.

## 요청 / 응답
라벨 추가는 대상 태스크와 붙일 라벨 이름을 받아, 라벨이 반영된 태스크 정보를 돌려준다. 라벨 제거는 대상 태스크와 뗄 라벨 이름을 경로로 지목하고, 마찬가지로 갱신된 태스크 정보를 돌려준다. 라벨로 역조회하면 그 라벨이 붙은 태스크 목록을 돌려준다.

## 권한 / 제약
대상 태스크가 없으면 "찾을 수 없음"으로 응답한다. 라벨 이름은 자유 문자열이며, 별도의 고정 어휘 제약은 코드 수준에서 강제되지 않는다.

## provenance
구현은 데몬의 라벨 라우터(daemon/src/routes/labels.rs)에 있다. 라벨↔태스크 매핑은 별도 매핑 저장소에 보관되며, 역조회는 그 매핑에서 태스크 식별자들을 모아 각 태스크를 다시 읽어 구성한다.

## 미확정 (OPEN)
- [ ] OPEN: 같은 라벨을 중복 추가했을 때의 동작(멱등 여부), 라벨 이름의 길이·형식 제약은 확인 필요.
