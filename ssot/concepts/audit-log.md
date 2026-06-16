---
id: concept.audit-log
kind: Concept
title: 감사 로그(tamper-evident)
definition: "엔티티에 일어난 모든 변경(생성·수정·상태전이·삭제·승인·활성화·완료·정책위반)을 누가·무엇을·언제 했는지로 한 줄씩 남기는 로컬 추적 기록. 각 기록은 직전 기록을 가리키는 체인 값을 함께 보관해 중간에 누가 기록을 손댔는지(변조)를 탐지할 수 있게 한다. 작성 후에는 API로 절대 수정·삭제되지 않는 추가 전용(append-only) 장부다."
relatesTo:
  - to: concept.task
    type: relates-to
    note: 태스크의 상태 변경·evidence 기입 등이 감사 로그에 기록된다
  - to: endpoint.daemon-activity
    type: relates-to
    note: 훅이 파일 변경·파괴적 명령 차단을 활동으로 보내면 추적에 남는다
  - to: capability.watch-replay
    type: relates-to
    note: 리플레이는 감사 로그에 쌓인 과거 이벤트를 재생한다
governedBy: []
governs: []
dependsOn: []
implementedIn:
  - daemon/src/repo/audit_log.rs
  - daemon/src/routes/audit.rs
  - daemon/migrations/016_audit_log.sql
  - daemon/migrations/018_audit_log_prev_hash.sql
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-03
tags:
  - status:active
  - type:concept
---

<!-- 작성 고도: 자연어 우선. 코드 식별자는 provenance/근거에만. -->

## 정의

감사 로그는 Clawket 안에서 어떤 작업 객체(프로젝트·플랜·유닛·사이클·태스크 등)에 변화가 생길 때마다 그 변화를 한 줄의 영구 기록으로 남기는 로컬 장부다. "활동 로그(activity log)"가 이전 이름이며, 현재는 감사 로그로 통합·대체되었다.

기록 한 줄은 다음을 답한다.
- **누가 했나(actor)**: 변경을 일으킨 주체. 허용되는 값은 네 가지뿐이다 — Claude(LLM 에이전트), CLI(명령줄 사용), 외부 API 호출, 시스템. 명시되지 않은 주체나 알 수 없는 값은 일괄적으로 CLI로 정규화해 기록한다(미상이 그대로 새지 않도록).
- **무엇에(entity)**: 변경 대상의 종류와 식별자.
- **어떤 종류의 변경(op_type)**: 상태 변경 / 생성 / 삭제 / 수정 / 승인 / 활성화 / 완료 / 정책 위반 중 하나.
- **무엇이 어떻게 바뀌었나**: 바뀐 항목 이름과 이전 값·새 값(해당될 때).
- **언제(at)**: 변경 시각.

각 기록에는 추가로 **직전 기록을 가리키는 체인 값(prev_hash)** 이 붙는다. 이는 바로 앞 기록의 식별자를 해시해 만든 값으로, 기록들이 사슬처럼 이어진다. 누군가 중간 기록을 몰래 지우거나 바꾸면 사슬이 끊겨 탐지할 수 있다(변조 탐지, tamper-evident). 첫 기록에는 직전이 없으므로 이 값이 비어 있다.

체인 값은 가벼운 변조 탐지를 위한 장치이지 암호학적으로 강한 보증은 아니다. 본 구현은 직전 기록 식별자에 대해 FNV-1a 계열의 결정적 해시(16자리 16진수를 4번 반복해 64자리로 채움)를 사용한다. 코드상 함수 이름은 `sha256_hex`이지만 실제 계산은 SHA-256이 아니라 FNV-1a 64비트 혼합이며, 주석이 이를 "암호학적으로 강하지 않으나 로컬 데몬의 변조 탐지에는 충분"하다고 명시한다. 풀 Merkle 트리 없이 사슬 무결성을 얻기 위한 의도적 단순화다.

## 엔티티 (DB)

감사 기록은 단일 장부 테이블(audit_log)에 한 행씩 쌓인다. 한 행이 담는 정보는 다음과 같다.

