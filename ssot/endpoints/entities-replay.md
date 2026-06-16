---
id: endpoint.entities-replay
kind: Endpoint
title: 데몬 GET /entities/{id}/replay
definition: 특정 엔티티(태스크·플랜·사이클 등)가 과거 임의 시점에 어떤 상태였는지를 감사 기록을 거슬러 되살려 돌려주는 조회 엔드포인트.
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/runs.rs]
relatesTo:
  - { to: concept.audit-log, type: reads, note: 엔티티 변경 이력(누가·언제·어떤 필드를 무엇으로 바꿨는지)을 시간순으로 읽어 상태를 재구성한다 }
  - { to: capability.watch-replay, type: realizes, note: 실시간 관찰·리플레이 능력 중 사후 상태 복원 부분을 구현한다 }
governedBy: [invariant.preserve-history]
impacts: []
consumedBy: [endpoint.cli-replay]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
하나의 엔티티가 "그때 그 순간 어떤 모습이었는지"를 알고 싶을 때 쓰는 조회 기능이다. 엔티티 식별자와 "어느 시점까지"를 함께 주면, 그 시점까지 누적된 모든 변경을 시간순으로 차곡차곡 덮어 적용한 결과 — 즉 그 시점의 스냅샷 — 을 돌려준다. 사후에 "이 태스크가 어쩌다 이 상태가 됐나"를 추적하거나, 사고 시점의 형상을 복기할 때 사용한다. CLI의 리플레이 명령이 이 기능을 사용자에게 노출하는 표면이다.

## 요청 / 응답
보내는 정보: 되살릴 대상 엔티티의 식별자와, 선택적으로 "기준 시점"(ISO 8601 형식의 시각). 기준 시점을 생략하면 "지금까지 전부" — 즉 전체 이력을 끝까지 접어 현재 상태에 해당하는 스냅샷을 만든다.

돌려주는 정보:
- 대상 엔티티 식별자
- 재구성된 스냅샷 — 그 시점까지의 변경을 모두 적용한 결과로 본 엔티티의 필드별 값 묶음. 이력이 하나도 없으면 스냅샷은 "없음(null)"으로 표시된다.
- 실제로 어느 시점까지 되살렸는지 (적용된 마지막 변경의 시각)
- 접어 넣은 변경 기록의 개수

이력 중간에 "삭제됨" 사건이 있으면 그 지점에서 스냅샷이 비워진 뒤 이후 변경부터 다시 쌓이므로, 삭제 후 재생성된 엔티티의 과거 모습도 정확히 재현된다.

## 권한 / 제약
로컬 데몬에 붙은 호출자라면 누구나 읽을 수 있는 조회성 엔드포인트로, 상태를 바꾸지 않는다. 기준 시점 비교는 ISO 8601 문자열의 사전식(글자) 정렬이 시간순과 일치한다는 전제에 의존하므로, 시각은 정규화된 ISO 8601 형식이어야 정확하다. 이 기능이 성립하려면 변경 이력이 보존되어 있어야 하며(파괴적 삭제로 이력을 지우지 않는다는 불변식에 의존), 이력은 감사 로그에 남는다.

## provenance
구현은 데몬 라우터의 runs 모듈(`daemon/src/routes/runs.rs`)에 `GET /entities/{id}/replay`로 등록되어 있고, 감사 로그 테이블의 행들을 시간순으로 접어 스냅샷을 만든다. 제공 주체는 로컬 clawketd 데몬(`component.daemon`)이다. 코드가 사라지더라도, "감사 이력을 시점 기준으로 접어 그 시점의 엔티티 상태를 복원한다"는 정책과 위 요청/응답 의미만으로 재구현 가능하다.

## 미확정 (OPEN)
- [ ] OPEN: 권한 모델(로컬 단독 사용 외에 인증·토큰 요구 여부)은 데몬 미들웨어 전반을 확인해 owner 가 확정할 것.
