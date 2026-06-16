---
id: endpoint.completions
kind: Endpoint
title: clawket completions (셸 자동완성 스크립트 생성)
definition: bash·zsh·fish·powershell·elvish 중 지정한 셸의 명령 자동완성 스크립트를 표준출력으로 찍어내는 명령. 상태를 바꾸지 않는 순수 생성기.
realizedBy:
  - component.cli
implementedIn:
  - cli/src/main.rs
relatesTo:
  - { to: component.cli, type: backed-by, note: "CLI 바이너리가 자기 명령 구조로부터 자동완성 스크립트를 생성한다" }
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
사용자가 터미널에서 clawket 명령을 칠 때 Tab 키로 서브커맨드·옵션을 자동완성할 수 있도록, 해당 셸이 이해하는 자동완성 스크립트를 만들어 화면(표준출력)에 출력하는 명령이다. 지원 셸은 bash·zsh·fish·powershell·elvish 다섯 가지다. 사용자는 출력된 스크립트를 자기 셸의 자동완성 설정 위치에 붙여 넣어(예: bash는 bash_completion에 덧붙이고, zsh는 함수 경로에 저장) 자동완성을 켠다.

이 명령은 어떤 상태도 바꾸지 않고 데몬도 필요로 하지 않는 순수 생성기다. CLI가 자기 자신의 명령 구조를 알고 있으므로 그로부터 스크립트를 즉석에서 만들어낸다.

## 요청 / 응답
- 보내는 것: 대상 셸 이름 하나(bash/zsh/fish/powershell/elvish).
- 돌려받는 것: 해당 셸용 자동완성 스크립트 텍스트 — 표준출력으로 나오며, 사용자가 파일로 리다이렉트해 설치한다.

## 권한 / 제약
같은 기기에서 CLI를 실행하는 누구나 호출할 수 있다. 데몬·DB·네트워크가 전혀 필요 없는 로컬 순수 함수다. 셸 인자는 지원하는 다섯 셸 중 하나여야 한다.

## provenance
명령 정의는 cli/src/main.rs의 Completions 변형(168-177행)에 있으며, 사용 예시(각 셸별 설치 경로)가 명령 도움말 주석에 함께 적혀 있다. 데몬·저장소에 의존하지 않으므로 CLI 바이너리(component.cli) 단독으로 제공된다. 코드가 사라져도 위 정책(다섯 셸 지원, 표준출력 생성, 상태 무변경)만으로 재구현 가능하다.

## 미확정 (OPEN)
- (없음 — 표준 셸 자동완성 생성 명령으로 정책이 단순하고 외부 의존이 없다.)
