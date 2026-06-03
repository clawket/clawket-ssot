---
id: invariant.migration-gate
kind: Invariant
title: 스키마 마이그레이션 중에는 변경 요청 차단
definition: 데몬이 스키마 마이그레이션을 진행하는 동안 상태를 바꾸는 요청은 일시적 거부(HTTP 503, MIGRATION_IN_PROGRESS)로 막히고, 두 리스너(유닉스·TCP)가 동일하게 동작한다.
governs:
  - component.daemon
  - component.sqlite-storage
  - domain.work-management
implementedIn:
  - daemon/src/middleware/migration_gate.rs
  - daemon/src/main.rs
decidedBy:
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:invariant
---

## 제약

데몬이 SQLite 스키마를 마이그레이션하는 도중에는, 데이터를 **바꾸는 요청**(생성·수정·삭제 계열)을 받아들이지 않는다. 이때 데몬은 요청을 일시적 거부로 응답한다 — "지금 스키마 마이그레이션 중"이라는 코드(`MIGRATION_IN_PROGRESS`)와 함께 HTTP 503 을 돌려주고, 잠시 후 재시도하라는 신호(Retry-After)를 붙인다.

반대로 **읽기 요청**(조회 계열)과 실시간 이벤트 스트림(SSE)은 마이그레이션 중에도 그대로 통과시킨다 — 대시보드가 마이그레이션이 도는 동안에도 진행 상황을 계속 볼 수 있어야 하기 때문이다.

핵심은 이 규칙이 **두 문 모두에 동일하게** 걸린다는 점이다. 유닉스 소켓으로 들어오든 TCP 포트로 들어오든, 마이그레이션 중 변경 요청은 같은 방식으로 막힌다. 즉 어느 진입점을 쓰는지에 따라 동작이 달라지지 않는다.

## 깨지면 무슨 일이 일어나나

스키마가 절반쯤 바뀐 상태에서 쓰기 요청이 통과하면, 변경 전 스키마를 가정한 쓰기와 변경 후 스키마가 충돌해 데이터가 깨지거나 트랜잭션이 어긋날 수 있다. 두 리스너에 규칙이 다르게 걸리면(예: TCP 만 막고 소켓은 안 막음), 막히지 않은 문으로 들어온 쓰기가 같은 손상을 일으킨다 — "어디로 들어왔는지"가 데이터 안전을 좌우하면 안 된다.

## 코드에서 어떻게 강제되나

이 검사는 두 리스너가 공유하는 기반 라우터 계층에 얹혀 있다. 그래서 유닉스·TCP 어느 쪽이든 동일한 미들웨어를 거친다. 미들웨어는 요청 메서드가 상태를 바꾸는 종류(POST/PATCH/PUT/DELETE)인지 보고, 그러면서 데몬이 마이그레이션 중 상태(`is_migrating()`)이면 503 응답에 재시도 헤더를 붙여 즉시 반환한다.

현재 아키텍처에서는 기동 시 마이그레이션이 HTTP 리스너가 열리기 **전에** 끝나므로, 콜드 스타트 상황에서는 이 게이트가 실제로 발동할 일이 거의 없다(휴면 상태). 다만 앞으로 런타임/온라인 마이그레이션 경로가 생기더라도 모든 마이그레이션 진입점에서 같은 계약이 일관되게 성립하도록 미리 자리를 잡아 둔 장치다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요 — 마이그레이션 중 쓰기 차단·읽기 통과 정책의 결정 기록 미확인
