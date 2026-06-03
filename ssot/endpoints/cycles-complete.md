---
id: endpoint.cycles-complete
kind: Endpoint
title: 사이클 완료 (POST /cycles/{id}/complete)
definition: 진행 중인 사이클을 완료(completed) 상태로 마무리하는 전용 통로. 완료된 사이클은 재시작할 수 없으므로 이 전환은 그 사이클의 작업 기간을 닫는 종결 행위다.
realizedBy:
  - component.daemon
implementedIn:
  - daemon/src/routes/cycles.rs
relatesTo:
  - to: concept.cycle
    type: mutates
    note: 사이클 상태를 완료로 바꿔 작업 기간을 닫는다
governedBy:
  - invariant.one-active-cycle-per-unit
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
사이클을 "이 스프린트는 끝났다"고 닫는 통로다. 사이클은 한 번 완료되면 다시 진행 중으로 되살릴 수 없고, 새 작업 기간이 필요하면 새 사이클을 만들어야 한다. 그래서 이 완료 처리는 단순한 상태 변경이 아니라 그 유닛의 한 작업 기간을 영구히 종결하는 행위다. 완료하면 그 유닛에서 다음 사이클을 진행 중으로 올릴 자리가 비게 된다(한 유닛에 진행 중 사이클은 하나뿐이므로).

## 요청 / 응답
보내는 정보는 대상 사이클의 식별자뿐이다(본문 없음). 돌려받는 것은 완료로 바뀐 사이클의 최신 상태다. 대상 사이클이 없으면 "찾을 수 없음"으로 응답한다. 완료가 실제로 일어나면 데몬은 사이클이 갱신되었다는 실시간 이벤트를 흘려보내 화면과 관찰자에게 즉시 반영한다.

## 권한 / 제약
데몬이 로컬에만 바인딩되므로 같은 기기 안에서만 호출된다. 한 유닛에는 진행 중 사이클이 하나뿐이라는 규칙(invariant.one-active-cycle-per-unit)과 맞물려, 완료는 그 유닛의 진행 중 슬롯을 비워 다음 사이클 활성화의 길을 연다. 완료된 사이클은 재시작이 불가능하다는 것이 이 통로의 의미상 전제이며, 재개가 필요하면 새 사이클을 만든다.

## provenance
근거는 `daemon/src/routes/cycles.rs`의 `/cycles/{id}/complete` POST 라우트(`complete` 핸들러)다. 전환 성공 시 `cycle:updated` 이벤트를 emit하고, 대상이 없으면 404로 응답한다. 제공 주체는 로컬 데몬(component.daemon), 소비 주체는 웹 대시보드(component.web-dashboard)와 `clawket cycle complete`(endpoint.cli-cycle)다.

## 미확정 (OPEN)
- [ ] OPEN: 완료 시 미완료 태스크(todo/in_progress)가 남아 있을 때의 처리(완료 차단 vs 그대로 종결 vs 백로그 회수)는 사이클 저장소의 complete 구현을 정독해 확인 필요.
