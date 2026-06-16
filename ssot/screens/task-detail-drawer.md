---
id: screen.task-detail-drawer
kind: Screen
title: 태스크 상세 드로어
purpose: 운영자가 하나의 태스크를 열어 메타데이터·진행 상황·실행 이력·하위 작업·검증 근거를 한 화면에서 읽고 편집하며, 연결된 다른 태스크로 자유롭게 이동한다.
servesPersona:
  - persona.human-operator
realizedBy:
  - component.web-dashboard
implementedIn:
  - web/src/components/TaskDetail.tsx
  - web/src/components/task-detail/TaskSections.tsx
  - web/src/components/task-detail/TaskSubTasks.tsx
  - web/src/components/task-detail/TaskComments.tsx
  - web/src/App.tsx
consumesApi:
  - endpoint.daemon-search
relatesTo:
  - to: concept.task
    type: relates-to
    note: 이 드로어가 표현·편집하는 1차 대상 엔티티.
  - to: concept.run
    type: reads
    note: 실행 이력 표·타임라인 리플레이·run diff 비교로 표시.
  - to: concept.execution-envelope
    type: reads
    note: 봉투 섹션에서 리프 태스크의 실행 계약을 보여주고 편집.
  - to: concept.question
    type: reads
    note: 사람의 명확화 질문과 답변을 섹션으로 노출.
  - to: concept.comment
    type: reads
    note: 코멘트 섹션으로 히스토리·논의를 보존·표시.
  - to: concept.scenario
    type: reads
    note: scenario_id 메타를 PDD 메타 블록에 표시.
  - to: concept.tsv-evidence
    type: reads
    note: evidence(file:line)를 소스 링크로, batch_id를 형제 태스크 묶음 링크로 표시.
  - to: capability.decompose-task
    type: realizes
    note: 분해 제안 패널에서 하위 태스크를 제안·수락해 트리를 만든다.
  - to: capability.manage-task-lifecycle
    type: realizes
    note: 상태 변경·담당자·사이클 배정·삭제 등 라이프사이클 조작을 수행.
impacts:
  - concept.task
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

운영자가 작업 보드나 목록에서 태스크 하나를 선택하면 화면 우측에서 미끄러져 들어오는 상세 패널이다. 한 태스크의 거의 모든 면 — 어디에 속해 있고(플랜·유닛 경로), 지금 어떤 상태이며 누가 맡고 있는지, 언제 만들어지고 시작·완료됐는지, 검증의 어떤 시나리오에 묶여 있고 그 근거가 코드 어디에 있는지, 어떤 하위 작업으로 쪼개졌는지, 실제로 몇 번 실행됐고 그 결과가 어땠는지 — 을 한 화면에 모아 보여준다. 단순 열람을 넘어 제목·담당자·상태·사이클 배정을 바로 고치고, 연결된 다른 태스크(상위/하위/의존 대상/같은 배치 형제)로 클릭 한 번에 이동할 수 있는 작업 허브 역할을 한다.

## UI 요소 / 입력 필드

- **상단 식별·동작 줄**: 태스크 ID 끝자리, 전체 편집(Edit) 진입, 삭제(Delete, 확인 대화 후), 닫기.
- **경로 이동(브레드크럼)**: 상위 플랜 → 유닛 → 현재 태스크 경로를 보여주고 각 단계로 이동. 별도로 태스크 자체의 조상·자식 이동 줄도 제공.
- **제목**: 클릭하면 인라인 편집되고 Enter 저장·Esc 취소. 티켓 번호 배지를 함께 표시.
- **우선순위·복잡도·예상 편집 규모** 배지.
- **상태**: 클릭하면 상태 변경 모달이 열린다. **담당자**: 인라인 편집(비우면 미배정). **사이클**: 드롭다운으로 배정/해제(사이클이 없으면 현재 배정만 읽기 표시).
- **타임스탬프**: 생성·시작·완료 시각.
- **PDD 메타 블록**: 시나리오 ID, 근거(evidence), 배치 ID를 항상 표시(없으면 em-dash). 근거가 `파일:줄` 형식이면 소스 링크로, 배치 ID는 같은 배치에 속한 형제 태스크 목록을 펼쳐 이동할 수 있는 링크로 렌더링.
- **의존성**: depends_on에 걸린 태스크들을 티켓 번호 버튼으로 표시하고 클릭 시 해당 태스크로 이동.
- **본문(Body)**: 마크다운 렌더링.
- **봉투(Envelope)**: 실행 계약 폼.
- **분해 트리 / 분해 제안 / 타임라인 리플레이 / 실행 비교(run diff)** 섹션.
- **하위 작업·아티팩트·질문·코멘트·실행 기록(Runs)** 섹션.

## 표시 데이터 / 호출 API

진입 시 해당 태스크 본체와 그에 딸린 아티팩트·실행 기록·질문·코멘트·자식 태스크를 한꺼번에 불러오고, 조상 추적을 위해 유닛·플랜을 추가로 조회한다. 프로젝트 단위로 사이클 목록도 가져와 사이클 배정 드롭다운을 채운다. 편집(제목·담당자·사이클·상태)은 태스크 갱신 호출로 즉시 반영하고, 삭제는 삭제 호출 후 드로어를 닫는다. 의존성·배치 형제는 각 대상 태스크를 조회해 티켓 번호로 해석한다. 실행 기록 표는 에이전트·시작/종료 시각·결과를, 질문 섹션은 질문 종류·작성자·본문과 답변을 보여준다. 분해 제안·타임라인 리플레이·실행 비교는 검색·실행 이력 API를 통해 데이터를 받아 렌더링한다(검색 경로는 데몬 검색 API에 의존).

## 상태 / 엣지케이스

- **로딩 중 / 태스크 미해결**: "Loading..." 자리표시.
- **빈 컬렉션**: 아티팩트·실행 기록·질문이 없으면 "No ..." 안내 문구, 본문이 비면 "No content".
- **조상 해석 실패**: 유닛/플랜 조회가 실패해도 치명적이지 않게 브레드크럼이 해석된 단계까지만 축소된다.
- **배치 조회**: 형제 조회 실패 시 "batch lookup failed", 형제가 없으면 "no siblings", 로딩 중이면 점 표시.
- **이동 핸들러 부재**: onSelectTask/onSelectItem이 연결되지 않으면 의존성·형제·트리의 버튼이 비활성으로 표시.
- **데몬 오프라인**: 상위 셸이 드로어 대신 데몬 오프라인 빈 상태를 띄우므로 이 드로어는 데몬 연결 상태에서만 의미를 가진다.

## 미확정 (OPEN)
- [ ] OPEN: 분해 제안·타임라인 리플레이·run 비교가 호출하는 구체 endpoint 노드 매핑(현재 검색 경로만 consumesApi로 연결).
