---
id: invariant.active-task-required
kind: Invariant
title: 변경 작업에는 활성 태스크가 반드시 있어야 한다
definition: 등록·활성화된 프로젝트 안에서 코드를 바꾸는 모든 행위는, 현재 진행 중(in_progress)인 태스크가 지정되어 있을 때에만 허용된다. 활성 태스크가 없으면 변경 도구 호출이 시작 자체부터 막힌다.
governs:
  - concept.task
  - domain.hook-enforcement
  - endpoint.hook-pre-tool-use
  - persona.llm-coding-agent
implementedIn:
  - clawket/hooks/hooks.json
  - clawket/adapters/claude/pre-tool-use.cjs
  - clawket/adapters/shared/claude-hooks.cjs
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

<!-- 작성 고도: 비개발자도 읽는 자연어. 식별자는 provenance/근거에만. -->

## 제약

Clawket이 관리하는 프로젝트 안에서는, "지금 무슨 일을 하고 있는가"를 가리키는 진행 중 태스크가 반드시 먼저 지정되어 있어야만 코드를 바꿀 수 있다. 파일을 고치거나 새로 쓰는 일, 임의의 쉘 명령 실행, 서브에이전트 스폰, 팀 생성, 에이전트 간 메시지 전송 같은 "변경을 일으키는 행위" 전부가 이 규칙의 대상이다. 활성 태스크가 없으면 이런 행위는 시작 단계에서 차단되며, 차단을 풀어주는 우회 플래그는 제공하지 않는다.

이것은 Clawket의 핵심 철학을 강제로 만드는 장치다. "작업은 곧 계약(태스크)이며, 계약 없이 일하지 않는다"는 원칙을 권고가 아니라 시스템 차원의 불변식으로 박아 둔 것이다. 차단을 만났을 때 올바른 대응은 차단을 피하는 것이 아니라, 먼저 무엇을 할지 태스크로 선언하고 그것을 진행 상태로 만든 다음 일을 시작하는 것이다.

## 깨지면 무슨 일이 일어나나

만약 이 제약이 없거나 우회된다면, 에이전트가 아무런 추적 단위 없이 코드베이스를 변경할 수 있게 된다. 그 변경은 어떤 의도(태스크)에도 귀속되지 않으므로 누가·왜·무엇을 위해 고쳤는지 사후에 복원할 수 없고, 실행 기록(run)·근거(evidence)·감사 추적과도 연결되지 않는다. 결과적으로 "상태로 추적되는 작업"이라는 Clawket의 존재 이유 자체가 무너지고, 사람 운영자는 대시보드에서 진행 상황을 신뢰할 수 없게 된다.

## 코드에서 어떻게 강제되나

Claude Code의 도구 사용 직전 이벤트(PreToolUse)에 훅이 걸려 있고, 그 매처가 변경을 일으키는 도구군(에이전트 스폰·팀 생성·메시지 전송·파일 편집·파일 쓰기·쉘 실행)을 대상으로 한다. 훅 핸들러는 현재 cwd가 속한 프로젝트에 진행 중 태스크가 있는지 확인하고, 없으면 도구 실행을 거부한다. 비활성화된(관리 끔) 프로젝트나 미등록 경로에서는 이 검사가 무동작이 되어 통과하지만, 활성 프로젝트 안에서는 예외가 없다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(이 불변식을 도입한 결정 근거) 연결 필요
