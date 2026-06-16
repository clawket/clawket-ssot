---
id: endpoint.tasks
kind: Endpoint
title: 웹 태스크 API (목록·조회·생성·수정·삭제·검색·일괄수정·본문추가·라벨)
definition: 웹 대시보드가 태스크를 조회·생성·수정·삭제·검색하고 여러 태스크를 한 번에 옮기거나 본문을 덧붙이고 라벨을 더하거나 빼기 위해 데몬에 요청하는 API 표면이다.
realizedBy: [component.web-api-client]
implementedIn: [web/src/api.ts]
relatesTo:
  - { to: concept.task, type: mutates, note: 태스크의 생성·수정·삭제·일괄수정·본문추가·라벨편집을 수행한다 }
  - { to: concept.task-evidence, type: mutates, note: done 전환 시 evidence 를 함께 보내야 한다 }
  - { to: concept.task-tier, type: mutates, note: 태스크 모델 등급(low/med/high)을 설정한다 }
  - { to: concept.backlog, type: reads, note: 사이클 미배정 태스크가 백로그로 조회·이동된다 }
governedBy: []
impacts: [endpoint.daemon-tasks]
consumedBy: [screen.web-board, screen.web-backlog, screen.task-detail-drawer]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
태스크(직접 다루는 유일한 원자적 작업 단위)를 웹에서 다루기 위한 가장 두꺼운 데이터 접근 묶음이다. 사람 운영자가 칸반 보드에서 태스크를 옮기거나, 백로그에서 사이클로 끌어다 놓거나, 태스크 상세에서 내용을 고치고 라벨을 붙이고 결과 근거를 넣어 완료 처리할 때 이 경로로 데몬과 대화한다.

## 요청 / 응답
- 목록 조회: 유닛·플랜·상태·배치(서브에이전트 그룹) 또는 상위 태스크로 걸러 태스크 목록을 받는다. 각 항목은 제목·본문·상태·우선순위·담당·모델 등급·라벨·의존 관계·소속 유닛/사이클 정보를 담는다.
- 단건 조회·검색: 식별자로 한 건을, 또는 검색어와 개수 제한으로 일치하는 태스크들을 받는다.
- 생성: 소속 유닛·사이클·순서·제목·본문과, 선택적으로 담당·의존 태스크·상위 태스크를 보내고 만들어진 태스크를 받는다.
- 수정: 바꿀 것만 부분적으로 보낸다 — 제목·본문·상태·우선순위·담당·모델 등급·라벨·결과 근거·예상 편집량·상위 태스크·소속 유닛/사이클·의존 관계, 그리고 이 변경에 대한 감사용 코멘트를 곁들일 수 있다. 비우기(지움)와 생략(유지)을 구분해 보낸다.
- 일괄 수정: 여러 태스크 식별자를 한꺼번에 보내 상태·소속 사이클·소속 유닛·담당을 동시에 바꾼다(보드 드래그·백로그 사이클 배정에 쓰임).
- 본문 추가: 기존 본문을 지우지 않고 텍스트 한 덩이를 이어 붙인다.
- 라벨 추가/제거: 라벨 하나를 더하거나 빼고 갱신된 태스크를 받는다.
- 삭제: 식별자로 삭제하며 본문 없는 성공 응답을 받는다.

## 권한 / 제약
데몬 세션 쿠키/토큰으로 인증되는 로컬 전용 경로다. 완료(done) 전환은 데몬의 근거 필수 규칙에 막혀 있어, 같은 수정 요청에 비어 있지 않은 결과 근거(evidence)를 반드시 함께 보내야 한다. 부분 수정의 nullable 필드는 비우기와 유지를 구분하는 3-상태 표현을 쓴다. 태스크 시작 자체는 상위 플랜이 active이고 사이클에 배정돼 있어야 하는 등 워크플로우 게이트의 적용을 받으며, 그 불변식은 데몬·훅이 강제한다.

## provenance
구현 위치는 web/src/api.ts 의 태스크 관련 호출(listTasks/listChildTasks/getTask/createTask/updateTask/deleteTask/bulkUpdateTasks/appendTaskBody/searchTasks/addTaskLabel/removeTaskLabel)이다. done 전환의 근거 필수 규칙·3-상태 nullable 규칙은 같은 파일의 UpdateTaskPatch 주석에서 역추론했다. 제공 컴포넌트는 component.web-api-client, 실제 검증·상태 전이는 데몬의 태스크 엔드포인트(endpoint.daemon-tasks)가 수행한다.

## 미확정 (OPEN)
- [ ] OPEN: 검색의 매칭 방식(키워드/의미/하이브리드)과 기본 개수, 일괄 수정 시 일부 실패의 원자성 보장 여부는 데몬 측 구현에서 확인 필요.
