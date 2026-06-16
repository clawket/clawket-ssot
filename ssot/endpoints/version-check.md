---
id: endpoint.version-check
kind: Endpoint
title: 새 버전 확인 (version-check)
definition: "clawket version-check — 설치하지 않고 최신 버전 가용 여부만 확인한다. 로컬 버전과 GitHub 최신 릴리스를 비교한다."
realizedBy: [component.cli, component.daemon]
implementedIn:
  - cli/src/main.rs
relatesTo:
  - { to: endpoint.update, type: triggers, note: "확인 결과 새 버전이 있으면 update 로 설치한다(짝 명령)." }
  - { to: decision.binaries-from-releases, type: depends-on, note: "비교 대상이 GitHub Releases 의 최신 릴리스다." }
governedBy: []
impacts: []
consumedBy: [capability.self-update]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

`clawket version-check` 는 "지금 새 버전이 나와 있는지"만 알려주는 명령이다. 실제 설치는 하지 않고, 로컬에 설치된 Clawket 버전과 GitHub Releases 의 최신 릴리스를 비교해 갱신이 필요한지 여부를 보여준다. 설치는 별도의 `update` 명령으로 한다.

누가 쓰나 — 사람 사용자가 "업데이트할 게 있나" 가볍게 확인할 때, 또는 갱신 안내를 띄우는 흐름에서 쓴다.

## 요청 / 응답

- 보내는 정보: 없음(현재 설치 버전은 로컬에서 자체 파악).
- 돌려주는 정보: 최신 버전 가용 여부와 비교 결과 — 현재 버전 대비 더 새로운 릴리스가 있는지에 해당하는 내용.

## 권한 / 제약

로컬 단일 사용자 환경 전제. 설치를 수행하지 않는 읽기성 조회이므로 시스템 상태를 바꾸지 않는다(외부 GitHub 조회만 발생).

## provenance

CLI 명령 정의·핸들러는 `cli/src/main.rs` 에 있다(`VersionCheck`). CLI 는 데몬의 `/version-check` 경로를 읽기(GET) 호출한다. 버전 비교 로직은 데몬 측에서 수행된다.

## 미확정 (OPEN)
- [ ] OPEN: CLI 는 데몬 `/version-check` 를 호출하지만 현재 데몬 라우터(`daemon/src/routes/mod.rs`)에 해당 경로 핸들러가 등록되어 있지 않다. 버전 비교 구현 위치(또는 미구현 여부)를 확인해 realizedBy/implementedIn 을 확정할 것.
- [ ] OPEN: 버전 비교 기준(semver 비교, 사전 릴리스 처리 등)과 결과 표현 형식을 확인할 것.
