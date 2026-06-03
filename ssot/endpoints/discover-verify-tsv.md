---
id: endpoint.discover-verify-tsv
kind: Endpoint
title: TSV 근거 스키마 검증
definition: POST /discover-loop/verify-tsv — 서브에이전트가 산출한 TSV 근거가 정해진 컬럼 스키마·필수 필드·값 형식을 지키는지 검사해 행별 오류 목록을 돌려준다. DB는 건드리지 않는다.
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/discover.rs]
relatesTo:
  - { to: concept.tsv-evidence, type: evaluates, note: "TSV 근거의 컬럼 수·필수 필드·값 형식을 검증한다" }
  - { to: concept.scenario, type: evaluates, note: "시나리오 ID가 US-<도메인>-<번호> 패턴을 따르는지 확인한다" }
  - { to: concept.tier, type: evaluates, note: "tier_used 값이 haiku|sonnet|opus인지, opus면 에스컬레이션 사유가 있는지 확인한다" }
  - { to: domain.pdd-verification, type: realizes, note: "동기화 전 근거 무결성 게이트를 구현한다" }
governedBy: []
impacts: [concept.tsv-evidence]
consumedBy: [endpoint.cli-discover-loop]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

QA 추론 서브에이전트가 한 배치를 검증하고 내놓은 근거 표(TSV)가 약속된 형식을 지키는지 미리 검사해 주는 명령이다. 근거를 DB에 동기화하기 전, 형식이 깨진 행이 섞여 들어가는 것을 막는 게이트 역할을 한다. 행마다 시나리오 ID·판정·근거 추론·증거 위치·사용 모델 등급·배치 ID(그리고 신형은 에스컬레이션 사유)가 제대로 들어 있는지 확인하고, 어긋난 행과 어긋난 필드, 무엇이 잘못됐는지를 모아 돌려준다. 데이터를 쓰지 않는 순수 검증이며, 동기화 엔드포인트도 내부적으로 같은 검사를 먼저 돌린다.

## 요청 / 응답

보내는 정보: 검사할 TSV 본문(원문 그대로). 첫 줄이 시나리오 ID 형식이 아니면 헤더로 보고 건너뛴다.

돌려주는 정보:
- 전체 유효 여부
- 데이터 행 수, 오류 수
- 오류 목록 — 각 항목은 몇 번째 행의 어느 필드가 왜 잘못됐는지를 사람이 읽을 수 있는 메시지로 담는다.

각 행이 담아야 하는 정보의 의미:
- 시나리오 ID — US로 시작하는 정해진 식별자 패턴.
- 판정 — 합격(pass)·결함(defect)·시나리오오류(scenario_error) 셋 중 하나.
- 근거 추론 — 왜 그렇게 판정했는지. 비어 있으면 안 된다.
- 증거 위치 — 추론한 코드 위치(파일:줄) 등. 합격 행이라도 반드시 있어야 한다(나중에 같은 증거로 회귀 비교를 하기 위함).
- 사용 모델 등급 — haiku·sonnet·opus 중 하나(또는 비움).
- 배치 ID — BATCH로 시작하는 배치 식별자. 반드시 있어야 한다.
- (신형) 에스컬레이션 사유 — 최상위 등급(opus)을 썼다면 왜 등급을 올렸는지 사유가 있어야 한다.

## 권한 / 제약

- 컬럼 수는 6개(구형) 또는 7개(신형)만 허용. 구형 6컬럼은 호환을 위해 받아 주되 에스컬레이션 사유를 빈 값으로 간주한다 — 앞으로는 7컬럼을 권장한다.
- 시나리오 ID는 정해진 패턴을 어기면 오류.
- 판정 값이 셋(합격/결함/시나리오오류) 밖이면 오류.
- 근거 추론이 비면 오류(빈 근거는 안티패턴).
- 증거 위치는 어떤 판정이든 비면 오류(과거: 결함·시나리오오류일 때만 → 현재: 합격 포함 항상 필수).
- 모델 등급이 정해진 셋 밖이면 오류.
- 배치 ID가 없거나 형식을 어기면 오류.
- 최상위 등급을 썼는데 에스컬레이션 사유가 비면 오류.
- 읽기 전용 — DB를 변경하지 않는다.

## provenance

데몬의 `daemon/src/routes/discover.rs`(verify_tsv 핸들러와 공용 검증 함수, POST `/discover-loop/verify-tsv`)가 제공한다. CLI `clawket discover-loop verify-tsv` 가 사용자 표면이다. 같은 검증 로직을 동기화 엔드포인트가 동기화 직전에 재사용하므로, 두 경로의 합격 기준은 항상 동일하다.

## 미확정 (OPEN)
- [ ] OPEN: 구형 6컬럼 허용을 언제까지 유지할지(폐기 시점)와, 검증이 내보낸다고 코드 주석에 적힌 schema_version 경고가 실제 응답에서 어떻게 전달되는지 확인 필요.
