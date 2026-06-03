---
id: invariant.daemon-bin-resolution
kind: Invariant
title: 데몬 바이너리 경로 해석 순서는 환경변수 → 플러그인 레이아웃 후보 → PATH
definition: "데스크탑 셸이 띄울 clawketd 실행 파일을 찾을 때, 환경변수 지정(CLAWKET_DAEMON_BIN)을 가장 먼저 보고, 없으면 플러그인 설치 레이아웃 후보(셸과 같은 디렉터리의 형제, ../daemon/bin/, XDG 설치 경로)를 순서대로 살핀 뒤, 마지막으로 시스템 PATH에 맡긴다. 이 순서는 CLI의 해석 순서와 일치시킨 약속이다."
governs:
  - component.tauri-shell
  - integration.clawketd-daemon
decidedBy: []
implementedIn:
  - desktop/apps/desktop/src-tauri/src/daemon.rs
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
relatesTo:
  - to: integration.clawketd-daemon
    type: relates-to
    note: 어느 데몬 실행 파일을 띄울지 결정하는 경로 규칙
  - to: invariant.daemon-flock-single-instance
    type: relates-to
    note: 해석된 바이너리를 띄우는 spawn 동작의 전제
tags: [status:active, "type:invariant"]
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어. 코드 옮겨적기 금지(식별자는 provenance/근거에만). 코드 분기/의도 불명은 OPEN. -->

## 제약

데스크탑 앱이 띄울 데몬 실행 파일(clawketd)을 찾는 순서는 고정되어 있다. 첫째로 환경변수로 직접 지정한 경로가 있으면 그것을 최우선으로 쓴다(개발·테스트에서 특정 빌드를 강제하기 위함). 그것이 없으면, 플러그인이 설치되는 표준 레이아웃의 후보들을 차례로 확인한다 — 앱 바이너리와 같은 폴더의 형제 파일, 그 위 단계의 `daemon/bin/` 아래, 그리고 XDG 사용자 데이터 설치 경로 순이다. 후보 중 실제로 존재하는 첫 번째를 채택한다. 끝까지 못 찾으면 마지막 수단으로 이름만 넘겨 시스템 PATH 탐색에 맡긴다.

중요한 점은 이 순서가 임의로 정해진 것이 아니라 CLI의 데몬 바이너리 해석 순서와 의도적으로 동일하게 맞춰졌다는 것이다 — 데스크탑이든 CLI든 같은 규칙으로 같은 데몬을 찾아야 한다.

## 깨지면 무슨 일이 일어나나

해석 순서가 CLI와 어긋나면, 같은 머신에서 CLI가 띄우는 데몬과 데스크탑이 띄우는 데몬이 서로 다른 바이너리(다른 버전)가 될 수 있다. 그러면 한쪽은 신버전, 한쪽은 구버전 데몬을 가리켜 스키마·동작이 미묘하게 달라지고, 사용자는 "CLI에서는 되는데 앱에서는 안 된다"는 재현 불가 버그를 만난다. 또 환경변수 지정을 무시하면 개발 중 특정 빌드를 강제할 방법이 사라진다. 두 진입점이 같은 우선순위로 같은 파일을 찾도록 맞추는 것이 일관성의 핵심이다.

## 코드에서 어떻게 강제되나

데몬 해석 로직(`apps/desktop/src-tauri/src/daemon.rs`)은 모듈 주석에서 "CLI의 resolve_daemon_bin과 동일한 순서를 미러링한다"고 명시하고, 실제로 환경변수 → 플러그인 레이아웃 후보(형제 / ../daemon/bin / XDG) → PATH 폴백 순으로 첫 존재 후보를 채택한다. CLI 크레이트에 직접 의존하지는 않는데(데스크탑 앱을 소스 수준으로 결합하지 않으려는 의도), 그래서 "동일하게 유지해야 할 약속"이 코드 결합이 아니라 이 불변식으로만 묶여 있다 — 둘을 함께 바꿔야 하는 합의 지점이다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요 — CLI와 해석 순서를 일치시키기로 한 결정 기록
