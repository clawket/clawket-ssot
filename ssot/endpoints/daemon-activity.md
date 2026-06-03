---
id: endpoint.daemon-activity
kind: Endpoint
title: 데몬 POST /activity
definition: 파일 변경·파괴적 명령 차단 같은 작업 행위를 감사 추적에 남기는 기록 통로. 파일 변경 후 훅과 파괴적 명령 가드가 호출한다.
realizedBy:
  - component.daemon
implementedIn:
  - clawket/adapters/shared/claude-hooks.cjs
relatesTo:
  - to: concept.audit-log
    type: mutates
    note: 행위 기록을 감사 로그에 적재한다
  - to: component.daemon
    type: backed-by
    note: 기록을 받아 적재하는 로컬 데몬
governedBy: []
impacts:
  - concept.audit-log
consumedBy:
  - endpoint.hook-post-tool-use
  - integration.git-destructive-guard
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의
에이전트가 한 일의 흔적을 남기는 기록 통로다. 어떤 파일이 바뀌었는지, 어떤 파괴적 명령이 차단되었는지 같은 "관찰용" 사건을 데몬에 보내 감사 추적에 적재한다. 작업 상태를 바꾸는 본 작업과 별개로, 나중에 "무슨 일이 있었나"를 되짚을 수 있도록 흔적만 모아두는 용도다. 사람이 직접 부르기보다, 파일 변경 후 자동으로 도는 훅과 파괴적 명령을 막는 가드가 자동으로 호출한다.

## 요청 / 응답
보내는 정보는 한 건의 사건을 설명하는 요소들이다 — 어떤 대상에 대한 일인지(엔티티 유형·식별자), 무슨 행위였는지(예: 파일 변경, 파괴적 명령 차단), 관련 세부 항목(예: 차단된 명령의 종류·잘려 보존된 명령 문자열). 응답은 적재 성공 신호 정도이며, 호출 측은 결과를 강하게 의존하지 않는다.

## 권한 / 제약
데몬이 로컬 루프백에만 바인딩되므로 같은 기기 안에서만 호출된다. 이 기록은 "최선 노력(best-effort)"이다 — 적재에 실패해도 본 작업을 막지 않는다. 특히 파괴적 명령 차단의 경우, 실제 방어는 차단 그 자체이고 이 기록은 관찰성(누가·언제·무엇을 차단했나)을 위한 보조 수단이라 실패해도 무방하다. 데몬 포트는 캐시 디렉터리의 포트 파일에서 읽으며, 포트 파일이 없으면 호출을 조용히 건너뛴다.

## provenance
호출 측 근거는 `clawket/adapters/shared/claude-hooks.cjs`의 `recordDestructiveBlock`(포트 파일에서 포트를 읽어 `/activity`에 entity_type·action·field·old/new_value를 전송, 실패는 비치명적)과 동일 파일의 PostToolUse 경로 `/activity` 호출이다. 적재 주체는 로컬 데몬(component.daemon)이며 감사 로그(concept.audit-log)에 쌓인다. 호출자는 파일 변경 후 훅(endpoint.hook-post-tool-use)과 파괴적 명령 가드(integration.git-destructive-guard)다.

## 미확정 (OPEN)
- [ ] OPEN: 기록되는 사건 유형의 전체 카탈로그와 감사 로그의 해시 체인 연결 방식(append 시 prev_hash 계산 위치가 데몬인지)은 데몬 라우터·감사 로그 코드를 정독해 확인 필요.
