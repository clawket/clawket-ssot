---
id: endpoint.discover-sync
kind: Endpoint
title: 데몬 POST /discover-loop/sync (TSV→DB 일괄 전사)
definition: 검증된 근거 표(TSV)의 한 행 한 행을 클로켓 태스크로 옮겨 적는 일괄 전사 엔드포인트로, 판정→상태 매핑만 수행하고 어떤 추론 결정도 하지 않는다.
realizedBy: [component.daemon]
implementedIn:
  - daemon/src/routes/discover.rs
relatesTo:
  - { to: concept.tsv-evidence, type: reads, note: "전사 전에 6/7필드 TSV 스키마를 다시 검증하고 그 행들을 입력으로 읽는다" }
  - { to: concept.task, type: mutates, note: "행마다 QA-<시나리오ID> 태스크를 생성하거나 같은 시나리오ID+사이클이면 갱신한다" }
  - { to: concept.scenario, type: reads, note: "각 행의 시나리오ID로 태스크를 식별·매칭한다" }
  - { to: concept.knowledge, type: mutates, note: "전사한 TSV를 Round 근거 아티팩트로, 실패 행은 재시도 큐로, 배치별 통계는 주의분산 아티팩트로 남긴다" }
  - { to: endpoint.cli-discover-loop, type: backed-by, note: "CLI discover-loop sync 하위명령이 이 라우트를 호출한다" }
governedBy:
  - invariant.sync-purity
  - invariant.no-destructive-sync
  - invariant.scenario-id-one-to-one
impacts: []
consumedBy:
  - endpoint.cli-discover-loop
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

검증 라운드에서 서브에이전트가 만들어 온 근거 표(TSV)를 클로켓의 작업 상태로 "옮겨 적는" 단계다. 판정은 이미 검증 라운드에서 끝났으므로 이 단계는 그 결과를 기계적으로 기록만 한다 — 무엇이 통과이고 무엇이 결함인지 다시 따지지 않는다. 검증 루프 스킬이 한 라운드의 채점이 끝난 뒤 이 엔드포인트를 호출해 라운드 결과를 작업 보드에 반영한다.

## 요청 / 응답

요청은 근거 표 본문(TSV)과 그 행들을 담을 대상 유닛·사이클, 그리고 만들어질 태스크에 붙일 담당자 라벨(기본값 "discover-loop")로 이뤄진다.

전사 규칙은 판정값을 작업 상태로 1:1 매핑하는 것이다 — 통과는 완료, 결함은 차단, 시나리오오류는 취소로 옮긴다. 같은 시나리오ID가 같은 사이클에 이미 태스크로 존재하면 새로 만들지 않고 그 자리에서 갱신한다(멱등). 시나리오오류 행은 그 행의 근거 설명을 "시나리오 수정 제안"으로도 함께 보존해 뒤이은 시나리오 정제 단계가 다시 파싱하지 않고 읽을 수 있게 한다.

응답은 전사된 행 수, 건너뛴 행 수, 그리고 행별 실패 내역(어느 행의 어느 시나리오가 왜 실패했는지)을 돌려준다. 일부 행이 실패하면 다음 전사에서 재개할 수 있도록 실패한 시나리오ID 목록을 별도 재시도 큐 지식으로 남긴다. 전사가 끝나면 이번에 받은 표 전체를 "Round N 근거" 지식으로 보존하고, 배치별로 앞쪽 절반과 뒤쪽 절반의 결함 비율을 비교한 "주의 분산" 통계 지식도 함께 남겨, 한 배치 안에서 뒤로 갈수록 판정 품질이 떨어졌는지(다시 배포가 필요한지)를 스킬이 판단할 단서를 제공한다.

## 권한 / 제약

데몬이 살아 있어야 동작하며, 전사 대상 사이클은 활성 상태여야 한다(작업 관리 규칙상 활성 사이클 밖 태스크는 시작될 수 없다). 가장 중요한 제약은 "전사 순수성"이다 — 이 핸들러는 어떤 판정·재추론도 하지 않고 오직 매핑·기록만 한다. 전사 전에 표 스키마를 다시 검증해 형식이 틀리면 DB를 건드리기 전에 거절한다(잘못된 입력으로 작업 보드가 오염되는 것을 막는다). 또한 행 삭제·테이블 비우기 같은 파괴적 동작은 설계상 절대 하지 않으며, 취소된 작업도 상태만 바꿔 히스토리를 보존한다.

## provenance

라우트 정의와 핸들러는 daemon/src/routes/discover.rs(/discover-loop/sync, bulk_sync 핸들러)에 있다. 사람이 읽는 명령 별칭과 호출 경로는 endpoint.cli-discover-loop, 한 라운드를 굴리는 상위 흐름은 capability.run-verification-loop·domain.pdd-verification 에 정리돼 있다. 전사 순수성·비파괴·시나리오 1:1 원칙은 각각 invariant.sync-purity, invariant.no-destructive-sync, invariant.scenario-id-one-to-one 으로 못 박혀 있다.

## 미확정 (OPEN)
- [ ] OPEN: 주의분산 의심 판정 임계값(뒤쪽 결함비율이 앞쪽의 1.5배 이상 + 절대 결함 2건 이상)이 정책으로 고정인지 조정 가능한지, 재시도 큐 지식의 후속 소비 흐름이 자동인지 수동인지 owner 확인.
