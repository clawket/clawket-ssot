---
id: endpoint.units
kind: Endpoint
title: 웹 유닛 API (목록·조회·생성·수정·삭제·승인)
definition: 웹 대시보드가 플랜 안 유닛을 조회·생성·수정·삭제하고 승인 처리하기 위해 데몬에 요청하는 API 표면이다.
realizedBy: [component.web-api-client]
implementedIn: [web/src/api.ts]
relatesTo:
  - { to: concept.unit, type: mutates, note: 유닛의 생성·수정·삭제·승인을 수행한다 }
  - { to: concept.plan, type: reads, note: 유닛은 플랜에 속하므로 플랜 범위로 조회된다 }
governedBy: []
impacts: [endpoint.daemon-units]
consumedBy: [screen.web-plans-tree, screen.unit-detail-drawer]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
유닛(플랜 안에서 태스크를 묶는 그룹핑 단위)을 웹에서 다루기 위한 데이터 접근 묶음이다. 사람 운영자가 플랜 트리나 유닛 상세 화면에서 유닛을 만들고 제목·목표를 다듬고 정리할 때 이 경로로 데몬과 대화한다.

## 요청 / 응답
- 목록 조회: 어느 플랜에 속하는지(플랜 범위)나 상태로 걸러 유닛 목록을 받는다. 각 항목은 제목·목표·플랜 내 순서·소속 플랜 정보를 담는다.
- 단건 조회: 유닛 식별자로 그 유닛의 같은 정보를 받는다.
- 생성: 소속 플랜·플랜 내 순서·제목과, 선택적으로 목표·승인 필요 여부를 보내고 만들어진 유닛을 받는다.
- 수정: 제목·목표 중 바꿀 것만 부분적으로 보낸다.
- 삭제: 유닛 식별자로 삭제하며 본문 없는 성공 응답을 받는다.
- 승인: 유닛을 승인 처리하며, 선택적으로 승인자 정보를 함께 보낸다.

## 권한 / 제약
데몬 세션 쿠키/토큰으로 인증되는 로컬 전용 경로다. 유닛 자체는 순수 그룹핑 엔티티라 상태 워크플로우의 게이트 역할은 작지만, 웹 표면에는 승인 호출 경로가 남아 있어 승인 필요 여부 설정·승인자 전달이 가능하다. 병렬 유닛의 태스크는 여러 에이전트가 동시에 실행할 수 있다. 실제 검증·정합성은 데몬 측 유닛 엔드포인트가 책임진다.

## provenance
구현 위치는 web/src/api.ts 의 유닛 관련 호출(listUnits/getUnit/createUnit/updateUnit/deleteUnit/approveUnit)이다. 제공 컴포넌트는 component.web-api-client, 실제 저장·검증은 데몬의 유닛 엔드포인트(endpoint.daemon-units)가 수행한다.

## 미확정 (OPEN)
- [ ] OPEN: 유닛 승인의 현행 의미(데몬에서 유닛 상태·승인 컬럼이 제거된 마이그레이션 이후 이 호출이 무동작인지 여전히 유효한지)는 데몬 측 구현에서 확인 필요.
