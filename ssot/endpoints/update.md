---
id: endpoint.update
kind: Endpoint
title: 최신 릴리스 설치 (update)
definition: "clawket update — GitHub Releases 에서 최신 Clawket 릴리스를 내려받아 로컬 CLI·데몬 바이너리를 교체한다. 데몬의 self-update 경로를 통해 수행된다."
realizedBy: [component.cli, component.daemon]
implementedIn:
  - cli/src/main.rs
relatesTo:
  - { to: endpoint.version-check, type: reads, note: "설치 전 최신 버전 가용 여부를 확인하는 짝 명령." }
  - { to: decision.binaries-from-releases, type: depends-on, note: "바이너리를 GitHub Releases 에서 배포한다는 결정에 따른 동작." }
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

`clawket update` 는 Clawket 자신을 최신 버전으로 갱신하는 명령이다. GitHub Releases 에 올라온 최신 릴리스를 내려받아, 로컬에 설치된 CLI 와 데몬 바이너리를 새 것으로 교체한다. 교체는 원자적 스왑(atomic swap) 방식이라 이미 실행 중인 데몬 프로세스는 다음 재시작 전까지 기존 버전 그대로 계속 동작한다 — 즉 갱신 즉시 동작이 바뀌지 않고, 데몬을 재시작해야 새 바이너리가 떠오른다.

누가 쓰나 — 사람 사용자가 새 기능/버그픽스를 받기 위해 직접 실행한다.

옵션:

- **미리보기(dry-run)**: 실제 설치 없이 "무엇이 내려받아질지"만 출력한다.
- **버전 고정(version)**: 최신 대신 특정 버전(예: v3.1.0)을 지정해 설치할 수 있다.

## 요청 / 응답

- 보내는 정보: "미리보기 여부"와 "고정할 버전(선택)".
- 돌려주는 정보: 설치(또는 미리보기) 결과 — 어떤 버전을 받았는지, 무엇이 교체되었는지에 해당하는 내용.

## 권한 / 제약

로컬 단일 사용자 환경 전제. 실행 중인 데몬은 즉시 교체되지 않으므로, 갱신을 완전히 반영하려면 데몬 재시작이 필요하다는 제약이 핵심이다.

## provenance

CLI 명령 정의·핸들러는 `cli/src/main.rs` 에 있다(`Update { dry_run, version }`). CLI 는 데몬의 `/self-update` 경로에 미리보기 여부와 버전을 실어 호출한다(POST). 실제 다운로드·바이너리 교체 로직은 데몬 측 self-update 처리에서 수행된다.

## 미확정 (OPEN)
- [ ] OPEN: CLI 는 데몬 `/self-update` 를 호출하지만 현재 데몬 라우터(`daemon/src/routes/mod.rs`)에 해당 경로 핸들러가 등록되어 있지 않다. self-update 다운로드/스왑 구현 위치(또는 미구현 여부)를 확인해 realizedBy/implementedIn 을 확정할 것.
- [ ] OPEN: 원자적 스왑의 구체 절차(임시 파일 → 교체, 권한, 롤백 처리)와 실패 시 동작을 확인할 것.
