---
id: invariant.sse-payload-id-only
kind: Invariant
title: 실시간 이벤트 본문은 식별자만 담는다 — 변경은 타입별 재조회, 삭제는 로컬 제거
definition: 데몬이 밀어주는 실시간 변경 알림에는 바뀐 엔티티의 식별자만 들어 있다. 그래서 삭제가 아닌 변경 알림을 받으면 해당 종류의 목록을 다시 불러오고, 삭제 알림은 네트워크 없이 화면에서 그 항목만 지운다.
governs:
  - concept.task
  - concept.cycle
  - concept.plan
  - concept.unit
  - concept.knowledge
  - concept.run
  - endpoint.daemon-events
  - component.web-dashboard
implementedIn:
  - desktop/apps/desktop/src/data/DataProvider.tsx
  - desktop/apps/desktop/src/data/hooks/useEvents.ts
decidedBy:
crossesBoundary: true
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:invariant
---

<!-- 작성 고도: 자연어 우선. 식별자는 provenance/근거에만. -->

## 제약

데몬은 무언가 바뀔 때마다 화면으로 실시간 알림을 밀어준다. 이 알림은 "무엇이 어떻게 바뀌었다"는 형식(예: 태스크가 생성됨, 사이클이 갱신됨)으로 오지만, 그 안에 담기는 데이터는 바뀐 대상의 식별자 하나뿐이다. 즉 알림만으로는 그 항목의 최신 내용을 알 수 없다.

그래서 화면은 두 갈래로 반응한다. 삭제가 아닌 변경(생성·갱신·시작·완료 등) 알림을 받으면, 식별자만으로 항목을 복원하려 들지 않고 그 종류(태스크면 태스크, 사이클이면 사이클)의 목록을 통째로 다시 불러와 최신 상태로 맞춘다. 반대로 삭제 알림은 어차피 지울 항목이므로, 서버를 다시 호출하지 않고 화면이 가진 목록에서 해당 식별자의 항목만 즉시 제거한다.

## 깨지면 무슨 일이 일어나나

만약 식별자만 든 변경 알림을 가지고 화면이 항목 내용을 직접 재구성하려 한다면, 알림에 없는 필드를 추측하거나 비워 채우게 되어 화면이 부정확한 상태를 보여준다. 반대로 삭제 알림에도 굳이 목록을 재조회하면, 이미 사라진 항목 때문에 불필요한 네트워크 왕복이 늘고, 삭제 알림과 재조회가 경합하면 지운 항목이 잠깐 되살아나 깜빡이는 현상이 생긴다. "변경은 타입별 재조회, 삭제는 로컬 제거"라는 분기가 무너지면 정확성(재구성 오류)과 반응성(불필요한 왕복) 둘 다 손상된다.

## 코드에서 어떻게 강제되나

실시간 구독 훅은 각 알림이 식별자와 이벤트 이름만 싣는다는 전선 계약을 명시하고, 알려진 이벤트 이름들(태스크·사이클·플랜·유닛·지식·코멘트의 생성/갱신/삭제 등)만 수신하도록 등록한다(`desktop/apps/desktop/src/data/hooks/useEvents.ts`). 데이터 공급 계층은 알림 본문이 식별자뿐이라 "엔티티를 재구성하기보다 한 번의 타입별 재조회가 더 싸다"고 적고, 삭제 이벤트는 식별자로 화면 목록에서 바로 제거하며 그 외 변경 이벤트는 해당 종류 목록을 다시 불러온다고 기록한다(`desktop/apps/desktop/src/data/DataProvider.tsx`). 태스크 변경 시에는 실행 기록과 타임라인도 함께 새로고침하되, 실패해도 다음 알림이나 수동 새로고침으로 복구되도록 부드럽게 처리한다. 이 규칙은 데몬의 알림 형식과 화면의 갱신 전략을 잇는 약속이라 시스템 경계를 가로지른다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요 — 실시간 알림 본문을 식별자만으로 최소화하고 재조회로 보강하는 설계 결정을 decision 노드로 연결.
