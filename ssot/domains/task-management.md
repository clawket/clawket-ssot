---
id: domain.task-management
kind: Domain
title: 작업 관리(Task Management)
purpose: "LLM 코딩 에이전트와 사람 운영자가 같은 작업을 구조화된 계약으로 나누고, 시작·진행·완료를 통제된 상태 전이로 추적하도록 하는 영역이다."
definition: "프로젝트 아래에 플랜(로드맵)을 두고, 그 안을 유닛(그룹핑)으로 나눈 뒤, 사이클(타임박스 스프린트)에 배정된 태스크를 직접 다루는 작업 계층 전체를 포괄한다. 지식·RAG 검색, 검증 수렴 루프, 외부 연동 자체는 이 도메인의 책임 밖이다."
servesPersona: [persona.llm-coding-agent, persona.human-operator]
relatesTo:
  - to: domain.work-management
    type: relates-to
    note: "동일한 작업 계층을 다루는 자매 영역. 본 도메인은 웹 앱이 소비하는 타입·API 표면 관점, work-management 는 데몬·CLI 정책 관점이다."
  - to: domain.agent-runs
    type: feeds
    note: "태스크 시작/완료가 run 기록을 만들어 agent-runs 영역에 흘려보낸다."
  - to: domain.knowledge-rag
    type: relates-to
    note: "태스크·유닛·플랜에 지식이 첨부되어 RAG 검색 대상이 된다."
governedBy:
  - invariant.no-work-without-task
  - invariant.start-needs-active-plan-cycle
  - invariant.one-active-plan-per-project
  - invariant.one-active-cycle-per-unit
  - invariant.unit-no-status
  - invariant.evidence-required-on-done
  - invariant.task-delete-only-draft
  - invariant.preserve-history
realizedBy: [component.cli, component.daemon, component.web-dashboard, component.sqlite-storage]
impacts:
  - concept.project
  - concept.plan
  - concept.unit
  - concept.cycle
  - concept.task
  - concept.backlog
  - capability.register-project
  - capability.plan-and-approve
  - capability.manage-task-lifecycle
  - capability.decompose-task
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:domain"]
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로 — 무엇을·왜·누가·어떤 규칙·무슨 데이터. 코드(테이블·필드·경로·SQL) 옮겨적기 금지(식별자는 provenance/근거에만). 코드 분기/의도 불명은 판정 말고 OPEN. -->

## 목적

이 영역은 "지금 무슨 일을 하고 있는가, 그 일을 시작해도 되는가, 끝났다고 말할 자격이 있는가" 를 통제하는 곳이다. LLM 코딩 에이전트와 사람 운영자가 같은 작업 상태를 공유하면서, 작업이 임의로 시작되거나 근거 없이 완료 처리되지 않도록 게이트를 건다. 작업을 거대한 한 덩어리로 두는 대신 프로젝트 → 플랜 → 유닛 → 사이클 → 태스크의 계층으로 쪼개, 어느 수준에서든 진행 상황을 읽고 책임 단위를 좁힐 수 있게 만든다.

## 경계와 핵심 개념

포함하는 개념:

- 프로젝트(concept.project): 하나 이상의 작업 디렉터리에 묶인 워크스페이스. 구조화된 작업의 출발점이며, 등록되어 있어야 작업을 시작할 수 있다.
- 플랜(concept.plan): 승인된 의도를 담는 로드맵. 초안에서 활성으로 승인되어야 그 아래 태스크를 시작할 수 있다.
- 유닛(concept.unit): 플랜 안에서 태스크를 묶는 순수 그룹핑. 자체 상태나 승인 절차가 없고, 병렬 유닛의 태스크는 여러 에이전트가 동시에 돌릴 수 있다.
- 사이클(concept.cycle): 단일 유닛에 묶인 타임박스 반복(스프린트). 유닛당 하나만 활성일 수 있고, 완료된 사이클은 다시 시작할 수 없다.
- 태스크(concept.task): 직접 다루는 유일한 원자적 작업 단위. 할 일 → 진행 중 → 완료/취소를 거치며, 외부 의존으로 막히면 차단 상태가 된다.
- 백로그(concept.backlog): 아직 어느 사이클에도 배정되지 않은 태스크 모음. 사이클로 끌어다 놓으면 일정에 편입된다.

제외하는 것: 지식 문서의 보관·검색(domain.knowledge-rag), 코드와 시나리오를 함께 진화시키는 검증 수렴(domain.pdd-verification), 실행 이력의 비교·리플레이(domain.agent-runs), 훅이 작업을 허용/차단하는 강제 로직 자체(domain.hook-enforcement)는 이 영역의 책임이 아니다. 이 영역은 "작업 계층의 상태와 전이 규칙" 에 집중한다.

## 기능

- 프로젝트 등록(capability.register-project): 작업 디렉터리를 프로젝트로 올려 구조화된 작업을 활성화한다.
- 플랜 작성·승인(capability.plan-and-approve): 플랜을 제안·등록하고 승인해 태스크 시작 자격을 부여한다.
- 태스크 라이프사이클 관리(capability.manage-task-lifecycle): 태스크를 만들고 시작·완료·취소·차단/해제하며, 완료에는 근거가 따른다.
- 태스크 분해(capability.decompose-task): 큰 태스크를 성공 기준과 분해 정책에 따라 하위 태스크로 쪼개 트리를 만든다.

## 시스템 흐름

전형적인 진행은 다음 순서로 흐른다. 작업 디렉터리를 프로젝트로 등록하고, 그 프로젝트에 플랜을 등록한 뒤 승인해 활성으로 만든다. 플랜 안에 유닛을 두어 작업을 묶고, 유닛에 사이클을 만들어 활성화한다. 활성 플랜과 활성 사이클이 모두 갖춰진 상태에서만 태스크를 시작할 수 있고, 태스크는 할 일에서 진행 중으로 올라간다. 작업을 마치면 근거(무엇을 어떻게 바꿨는지)를 붙여 완료 처리한다. 시작/완료의 매 순간은 실행 기록(run)으로 남아 agent-runs 영역으로 흘러간다. 모든 상태와 전이는 단일 로컬 저장소(component.sqlite-storage)에 보존되며, CLI·웹 대시보드·데몬이 같은 저장소를 본다.

## 다른 도메인과의 관계

- domain.work-management 와는 같은 작업 계층을 다루되 보는 각도가 다르다. 본 영역은 웹 앱이 소비하는 타입·API 표면 관점에서 작업 계층을 정의하고, work-management 는 데몬·CLI 가 강제하는 정책 관점에서 같은 계층을 다룬다.
- domain.agent-runs 로는 태스크의 시작/완료가 실행 기록을 만들어 흘려보낸다(feeds).
- domain.knowledge-rag 와는 태스크·유닛·플랜에 첨부된 지식이 검색 대상이 되는 지점에서 만난다.

## 미확정 (OPEN)
- [ ] OPEN: 본 도메인(task-management)과 domain.work-management 의 경계 분리가 의도적 이중 노드인지, 통합 대상인지 확정 필요. 둘은 같은 작업 계층을 다른 관점에서 서술한다.
- [ ] OPEN: 배치가 지정한 소스 경로(src/types.ts, src/api.ts)는 실제 레포에서 web/src 또는 desktop 하위에 대응한다 — 정확한 정본 경로 확정 필요.
