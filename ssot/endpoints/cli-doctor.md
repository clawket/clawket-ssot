---
id: endpoint.cli-doctor
kind: Endpoint
title: clawket doctor
definition: "로컬 Clawket 설치를 진단하는 읽기 전용 명령으로, 데몬 건강·바이너리·경로 분리·연결성·티어 분포·에스컬레이션율·플러그인 설치·로케일·스킬 등을 점검하고 하나라도 실패하면 비정상 종료한다."
realizedBy:
  - component.cli
  - component.daemon
implementedIn:
  - cli/src/doctor.rs
  - cli/src/doctor_checks.rs
relatesTo:
  - to: endpoint.daemon-health
    type: reads
    note: 데몬 건강 점검에 헬스 응답을 사용한다
  - to: concept.tier
    type: reads
    note: 태스크 티어 분포와 에스컬레이션율을 집계한다
  - to: component.install-gate
    type: relates-to
    note: 플러그인 설치 상태를 진단 항목으로 점검한다
governedBy:
  - invariant.path-separation
  - invariant.claude-only-vendor
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

"내 Clawket 설치가 건강한가, 어디가 고장 났나"를 한 번에 진단하는 읽기 전용 명령이다. 데몬이 왜 못 떴는지 같은 문제를 사용자와 데몬 보장 훅이 짚어낼 수 있도록, 구조화된 건강 스냅샷을 출력한다. 사람이 사용 가능한 텍스트로 보거나, 기계가 읽을 JSON으로 받을 수 있다. 모든 점검은 심각도 태그를 달고 끝까지 실행되므로, 한 번 돌리면 여러 문제를 동시에 볼 수 있다 — 한 번에 하나씩 고치며 재실행할 필요가 없다. 점검 중 하나라도 오류 심각도에 닿으면 명령은 비정상 종료 코드로 끝나, 스크립트나 훅이 실패를 감지할 수 있다.

## 요청 / 응답

옵션으로 JSON 출력 여부, 티어·에스컬레이션 집계를 특정 플랜으로 한정하는 필터, 그리고 에스컬레이션율 보고서(에스컬레이션 사유가 붙은 태스크의 수와 비율을 티어별로 집계)를 별도로 요청하는 플래그를 받는다.

출력은 여러 진단 구획으로 나뉜다 — 환경변수 오버라이드, XDG 경로 해석, 데몬, 데이터베이스, 훅, MCP, 플러그인 설치, 호환성, 다국어 로케일, 감사 로그, 데이터 손실 위험(활동 로그 보존량·DB 신선도·태스크 수 변화), 프로젝트 활성 상태, 레거시 잔존 데이터, 티어 분포와 에스컬레이션율, 스킬, 그리고 경로 분리 불변식 점검이다. 각 구획은 상태와 심각도를 함께 보고하며, 최종 종료 코드는 "어떤 점검이라도 오류 심각도에 닿았는가"의 순수한 함수다.

## 권한 / 제약

읽기 전용이다 — 어떤 상태도 변경하지 않는다(쓰기까지 검증하는 것은 별도의 verify 명령이다). 경로 분리 불변식 위반은 과거에는 즉시 중단 사유였지만, 지금은 오류 심각도로 표시하되 단락하지 않고 나머지 진단을 끝까지 보여 준다. 티어/에스컬레이션 집계는 Claude 계열만을 대상 모델로 삼는 v3 전제 위에서 동작한다.

## provenance

`cli/src/doctor.rs`(구획 구성·텍스트/JSON 출력·종료 코드 산정)와 `cli/src/doctor_checks.rs`(심각도 분류, 활동 로그 보존·DB 신선도·태스크 수 변화·월드-라이터블·레거시 잔존 판정 헬퍼)에서 확인했다. 데몬 건강은 헬스 응답(`endpoint.daemon-health`)에, 티어 분포·에스컬레이션율은 데몬이 집계하는 태스크 데이터에 의존한다.

## 미확정 (OPEN)
- [ ] OPEN: 각 구획이 오류/경고/정상 심각도를 가르는 정확한 임계값(예: 활동 로그 보존 한도, DB 신선도 기준 시간)은 헬퍼 함수 정독으로 수치까지 확정 필요.
