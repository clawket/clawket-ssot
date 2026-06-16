---
id: endpoint.discover-converged
kind: Endpoint
title: 데몬 GET /discover-loop/converged (2라운드 연속 수렴 최종 판정)
definition: 현재 라운드와 직전 라운드가 모두 결함=0·시나리오오류=0인지를 따져 검증 루프를 끝내도 되는지(수렴 완료)를 최종 판정하는 조회 엔드포인트다.
realizedBy: [component.daemon]
implementedIn:
  - daemon/src/routes/discover.rs
relatesTo:
  - { to: concept.convergence, type: reads, note: "마지막 2라운드 연속 0 규칙으로 루프 종료 가능 여부를 판정한다" }
  - { to: concept.plan, type: reads, note: "현재·직전 라운드 플랜을 찾아 라운드 번호를 추론한다" }
  - { to: concept.task, type: reads, note: "각 라운드 플랜 산하 태스크의 판정 건수를 집계한다" }
  - { to: concept.discover-round, type: reads, note: "연속 두 라운드를 비교 단위로 삼는다" }
  - { to: concept.plan, type: mutates, note: "수렴 시 플랜 본문에 다음 단계(수동 QA 활성화) 안내 한 줄을 덧붙인다" }
  - { to: endpoint.discover-status, type: complements, note: "status는 한 라운드 스냅샷, 이 엔드포인트는 2라운드 연속 최종 판정을 맡는다" }
governedBy:
  - invariant.convergence-two-rounds
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

검증 루프를 멈춰도 되는지를 결정하는 최종 신호다. 한 라운드만 깨끗한 것으로는 부족하고, "마지막 2라운드가 연속으로 결함 0·시나리오오류 0"이어야 수렴으로 본다(우연한 0이 아님을 확인하는 안전 마진). 검증 루프 스킬이 매 라운드 끝에 이 엔드포인트를 물어 루프를 계속할지 다음 단계(수동 QA)로 넘어갈지 판단한다.

## 요청 / 응답

요청은 프로젝트 ID 또는 플랜 ID로 대상 라운드를 지정한다.

응답은 수렴 여부(참/거짓)와 그 사유를 사람이 읽는 문장으로 돌려주고, 현재 라운드 번호와 직전 라운드 번호도 함께 담는다. 수렴은 현재 라운드와 직전 라운드가 둘 다 결함 0·시나리오오류 0일 때만 참이 된다. 1라운드뿐이거나 현재 라운드가 아직 0이 아니거나 직전 라운드가 0이 아니면 거짓이며, 사유 문장이 그중 무엇 때문인지 알려준다(예: "현재 라운드 결함 N", "1라운드만 완료 — 연속 0 라운드 2개 필요", "직전 라운드가 0이 아님").

수렴이 확정되면 그 라운드 플랜 본문에 "다음: <도메인> 수동 QA 플랜 활성화" 한 줄을 덧붙여, 나중에 다른 세션이나 검토자가 다시 추론하지 않고도 다음 단계가 무엇인지 알 수 있게 한다(이미 같은 줄이 있으면 중복 추가하지 않는다).

## 권한 / 제약

데몬이 살아 있어야 한다. 수렴 판정의 핵심 규칙은 "연속 두 라운드 모두 0"이며, 이는 우연한 단일 라운드 통과로 검증을 조기 종료하는 것을 막기 위한 것이다. 항상 200으로 응답하고 수렴 여부는 본문의 불리언으로 전달한다(미수렴이 오류가 아니라 정상적인 "아직 아님" 상태이기 때문).

## provenance

라우트와 핸들러는 daemon/src/routes/discover.rs(/discover-loop/converged, converged 핸들러)에 있다. 연속 두 라운드 규칙은 invariant.convergence-two-rounds, 한 라운드 스냅샷 조회는 endpoint.discover-status, 수렴 개념은 concept.convergence 에 정리돼 있다.

## 미확정 (OPEN)
- [ ] OPEN: 수렴 시 플랜 본문에 덧붙이는 "수동 QA" 안내가 실제 후속 플랜 자동 생성으로 이어지는지 안내 문구에 그치는지 owner 확인.
