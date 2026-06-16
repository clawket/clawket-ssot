---
id: endpoint.restore
kind: Endpoint
title: clawket restore
definition: "백업 아카이브로부터 데이터를 복원하는 CLI 명령. 기본은 현재 DB 교체, --merge 는 위에 덮어 합치기, --dry-run 은 적용 없이 미리보기."
realizedBy:
  - component.cli
  - component.daemon-client
implementedIn:
  - cli/src/main.rs
relatesTo:
  - to: component.daemon
    type: calls
    note: "데몬에 복원을 요청한다(POST /restore)."
  - to: component.sqlite-storage
    type: mutates
    note: "아카이브 내용으로 DB 를 교체하거나 병합한다."
  - to: concept.knowledge
    type: mutates
    note: "아카이브에 포함된 지식 항목도 함께 복원된다."
  - to: endpoint.backup
    type: relates-to
    note: "backup 이 만든 아카이브를 입력으로 받는 짝 명령."
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

`clawket restore` 는 backup 명령으로 만든 아카이브를 받아 그 안의 데이터를 다시 Clawket 에 들여놓는 명령이다. 다른 기기로 이전하거나, 손상·초기화된 워크스페이스를 이전 시점으로 되살리는 데 쓴다. 데이터를 어떻게 들여놓을지에 대해 세 가지 모드를 제공한다.

## 요청 / 응답

요청에는 복원할 아카이브 파일 경로를 지정한다. 동작 모드는 다음과 같다.

- 기본(교체): 현재 데이터베이스를 아카이브 내용으로 대체한다. 기존 상태가 사라질 수 있는 파괴적 동작이다.
- 합치기(merge): 현재 데이터를 지우지 않고 아카이브 내용을 그 위에 얹어 합친다.
- 미리보기(dry-run): 실제로 아무것도 바꾸지 않고, 적용했을 때 무엇이 어떻게 될지만 보여 준다. 교체/합치기를 실행하기 전에 영향을 확인하는 안전장치다.

응답은 복원 결과 정보다(미리보기 모드에서는 적용 예정 내역).

## 권한 / 제약

데몬이 실행 중이어야 한다. 기본 교체 모드는 기존 데이터를 덮어쓰는 파괴적 동작이므로, 실행 전 dry-run 으로 영향을 확인하거나 사전에 백업을 받아 두는 것이 안전하다. 합치기 모드는 기존 데이터를 보존한 채 병합한다.

## provenance

CLI 정의·디스패치는 `cli/src/main.rs`(Restore 서브커맨드: input 경로·merge·dry_run 옵션을 받아 데몬의 복원을 요청). 실제 복원·병합은 데몬 측에서 수행된다. "백업 아카이브를 교체/합치기/미리보기 모드로 다시 들여놓는, backup 의 짝이 되는 복원 명령"이라는 의미만으로 재구현 가능하다.

## 미확정 (OPEN)
- [ ] OPEN: 데몬의 복원 경로에서 교체와 합치기 모드의 정확한 충돌 처리 규칙(중복 ID·버전 충돌 시 어느 쪽이 이기는지)을 데몬 측에서 확인할 것.
