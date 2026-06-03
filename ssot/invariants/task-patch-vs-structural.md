---
id: invariant.task-patch-vs-structural
kind: Invariant
title: task 이벤트는 제자리 패치, 그 외 엔티티 이벤트는 구조 새로고침
definition: 실시간 이벤트 스트림에서 task 변경은 화면 상태를 제자리에서 부분 갱신(patch)하지만, unit·plan·cycle·knowledge·comment 변경은 트리 전체를 다시 그리는 구조적 새로고침을 트리거한다.
governs:
  - component.web-dashboard
  - endpoint.daemon-events
  - concept.task
  - concept.unit
  - concept.plan
  - concept.cycle
  - concept.knowledge
  - concept.comment
implementedIn:
  - web/src/App.tsx
decidedBy:
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:invariant
---

<!-- 작성 고도: 자연어. 코드 옮겨적기 금지. -->

## 제약

웹 대시보드는 데몬이 보내는 실시간 변경 이벤트를 두 갈래로 다르게 처리한다.

- **task 이벤트(생성·수정·시작·완료·취소·삭제)** 는 화면 전체를 다시 그리지 않고, 해당 태스크 하나의 상태만 **제자리에서 덮어쓰는 부분 갱신**으로 반영한다. 삭제는 그 항목만 목록에서 제거한다.
- **그 외 엔티티(unit·plan·cycle·knowledge·comment) 이벤트** 는 부분 갱신 경로가 아직 없어, **구조적 새로고침 신호**를 한 단계 올린다. 이 신호가 바뀌면 화면은 트리(플랜→유닛→사이클→태스크 계층)를 통째로 다시 구성한다.

핵심은 이 두 처리 경로가 섞이지 않는다는 점이다. task 는 가볍게 패치, 상위·주변 엔티티는 무겁게 전체 새로고침으로 명확히 나뉜다. 페이로드에 태스크의 식별자와 소속 유닛 정보가 온전히 갖춰지지 않은 경우에는 패치를 포기하고 안전하게 구조적 새로고침으로 떨어진다. 의미를 알 수 없는 keepalive 성격의 이벤트는 데이터 갱신으로 취급하지 않고 무시한다.

## 깨지면 무슨 일이 일어나나

이 분기 규칙이 무너지면 두 방향의 오류가 생긴다.

- task 가 아닌 변경(예: 새 유닛 추가, 사이클 활성화)을 패치 경로로 잘못 흘리면, 부분 갱신할 자리가 없어 화면의 계층 구조가 실제 상태와 어긋난 채 남는다(새 유닛이 안 보이거나, 옮겨진 태스크가 옛 위치에 남는 등).
- 반대로 모든 task 변경까지 매번 전체 새로고침으로 처리하면, 빈번한 태스크 상태 변화마다 트리 전체가 다시 그려져 화면이 끊기고 비효율적이 된다.

부분 갱신이 불가능한 불완전 페이로드를 그래도 패치로 강행하면, 잘못된 위치에 항목이 박히거나 누락된다. 그래서 "불완전하면 구조 새로고침으로 폴백" 이 안전망으로 함께 묶여 있다.

## 코드에서 어떻게 강제되나

웹 앱의 이벤트 라우팅 로직(`web/src/App.tsx`)이 엔티티 종류를 보고 분기한다. task 이면 부분 갱신 액션을, unit/plan/cycle/knowledge/comment 이면 구조적 새로고침 신호 증가를 디스패치한다. 구조적 새로고침은 단조 증가하는 시퀀스 값으로 표현되며, 이 값이 트리 렌더링의 키로 쓰여 값이 바뀔 때마다 트리가 다시 구성된다. task 페이로드에 식별자와 소속 유닛 정보가 모두 있을 때만 패치하고, 그렇지 않으면 구조적 새로고침으로 폴백하는 가드가 같은 위치에 들어 있다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(patch/structural 분기 정책의 근거 decision) 연결 필요
