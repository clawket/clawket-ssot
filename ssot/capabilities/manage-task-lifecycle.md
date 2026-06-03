---
id: capability.manage-task-lifecycle
kind: Capability
title: 태스크 라이프사이클 관리
definition: 작업의 최소 단위인 태스크를 만들고, 시작·완료·취소하며, 외부 의존이 생기면 차단·해제하는 일련의 흐름을 한 사람(또는 에이전트)이 직접 다루는 기능. 완료 처리에는 외부에서 확인 가능한 근거(evidence)가 반드시 따라붙는다.
purpose: LLM 에이전트와 사람이 같은 작업 단위를 일관된 상태로 추적하고, 무엇을·언제·왜 끝냈는지 근거와 함께 남겨 세션이 끊겨도 작업의 진실이 사라지지 않게 한다.
servesPersona:
  - persona.llm-coding-agent
  - persona.human-operator
realizedBy:
  - component.cli
  - component.daemon
implementedIn:
  - daemon/src/routes/tasks.rs
relatesTo:
  - to: concept.task
    type: mutates
    note: 태스크의 상태를 todo→in_progress→done/cancelled(또는 blocked)로 전이시킨다
  - to: concept.run
    type: mutates
    note: 태스크를 시작하면 보통 훅이 실행 기록(run)을 자동으로 만든다
  - to: concept.comment
    type: mutates
    note: 취소를 파괴적 삭제 대신 코멘트로 히스토리를 보존하는 수단으로 쓴다
  - to: concept.cycle
    type: depends-on
    note: 태스크를 시작하려면 active 사이클에 배정되어 있어야 한다
  - to: concept.plan
    type: depends-on
    note: 태스크 시작 자격은 소속 플랜이 active일 때만 부여된다
impacts:
  - concept.task
  - concept.run
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:capability
---

## 사용자가 할 수 있는 일

사람이든 LLM 에이전트든, 실제로 직접 손대는 작업 단위는 태스크 하나다. 플랜·유닛·사이클은 태스크를 담고 묶는 그릇이고, 일을 시작하고 끝내는 행위는 모두 태스크 위에서 일어난다. 이 기능은 그 태스크를 생애 전체에 걸쳐 다룰 수 있게 한다 — 새로 만들고, 작업을 시작하고, 끝내고, 필요하면 취소하거나, 외부 사정으로 더 못 나아갈 때 막아두었다가 다시 푼다.

핵심 가치는 두 가지다. 첫째, 완료에는 근거가 강제된다 — "끝냈다"고 선언하는 것만으로는 done이 되지 않고, 외부에서 확인 가능한 근거를 함께 남겨야 한다. 둘째, 취소조차 파괴가 아니라 기록이다 — 작업을 지워 없애는 대신 취소된 이력을 코멘트로 남겨 "왜 안 했는지"가 사라지지 않는다.

## 행위

- 태스크 생성: 의도·성공 기준 등을 담아 새 태스크를 만든다. 사이클을 지정하지 않으면 백로그로 들어간다.
- 시작: 태스크를 진행 중 상태로 전환한다. 이때 소속 플랜이 active이고 active 사이클에 배정되어 있어야 한다.
- 완료: 근거(evidence)를 함께 제시해 done으로 닫는다. 근거 없이는 완료할 수 없다.
- 취소: 더 진행하지 않기로 한 태스크를 cancelled로 닫되, 사유를 코멘트로 남겨 히스토리를 보존한다.
- 차단·해제: 외부 의존(다른 작업·사람 결정 등)으로 진행이 막히면 blocked로 두었다가, 의존이 풀리면 다시 진행 가능 상태로 되돌린다.
- 본문 이어쓰기·조회: 태스크 설명을 누적하거나 현재 상태를 확인한다.

## 시스템 흐름

사람이나 에이전트가 태스크 상태를 바꾸려 하면, 그 요청은 데몬으로 전달되어 상태 전이 규칙(active 플랜·active 사이클 요건, 완료 시 근거 요건)을 통과한 뒤에야 반영된다. 완료가 받아들여지면 데몬은 그 변경을 감사 추적에 남기고, 한 태스크가 done/cancelled로 닫혀 같은 그릇(유닛·사이클·플랜)의 모든 형제가 종료 상태가 되면 상위 단위의 완료를 자동으로 이어 처리한다. LLM 에이전트가 활성 태스크 없이 변경 작업을 시도하면 그 이전 단계의 훅 가드가 먼저 막아서므로, 이 기능은 항상 "지금 다루는 활성 태스크"가 있다는 전제 위에서 동작한다.

## 어디에 구현되어 있나

태스크 상태 전이와 완료 시 근거 검증 같은 결정적 규칙은 데몬의 태스크 라우트(`daemon/src/routes/tasks.rs`)가 소유하며, 사람·에이전트는 CLI의 태스크 명령을 통해 이 규칙에 접근한다. 자동 실행 기록 생성과 활성 태스크 강제는 플러그인 훅 계층의 책임으로, 이 기능의 전제 조건을 형성한다.

## 미확정 (OPEN)
- [ ] OPEN: 완료 시 evidence 강제의 정확한 검증 지점(데몬 라우트 vs SubagentStop 훅)의 책임 분담 — 두 곳 모두 관여하는 것으로 보이나 권위 있는 단일 지점 확인 필요
- [ ] OPEN: blocked↔진행 가능 상태로의 해제(unblock)가 자동(의존 태스크 done 시 auto-unblock)인지 수동 명령인지의 경계 명시 필요
