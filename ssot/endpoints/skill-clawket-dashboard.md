---
id: endpoint.skill-clawket-dashboard
kind: Endpoint
title: 스킬 clawket-dashboard
definition: 작업 상태 확인·태스크 시작·완료·생성을 하나의 대시보드 표면에서 라이프사이클 게이트를 거쳐 라우팅하는 기본 스킬로, 슬래시 없이 작업 상태 관련 의도가 감지될 때 자동 진입한다.
realizedBy:
  - component.skills
  - component.cli
implementedIn:
  - clawket/.claude-plugin/plugin.json
  - clawket/skills/clawket-dashboard/SKILL.md
relatesTo:
  - to: concept.task
    type: mutates
    note: 태스크를 시작·완료·생성하며 시작·완료 전 게이트 검사를 강제한다
  - to: concept.cycle
    type: reads
    note: 시작 전 사이클이 active인지, 같은 사이클에 진행 중 태스크가 없는지 확인한다
  - to: concept.execution-envelope
    type: relates-to
    note: 새 태스크 생성 시 의도·프롬프트 템플릿·성공 기준 등 실행 봉투 필수 필드를 요구한다
governedBy:
  - invariant.start-needs-active-plan-cycle
  - invariant.one-active-cycle
  - invariant.evidence-on-done
  - invariant.envelope-required-fields
governs: []
dependsOn:
  - endpoint.cli-dashboard
  - endpoint.cli-task
consumedBy: []
servesPersona:
  - persona.llm-coding-agent
  - persona.human-operator
impacts:
  - concept.task
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

작업 보드를 다루는 가장 기본이 되는 진입점이다. 슬래시 명령으로 부르는 것이 아니라, 작업 상태를 보거나 태스크를 시작·완료·생성하려는 의도가 감지되면 자동으로 떠오른다. 현재 작업 디렉터리에 묶인 플랜·유닛·사이클·태스크 현황을 한눈에 보여주고, "시작/완료/생성" 같은 하위 동작을 받으면 그 동작이 라이프사이클 규칙을 어기지 않도록 게이트를 거쳐 안전하게 수행한다. 상태는 로컬에 영구 저장되어 세션이 바뀌어도 이어진다.

## 요청 / 응답

들어오는 것은 첫 단어로 구분되는 하위 동작이다. 인자가 없으면 현재 현황 대시보드와 빠른 참조를 보여준다.

- 상태 보기: 현재 작업 디렉터리의 진행 중인 플랜·유닛·사이클·태스크 요약을 돌려준다.
- 태스크 시작: 대상 태스크를 active 사이클에 배정하고 진행 중 상태로 올린 뒤 그 상세를 보여준다. 단, 시작 전 두 가지 사전 점검을 모두 통과해야 한다.
- 태스크 완료: 대상 태스크를 끝난 상태로 전환한다. 이때 근거(파일·라인 참조 또는 외부에서 확인 가능한 근거)가 반드시 동반되어야 하며 코멘트를 함께 남길 수 있다.
- 새 태스크 생성: 제목과 함께 소속 유닛·active 사이클을 지정하고, 의도·프롬프트 템플릿·성공 기준 같은 실행 계약 필수 항목을 채워 만든다. 우선순위도 지정할 수 있다.

## 권한 / 제약

사람 운영자와 코딩 에이전트 모두 이 스킬을 통해 작업 보드를 다룬다. 게이트로 강제되는 제약은 다음과 같다.

- 시작 전 첫째, 대상이 속한 사이클이 active여야 한다. planning 상태의 사이클은 일을 받을 수 없으므로 먼저 활성화하라고 안내하고 시작을 중단한다.
- 시작 전 둘째, 같은 사이클에서 이미 진행 중인 다른 태스크가 있으면 안 된다 — 한 사이클에는 한 번에 하나의 진행 중 태스크만 허용된다. 다른 태스크가 돌고 있으면 그것을 먼저 끝내거나 멈추라고 안내한다.
- 태스크는 시작 전 반드시 active 사이클에 배정되어야 한다. 사이클이 배정되지 않은 진행 중 태스크 상태에서는 변경 도구(편집·쓰기·셸)가 별도 가드에 의해 차단된다.
- 완료에는 근거가 필수다. 근거가 없으면 데몬이 완료를 거부한다.
- 새 태스크 생성에는 실행 봉투의 필수 항목(의도·프롬프트 템플릿·성공 기준)이 빠지면 데몬이 생성을 거부한다.
- 태스크가 완료되면 데몬이 유닛·사이클·플랜 완료를 자동으로 연쇄 처리하므로, 정상 경로에서는 사이클을 손으로 완료 처리할 필요가 없다.

## provenance

플러그인 매니페스트(`clawket/.claude-plugin/plugin.json`)가 이 스킬을 등록하고, 동작 절차는 스킬 명세(`clawket/skills/clawket-dashboard/SKILL.md`)에 정의된다. 실제 상태 조회·변경은 CLI 대시보드 명령(`endpoint.cli-dashboard`)과 태스크 명령(`endpoint.cli-task`)을 거쳐 데몬으로 전달된다. 제공 컴포넌트는 번들 스킬(`component.skills`)과 CLI(`component.cli`)다.

## 미확정 (OPEN)
- [ ] OPEN: 스킬 자동 진입(슬래시 없음)의 트리거 조건이 다른 작업 스킬과 충돌·중복되는지 owner 확인 필요.
