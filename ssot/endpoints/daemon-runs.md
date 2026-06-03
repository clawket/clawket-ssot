---
id: endpoint.daemon-runs
kind: Endpoint
title: 데몬 실행 이력 API (/runs)
definition: "작업의 에이전트 실행 기록(Run)을 조회하는 데몬 엔드포인트. 작업 단위 또는 프로젝트 단위로 필터해 실행 이력 목록을 돌려준다. 데스크탑·웹은 이를 읽기 전용으로만 소비하고, 실제 생성·종료는 훅(어댑터)이 작업 시작/종료 시 처리한다."
realizedBy: [component.daemon]
implementedIn:
  - desktop/apps/desktop/src/data/api.ts
relatesTo:
  - { to: concept.run, type: reads, note: "작업별 실행 기록을 조회한다" }
  - { to: concept.task, type: reads, note: "실행 기록은 항상 특정 작업에 종속된다" }
impacts: [concept.run]
consumedBy: [component.web-dashboard]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

작업이 에이전트에 의해 실행된 기록(Run)을 조회하는 API다. 한 작업이 언제 시작되고 끝났는지, 어떤 에이전트가 어떤 결과를 냈는지 같은 실행 이력을 화면이 사후 점검·타임라인 표시·비교에 쓸 수 있게 노출한다.

이 엔드포인트는 조회 전용이다. 실행 기록의 생성·종료는 별도 경로 — Claude Code 어댑터(훅)가 작업이 시작될 때 자동으로 만들고 끝날 때 마감한다 — 에서 이뤄지고, 데스크탑·웹은 그렇게 쌓인 이력을 읽기만 한다.

## 요청 / 응답

- **목록**: 실행 기록들을 돌려준다. 특정 작업으로 좁히거나 특정 프로젝트 범위로 추릴 수 있다. 각 항목은 시작/종료 시각·실행 주체(에이전트)·결과·노트 등을 담는다.

## 권한 / 제약

- 데스크탑·웹 표면에서는 읽기 전용이다 — 화면을 통해 실행 기록을 직접 만들거나 고치지 않는다.
- 로컬 데몬 특성상 호출 주체는 같은 머신의 사람·에이전트로 한정된다.

## provenance

- implementedIn: 데스크탑 데몬 클라이언트의 listRuns가 작업·프로젝트 필터로 /runs를 조회하며, 주석이 "실행 기록은 어댑터가 작업 시작/종료에 만들고 마감한다"는 책임 분리를 명시한다.
- realizedBy: clawketd 데몬이 실행 기록 저장소를 소유하고 이 조회 API를 제공한다.

## 미확정 (OPEN)
- [ ] OPEN: 실행 기록 생성·종료(start/finish) 쓰기 경로의 정확한 데몬 엔드포인트와 어댑터 호출 시점 매핑.
