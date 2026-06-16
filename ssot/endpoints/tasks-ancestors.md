---
id: endpoint.tasks-ancestors
kind: Endpoint
title: 태스크 조상 체인 조회 (GET /tasks/{id}/ancestors)
definition: 지정한 태스크에서 출발해 부모를 따라 루트까지 거슬러 올라가는 조상 태스크 목록을 깊이순으로 돌려준다.
realizedBy:
  - component.daemon
implementedIn:
  - daemon/src/routes/tasks.rs
relatesTo:
  - to: concept.task
    type: reads
    note: 부모 링크를 따라 조상 태스크들을 읽어 체인을 만든다
  - to: concept.execution-envelope
    type: reads
    note: 각 조상에 대해 상속 해석된 실행 봉투를 함께 실어줄 수 있다
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
하나의 태스크가 작업 트리 안에서 어디에 속해 있는지, 그 위쪽 맥락을 한눈에 보기 위한 조회 기능이다. 대상 태스크를 기점으로 그 부모, 부모의 부모를 차례로 따라 올라가 최상위(루트)까지의 조상 목록을 반환한다. 분해(decompose)로 만들어진 깊은 태스크 트리에서 "이 작업이 어떤 상위 의도 아래 놓여 있는가"를 확인하거나, 상위 실행 계약(봉투)을 거슬러 살펴볼 때 쓰인다. 사람이 웹 대시보드에서 태스크 맥락을 펼쳐 보거나, LLM 에이전트가 작업의 상위 의도를 끌어올 때 활용한다.

## 요청 / 응답
요청에는 대상 태스크 식별자와 함께 선택적으로 (1) 얼마나 깊이까지 거슬러 올라갈지(깊이 한도), (2) 각 조상의 실행 봉투를 함께 받을지 여부를 지정할 수 있다. 깊이를 지정하지 않으면 트리 노드 상한선(약 1,000여 개)까지 끝까지 거슬러 올라가며, 봉투 포함은 기본적으로 켜져 있다.

응답은 조상 태스크들의 목록이다. 각 항목은 해당 태스크의 본문 정보와 함께, 대상에서 얼마나 떨어진 단계인지를 나타내는 깊이 값(바로 위 부모가 1단계)을 가진다. 봉투 포함을 요청하면 각 조상에 대해 상위 사슬을 접어 합친 "해석된 실행 봉투"가 함께 실린다.

## 권한 / 제약
로컬 데몬에 붙는 호출자가 사용한다. 존재하지 않는 태스크를 가리키면 찾을 수 없음으로 응답한다. 깊이 값은 음수가 들어와도 0 이상으로 보정되며, 반환 노드 수에는 트리 전체 상한이 적용되어 비정상적으로 큰 트리에서도 과도한 응답이 생기지 않는다.

## provenance
구현은 데몬의 태스크 라우터(`daemon/src/routes/tasks.rs`)의 조상 조회 핸들러에 있으며, 부모 링크를 재귀적으로 따라가는 조상 탐색과 봉투 상속 해석을 결합한다. 봉투를 제공하는 컴포넌트는 데몬(component.daemon)이다.

## 미확정 (OPEN)
- [ ] OPEN: 깊이 한도와 봉투 미포함 옵션이 사람 대시보드/에이전트 각각에서 실제로 어떻게 쓰이는지(소비자 확정) 확인 필요.
