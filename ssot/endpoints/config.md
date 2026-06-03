---
id: endpoint.config
kind: Endpoint
title: 설정 읽기/쓰기 (config)
definition: "clawket config — 사용자 홈 아래 설정 저장소에 보관되는 Clawket 환경설정 값을 읽고 쓰고 지운다. get / set / unset / list 네 가지 동작을 가진다."
realizedBy: [component.cli, component.daemon]
implementedIn:
  - cli/src/main.rs
relatesTo:
  - { to: concept.project, type: reads, note: "기본 프로젝트(default_project) 같은 작업 컨텍스트 키를 설정값으로 보관한다." }
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

`clawket config` 는 Clawket 의 사용자 환경설정 값을 다루는 명령이다. 설정값은 사용자 홈 디렉터리 아래의 설정 저장소(`~/.config/clawket/`)에 보관되며, 한 번 정해두면 이후 명령 실행 시 기본값으로 재사용된다. 대표적으로 "기본 프로젝트"(default_project)나 데몬 접속 포트(daemon.port) 같은, 매 명령마다 다시 지정하기 번거로운 값을 한 곳에 박아두는 용도다.

누가 쓰나 — 주로 사람 사용자가 온보딩 직후 또는 환경을 바꿀 때 직접 쓴다. 네 가지 동작이 있다.

- **get**: 키 하나의 현재 값을 읽는다.
- **set**: 키에 값을 쓴다(없으면 새로 만들고, 있으면 덮어쓴다).
- **unset**: 키를 제거한다.
- **list**: 저장된 모든 키와 값을 한 번에 보여준다.

## 요청 / 응답

- **읽기(get)**: 설정 키 이름 하나를 받아, 그 키의 현재 값을 돌려준다.
- **쓰기(set)**: 키 이름과 새 값을 받아 저장하고, 저장 결과를 돌려준다.
- **삭제(unset)**: 키 이름을 받아 그 항목을 지운다.
- **목록(list)**: 입력 없이, 현재 저장된 모든 설정 항목(키와 값의 쌍)을 돌려준다.

키는 `default_project` 처럼 단순한 이름일 수도 있고 `daemon.port` 처럼 점으로 구분된 계층형 이름일 수도 있다.

## 권한 / 제약

로컬 단일 사용자 환경이 전제다(별도 인증/권한 분리 없음). 설정은 사용자 본인의 로컬 파일에만 영향을 주므로 다른 프로젝트나 다른 사용자에게 파급되지 않는다.

## provenance

CLI 명령 정의와 핸들러는 `cli/src/main.rs` 에 있다(`Config` 명령 + `ConfigAction` 하위 명령 get/set/unset/list). CLI 는 각 동작을 데몬의 설정 경로(`/config`, `/config/{key}`)에 대한 읽기/쓰기/삭제 호출로 전달하도록 작성되어 있다.

## 미확정 (OPEN)
- [ ] OPEN: CLI 는 데몬의 `/config` 및 `/config/{key}` 경로를 호출하지만, 현재 데몬 라우터(`daemon/src/routes/mod.rs` 의 `router()`)에 해당 경로 핸들러가 등록되어 있지 않다. 데몬 측 설정 저장/조회 구현 위치(또는 미구현 여부)를 확인해 realizedBy/implementedIn 을 확정할 것.
- [ ] OPEN: 유효한 설정 키 전체 목록과 각 키의 의미·기본값(예: default_project, daemon.port 외)을 코드/문서에서 확정할 것.
