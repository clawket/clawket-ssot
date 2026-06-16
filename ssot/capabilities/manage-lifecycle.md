---
id: capability.manage-lifecycle
kind: Capability
title: 워크플로우 라이프사이클 관리(CLI)
purpose: "프로젝트·플랜·유닛·사이클·태스크를 각자의 상태 단계에 따라 만들고 승인·활성화·완료하여, 구조화된 작업 계층을 처음부터 끝까지 한 곳에서 진행한다."
servesPersona:
  - persona.llm-coding-agent
  - persona.human-operator
realizedBy:
  - component.cli
  - component.daemon
implementedIn:
  - cli/src/main.rs
relatesTo:
  - { to: domain.work-management, type: relates-to, note: "이 기능이 다루는 작업 계층 도메인 전체" }
  - { to: concept.project, type: relates-to, note: "최상위 워크스페이스 — 모든 라이프사이클의 출발점" }
  - { to: concept.plan, type: relates-to, note: "승인(draft→active)으로 태스크 시작 자격을 여는 단계" }
  - { to: concept.unit, type: relates-to, note: "상태 없는 그룹핑 단계" }
  - { to: concept.cycle, type: relates-to, note: "활성화(planning→active)로 작업을 담는 타임박스" }
  - { to: concept.task, type: relates-to, note: "직접 다루는 원자 작업 — done에는 evidence 필수" }
impacts:
  - concept.plan
  - concept.cycle
  - concept.task
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:capability
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로. -->

## 사용자가 할 수 있는 일

사람 운영자와 LLM 코딩 에이전트는 같은 명령으로 작업 계층 전체를 진행한다. 큰 그림은 "프로젝트를 만들고 → 그 안에 의도(플랜)를 세워 승인하고 → 작업을 묶을 유닛을 두고 → 타임박스(사이클)를 활성화한 뒤 → 사이클 안에 실제 태스크를 넣어 시작·완료한다"는 한 줄의 흐름이다. 이 기능의 가치는, 흩어진 다섯 종류의 작업 단위를 각자의 정해진 단계(상태 기계)에 따라 안전하게 전진시키는 것이다. 단계를 건너뛰면 시스템이 막는다 — 플랜이 승인되지 않으면 태스크를 시작할 수 없고, 사이클이 활성이 아니면 그 안의 태스크를 시작할 수 없다.

## 행위

- **만들기**: 각 단위(프로젝트/플랜/유닛/사이클/태스크)를 생성한다. 사이클은 반드시 하나의 유닛에 속하고, 태스크는 사이클 지정이 필요하다.
- **승인·활성화**: 플랜은 초안(draft)에서 승인하면 활성(active)이 되어 태스크 시작 자격이 생긴다. 사이클은 계획(planning)에서 활성화하면 작업을 받을 수 있다. 유닛은 상태도 승인도 없는 순수 그룹핑이다.
- **진행·완료**: 태스크는 할 일(todo) → 진행 중(in_progress) → 완료(done)/취소(cancelled)로 옮겨지며, 외부 의존이 있으면 차단(blocked)된다. 완료로 넘어가려면 근거(evidence)가 반드시 있어야 한다.
- **자동 마감(cascade)**: 사이클·플랜은 그 안 태스크가 모두 종료 상태가 되면 자동으로 완료된다. 유닛은 상태가 없어 이 자동 마감에 끼지 않는다.
- **조회·수정·삭제**: 각 단위를 보고, 속성을 고치고, 지우거나 목록으로 본다.

## 시스템 흐름

명령은 CLI(`endpoint.cli-project`·`endpoint.cli-plan`·`endpoint.cli-unit`·`endpoint.cli-cycle`·`endpoint.cli-task`)로 들어가고, CLI는 이를 로컬 데몬의 HTTP API로 보낸다. 데몬이 상태 기계와 불변식을 강제한다 — 승인되지 않은 플랜의 태스크 시작 거부, 근거 없는 완료 거부, 종료 시 상위 자동 마감 등이 모두 데몬 쪽 규칙이다. 그래서 단순한 "필드 수정"이 아니라 정해진 전이만 통과하는 흐름이다.

## 어디에 구현되어 있나

CLI 명령 정의와 라우팅은 `cli/src/main.rs`에 있다. 다섯 단위 각각의 하위 명령(create/view/list/update/delete와 plan approve·cycle activate·task complete 등)이 여기 선언되며, 실제 상태 강제는 데몬으로 위임된다.

## 미확정 (OPEN)
- [ ] OPEN: 데몬 측 상태 전이 검증 코드 경로(daemon/src) 정밀 확인 필요
