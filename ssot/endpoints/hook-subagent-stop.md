---
id: endpoint.hook-subagent-stop
kind: Endpoint
title: 훅 SubagentStop
definition: 서브에이전트가 종료될 때 발화하는 Claude Code 훅으로, 그 에이전트에 묶인 진행 중 작업마다 결과 요약을 붙이고 근거가 충분하면 자동 완료시키되 근거가 부족하면 완료를 막고 진행 중으로 남긴다.
realizedBy:
  - component.claude-adapter
implementedIn:
  - clawket/hooks/hooks.json
  - clawket/adapters/claude/subagent-stop.cjs
  - clawket/adapters/shared/claude-hooks.cjs
relatesTo:
  - to: concept.task
    type: mutates
    note: 묶인 진행 중 작업에 결과 요약을 추가하고 근거가 충족되면 done으로 전환한다
  - to: concept.task-evidence
    type: mutates
    note: 근거가 비어 있으면 마지막 응답에서 자동 채워넣고, 그래도 부족하면 완료를 막는다
  - to: concept.tier
    type: mutates
    note: 사용된 모델 등급 정보가 있으면 작업에 기록한다
governedBy:
  - invariant.evidence-on-done
  - invariant.terminal-auto-cascade
  - invariant.sync-purity
governs: []
dependsOn:
  - endpoint.hook-subagent-start
impacts:
  - concept.task
consumedBy:
  - persona.qa-reasoner-subagent
servesPersona:
  - persona.llm-coding-agent
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

서브에이전트가 일을 마치고 사라지는 순간에 자동으로 한 번 실행되는 마무리 장치다. 시작 훅이 작업자를 작업에 묶었다면, 이 종료 훅은 그 묶음을 깔끔하게 닫는 역할을 한다. 작업자가 남긴 결과를 작업 기록에 보존하고, 완료 자격을 갖췄으면 작업을 끝난 것으로 표시하며, 자격이 부족하면 함부로 닫지 않고 진행 중으로 남겨 사람이 보완하도록 신호를 보낸다.

## 요청 / 응답

받는 정보는 종료한 서브에이전트의 식별자와 그 에이전트의 마지막 응답 메시지다. 식별자가 없으면 아무 일도 하지 않는다. 데몬이 떠 있지 않으면 정리 작업을 모두 건너뛰고 그 사유를 눈에 보이게 남긴다(그대로 두면 진행 중 작업이 닫히지 않고 새는 것을 막기 위함).

별도의 응답을 호출자에게 돌려주기보다, 상태 저장소에 부수효과를 남기는 것이 본질이다. 해당 에이전트에 묶인 진행 중 작업을 전부(하나만이 아니라 여러 개 모두) 찾아 각각에 대해 다음을 수행한다. 마지막 응답을 "서브에이전트 결과"라는 정해진 문구로 작업 본문에 덧붙이고(너무 길면 일정 크기로 잘라 보존), 작업에 근거가 비어 있으면 그 응답에서 일부를 떼어 근거로 자동 채워넣으며, 사용된 모델 등급 정보가 있으면 함께 기록한다.

## 권한 / 제약

호출 주체는 사람이 아니라 Claude Code 런타임이다 — 서브에이전트 종료 이벤트가 곧 호출이다. 자동 완료를 좌우하는 제약은 다음과 같다.

- 작업을 끝난 것으로 표시하기 직전에 근거가 충분한지 마지막으로 검사한다. 근거가 비었거나 너무 짧으면(자리채우기 수준이면) 그 작업만 완료를 막고 진행 중으로 남긴 뒤, 어떻게 근거를 채우라는 안내를 남기고 나머지 작업 처리는 계속 이어간다. 한 작업의 실패가 다른 작업의 마무리를 막지 않는다.
- 완료가 허용되면 "자동 완료: 에이전트 종료"라는 코멘트와 함께 작업을 끝난 상태로 전환한다. 이 종결은 데몬의 자동 연쇄(유닛·사이클·플랜 완료)로 이어질 수 있다.
- 처리 마지막에는 대량 동기화 맥락을 정리해, 같은 프로세스에서 곧이어 새 작업자를 띄울 때 동기화-추론 혼재로 오분류되지 않게 한다.

## provenance

훅 매니페스트(`clawket/hooks/hooks.json`)의 SubagentStop 항목이 얇은 진입점(`clawket/adapters/claude/subagent-stop.cjs`)을 호출하고, 실제 로직은 공유 핸들러(`clawket/adapters/shared/claude-hooks.cjs`의 `runSubagentStop`)에 있다. 완료 직전 근거 검사는 같은 파일의 `checkX8Evidence`, 동기화 맥락 정리는 `clearSyncContext`가 담당한다. 제공 컴포넌트는 Claude 어댑터(`component.claude-adapter`)이며, 작업 본문 추가·근거 기록·상태 전환은 CLI 작업 명령(`endpoint.cli-task`)을 거친다.

## 미확정 (OPEN)
- [ ] OPEN: 근거 자동 채움(마지막 응답 일부 절단)이 감사 추적 품질에 미치는 영향과 허용 길이 정책의 의도를 owner 확인 필요.
