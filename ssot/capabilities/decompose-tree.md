---
id: capability.decompose-tree
kind: Capability
title: 태스크 분해 제안과 트리 탐색
purpose: "큰 태스크를 실행 봉투의 성공 기준에 따라 하위 태스크로 쪼개 제안받고, 만들어진 작업 트리를 부모-자식 방향으로 자유롭게 훑어보며 전체 구조를 파악한다."
servesPersona:
  - persona.llm-coding-agent
  - persona.human-operator
realizedBy:
  - component.cli
  - component.daemon
implementedIn:
  - cli/src/main.rs
relatesTo:
  - { to: capability.decompose-task, type: relates-to, note: "decompose-task 는 분해 제안과 선택 수락의 핵심 가치를 정의하고, 본 capability 는 그것을 트리 조회(부모 사슬·하위 트리·자손 나열)까지 확장한 CLI 표면이다." }
  - { to: concept.execution-envelope, type: depends-on, note: "분해 제안은 부모 태스크에 서명된 실행 봉투의 성공 기준과 분해 정책(최대 깊이)을 입력으로 삼는다." }
  - { to: concept.task, type: relates-to, note: "분해 결과로 만들어지는 자식 태스크와, 트리 탐색이 대상으로 삼는 노드가 모두 태스크다." }
  - { to: endpoint.cli-task, type: realizes, note: "task 서브커맨드의 decompose/tree/ancestors/descendants 액션으로 노출된다." }
impacts:
  - concept.task
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:capability
---

## 사용자가 할 수 있는 일

규모가 큰 태스크를 손으로 잘게 나누는 대신, 그 태스크에 서명된 실행 계약(성공 기준과 분해 정책)을 근거로 시스템이 하위 태스크 후보를 자동으로 뽑아 보여준다. 사용자는 제안 목록을 먼저 미리보기로 확인하고, 그중 원하는 항목만 골라 실제 자식 태스크로 만들 수 있다. 무턱대고 다 만들지 않고 "제안 → 검토 → 선택 수락" 단계를 거치므로, 분해가 의도에서 벗어나는 것을 막는다.

이미 만들어진 작업 트리에 대해서는 위(부모 방향)와 아래(자식 방향)를 모두 훑어볼 수 있다. 한 태스크를 기준으로 그 조상 사슬, 전체 하위 트리, 또는 모든 자손을 나열해 작업이 어떤 맥락에 놓여 있고 어디까지 펼쳐졌는지 한눈에 파악한다. 사람 운영자와 LLM 에이전트 모두 같은 구조를 같은 방식으로 들여다본다.

## 행위

- 분해 제안 받기: 부모 태스크의 성공 기준과 분해 정책으로부터 하위 태스크 후보를 만들어 번호 매긴 미리보기로 보여준다. 분해 깊이 예산을 정할 수 있고, 봉투의 최대 깊이 정책과 대조된다.
- 선택 수락: 미리보기에서 전체 또는 일부(예: 1, 3, 5번)만 골라 실제 자식 태스크로 생성한다. 별도 수락 지시 없이는 어떤 것도 만들어지지 않는다(미리보기 전용).
- 하위 트리 조회: 한 태스크를 뿌리로 그 자신과 모든 자손을 트리 형태로 그린다. 각 줄에 봉투의 의도 요약을 덧붙여 볼 수도 있다.
- 조상 조회: 한 태스크의 부모 사슬을 평면 목록으로 반환한다. 사슬 깊이를 제한할 수 있고 스크립트로 처리하기 좋은 출력 형식을 고를 수 있다.
- 자손 조회: 한 태스크의 모든 자손을 평면 목록으로 나열한다. 깊이 우선/너비 우선 순회 중 선택한다.

## 시스템 흐름

CLI에서 분해를 요청하면 부모 태스크의 실행 봉투를 읽어 성공 기준과 분해 정책을 해석하고, 그로부터 하위 태스크 후보와 정책 위반 사항을 만들어 사용자에게 미리보기로 제시한다. 사용자가 수락 대상을 지정하면 그때 비로소 자식 태스크가 생성된다. 트리·조상·자손 조회는 데몬이 보유한 태스크 관계 데이터를 순회해 구조를 반환하며, 분해 결과로 생긴 부모-자식 관계가 바로 이 조회의 입력이 된다.

## 어디에 구현되어 있나

CLI(Rust 바이너리)의 명령 정의에 분해(decompose)와 트리·조상·자손 조회(tree/ancestors/descendants) 액션이 task 서브커맨드 아래 모여 있다. 실제 후보 생성과 트리 순회 자체는 데몬 측에서 수행되며 CLI는 그 결과를 렌더링한다.

## 미확정 (OPEN)
- [ ] OPEN: 데몬 측 분해 제안 로직(success_criteria → 후보 생성 알고리즘)과 트리 순회의 노드 상한(TREE_NODE_CAP) 세부는 데몬 코드 확인 필요.
