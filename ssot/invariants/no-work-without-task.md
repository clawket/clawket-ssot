---
id: invariant.no-work-without-task
kind: Invariant
title: 활성 태스크 없이는 변경 작업 불가
definition: "Clawket가 관리하는 프로젝트에서 진행 중인 태스크가 하나도 없으면 코드를 바꾸거나 명령을 실행하는 모든 도구 호출이 사전에 차단된다."
governs:
  - domain.hook-enforcement
  - concept.task
  - persona.llm-coding-agent
implementedIn:
  - clawket/adapters/shared/claude-hooks.cjs
  - clawket/prompts/shared/rules.md
decidedBy:
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:invariant
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지. -->

## 제약

Clawket가 켜져 있는 프로젝트에서 일을 하려면, 먼저 "지금 이 작업을 한다"는 의미의 진행 중(in_progress) 태스크가 반드시 하나는 있어야 한다. 이 조건이 충족되지 않으면 파일을 고치거나(편집/쓰기), 셸 명령을 돌리거나, 서브에이전트·팀을 띄우거나, 에이전트 간 메시지를 보내는 등 "상태를 바꾸는" 모든 행위가 시작조차 되지 못하고 사전에 거부된다.

핵심 의도는 단순하다. 모든 작업이 추적 가능한 단위(태스크)에 묶여 기록으로 남도록 강제하는 것이다. "왜 이 변경을 했는지" 가 어디에도 연결되지 않은 채로 코드가 바뀌는 상황을 막는다.

한 가지 예외가 있다. 프로젝트가 Clawket 관리에서 비활성화(disable)된 경우에는 이 제약이 적용되지 않고 작업이 자유롭게 통과한다. 다만 비활성화 상태여도 사용자 데이터를 직접 망가뜨리는 파괴적 명령에 대한 보호는 그대로 유지된다.

## 깨지면 무슨 일이 일어나나

이 제약이 없으면 에이전트가 아무 맥락 없이 코드를 바꿀 수 있고, 그 변경이 어떤 계획·목표·검증 단위에 속하는지 사후에 복원할 수 없게 된다. 작업 이력이 비어 있는 변경이 쌓이면 Clawket의 존재 이유 — 모든 작업을 구조화된 상태 계층에 묶어 추적한다는 목적 — 자체가 무너진다. 그래서 이 게이트는 "경고"가 아니라 도구 호출을 실제로 막는 하드 블록이다.

## 코드에서 어떻게 강제되나

Claude Code의 PreToolUse 훅이 도구가 실제로 실행되기 직전에 호출되고, 공유 핸들러가 현재 작업 디렉터리에 묶인 프로젝트를 찾아 "진행 중 태스크가 있는지"를 데몬에 조회한다. 진행 중 태스크가 0개면 핸들러가 도구 실행을 거부(deny)하고, 사용자에게 먼저 태스크를 시작하라는 안내를 돌려준다. 변경 대상으로 보는 도구군은 파일 편집·쓰기, 셸 실행, 그리고 서브에이전트·팀 생성·메시지 전송 계열이다.

이 게이트는 plan·cycle 관련 다른 게이트보다 먼저 평가되어 "태스크부터 만들라"는 가장 실행 가능한 안내가 우선 노출되도록 되어 있다. 프로젝트가 비활성화되었거나 작업 디렉터리가 어떤 프로젝트에도 묶이지 않은 경우에는 통과시킨다(`clawket/adapters/shared/claude-hooks.cjs`의 PreToolUse 처리). 규칙 자체는 프롬프트 규약(`clawket/prompts/shared/rules.md`의 "No implementation work without an active task")으로도 명문화되어 에이전트에게 함께 전달된다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요 — 이 가드레일을 도입한 결정 노드 미확인
