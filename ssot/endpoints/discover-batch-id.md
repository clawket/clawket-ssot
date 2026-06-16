---
id: endpoint.discover-batch-id
kind: Endpoint
title: 배치 ID 발급
definition: POST /discover-loop/batch-id — TSV 근거 행에 박아 넣을 새 배치 식별자(BATCH-<ULID>)를 한 개 발급해 돌려준다.
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/discover.rs]
relatesTo:
  - { to: concept.tsv-evidence, type: mutates, note: "TSV 근거 행의 배치 열에 박을 새 배치 식별자를 생성한다" }
  - { to: domain.pdd-verification, type: realizes, note: "서브에이전트 배치 식별 부여 프리미티브를 구현한다" }
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

검증 라운드에서 서브에이전트 배치를 식별할 새 배치 ID를 하나 즉석 발급해 주는 명령이다. 발급된 ID는 그 배치가 산출하는 근거 표(TSV)의 배치 열에 박혀, 나중에 어느 배치가 어떤 행을 냈는지 추적하고 배치 단위 통계(예: 한 배치 안에서 앞쪽·뒤쪽 절반의 결함 비율 비교)를 낼 수 있게 한다. 디스패치 매니페스트 조회가 배치 ID를 미리 묶음으로 발급해 주지만, 추가로 한 개가 더 필요할 때 이 명령으로 단발 발급한다.

## 요청 / 응답

보내는 정보: 없음.

돌려주는 정보: 새로 만든 배치 식별자 하나(BATCH로 시작하고 뒤에 ULID가 붙는 형식).

## 권한 / 제약

- 입력 없이 호출하면 매번 새롭고 고유한 식별자를 만들어 준다.
- 발급 형식은 TSV 근거 검증 엔드포인트가 받아들이는 배치 ID 형식과 동일해야 한다 — 그래야 발급한 ID가 검증을 통과한다.

## provenance

데몬의 `daemon/src/routes/discover.rs`(generate_batch_id 핸들러, POST `/discover-loop/batch-id`)가 제공한다. CLI `clawket discover-loop batch-id` 가 사용자 표면이다. 식별자 형식(BATCH 접두 + ULID)은 디스패치 매니페스트가 묶음 발급하는 형식, 그리고 TSV 검증이 요구하는 형식과 한 벌로 묶여 있다.

## 미확정 (OPEN)
- [ ] OPEN: 발급된 배치 ID가 어딘가에 등록·예약되는지(중복 방지·미사용 회수 정책 유무) 아니면 순수 stateless 생성인지 확인 필요.
