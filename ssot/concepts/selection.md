---
id: concept.selection
kind: Concept
title: Selection(현재 선택)
definition: "데스크탑/웹 셸이 들고 있는 '지금 사용자가 고른 엔티티 하나'를 가리키는 전역 상태. 어떤 엔티티(plan·unit·task·cycle)가, 어떤 종류로 선택되었는지를 기억해 상세 패널(드로어)을 무엇으로 채울지 결정한다."
relatesTo:
  - to: concept.plan
    type: reads
    note: 선택 대상이 될 수 있는 엔티티 종류 중 하나
  - to: concept.unit
    type: reads
    note: 선택 대상이 될 수 있는 엔티티 종류 중 하나
  - to: concept.task
    type: reads
    note: 선택 대상이 될 수 있는 엔티티 종류 중 하나
  - to: concept.cycle
    type: reads
    note: 트리 행은 아니지만 유닛 상세에서 파생 선택될 수 있는 엔티티
  - to: concept.active-project
    type: depends-on
    note: 활성 프로젝트가 바뀌면 이전 선택은 무효가 되어 비워진다
governedBy: []
governs: []
dependsOn:
  - concept.active-project
implementedIn:
  - desktop/apps/desktop/src/shell/selection.tsx
  - desktop/apps/desktop/src/shell/DetailDrawer.tsx
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:concept
---

<!-- 작성 고도: 자연어 우선. 식별자는 provenance/근거에만. -->

## 정의

Selection(현재 선택)은 데스크탑/웹 셸이 화면 전역에서 단 하나만 유지하는 "지금 사용자가 클릭해 고른 항목"이다. 두 가지를 함께 기억한다 — 고른 항목의 식별자와, 그 항목이 어떤 종류(plan·unit·task·cycle)인지다. 이 종류 정보가 필요한 이유는, 같은 식별자라도 무엇으로 취급하느냐에 따라 상세 화면이 완전히 달라지기 때문이다. 즉 Selection은 "상세 드로어를 열지 말지, 연다면 무엇으로 채울지"를 결정하는 단일 기준점이다.

선택 가능한 종류는 작업 계층의 네 엔티티다. 그중 plan·unit·task는 화면 좌측 트리에 행으로 나타나는 항목이고, cycle은 트리 행이 아니라 여러 유닛을 가로지르는 개념이지만, 유닛 상세에서 사이클 패널로 이어 들어갈 수 있어 선택 종류에 함께 포함되어 있다. 선택을 거는 동작(고르기)과 비우는 동작(닫기) 두 가지만 존재하며, 상세 드로어를 닫거나 Escape 키를 누르면 선택이 비워진다.

Selection은 활성 프로젝트(`concept.active-project`)에 종속한다. 사용자가 보고 있는 프로젝트를 다른 프로젝트로 바꾸면, 이전에 골라 둔 항목은 옛 프로젝트에 속한 것이므로 그대로 상세 드로어에 남아 있으면 안 된다. 그래서 활성 프로젝트가 실제로 다른 값으로 바뀌는 순간 선택은 자동으로 비워진다(앱이 처음 떠서 프로젝트가 비어 있다가 처음 채워지는 전환은 이미 선택이 없는 상태이므로 불필요하게 비우지 않는다).

## 엔티티 (DB)

Selection은 데이터베이스에 저장되는 비즈니스 엔티티가 아니라 화면 셸이 메모리에 들고 있는 휘발성 UI 상태다. 별도 테이블/컬럼이 없으며, 앱을 새로 열면 선택은 비어 있는 상태에서 시작한다. (상세 드로어의 폭처럼 사용자 편의를 위해 로컬에 보존되는 값과 달리, 어떤 항목을 골랐는지 자체는 저장되지 않는다.)

## API 표면

Selection은 클라이언트 측 상태이므로 데몬 API를 직접 가지지 않는다. 다만 선택된 항목의 상세 내용을 채우기 위해 셸은 이미 메모리에 로드된 plan·unit·task·cycle 목록에서 해당 식별자를 찾아 표시하며, 그 목록 자체는 활성 프로젝트 기준으로 데몬에서 가져온 데이터다. 선택과 무관하게 데이터 갱신은 데몬의 실시간 이벤트 스트림(`endpoint.daemon-events`)으로 반영된다.

## 불변식

- 동시에 선택될 수 있는 항목은 최대 하나다(식별자와 종류가 한 쌍으로만 유지된다).
- 선택을 표현하려면 식별자와 종류가 함께 정해져야 한다 — 종류 없이 식별자만으로는 어떤 상세를 열지 결정할 수 없다.
- 활성 프로젝트가 다른 값으로 바뀌면 기존 선택은 반드시 비워진다(옛 프로젝트 항목이 새 프로젝트 화면에 남지 않도록).

## 구현 위치 (provenance)

선택 상태의 정의·고르기/비우기 동작·프로젝트 전환 시 자동 비움 로직은 셸의 selection 컨텍스트에 있고, 그 선택을 읽어 상세 패널을 열고 채우며 닫기/Escape로 선택을 비우는 소비처는 상세 드로어 컴포넌트에 있다.

## 미확정 (OPEN)
- [ ] OPEN: 웹 빌드(데스크탑 외 표면)에서 Selection 동작이 동일하게 적용되는지 — 본 노드는 데스크탑 셸 구현을 근거로 작성됨.
