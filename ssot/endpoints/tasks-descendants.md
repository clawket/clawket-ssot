---
id: endpoint.tasks-descendants
kind: Endpoint
title: 태스크 자손 조회 (GET /tasks/{id}/descendants)
definition: 지정한 태스크 아래로 펼쳐진 하위 태스크들을 깊이순 또는 너비순으로 훑어 자손 목록을 돌려준다.
realizedBy:
  - component.daemon
implementedIn:
  - daemon/src/routes/tasks.rs
relatesTo:
  - to: concept.task
    type: reads
    note: 부모-자식 링크를 따라 하위 태스크들을 읽어 트리를 펼친다
  - to: concept.execution-envelope
    type: reads
    note: 각 자손에 대해 상속 해석된 실행 봉투를 함께 실어줄 수 있다
governedBy: []
governs: []
dependsOn: []
consumesApi: []
providesApi: []
impacts:
  - domain.work-management
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의
하나의 태스크가 어떤 하위 작업들로 분해되어 있는지를 펼쳐 보기 위한 조회 기능이다. 대상 태스크를 뿌리로 삼아 그 아래에 매달린 자식, 손자 태스크를 따라 내려가며 자손 전체를 반환한다. 태스크 분해 후 "이 큰 작업이 어떤 하위 작업들로 쪼개졌는가"를 확인하거나, 상위 태스크 완료 여부를 따지기 위해 남은 하위 작업을 점검할 때 쓰인다. 조상 조회가 위쪽 맥락을 본다면, 이 기능은 아래쪽 작업 구조를 본다.

## 요청 / 응답
요청에는 대상 태스크 식별자와 함께 선택적으로 (1) 몇 단계까지 내려갈지(깊이 한도), (2) 훑는 순서를 너비 우선으로 할지 여부, (3) 각 자손의 실행 봉투를 함께 받을지 여부를 지정할 수 있다. 깊이를 지정하지 않으면 트리 노드 상한선까지 끝까지 내려가며, 순서를 별도로 지정하지 않으면 깊이 우선으로 펼친다. 봉투 포함은 기본적으로 켜져 있다.

응답은 자손 태스크들의 목록이다. 각 항목은 태스크 본문 정보와 함께 대상으로부터의 깊이 값을 가지며, 봉투 포함을 요청하면 상위 사슬을 접어 합친 해석된 실행 봉투가 함께 실린다.

## 권한 / 제약
로컬 데몬 호출자가 사용한다. 존재하지 않는 태스크를 가리키면 찾을 수 없음으로 응답한다. 깊이 값은 최소 1단계로 보정되고, 반환되는 노드 수에는 트리 전체 상한(약 1,000여 개)이 걸려 있어 매우 깊거나 넓은 트리에서도 응답 폭주를 막는다.

## provenance
구현은 데몬의 태스크 라우터(`daemon/src/routes/tasks.rs`)의 자손 조회 핸들러에 있으며, 부모-자식 링크를 깊이 우선 또는 너비 우선으로 펼치는 자손 탐색과 봉투 상속 해석을 결합한다. 봉투를 제공하는 컴포넌트는 데몬(component.daemon)이다.

## 미확정 (OPEN)
- [ ] OPEN: 너비 우선/깊이 우선 순서가 실제 소비처(대시보드 트리 뷰 등)에서 어떤 의미로 쓰이는지 확인 필요.
