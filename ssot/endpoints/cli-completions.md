---
id: endpoint.cli-completions
kind: Endpoint
title: clawket completions
definition: "지정한 셸(bash·zsh·fish·powershell·elvish)용 명령어 자동완성 스크립트를 생성해 표준 출력으로 내보내는 CLI 명령."
realizedBy:
  - component.cli
implementedIn:
  - cli/src/main.rs
relatesTo:
  - to: component.cli
    type: backed-by
    note: "CLI가 자신의 명령 구조로부터 자동완성 스크립트를 직접 생성한다 — 데몬을 거치지 않는다."
governedBy: []
impacts: []
consumedBy:
  - persona.human-operator
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의
명령줄에서 `clawket`을 칠 때 탭 키로 하위 명령·옵션을 자동완성하려면, 사용하는 셸이 이해하는 형식의 완성 규칙 스크립트가 있어야 한다. `clawket completions`는 그 스크립트를 셸 종류에 맞게 만들어 화면(표준 출력)으로 내보낸다. 사용자는 그 출력을 셸 설정 파일에 저장해 자동완성을 켠다.

지원하는 셸은 bash·zsh·fish·powershell·elvish 다섯 가지다. 셸 환경을 편하게 만들려는 사람 운영자가 설치 직후 한 번 쓰는 편의 명령이다.

## 요청 / 응답
- 보내는 정보: 어느 셸용 스크립트를 만들지(bash·zsh·fish·powershell·elvish 중 하나).
- 돌려주는 정보: 해당 셸이 읽을 수 있는 자동완성 스크립트 텍스트를 표준 출력으로 내보낸다. 지원하지 않는 셸 이름을 넘기면 오류를 알리고 비정상 종료한다.

## 권한 / 제약
- 이 명령은 전적으로 CLI 자체에서 처리된다 — 로컬 데몬이 꺼져 있어도, 프로젝트가 등록돼 있지 않아도 동작한다. 어떤 저장된 데이터도 읽거나 바꾸지 않는다.
- 지원 셸 다섯 가지 외의 값을 주면 거부한다.
- 출력은 스크립트 텍스트일 뿐이며, 자동완성을 실제로 켜려면 사용자가 그 출력을 셸 설정에 반영해야 한다.

## provenance
명령 정의·핸들러는 `cli/src/main.rs`에 있다. CLI는 자신의 명령 정의 구조로부터 셸별 완성 스크립트를 생성한다(데몬·저장소 비의존). 이 처리는 데몬 클라이언트를 만들기 전 단계에서 끝나고 바로 종료한다.

## 미확정 (OPEN)
- (없음 — CLI 단독 처리로 동작·제약이 명확함)
