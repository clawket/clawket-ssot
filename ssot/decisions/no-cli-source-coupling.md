---
id: decision.no-cli-source-coupling
kind: Decision
title: 데스크탑은 CLI에 소스로 의존하지 않고 불변식만 공유한다
purpose: "데스크탑 앱과 CLI가 둘 다 데몬을 찾아 띄우고 같은 소켓으로 통신해야 할 때, 둘은 공통 코드를 공유해 묶일 것인가 아니면 규칙만 맞출 것인가?"
definition: "데스크탑 앱은 CLI 크레이트에 소스 수준으로 의존하지 않고, 데몬 바이너리 탐색 순서와 소켓 전용·TCP 폴백 없음 같은 불변식만 CLI와 공유한다(같은 규칙을 각자 구현)."
relatesTo:
  - to: component.desktop-app
    type: governs
    note: "CLI에 소스로 묶이지 않는다는 규약이 적용되는 대상"
  - to: component.cli
    type: relates-to
    note: "불변식(탐색 순서·소켓 규약)의 기준이 되는 동등 구현"
  - to: invariant.daemon-bin-resolution
    type: relates-to
    note: "양쪽이 동일하게 따르는 데몬 바이너리 탐색 순서"
  - to: invariant.socket-only-no-tcp-fallback
    type: relates-to
    note: "양쪽이 동일하게 따르는 소켓 전용·TCP 폴백 없음 규약"
  - to: component.tauri-shell
    type: governs
    note: "이 규약을 실제로 구현하는 데스크탑 네이티브 셸 계층"
supersedes: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:decision"]
---

> Decision은 append-only다. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 `supersedes`로 잇는다.

## 맥락 (Context)

데스크탑 앱과 CLI는 둘 다 같은 데몬(clawketd)을 상대한다. 둘 다 데몬 바이너리를 어디서 찾을지 같은 순서로 탐색해야 하고, 둘 다 같은 방식(유닉스 소켓 전용, TCP로 우회하지 않음)으로 데몬에 붙어야 한다. 동작이 같아야 하므로, 데스크탑이 CLI의 코드를 그대로 끌어다 쓰면(소스 수준 의존) 중복 없이 동작을 맞출 수 있다는 유혹이 있다.

문제는 그렇게 묶이는 순간 데스크탑이 CLI 크레이트의 변경에 끌려다니게 된다는 점이다. CLI의 내부 구조가 바뀌면 데스크탑 빌드가 깨지고, 데스크탑은 자기에게 필요 없는 CLI의 다른 의존성까지 떠안게 된다. 즉 두 클라이언트가 한 몸으로 묶여 독립적으로 발전하지 못한다.

## 결정 (Decision)

데스크탑 앱은 CLI 크레이트에 소스 수준으로 의존하지 않는다. 대신 두 곳이 반드시 똑같아야 하는 것 — 데몬 바이너리 탐색 순서(환경변수 우선, 그다음 설치 위치 후보들, 마지막으로 PATH)와, 데몬과는 유닉스 소켓으로만 통신하고 TCP로 폴백하지 않으며 토큰을 싣지 않는다는 규약 — 만 불변식으로 공유한다. 그 불변식을 데스크탑은 자기 코드 안에 별도로 구현한다.

공유되는 것은 "어떤 순서로 찾고 어떻게 붙느냐"는 규칙과 바이너리 이름뿐이고, 그것을 실행하는 코드는 각자 가진다.

## 근거와 결과 (Consequences)

이 선택은 두 클라이언트의 독립성을 지킨다(낮은 결합도). CLI의 내부가 바뀌어도 데스크탑 빌드가 영향받지 않고, 데스크탑은 필요 없는 CLI 의존성을 떠안지 않는다.

대신 같은 규칙이 두 곳에 각각 구현되어 있으므로, 탐색 순서나 소켓 규약이 바뀌면 양쪽을 함께 고쳐야 한다는 책임이 따른다. 코드는 이 동일성을 주석으로 명시해(서로의 구현 위치를 가리키며) 둘이 어긋나지 않도록 신호를 남긴다.

막아주는 실패는 "CLI 리팩터링이 데스크탑을 무단으로 깨뜨리는" 전이 실패다. 감수하는 비용은 "불변식이 바뀔 때 두 곳 동기화를 사람이 챙겨야 한다"는 점이며, 이는 불변식 노드로 명시해 둠으로써 관리한다.

## 근거 (Provenance)

- `desktop/apps/desktop/src-tauri/src/daemon.rs` — CLI의 `resolve_daemon_bin`과 같은 탐색 순서를 의도적으로 미러링하되 "CLI 크레이트에 의존하지 않아 `apps/desktop`을 소스 수준 결합에서 독립시킨다", 공유되는 것은 탐색 순서와 바이너리 이름뿐이라는 주석.
- `desktop/apps/desktop/src-tauri/src/socket.rs` — "socket-only, no TCP fallback, no token loading" CLI 불변식을 미러링한다는 주석 및 별도 구현.
