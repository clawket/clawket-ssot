---
id: capability.disable-project
kind: Capability
title: 프로젝트 훅 비활성화
definition: 사람이나 LLM 에이전트가 한 프로젝트의 Clawket 관리(가드레일)를 꺼서, 그 프로젝트의 작업 디렉터리에서는 훅이 아무 동작도 하지 않아 활성 태스크 없이도 자유롭게 변경할 수 있게 하되, 이미 쌓인 데이터는 그대로 보존하는 일이다.
purpose: 특정 프로젝트의 훅 강제를 일시적으로 꺼서 제약 없이 작업할 수 있게 하고, 다시 켤 때까지 기존 작업 데이터는 손상 없이 보존한다.
servesPersona:
  - persona.human-operator
  - persona.llm-coding-agent
realizedBy:
  - component.cli
  - component.daemon
relatesTo:
  - { to: endpoint.cli-project, type: calls, note: "project disable / enable 로 enabled 플래그를 0/1 로 토글한다" }
  - { to: concept.project, type: mutates, note: "프로젝트의 enabled 상태를 바꾼다 — 데이터는 그대로 보존" }
  - { to: domain.hook-enforcement, type: relates-to, note: "enabled=0 이면 PreToolUse / UserPromptSubmit / ExitPlanMode 훅이 무동작이 된다" }
impacts:
  - concept.project
  - domain.hook-enforcement
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:capability"]
---

## 사용자가 할 수 있는 일

Clawket의 가드레일은 등록된 프로젝트의 작업 디렉터리에서 "활성 태스크 없이는 변경 작업을 막는" 식으로 동작한다. 그런데 어떤 상황에서는 이 제약을 잠시 끄고 싶을 수 있다 — 빠른 실험, 관리 흐름 밖의 손질 등. 이때 프로젝트 단위로 관리를 꺼두면, 그 프로젝트에 묶인 디렉터리에서는 훅이 아무것도 하지 않아 활성 태스크 없이도 편집할 수 있다.

중요한 점은 끄는 것이 "삭제"가 아니라는 것이다. 비활성화는 가드레일만 끌 뿐, 이미 만든 플랜·태스크·지식·이력 같은 데이터는 그대로 남는다. 다시 켜면 같은 데이터 위에서 가드레일이 곧장 복귀한다. 이 토글은 웹 대시보드의 프로젝트 설정에 있는 enabled 스위치와 같은 것을 명령줄로 다루는 것이다.

## 행위

- 프로젝트의 관리를 끈다(disable) — enabled 플래그가 0이 되어 그 디렉터리에서 PreToolUse·UserPromptSubmit·ExitPlanMode 훅이 무동작이 된다.
- 프로젝트의 관리를 다시 켠다(enable) — enabled 플래그가 1로 돌아가 가드레일이 복귀한다.
- 끄고 켜는 동안 기존 작업 데이터(플랜·태스크·지식·이력)는 보존된다.

## 시스템 흐름

사용자나 에이전트가 비활성화를 요청하면 CLI가 데몬에 프로젝트의 enabled 값을 0으로 바꾸도록 요청하고(켤 때는 1), 데몬이 SQLite의 프로젝트 레코드를 갱신한다. 이후 세션이 시작될 때마다 훅 어댑터는 현재 디렉터리를 어떤 프로젝트에 속하는지 되짚고, 그 프로젝트의 enabled 값을 보고 가드레일을 적용할지 말지 결정한다 — enabled가 0이면 변경 차단·컨텍스트 주입 등 훅 동작 전체가 건너뛰어진다.

## 어디에 구현되어 있나

disable·enable 명령은 CLI의 project 서브커맨드에 있으며, enabled 플래그(0/1)를 데몬의 프로젝트 갱신 경로로 보내 SQLite에 반영한다. 그 값을 읽어 가드레일 적용 여부를 가르는 판단은 훅 어댑터(훅 강제 영역)가 프로젝트 resolve 결과의 enabled 필드를 보고 수행한다.

## 미확정 (OPEN)
- [ ] OPEN: enabled=0 상태에서도 데몬이 파일 변경 활동을 감사 로그에 계속 기록하는지(가드레일만 끄는지, 기록까지 멈추는지)는 데몬·훅 어댑터에서 교차 확인 필요.
