---
id: concept.compute-tier
kind: Concept
title: 연산 등급(Compute Tier · low/med/high)
definition: "태스크별 연산 등급. low · med · high 세 단계(G1/G2/G3로도 불림)이며, 환경변수 CLAWKET_TIER로 서브에이전트에 전파되고, 등급 분포·에스컬레이션율 진단의 기준이 된다."
relatesTo:
  - { to: concept.tier, type: mirrors, note: "인벤토리의 concept.tier(모델 등급)와 같은 등급 축을 CLI 전파·진단 관점에서 본 것." }
  - { to: concept.task, type: belongs-to, note: "각 태스크에 부여되는 연산 등급 속성." }
  - { to: endpoint.cli-doctor, type: reads, note: "doctor가 등급 분포·에스컬레이션율을 이 등급으로 집계·진단한다." }
governedBy: []
implementedIn:
  - cli/src/main.rs
  - cli/src/doctor.rs
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:concept"]
---

## 정의
연산 등급은 한 태스크가 어느 정도의 연산(모델 능력)을 필요로 하는지를 나타내는 세 단계 라벨이다.

- **low** (별칭 G1, 가장 낮은 등급) — 가벼운 작업.
- **med** (별칭 G2)
- **high** (별칭 G3) — 가장 무거운 작업.

등급은 두 가지 흐름에서 작동한다.

1. **전파**: 전역 기본 등급을 지정하면 하위 명령과 플러그인 훅을 통해 스폰되는 서브에이전트에게 환경변수(CLAWKET_TIER)로 전달된다. 태스크 단위 등급 옵션은 그 명령에 한해 전역값을 덮어쓴다.
2. **진단**: 작업 도구의 점검 명령이 이 등급을 기준으로 "등급 분포(tier_distribution)"와 "에스컬레이션율(escalation_rate)"을 집계한다. 여기서 에스컬레이션은 "가장 낮은 등급(low/none/G1)이 아닌 태스크"의 비율을 뜻하며, 그 비율이 30%를 넘으면 경고를 띄워 상위 등급 배정 정책을 점검하라고 안내한다.

요컨대 연산 등급은 "이 태스크에 얼마나 센 모델을 쓸 것인가"를 태스크에 박아두고, 그 분포가 적정한지를 사후에 진단 가능하게 하는 축이다.

## 엔티티 (DB)
태스크는 tier 속성을 가지며 값이 비어 있으면(none) 가장 낮은 등급으로 간주된다. 진단 시 태스크의 tier 값과(있다면) escalation_reason을 읽어 분포·에스컬레이션율을 계산한다.

## API 표면
- 전역 등급 플래그: low | med | high를 받아 하위 명령과 서브에이전트(CLAWKET_TIER)로 전파.
- 태스크 생성/갱신의 등급 옵션: 해당 명령에 한해 전역값을 덮어쓴다.
- 점검 명령(doctor): 등급 분포·에스컬레이션율 섹션에서 등급별 집계와 30% 경고 임계를 출력한다.

## 불변식
- (등급 자체에 강한 데이터 불변식은 확인되지 않음 — 진단상 임계값은 정책적 권고(>30% 경고)이지 강제 제약이 아니다.)

## 구현 위치 (provenance)
- CLI의 전역 tier 플래그가 low/med/high를 받아 하위 명령·서브에이전트에 CLAWKET_TIER로 전파되며, 태스크 생성 시의 per-task tier 옵션이 전역값을 덮어쓴다고 선언되어 있다(`cli/src/main.rs`의 전역 tier·태스크 tier 옵션 부근).
- doctor의 등급 분포·에스컬레이션율 점검에서 low/haiku/none/g1을 최저 등급(비에스컬레이션)으로 묶고 그 외를 에스컬레이션으로 집계하며 30% 초과 시 경고한다(`cli/src/doctor.rs`의 tier 분포·에스컬레이션율 함수).

## 미확정 (OPEN)
- [ ] OPEN: 등급→실제 모델(Haiku/Sonnet/Opus) 매핑과 다운그레이드 처리(현재 advisory인지)는 모델 티어 라우팅 통합 노드(integration.model-tier-routing)와 교차 확인 필요.
