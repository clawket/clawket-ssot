---
id: concept.timeline-event
kind: Concept
title: TimelineEvent (타임라인 이벤트)
definition: 엔티티 변경 이력을 한 건씩 담는 활동 이벤트로, 무슨 일이(이벤트 종류) 어떤 대상에(엔티티) 누가(행위자) 어떤 상세와 함께 일어났는지를 기록해 요약·타임라인 화면의 활동 스트림을 구성한다.
relatesTo:
  - to: screen.timeline
    type: relates-to
    note: Timeline 화면의 활동 스트림 탭이 이 이벤트들을 시간순으로 재생한다.
  - to: screen.summary
    type: relates-to
    note: Summary 화면의 최근 활동 목록이 이 이벤트들로 채워진다.
  - to: concept.task
    type: reads
    note: 이벤트는 작업 등 엔티티의 변경을 대상으로 가리킨다.
governedBy: []
dependsOn:
  - concept.project
implementedIn:
  - desktop/apps/desktop/src/data/types.ts
  - desktop/apps/desktop/src/views/timeline/ActivityStream.tsx
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:concept
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지(식별자는 provenance에만). 분기/의도 불명은 OPEN. -->

## 정의

TimelineEvent는 프로젝트 안에서 일어난 변화 한 건을 기록하는 활동 이벤트다. 각 이벤트는 다섯 가지를 담는다.

- **이벤트 종류** — 어떤 일이 일어났는지(예: 생성·상태 변경 등).
- **대상 엔티티** — 그 일이 어떤 종류의 대상에, 어떤 식별자·제목으로 일어났는지.
- **행위자** — 누가 그 변경을 일으켰는지(사람 또는 에이전트).
- **발생 시각** — 언제 일어났는지.
- **상세** — 변경에 대한 부가 정보(자유로운 키-값 모양의 부속 데이터).

이 이벤트들은 데스크탑에서 "활동 스트림"을 구성한다. 요약 화면은 최근 활동 목록으로 몇 건을 보여주고, 타임라인 화면은 같은 이벤트들을 시간순으로 흐르는 활동 탭으로 재생한다. 즉 TimelineEvent는 "이 프로젝트에서 최근 무슨 일이 있었나"를 사람이 한눈에 따라갈 수 있게 해주는 단위다.

TimelineEvent는 작업 실행 기록(Run)과는 다른 층위다. 타임라인 화면에는 에이전트별 실행 트랙을 보여주는 스윔레인 탭과, 이 이벤트들을 모은 활동 스트림 탭이 함께 있는데, TimelineEvent는 후자(범용 변경 이력)에 해당한다.

## 엔티티 (DB)

TimelineEvent는 엔티티 변경마다 누적되는 이력 기록이다. 한 건은 이벤트 종류, 대상 엔티티의 종류·식별자·제목, 행위자, 발생 시각, 그리고 상세 부속 정보를 함께 담는다. 데스크탑은 이를 프로젝트 단위로 조회해 화면에 흘려보내며, 직접 생성·수정하지 않고 읽기 전용으로 소비한다.

## API 표면

- 프로젝트별 타임라인 이벤트 조회: 개수·오프셋·이벤트 종류 필터를 받아 해당 프로젝트의 이벤트를 시간순으로 반환한다. 데스크탑은 이 결과를 활동 스트림으로 렌더링한다.

## 불변식

- TimelineEvent는 읽기 전용 활동 이력이다 — 데스크탑이 직접 생성·편집하지 않고, 다른 엔티티의 변경에 의해 파생적으로 쌓인다.
- 이벤트는 항상 어떤 대상 엔티티와 발생 시각을 동반해 활동 스트림에서 시간순으로 정렬·재생될 수 있어야 한다.

## 구현 위치 (provenance)

- 이벤트 구조 정의: `desktop/apps/desktop/src/data/types.ts`
- 활동 스트림 렌더링: `desktop/apps/desktop/src/views/timeline/ActivityStream.tsx`

## 미확정 (OPEN)
- [ ] OPEN: 이벤트 종류의 전체 집합과 상세 부속 정보의 표준 스키마(어떤 종류가 어떤 상세를 싣는지) 확인 필요.
