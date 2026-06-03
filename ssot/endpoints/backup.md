---
id: endpoint.backup
kind: Endpoint
title: clawket backup
definition: "전체 데이터(DB + 첨부 지식)를 다른 기기로 옮기거나 오프사이트 보관하기 위해 휴대 가능한 tar.gz 압축본으로 내보내는 CLI 명령."
realizedBy:
  - component.cli
  - component.daemon-client
implementedIn:
  - cli/src/main.rs
relatesTo:
  - to: component.daemon
    type: calls
    note: "데몬에 백업 생성을 요청한다(POST /backup)."
  - to: component.sqlite-storage
    type: reads
    note: "DB 전체 상태를 아카이브로 내보내는 원천."
  - to: concept.knowledge
    type: reads
    note: "첨부된 지식 항목도 함께 아카이브에 포함된다."
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

`clawket backup` 은 Clawket 이 보관 중인 모든 작업 데이터를 한 덩어리로 묶어 내보내는 명령이다. 작업 상태(프로젝트·플랜·유닛·사이클·태스크 등)와 첨부된 지식 항목까지 함께 휴대 가능한 압축 아카이브로 만들어, 다른 기기로 이전하거나 외부에 안전하게 보관할 수 있게 한다. 로컬 우선 도구이므로 클라우드 동기화 대신 이 명령이 데이터 이동·보존의 표준 수단이다.

## 요청 / 응답

요청 시 아카이브를 저장할 경로를 지정할 수 있으며, 지정하지 않으면 타임스탬프가 붙은 기본 파일명으로 현재 위치에 만들어진다. 백업 범위를 특정 프로젝트로 한정할 수도 있고, 한정하지 않으면 모든 프로젝트를 대상으로 한다. 응답은 생성된 아카이브의 결과 정보다(생성 경로 등).

## 권한 / 제약

데몬이 실행 중이어야 한다. 데이터를 읽어 내보내는 동작이므로 원본 상태는 변경하지 않는다. 결과물은 압축 아카이브 한 개로, 복원은 짝이 되는 restore 명령으로 수행한다.

## provenance

CLI 정의·디스패치는 `cli/src/main.rs`(Backup 서브커맨드: output 경로·project 범위 옵션을 받아 데몬의 백업 생성을 요청). 실제 아카이브 생성은 데몬 측에서 수행된다. "DB 전체 상태와 첨부 지식을 프로젝트 단위로 범위 지정 가능하게, 휴대 가능한 단일 압축 아카이브로 내보내는 명령"이라는 의미만으로 재구현 가능하다.

## 미확정 (OPEN)
- [ ] OPEN: 데몬의 백업 생성 경로가 아카이브에 정확히 어떤 데이터(테이블·첨부물 범위)를 담는지, 응답 형태를 데몬 측에서 확인할 것.
