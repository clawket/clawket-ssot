---
id: endpoint.project-timeline
kind: Endpoint
title: 웹 프로젝트 타임라인 API
definition: 웹 대시보드가 한 프로젝트의 활동 이벤트 스트림(상태변경·코멘트·지식·실행·질문·생성·수정·배정)을 개수·시작위치·종류로 걸러 조회하는 API 표면이다.
realizedBy: [component.web-api-client]
implementedIn: [web/src/api.ts, web/src/components/TimelineView.tsx]
relatesTo:
  - { to: concept.project, type: reads, note: 특정 프로젝트 범위의 활동 이력을 읽어온다 }
  - { to: concept.run, type: reads, note: 실행 시작·종료 이벤트가 타임라인 항목으로 노출된다 }
  - { to: concept.comment, type: reads, note: 코멘트 추가가 타임라인 항목으로 노출된다 }
governedBy: []
impacts: [endpoint.daemon-timeline]
consumedBy: [screen.web-timeline]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
프로젝트 안에서 일어난 일들을 시간순 이벤트 흐름으로 보여주기 위한 조회 경로다. 사람 운영자가 타임라인 화면에서 "이 프로젝트에서 최근 무슨 일이 있었나"를 훑을 때, 이 호출이 데몬에 프로젝트 식별자와 함께 페이지네이션·필터 조건을 보내 활동 목록을 받아온다.

## 요청 / 응답
- 보내는 정보: 어느 프로젝트인지(식별자), 한 번에 몇 건을 받을지(개수), 어디서부터 받을지(시작 위치), 어떤 종류의 이벤트만 볼지(종류 필터).
- 받는 정보: 활동 이벤트 목록. 각 이벤트는 그 일이 무슨 종류였는지(상태변경·코멘트·지식 추가·실행 시작/종료·질문·생성·수정·배정 중 하나), 어떤 대상에서 일어났는지(태스크·유닛·사이클·플랜 중 무엇이며 그 제목), 누가 했는지(행위자), 언제(시각), 그리고 종류별 상세(예: 상태변경이면 어떤 필드가 무엇에서 무엇으로 바뀌었는지, 실행이면 어떤 에이전트가 얼마나 걸려 어떤 결과였는지)를 담는다.

## 권한 / 제약
데몬 세션 쿠키/토큰으로 인증되는 로컬 전용 읽기 경로다. 종류 필터·개수·시작 위치는 모두 선택값이며, 생략하면 데몬 기본값으로 동작한다. 이벤트는 읽기 전용으로, 이 경로로는 활동을 만들거나 바꾸지 않는다(활동 기록 자체는 다른 경로에서 발생한다).

## provenance
구현 위치는 web/src/api.ts 의 listProjectTimeline 와 이를 렌더링하는 web/src/components/TimelineView.tsx 다. 이벤트 종류·대상 종류·상세 항목의 의미는 같은 웹 패키지의 TimelineEvent 모양에서 역추론했다. 실제 이벤트 집계·정렬은 데몬의 타임라인 엔드포인트(endpoint.daemon-timeline)가 수행하며, 이 표면은 그 위의 브라우저 호출 계층(component.web-api-client)이다.

## 미확정 (OPEN)
- [ ] OPEN: 종류 필터의 정확한 입력 형식(콤마 결합 등)과 기본 개수·정렬 기준은 데몬 측 구현에서 확인 필요.
