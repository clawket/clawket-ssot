---
id: concept.task-status
kind: Concept
title: Task 상태 (todo/in_progress/blocked/done/cancelled)
definition: 작업이 가질 수 있는 5개의 진행 단계 값으로, 데스크탑 대시보드에서 칸반 컬럼·요약 지표·상태 전이 버튼이 모두 이 값을 기준으로 그려진다.
relatesTo:
  - to: concept.task
    type: belongs-to
    note: Task 엔티티의 status 속성이 가질 수 있는 값 집합을 정의한다.
  - to: screen.board
    type: relates-to
    note: 칸반 보드의 컬럼 구성이 이 상태 값에 1:1로 대응한다(cancelled 제외).
  - to: screen.summary
    type: relates-to
    note: 요약 화면의 상태별 KPI 집계 기준이 된다.
governedBy: []
dependsOn:
  - concept.task
implementedIn:
  - desktop/apps/desktop/src/data/types.ts
  - desktop/apps/desktop/src/views/board/constants.ts
  - desktop/apps/desktop/src/views/SummaryView.tsx
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:concept
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지(식별자는 provenance에만). 분기/의도 불명은 OPEN. -->

## 정의

Task 상태는 하나의 작업이 진행되는 동안 거치는 다섯 가지 단계를 가리킨다. 다섯 값은 다음과 같다.

- **todo** — 아직 시작하지 않은, 대기 중인 작업.
- **in_progress** — 현재 진행 중인 작업.
- **blocked** — 외부 의존 등으로 더 진행할 수 없어 막힌 작업.
- **done** — 완료된 작업.
- **cancelled** — 진행을 취소한 작업(삭제 대신 소프트 취소로 보존된다).

이 다섯 값은 데스크탑 화면 곳곳에서 "작업이 지금 어디까지 왔는가"를 표현하는 공통 어휘로 쓰인다. 칸반 보드는 이 중 네 값(todo·in_progress·blocked·done)을 컬럼으로 세우고, 취소된 작업은 보드 아래 접이식 아카이브 영역에 따로 모은다. 요약 화면은 같은 값을 묶어 상태별 진행 지표(KPI)를 계산하고, 작업 카드 아래의 전이 버튼도 현재 상태에 따라 이동 가능한 다음 상태를 노출한다.

상태 사이의 이동에는 정해진 흐름이 있다. todo에서 시작하면 in_progress로, 거기서 done 또는 다시 todo로 돌아갈 수 있고, blocked는 풀리면 todo로, done은 다시 열어 in_progress로 되돌릴 수 있다. cancelled로의 직접 이동 버튼은 카드에 두지 않는다 — 취소는 상세 드로어의 삭제(=소프트 취소) 경로로만 도달하도록 분리되어 있다.

## 엔티티 (DB)

Task 상태는 독립 테이블이 아니라 Task 레코드의 한 속성으로 존재한다. 작업 한 건마다 현재 상태 값 하나를 들고 있으며, 허용되는 값은 위 다섯 가지로 고정된 열거형이다. 데스크탑은 이 값을 데몬이 내려준 그대로 신뢰하고, 별도로 상태 집합을 가공하지 않는다.

## API 표면

상태 자체를 다루는 전용 엔드포인트는 없다. 상태 변경은 작업 수정 흐름(작업 PATCH)을 통해 일어나며, 특히 done으로의 전이는 별도의 게이트가 걸려 있다 — 완료 처리에는 근거(evidence)가 필수라 데몬이 근거 없는 done을 거부한다. 따라서 보드 카드의 "Done →" 버튼도 이 게이트를 우회하지 않는다.

## 불변식

- 칸반 보드 컬럼은 todo·in_progress·blocked·done 네 가지만 노출하고 cancelled는 컬럼에서 의도적으로 제외한다(취소 작업은 아카이브 영역으로 분리).
- 작업 카드의 인라인 전이 버튼은 cancelled로 가는 경로를 제공하지 않는다 — 취소는 상세 드로어의 삭제 경로 전용이다.
- done 전이는 근거 필수 게이트로 보호되므로, 상태 값만 바꾼다고 완료 처리가 무조건 성립하지는 않는다.

## 구현 위치 (provenance)

- 상태 값 집합의 정의: `desktop/apps/desktop/src/data/types.ts`
- 칸반 컬럼·전이 버튼·우선순위 팔레트 등 상태 기반 화면 상수: `desktop/apps/desktop/src/views/board/constants.ts`
- 상태별 KPI 집계: `desktop/apps/desktop/src/views/SummaryView.tsx`

## 미확정 (OPEN)
- [ ] OPEN: blocked 상태로의 전이를 어떤 표면이 트리거하는지(작업 카드에 unblock만 있고 block 진입 경로는 별도 확인 필요).
