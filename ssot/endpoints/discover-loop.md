---
id: endpoint.discover-loop
kind: Endpoint
title: clawket discover-loop (CLI 명령 정의)
definition: 검증 루프 자동화 명령군 — 라운드 플랜·사이클·유닛 자동 생성, 배치 디스패치 매니페스트, TSV 근거 검증·전사, 3-way 수렴·라운드 조회를 한 명령 아래 모은다.
realizedBy: [component.cli]
implementedIn:
  - cli/src/main.rs
relatesTo:
  - { to: concept.scenario, type: reads, note: "유닛에 매달린 시나리오 수를 읽어 배치 매니페스트를 만든다" }
  - { to: concept.tsv-evidence, type: reads, note: "7필드 TSV 근거 파일을 검증하고 태스크로 전사한다" }
  - { to: concept.convergence, type: reads, note: "결함=0·시나리오오류=0의 2라운드 연속 조건을 판정한다" }
  - { to: concept.plan, type: mutates, note: "라운드 플랜을 draft→active로 자동 생성한다" }
  - { to: concept.cycle, type: mutates, note: "라운드의 active 사이클을 자동 생성한다" }
  - { to: concept.unit, type: mutates, note: "QA 유닛(parallel 모드)을 자동 생성한다" }
  - { to: concept.task, type: mutates, note: "TSV 행을 태스크로 전사(생성·갱신)한다" }
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

PDD 검증·수렴 루프(domain.pdd-verification)를 사람이 손으로 챙기지 않아도 한 라운드가 끝까지 굴러가도록 자동화한 명령군이다. 검증 루프 스킬이 라운드를 진행할 때 이 명령을 호출해 라운드용 작업 구조를 깔고, 서브에이전트에게 배포할 작업 묶음을 산출하고, 서브에이전트가 만들어 온 근거 표를 검증·전사하고, 라운드 결과가 수렴했는지 판정한다. 별칭은 dl 이다.

## 요청 / 응답

여러 하위 동작으로 갈린다.

- 라운드 시작·다음 라운드: 프로젝트·도메인·라운드 번호·QA 영역 목록을 받아 "<도메인> Round <N>" 플랜 1개를 draft에서 active로 만들고, 영역마다 병렬 모드 QA 유닛을 만들고, 첫 유닛에 닻을 내린 active 사이클 1개를 만든다. 다음 라운드 동작은 직전 플랜에서 도메인·영역을 추론해 라운드 번호를 +1 해 같은 구조를 다시 깐다.
- 배치 매니페스트: 플랜의 유닛별 시나리오 수를 읽어 배치 식별자(BATCH-<ULID>)를 붙인 디스패치 명세를 출력하고, 배치 상한(기본 30)을 넘는 유닛이 있으면 경고한다.
- TSV 검증·배치ID 생성: 근거 TSV 파일을 정해진 필드 스키마(시나리오ID·판정·근거설명·근거위치·사용티어·배치ID)에 맞는지만 검사하고 DB는 건드리지 않는다. 배치ID 생성 동작은 근거 행에 붙일 새 배치 식별자만 발급한다.
- 전사(sync): 검증된 TSV 행을 태스크로 옮긴다. 판정→상태 매핑은 pass→done, defect→blocked, scenario_error→cancelled 이며, 같은 시나리오ID+사이클 조합의 기존 태스크는 중복 생성하지 않고 갱신한다(멱등).
- 수렴 조회: 활성 라운드의 결함·시나리오오류·통과 건수를 직전 라운드와 비교해 회귀를 탐지하고, 결함=0·시나리오오류=0이 2라운드 연속이면 수렴으로 판정한다. 라운드 목록 동작은 프로젝트의 Round-N 플랜들을 라운드별 태스크 수와 함께 나열해 단조감소 수렴 그래프를 그릴 수 있게 한다.

## 권한 / 제약

데몬이 살아 있어야 동작한다. 전사 단계는 "추론 없는 순수 전사"를 원칙으로 하여 판정 자체는 검증 라운드(서브에이전트)에서 끝내고 sync는 매핑·기록만 한다 — 이 분리가 멱등성을 지키는 핵심이다. 전사 대상 사이클은 active 여야 한다(작업 관리 불변식상 active 사이클 밖 태스크는 시작 불가). TSV 검증은 DB를 바꾸지 않는 순수 검사이며, 수렴 판정 동작은 미수렴 시 비정상 종료(exit 1)로 신호해 스킬 루프가 다음 라운드로 넘어가게 한다.

## provenance

명령군 정의는 cli/src/main.rs 의 DiscoverAction 열거(1384-1500행)에 있고, 명령 진입 정의는 같은 파일 342-352행이다. 검증 루프 스킬(skills/clawket-verify-loop/SKILL.md)이 이 명령군의 주된 소비자이며, 사람이 읽는 별칭 정리는 endpoint.cli-discover-loop 에 별도로 있다.

## 미확정 (OPEN)
- [ ] OPEN: 배치 상한(기본 30) 초과 시 경고에 그치는지 분할을 강제하는지, 전사 단계의 assignee 기본값(discover-loop) 외 라벨 정책을 데몬 라우트 정독으로 확인.
