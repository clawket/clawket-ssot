---
id: endpoint.cycles-list
kind: Endpoint
title: 사이클 목록 조회 (GET /cycles)
definition: 등록된 사이클들을 조건에 맞춰 돌려주는 데몬 조회 통로. 프로젝트·소속 유닛·상태로 범위를 좁혀 사이클 목록을 받는다.
realizedBy:
  - component.daemon
implementedIn:
  - daemon/src/routes/cycles.rs
relatesTo:
  - to: concept.cycle
    type: reads
    note: 사이클 엔티티를 조회 대상으로 읽는다
  - to: concept.unit
    type: reads
    note: 소속 유닛으로 사이클 범위를 좁힐 수 있다
  - to: concept.project
    type: reads
    note: 프로젝트 단위로 사이클 범위를 좁힐 수 있다
governedBy: []
impacts:
  - domain.work-management
consumedBy:
  - component.web-dashboard
  - endpoint.cli-cycle
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의
"지금 어떤 사이클들이 있나"를 묻는 조회 통로다. 사이클은 단일 유닛에 묶인 타임박스 반복(스프린트)이므로, 이 조회는 특정 프로젝트나 특정 유닛에 속한 사이클만 골라 보거나 상태(계획 중·진행 중·완료)별로 추려 볼 때 쓴다. 사람은 보드·백로그 화면에서 사이클 띠를 채우는 재료로, 터미널 사용자는 `clawket cycle list`의 결과로 이 목록을 받는다.

## 요청 / 응답
보내는 정보는 모두 선택값인 세 가지 범위 조건이다 — 어떤 프로젝트인지, 어떤 유닛에 속하는지, 어떤 상태인지. 아무 조건도 주지 않으면 전체를 받는다. 프로젝트는 내부 식별자뿐 아니라 사람이 부르는 참조로도 지정할 수 있어, 호출자가 식별자를 미리 풀어둘 필요 없이 그대로 넘겨도 된다. 돌려받는 것은 조건에 맞는 사이클 목록이며, 각 항목은 사이클의 제목·목표·소속 유닛·상태 같은 그 사이클을 이해하고 화면에 그릴 수 있는 정보를 가진다.

## 권한 / 제약
데몬이 로컬 루프백·유닉스소켓에만 바인딩되므로 같은 기기 안에서만 조회된다. 프로젝트 참조는 내부에서 실제 프로젝트로 변환(resolve)된 뒤 필터에 쓰이며, 존재하지 않는 참조를 주면 조회가 그에 맞게 처리된다. 읽기 전용 조회이므로 상태를 바꾸지 않는다.

## provenance
근거는 `daemon/src/routes/cycles.rs`의 `/cycles` GET 라우트(`list` 핸들러)다. 선택적 프로젝트·유닛·상태 조건으로 사이클을 거르며, 프로젝트 참조는 변환 단계를 거쳐 필터로 들어간다. 제공 주체는 로컬 데몬(component.daemon)이고, 소비 주체는 웹 대시보드(component.web-dashboard)와 `clawket cycle`의 list(endpoint.cli-cycle)다.

## 미확정 (OPEN)
- [ ] OPEN: 목록의 기본 정렬 기준(예: 생성순·인덱스순)과 페이지네이션 유무는 사이클 저장소 코드를 정독해 확인 필요.
