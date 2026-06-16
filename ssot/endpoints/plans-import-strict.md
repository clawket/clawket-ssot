---
id: endpoint.plans-import-strict
kind: Endpoint
title: 엄격 검증 플랜 가져오기 (POST /plans/import/strict)
definition: 인라인으로 받은 플랜 본문을 엄격한 형식 규칙으로 검증한 뒤 플랜 구조로 등록하는 데몬 엔드포인트.
realizedBy: [component.daemon]
implementedIn:
  - daemon/src/routes/import_export.rs
relatesTo:
  - { to: concept.plan, type: mutates, note: "검증을 통과한 본문을 플랜으로 등록한다" }
  - { to: concept.unit, type: mutates, note: "플랜에 속한 유닛도 함께 등록한다" }
  - { to: concept.task, type: mutates, note: "유닛 아래 태스크까지 만든다" }
  - { to: endpoint.daemon-plans-import-strict, type: mirrors, note: "동일 경로(POST /plans/import/strict)에 대한 또 다른 노드 — 훅 관점 기술" }
  - { to: endpoint.hook-plan-sync, type: triggers, note: "Plan Mode 종료 훅이 이 경로로 본문을 검증·등록한다" }
governedBy: [invariant.db-is-sot-plan-markdown-view]
impacts: [concept.plan, concept.unit, concept.task]
consumedBy: [endpoint.hook-plan-sync]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

플랜 본문(마크다운)을 파일이 아니라 그 자리에서(인라인) 받아, 정해진 엄격한 형식 규칙에 맞는지 먼저 검사한 다음 통과하면 플랜·유닛·태스크 구조로 등록하는 입구다. 일반 가져오기가 디스크의 파일 경로를 읽는 것과 달리, 이쪽은 본문 자체를 통째로 전달받는다. 주된 사용처는 Claude Code의 Plan Mode를 빠져나갈 때다 — 그 출력(계획 마크다운)을 Clawket 플랜으로 곧장 받아들이기 전에 형식이 규격에 맞는지 게이트로 걸러야 하기 때문이다. 파일 시스템을 매번 거치지 않고 인라인으로 처리하려는 의도로 일반 경로와 분리되어 있다.

## 요청 / 응답

보내는 정보는 검증·등록할 플랜 본문 텍스트와, 어느 프로젝트에 들일지(프로젝트 이름 또는 현재 작업 디렉터리로 식별), 그리고 실제 등록 없이 시험만 하는 미리보기(dry-run) 여부다. 본문이 비어 있으면 들어올 수 없다.

본문이 형식 규칙을 어기면 단순 거절이 아니라, 어디가 왜 잘못됐는지 짚어 주는 구조화된 오류 — 문제가 난 줄·열 위치, 위반 유형, 그리고 바로잡을 힌트 — 를 돌려준다. 이는 호출한 쪽(특히 Plan Mode 종료 훅)이 사람에게 친절한 안내 메시지를 만들 수 있게 하기 위함이다. 통과하면 일반 가져오기와 동일한 형태의 등록 결과 요약을 돌려주며, 미리보기였다면 예상 결과만 같은 형태로 반환한다.

## 권한 / 제약

로컬 데몬 경계 안에서 동작하며 별도 외부 인증은 없다. 형식 검증에서 오류가 나도 처리가 비정상 종료(panic)하지 않고 항상 구조화된 거절 응답으로 끝나도록 보장된다. 이 경로로 들어온 플랜은 감사 기록상 "엄격 가져오기"라는 출처로 표시되어, 파일 기반의 일반 가져오기와 구분된다. 등록 결과는 데이터베이스에 영구 기록되며, 마크다운 표현은 그 상태를 비추는 뷰로 취급된다(데이터베이스가 단일 진실원천). 미리보기 모드에서는 아무 것도 영구 기록되지 않는다.

## provenance

데몬의 가져오기/내보내기 라우터(`daemon/src/routes/import_export.rs`)가 `/plans/import/strict` 경로로 제공한다. 본문은 엄격 파서로 먼저 검증되고, 통과 시 일반 가져오기와 같은 영속화 경로를 거쳐 플랜·유닛·태스크로 기록된다. 제공 주체는 clawketd 데몬이며, 호출 주체는 Plan Mode 종료 훅(plan-sync)이다. 같은 경로를 훅 관점에서 별도로 서술한 노드(endpoint.daemon-plans-import-strict)가 존재한다.

## 미확정 (OPEN)
- [ ] OPEN: 엄격 형식 규칙의 구체 항목(필수 섹션·헤더 구조 등)이 무엇인지, 위반 유형의 표준 목록을 owner와 확정.
- [ ] OPEN: 이 노드와 endpoint.daemon-plans-import-strict의 역할 분담(같은 경로의 일반 관점 vs 훅 관점)을 SSOT에서 어떻게 정리·통합할지 결정.
