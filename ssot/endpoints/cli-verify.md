---
id: endpoint.cli-verify
kind: Endpoint
title: clawket verify
definition: "설치 직후 데몬을 끝에서 끝까지 점검하는 스모크 명령으로, 헬스를 확인한 뒤 일회용 프로젝트를 만들었다 지워 전체 쓰기 경로가 살아 있는지 증명한다."
realizedBy:
  - component.cli
  - component.daemon
implementedIn:
  - cli/src/verify.rs
relatesTo:
  - to: endpoint.daemon-health
    type: reads
    note: 첫 단계로 데몬 헬스를 프로브한다
  - to: endpoint.cli-project
    type: relates-to
    note: 일회용 프로젝트 생성·삭제로 쓰기 경로를 검증한다
  - to: concept.project
    type: mutates
    note: 검증용 throwaway 프로젝트를 만들었다가 즉시 삭제한다
governedBy: []
impacts: []
consumedBy:
  - component.install-gate
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

설치가 끝난 직후 "데몬이 실제로 데이터를 쓸 수 있는가"를 증명하는 스모크 점검 명령이다. doctor가 건드리지 않고 읽기만 하는 진단이라면, verify는 능동적으로 쓴다 — 일회용 프로젝트를 하나 만들었다가 곧바로 지워, 데몬의 쓰기 경로가 끝까지 연결돼 있음을 한 번에 보여 준다. 설치 스크립트의 성공 게이트로 쓰여, 바이너리가 자리를 잡은 뒤 사용자가 자기 실제 데이터를 만지기 전에 돌린다.

## 요청 / 응답

네 단계를 순서대로 밟는다. (1) CLI 자기 버전 확인, (2) 유닉스 소켓을 통한 데몬 헬스 프로브, (3) 일회용 프로젝트 생성, (4) 정리 — 그 프로젝트를 삭제(딸린 플랜·태스크까지 연쇄 삭제). 각 단계의 진행 상황을 사람이 읽을 수 있게 출력하고, 모두 통과하면 OK로 끝난다. 헬스 프로브가 실패하면 "데몬을 먼저 띄우라"는 안내와 함께 멈추고, 정리 단계에서 삭제가 실패하면 "일회용 프로젝트가 남았다"는 경고와 함께 비정상 종료한다.

드라이런 옵션을 주면 데몬에 접속하지 않고 위 단계 목록만 출력한다 — 상태를 띄우지 않고 바이너리가 정상 파싱되는지만 보려는 CI나 설치 스크립트, 문서 용도다.

## 권한 / 제약

라이브 모드는 데몬이 떠 있어야 한다. 만드는 프로젝트는 시간 기반 난스를 붙인 이름과 임시 디렉터리 경로를 써서 사용자의 실제 프로젝트와 절대 충돌하지 않게 하고, 검증이 끝나면 즉시 삭제해 흔적을 남기지 않는다. 삭제가 실패하면 명령은 성공으로 끝나지 않는다(잔존 프로젝트를 침묵으로 넘기지 않는다). 드라이런 모드는 어떤 쓰기도, 데몬 접속도 하지 않는다.

## provenance

`cli/src/verify.rs`에서 확인했다. 네 단계 목록(STEPS), 라이브 모드의 헬스 프로브 → 일회용 프로젝트 생성 → 삭제 흐름, 난스 기반 이름과 임시 디렉터리 사용, 정리 실패 시 비정상 종료, 드라이런의 무접속 동작을 직접 추적했다. 프로젝트 생성·삭제는 데몬의 프로젝트 경로(`endpoint.cli-project`가 다루는 동일 자원)에 위임된다.

## 미확정 (OPEN)
- [ ] OPEN: 없음 — verify의 단계·동작은 소스에서 전부 확정됨. (이름/경로 충돌 회피와 연쇄 삭제 동작은 데몬 측 프로젝트 삭제 구현에 의존하므로 그쪽 불변식 노드에서 별도 검증.)
