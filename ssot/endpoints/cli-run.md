---
id: endpoint.cli-run
kind: Endpoint
title: clawket run (start/finish/view/list)
definition: 태스크 한 건의 실행 시도를 시작·종료·조회하는 CLI 명령군으로, 보통 훅이 자동으로 만든다.
realizedBy:
  - component.cli
  - component.daemon
implementedIn:
  - cli/src/main.rs
relatesTo:
  - to: concept.run
    type: mutates
    note: 실행 기록의 시작·종료를 다루는 주 진입점
  - to: concept.task
    type: backed-by
    note: 실행 기록은 항상 한 태스크에 매여 있음
  - to: concept.audit-log
    type: relates-to
    note: 실행 시작·종료가 감사 로그에 기록됨
governedBy: []
impacts:
  - domain.agent-runs
consumedBy:
  - component.claude-adapter
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의
실행 기록(run)은 태스크 한 건을 실제로 시도한 한 번의 작업 세션이다. "이 태스크를 누가, 언제 시작해서, 어떤 결과로 끝냈는가"를 보존한다. 같은 태스크라도 여러 번 시도하면 여러 실행 기록이 쌓여, 나중에 타임라인에서 재생하거나 시도끼리 비교할 수 있다. 보통은 사람이 직접 만들지 않고, 태스크가 진행 중으로 바뀔 때 훅이 자동으로 실행 기록을 시작한다.

## 요청 / 응답
- **시작(start)**: 대상 태스크와 실행 주체(에이전트)·세션 정보를 받아 새 실행 기록을 연다. 세션 식별자는 훅이 환경 변수로 넘기는 내부 값이다.
- **종료(finish)**: 열린 실행 기록을 결과(성공·실패·취소)와 자유 서술 노트로 닫는다.
- **보기/목록(view/list)**: 단건 상세 또는 태스크·세션 기준으로 거른 실행 기록 목록을 돌려준다.

## 권한 / 제약
- 실행 기록은 반드시 특정 태스크에 매여 생성된다.
- 시작·종료는 보통 Claude 훅 흐름(태스크 시작 시 자동 start, 서브에이전트 종료 시 결과 반영)이 대행하므로, 사람·에이전트가 수동으로 직접 호출하는 경우는 드물다.

## provenance
- 명령·옵션 정의: `cli/src/main.rs`의 RunAction 정의(start/finish/view/list).
- 자동 생성 경로: Claude 어댑터 훅이 태스크 시작·서브에이전트 종료 시점에 실행 기록을 열고 닫는다. 실제 저장은 clawketd 데몬의 run HTTP API.

## 미확정 (OPEN)
- [ ] OPEN: 실행 기록에 보존되는 봉투 스냅샷·결과 메타데이터의 구체 의미와, 수동 start와 훅 자동 start의 동작 차이 확인 필요.
