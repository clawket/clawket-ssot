---
id: integration.clawketd-daemon
kind: Integration
title: 데스크탑 ↔ clawketd 데몬 연동
purpose: 데스크탑 앱은 자체적으로 데이터를 저장하지 않고, 모든 상태(작업·플랜·지식 등)를 clawketd 데몬에 위임한다. 데몬이 떠 있어야 화면이 무엇이든 보여줄 수 있으므로, 앱은 켜질 때 데몬을 책임지고 띄운다.
definition: 데스크탑 셸(Tauri)이 백엔드 데몬 clawketd를 자식 프로세스로 직접 띄우고, 그 데몬이 제공하는 로컬 저장과 HTTP API·실시간 이벤트에 의존하는 내부 연동이다.
integratesWith:
  - component.desktop-app
  - component.daemon
  - component.daemon-supervisor
  - component.daemon-client
implementedIn:
  - desktop/apps/desktop/src-tauri/src/daemon.rs
  - desktop/apps/desktop/src-tauri/src/lib.rs
  - desktop/apps/desktop/src/data/api.ts
impacts:
  - concept.run
  - screen.daemon-offline-state
governedBy:
  - invariant.daemon-bin-resolution
  - invariant.daemon-flock-single-instance
  - invariant.daemon-liveness-socket-probe
relatesTo:
  - to: integration.unix-socket-health
    type: relates-to
    note: 데몬을 띄운 뒤 살아 있는지 확인하는 절차가 이 연동의 후속 단계다.
  - to: integration.session-token
    type: relates-to
    note: 띄운 데몬과 실제로 통신하려면 데몬이 발급한 토큰이 필요하다.
  - to: decision.daemon-single-source
    type: relates-to
    note: "상태를 데몬 한 곳에만 두는 결정의 데스크탑 측 구현."
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:integration
---

## 무엇과 연동하나

데스크탑 앱은 화면(렌더러)만 가지고 있을 뿐, 작업·플랜·지식 같은 데이터를 스스로 저장하지 않는다. 이 모든 것은 별도의 백엔드 프로그램인 clawketd(데몬)가 관리한다. 따라서 데스크탑 앱은 켜지는 순간 데몬을 함께 살려 두어야 한다.

연동 방식은 두 단계다. 첫째, 앱은 시작할 때 데몬 실행 파일을 찾아 자식 프로세스로 직접 실행한다. 데몬 실행 파일은 정해진 우선순위로 탐색한다 — 먼저 환경 변수로 직접 지정된 경로, 다음으로 앱 바이너리 옆이나 플러그인 배치 위치, 마지막으로 시스템 PATH 순이다. 어디에도 없으면 이름만으로 시스템 검색에 맡기는 마지막 폴백을 둔다. 둘째, 데몬이 자리를 잡을 시간을 잠깐 준 뒤 생존 여부를 확인한다(이 확인 절차 자체는 별도 연동 노드로 분리되어 있다).

특별한 동작이 하나 있다. 데몬이 이미 다른 경로(예: CLI나 이전 세션)에서 떠 있는 경우, 앱이 두 번째 데몬을 띄우려 해도 데몬 스스로가 파일 잠금(flock)으로 두 번째 인스턴스를 거부한다. 앱은 이 거부를 정상으로 받아들이고, 이미 떠 있는 데몬을 그대로 사용한다. 즉 "내가 띄운 데몬"이든 "이미 떠 있던 데몬"이든 상관없이, 살아 있는 데몬 한 개에만 연결한다.

자동 실행을 끄는 환경 변수 스위치도 있다. 이 스위치가 켜져 있으면 앱은 데몬을 띄우지 않고, 이미 떠 있는 데몬을 찾아 쓰는 쪽으로만 동작한다(개발 시나리오 대응).

## 구현 위치 (provenance)

- `desktop/apps/desktop/src-tauri/src/daemon.rs` — 데몬 실행 파일 탐색 순서와 자식 프로세스 실행(표준 입출력을 모두 끊은 분리 실행)을 담당한다. 자동 실행 비활성화 스위치 처리도 여기 있다.
- `desktop/apps/desktop/src-tauri/src/lib.rs` — 앱 시작 시퀀스. 데몬을 띄우려 시도하고, 성공하면 핸들을 보관하며, 실패하면 "이미 떠 있는 데몬을 쓰겠다"는 메시지를 남긴 뒤, 잠시 후 생존 확인을 비동기로 수행한다.
- `desktop/apps/desktop/src/data/api.ts` — 렌더러가 데몬의 HTTP API에 붙는 클라이언트. 기본 접속 주소(루프백 19400)와 모든 요청 처리가 여기 있다.

이 연동을 사용하는 내부 컴포넌트는 데스크탑 앱(component.desktop-app), 그 안에서 데몬 생애를 관리하는 감독 모듈(component.daemon-supervisor)과 HTTP 클라이언트(component.daemon-client), 그리고 연동 대상인 데몬 본체(component.daemon)다.

## 불변식

- 데몬 실행 파일 탐색 순서는 CLI 쪽과 동일하게 맞춰져 있어야 한다(invariant.daemon-bin-resolution). 데스크탑은 CLI 코드에 직접 의존하지 않고 "탐색 순서와 바이너리 이름"이라는 공유 규칙만 따른다.
- 데몬은 한 번에 하나만 떠야 한다(invariant.daemon-flock-single-instance). 앱이 중복 실행을 시도해도 데몬의 파일 잠금이 두 번째를 막는다.
- 띄운 뒤에는 반드시 생존을 확인한다(invariant.daemon-liveness-socket-probe). 살아 있음의 판단은 "내가 프로세스를 만들었는지"가 아니라 "응답하는 데몬이 있는지"로 한다.

## 영향 범위

- 작업 실행 기록(concept.run)은 데몬이 보관하므로, 데몬 연결이 끊기면 실행 이력 표시가 영향을 받는다.
- 데몬이 끝내 응답하지 않으면 앱은 오프라인 상태 화면(screen.daemon-offline-state)으로 떨어진다 — 이 연동의 실패는 곧 그 화면으로 이어진다.

## 미확정 (OPEN)
- [ ] OPEN: 앱 종료 시 자식 데몬을 함께 종료하는지(또는 의도적으로 살려 두는지)의 정책 — lib.rs는 핸들을 보관만 하고 종료 훅은 확인되지 않음.
