---
id: endpoint.hook-user-prompt-submit
kind: Endpoint
title: 훅 UserPromptSubmit
definition: "사용자가 프롬프트를 보낼 때마다 호출되는 훅으로, 진행 중인 태스크 목록을 그 턴의 컨텍스트로 주입하고, 진행 중 태스크가 하나도 없으면 변경 작업을 시작하기 전에 태스크부터 지정하라고 경고한다."
realizedBy:
  - component.claude-adapter
implementedIn:
  - clawket/hooks/hooks.json
  - clawket/adapters/claude/user-prompt-submit.cjs
  - clawket/adapters/shared/claude-hooks.cjs
relatesTo:
  - to: concept.task
    type: reads
    note: 진행 중(in_progress) 태스크를 읽어 매 턴 활성 작업 목록으로 주입한다
  - to: concept.active-project
    type: reads
    note: 현재 디렉터리의 프로젝트를 해석해 등록·활성 여부에 따라 동작을 가른다
  - to: endpoint.cli-dashboard
    type: relates-to
    note: 활성 작업 요약을 대시보드 조회로 얻어 진행 중 태스크를 추린다
governedBy:
  - invariant.active-task-required
impacts: []
consumedBy:
  - integration.claude-code-hooks
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

사용자가 메시지를 입력해 보낼 때마다 그 직전에 자동 실행되는 훅이다. 목적은 매 대화 턴에서 에이전트가 "지금 활성화된 작업이 무엇인지"를 잊지 않도록 현재 진행 중 태스크 목록을 그 턴의 컨텍스트에 다시 심어 주는 것이다. 긴 대화에서 앞선 작업 맥락이 흐려지는 것을 막는, 가벼운 컨텍스트 리마인더다.

동시에 가드레일의 첫 단계 역할도 한다. 진행 중 태스크가 하나도 없으면 이를 알리는 경고 메시지를 주입해, 사람·에이전트가 변경 작업을 시도하기 전에 먼저 태스크를 시작하도록 유도한다(실제 차단은 도구 실행 직전 단계에서 일어난다).

## 요청 / 응답

입력은 이번에 제출된 프롬프트의 맥락 정보(현재 작업 디렉터리)뿐이다. 응답은 그 턴에만 끼워 넣는 추가 컨텍스트 한 덩어리다.

세 갈래로 갈린다. 프로젝트가 비활성화돼 있거나 현재 디렉터리가 등록된 프로젝트가 아니면 아무것도 주입하지 않고 통과시킨다. 등록된 프로젝트이고 진행 중 태스크가 있으면 그 목록(식별자·제목·부가정보)을 "활성 작업" 컨텍스트로 주입한다. 등록된 프로젝트인데 진행 중 태스크가 없으면 "활성 태스크 없음" 경고를 주입한다.

## 권한 / 제약

매 프롬프트마다 자동 실행되며 응답 시간 예산이 매우 짧다(짧은 타임아웃 내에 끝나야 한다). 그래서 무거운 계산 없이 활성 작업 요약 조회 정도만 한다. 비활성 프로젝트·미등록 디렉터리에서는 의도적으로 무동작이며, 이 훅 자체는 작업을 막지 않는다 — 경고만 주입하고 통과시킨다. 활성 태스크 없이 변경을 막는 강제는 도구 실행 직전(PreToolUse) 단계의 책임이다.

## provenance

`clawket/hooks/hooks.json`이 UserPromptSubmit 이벤트를 어댑터에 연결하고, `clawket/adapters/claude/user-prompt-submit.cjs`가 공유 핸들러의 루틴(`clawket/adapters/shared/claude-hooks.cjs`의 `runUserPromptSubmit`)에 위임한다. 그 루틴에서 비활성·미등록 통과, 진행 중 태스크 목록 주입, 활성 태스크 없음 경고의 3분기를 확인했다.

## 미확정 (OPEN)
- [ ] OPEN: 활성 작업 목록에 함께 실리는 부가정보(meta)에 어떤 항목까지 담기는지(사이클·플랜·시나리오 등) 정확한 구성은 추가 정독 필요.
