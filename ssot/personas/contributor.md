---
id: persona.contributor
kind: Persona
title: 기여자(외부 개발자)
purpose: "Clawket 멀티레포에 코드·문서·플랜·버그리포트를 기여하되, decompose→contract→execute 규약과 태스크-PR 1:1 규칙을 지키며 작업하는 것"
definition: "Clawket의 여러 레포에 기여하는 외부 개발자. 모든 기여를 분해→계약→실행 3단계로 진행하며, 코드 편집 전 활성 태스크가 필수이고, 하나의 태스크는 하나의 PR로 닫고 PR에 태스크 티켓을 링크한다."
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:persona"]
governedBy: [domain.hook-enforcement, concept.execution-envelope]
servesPersona: []
relatesTo:
  - to: persona.coding-agent
    type: relates-to
    note: "기여자는 코딩 에이전트를 통해 작업하며 같은 decompose→contract→execute 루프와 훅 제약을 공유한다."
  - to: concept.execution-envelope
    type: depends-on
    note: "각 태스크(리프)에 intent·prompt_template·success_criteria를 담은 19필드 실행 봉투를 서명한 뒤 실행한다."
  - to: endpoint.hook-pre-tool-use
    type: relates-to
    note: "활성 태스크 없이 코드 편집을 시도하면 이 훅이 변경 도구를 하드블록한다."
  - to: concept.task
    type: relates-to
    note: "하나의 태스크당 하나의 PR을 닫고, PR 설명에 태스크 티켓(예: Closes LM-128)을 링크한다."
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지. 분기/의도 불명은 OPEN. -->

## 누구인가

Clawket 제품 자체에 **코드·문서·플랜·버그리포트를 기여하는 외부 개발자**다. Clawket은 여섯 개의 활성 레포(플러그인 셸, CLI, 데몬, 웹, 데스크탑, 랜딩)로 나뉘어 있고, 기여자는 그중 하나 이상에 변경을 제출한다.

이 페르소나의 핵심은 기여자도 **Clawket의 작업 규약을 그대로 따른다**는 점이다. Clawket은 자기 자신을 Clawket으로 관리하는 재귀적 개발 구조를 갖는다 — 기여자가 코드를 고치려면 먼저 활성 Clawket 태스크가 있어야 하고, 활성 태스크 없이 파일을 건드리면 훅이 작업을 막는다. 즉 기여 워크플로우 자체가 제품의 가드레일 위에서 돌아간다.

## 무엇을 하려고 제품을 쓰나

기여자의 목표는 **"규약을 지키며 추적 가능한 변경을 멀티레포에 안착시키는 것"**이다. 모든 기여는 코드든 문서든 버그리포트든 동일한 3단계를 순서대로 거친다.

- **분해(decompose)** — 일을 플랜→유닛→태스크로 쪼개, 에이전트가 한 호흡에 끝낼 수 있는 단위까지 내린다. 트리도 없이 "그냥 코드부터 쓰고 싶다"면 이 단계를 건너뛰는 것이다.
- **계약(contract)** — 각 태스크에 실행 봉투를 서명한다. 무엇을 위한 일인지·어떻게 접근할지·완료 조건이 무엇인지를 채워야 하며, 이 계약이 곧 에이전트가 읽고 테스트가 통과 여부를 가르고 타임라인이 재생하는 단일 진실이 된다.
- **실행(execute)** — 비로소 코드를 돌린다. 1·2단계가 안 됐으면 실행이 훅에서 차단되며, 차단의 올바른 대응은 돌아가서 계약을 마무리하는 것이다.

제출 단계에서 기여자가 지키는 구체 규칙은 다음과 같다.

- **활성 태스크 먼저** — 활성 Clawket 태스크 없이 코드를 편집하지 않는다. 태스크가 없으면 먼저 만들거나 활성화한다.
- **하나의 태스크, 하나의 PR** — 하나의 PR은 정확히 하나의 태스크(또는 같은 유닛 아래 밀접하게 묶인 태스크 집합)를 닫는다. 유닛을 가로지르는 PR은 분해 단계를 건너뛴 신호다.
- **태스크 링크** — PR 설명에 태스크 티켓을 참조한다(예: `Closes LM-128`).
- **명시 지시 없는 커밋 금지** — 코딩 에이전트는 자율적으로 커밋하지 않는다. `git commit`은 사람이 주도한다.

## 미확정 (OPEN)
- [ ] OPEN: 이 페르소나의 owner(책임자) 미지정.
- [ ] OPEN: 외부 기여자가 Clawket을 설치·활성 태스크를 만드는 온보딩 경로(사내 빌더와 동일한지, 별도 안내가 있는지) 코드/문서 근거 미확인.
