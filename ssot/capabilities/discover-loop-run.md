---
id: capability.discover-loop-run
kind: Capability
title: 검증 루프 자동화(라운드 생성·디스패치·동기화·수렴 판정)
purpose: "검증 라운드 운영을 자동화한다 — 라운드 플랜·사이클·유닛을 한 번에 생성하고, 배치 디스패치 매니페스트를 만들고, TSV 근거를 스키마 검증·벌크 동기화하고, 3-way 수렴 여부를 판정한다."
servesPersona:
  - persona.llm-coding-agent
  - persona.qa-reasoner-subagent
  - persona.human-operator
realizedBy:
  - component.cli
  - component.daemon
implementedIn:
  - cli/src/main.rs
relatesTo:
  - { to: endpoint.cli-discover-loop, type: realizes, note: "discover-loop 서브커맨드(start/next-round/dispatch-plan/verify-tsv/batch-id/sync/status/converged/rounds)로 전부 노출된다." }
  - { to: domain.pdd-verification, type: relates-to, note: "PDD 검증·수렴 루프 도메인을 실행 가능한 CLI 자동화로 구현한다." }
  - { to: concept.tsv-evidence, type: depends-on, note: "스키마 검증·벌크 동기화의 입력이 7필드 TSV 근거 파일이다." }
  - { to: concept.convergence, type: relates-to, note: "status/converged 액션이 결함=0·시나리오오류=0의 2라운드 연속 수렴 조건을 판정한다." }
  - { to: concept.scenario, type: depends-on, note: "디스패치 매니페스트는 유닛에 붙은 시나리오 지식 개수를 읽어 배치를 나눈다." }
  - { to: concept.task, type: relates-to, note: "동기화는 TSV 근거 행을 태스크로 전사하며 status를 done/blocked/cancelled로 매핑한다." }
impacts:
  - concept.task
  - concept.convergence
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:capability
---

## 사용자가 할 수 있는 일

PDD 검증 루프의 한 라운드를 손으로 일일이 세팅하지 않고 명령으로 굴린다. 라운드를 시작하면 해당 라운드의 플랜·활성 사이클·QA 유닛이 한 번에 만들어지고, 이전 라운드를 근거로 다음 라운드를 자동으로 이어 만들 수도 있다. 검증을 나눠 돌리기 위한 배치 디스패치 매니페스트를 뽑고, 서브에이전트들이 산출한 TSV 근거가 규격에 맞는지 먼저 검증한 뒤, 그 결과를 한꺼번에 태스크로 전사(동기화)한다. 마지막으로 이번 라운드의 결함·시나리오오류·통과 집계와 직전 라운드 대비 회귀 여부를 보고, 수렴 조건(결함=0·시나리오오류=0이 2라운드 연속)을 충족했는지 자동으로 판정한다.

핵심은 "판단(추론)"과 "기록(전사)"을 분리한다는 점이다. 동기화 단계는 추론을 하지 않고 TSV에 적힌 결과를 그대로 옮겨 적기만 하며, 같은 시나리오·사이클의 태스크는 새로 만들지 않고 갱신해 중복을 막는다.

## 행위

- 라운드 시작: 도메인과 라운드 번호, QA 유닛 영역을 받아 라운드 플랜(draft→active) + 활성 사이클 + 병렬 QA 유닛을 한 번에 만든다.
- 다음 라운드 생성: 이전 플랜에서 도메인·영역을 추론해 라운드 번호를 1 올린 다음 라운드를 자동 생성한다(개별 항목 덮어쓰기 가능).
- 디스패치 매니페스트: 플랜의 유닛별 시나리오 개수를 읽어 배치 식별자를 발급하고, 배치 크기 상한을 넘는 유닛을 경고한다.
- TSV 근거 검증: TSV 파일을 정해진 필드 스키마에 맞춰 순수 검증한다(DB에 쓰지 않음).
- 배치 식별자 발급: 근거 행에 태깅할 새 배치 식별자를 만든다.
- 벌크 동기화: TSV 근거 행을 태스크로 전사한다. 추론 없이 옮겨 적기만 하며 status를 done/blocked/cancelled로 매핑하고, 같은 시나리오·사이클의 태스크는 갱신해 멱등성을 보장한다.
- 수렴 조회·판정: 활성 라운드의 결함/시나리오오류/통과 집계와 직전 라운드 대비 회귀를 보고하고, 2라운드 연속 0 조건 충족 시 수렴으로 판정한다(미수렴이면 비정상 종료).
- 라운드 목록: 프로젝트의 라운드별 플랜과 태스크 개수를 나열해 단조 감소 수렴 그래프를 가늠하게 한다.

## 시스템 흐름

라운드 시작·다음 라운드 생성은 데몬에 플랜·사이클·유닛 생성을 요청해 검증 가능한 골격을 세운다. 디스패치 매니페스트는 유닛에 붙은 시나리오 지식 개수를 읽어 배치로 나누고 식별자를 발급한다. 서브에이전트들이 만든 TSV 근거는 먼저 순수 검증(쓰기 없음)을 통과한 뒤, 동기화 단계에서 추론 없이 태스크로 전사된다 — 이때 status 매핑과 시나리오·사이클 기준 멱등 갱신이 적용된다. 수렴 조회·판정은 라운드 집계와 직전 라운드를 비교해 회귀를 잡고, 2라운드 연속 0 조건으로 루프 종료 여부를 결정한다.

## 어디에 구현되어 있나

CLI(Rust 바이너리)에 discover-loop 서브커맨드 일체가 정의되어 있다. 라운드 자동 생성(A), 디스패치 메타데이터·TSV 스키마 검증(B), 벌크 동기화 전사(C), 3-way 수렴 조회(D)의 네 묶음으로 구성되며, 실제 엔티티 생성·집계는 데몬을 통해 이뤄진다.

## 미확정 (OPEN)
- [ ] OPEN: TSV 스키마 필드 수가 명령 도움말에는 6필드로, 시나리오/근거 개념에는 7필드로 표기되어 있어 동기화 시 실제 파싱 필드 구성 확인 필요(데몬·동기화 로직).
- [ ] OPEN: 회귀 감지·단조 감소 판정의 정확한 비교 규칙 세부는 데몬 측 집계 로직 확인 필요.
