---
id: endpoint.task-tree
kind: Endpoint
title: 태스크 트리 조회 (subtree / ancestors / descendants)
definition: 한 태스크를 기준으로 그 하위 트리·조상 체인·자손 트리를 깊이·순회순서·봉투 포함 여부 옵션과 함께 조회하는 웹 대시보드용 조회 묶음.
realizedBy: [component.daemon]
implementedIn:
  - web/src/api.ts
  - web/src/components/TaskTreeView.tsx
relatesTo:
  - { to: concept.task, type: reads, note: "부모-자식 관계로 연결된 태스크들을 트리/체인 형태로 읽어 온다" }
  - { to: concept.execution-envelope, type: reads, note: "각 노드의 실행 봉투를 함께 펼칠 수 있다(include_envelope 옵션)" }
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

하나의 태스크를 출발점으로 삼아 그 태스크가 속한 트리 구조를 세 방향으로 펼쳐 보는 조회 기능 묶음이다. 태스크는 부모-자식으로 분해될 수 있는데, 이 조회는 (1) 한 태스크를 뿌리로 한 아래쪽 전체 트리, (2) 그 태스크에서 위로 거슬러 올라가는 조상들의 줄(부모 → 그 부모 …), (3) 자기 자신을 뺀 자손들의 트리를 각각 돌려준다. 웹 대시보드에서 태스크 상세를 열어 "이 일이 어떤 큰 일의 일부인지(조상)", "이 일이 어떤 하위 일들로 쪼개졌는지(자손)"를 그래픽 트리로 보여 줄 때 바탕이 된다.

## 요청 / 응답

요청에는 기준이 되는 태스크 하나를 지정하고, 다음 옵션을 곁들일 수 있다.

- 얼마나 깊이까지 펼칠지(깊이 한도). 예컨대 "바로 아래 자식만"으로 한정하면 한 단계만 돌려준다.
- 펼치는 순서(깊이 우선 / 너비 우선).
- 각 노드에 실행 봉투까지 함께 담을지 여부. 구조만 필요하면 봉투를 빼서 조회 비용을 줄일 수 있다(봉투 포함이 기본값).

응답은 태스크 노드들의 목록이다.

- 하위 트리 조회는 항상 기준 태스크 자신을 첫 항목으로 포함한다.
- 조상 조회는 가장 먼 조상부터 시작해 바로 위 부모로 끝나며, 기준 태스크 자신은 포함하지 않는다(화면 쪽에서 직접 앞에 붙여 빵부스러기 경로를 그린다).
- 자손 조회는 자기 자신을 빼고 아래쪽만 돌려주며, 깊이를 1로 주면 바로 아래 자식들만 나온다.

각 노드는 제목·상태·상위 관계 등 태스크의 일반 속성과(옵션에 따라) 실행 봉투를 담는다.

## 권한 / 제약

- 로컬 데몬에 붙는 읽기 전용 조회로 상태를 바꾸지 않는다.
- 트리가 무한정 커지지 않도록 한 번에 돌려주는 노드 수에 상한이 있다(과도하게 큰 트리는 상한선에서 잘린다). 따라서 깊이 옵션은 성능·가독성을 위한 안전장치이기도 하다.
- 봉투 포함은 노드마다 봉투를 풀어내는 비용이 있으므로, 구조만 필요한 화면에서는 봉투를 빼고 호출하는 것이 권장된다.

## provenance

웹 클라이언트(`web/src/api.ts`)가 데몬의 태스크 하위트리·조상·자손 경로(`/tasks/{id}/subtree`, `/ancestors`, `/descendants`)를 감싸는 함수들을 제공하고, `web/src/components/TaskTreeView.tsx`가 이를 트리 UI로 렌더링한다. 실제 트리 계산과 노드 수 상한은 데몬(component.daemon)이 책임진다. 코드가 사라져도 "한 태스크 기준 위·아래·자기 트리를 깊이/순서/봉투 옵션으로 조회"라는 의미만으로 재구현이 가능해야 한다.

## 미확정 (OPEN)
- [ ] OPEN: 노드 수 상한의 정확한 값과 상한 초과 시 동작(잘림 표시 여부)을 데몬 라우트에서 확인해 보강할 것.
