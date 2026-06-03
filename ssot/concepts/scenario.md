---
id: concept.scenario
kind: Concept
title: Scenario(시나리오)
definition: "검증의 최소 단위가 되는 원자적 사용자 시나리오. '누가(As a)·무엇을 원하나(I want)·왜(So that)' 세 줄에 '어떤 상태에서(Given)·무슨 일이 일어나면(When)·무엇이 기대되나(Then)' 하나를 더한 형식으로, 코드가 아니라 의도(원하는 동작)만 담는다. US-<도메인>-<번호> 형태의 영구 식별자를 가지며, 한 시나리오는 검증 라운드에서 하나의 검증 태스크와 1:1로 맞물린다."
relatesTo:
  - to: concept.knowledge
    type: belongs-to
    note: 시나리오들은 도메인/메뉴 단위로 묶여 type=spec 지식 항목 한 건의 본문으로 보관된다
  - to: concept.task
    type: relates-to
    note: 검증 라운드에서 시나리오 한 건은 검증 태스크 한 건과 1:1로 매핑되고, 태스크는 scenario_id 로 어느 시나리오를 검증하는지 가리킨다
  - to: concept.tsv-evidence
    type: relates-to
    note: 시나리오를 코드에 대조한 판정 결과가 TSV 근거 한 행(scenario_id 로 식별)으로 산출된다
  - to: capability.author-scenarios
    type: realizes
    note: 시나리오 작성·정제 능력이 이 개념을 만들어내고 진화시킨다
governedBy: []
governs: []
dependsOn: []
implementedIn:
  - clawket/skills/clawket-scenario-author/SKILL.md
  - clawket/skills/clawket-scenario-refine/SKILL.md
  - clawket/skills/clawket-verify-loop/SKILL.md
  - cli/src/main.rs
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:concept
---

<!-- 작성 고도: 자연어 우선. 코드 식별자는 provenance/근거에만. -->

## 정의

시나리오는 Clawket의 PDD(시나리오 주도 개발) 흐름에서 "코드가 무엇을 해야 하는가"를 한 덩어리로 표현하는 원자적 진술이다. 검증의 기준점이자 단위이며, 실제 코드와 대조되는 "정답지" 역할을 한다.

한 시나리오는 다음 형식을 따른다.

- **As a(누가)**: 그 동작을 원하는 주체.
- **I want(무엇을)**: 하고자 하는 행위.
- **So that(왜)**: 그 행위로 얻는 가치.
- **Acceptance — Given/When/Then**: 어떤 상태에서(Given), 무슨 트리거가 발생하면(When), 무엇이 일어나야 하는가(Then). 한 시나리오에는 Given/When/Then 한 묶음만 들어간다.

핵심 성질은 "원자성"과 "의도 한정"이다.

- **원자성**: 한 시나리오 = 검증 가능한 단언 하나. 트리거가 다르거나, 기대 결과가 다르거나, 선행 조건이 다르거나, 코드 분기가 다르면 별개의 시나리오로 쪼갠다. 메뉴 하나에 수십~수백 개의 시나리오가 나오는 것이 정상이다.
- **의도 한정**: 시나리오 본문에는 코드 위치(file:line)·구현 상태 표현("구현됨", "동작함", "처리됨")·알려진 버그·코드 스니펫을 절대 적지 않는다. 이미 알려진 버그가 있어도 "의도된 올바른 동작"으로 다시 써서 기록한다. 즉 시나리오는 "현재 코드가 무엇을 하는가"가 아니라 "코드가 무엇을 해야 하는가"를 담는다.

시나리오는 자유롭게 떠다니지 않는다. 도메인/메뉴/화면/기능 단위로 묶여 위키 지식 항목(type=spec) 한 건의 본문으로 보관된다(→ concept.knowledge). 검증 라운드에 들어가면 시나리오 한 건마다 검증 태스크 한 건이 생겨 코드와 대조되고(→ concept.task), 그 대조 결과가 TSV 근거 한 행으로 떨어진다(→ concept.tsv-evidence).

