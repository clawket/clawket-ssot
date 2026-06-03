---
id: endpoint.activity
kind: Endpoint
title: 데몬 GET/POST /activity
definition: 엔티티별 활동 로그(누가 언제 무엇을 어떻게 바꿨는지)를 조회하거나 새 활동 한 건을 기록하는 엔드포인트.
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/activity.rs]
relatesTo:
  - { to: concept.audit-log, type: reads, note: 엔티티 변경 활동 로그를 시간순으로 조회한다 }
  - { to: concept.audit-log, type: mutates, note: 파일 변경·차단 등 활동 한 건을 추적 기록에 남긴다 }
governedBy: [invariant.preserve-history]
impacts: []
consumedBy: [endpoint.daemon-activity]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
작업 과정에서 일어난 활동 — 파일 변경, 파괴적 명령 차단, 엔티티 필드 수정 등 — 을 시간순으로 들여다보거나 새로 한 건 남기는 기능이다. "이 태스크에서 무슨 일이 있었나"를 사후에 훑거나, 훅이 자동으로 활동을 적재할 때 쓰인다. 같은 활동 모듈은 보관 용량을 점검하는 통계 조회도 함께 제공한다.

## 요청 / 응답
활동 조회 시 보내는 정보(모두 선택): 대상 엔티티의 종류와 식별자로 범위를 좁히는 조건, 그리고 가져올 최대 건수. 돌려주는 정보: 조건에 맞는 활동 기록 목록 — 각 기록은 대상 엔티티(종류·식별자), 동작 유형, 바뀐 항목과 그 이전·이후 값, 그리고 행위 주체를 가진다.

활동 기록 시 보내는 정보: 대상 엔티티 종류·식별자, 동작 유형(필수)과, 선택적으로 바뀐 항목·이전 값·이후 값·행위 주체. 돌려주는 정보: 막 적재된 활동 기록 한 건.

같은 모듈의 통계 조회는 현재 사용 용량과 상한, 활성(hot) 행 수, 보관 배치 수, 가장 오래된 보관 구간 시작 시각, 보존 일수(활성·전체) 등 활동 로그 보관 현황을 돌려준다.

## 권한 / 제약
로컬 데몬에 붙은 호출자가 사용한다. 활동 기록에는 대상 엔티티(종류·식별자)와 동작 유형이 반드시 있어야 한다. 활동 로그는 보존 정책(활성 보관 일수·전체 보관 일수·용량 상한)에 따라 일정 시점이 지나면 활성 테이블에서 보관 영역으로 이관되며, 이력 자체는 보존된다.

## provenance
구현은 데몬 라우터의 activity 모듈(`daemon/src/routes/activity.rs`)로, `/activity` 경로에 목록(GET)과 기록(POST)을, `/activity/stats`에 보관 통계(GET)를 등록한다. 보존 정책은 환경 설정에서 읽고, 적재·이관은 활동 로그 롤업 잡(activity_log_rollup)이 담당한다. 제공 주체는 로컬 clawketd 데몬(`component.daemon`)이다. 훅이 활동을 적재하는 표면은 별도로 `endpoint.daemon-activity`로 정리되어 있으며 이 모듈이 그 적재를 받는다. 위 요청/응답·보존 정책 의미만으로 재구현 가능하다.

## 미확정 (OPEN)
- [ ] OPEN: 활동 로그(activity_log)와 감사 로그(audit_log)가 동일 추적 체계인지 별개 테이블인지, 동작 유형 표준 어휘는 모델·롤업 잡을 함께 확인해 owner 가 확정할 것.
