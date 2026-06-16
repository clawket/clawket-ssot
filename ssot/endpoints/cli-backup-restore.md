---
id: endpoint.cli-backup-restore
kind: Endpoint
title: clawket backup / restore
definition: "Clawket의 모든 작업 데이터(데이터베이스 + 첨부된 지식)를 옮길 수 있는 압축 묶음으로 내보내고, 그 묶음을 다시 들여와 현재 데이터를 교체하거나 덮어 합치는 CLI 명령 한 쌍이다."
realizedBy: [component.cli, component.daemon]
implementedIn:
  - cli/src/main.rs
relatesTo:
  - { to: concept.knowledge, type: reads, note: "첨부된 지식 항목까지 함께 묶음에 담는다" }
  - { to: component.sqlite-storage, type: backed-by, note: "내보내고 복원하는 대상이 로컬 SQLite 저장소 전체다" }
governedBy: []
impacts: []
consumedBy: []
servesPersona: [persona.human-operator]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
`clawket backup`과 `clawket restore`는 Clawket이 들고 있는 작업 데이터 전체를 한 덩어리로 묶어 다른 머신으로 옮기거나, 외부에 안전하게 보관해 두기 위한 명령이다. 묶음에는 프로젝트·플랜·태스크 같은 데이터베이스 내용과 함께 첨부된 지식 항목까지 들어간다. backup은 그 전부를 휴대 가능한 압축 파일로 내보내고, restore는 그 파일을 다시 들여온다. 머신을 옮기거나 백업을 만들어 두고 싶은 사람 운영자가 사용한다.

## 요청 / 응답
- backup은 만들 묶음 파일의 위치를 지정할 수 있고(생략하면 시각이 박힌 기본 이름), 특정 프로젝트만 담을지 전체를 담을지 고를 수 있다. 응답은 만들어진 묶음에 대한 결과 요약이다.
- restore는 들여올 묶음 파일의 위치를 받는다. 들여오는 방식으로 두 가지가 있다 — 기본은 현재 데이터베이스를 묶음 내용으로 교체하는 것이고, 합치기 방식을 택하면 기존 데이터 위에 묶음을 겹쳐 올린다. 실제로 적용하지 않고 무엇이 바뀔지 미리 보기만 하는 모드도 있다.

## 권한 / 제약
- 묶음은 데이터베이스와 첨부 지식을 함께 담은 휴대용 압축 파일이며, 머신 간 이동과 외부 보관을 전제로 한다.
- restore의 기본 동작은 "교체"다 — 현재 데이터를 묶음 내용으로 덮어쓴다. 기존 것을 살리면서 합치려면 합치기 방식을 명시해야 한다. 적용 전에 미리 보기로 영향 범위를 확인할 수 있다.
- 내보내기·들여오기는 모두 로컬에서 처리되며 데이터가 외부로 전송되지 않는다.

## provenance
CLI(`component.cli`)가 명령 표면을 제공하고, 실제 묶기·복원은 로컬 데몬(`component.daemon`)의 백업/복원 라우트가 수행한다. 대상이 되는 데이터는 로컬 SQLite 저장소(`component.sqlite-storage`)와 첨부 지식이다. 명령 정의(출력 위치·프로젝트 한정·교체/합치기/미리보기 옵션)는 `cli/src/main.rs`의 Backup·Restore 분기에 있다.

## 미확정 (OPEN)
- [ ] OPEN: 합치기 방식에서 같은 식별자를 가진 항목이 충돌할 때의 우선순위(묶음 우선 vs 현재 우선)와 미리 보기가 보여주는 정보의 범위는 데몬 측 backup/restore 라우트를 정독해 확정 필요.