## 엔티티 (DB)

시나리오 자체는 독립된 테이블로 저장되지 않는다. 시나리오 본문은 위키 지식(type=spec) 항목 안에 텍스트로 들어가며, 검증 흐름에서 코드와 연결될 때 비로소 식별자 형태로 다른 데이터에 박힌다.

- **시나리오 식별자(scenario_id)**: `US-<도메인>-<3자리 번호>` 형식(예: `US-DAEMON-API-001`). 도메인 접두사를 반드시 포함한다. 번호는 한 번 부여하면 영구히 그 시나리오의 것이며, 삭제되거나 분할로 사라진 번호도 재사용하지 않는다(다음 라운드 간 추적성 보존).
- **태스크와의 연결**: 검증 태스크는 자신이 검증하는 시나리오를 가리키는 scenario_id 값을 보관한다. 태스크 목록·검색을 scenario_id 로 필터할 수 있다. 검증 라운드에서 새로 만든 태스크가 scenario_id 를 비워두면 훅이 이를 취소시킨다(시나리오 없는 검증 태스크 금지).

## API 표면

시나리오는 두 경로로 다뤄진다.

- **작성·보관**: 시나리오 본문은 위키 지식(type=spec)으로 만들고 수정한다. 한 메뉴/화면/기능당 지식 한 건이 원칙이며, 제목은 "<도메인> <영역> 시나리오" 형식을 쓴다.
- **검증 연결**: 검증 라운드에서 태스크를 만들 때 scenario_id 를 부여해 그 시나리오와 묶는다. 태스크 조회·검색을 scenario_id 로 필터해 특정 시나리오에 연결된 태스크들을 찾을 수 있다.

## 불변식

- **식별자 영구성**: scenario_id 의 번호는 한 번 부여되면 영구히 고정이고 재사용 불가. 삭제·분할로 사라진 번호도 다시 쓰지 않는다.
- **원자성**: 한 시나리오는 Given/When/Then 한 묶음, 검증 가능한 단언 하나만 담는다. 둘 이상 섞이면 분할 대상이다.
- **의도 한정(히스토리 무유출)**: 시나리오 본문에는 코드 위치·구현 상태·버그·변경 이력(changelog)이 들어가지 않는다. 변경 사유·이력은 시나리오 본문이 아니라 취소된 검증 태스크 코멘트와 별도 감사용 지식(type=note)에만 쌓인다.
- **수정의 유일한 정당 사유**: 시나리오를 고치는 정당한 이유는 "의도 불일치"뿐이다(예: 한 시나리오에 두 가정이 섞임, 기대 결과가 제품 비전과 충돌, 보류 결정). 시간·비용·복잡도·코드 영향 크기를 이유로 시나리오를 약화하는 것은 거부되며, 그런 경우 시나리오를 건드리는 대신 별도 수정 플랜을 만든다.

## 구현 위치 (provenance)

- 시나리오 작성 규칙(형식·금지 내용·보관 위치): `clawket/skills/clawket-scenario-author/SKILL.md`.
- 시나리오 정제(분할/의도 재정의/삭제, 식별자 영구성): `clawket/skills/clawket-scenario-refine/SKILL.md`.
- 검증 라운드에서 시나리오↔태스크 1:1 매핑과 scenario_id 검사: `clawket/skills/clawket-verify-loop/SKILL.md`.
- 태스크에 scenario_id 를 부여·필터하는 CLI 표면(`--scenario-id` 인자, `US-...` 예시): `cli/src/main.rs`(태스크 create/list 정의 부근).

## 미확정 (OPEN)
- [ ] OPEN: scenario_id 형식 검증(US-<DOMAIN>-<NNN> 정규식)을 데몬/훅 중 어디서 강제하는지 — 스킬 문서상 검증 라운드에서 강제한다고만 확인됨, 데몬측 스키마 강제 여부 추적 미완.
