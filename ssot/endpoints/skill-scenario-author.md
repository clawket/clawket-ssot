---
id: endpoint.skill-scenario-author
kind: Endpoint
title: 스킬 clawket-scenario-author
definition: 새 도메인·메뉴 영역의 원자적 사용자 시나리오를 As-a/I-want/So-that + Given/When/Then 엄격 형식으로 작성해 의도만 담은 도메인별 명세 지식으로 영속화하는 작업 루프 진입 스킬이다.
realizedBy:
  - component.skills
  - component.cli
implementedIn:
  - clawket/.claude-plugin/plugin.json
  - clawket/skills/clawket-scenario-author/SKILL.md
relatesTo:
  - to: concept.scenario
    type: mutates
    note: US-<DOMAIN>-<NNN> 식별자를 가진 원자적 시나리오를 작성한다
  - to: concept.knowledge
    type: mutates
    note: 도메인별 시나리오를 type=spec 지식 항목으로 영속화한다
governedBy:
  - invariant.scenario-snapshot-only
  - invariant.scenario-amend-intent-only
governs: []
dependsOn:
  - endpoint.cli-knowledge
consumedBy:
  - endpoint.skill-plan-design
servesPersona:
  - persona.llm-coding-agent
  - persona.human-operator
impacts:
  - concept.scenario
  - concept.knowledge
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

새로운 도메인이나 메뉴 영역을 다룰 때, 그 영역에서 사용자가 무엇을 할 수 있어야 하는지를 잘게 쪼갠 시나리오로 적어내는 스킬이다. 핵심 원칙은 "의도만 담는다"는 것 — 코드 위치를 인용하지 않고, 지금 구현됐는지 같은 상태도 적지 않으며, 오직 의도된 동작만 기록한다. 산출물은 다음 단계인 플랜 설계 스킬의 입력이 된다. 작업 루프의 도메인 시나리오 작성 진입점이다.

## 요청 / 응답

들어오는 입력은 적어도 하나 이상이다 — 도메인과 메뉴/세부 영역, 코드 위치·UI 라벨 목록·비즈니스 정책 같은 원자료, 또는 정제 스킬이 넘긴 기존 시나리오 지식(개정용)이다.

산출은 엄격한 형식의 원자적 시나리오다. 각 시나리오는 "누가-무엇을 원하고-그래서 어떤 가치를 얻는가" 세 줄과, 정확히 하나의 "주어진 상태에서-어떤 트리거가 일어나면-어떤 결과가 기대된다"는 수용 기준으로 구성된다. 식별자에는 도메인이 들어가며(예: 도메인-영역-번호), 트리거·결과·전제·분기가 달라지면 별도 시나리오로 나눈다. 메뉴 하나당 50개에서 수백 개 시나리오가 사전 설계 시점에 정상 범위다. 완성된 시나리오는 메뉴·화면·기능 단위로 하나의 지식 항목(명세 유형)으로 저장하며, 결과로 도메인 영역별 지식 항목, 그 식별자, 다음 단계(플랜 설계) 안내를 돌려준다.

## 권한 / 제약

사람 운영자와 코딩 에이전트가 사용한다. 강제·점검되는 제약은 다음과 같다.

- 시나리오 본문에 들어가면 안 되는 것: 코드 위치 인용, "구현됨/동작함/처리됨" 같은 모호한 상태 표현, 알려진 결함·버그 언급(의도된 동작만 적는다), 코드 조각, 여러 시나리오에 걸쳐 한 번만 적는 그룹 헤더, 변경 이력.
- 시나리오 하나는 하나의 검증 가능한 단언이어야 한다 — 둘 이상의 수용 기준이 섞였으면 쪼갠다.
- 자료가 충돌하면 우선순위는 명시적 사용자 결정 > 현재 구현 > 옛 문서 순이다.
- 지식 본문은 항상 현재 의도(스냅샷)만 담는다. 개정이 일어나도 과거 내용을 본문에 병기하지 않으며, 변경 이력은 취소된 QA 태스크 코멘트와 감사 지식으로 분리해 보관한다.
- 정제 스킬이 부른 개정의 유일한 정당 사유는 의도 불일치다. 시간·비용·복잡도·코드 영향 크기를 이유로 시나리오를 약화시키는 개정은 거부하고 별도 수정 플랜을 등록한다.
- 개정은 세 갈래다 — 원자적 분할(원래 식별자는 영구 폐기, 새 식별자가 순번을 잇는다), 의도 재정의(식별자 유지, 본문만 변경), 삭제(식별자 영구 폐기).

## provenance

플러그인 매니페스트(`clawket/.claude-plugin/plugin.json`)가 이 스킬을 등록하고, 작성 절차·금지 항목·자기점검·개정 규칙은 스킬 명세(`clawket/skills/clawket-scenario-author/SKILL.md`)에 정의된다. 시나리오 지식의 영속화는 CLI 지식 명령(`endpoint.cli-knowledge`)을 거치며, 산출물은 플랜 설계 스킬(`endpoint.skill-plan-design`)이 소비한다. 제공 컴포넌트는 번들 스킬(`component.skills`)과 CLI(`component.cli`)다.

## 미확정 (OPEN)
- [ ] OPEN: 시나리오 분할·삭제 시 폐기된 식별자의 재사용 금지가 데몬 차원에서 강제되는지(스킬 규약만인지) owner 확인 필요.
