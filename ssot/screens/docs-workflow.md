---
id: screen.docs-workflow
kind: Screen
title: 워크플로우 문서 화면(Workflow)
definition: /docs/workflow 경로의 영문 문서 화면. 다섯 엔티티(Project·Plan·Unit·Cycle·Task)와 그 라이프사이클, decompose/contract/execute 3단계 루프, 7개 라이프사이클 훅이 하는 일을 표로 정리하고, PreToolUse가 왜 변경을 막는지와 Plan Mode 출력을 Clawket 플랜으로 대체하는 통합을 설명한다.
purpose: Clawket의 강제된 작업 구조를 이해하려는 사람이 다섯 엔티티의 관계와 상태 전이, 작업을 시작하려면 왜 먼저 분해·계약해야 하는지를 한 페이지에서 파악하도록 한다.
servesPersona: [persona.human-operator, persona.llm-coding-agent]
realizedBy: [component.landing-site]
implementedIn: [landing/src/pages/docs/Workflow.tsx]
consumesApi: []
relatesTo:
  - { to: concept.project, type: relates-to, note: "다섯 엔티티 중 Project 설명" }
  - { to: concept.plan, type: relates-to, note: "Plan draft→active→completed 라이프사이클 설명" }
  - { to: concept.unit, type: relates-to, note: "Unit이 상태·승인 없는 순수 그룹핑임을 설명" }
  - { to: concept.cycle, type: relates-to, note: "Cycle planning→active→completed, 유닛당 1 active, 재시작 불가 설명" }
  - { to: concept.task, type: relates-to, note: "Task 라이프사이클과 tier 라벨 설명" }
  - { to: concept.execution-envelope, type: relates-to, note: "contract 단계에서 intent·prompt_template·success_criteria 봉투 서명 설명" }
  - { to: concept.tier, type: relates-to, note: "Task의 low/med/high tier가 모델 라우팅에 쓰임을 설명" }
  - { to: integration.claude-code-hooks, type: relates-to, note: "7개 라이프사이클 훅 표" }
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:screen"]
---

## 화면 목적

Clawket의 핵심 개념인 "강제된 워크플로우"를 설명하는 영문 문서다. 제품의 의견(opinion)은 "모든 기여는 분해(decompose) → 계약(contract) → 실행(execute) 순서를 거친다"이며, 순서를 어기면 PreToolUse 훅이 파일을 건드리기 전에 작업을 거부한다는 점을 화면 전체로 풀어 설명한다.

## UI 요소 / 입력 필드

- 머리말: decompose/contract/execute 3단계를 순서대로 지키는 것이 모든 규칙의 출발점이라는 선언.
- "다섯 엔티티": Project(등록된 작업 디렉터리), Plan(상위 의도, draft→active→completed, 프로젝트당 최대 1 active, active 전엔 태스크 시작 불가), Unit(플랜 내 순수 그룹핑, 상태·승인 없음), Cycle(스프린트, planning→active→completed, --unit 필요, 유닛당 1 active, 완료 후 재시작 불가), Task(직접 다루는 유일한 엔티티, todo→in_progress→done|cancelled + blocked, low/med/high tier로 모델 라우팅).
- "decompose/contract/execute": 세 단계를 번호로 설명 — 1) 한 호흡에 끝낼 수 있는 태스크가 될 때까지 Plan→Unit→Task로 분해, 2) 각 태스크를 실행 봉투(최소 intent·prompt_template·success_criteria)로 서명(자유형 프롬프트는 계약이 아님), 3) 그제서야 코드 실행 — PreToolUse가 활성 태스크 없으면 막고, 막힘에 대한 옳은 대응은 우회가 아니라 계약을 끝내는 것.
- "라이프사이클 훅" 표: 7개 핸들러(SessionStart, UserPromptSubmit, PreToolUse, PostToolUse(Edit/Write), PostToolUse(ExitPlanMode), SubagentStart, SubagentStop) 각각이 하는 일. 각 게이트는 그 훅이 유일한 강제 지점이며 데몬이 중복 검사하지 않는다는 주석. 태스크가 done/cancelled가 되면 자식이 모두 종료된 경우 데몬이 Unit/Plan/Cycle 완료를 자동 cascade한다는 설명.
- "PreToolUse가 막는 이유": 게이트가 없으면 워크플로우가 opt-in이 되어 몇 시간 만에 자유형 프롬프트로 회귀하므로, 막힘 자체가 기능이라는 논지. 플러그인 자체 개발용 탈출 밸브(CLAWKET_BYPASS_HOOKS=1)는 일상 작업용이 아니며 PreToolUse엔 다른 우회가 없다는 설명.
- "Plan Mode 통합": Claude Code의 Plan Mode가 만드는 ~/.claude/plans/ 파일은 휘발적·스키마 없음이라 Clawket이 ExitPlanMode 훅으로 plan create를 유도해 안정 ID·버전 이력·하위 Unit/Cycle/Task 연결을 갖춘 DB 플랜으로 대체한다는 설명.

입력 필드는 없다.

## 표시 데이터 / 호출 API

화면에 하드코딩된 영문 설명과 1개의 표다. 데몬·DB를 호출하지 않으므로 consumesApi 가 없다.

## 상태 / 엣지케이스

- 정적 문서라 로딩/에러 상태가 없다.
- 진입 시 자기 SEO 메타데이터를 직접 설정한다.

## 미확정 (OPEN)
- 없음
