---
id: concept.batch
kind: Concept
title: 배치(Batch · BATCH-<ULID>)
definition: "한 번의 서브에이전트 디스패치로 묶인 태스크 묶음. BATCH-<26자 ULID> 형식의 식별자로 표시되며, qa_status 히스토그램 집계와 큰 배치 후반부의 주의력 분산(attention dispersion) 진단의 단위가 된다."
relatesTo:
  - { to: concept.batch-id, type: mirrors, note: "인벤토리의 concept.batch-id와 같은 개념(배치 식별자 축)을 묶음 관점에서 본 것." }
  - { to: concept.task, type: reads, note: "배치는 동일 디스패치로 생성된 태스크들의 그룹이다." }
  - { to: concept.tsv-evidence, type: reads, note: "검증 라운드의 TSV 근거에 batch_id 필드가 들어가 배치별로 누적된다." }
governedBy:
  - invariant.batch-size-30
implementedIn:
  - cli/src/main.rs
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:concept"]
---

## 정의
배치는 "한 번에 함께 내보낸 태스크 묶음"이다. 서브에이전트를 한 차례 디스패치할 때 생성되는 태스크들이 하나의 배치 식별자(BATCH-로 시작하는 26자 ULID)를 공유한다.

배치의 목적은 묶음 단위의 품질 추적이다.

- **품질 히스토그램**: 한 배치에 속한 태스크들의 검증 결과(qa_status)를 합산해 통과(pass)·결함(defect)·시나리오오류(scenario_error) 분포를 본다.
- **주의력 분산 탐지**: 큰 배치의 후반부 태스크에서 품질이 떨어지는(주의력이 흩어지는) 패턴을 발견하는 데 쓰인다. 이것이 배치 크기에 상한을 두는 이유이기도 하다.

검증 흐름에서 배치 식별자는 TSV 근거의 한 필드로도 기록되어, 어느 디스패치에서 나온 판정인지를 추적할 수 있게 한다.

## 엔티티 (DB)
배치는 태스크에 부여되는 batch_id 값으로 표현된다(형식: BATCH-<26자 Crockford base32 ULID>). 같은 디스패치의 태스크들이 같은 batch_id를 공유하며, 이 값으로 묶어 qa_status를 집계한다.

## API 표면
- 태스크 생성 시 batch_id 옵션으로 동일 서브에이전트 디스패치 묶음을 지정한다.
- 배치 통계 조회: 특정 batch_id에 대해 `{batch_id, total, pass, defect, scenario_error}` 형태의 qa_status 히스토그램을 반환한다(큰 배치 후반부 주의력 분산 탐지용).

## 불변식
- 한 배치의 크기는 상한(30)을 넘지 않도록 강제된다(invariant.batch-size-30). 큰 배치 후반부 주의력 분산을 막기 위한 제약이다.

## 구현 위치 (provenance)
- CLI 태스크 생성 옵션에 batch_id가 "같은 서브에이전트 디스패치로 묶인 태스크 그룹, 형식 BATCH-<26자 ULID>"로 선언되어 있다(`cli/src/main.rs`의 batch_id 옵션 부근).
- 배치 통계(Stats) 명령이 batch_id별 qa_status 히스토그램(`{batch_id, total, pass, defect, scenario_error}`)을 반환하며 큰 배치 후반부 주의력 분산 탐지 용도임이 기록되어 있다(`cli/src/main.rs`의 Stats 정의 부근).

## 미확정 (OPEN)
- [ ] OPEN: 배치 식별자 생성 주체(훅 디스패치 자동 생성인지, 호출자 지정인지)의 정확한 책임 경계는 훅 어댑터 교차 확인 필요.