- 기록 식별자: 각 기록의 고유 ID(접두사 AUD 가 붙은 ULID 형태의 시간정렬 식별자).
- 대상 종류 / 대상 식별자: 어떤 객체에 대한 변경인지.
- 변경 종류: 허용 집합은 상태변경·생성·삭제·수정·승인·활성화·완료·정책위반 여덟 가지로 DB 제약(CHECK)으로 강제된다. 이 외의 값은 저장 자체가 거부된다.
- 변경된 항목 이름, 이전 값, 새 값(선택).
- 주체(actor): claude / cli / external-api / system 네 값만 DB 제약으로 허용되며 기본값은 cli.
- 시각: ISO 8601 UTC 문자열(예 2026-05-04T12:00:00.000Z). 기존 다른 컬럼들이 정수(에포크 밀리초)인 것과 달리 감사 시각은 사람이 읽는 ISO 문자열로 저장한다.
- 직전 기록 체인 값(prev_hash, 선택): 첫 기록은 비어 있음.

조회 효율을 위해 대상별·시각순(최신순)·주체별·변경종류별 색인이 있으며, 체인 값에도 별도 색인이 있어 사슬을 따라 걷는 조회가 가능하다.

과거 활동 로그(activity_log) 테이블의 기존 행들은 감사 로그 도입 시 일괄 이전되었다. 이전 과정에서 옛 주체 표기(human·main·빈 값 등)는 새 허용 집합으로 매핑되고, 정수 시각은 ISO 문자열로 변환된다. 호환을 위해 옛 활동 로그 테이블은 롤업 작업이 참조하므로 당분간 함께 남는다.

## API 표면

감사 로그는 데몬의 HTTP 표면에서 **읽기 전용**이다.

- 목록 조회(GET /audit): 대상 종류·대상 식별자·주체·변경 종류·개수 제한으로 필터해 시간 오름차순(오래된 것부터)으로 반환한다. 기본 반환 개수는 50건.
- 쓰기 시도(POST/PUT/DELETE/PATCH /audit): 모두 405(METHOD_NOT_ALLOWED)로 거부된다. "감사 로그는 불변이며 쓰기 메서드는 허용되지 않는다"는 메시지를 돌려준다.

즉 감사 기록은 외부에서 추가·수정·삭제할 수 없다. 기록은 오직 데몬 내부 코드가 다른 변경을 처리하는 흐름 속에서만 남긴다(record 경로를 거쳐 직전 기록을 찾아 체인 값을 계산한 뒤 한 행을 추가). 리플레이 기능은 이 장부에 쌓인 과거 이벤트를 SSE로 재생하는 데 사용한다.

## 불변식

- **추가 전용**: 기록은 작성 후 API로 수정·삭제할 수 없다. 모든 쓰기 메서드가 405로 하드블록된다.
- **주체 폐집합**: actor 는 claude / cli / external-api / system 네 값만 허용. 미상·비표준 값은 cli 로 정규화. DB CHECK 가 최종 방어선.
- **변경 종류 폐집합**: op_type 은 정해진 여덟 값만 허용(DB CHECK).
- **체인 연속성**: 첫 기록을 제외한 모든 기록은 직전 기록을 가리키는 체인 값을 가진다. 사슬이 끊기면 변조 신호로 본다(다만 해시는 강암호가 아닌 FNV-1a 계열의 경량 탐지 수단).

## 구현 위치 (provenance)

- 기록 작성·목록·필터·체인 값 계산: `daemon/src/repo/audit_log.rs` (record / list, FNV-1a 기반 체인 해시 포함).
- 읽기 전용 라우트와 405 강제: `daemon/src/routes/audit.rs`.
- 장부 테이블 생성·옛 활동 로그 이전·색인: `daemon/migrations/016_audit_log.sql`.
- 체인 값 컬럼·체인 색인 추가: `daemon/migrations/018_audit_log_prev_hash.sql`.

## 미확정 (OPEN)
- [ ] OPEN: CLI(`clawket`)에서 감사 로그를 직접 조회하는 최상위 서브커맨드가 있는지(노드 소스의 `src/main.rs:1340-1358`는 CLI 레포 기준 경로이나 현 코드 지도상 audit 전용 CLI 명령 미확인) — 데몬 GET /audit 만 확인됨.
- [ ] OPEN: 정책 위반(POLICY_VIOLATION) op_type 이 실제로 어떤 경로에서 기록되는지(훅 차단과의 연결) 추적 미완.
