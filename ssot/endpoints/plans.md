---
id: endpoint.plans
kind: Endpoint
title: 웹 플랜 API (목록·조회·생성·수정·삭제·승인·유닛별 집계)
definition: 웹 대시보드가 플랜을 조회·생성·수정·삭제하고 승인으로 active 전환하며 플랜 안 유닛별 태스크 개수를 집계 조회하는 API 표면이다.
realizedBy: [component.web-api-client]
implementedIn: [web/src/api.ts]
relatesTo:
  - { to: concept.plan, type: mutates, note: 플랜의 생성·수정·삭제·승인(active 전환)을 수행한다 }
  - { to: concept.unit, type: reads, note: 플랜의 유닛별 태스크 상태 개수를 집계해 읽어온다 }
governedBy: []
impacts: [endpoint.daemon-plans]
consumedBy: [screen.web-plans-tree, screen.plan-detail-drawer, screen.web-summary]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
플랜(승인된 의도를 담는 로드맵 컨테이너)을 웹에서 다루기 위한 데이터 접근 묶음이다. 사람 운영자가 플랜 트리·플랜 상세 화면에서 플랜을 만들고 다듬고 승인할 때, 그리고 플랜 한눈 보기에서 유닛별 진행 분포(할 일/진행 중/막힘/완료/취소)를 볼 때 이 경로가 데몬과 대화한다.

## 요청 / 응답
- 목록 조회: 프로젝트 범위나 상태로 걸러 플랜 목록을 받는다. 각 항목은 제목·설명·상태(초안/활성/완료)와 소속 프로젝트 정보를 담는다.
- 단건 조회: 플랜 식별자로 그 플랜의 같은 정보를 받는다.
- 생성: 소속 프로젝트·제목과, 선택적으로 설명·출처(어디서 온 플랜인지)·출처 경로를 보내고 만들어진 플랜을 받는다.
- 수정: 제목·설명·상태 중 바꿀 것만 부분적으로 보낸다.
- 삭제: 플랜 식별자로 삭제하며 본문 없는 성공 응답을 받는다.
- 승인: 플랜을 active로 전환한다 — 이 전환이 일어나야 그 플랜 아래 태스크를 시작할 수 있다.
- 유닛별 집계: 플랜 안 각 유닛마다 태스크 상태별 개수(할 일·진행 중·막힘·완료·취소·합계)를 한 번에 받는다. 데몬에 이 집계 경로가 아직 없으면 호출 측은 유닛별 태스크를 따로 세는 방식으로 대체한다.

## 권한 / 제약
데몬 세션 쿠키/토큰으로 인증되는 로컬 전용 경로다. 승인은 단순 상태 변경이 아니라 워크플로우 게이트로, plan이 active여야 하위 태스크 시작이 허용된다(불변식은 데몬·훅이 강제). 유닛별 집계 조회는 데몬 측 미구현 시 404/405를 정상적인 "없음"으로 받아들여 호출 측이 대체 경로로 폴백하도록 설계됐다.

## provenance
구현 위치는 web/src/api.ts 의 플랜 관련 호출(listPlans/getPlan/createPlan/updatePlan/deletePlan/approvePlan/getPlanCounts)이다. 제공 컴포넌트는 component.web-api-client, 실제 상태 전이·집계는 데몬의 플랜 엔드포인트(endpoint.daemon-plans)가 수행한다. 유닛별 집계의 폴백 동작은 같은 파일 주석에서 역추론했다.

## 미확정 (OPEN)
- [ ] OPEN: 플랜 출처(source)의 허용 값 집합, 승인 시 부수 효과(유닛/사이클 자동 생성 여부)는 데몬 측 구현에서 확인 필요.
