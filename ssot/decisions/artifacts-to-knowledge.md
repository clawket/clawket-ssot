---
id: decision.artifacts-to-knowledge
kind: Decision
title: 별도 artifacts 테이블을 knowledge로 통합해 지식과 산출물을 단일 모델로 다룬다
purpose: "작업 산출물(artifacts)과 위키성 지식(knowledge)을 별개 모델로 둘지, 하나의 모델로 합칠지"
definition: "기존에 분리돼 있던 artifacts 저장소를 knowledge로 흡수·통합해, 결정·노트·근거·산출물 같은 모든 위키성 콘텐츠를 하나의 지식 모델로 다루고, 실행 계약(envelope)의 조건 판정도 새 knowledge 이름 체계를 쓰도록 일원화한다."
relatesTo:
  - to: concept.knowledge
    type: relates-to
    note: 통합 후 모든 위키성 콘텐츠를 담는 단일 모델 — artifacts가 여기로 흡수됐다
  - to: concept.knowledge-entry
    type: relates-to
    note: 통합된 지식 모델의 항목 단위
  - to: concept.execution-envelope
    type: relates-to
    note: 실행 계약의 조건 판정이 artifact 이름에서 knowledge 이름 체계로 함께 전환됐다
  - to: component.sqlite-storage
    type: relates-to
    note: 테이블 통합·뷰 제거가 일어난 저장소
supersedes: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:decision"]
---

<!-- Decision은 append-only. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 supersedes로 잇는다. -->

> Decision은 append-only다. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 `supersedes`로 잇는다. (스냅샷 원칙 — 히스토리는 여기와 git에만)

## 맥락 (Context)

초기 모델에는 작업 산출물을 담는 artifacts와 위키성 지식을 담는 knowledge가 별개로 존재했다. 두 개념은 실제로는 거의 같은 것 — 작업 중 만들어지는 콘텐츠(결정·노트·근거·참조·산출물) — 를 다루는데, 모델이 둘로 갈라져 있어 콘텐츠를 어디에 둘지 모호하고, 검색·임베딩·버전 관리 같은 공통 기능을 두 곳에 중복 구현해야 했다. 또한 실행 계약(envelope)의 조건 판정은 "특정 artifact가 존재하는가"를 옛 artifact 이름으로 검사하고 있었다.

## 결정 (Decision)

artifacts를 knowledge로 통합한다. 살아 있는 콘텐츠(작업용·지식용)를 knowledge로 옮기고, 옛 이름으로 접근하던 코드의 유예를 위해 한동안 읽기 전용 호환 경로(별칭 뷰)를 둔 뒤, 모든 코드가 새 모델을 직접 쓰게 되면 그 별칭을 제거한다. 동시에 실행 계약의 조건 판정도 옛 artifact 기반 술어(존재 조건·필터 키)를 새 knowledge 기반 이름으로 바꿔, 콘텐츠 모델과 계약 판정이 같은 어휘를 쓰게 한다.

## 근거와 결과 (Consequences)

- **단일 콘텐츠 모델**: 지식과 산출물이 한 모델로 합쳐져 "어디에 둘지" 모호함이 사라지고, 검색·임베딩·버전 관리 같은 기능을 한 번만 구현한다. single source of truth를 콘텐츠 모델에 적용한 결과다.
- **계약 판정 일관성**: envelope의 조건 판정이 새 이름 체계로 정렬되어, 콘텐츠 모델과 계약이 다른 어휘로 어긋나는 일이 없어진다. 옛 이름이 남은 과거 데이터는 마이그레이션이 제자리에서 다시 써, 미인식 술어로 조용히 실패하는 일을 막는다.
- **단계적 전환**: 호환 별칭 뷰를 유예 기간 동안 두어 옛 이름 접근 코드가 깨지지 않게 한 뒤 제거하는, 점진적·비파괴적 전환을 택했다. 살아남는 콘텐츠 행은 통합 과정에서 보존된다.

provenance: artifacts → knowledge 테이블 통합·읽기 전용 별칭 뷰 도입 — `daemon/migrations/024_artifacts_to_knowledge.sql`. 별칭 뷰 제거와 envelope 조건 판정의 artifact_exists/artifact_type → knowledge_exists/knowledge_type 재작성 — `daemon/migrations/026_envelope_predicate_to_knowledge.sql`.
