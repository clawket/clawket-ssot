---
id: endpoint.skill-plan-design
kind: Endpoint
title: 스킬 clawket-plan-design
definition: 작성된 시나리오로부터 Done 명제를 고정하고 시나리오를 유닛으로 분해해 의존 그래프와 수렴 조건을 선언함으로써, 어떤 작업이든 시작되기 전에 플랜과 유닛 골격을 사전 설계하는 스킬이다.
realizedBy:
  - component.skills
  - component.cli
implementedIn:
  - clawket/.claude-plugin/plugin.json
  - clawket/skills/clawket-plan-design/SKILL.md
relatesTo:
  - to: concept.plan
    type: mutates
    note: 8개 필수 본문 섹션을 갖춘 플랜을 만들고 승인해 active로 전환한다
  - to: concept.unit
    type: mutates
    note: 시나리오를 유닛으로 분해하고 순차·병렬 모드와 의존 그래프를 선언한다
  - to: concept.scenario
    type: reads
    note: 시나리오 작성 스킬이 산출한 시나리오 지식을 입력으로 받아 유닛에 1:1로 환원한다
  - to: concept.convergence
    type: relates-to
    note: 결함=0·시나리오오류=0이 2라운드 연속이라는 수렴 조건을 Done 정의에 못박는다
governedBy:
  - invariant.scenario-id-one-to-one
  - invariant.one-active-plan
  - invariant.convergence-two-rounds
  - invariant.plan-is-source-of-truth
governs: []
dependsOn:
  - endpoint.cli-plan
  - endpoint.cli-unit
consumedBy: []
servesPersona:
  - persona.llm-coding-agent
  - persona.human-operator
impacts:
  - concept.plan
  - concept.unit
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

작업을 시작하기 전에 "무엇을 만들 것이고 무엇이 완료의 증거인가"를 먼저 못박는 사전 설계 스킬이다. 시나리오 작성 스킬이 만들어낸 도메인별 시나리오 지식을 받아, 이를 플랜과 유닛이라는 골격으로 옮긴다. 핵심은 일을 시작하기 전에 Done의 정의를 외부에서 검증 가능한 명제로 고정하고, 모든 시나리오를 유닛에 빠짐없이 배치하며, 유닛 사이의 선후 관계와 언제 끝났다고 판정할지를 명시하는 것이다. 작업 루프의 "플랜+유닛 사전 설계" 진입점이다.

## 요청 / 응답

들어오는 입력은 도메인별 시나리오 지식, 프로젝트 식별자, 그리고 플랜 제목에 쓸 도메인 이름이다.

산출은 다음과 같다. 먼저 8개 필수 본문 섹션을 갖춘 플랜을 만든다 — 제목(한 개의 명사구), Done 정의(외부에서 검증 가능한 명제이며 수렴 조건 포함), 유닛 분해(모든 시나리오는 정확히 하나의 유닛으로 환원), 유닛별 시나리오 하한과 세부 영역, 유닛 의존 그래프(순차/병렬과 의존 식별자), 수렴 조건, 롤백 트리거, 단일 진실 소재(플랜 식별자와 시나리오 지식의 권위 있는 위치)다. 다음으로 시나리오를 유닛으로 분해하며 각 유닛에 순번·모드·목표·시나리오 수·의존을 부여한다. 마지막으로 플랜을 승인해 draft에서 active로 올린다. 결과로 플랜 식별자, 유닛 식별자 목록과 순번 매핑, 다음 단계(검증 라운드 실행) 안내를 돌려준다.

## 권한 / 제약

사람 운영자와 코딩 에이전트가 사용한다. 강제·점검되는 제약은 다음과 같다.

- Done 정의는 "구현 완료" 같은 자기참조 표현이 아니라 외부에서 검증 가능한 명제여야 하며, 수렴 조건(결함=0 그리고 시나리오오류=0이 2라운드 연속)을 반드시 포함한다.
- 모든 시나리오는 정확히 하나의 유닛으로 환원되어야 한다.
- 태스크는 플랜이 active가 되기 전에는 시작할 수 없다. 한 프로젝트에는 active 플랜이 하나만 존재한다(전환 구간에서 잠시 둘이 겹칠 수 있으며, 그때는 이전 플랜을 완료 또는 draft로 먼저 되돌린다).
- 유닛 수는 12개 이하를 기본으로 하며 초과는 플랜 본문에서 정당화해야 한다.
- 실행 가능한 동사가 없는 검토·분석·조사형 태스크, "유연하게/추후 정제" 같은 미결정, 시나리오 식별자로 추적되지 않는 태스크, 시간 기반 종료 조건은 즉시 거부하고 플랜을 재구성한다.
- 자율 루프가 이 스킬을 부를 때는 사용자 데이터 경로 쓰기, DB 파괴 연산, git 되돌리기·커밋·푸시 등이 금지된다.

## provenance

플러그인 매니페스트(`clawket/.claude-plugin/plugin.json`)가 이 스킬을 등록하고, 절차·검증 체크리스트·안티패턴은 스킬 명세(`clawket/skills/clawket-plan-design/SKILL.md`)에 정의된다. 실제 플랜·유닛 생성과 승인은 CLI 플랜 명령(`endpoint.cli-plan`)과 유닛 명령(`endpoint.cli-unit`)을 거친다. 제공 컴포넌트는 번들 스킬(`component.skills`)과 CLI(`component.cli`)다.

## 미확정 (OPEN)
- [ ] OPEN: 유닛 수 12개 상한의 근거와 초과 정당화의 판정 주체(스킬 내부 자동 점검 vs owner)를 확인 필요.
