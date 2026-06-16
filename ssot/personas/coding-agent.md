---
id: persona.coding-agent
kind: Persona
title: 코딩 에이전트(LLM 실행 주체)
purpose: "활성 태스크의 계약(실행 봉투) 안에서 실제 코드를 작성·수정하고, 작업 상태를 스스로 등록·진행·완료하며 세션 간 컨텍스트를 잃지 않는 것"
definition: "Claude Code 위에서 실제 코드를 짜고 고치는 LLM 실행 주체. decompose→contract→execute 루프를 따라 태스크를 다루며, CLI와 MCP로 사람과 같은 상태를 읽고 쓴다. 활성 태스크가 없으면 변경 도구가 훅에 의해 차단된다."
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:persona"]
governedBy: [domain.hook-enforcement, concept.execution-envelope]
servesPersona: []
relatesTo:
  - to: persona.builder-human
    type: relates-to
    note: "사람의 승인·검토를 전제로 실제 실행을 담당하며, 동일한 작업 상태를 공유한다."
  - to: endpoint.hook-pre-tool-use
    type: relates-to
    note: "활성 태스크가 없으면 Edit·Write·Bash·Agent 등 변경 도구 호출이 이 훅에서 차단된다."
  - to: concept.execution-envelope
    type: depends-on
    note: "intent·prompt_template·success_criteria를 담은 실행 계약을 읽어 그 계약대로 실행한다."
  - to: capability.manage-task-lifecycle
    type: calls
    note: "태스크를 생성·시작·완료·취소하고 evidence와 함께 done 처리한다."
  - to: capability.decompose-task
    type: calls
    note: "success_criteria와 분해 정책으로부터 하위 태스크를 제안·수락해 작업 트리를 만든다."
  - to: capability.rag-pull
    type: calls
    note: "MCP 도구로 과거 지식·태스크·결정을 의미 검색해 세션 간 컨텍스트를 복원한다."
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지. 분기/의도 불명은 OPEN. -->

## 누구인가

Clawket이 통제하는 **실제 코드 실행 주체**다. 사람이 큰 그림을 잡고 승인하면, 그 계획 안에서 파일을 고치고 명령을 돌리며 작업을 완료하는 쪽이 바로 이 코딩 에이전트(Claude Code)다.

사람 운영자와 달리 이 주체는 항상 **규칙의 적용 대상**이다. Clawket의 핵심 제약 한 줄 — "태스크 없으면 코드 수정 못 함" — 이 겨냥하는 대상이 바로 코딩 에이전트다. 활성 태스크가 지정되어 있지 않으면 Edit·Write·Bash·Agent 같은 변경 도구를 호출하려는 순간 훅이 작업을 막는다. 처음에는 답답해하지만 방향이 잡히면 스스로 태스크를 만들고 일하게 된다.

이 에이전트는 사람과 **같은 상태를 본다**. 사람은 웹 대시보드로, 에이전트는 CLI와 MCP로 동일한 프로젝트·플랜·태스크·지식에 접근한다. 한쪽이 만든 변경은 다른 쪽에 그대로 보인다.

## 무엇을 하려고 제품을 쓰나

이 주체의 목표는 **"계약된 작업을, 기록을 남기며, 세션이 끊겨도 이어지게 수행하는 것"**이다. 기여의 표준 흐름인 decompose→contract→execute 3단계를 따른다.

- **분해(decompose)** — 일을 플랜→유닛→태스크로 쪼개, 한 번에 끝낼 수 있는 단위까지 내린다.
- **계약(contract)** — 각 태스크에 실행 봉투를 서명한다. 최소한 무엇을 위한 일인지(intent), 어떻게 접근할지(prompt_template), 무엇이 완료 조건인지(success_criteria)를 채운다. 자유 형식 프롬프트("그냥 이 기능 추가해줘")는 검증 가능한 완료 조건이 없어 계약으로 인정되지 않는다.
- **실행(execute)** — 비로소 코드를 돌린다. 1·2단계를 건너뛰면 실행 단계가 훅에서 하드블록되며, 건너뛰기 플래그는 없다. 차단의 올바른 대응은 "돌아가서 계약을 마무리하라"이다.

작업을 마치면 태스크를 evidence와 함께 done으로 닫고, 데몬이 유닛·사이클·플랜의 완료를 자동으로 상위로 전파한다.

또한 새 세션을 시작할 때 "어제 어디까지 했지"를 사람이 다시 설명할 필요가 없도록, MCP의 읽기 전용 도구로 과거 지식·결정·유사 태스크를 직접 끌어와(pull) 컨텍스트를 복원한다.

## 미확정 (OPEN)
- [ ] OPEN: 이 페르소나의 owner(책임자) 미지정.
- [ ] OPEN: 인벤토리에는 `persona.llm-coding-agent`(README/rules/hooks 출처)가 별도로 존재 — 두 노드의 경계/통합 여부는 SSOT 차원 정리 필요(현재는 BLOG/CONTRIBUTING 출처 기반의 decompose→contract→execute 관점으로 기술).
