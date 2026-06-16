---
id: endpoint.admin-exec-sql
kind: Endpoint
title: 데몬 POST /admin/exec_sql
definition: 관리자가 저장소에 직접 SQL을 실행하되, 파괴적 DDL/DML은 거부하고 온라인 변경 중에는 마이그레이션 게이트를 거는 로컬 전용 채널.
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/admin.rs]
relatesTo:
  - { to: concept.task, type: mutates, note: "추가형 스키마 변경은 허용하지만 tasks에 대한 무조건 일괄 삭제는 거부한다" }
governedBy: [invariant.migration-gate, invariant.tcp-auth]
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
운영·진단·온라인 스키마 변경을 위해 관리자가 저장소에 SQL을 직접 보내 실행할 수 있는 로컬 전용 채널이다. 다만 데이터를 영구적으로 잃게 만드는 형태의 명령은 받아들이지 않는다 — 읽기 조회와 "추가형(데이터를 잃지 않는)" 변경만 통과시키고, 파괴적 명령은 명확한 거부 응답으로 막는다. 이 엔드포인트의 존재 이유 중 하나는 "파괴적 변경은 금지된다"는 계약을 단순한 부재가 아니라 능동적인 거부 응답으로 관측 가능하게 만드는 것이다.

## 요청 / 응답
- 요청: 실행할 SQL 문장 하나를 보낸다.
- 통과(허용) 경로:
  - 읽기 조회(SELECT): 결과 행들을 컬럼명-값 형태의 목록으로 돌려준다.
  - 추가형 변경(컬럼 추가, 인덱스/테이블 생성 등 데이터를 잃지 않는 변경): 영향받은 행 수를 돌려준다.
- 거부 경로(모두 "잘못된 요청"으로 코드화된 오류 반환):
  - 컬럼 삭제, 테이블/인덱스 삭제 → "파괴적 DDL 금지"
  - 테이블 비우기(TRUNCATE), 조건 없는 태스크 일괄 삭제 → "태스크 일괄 삭제 금지"

## 권한 / 제약
- 로컬 전용 관리자 채널이다. 외부 네트워크 경유 호출을 전제로 하지 않는다.
- 인증: TCP로 들어오는 호출은 데몬 토큰(`invariant.tcp-auth`)으로 보호되고, 유닉스 소켓 경로는 파일시스템 권한으로 이미 격리되므로 추가 인증을 두지 않는다.
- 마이그레이션 게이트(`invariant.migration-gate`): 읽기가 아닌(=온라인 변경) 문장이 실행되는 동안에는 데몬을 "마이그레이션 진행 중" 상태로 전환해, 동시에 들어오는 데이터 변경 요청이 일시적으로 거부되도록 한다. 이 게이트는 성공·실패 어느 경로로 끝나든 자동으로 해제되어 누수되지 않는다(조회 문장은 게이트를 걸지 않는다).
- 파괴적 판별은 정밀 파서가 아니라 단순 패턴 검사다. 관리자 전용 채널이라는 성격상, 모호하면 거부 쪽으로 기운다(거짓 양성은 감수).

## provenance
- 라우터/핸들러: `daemon/src/routes/admin.rs`
- 제공 컴포넌트: 로컬 데몬(`component.daemon`)
- 관련 불변식: 마이그레이션 게이트(`invariant.migration-gate`), TCP 토큰 인증(`invariant.tcp-auth`)

## 미확정 (OPEN)
- [ ] OPEN: 허용/거부 SQL 카탈로그의 범위(향후 추가될 패턴, 화이트리스트화 여부)와 거부 오류 코드의 안정성 보증을 owner가 정책으로 확정할 필요가 있는지 확인.
