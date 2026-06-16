---
id: capability.daemon-reconnect
kind: Capability
title: 데몬 연결 상태 확인·재연결
purpose: 데스크탑 대시보드가 의존하는 로컬 데몬이 살아있는지 한눈에 보여주고, 연결이 끊겼을 때 사용자가 클릭 한 번으로 즉시 재확인하고 최신 데이터를 다시 불러올 수 있게 한다.
servesPersona:
  - persona.human-operator
realizedBy:
  - component.desktop-app
consumesApi:
  - endpoint.daemon-health
implementedIn:
  - desktop/apps/desktop/src/shell/Topbar.tsx
  - desktop/apps/desktop/src/hooks/useDaemonHealth.ts
  - desktop/apps/desktop/src/App.tsx
relatesTo:
  - to: component.daemon
    type: depends-on
    note: 데몬 생존 여부가 이 표시의 진실 원천
  - to: platform.desktop
    type: belongs-to
    note: 데스크탑 셸 상단 바의 연결 상태 표시기
impacts:
  - persona.human-operator
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:capability
---

## 사용자가 할 수 있는 일

데스크탑 대시보드는 모든 데이터를 로컬 데몬에서 가져오기 때문에, 데몬이 떠 있는지가 곧 앱이 정상 동작하는지를 좌우한다. 사용자는 상단 바의 작은 상태 알약(pill)으로 그 연결 상태를 늘 확인할 수 있다. 데몬이 정상이면 초록색 점과 "daemon ok", 끊기면 빨간색 점이 깜빡이며 "daemon down"으로 표시된다.

연결이 끊긴 상태에서는 그 알약 자체가 버튼이 되어, 클릭하면 즉시 데몬 생존을 다시 확인하고 동시에 화면 데이터를 새로 불러온다. 사용자가 데몬을 수동으로 다시 띄운 뒤 자동 주기를 기다릴 필요 없이 바로 복구를 시도할 수 있게 하는 것이 의도다.

## 행위

- **주기적 헬스 확인**: 약 10초 간격으로 데몬의 헬스 응답을 확인해 연결 여부를 갱신한다(웹 대시보드와 동일한 주기·판정 방식).
- **상태 표시**: 정상이면 알약이 초록(클릭 불가), 끊기면 빨강(깜빡임 + 클릭 가능)으로 바뀐다. 정상일 때는 "Daemon connected", 끊겼을 때는 "Daemon down — click to reconnect" 안내가 뜬다.
- **상태 전환 알림**: 연결이 끊기는 순간 "Daemon disconnected. Trying to reconnect…" 토스트가, 다시 붙는 순간 "Daemon reconnected." 토스트가 표시된다.
- **수동 재연결**: 끊긴 상태에서 알약을 클릭하면 즉시 헬스를 재확인하고(주기를 기다리지 않음) 동시에 데이터 전체를 새로고침한다.

## 시스템 흐름

화면에 마운트되면 즉시 한 번, 이후 약 10초마다 데몬의 헬스 엔드포인트를 호출해 살아있는지 확인한다. 직전 상태와 비교해 연결 여부가 바뀌면(붙음↔끊김) 그때만 상태 변경 콜백이 불려 토스트로 사용자에게 알린다.

상단 바의 알약은 이 연결 여부를 그대로 반영한다. 끊긴 상태에서 사용자가 알약을 클릭하면 재연결 핸들러가 두 가지를 동시에 한다 — 헬스를 즉시 재확인하고, 데몬에서 전체 데이터를 다시 가져오는 새로고침을 건다. 데몬이 그사이 복구되어 있었다면 다음 폴링을 기다릴 것 없이 화면이 바로 최신 상태로 회복된다.

## 어디에 구현되어 있나

연결 상태 폴링·전환 감지·수동 재연결 트리거는 데몬 헬스 훅(`desktop/apps/desktop/src/hooks/useDaemonHealth.ts`)에 있고, 상태 알약 UI와 끊김 시 클릭 동작은 상단 바 컴포넌트(`desktop/apps/desktop/src/shell/Topbar.tsx`)에 있다. 둘을 묶어 재연결 시 데이터 새로고침까지 함께 거는 배선은 앱 루트(`desktop/apps/desktop/src/App.tsx`)에서 이뤄진다. 이 동작은 웹 대시보드의 동일 기능을 데스크탑으로 이식한 것으로, 주기와 판정 방식이 웹과 동일하다.

## 미확정 (OPEN)
- [ ] OPEN: Tauri 데스크탑 앱이 아직 미출시(`component.desktop-app` 핀 null) 상태라, 데스크탑 렌더러 코드 기준으로만 확인되었고 배포 앱에서의 실제 동작 검증은 보류.
