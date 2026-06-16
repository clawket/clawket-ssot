---
id: capability.author-scenarios
kind: Capability
title: 시나리오 작성·정제
definition: 사람이나 LLM 에이전트가 어떤 도메인·메뉴 영역의 사용자 의도를 As-a / I-want / So-that + Given/When/Then 형식의 원자적 시나리오로 작성하고, 검증 라운드가 "시나리오 자체가 잘못됐다"고 지적한 항목에 대해 분할·의도 재정의·삭제 중 하나로 정리하는 일이다.
purpose: 검증·구현의 단위가 되는 원자적 사용자 시나리오를 의도 수준에서 작성하고, 라운드 중 드러난 잘못된 시나리오를 손상 없이 정제한다.
servesPersona:
  - persona.human-operator
  - persona.llm-coding-agent
realizedBy:
  - component.cli
  - component.daemon
  - component.skills
implementedIn:
  - clawket/skills/clawket-scenario-author/SKILL.md
  - clawket/skills/clawket-scenario-refine/SKILL.md
relatesTo:
  - { to: concept.scenario, type: mutates, note: "시나리오를 새로 만들거나(작성) 분할·재정의·삭제로 바꾼다(정제)" }
  - { to: concept.knowledge, type: mutates, note: "시나리오는 type=spec 지식으로, 정제 이력은 type=note 감사 지식으로 보존된다" }
  - { to: endpoint.cli-knowledge, type: calls, note: "knowledge create 로 시나리오 지식과 감사 지식을 등록한다" }
  - { to: endpoint.cli-comment, type: calls, note: "정제 결정의 영구 흔적을 취소된 QA 태스크 코멘트로 남긴다" }
  - { to: domain.pdd-verification, type: relates-to, note: "작성은 검증 루프의 입력, 정제는 scenario_error 행에 대한 라운드 내 후속 처리" }
  - { to: capability.run-verification-loop, type: relates-to, note: "검증 라운드가 scenario_error 를 보고하면 이 역량의 정제 절차가 호출된다" }
impacts:
  - concept.scenario
  - concept.knowledge
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:capability"]
---

## 사용자가 할 수 있는 일

새 도메인이나 메뉴 영역의 동작을 검증 가능한 단위로 쪼개 적는 일이다. "어떤 사용자가, 무엇을 하고 싶고, 그래서 무슨 가치를 얻는가"를 한 줄씩 적고, 그 아래에 "이런 상태에서 이렇게 하면 이런 결과가 나온다"는 합격 조건을 정확히 하나씩 붙인다. 이렇게 만든 시나리오는 곧 검증의 단위이자, 나중에 태스크가 1:1로 매달리는 대상이 된다.

작성의 핵심 원칙은 "의도만 담는다"는 것이다. 코드 위치(file:line)나 "구현됨/동작함" 같은 구현 상태, 알려진 버그는 시나리오에 넣지 않는다 — 시나리오는 "원래 의도된 동작"을 기술하지, 지금 코드가 어떤지를 옮겨 적지 않는다. 한 시나리오에는 합격 조건이 정확히 하나여야 하며, 트리거·결과·전제·분기가 달라지면 별개의 시나리오로 나눈다.

정제는 검증 라운드가 "이 시나리오는 코드가 아니라 시나리오 쪽이 잘못됐다(scenario_error)"고 판정한 항목을 손보는 일이다. 정제의 정당한 사유는 오직 "의도 불일치" 하나뿐이다 — 한 시나리오에 두 가정이 섞였거나, 기대 결과가 제품 비전과 어긋나거나, 범위에서 빠진 경우다. 시간·비용·복잡도·코드 영향 크기를 이유로 시나리오를 약화하는 것은 거부된다.

## 행위

- 도메인·영역을 정해 원자적 시나리오 묶음을 작성하고, type=spec 지식 한 건(메뉴/화면/기능당 하나)으로 저장한다.
- 작성 시 금지 사항을 자가 점검한다: 코드 인용 0건, "구현됨/동작함" 류 표현 0건, 알려진 버그를 의도로 환원, ID 전부 고유, 변경 이력 본문 혼입 금지.
- 정제 시 세 갈래 중 정확히 하나를 적용한다 — 원자 분할(1→N, 원본 ID는 영구 폐기하고 새 ID를 이어 발번), 의도 재정의(ID 유지, 본문만 교체), 삭제(ID 영구 폐기).
- 정제 결정의 영구 흔적을 취소된 QA 태스크 코멘트와 감사 지식(type=note, append-only)에 남긴다.
- 시나리오 지식 본문은 항상 현재 의도만 담고 변경 이력을 본문에 남기지 않는다 — 이력은 코멘트와 감사 지식으로 분리된다.

## 시스템 흐름

사용자나 에이전트가 도메인·소스 자료를 주면, 에이전트가 현재 구현·UI 라벨·메뉴 구조를 1차 단서로 삼아 원자적 시나리오 초안을 만든다(낡은 문서는 참고용, 충돌 시 "명시적 사용자 결정 > 현재 구현 > 낡은 문서" 순). 완성된 묶음은 CLI를 통해 데몬에 type=spec 지식으로 등록되어 검색·검증의 입력이 된다.

정제는 검증 라운드가 scenario_error 행을 보고할 때 시작된다. 각 행의 수정 제안과 사유를 검토해 "의도 불일치"인지 검증한 뒤(시간·비용 사유면 멈추고 사용자 확인을 요청), 세 갈래 중 하나를 적용한다. 결정은 취소된 QA 태스크 코멘트와 감사 지식에 기록하고, 원본 시나리오 지식을 현재 의도만 남도록 갱신한 다음, 다음 라운드(R+1)로 제어를 넘긴다. 한 번 발번된 시나리오 ID는 재사용되지 않으며 폐기·분할된 ID는 영구히 비워둔다(라운드 간 추적성 보존).

## 어디에 구현되어 있나

작성·정제 절차는 두 개의 번들 스킬에 정의되어 있다 — 작성은 `clawket/skills/clawket-scenario-author/SKILL.md`, 라운드 내 정제는 `clawket/skills/clawket-scenario-refine/SKILL.md`. 실제 시나리오·감사 지식의 저장은 CLI의 knowledge 명령을 통해 데몬·SQLite로, 정제 흔적은 comment 명령을 통해 보존된다.

## 미확정 (OPEN)
- [ ] OPEN: scenario_error 정제가 어떤 도메인 검색·전사 엔드포인트를 거치는지(정제된 시나리오의 재평가 경로)는 코드에서 추가 확인 필요.
