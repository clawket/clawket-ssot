---
id: domain.work-management
kind: Domain
title: 작업 관리(Work Management)
purpose: LLM 주도 개발 작업을 세션을 넘어 영구히 추적·관리하는 구조화된 상태 계층을 제공한다.
definition: Jira+Confluence를 대체하는 작업 관리 영역으로, 프로젝트·플랜·유닛·사이클·태스크라는 다섯 단계 워크플로우와 그 상태 전이를 책임진다. 코드 변경 자체는 포함하지 않고, "무엇을 누가 어떤 순서·자격으로 할 수 있는가"라는 작업 상태의 진실만 다룬다.
servesPersona:
  - persona.llm-coding-agent
  - persona.human-operator
relatesTo:
  - to: domain.hook-enforcement
    type: governs
    note: 작업 상태(활성 태스크·플랜·사이클)가 훅 강제의 허용/차단 판단 기준이 된다.
  - to: domain.knowledge-rag
    type: relates-to
    note: 태스크·결정 컨텍스트를 지식 항목으로 보존해 세션 간 복원의 원천이 된다.
  - to: domain.pdd-verification
    type: relates-to
    note: 검증 루프가 이 워크플로우의 플랜·사이클·유닛·태스크 위에서 돌아간다.
governedBy:
  - invariant.no-work-without-task
  - invariant.start-needs-active-plan-cycle
  - invariant.one-active-plan-per-project
  - invariant.one-active-cycle-per-unit
  - invariant.unit-no-status
  - invariant.evidence-required-on-done
  - invariant.preserve-history
realizedBy:
  - component.cli
  - component.daemon
  - component.sqlite-storage
  - component.web-dashboard
impacts:
  - persona.llm-coding-agent
  - persona.human-operator
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-03
tags:
  - status:active
  - type:domain
---

## 목적

이 영역은 LLM 코딩 에이전트가 하는 개발 작업을 "지금 무엇을, 누가, 어떤 자격으로, 어떤 순서로 하고 있는가"라는 구조화된 상태로 붙잡아 둔다. 구조화된 상태가 없으면 에이전트 세션은 매번 처음부터 시작되어 "내가 어디까지 했지?"에 답이 없고, 무엇을 언제 왜 바꿨는지 기록이 남지 않으며, 병렬로 일하는 여러 에이전트가 서로의 진행을 알지 못한다. 작업 관리 영역은 이 공백을 영구 저장소 위의 다섯 단계 작업 흐름으로 메운다.

핵심 비유는 "Jira + Confluence를 대체한다"이다. 사람 팀이 이슈 트래커와 위키로 일하듯, LLM 에이전트는 이 작업 흐름과 (지식·RAG 영역의) 위키로 일한다.

## 경계와 핵심 개념

이 영역에 포함되는 것은 작업의 골격을 이루는 다섯 계층과 그 부속 기록이다.

- **프로젝트(Project)** — 하나 이상의 작업 디렉터리에 묶인 작업 공간. 구조화된 작업을 시작하기 전 반드시 등록되어야 하며, 활성화 토글로 관리 자체를 켜고 끌 수 있다.
- **플랜(Plan)** — 승인된 의도를 담는 컨테이너(로드맵). 초안→활성→완료 상태를 가지며, 활성 상태가 되어야 태스크를 시작할 수 있다.
- **유닛(Unit)** — 플랜 안에서 태스크를 묶는 순수 그룹핑. 상태도 승인도 없으며, 서로 독립된 유닛의 태스크는 여러 에이전트가 동시에 다룰 수 있다.
- **사이클(Cycle)** — 단일 유닛에 묶인 타임박스 반복(스프린트). 계획→활성→완료를 거치고, 한 유닛에 동시에 활성인 사이클은 하나뿐이며 완료된 사이클은 재시작할 수 없다.
- **태스크(Task)** — 직접 다루는 유일한 원자적 작업 단위. 할 일→진행 중→완료/취소를 거치고, 외부 의존이 있으면 차단됨 상태를 쓴다. 완료로 넘어가려면 근거(evidence)가 반드시 필요하다.

여기에 작업의 부속 기록인 백로그(사이클에 배정되지 않은 태스크 집합), 실행 기록(run), 코멘트, 질문, 사람이 읽는 티켓 번호(CK-N)가 포함된다. 각 리프 태스크에 서명되는 실행 봉투(execution envelope)도 이 영역의 작업 계약 개념이다.

제외되는 것: 실제 코드 변경(에이전트가 별도로 수행), 위키성 지식의 검색·임베딩(지식·RAG 영역), 훅에 의한 도구 차단 자체의 메커니즘(훅 강제 영역).

## 기능

이 영역이 제공하는 사용자 기능은 다음과 같다.

- 작업 디렉터리를 프로젝트로 등록해 구조화된 작업과 가드레일을 켠다.
- 플랜을 제안·등록하고 승인해 활성으로 만들어 태스크 시작 자격을 부여한다.
- 태스크를 생성·시작·완료·취소·차단/해제하고, 근거와 함께 완료 처리한다.
- 백로그의 태스크를 사이클에 배정해 일정에 편입한다.
- 실행 기록·코멘트·태스크 본문 업데이트로 진행 히스토리를 보존한다.

이 기능들은 명령줄 도구와 로컬 데몬·SQLite 저장소가 실제로 구현하고, 사람은 웹 대시보드로 같은 상태를 보고 관리한다.

## 시스템 흐름

전형적인 작업 흐름은 한 방향으로 흐른다. 먼저 작업 디렉터리를 **프로젝트로 등록**한다. 그 안에 **플랜을 만들고 승인**하면 플랜이 활성이 되어 태스크 시작 자격이 생긴다. 플랜 아래에서 작업을 **유닛으로 묶고**, 유닛에 **사이클(스프린트)을 만들어 활성화**한다. 이제 비로소 **태스크를 시작**할 수 있다 — 시작에는 활성 플랜과 활성 사이클이 둘 다 있어야 하기 때문이다.

태스크가 진행되면 실행 기록·코멘트가 쌓이고, 완료로 넘어갈 때는 근거가 요구된다. 태스크가 완료/취소로 끝나면, 그 유닛·플랜·사이클의 모든 하위 작업이 종료 상태인 경우 데몬이 자동으로 상위 단계의 완료를 연쇄(cascade) 처리한다 — 별도 조작이 필요 없다. 취소된 작업은 파괴적 삭제 대신 코멘트로 히스토리를 남기는 것이 규칙이다.

## 다른 도메인과의 관계

- **훅 강제 영역(domain.hook-enforcement)** — 이 영역의 작업 상태(활성 태스크·플랜·사이클이 있는지)가 곧 훅이 도구 호출을 허용할지 차단할지 판단하는 기준이다. 작업 상태가 가드레일을 통치한다.
- **지식·RAG 영역(domain.knowledge-rag)** — 태스크와 의사결정 컨텍스트가 지식 항목으로 보존되어, 다음 세션이 의미 검색으로 과거 맥락을 복원하는 원천이 된다.
- **PDD 검증 영역(domain.pdd-verification)** — 검증 루프의 라운드·배치는 이 영역의 플랜·사이클·유닛·태스크 구조 위에서 실행된다.

## 미확정 (OPEN)
- [ ] OPEN: node.sources의 `src/main.rs:16-49` 등 라인 범위는 CLI(cli/) 구현부를 가리키나, 본 작성에서는 daemon 측 라우터·repo 계층을 근거로 삼았다 — CLI 서브커맨드 정의의 정확한 위치 확정 필요.
