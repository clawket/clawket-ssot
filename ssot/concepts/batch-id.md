---
id: concept.batch-id
kind: Concept
title: 배치 ID(batch_id)
definition: "같은 서브에이전트 호출로 한꺼번에 생성된 태스크들을 묶는 식별자로, 주의 분산(attention-drift) 감사를 위해 태스크 상세에서 같은 배치를 공유하는 형제 태스크 링크로 노출된다."
relatesTo:
  - to: concept.task
    type: belongs-to
    note: 배치 ID는 태스크의 PDD v3.0 메타데이터 한 필드이며, 같은 값을 가진 태스크들이 한 배치를 이룬다.
  - to: persona.qa-reasoner-subagent
    type: relates-to
    note: 같은 서브에이전트(검증 추론) 호출이 만든 태스크들이 같은 배치 ID로 묶인다.
  - to: concept.tsv-evidence
    type: relates-to
    note: 검증 TSV 근거의 batch_id 항목과 같은 배치 식별자로, 어느 배치가 어떤 판정을 냈는지 추적한다.
governedBy: []
governs: []
dependsOn:
  - concept.task
implementedIn:
  - web/src/types.ts
  - web/src/components/TaskDetail.tsx
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:concept"]
---

<!-- 작성 고도: 자연어. 코드 옮겨적기 금지. 분기/의도 불명은 OPEN. -->

## 정의

배치 ID(batch_id)는 **같은 서브에이전트 호출이 한꺼번에 만들어 낸 태스크들을 한 묶음으로 묶는 식별자**다. PDD v3.0 의 태스크 메타데이터 한 필드로, 같은 batch_id 값을 공유하는 태스크들이 곧 하나의 "배치(형제 태스크 집합)"다.

존재 이유는 **주의 분산(attention-drift) 감사**다 — 한 서브에이전트가 한 번의 호출에서 너무 많은 태스크를 쏟아내면 뒤로 갈수록 주의가 흐려질 수 있다. 배치 ID로 묶어 두면, 사람이 "이 태스크가 어떤 배치의 일부였고, 같은 호출에서 함께 만들어진 형제가 몇 개였는지"를 추적해 그 영향을 점검할 수 있다.

웹 태스크 상세 화면은 배치 ID가 있으면 그것을 표시하고, **같은 배치를 공유하는 형제 태스크 링크**를 함께 보여준다: 같은 batch_id 를 가진 태스크들을 조회해 현재 태스크를 뺀 형제 수를 배지로 보이고("배치 내 N개"), 펼치면 형제 목록에서 특정 태스크로 바로 이동할 수 있다. 형제가 없으면 "형제 없음", 조회 실패 시 실패 표시, 값이 없으면 대시로 그린다.

## 엔티티 (DB)

배치 ID는 태스크 레코드의 한 선택적 문자열 필드다(스키마 v3.0 이전 옛 레코드는 null). 같은 PDD v3.0 묶음에 시나리오 ID·근거(evidence)가 함께 있다. 형제 조회는 batch_id 로 태스크 목록을 필터하는 방식이다.

## API 표면

웹은 배치 전용 엔드포인트를 두지 않는다 — 같은 배치 형제를 찾을 때는 태스크 목록 조회를 batch_id 로 필터한다. 배치 ID 자체는 태스크 레코드의 한 필드로 함께 조회·갱신된다.

## 불변식

- 같은 batch_id 값을 가진 태스크들이 한 배치다 — 배치는 별도 엔티티가 아니라 공유 식별자로 정의되는 묶음이다.
- 형제 집합에서 현재 태스크 자신은 제외하고 센다(형제 수 = 같은 배치 - 본인).
- 배치 ID는 태스크에 종속된 한 필드이며 독립 엔티티가 아니다.

## 구현 위치 (provenance)

- `web/src/types.ts` — 태스크의 배치 ID 필드 선언과 "같은 서브에이전트 호출이 만든 태스크를 attention-drift 감사를 위해 묶는다" 의도 주석.
- `web/src/components/TaskDetail.tsx` — batch_id 로 형제 태스크를 조회해 개수 배지·펼침 목록·형제 이동을 그리는 배치 형제 링크 로직과 PDD 메타데이터 표시 영역.

## 미확정 (OPEN)
- [ ] OPEN: 한 배치당 태스크 개수 상한(서브에이전트 호출당 배치 크기 한도)의 데몬·훅 측 강제 규칙은 별도 확인 필요.
