---
id: capability.decompose-task
kind: Capability
title: 태스크 분해(decompose)
definition: 부모 태스크가 가진 성공 기준(success_criteria)과 분해 정책(decomposition_policy)을 입력으로, 결정적 규칙에 따라 하위 태스크 후보 목록을 제안하고, 사람이 그중 원하는 것만 골라 수락해 태스크 트리를 만드는 기능. LLM 호출 없이 규칙 기반으로 후보를 산출한다.
purpose: 큰 작업을 검증 가능한 작은 단위로 쪼개되, 무엇을 근거로 쪼갰는지(어느 성공 기준에서 나온 하위 작업인지)를 추적 가능하게 하고, 분해 정책 위반을 사람에게 경고로 드러낸다.
servesPersona:
  - persona.llm-coding-agent
  - persona.human-operator
realizedBy:
  - component.cli
  - component.daemon
  - component.web-dashboard
implementedIn:
  - daemon/src/decomposition/suggest.rs
  - daemon/src/routes/tasks.rs
  - web/src/features/decomposition/SuggestionPanel.tsx
  - desktop/apps/desktop/src/features/decomposition/SuggestionPanel.tsx
relatesTo:
  - to: concept.task
    type: mutates
    note: 수락된 후보가 부모 아래 하위 태스크로 생성되어 트리를 이룬다
  - to: concept.execution-envelope
    type: reads
    note: 부모의 실행 봉투(success_criteria·decomposition_policy)를 읽어 후보와 정책 위반을 도출한다
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

하나의 큰 태스크를 그냥 통째로 진행하는 대신, 그 태스크가 약속한 성공 기준을 기준으로 더 작은 하위 태스크로 쪼갤 수 있다. 분해는 마법이 아니라 결정적 규칙이다 — 부모 태스크에 적어 둔 성공 기준 한 줄 한 줄이 곧 하위 태스크 후보 하나가 되고, 그 후보가 "어느 성공 기준에서 왔는지"가 근거로 함께 붙는다. 사람은 제안된 후보 전체를 받아들이는 게 아니라 필요한 것만 골라 수락한다.

이 기능의 가치는 추적 가능성이다. 분해 결과는 임의로 만들어진 작업 목록이 아니라 부모의 약속(성공 기준)에서 기계적으로 파생된 것이므로, 나중에 "이 하위 작업이 왜 있지?"라는 질문에 항상 답이 있다. 동시에 분해 정책(최소·최대 하위 개수, 최대 깊이)에 어긋나면 그 사실이 경고로 드러나, 너무 잘게 또는 너무 성기게 쪼개는 것을 사람이 알아챌 수 있다.

## 행위

- 분해 제안 요청: 부모 태스크에 대해 분해 전략(auto / scoped / by-repo)과 최대 깊이를 정해 후보 목록을 받는다.
- 후보 검토: 각 후보가 어느 성공 기준에서 파생됐는지, 부모로부터 무엇을 물려받는지(의도·프롬프트 템플릿·분해 정책)를 본다.
- 정책 위반 확인: 후보 수가 분해 정책의 최소·최대 범위를 벗어나거나, 요청한 깊이가 정책 한도를 넘거나, 성공 기준이 아예 없으면 경고·오류로 표시된다.
- 선택 수락: 제안된 후보 중 원하는 것만 골라 하위 태스크로 확정한다.

## 시스템 흐름

분해를 요청하면 데몬은 부모 태스크의 실행 봉투를 상속 병합해 해석한 뒤, 그 성공 기준을 한 줄씩 떼어 하위 태스크 후보로 만든다. 후보 산출에는 LLM도 DB 접근도 개입하지 않는 순수 규칙 계산이라, 같은 입력에는 항상 같은 결과가 나온다. 데몬은 동시에 분해 정책과 대조해 최소·최대 하위 개수, 최대 깊이, 성공 기준 부재 같은 위반을 모아 함께 돌려준다(전략이 정의되지 않은 값이면 auto로 보정). 그 결과는 웹/데스크탑의 분해 제안 패널에 표시되고, 사람이 고른 후보만 실제 하위 태스크로 생성되어 부모 아래 트리에 편입된다.

## 어디에 구현되어 있나

후보 산출의 핵심 규칙은 데몬의 분해 제안기(`daemon/src/decomposition/suggest.rs`)에 순수 함수로 들어 있고, HTTP 진입은 태스크 라우트의 분해 엔드포인트(`daemon/src/routes/tasks.rs`)가 입력 수집과 JSON 직렬화를 맡는다. 사람이 후보를 보고 고르는 화면은 웹과 데스크탑 양쪽의 분해 제안 패널(`web/src/features/decomposition/SuggestionPanel.tsx`, `desktop/apps/desktop/src/features/decomposition/SuggestionPanel.tsx`)이다.

## 미확정 (OPEN)
- [ ] OPEN: 후보 "수락→실제 하위 태스크 생성"의 책임 위치(웹 클라이언트가 개별 task create를 호출하는지, 전용 수락 엔드포인트가 있는지) 확인 필요 — 제안기는 후보만 산출할 뿐 생성은 하지 않는다
- [ ] OPEN: scoped / by-repo 전략의 scope_hint가 실제 후보 생성에 미치는 효과(현재는 표시용 힌트 문자열로만 보임) 확인 필요
