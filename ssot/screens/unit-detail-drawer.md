---
id: screen.unit-detail-drawer
kind: Screen
title: 유닛 상세 드로어
purpose: 운영자가 하나의 유닛을 열어 묶인 태스크들의 진행률·목표·승인 상태를 한눈에 보고, 소속 태스크나 상위 플랜으로 이동한다.
servesPersona:
  - persona.human-operator
realizedBy:
  - component.web-dashboard
implementedIn:
  - web/src/components/UnitDetail.tsx
  - web/src/App.tsx
consumesApi: []
relatesTo:
  - to: concept.unit
    type: relates-to
    note: 이 드로어가 표현·편집하는 1차 대상 엔티티.
  - to: concept.task
    type: reads
    note: 유닛에 묶인 태스크 목록·상태별 집계·진행률을 표시하고 클릭 시 태스크 상세로 이동.
  - to: concept.plan
    type: reads
    note: 상위 플랜으로 거슬러 올라가는 브레드크럼 제공.
  - to: concept.question
    type: reads
    note: 유닛에 달린 질문·답변을 섹션으로 노출.
impacts:
  - concept.unit
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

운영자가 유닛 하나를 선택하면 우측 드로어로 열려, 그 유닛이 무엇을 목표로 하며 어디까지 진행됐는지를 보여준다. 유닛은 상태도 승인 개념의 강제도 없는 순수 그룹핑 엔티티지만, 이 화면은 그 안에 묶인 태스크들의 완료 비율과 상태 분포를 집계해 "이 묶음이 얼마나 끝났는가"를 한눈에 읽게 한다. 동시에 소속 태스크 하나하나로 들어가거나 상위 플랜으로 올라가는 이동 거점이 된다.

## UI 요소 / 입력 필드

- **상단 줄**: 유닛 ID 앞자리, 완료 전이면 노출되는 편집(Edit) 버튼, 닫기. (유닛은 상태 배지가 없다.)
- **경로 이동(브레드크럼)**: 상위 플랜 → 현재 유닛.
- **제목**과 **순번(#index)**.
- **승인 표시**: 승인이 필요한 유닛이면 승인됨(승인자 표기)·필요 중 하나로, 필요 없으면 "Not required"로 읽기 표시.
- **목표(Goal)**: 있을 때만 텍스트 블록으로 표시.
- **타임스탬프**: 생성·시작·완료(승인 시각은 있을 때만).
- **진행률**: 닫힌 상태(완료/취소 등) 태스크 비율을 막대로 보여주고, 상태별 개수를 배지로 나열.
- **태스크 목록**: 상태 배지·티켓 번호·제목·담당자를 줄로 보여주고 클릭 시 해당 태스크 상세로 이동.
- **아티팩트·질문** 섹션(있을 때만).
- **편집 모달**: 유닛 메타를 수정.

## 표시 데이터 / 호출 API

진입 시 유닛 본체와 그 유닛에 속한 태스크·아티팩트·질문을 함께 불러오고, 유닛이 플랜에 속해 있으면 상위 플랜을 추가 조회해 브레드크럼을 채운다. 태스크는 순번 순으로 정렬해 표시하며, 닫힌 상태 집합을 기준으로 완료 개수와 진행률을 계산한다. 편집은 유닛 갱신 호출로 반영된다.

## 상태 / 엣지케이스

- **로딩 중 / 유닛 미해결**: "Loading...".
- **태스크 없음**: 진행률 0%, 상태 배지 없음.
- **상위 플랜 해석 실패**: 브레드크럼이 유닛 단계까지만 축소(비치명).
- **완료된 유닛**: 편집 버튼을 숨겨 사후 수정을 막는다.
- **이동 핸들러 부재**: onSelectItem이 없으면 태스크 줄이 비활성(클릭 불가)으로 표시.
- **아티팩트/질문 없음**: 해당 섹션 자체를 렌더링하지 않음.

## 미확정 (OPEN)
- [ ] OPEN: 유닛 데이터 로딩이 거치는 데몬 HTTP endpoint 노드 매핑(현재 SSOT에 유닛 조회 endpoint 노드 부재로 consumesApi 비움).
