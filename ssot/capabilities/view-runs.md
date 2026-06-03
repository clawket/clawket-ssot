---
id: capability.view-runs
kind: Capability
title: 에이전트 실행 이력 보기
definition: 데스크탑 앱에서 어떤 작업이 실행된 이력(Run)을 읽기 전용으로 본다. 작업 상세 패널에서는 최신순 목록으로, Timeline 스윔레인에서는 에이전트별 세로 트랙의 시간 막대로 본다. 데스크탑은 실행 기록을 만들거나 고치지 않으며 어댑터가 작업 시작/종료 시 자동 생성한다.
purpose: 어떤 에이전트가 언제 어떤 작업을 얼마나 돌렸고 결과가 무엇이었는지를 사람이 사후에 확인할 수 있게 한다.
servesPersona:
  - persona.human-operator
realizedBy:
  - component.desktop-app
implementedIn:
  - desktop/apps/desktop/src/shell/TaskRunsPanel.tsx
  - desktop/apps/desktop/src/views/timeline/Swimlane.tsx
relatesTo:
  - to: concept.run
    type: reads
    note: 작업별 실행 기록을 읽기 전용으로 표시한다
  - to: concept.task
    type: relates-to
    note: 실행 기록은 특정 작업에 매여 있다
  - to: endpoint.cli-run
    type: relates-to
    note: 같은 실행 데이터를 CLI run 명령과 공유한다
impacts:
  - concept.run
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:capability
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로. 코드 옮겨적기 금지. -->

## 사용자가 할 수 있는 일

사람이 작업이 실제로 어떻게 돌아갔는지 사후에 들여다볼 수 있다. 두 곳에서 같은 실행 기록을 다른 모양으로 본다.

- **작업 상세 패널**: 한 작업의 실행을 최신순 목록으로 본다. 각 항목은 상태(대기/시작됨/끝남), 결과(성공/오류/취소/중단), 어느 에이전트가 돌렸는지, 시작·종료 시각, 걸린 시간을 보여주고 메모가 있으면 함께 표시한다.
- **Timeline 스윔레인**: 에이전트별로 세로 트랙을 나눠, 각 실행을 시간축 위 막대로 그린다. 색으로 성공·실행중·실패·종료를 구분하고, 막대에 마우스를 올리면 어느 작업이었는지·얼마나 걸렸는지 상세를 띄운다. 끝나지 않은 실행이라도 시작한 지 너무 오래(1시간 초과)면 멈춘 것으로 간주해 화면이 한 막대로 가려지지 않게 한다.

이 기능은 읽기 전용이다 — 데스크탑에서 실행 기록을 만들거나 고치지 않는다. 실행 기록은 작업 시작/종료 때 어댑터(훅)가 자동으로 만든다.

## 행위

- 한 작업의 실행 목록을 읽어 최신 시작순으로 정렬해 보여준다.
- 여러 작업의 실행을 에이전트별로 묶어 시간축 막대 그래프(스윔레인)로 보여준다.
- 막대를 클릭하면 그 실행이 속한 작업을 선택해 상세로 넘어간다.
- 실행 기록을 생성·수정·삭제하지 않는다(쓰기 동작 없음).

## 시스템 흐름

작업 상세(또는 Timeline) → 실행 목록을 데몬에서 읽음 → 상세 패널은 최신순으로 정렬해 목록 렌더, 스윔레인은 에이전트별 트랙에 시간 막대로 배치 → 막대 클릭 시 해당 작업 선택. 모든 경로가 조회만 수행한다.

## 어디에 구현되어 있나

작업 상세의 실행 패널(`TaskRunsPanel.tsx`)이 단일 작업의 실행 목록을 최신순으로 보여주고, Timeline 스윔레인(`views/timeline/Swimlane.tsx`)이 에이전트별 시간 막대 시각화를 담당한다. 스윔레인은 웹 대시보드의 동일 화면과 시각·구조를 맞춘 데스크탑판이다.

## 미확정 (OPEN)
- [ ] OPEN: 실행 목록 조회 호출이 데스크탑 데이터 계층의 어떤 경로를 타는지(소스 목록에 api.ts 미포함) 정확한 경로 미확인.
- [ ] OPEN: 멈춘 실행으로 간주하는 1시간 임계가 정책상 고정값인지 설정 가능한지 미확인.
