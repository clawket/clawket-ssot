---
id: invariant.daemon-flock-single-instance
kind: Invariant
title: 셸은 항상 데몬 spawn을 시도하되, 단일 인스턴스는 데몬 자체 flock이 보장한다
definition: "데스크탑 셸은 시작할 때마다 데몬 자식 프로세스 띄우기를 시도한다. 이미 데몬이 떠 있으면 두 번째 인스턴스는 데몬 자신의 파일 잠금(flock)이 거부하므로 중복 실행이 생기지 않는다. 셸은 이 거부를 정상으로 받아들이고, 실제 생존 여부는 소켓 헬스 핑으로 확인한다."
governs:
  - component.tauri-shell
  - integration.clawketd-daemon
  - integration.unix-socket-health
decidedBy: []
implementedIn:
  - desktop/apps/desktop/src-tauri/src/lib.rs
  - desktop/apps/desktop/src-tauri/src/daemon.rs
crossesBoundary: true
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
relatesTo:
  - to: integration.clawketd-daemon
    type: relates-to
    note: 데몬 단일 인스턴스 보장의 책임 소재를 규정한다
  - to: invariant.socket-only-no-tcp-fallback
    type: relates-to
    note: 생존 판정을 소켓 핑에 위임하는 같은 셸의 동작
tags: [status:active, "type:invariant"]
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어. 코드 옮겨적기 금지(식별자는 provenance/근거에만). 코드 분기/의도 불명은 OPEN. -->

## 제약

데스크탑 앱은 시작할 때마다 "데몬을 자식 프로세스로 띄운다"를 무조건 시도한다. 그런데 이미 다른 경로(다른 앱 창, CLI, 직접 실행 등)로 데몬이 떠 있을 수도 있다. 이 경우 두 번째 데몬을 막는 책임은 셸이 아니라 데몬 자신에게 있다 — 데몬은 자체 파일 잠금(flock)으로 두 번째 인스턴스의 기동을 거부한다. 따라서 셸은 spawn이 거부되거나 실패해도 이를 오류로 취급하지 않고, "이미 떠 있는 데몬을 쓰면 된다"고 받아들인다. 정말로 데몬이 살아 있는지는 별도로 소켓 헬스 핑을 보내 확인한다.

## 깨지면 무슨 일이 일어나나

만약 단일 인스턴스 보장을 셸 쪽 조건 분기에 맡긴다면, 여러 진입 경로(여러 데스크탑 창, CLI 동시 실행)마다 제각각 "내가 띄울까 말까"를 판단해야 하고, 경합 상황에서 두 데몬이 같은 SQLite 파일을 동시에 잡는 사고가 날 수 있다. 단일 인스턴스 책임을 데몬의 flock 한 곳으로 모으면, 누가 spawn을 시도하든 결과적으로 데몬은 항상 하나만 살아 있게 되고 셸은 단순히 "시도 → 핑 확인"만 하면 된다. 책임을 한 군데로 모으는 설계(single source of enforcement)다.

## 코드에서 어떻게 강제되나

셸 진입점(`apps/desktop/src-tauri/src/lib.rs`)은 시작 시 먼저 데몬 spawn을 시도하고, 주석에서 "이미 clawketd가 떠 있으면 두 번째 인스턴스는 데몬 자신의 flock이 거부하므로 그것을 용인하고 ping_health로 생존을 확인한다"고 명시한다. spawn이 실패해도 에러를 stderr에 남길 뿐 진행을 막지 않으며, 잠시 뒤 소켓으로 헬스 핑을 보내 실제 생존을 확인한다. spawn 로직(`apps/desktop/src-tauri/src/daemon.rs`)은 데몬 바이너리를 해석해 자식 프로세스로 띄우기만 하고 중복 여부는 판단하지 않는다 — 중복 거부는 데몬 측 flock의 몫이다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요 — 단일 인스턴스 책임을 데몬 flock에 두기로 한 결정 기록
