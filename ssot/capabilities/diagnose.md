---
id: capability.diagnose
kind: Capability
title: 설치 진단과 쓰기 경로 스모크 검증
purpose: "Clawket 설치가 제대로 동작하는지 진단하고, 데몬의 전체 쓰기 경로가 실제로 작동하는지 일회성 실증으로 확인한다."
servesPersona:
  - persona.human-operator
  - persona.llm-coding-agent
realizedBy:
  - component.cli
  - component.daemon
implementedIn:
  - cli/src/main.rs
  - cli/src/verify.rs
  - cli/src/doctor.rs
relatesTo:
  - { to: endpoint.cli-doctor, type: realizes, note: "doctor 명령으로 데몬 건강·바이너리·경로분리 불변식·연결성·티어 분포·에스컬레이션율·플러그인 설치·로케일·스킬을 진단한다." }
  - { to: endpoint.cli-verify, type: realizes, note: "verify 명령으로 일회용 프로젝트를 만들고 지웠다가 전체 쓰기 경로를 실증한다." }
  - { to: endpoint.daemon-health, type: depends-on, note: "두 진단 모두 먼저 데몬 헬스 엔드포인트를 핑해 데몬 생존을 확인한다." }
  - { to: component.daemon, type: relates-to, note: "진단·검증의 대상이 로컬 데몬과 그 쓰기 경로다." }
impacts:
  - component.daemon
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:capability
---

## 사용자가 할 수 있는 일

설치가 끝났거나 무언가 이상할 때, 두 단계로 상태를 확인한다. 먼저 읽기 전용 진단으로 데몬이 살아 있는지, 바이너리가 제대로 깔렸는지, 경로 분리 같은 불변식이 지켜지는지, 티어 분포·에스컬레이션율 같은 운영 지표는 정상인지, 플러그인·로케일·스킬이 갖춰졌는지를 한 번에 점검한다. 진단은 아무것도 바꾸지 않으며, 실패가 있으면 비정상 종료로 알려 자동화에서 게이트로 쓸 수 있다.

그다음, 진단만으로는 알 수 없는 "쓰기가 실제로 되는가"를 실증 검증으로 확인한다. 일회용 임시 프로젝트를 만들었다가 곧바로 지워, 데몬의 HTTP 쓰기 경로가 처음부터 끝까지 연결되어 있음을 한 번에 증명한다. 실데이터를 건드리기 전 안심 장치다.

## 행위

- 진단(doctor): 데몬 건강, 바이너리, 경로 분리 불변식, 연결성, 티어 분포, 에스컬레이션율, 플러그인 설치, 로케일 체인, 스킬을 점검한다. 결과를 사람이 읽는 텍스트 또는 기계용 JSON으로 내고, 특정 플랜으로 범위를 좁히거나 에스컬레이션율 리포트만 따로 뽑을 수 있다. 실패가 하나라도 있으면 비정상 종료한다.
- 검증(verify): 먼저 데몬 헬스를 핑하고, 일회용 프로젝트를 생성한 뒤 삭제해 전체 쓰기 경로를 확인한다. 삭제는 딸린 플랜·태스크까지 연쇄 정리한다. 드라이런으로는 데몬에 접속하지 않고 수행할 단계 목록만 출력한다.

## 시스템 흐름

진단은 데몬에 읽기 요청만 보내 각 점검 항목의 상태를 모아 보고하고 어떤 데이터도 바꾸지 않는다. 검증은 능동적으로 쓴다 — 데몬 헬스 확인 후 임시 프로젝트를 생성하고 곧바로 삭제하여, 헬스 핑만으로는 알 수 없는 실제 쓰기·연쇄 삭제 경로가 살아 있음을 일회성으로 증명한다. 검증은 설치 스크립트의 성공 게이트로도 쓰인다.

## 어디에 구현되어 있나

CLI(Rust 바이너리)에 doctor와 verify 명령이 정의되어 있다. 진단 점검 항목 구현은 doctor 모듈에, 임시 프로젝트 생성·삭제로 쓰기 경로를 훑는 스모크 로직은 verify 모듈에 있다. 실제 상태 조회·생성·삭제는 데몬이 수행한다.

## 미확정 (OPEN)
- [ ] OPEN: doctor의 개별 점검 항목 판정 기준(불변식 위반 조건, 에스컬레이션율 임계 등) 세부는 진단 점검 모듈 추가 확인 필요.
