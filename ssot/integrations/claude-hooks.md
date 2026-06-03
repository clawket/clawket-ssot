---
id: integration.claude-hooks
kind: Integration
title: Claude Code 훅 어댑터 연동
purpose: "에이전트가 활성 태스크 없이 코드를 바꾸거나 파괴적 명령을 돌리지 못하게 막고, 세션 시작 시 작업 맥락을 자동으로 떠먹여 주며, 무엇을 했는지 자동으로 기록하기 위해 — Claude Code 훅 이벤트에 Clawket 동작을 끼워 넣는다."
definition: "Claude Code의 훅 이벤트(세션 시작·프롬프트 제출·도구 실행 전/후·플랜 모드 종료·서브에이전트 시작/종료)마다 도는 Node .cjs 스크립트 모음. 컨텍스트 주입, 변경 차단 가드레일, run/태스크 자동 기록을 수행한다."
integratesWith:
  - component.claude-adapter
  - component.cli
  - component.daemon
consumedBy:
  - persona.llm-coding-agent
impacts:
  - endpoint.hook-session-start
  - endpoint.hook-user-prompt-submit
  - endpoint.hook-pre-tool-use
  - endpoint.hook-post-tool-use
  - endpoint.hook-plan-sync
  - endpoint.hook-subagent-start
  - endpoint.hook-subagent-stop
relatesTo:
  - to: domain.hook-enforcement
    type: belongs-to
    note: "이 어댑터들이 곧 훅 강제(가드레일) 도메인의 실행 표면이다."
  - to: integration.clawketd
    type: depends-on
    note: "훅은 데몬에 붙어 상태를 읽고 활동을 기록한다 — 데몬 자동기동도 훅 글루가 보장한다."
governedBy:
  - invariant.daemon-liveness-socket-probe
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:integration
---

## 무엇과 연동하나

상대는 Claude Code의 훅 시스템이다. Claude Code는 작업 흐름의 여러 길목에서 정해진 이벤트를 발생시키고, Clawket은 그 이벤트마다 돌 Node .cjs 스크립트를 등록해 둔다. 다루는 이벤트는 일곱 가지다: 세션 시작(SessionStart), 사용자 프롬프트 제출(UserPromptSubmit), 도구 실행 전(PreToolUse), 도구 실행 후(PostToolUse), 플랜 모드 종료(ExitPlanMode), 서브에이전트 시작(SubagentStart), 서브에이전트 종료(SubagentStop).

이 어댑터들이 하는 일은 세 갈래다. (1) 컨텍스트 주입 — 세션이 시작되면 데몬을 보장하고 활성 프로젝트의 작업 대시보드와 규칙을 에이전트에게 떠먹이고, 프롬프트 제출 시 활성 태스크 맥락을 끼워 준다. (2) 가드레일 — 활성 태스크가 없으면 코드 변경 도구를 막고, 파괴적 명령(예: 되돌릴 수 없는 git 명령)을 차단하며, 플랜 모드 종료를 가로채 Clawket 플랜으로 등록하도록 유도한다. (3) 자동 기록 — 파일 변경을 활성 태스크에 기록하고, 태스크별 실행(run)을 자동 생성·요약하며, 서브에이전트 종료 시 결과를 정리한다.

## 구현 위치 (provenance)

훅 핸들러 진입점들은 플러그인의 `adapters/claude/` 아래 .cjs 파일로 배포되며(세션 시작·프롬프트 제출·도구 전/후·플랜 동기화·서브에이전트 시작/종료의 일곱 핸들러), 공통 로직은 `adapters/shared/claude-hooks.cjs`의 공유 핸들러에 모여 있다. 이 글루가 데몬 자동기동까지 포함하므로, `cli/src/daemon_autostart.rs`의 Rust 자동기동은 의도적으로 같은 동작을 거울처럼 따라 만들어졌다(코드 주석이 이 대응을 명시).

doctor 명령(`cli/src/doctor.rs`)이 이 연동의 건강을 점검한다. Hooks 섹션은 일곱 개 핸들러 파일이 표준 플러그인 설치 경로 또는 레포 내 개발 경로에 존재하는지 확인하고, 이어서 각 Claude Code 훅 이벤트별로 `hooks.json`에 선언된 핸들러 경로와 상태 로그에서 읽은 마지막 발화 시각을 한 줄씩 보여 준다. 로그가 없거나 한 번도 발화하지 않은 이벤트는 시각을 "—"로 표시한다.

이 연동을 소비하는 주체는 LLM 코딩 에이전트(`persona.llm-coding-agent`)이며, 내부적으로 Claude 어댑터(`component.claude-adapter`)가 구현하고 CLI·데몬과 함께 동작한다.

## 불변식

- 훅의 컨텍스트 주입과 기록은 데몬이 떠 있어야 성립한다 — 어댑터는 데몬을 보장하고 생존을 유닉스 소켓 접속으로 판정한다(`invariant.daemon-liveness-socket-probe`).
- 가드레일의 핵심 규칙: 활성 태스크가 없으면 코드 변경 도구는 차단된다. 즉 모든 변경은 추적되는 작업 단위에 묶여야 한다.
- 일곱 핸들러 파일이 어댑터 경로에 모두 존재해야 한다 — doctor가 누락을 오류로 잡는다.

## 영향 범위

이 연동은 일곱 개 훅 엔드포인트 전부(세션 시작·프롬프트 제출·도구 전/후·플랜 동기화·서브에이전트 시작/종료)에 직접 닿는다. 어댑터가 빠지거나 동작이 바뀌면 컨텍스트 주입·변경 차단·자동 기록이 한꺼번에 영향을 받으며, 이는 곧 훅 강제 도메인 전체의 신뢰도와 직결된다.

## 미확정 (OPEN)
- [ ] OPEN: PreToolUse가 차단하는 변경 도구 목록과 파괴적 명령 카탈로그의 정확한 범위 확정(별도 가드 연동 노드와의 경계)
- [ ] OPEN: SubagentStop의 자동 태스크 완료 조건(evidence 충족 판정)을 불변식으로 확정할지 여부
