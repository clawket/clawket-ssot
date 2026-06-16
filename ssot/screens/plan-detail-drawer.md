---
id: screen.plan-detail-drawer
kind: Screen
title: 플랜 상세 드로어
purpose: 운영자가 하나의 플랜을 열어 출처·설명·승인 시점과 그 아래 유닛·태스크 트리를 한눈에 보고, 구성 요소로 내려가며 로드맵을 파악한다.
servesPersona:
  - persona.human-operator
realizedBy:
  - component.web-dashboard
implementedIn:
  - web/src/components/PlanDetail.tsx
  - web/src/App.tsx
consumesApi: []
relatesTo:
  - to: concept.plan
    type: relates-to
    note: 이 드로어가 표현·편집하는 1차 대상 엔티티.
  - to: concept.unit
    type: reads
    note: 플랜에 속한 유닛을 순번 순으로 나열하고 클릭 시 유닛 상세로 이동.
  - to: concept.task
    type: reads
    note: 각 유닛 아래 중첩된 태스크를 트리로 보여주고 클릭 시 태스크 상세로 이동.
  - to: concept.question
    type: reads
    note: 플랜에 달린 질문·답변을 섹션으로 노출.
impacts:
  - concept.plan
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:screen
---

<!-- 본문 섹션은 x-required-sections-by-kind.Screen 와 1:1 정렬. -->

## 화면 목적

운영자가 플랜 하나를 선택하면 우측 드로어로 열려, 그 플랜이 어디서 왔고(출처·경로), 무엇을 하려는지(설명), 언제 승인됐는지, 그리고 어떤 유닛·태스크로 구성돼 있는지를 트리 형태로 보여준다. 플랜은 승인된 의도를 담는 컨테이너(로드맵)이므로, 이 화면은 그 로드맵의 골격 — 유닛 단위로 묶이고 각 유닛 아래 태스크가 매달린 구조 — 을 펼쳐 보여주는 동시에, 구성 요소 어디로든 클릭으로 내려가는 진입점이 된다.

## UI 요소 / 입력 필드

- **상단 줄**: 플랜 ID 앞자리, 상태 배지, 완료 전이면 노출되는 편집(Edit) 버튼, 닫기.
- **경로 이동(브레드크럼)**: 현재 플랜 단계.
- **제목**.
- **출처(Source)·경로(Path)**: 플랜이 어떤 경로로 들어왔는지(있을 때) 표시.
- **설명(Description)**: 있을 때만 텍스트 블록.
- **타임스탬프**: 생성·승인 시각.
- **유닛 개요(트리)**: 유닛을 순번 순으로 나열하고 각 유닛에 속한 태스크 개수를 표시. 유닛 줄 클릭 시 유닛 상세로 이동하며, 그 아래 중첩 목록으로 소속 태스크(상태 배지·티켓 번호·제목)를 보여주고 클릭 시 태스크 상세로 이동.
- **아티팩트·질문** 섹션(있을 때만).
- **편집 모달**: 플랜 메타를 수정.

## 표시 데이터 / 호출 API

진입 시 플랜 본체와 그 플랜에 속한 유닛·태스크·아티팩트·질문을 함께 불러온다. 유닛과 태스크는 각각 순번 순으로 정렬하고, 유닛별로 unit_id가 일치하는 태스크를 묶어 중첩 트리를 구성한다. 편집은 플랜 갱신 호출로 반영된다.

## 상태 / 엣지케이스

- **로딩 중 / 플랜 미해결**: "Loading...".
- **유닛 없음**: "No units yet" 안내.
- **태스크 없는 유닛**: 중첩 목록을 생략하고 "0 tasks"만 표시.
- **완료된 플랜**: 편집 버튼을 숨겨 사후 수정을 막는다.
- **이동 핸들러 부재**: onSelectItem이 없으면 유닛·태스크 줄이 비활성(클릭 불가)으로 표시.
- **아티팩트/질문 없음**: 해당 섹션 자체를 렌더링하지 않음.

## 미확정 (OPEN)
- [ ] OPEN: 플랜·유닛·태스크 로딩이 거치는 데몬 HTTP endpoint 노드 매핑(현재 SSOT에 해당 조회 endpoint 노드 부재로 consumesApi 비움).
