---
id: decision.web-parity
kind: Decision
title: 데스크탑 뷰는 웹 대시보드와 외형을 맞추되 데이터는 한 소스로 흘린다
purpose: "데스크탑 앱과 웹 대시보드가 같은 화면(Board/Timeline/Summary)을 보여줄 때, 외형은 어디까지 똑같이 하고 데이터는 어떻게 가져올 것인가?"
definition: "Board·Timeline·Summary 등 데스크탑 뷰는 웹 대시보드와 시각·구조를 의도적으로 동일하게 유지하되, 데이터는 화면이 직접 데몬 API를 호출하지 않고 단일 데이터 공급 계층(DataProvider)을 통해 받아 SSE 동기화를 한 소스로 묶는다."
relatesTo:
  - to: component.desktop-app
    type: governs
    note: "데스크탑 앱의 뷰 계층이 따르는 외형·데이터 규약"
  - to: component.web-dashboard
    type: relates-to
    note: "외형·구조의 기준이 되는 원본 화면"
  - to: component.data-provider
    type: governs
    note: "화면이 데몬을 직접 부르지 않고 거치는 단일 데이터 공급 계층"
  - to: component.sse-hook
    type: relates-to
    note: "DataProvider가 한 소스로 흡수하는 실시간 동기화 채널"
supersedes: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:decision"]
---

> Decision은 append-only다. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 `supersedes`로 잇는다.

## 맥락 (Context)

clawket은 같은 작업 데이터를 두 곳에서 보여준다. 하나는 브라우저에서 여는 웹 대시보드이고, 다른 하나는 데스크탑 앱이다. 두 곳 모두 Board(칸반), Timeline(스윔레인·활동 스트림), Summary(요약·KPI) 같은 동일한 화면을 제공한다.

이때 두 갈래의 결정이 필요했다. 첫째, 데스크탑 화면을 웹과 얼마나 똑같이 보이게 할 것인가. 둘째, 화면이 보여줄 데이터를 어디서 끌어올 것인가 — 각 화면이 데몬을 직접 호출할 것인가, 아니면 공통 통로를 거칠 것인가. 작업 데이터는 데몬이 실시간으로 흘려보내는(SSE) 변경 알림을 받아 갱신되는데, 화면마다 따로 데몬을 붙잡으면 같은 화면의 여러 뷰가 서로 다른 시점의 데이터를 들고 어긋날 위험이 있다.

## 결정 (Decision)

외형은 의도적으로 웹과 같게 맞춘다. Board는 동일한 4개 컬럼(todo / in_progress / blocked / done)과 같은 헤더 색, 같은 카드 구성(티켓·티어·제목·담당자·우선순위 점·인라인 전환 버튼)으로 그린다. Timeline은 같은 탭 전환(스윔레인 / 활동)과 진행 띠를, Summary는 같은 KPI 구성을 따른다. 즉 사용자가 웹과 데스크탑을 오갈 때 같은 화면이라고 느끼게 한다.

반면 데이터를 가져오는 방식은 웹과 일부러 다르게 둔다. 데스크탑 화면은 데몬 API를 직접 호출하지 않고, 단일 데이터 공급 계층(DataProvider)을 통해서만 데이터를 읽고 변경을 보낸다. DataProvider가 실시간 동기화 알림(SSE)을 받아 메모리에 한 벌의 최신 데이터를 유지하므로, 같은 데이터에 의존하는 여러 뷰가 항상 같은 시점을 본다. 화면 고유의 일부 데이터(예: 특정 사이클의 작업 목록)는 필요할 때만 따로 불러오되, 데이터 변경(작업 상태 전환, 사이클 생성·수정·활성화·완료)은 모두 이 공급 계층을 거친다.

## 근거와 결과 (Consequences)

외형 동일성은 두 클라이언트를 하나의 제품으로 인지하게 하고, 화면 디자인의 단일 기준(웹)을 유지해 두 곳이 따로 어긋나는 것을 막는다.

데이터를 단일 공급 계층으로 모으는 선택은 "한 화면 안의 여러 뷰가 서로 다른 데이터 시점을 들고 어긋나는" 실패를 구조적으로 차단한다(single source of truth). 화면 단위로 데몬을 직접 호출했다면 각 뷰가 독립적으로 갱신되어 칸반과 타임라인이 다른 작업 상태를 보여줄 수 있었을 것이다.

따라오는 제약은, 새 데스크탑 화면을 추가할 때도 같은 규칙을 지켜야 한다는 점이다 — 직접 데몬을 부르는 지름길을 쓰면 이 결정이 무너진다.

## 근거 (Provenance)

- `desktop/apps/desktop/src/views/BoardView.tsx` — 웹 BoardView와의 "Visual + structural parity" 명시, `useData()`(DataProvider) 경유·`api.*` 직접 호출 회피 의도 주석.
- `desktop/apps/desktop/src/views/TimelineView.tsx` — 웹과의 시각·구조 동일성, DataProvider가 runs·timeline을 SSE로 스트리밍해 한 소스를 유지한다는 주석.
- `desktop/apps/desktop/src/views/SummaryView.tsx` — `useData()` 경유 KPI 구성.
