---
id: endpoint.tasks-envelope-history
kind: Endpoint
title: 태스크 봉투 버전 이력 조회 (GET /tasks/{id}/envelope/history)
definition: 태스크에 서명되어 온 실행 봉투의 모든 버전을 최신순으로, 각 버전이 언제 누가 서명하고 언제 대체되었는지와 함께 돌려준다.
realizedBy:
  - component.daemon
implementedIn:
  - daemon/src/routes/tasks.rs
relatesTo:
  - to: concept.execution-envelope
    type: reads
    note: 한 태스크에 누적된 봉투 버전들을 버전·서명자·대체 시점과 함께 읽는다
  - to: concept.task
    type: reads
    note: 대상 태스크를 확인한 뒤 그 태스크의 봉투 이력을 조회한다
governedBy: []
governs: []
dependsOn: []
consumesApi: []
providesApi: []
impacts:
  - domain.work-management
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의
한 태스크의 실행 봉투가 시간에 따라 어떻게 바뀌어 왔는지(재계획 이력)를 보여 주는 조회 기능이다. 봉투는 수정될 때마다 이전 버전을 대체(supersede)하며 버전을 올려 보존하므로, 이 기능은 그 누적된 버전 전체를 최신 버전부터 거슬러 보여 준다. "이 작업의 계약이 몇 번 바뀌었나, 언제 어떻게 재계획됐나"를 추적하거나, 특정 시점의 봉투 내용을 되짚을 때 쓰인다.

## 요청 / 응답
요청에는 대상 태스크 식별자와 함께 선택적으로 페이지네이션을 위한 개수와 시작 위치를 지정할 수 있다. 개수는 기본 50건이며 1~500건 범위로 제한되고, 시작 위치는 0 이상으로 보정된다.

응답은 봉투 버전들의 목록으로, 최신 버전이 앞에 온다. 각 항목은 그 봉투의 식별자, 버전 번호, 서명된 시각, 서명 주체, 그리고 (이미 대체된 버전이면) 대체된 시각과 함께 해당 버전의 봉투 내용 전체를 담는다.

## 권한 / 제약
로컬 데몬 호출자가 사용한다. 존재하지 않는 태스크를 가리키면 찾을 수 없음으로 응답한다. 개수·시작 위치는 위 범위로 자동 보정되어, 비정상 입력에도 안전한 페이지네이션이 보장된다.

## provenance
구현은 데몬의 태스크 라우터(`daemon/src/routes/tasks.rs`)의 봉투 이력 핸들러에 있으며, 태스크별 봉투 버전 저장소에서 버전·서명자·대체 시점을 페이지 단위로 읽어 최신순으로 직렬화한다. 봉투 버전을 보존·제공하는 컴포넌트는 데몬(component.daemon)이다.

## 미확정 (OPEN)
- [ ] OPEN: 봉투 이력이 웹 대시보드(예: 봉투 비교/리플레이 화면)에서 어떻게 소비되는지(consumedBy 확정) 확인 필요.
