---
id: endpoint.tasks-subtree
kind: Endpoint
title: 데몬 GET /tasks/{id}/subtree
definition: 한 태스크를 뿌리로 그 아래 하위 트리 전체를 한 번에 조회하는 통로. 뿌리부터 자손까지 깊이 정보와 함께 펼쳐 돌려준다.
realizedBy:
  - component.daemon
implementedIn:
  - daemon/src/routes/tasks.rs
relatesTo:
  - to: concept.task
    type: reads
    note: 뿌리 태스크와 그 자손 전체를 읽어 트리로 반환한다
  - to: concept.execution-envelope
    type: reads
    note: 각 노드의 해소된 실행 봉투를 옵션으로 함께 반환한다
  - to: component.daemon
    type: backed-by
    note: 트리를 조회해 펼쳐 반환하는 로컬 데몬
governedBy: []
impacts:
  - concept.task
consumedBy:
  - capability.decompose-task
  - endpoint.cli-task
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의
한 태스크를 뿌리로 삼아, 그 아래 매달린 하위 태스크 전체(자식·손주…)를 한 번에 조회하는 통로다. 뿌리 자신을 0단계로 두고 그 아래 자손들을 깊이 표시와 함께 펼쳐 돌려주어, 큰 작업이 어떻게 잘게 나뉘어 있는지 트리 모양을 한눈에 파악하게 한다. 분해해 둔 작업의 전모를 확인하거나, 트리 단위로 진행 상황을 살필 때 쓴다.

## 요청 / 응답
보내는 정보는 펼치는 방식을 조절하는 선택 항목이다 — 몇 단계 깊이까지 펼칠지(깊이), 자손을 너비 우선으로 훑을지 여부(순회 순서), 각 노드에 실행 봉투까지 함께 담을지 여부다. 깊이를 생략하면 시스템 상한(노드 수 한도)까지 펼치고, 봉투 포함 여부를 생략하면 기본으로 포함한다. 응답으로는 평평하게 나열된 노드 목록을 돌려주는데, 첫 항목은 깊이 0의 뿌리이고 이어서 자손들이 각자의 깊이 값과 함께 따라온다. 봉투 포함을 켜면 각 노드의 해소된 실행 계약도 함께 실린다. 뿌리 태스크가 없으면 "찾을 수 없음"으로 응답한다.

## 권한 / 제약
데몬이 로컬 루프백에만 바인딩되므로 같은 기기 안에서만 호출된다. 트리가 무한정 커지지 않도록 반환 노드 수에 상한(약 1,024개)이 걸려 있어, 거대한 트리는 그 한도까지만 펼쳐진다. 이 통로는 읽기 전용이라 상태를 바꾸지 않는다. 비슷한 조회로 자손만(뿌리 제외) 또는 조상 사슬만 보는 통로가 따로 있으며, 이쪽은 "뿌리 포함 전체 하위 트리"를 한 응답에 담는다는 점이 다르다.

## provenance
근거는 `daemon/src/routes/tasks.rs`의 라우트 등록(`/tasks/{id}/subtree`, GET → `get_subtree`)과 핸들러 `get_subtree`다. 핸들러는 뿌리를 깊이 0 노드로 먼저 담고, 깊이>0이면 `tasks::descendants`(깊이·BFS 여부·노드 상한 `TREE_NODE_CAP`=1024 적용)로 자손을 모아 각 노드에 깊이와(옵션) 해소된 봉투(`resolve_envelope`)를 실어 평평한 목록으로 반환한다. 쿼리 항목은 깊이(`depth`), 순회 순서(`order=bfs`), 봉투 포함(`include_envelope`, 기본 true)이다. 조회 주체는 로컬 데몬(component.daemon)이다.

## 미확정 (OPEN)
- [ ] OPEN: 노드 상한(1024) 도달 시의 잘림 표시 방식(부분 결과임을 호출 측에 알리는지)과 기본 순회 순서(미지정 시 DFS인지)는 `tasks::descendants` 본체를 정독해 확인 필요.
