---
id: endpoint.plans-list
kind: Endpoint
title: 플랜 목록 조회
definition: "GET /plans — 플랜(승인된 의도를 담는 로드맵)을 목록으로 돌려주며, 프로젝트 범위와 상태(draft/active/completed)로 좁혀 조회할 수 있다."
realizedBy: [component.daemon]
implementedIn:
  - daemon/src/routes/plans.rs
relatesTo:
  - { to: concept.plan, type: reads, note: "Plan 엔티티를 프로젝트·상태 필터로 읽어 반환" }
governedBy:
  - invariant.one-active-plan-per-project
governs: []
dependsOn: []
impacts: []
consumedBy:
  - endpoint.cli-plan
  - component.web-dashboard
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
---

## 정의
어떤 플랜들이 있는지, 그 가운데 무엇이 초안이고 무엇이 활성이며 무엇이 완료됐는지를 한눈에 보기 위한 조회 창구다. 웹 대시보드가 프로젝트의 플랜 현황을 그릴 때, CLI의 플랜 목록 명령이 표를 출력할 때 사용한다. 플랜은 태스크 시작 자격의 전제(활성 플랜이 있어야 태스크를 시작할 수 있음)이므로, 이 목록은 "지금 일을 시작할 수 있는 상태인가"를 가늠하는 출발점이기도 하다.

## 요청 / 응답
선택적으로 두 가지로 범위를 좁힐 수 있다 — 어느 프로젝트의 플랜인지(프로젝트 식별자 또는 사람이 읽는 키로 지정, 둘 다 해석된다)와, 어떤 상태(초안·활성·완료)의 플랜만 볼지. 둘 다 생략하면 전체를 돌려준다. 응답으로는 조건에 맞는 플랜들의 목록을 돌려주며, 각 항목은 플랜을 식별·표시하는 데 필요한 정보 — 제목, 설명, 소속 프로젝트, 상태, 출처, 승인 시각 등 — 의 의미를 담는다.

## 권한 / 제약
로컬 전용으로 인증 없이 호출하며 읽기 전용이라 상태를 바꾸지 않는다. 프로젝트 범위 지정 시 식별자든 사람이 읽는 키든 모두 받아 동일하게 해석한다. 한 프로젝트 안에서 활성 플랜은 동시에 하나만 존재할 수 있다는 불변식이 적용되므로, 상태를 active로 좁히면 프로젝트당 최대 하나가 돌아오는 것이 정상이다(이 단일성은 목록이 아니라 플랜 승인·활성화 쪽에서 강제된다).

## provenance
데몬 라우터의 `/plans` GET 핸들러로 구현되며(`daemon/src/routes/plans.rs`), 프로젝트 참조 해석 후 상태 필터를 적용해 플랜 저장소의 목록 조회를 노출한다. 제공 주체는 데몬 컴포넌트.

## 미확정 (OPEN)
- [ ] OPEN: 기본 정렬 기준(생성순/갱신순 등)과 다건일 때의 페이징 정책 확인 필요 — 판정은 owner.
