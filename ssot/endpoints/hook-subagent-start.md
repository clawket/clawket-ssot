---
id: endpoint.hook-subagent-start
kind: Endpoint
title: 훅 SubagentStart
definition: 서브에이전트가 스폰될 때 발화하는 Claude Code 훅으로, 그 에이전트를 부모 태스크 아래 자식 태스크로 묶고 배치 크기·시나리오·동기화 순수성 가드를 통과시킨 뒤 부모 작업 맥락을 에이전트에게 주입한다.
realizedBy:
  - component.claude-adapter
implementedIn:
  - clawket/hooks/hooks.json
  - clawket/adapters/claude/subagent-start.cjs
  - clawket/adapters/shared/claude-hooks.cjs
relatesTo:
  - to: concept.task
    type: mutates
    note: 부모 태스크 아래 type=subagent 자식 태스크를 생성·시작하고 에이전트 식별자를 묶는다
  - to: concept.scenario
    type: reads
    note: 에이전트 프롬프트에서 시나리오 식별자를 세어 배치 한도 검사에 사용한다
  - to: persona.qa-reasoner-subagent
    type: relates-to
    note: 검증 라운드에서 배치당 1개로 스폰되는 추론 서브에이전트가 이 훅을 통과해 바인딩된다
governedBy:
  - invariant.batch-size-30
  - invariant.sync-purity
  - invariant.scenario-id-one-to-one
governs: []
dependsOn:
  - endpoint.hook-pre-tool-use
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

서브에이전트(병렬 작업자)가 새로 떠오르는 순간에 자동으로 한 번 실행되는 가드 겸 연결 장치다. 사람이 직접 부르는 명령이 아니라, 코딩 에이전트가 작업을 잘게 나눠 여러 하위 작업자에게 맡길 때 시스템이 알아서 발화시킨다. 목적은 두 가지다. 첫째, 새로 뜬 작업자를 "지금 진행 중인 부모 작업의 자식"으로 명확히 등록해 누가 무엇을 하고 있는지 추적 가능하게 만든다. 둘째, 그 작업자가 다루려는 작업량과 맥락이 안전 기준을 넘지 않는지 검사해, 넘으면 출발 자체를 막는다.

## 요청 / 응답

받는 정보는 새로 뜬 서브에이전트의 식별자, 에이전트 유형, 그리고 그 에이전트에게 건넨 작업 지시문(프롬프트)이다. 식별자가 없으면 아무 일도 하지 않고 조용히 통과시킨다.

돌려주는 것은 두 갈래다. 가드를 통과하지 못하면 "이 작업자의 출발을 거부한다"는 결정과 사람이 읽을 수 있는 거부 사유(예: 한 번에 다루려는 시나리오가 너무 많다, 동기화 작업 중에는 새 추론 작업자를 띄울 수 없다)를 돌려준다. 통과하면 부모 작업의 식별자·제목·작업 범위 요약을 정리한 맥락 문장을 서브에이전트의 시스템 프롬프트로 주입해, 작업자가 자기가 속한 큰 그림을 알고 시작하게 한다.

내부적으로는 부모 작업 아래에 "subagent" 유형의 자식 작업을 새로 만들어 곧바로 진행 중 상태로 두고, 에이전트 식별자를 그 자식 작업에 묶는다. 자식 작업을 만들지 못하면 부모 작업에 직접 식별자를 묶는 방식으로 후퇴한다. 어느 경우든 "서브에이전트가 시작됨"이라는 사람이 읽을 수 있는 흔적을 작업 본문에 남겨, 나중에 식별자 컬럼이 덮어써지더라도 시작 시점의 기록이 사라지지 않게 한다.

## 권한 / 제약

호출 주체는 사람이 아니라 Claude Code 런타임이다 — 서브에이전트 스폰 이벤트가 곧 호출이다. 통과 여부를 좌우하는 제약은 다음과 같다.

- 한 작업자가 한 번에 다루는 고유 시나리오 수가 정해진 상한을 넘으면 출발을 막고, 어디서 끊어 나눠야 하는지 구간 단위 안내까지 곁들인다. 이 검사는 별도 설정으로 강도를 낮추거나 끌 수 있고, 데몬이 떠 있지 않으면 검사를 건너뛴다.
- 대량 동기화(전사) 작업이 진행 중인 동안에는 새 추론 작업자를 띄우는 것을 막는다. 동기화 단계와 추론 단계를 섞지 말라는 규칙이며, 이 출발 차단이 그 규칙을 강제하는 정식 지점이다.
- 묶이는 부모 작업에 시나리오 식별자가 없으면 경고를 남긴다(차단이 아닌 경고가 기본).
- 마침 등록할 부모 작업을 찾지 못하면 고아 작업을 만들지 않고 조용히 지나간다.

## provenance

훅 매니페스트(`clawket/hooks/hooks.json`)의 SubagentStart 항목이 얇은 진입점(`clawket/adapters/claude/subagent-start.cjs`)을 호출하고, 실제 로직은 공유 핸들러(`clawket/adapters/shared/claude-hooks.cjs`의 `runSubagentStart`)에 있다. 배치·동기화·시나리오 가드는 같은 파일의 `checkX7BatchSize`·`checkX9SyncReasoning`·`checkX3ScenarioId`가 담당한다. 제공 컴포넌트는 Claude 어댑터(`component.claude-adapter`)이며, 자식 작업 생성·바인딩은 CLI 작업 명령(`endpoint.cli-task`)을 거친다.

## 미확정 (OPEN)
- [ ] OPEN: 자식 작업 생성 실패 시 부모로 후퇴하는 분기의 운영 빈도·영향이 SSOT 본문 수준에서 충분히 설명됐는지 owner 확인 필요.
