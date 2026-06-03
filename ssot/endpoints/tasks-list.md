---
id: endpoint.tasks-list
kind: Endpoint
title: 데몬 GET /tasks
definition: 다양한 필터 조건으로 태스크 목록을 조회하는 데몬 HTTP 엔드포인트.
realizedBy: [component.daemon]
implementedIn:
  - daemon/src/routes/tasks.rs
relatesTo:
  - { to: concept.task, type: reads, note: "조건에 맞는 태스크 집합을 페이지 단위로 반환한다" }
  - { to: concept.cycle, type: reads, note: "사이클 배정 여부로 백로그/편성 태스크를 갈라 조회할 수 있다" }
governedBy: []
impacts: []
consumedBy: [endpoint.cli-task, component.web-dashboard]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

태스크를 여러 조건으로 걸러서 한 번에 가져오는 핵심 조회 기능이다. 칸반 보드·백로그 화면을 채우거나, CLI에서 "이 플랜의 진행 중 태스크" "이 담당자의 할 일" 같은 목록을 뽑을 때 모두 이 조회를 바탕으로 한다. 작업 관리 전반에서 가장 자주 쓰이는 읽기 경로다.

## 요청 / 응답

요청에는 좁히고 싶은 조건들을 자유롭게 조합해 넘긴다 — 어느 유닛/플랜/사이클에 속하는지, 상태(할 일/진행 중/완료/차단/취소)가 무엇인지, 담당자·실행 에이전트·상위 태스크가 무엇인지, 모델 등급(낮음/중간/높음)이나 검증 결과(통과/결함/시나리오오류)가 무엇인지, 어떤 시나리오·검증 배치에 묶이는지 등이다. 사이클 조건에는 "사이클에 아직 배정되지 않은 것만"이라는 백로그 지정도 가능하다. 결과가 많을 때를 대비해 가져올 개수와 건너뛸 개수(페이지네이션)도 줄 수 있다. 응답은 조건에 맞는 태스크 목록이며, 각 항목은 제목·상태·담당·관계 등 태스크의 일반 속성을 담는다.

## 권한 / 제약

로컬 데몬에 한정된 읽기 전용 조회다. 아무 조건도 주지 않으면 전체 태스크가 대상이 되며, 개수 제한을 주지 않으면 제한 없이 반환된다(페이지네이션은 선택). 사이클 조건을 "비어 있음/없음"으로 주면 사이클 미배정(백로그) 태스크만 골라내는 특수 모드로 동작한다.

## provenance

`daemon/src/routes/tasks.rs`의 `/tasks` 라우트가 질의 파라미터를 태스크 저장소의 목록 필터로 변환해 조회한다. 제공 주체는 clawketd 데몬(component.daemon)이고, CLI의 task 명령과 웹 대시보드가 소비한다.

## 미확정 (OPEN)
- [ ] OPEN: 기본 정렬 기준과 각 필터의 결합 의미(AND/OR)를 저장소 계층에서 확인해 보강할 것.
