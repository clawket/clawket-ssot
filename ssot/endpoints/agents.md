---
id: endpoint.agents
kind: Endpoint
title: 데몬 GET /agents
definition: 지금까지 작업 실행에 참여한 에이전트(작업 주체)들의 이름 목록을 조회한다.
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/agents.rs]
relatesTo:
  - { to: concept.run, type: reads, note: "실행 기록(run)에 남은 agent 값을 출처로 삼아 목록을 만든다" }
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
작업 실행 기록에 한 번이라도 이름을 남긴 에이전트(작업을 수행한 주체)들의 명단을 돌려주는 조회용 엔드포인트다. 어떤 에이전트들이 이 워크스페이스에서 활동했는지 한눈에 파악하거나, 에이전트 단위로 작업을 필터링·집계하려는 화면·도구가 출발점으로 사용한다. 상태를 바꾸지 않는 순수 읽기 동작이다.

## 요청 / 응답
- 요청: 별도의 입력 없이 호출한다. 범위 필터나 페이지 구분은 두지 않는다.
- 응답: 에이전트 이름의 단순 목록을 돌려준다. 같은 이름은 한 번만 나오도록 중복을 제거하고 이름순으로 정렬한다. 이름이 비어 있는(미기재) 실행은 목록에서 제외한다. 아직 어떤 실행도 기록되지 않았다면 빈 목록을 돌려준다.

## 권한 / 제약
- 로컬 데몬에 붙은 클라이언트(웹 대시보드·CLI 등)가 호출하는 읽기 전용 엔드포인트다.
- 에이전트는 별도로 등록·관리되는 엔티티가 아니라 "실행 기록에 등장한 이름"을 그대로 모아 보여주는 파생 정보다. 따라서 실행 기록이 하나도 없으면 명단도 비어 있고, 실행이 삭제·정리되면 명단도 따라 줄어든다.

## provenance
- 라우터/핸들러: `daemon/src/routes/agents.rs`
- 제공 컴포넌트: 로컬 데몬(`component.daemon`)

## 미확정 (OPEN)
- [ ] OPEN: 에이전트별 상세(유형·최근 활동 시각 등)나 상태 필드를 추가로 노출할 계획이 있는지(현재는 이름만 반환). owner 확인 필요.
