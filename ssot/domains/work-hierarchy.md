---
id: domain.work-hierarchy
kind: Domain
title: 작업 계층 (Project→Plan→Unit→Task→Cycle)
purpose: "LLM 주도 개발 작업을 사람과 에이전트가 함께 읽고 쓸 수 있는 구조화된 상태 트리로 묶어 추적한다."
definition: "프로젝트를 최상위로, 그 아래 계획·유닛·태스크가 부모-자식 트리를 이루고, 사이클이 실행 단위(스프린트)를 가로질러 묶는 작업 데이터 모델. 지식·근거·검색 등 부가 영역은 제외하고 순수 작업 골격만 포함한다."
servesPersona:
  - persona.llm-coding-agent
  - persona.human-operator
relatesTo:
  - { to: domain.work-management, type: belongs-to, note: "작업 관리 도메인의 데이터 골격 부분을 이루는 하위 구조" }
  - { to: domain.knowledge-rag, type: relates-to, note: "지식·근거 항목이 이 계층의 태스크·유닛·계획에 첨부된다" }
  - { to: domain.qa-discover, type: relates-to, note: "QA 라운드가 이 계층 위에 계획·유닛·사이클·태스크를 자동 생성한다" }
governedBy:
  - invariant.start-needs-active-plan-cycle
  - invariant.one-active-plan-per-project
  - invariant.one-active-cycle-per-unit
  - invariant.cycle-unit-required
  - invariant.unit-no-status
  - invariant.evidence-required-on-done
realizedBy:
  - component.daemon
impacts:
  - concept.project
  - concept.plan
  - concept.unit
  - concept.cycle
  - concept.task
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:domain"]
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 식별자는 provenance/근거에만. -->

## 목적

이 도메인은 LLM 코딩 에이전트와 사람 운영자가 **하나의 같은 작업 상태를 공유**할 수 있도록, 개발 작업을 구조화된 계층으로 정리하는 책임을 진다. 즉흥적인 할 일 메모가 아니라, "어떤 프로젝트의 / 어떤 승인된 계획 아래 / 어떤 묶음에 속한 / 어떤 일을 / 언제 실행 중인 묶음(스프린트)에서 처리하는가"를 일관된 트리로 표현한다. 이렇게 정리된 상태가 있어야 세션이 끊겨도 다음 세션이 이어서 작업할 수 있고, 훅이 "지금 활성 작업이 없으니 변경을 막아라" 같은 규율을 강제할 수 있다.

## 경계와 핵심 개념

포함되는 개념은 다섯 가지 작업 엔티티다.

- **프로젝트(Project)**: 하나 이상의 작업 디렉터리에 묶이는 최상위 워크스페이스. 모든 계획과 사이클이 프로젝트에 속한다.
- **계획(Plan)**: 승인된 의도를 담는 컨테이너(로드맵). 초안→활성→완료 상태를 거치며, 활성 상태가 되어야 그 아래 태스크를 시작할 수 있다.
- **유닛(Unit)**: 계획 안에서 태스크를 묶는 순수 그룹핑 단위. 자체 상태도 승인도 없이 단지 묶음 역할만 한다.
- **태스크(Task)**: 실제로 직접 다루는 원자적 작업 항목. 반드시 한 유닛에 속하며, 자기 자신을 부모로 갖는 하위 태스크 트리를 형성할 수 있다.
- **사이클(Cycle)**: 타임박스된 실행 묶음(스프린트). 태스크를 가로질러 묶으며, 태스크는 사이클에 배정될 때 비로소 실행 일정 안으로 들어온다.

제외되는 것: 지식·위키·근거 검색은 별도 도메인(지식·로컬 RAG), QA 라운드 자동화는 별도 도메인(QA·디스커버 루프)이 담당한다. 이 도메인은 그 위에 올라타는 **순수한 작업 뼈대**만을 다룬다.

## 기능

- 프로젝트를 등록하고 작업 디렉터리에 바인딩한다.
- 계획을 만들어 승인(활성화)함으로써 태스크 시작 자격을 부여한다.
- 유닛으로 태스크를 묶고, 부모-자식 태스크 트리를 구성한다.
- 사이클을 만들어 활성화하고, 태스크를 사이클에 배정해 실행 일정에 편입한다.
- 태스크의 생애주기(할 일→진행 중→완료/취소, 외부 의존 시 차단)를 전이한다.

이 기능들은 모두 로컬 데몬(`component.daemon`)이 소유한 SQLite 위에서 결정론적으로 처리된다.

## 시스템 흐름

핵심 흐름은 위에서 아래로 내려가며 자격을 얻는 구조다. 먼저 프로젝트가 있어야 하고, 그 안에 계획을 만들어 **승인(활성화)** 해야 한다. 활성 계획이 없으면 태스크를 시작할 수 없다. 유닛은 계획 아래에서 태스크를 묶고, 태스크가 실제로 실행되려면 **활성 사이클에 배정**되어 있어야 한다. 한 프로젝트에 활성 계획은 원칙적으로 하나, 한 유닛에 활성 사이클은 하나만 둔다. 완료된 사이클은 다시 시작하지 않고 새로 만든다. 태스크를 완료(done)로 전이할 때는 근거(evidence)가 함께 있어야 한다 — 결과를 입증하지 못한 완료는 허용되지 않는다.

이 모든 전이는 데몬이 단일 진실 저장소(SQLite)에 기록하므로, 사람이 웹 대시보드에서 보든 에이전트가 CLI로 보든 같은 상태를 본다.

## 다른 도메인과의 관계

- **작업 관리 도메인**의 일부분으로서, 그 도메인이 다루는 워크플로의 데이터 골격을 제공한다.
- **지식·로컬 RAG 도메인**과 맞물려, 결정·노트·근거 같은 지식 항목이 이 계층의 계획·유닛·태스크에 첨부된다.
- **QA·디스커버 루프 도메인**은 이 계층 위에서 동작한다 — 검증 라운드를 시작하면 계획·유닛·사이클·태스크가 이 골격대로 자동 생성된다.

## 미확정 (OPEN)
- [ ] OPEN: 활성 계획 "원칙 1개, 전환기 ≤ 2개" 권고가 강제 불변식인지 advisory 경고인지 — 시작 경로 코드는 경고만 내보내므로 invariant.one-active-plan-per-project 의 강제 수준 확정 필요.
