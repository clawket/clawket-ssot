---
id: capability.handoff
kind: Capability
title: 세션 인계(handoff) 요약
definition: 현재 작업 디렉터리가 속한 프로젝트의 활성 작업 컨텍스트(진행 상황·진행 중/차단 태스크·다음 할 일·미해결 질문·설계 결정)를 한 장의 마크다운으로 모아, 새 세션이 곧바로 이어받을 수 있게 인계하는 기능이다.
purpose: 세션이 끊겨도 다음 세션(또는 다른 에이전트)이 무엇이 진행 중이고 무엇이 막혔고 무엇이 다음인지 한눈에 파악하도록, 흩어진 작업 상태를 단일 인계 문서로 압축한다.
servesPersona:
  - persona.llm-coding-agent
  - persona.human-operator
realizedBy:
  - component.daemon
implementedIn:
  - daemon/src/routes/handoff.rs
relatesTo:
  - to: concept.project
    type: reads
    note: 작업 디렉터리(cwd)로 프로젝트를 해석하거나 단일 프로젝트로 폴백한다.
  - to: concept.plan
    type: reads
    note: 활성(또는 승인·초안) 플랜 하나를 인계 대상 범위로 삼는다.
  - to: concept.task
    type: reads
    note: 진행 중·차단·다음 할 일 태스크를 상태별로 모은다.
  - to: concept.question
    type: reads
    note: 미해결 질문을 인계 항목으로 포함한다.
  - to: concept.knowledge
    type: reads
    note: type=decision 지식을 설계 결정 요약으로 끌어온다.
impacts:
  - domain.work-management
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:capability
---

## 사용자가 할 수 있는 일

세션을 새로 시작하는 에이전트나 사람은, 지금 디렉터리가 속한 프로젝트의 작업 상태를 한 번에 받아볼 수 있다. 인계 요약에는 전체 진행률, 지금 진행 중인 태스크, 막혀 있는 태스크, 다음에 손댈 후보, 아직 답이 없는 질문, 그동안 내려진 설계 결정이 담긴다. 이걸 그대로 읽으면 "어디까지 했고 무엇이 다음인지"를 다시 파헤치지 않아도 된다.

## 행위

- 프로젝트 해석: 전달된 작업 디렉터리로 프로젝트를 찾는다. 못 찾고 등록된 프로젝트가 하나뿐이면 그것으로 폴백하며, 그래도 없으면 "프로젝트 없음"을 알린다.
- 대상 플랜 선택: 활성(또는 승인·초안) 상태 플랜 하나를 인계 범위로 잡는다. 그런 플랜이 없으면 "활성 플랜 없음"으로 끝낸다.
- 진행 요약 집계: 플랜 아래 모든 유닛의 태스크를 모아 완료/전체와 백분율을 계산한다.
- 상태별 모음: 진행 중 태스크(담당자 포함), 차단 태스크, 다음 할 일 후보(상위 일부)를 각각 목록으로 정리한다.
- 미해결 질문·결정: 플랜에 걸린 답 없는 질문과 type=decision 지식의 제목·요약을 인계 항목으로 덧붙인다.
- 출력: 위 내용을 한 편의 마크다운(제목·생성시각 포함)으로 합쳐 돌려준다. 읽기 전용이며 상태를 바꾸지 않는다.

## 시스템 흐름

호출자가 작업 디렉터리를 넘기면 데몬은 그 디렉터리로 프로젝트를 해석하고(또는 단일 프로젝트 폴백), 인계 범위가 될 활성 플랜을 고른다. 그 플랜의 유닛과 태스크를 훑어 진행률을 계산하고, 진행 중·차단·다음 할 일을 상태별로 나눠 담은 뒤, 미해결 질문과 설계 결정을 더해 한 장의 마크다운으로 조립해 응답한다. 데이터는 모두 로컬에서 읽으며 어떤 상태도 변경하지 않는다.

## 어디에 구현되어 있나

- `daemon/src/routes/handoff.rs`: 작업 디렉터리→프로젝트 해석, 활성 플랜 선택, 진행률·상태별 태스크·질문·결정 집계, 마크다운 조립을 한 곳에서 수행한다.

## 미확정 (OPEN)
- [ ] OPEN: 인계 대상 플랜을 "활성 하나"로 좁히는 규칙이 복수 활성 플랜 환경에서 어떻게 우선순위를 정하는지(첫 매칭 외) 추가 정책은 확인되지 않았다.
