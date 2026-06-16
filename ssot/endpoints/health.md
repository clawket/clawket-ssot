---
id: endpoint.health
kind: Endpoint
title: 데몬 헬스 엔드포인트
definition: "GET /health — 로컬 데몬이 살아 있는지와 함께 버전·엔진·벡터검색 가용성·프로세스 식별자·기동 후 경과시간·DB 스키마 버전을 한 번에 돌려준다."
realizedBy: [component.daemon]
implementedIn:
  - daemon/src/main.rs
relatesTo:
  - { to: component.web-dashboard, type: relates-to, note: "대시보드와 CLI status/설치 게이트가 살아있음과 버전·스키마 확인용으로 핑한다" }
governedBy:
  - invariant.daemon-liveness-socket-probe
  - invariant.schema-version-discipline
impacts:
  - component.install-gate
consumedBy:
  - component.web-dashboard
  - endpoint.cli-daemon
  - component.install-gate
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
---

## 정의
데몬이 살아 있는지, 그리고 어떤 상태로 살아 있는지를 묻는 진단용 창구다. 누군가 "지금 이 백그라운드 서비스가 정상이냐"를 물을 때 가장 먼저 두드리는 문으로, 설치 직후 점검(설치 게이트), CLI의 상태 확인 명령, 웹 대시보드의 초기 연결 확인이 모두 이 엔드포인트를 사용한다. 응답이 정상적으로 돌아오면 데몬이 떠 있다는 뜻이고, 응답 자체로 추가 진단 정보까지 함께 받는다.

## 요청 / 응답
요청은 별도 입력 없이 보내기만 하면 된다. 응답으로는 다음 의미의 정보를 돌려준다.
- 상태: 정상 동작 중인지를 나타내는 표시("ok").
- 버전: 현재 실행 중인 데몬 빌드의 버전 — CLI가 새 버전 설치 필요 여부를 판단하는 근거.
- 엔진: 어떤 구현체가 돌고 있는지("rust"). 과거 Node 구현과 구분하기 위한 표식.
- 벡터검색 가용성: 의미 검색(임베딩 기반)이 이 인스턴스에서 켜져 있는지 여부. 임베딩 엔진/벡터 테이블이 준비됐는지 알려준다.
- 프로세스 식별자: 데몬이 점유한 OS 프로세스 번호. 중복 기동 방지나 종료 시 참조.
- 기동 후 경과시간: 데몬이 떠 있은 지 얼마나 됐는지. 비정상 재시작을 감지하는 단서.
- 스키마 버전: 현재 DB가 적용받은 마이그레이션 단계 번호. CLI status가 이 값을 읽어 DB가 기대 버전과 맞는지 판단한다.

## 권한 / 제약
로컬 전용이다. 데몬은 루프백·유닉스소켓에만 바인딩하므로 외부 네트워크에서 호출되지 않으며, 인증 없이 같은 머신의 클라이언트(CLI·웹·설치 게이트)가 자유롭게 핑한다. 이 엔드포인트는 읽기 전용 진단으로 어떤 상태도 변경하지 않는다. 살아있음 판정은 소켓 프로브 불변식을 따르고, 스키마 버전 값은 마이그레이션 규율 불변식의 관찰 창구 역할을 한다.

## provenance
구현은 데몬 바이너리의 라우터에 `/health` 경로로 등록되어 있으며(`daemon/src/main.rs`), 응답은 위 7가지 의미의 값을 조립한 구조다. CLI의 status 명령은 이 엔드포인트를 별도로 프로빙해 기동 경과시간과 스키마 버전을 끌어온다. 제공 주체는 데몬 컴포넌트.

## 미확정 (OPEN)
- [ ] OPEN: 벡터검색 비가용(vec_enabled=false) 시 다운스트림(검색 엔드포인트/위키)이 어떤 폴백을 취하는지의 명시적 계약 확인 필요 — 판정은 owner.
