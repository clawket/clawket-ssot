---
id: screen.daemon-offline-state
kind: Screen
title: 데몬 오프라인 빈 상태
purpose: 로컬 데몬에 연결할 수 없을 때 빈 캔버스 대신 원인 안내·실행 명령·재연결 버튼을 보여줘 운영자가 스스로 복구하게 한다.
servesPersona:
  - persona.human-operator
realizedBy:
  - component.web-dashboard
implementedIn:
  - web/src/App.tsx
consumesApi:
  - endpoint.daemon-health
relatesTo:
  - to: component.daemon
    type: reads
    note: 이 화면이 감지·복구 대상으로 삼는 로컬 clawketd 데몬.
  - to: endpoint.cli-daemon
    type: reads
    note: 안내 문구가 'clawket daemon start' 실행을 직접 제시.
impacts:
  - component.web-dashboard
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

웹 대시보드는 모든 데이터를 로컬 데몬에서 받아오므로, 데몬이 꺼져 있으면 화면 전체가 동작하지 않는다. 이 상태에 빈 캔버스를 그대로 두면 사용자는 무엇이 잘못됐는지 알 수 없다. 그래서 데몬 연결이 끊긴 동안 메인 영역을 이 안내 화면으로 대체한다 — 무슨 일이 일어났는지(데몬에 닿을 수 없음), 어떻게 고치는지(`clawket daemon start` 실행), 그리고 클릭 한 번으로 재연결을 시도하는 길을 제시해 운영자가 자력으로 복구하도록 돕는다.

## UI 요소 / 입력 필드

- 경고 아이콘과 "Daemon offline" 제목.
- 설명 문구: 로컬 데몬에 닿을 수 없으며 `clawket daemon start`로 시작하거나 아래 재연결 버튼을 누르라는 안내. 명령은 코드 스타일로 강조.
- **Reconnect 버튼**: 데몬 헬스 재확인과 SSE 재연결을 함께 트리거.
- **도움말(?) 버튼**: 도움말 모달 열기.
- 별도로 상단바의 데몬 다운 표식(pill)이 이 상태 동안 계속 떠 있어, 재연결 신호가 항상 보이게 유지.

## 표시 데이터 / 호출 API

데몬 연결 여부는 헬스 폴링으로 판정한다(데몬 GET /health). 연결이 끊긴 것으로 판정되면 메인 영역이 프로젝트 뷰 대신 이 빈 상태로 전환된다. Reconnect를 누르면 데몬 재연결(헬스 재확인)과 이벤트 스트림(SSE) 재연결을 함께 수행하고, 데몬이 다시 살아나면 자동으로 정상 뷰로 복귀한다. 끊김/복구 시 토스트로도 알린다.

## 상태 / 엣지케이스

- **연결 끊김(disconnected)**: 이 화면을 메인 영역에 표시.
- **재연결 시도 중**: 상단바 표식이 연결 상태를 반영(connecting/reconnecting).
- **복구**: 데몬이 다시 연결되면 이전 선택 프로젝트/뷰로 자동 복귀하고 SSE를 다시 연다.
- **데몬 정상이나 SSE 지연(lagging)**: 별개 신호로, 이 전면 빈 상태가 아니라 상단바의 상태 표식으로만 드러난다(데몬 자체는 연결됨).

## 미확정 (OPEN)
- [ ] OPEN: 데몬 헬스 폴링 주기·실패 임계치 등 연결 판정의 구체 파라미터(useDaemonHealth 훅 정독 필요).
