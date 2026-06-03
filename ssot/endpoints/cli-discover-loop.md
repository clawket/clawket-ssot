---
id: endpoint.cli-discover-loop
kind: Endpoint
title: clawket discover-loop (start/next-round/dispatch-plan/verify-tsv/batch-id/sync/status/converged/rounds)
definition: "PDD 검증 루프를 자동화하는 CLI 명령군으로, 라운드 플랜·사이클·유닛을 한 번에 만들고 배치 디스패치 매니페스트를 뽑고 TSV 근거를 검증·전사하며 3-way 수렴 여부를 판정·조회한다."
realizedBy: [component.cli, component.daemon]
implementedIn:
  - cli/src/main.rs
relatesTo:
  - { to: concept.scenario, type: reads, note: "유닛에 첨부된 시나리오 지식 수를 읽어 배치 매니페스트를 만든다" }
  - { to: concept.tsv-evidence, type: reads, note: "verify-tsv가 6필드 TSV 근거 행의 형식을 검증한다" }
  - { to: concept.task, type: mutates, note: "sync가 TSV 행을 태스크로 전사·갱신한다" }
  - { to: concept.convergence, type: reads, note: "status/converged가 라운드별 결함·시나리오오류·통과 수를 집계해 수렴을 판정한다" }
governedBy:
  - invariant.batch-size-30
  - invariant.sync-purity
  - invariant.convergence-two-rounds
  - invariant.evidence-on-tsv
impacts:
  - domain.pdd-verification
consumedBy: []
servesPersona: [persona.llm-coding-agent, persona.qa-reasoner-subagent]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
`clawket discover-loop`(별칭 `dl`)은 시나리오 기반 개발(PDD)의 한 검증 라운드를 처음부터 끝까지 굴리는 데 필요한 반복 작업을 명령 하나로 묶은 자동화 도구다. 검증은 "라운드를 연다 → 검증할 시나리오를 배치로 나눠 서브에이전트에 분배한다 → 각 행의 판정 근거(TSV)를 모은다 → 그 근거를 태스크로 옮긴다 → 결함과 시나리오 오류가 충분히 줄었는지 본다"는 흐름을 라운드마다 반복한다. 이 명령은 그 흐름의 각 단계를 사람이 손으로 플랜·사이클·유닛을 만들지 않아도 되게 자동화한다. 주로 LLM 에이전트가 검증 루프를 운영할 때 사용한다.

동작은 네 묶음으로 나뉜다.
- 라운드 개설: 라운드 플랜을 만들어 곧장 활성화하고, 활성 사이클과 영역별 QA 유닛까지 한 번에 생성한다. 이전 라운드 플랜으로부터 도메인·영역을 물려받아 "다음 라운드"를 자동으로 열 수도 있다.
- 분배·형식검증: 플랜의 유닛들에 대해 배치 디스패치 매니페스트를 뽑고(각 배치에 고유 식별자 부여), TSV 근거 파일이 정해진 형식을 지키는지 검사하며, 근거 행에 붙일 새 배치 식별자를 발급한다.
- 전사(sync): 검증으로 얻은 TSV 근거 행을 태스크로 옮겨 적는다.
- 수렴 조회: 활성 라운드의 결함·시나리오오류·통과 수를 집계해 직전 라운드 대비 회귀를 감지하고, 루프를 끝내도 되는 수렴 조건을 충족했는지 판정하며, 라운드별 태스크 수 추이를 나열한다.

## 요청 / 응답
- 라운드 개설은 대상 프로젝트, 도메인 이름, 라운드 번호, 그리고 검증을 나눌 영역 이름들을 받는다. "다음 라운드"는 직전 라운드 플랜을 가리키면 도메인·영역·번호를 알아서 추론한다(필요 시 개별 항목을 직접 덮어쓸 수 있다). 응답은 새로 만들어진 라운드 플랜·사이클·유닛 묶음이다.
- 디스패치 매니페스트는 대상 플랜과 한 배치에 담을 시나리오 최대치를 받아, 유닛별로 어떤 배치를 어떻게 던질지와 각 배치 식별자를 돌려준다. 한 유닛이 배치 상한을 넘으면 경고한다.
- TSV 형식검증은 근거 파일 경로를 받아 형식 적합 여부만 판정한다 — 데이터베이스에 아무것도 쓰지 않는 순수 검사다.
- 전사는 근거 파일 경로와 옮겨 담을 대상 유닛·사이클(활성이어야 함)을 받아, 행을 태스크로 만들거나 갱신한다. 응답은 전사 결과 요약이다.
- 수렴 조회는 플랜(또는 프로젝트의 활성 플랜)을 받아 현재 라운드의 통과/결함/시나리오오류 수와 회귀 여부, 수렴 충족 여부, 라운드별 추이를 돌려준다.

## 권한 / 제약
- 한 배치에 담을 수 있는 시나리오 수에는 상한이 있다(기본 30) — 이를 넘으면 경고한다(invariant.batch-size-30).
- 전사(sync)는 "추론하지 않는 순수 옮겨적기"다. 판정 결과(통과/결함/시나리오오류)를 태스크 상태에 1:1로 대응시켜 기록할 뿐, sync 안에서 새로 판단하지 않는다. 같은 시나리오·같은 사이클 조합은 중복 생성하지 않고 기존 태스크를 갱신한다(invariant.sync-purity).
- TSV 근거 행은 정해진 필드 구성을 갖춰야 하며, 근거(어느 파일 어느 위치인지)가 비어 있으면 형식검증에서 걸린다(invariant.evidence-on-tsv).
- 수렴은 "결함=0이고 시나리오오류=0"이 두 라운드 연속일 때만 인정된다. 수렴 조회 명령은 충족 시 정상 종료, 미충족 시 비정상 종료로 신호를 보내 스크립트가 루프 종료를 판단할 수 있게 한다(invariant.convergence-two-rounds).
- 라운드 개설로 만든 플랜은 곧바로 활성화되고, 사이클은 첫 유닛에 묶이되 유닛 간 태스크를 허용한다.

## provenance
CLI(`component.cli`)가 명령 표면을 제공하고, 실제 라운드 생성·매니페스트·전사·집계는 로컬 데몬(`component.daemon`)의 discover-loop 라우트가 수행한다. 명령·서브커맨드 정의와 인자, 상태 매핑(통과→done, 결함→blocked, 시나리오오류→cancelled)은 `cli/src/main.rs`의 DiscoverAction 열거형과 그 핸들러에 있다. 검증 루프 전체 절차의 의미는 검증 루프 스킬 문서에 정의되어 있다.

## 미확정 (OPEN)
- [ ] OPEN: status의 회귀 감지 기준(직전 라운드 대비 어떤 증가를 회귀로 보는지)과 rounds 추이의 정확한 집계 단위는 데몬 측 discover-loop 라우트를 정독해 확정 필요.
