---
id: endpoint.task-subtasks
kind: Endpoint
title: 하위 태스크 생성 · 분해 제안 (subtasks / decompose)
definition: 부모의 봉투·소속을 물려받는 자식 태스크를 만들고, 데몬이 성공 기준에서 하위 태스크를 제안하도록 요청하는 웹 대시보드용 분해 기능.
realizedBy: [component.daemon]
implementedIn:
  - web/src/api.ts
  - web/src/features/decomposition/SuggestionPanel.tsx
relatesTo:
  - { to: concept.task, type: mutates, note: "부모 아래에 자식 태스크를 새로 만들어 트리를 확장한다" }
  - { to: concept.execution-envelope, type: reads, note: "분해 제안은 부모 봉투의 성공 기준에서 하위 태스크를 도출한다" }
governedBy: []
impacts: []
consumedBy: [component.web-dashboard, component.web-api-client, capability.decompose-task, screen.task-detail-drawer]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

큰 태스크를 더 작은 태스크들로 쪼개는(분해하는) 두 가지 기능을 한 묶음으로 다룬다. 하나는 "이 부모 밑에 자식 태스크를 만들어라"라는 직접 생성이고, 다른 하나는 "이 태스크를 어떻게 쪼개면 좋을지 제안해 달라"라는 분해 제안이다. 웹 대시보드의 분해 패널에서, 사람이 태스크를 직접 자식으로 쪼개거나 데몬이 내놓은 분해안을 받아 트리를 만들 때 쓰인다.

## 요청 / 응답

자식 태스크 생성은 부모 태스크를 지정하고 새 자식의 제목을 비롯한 정보를 보낸다. 핵심 동작은 상속이다 — 자식이 어느 유닛·사이클에 속할지를 따로 주지 않으면 부모의 소속을 그대로 물려받고, 자식의 부모 관계는 경로에 담긴 부모로 고정된다. 자식의 실행 봉투도 기본적으로 부모 것을 물려받되, 필요하면 일부를 덮어쓸 값(봉투 오버라이드)을 함께 넘겨 새 자식의 봉투를 미리 채워 둘 수 있다. 응답은 새로 만들어진 자식 태스크다.

분해 제안은 대상 태스크와 분해 전략·최대 깊이 같은 힌트를 보낸다. 데몬은 그 태스크의 (풀어낸) 실행 봉투에 적힌 성공 기준을 바탕으로 어떤 하위 태스크로 쪼개면 좋을지 후보를 산출해 돌려준다. 이 제안 결과는 CLI의 분해 도구가 내놓는 것과 같은 형태이며, 사람은 제안 중 일부를 골라 실제 자식 태스크로 수락한다(수락은 위의 자식 생성 경로로 이어진다).

## 권한 / 제약

- 자식 생성은 상태를 바꾸는 쓰기 작업이다. 소속(유닛·사이클) 미지정 시 부모 상속이 기본 규칙이다.
- 분해 제안 자체는 후보를 계산해 보여 줄 뿐 태스크를 만들지 않는다 — 실제 트리 변경은 사람이 제안을 수락해 자식을 생성할 때 일어난다.
- 분해 제안의 품질은 부모 태스크에 실행 봉투(특히 성공 기준)가 채워져 있느냐에 달려 있다. 성공 기준이 비어 있으면 제안이 빈약할 수 있다.
- 분해 계산의 단일 진실 주체는 데몬이다(웹·CLI가 같은 결과를 보도록).

## provenance

웹 클라이언트(`web/src/api.ts`)가 데몬의 하위태스크 생성(`POST /tasks/{parent}/subtasks`)과 분해 제안(`POST /tasks/{id}/decompose`)을 감싸고, `web/src/features/decomposition/SuggestionPanel.tsx`가 제안을 받아 사람이 고르는 UI를 제공한다. 분해 도출과 봉투 상속 규칙은 데몬(component.daemon)이 책임진다. 코드가 사라져도 "부모 봉투·소속을 상속하는 자식 생성 + 성공 기준 기반 분해 제안"이라는 의미만으로 재구현이 가능해야 한다.

## 미확정 (OPEN)
- [ ] OPEN: 분해 전략(auto/scoped/by-repo)별로 후보 도출 방식이 어떻게 달라지는지 데몬 분해 로직에서 확인해 자연어로 보강할 것.
