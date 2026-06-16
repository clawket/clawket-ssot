---
id: endpoint.cli-update-versioncheck
kind: Endpoint
title: clawket update / version-check
definition: "최신 릴리스를 받아 CLI·데몬 바이너리를 통째로 새 버전으로 교체하는 자가 업데이트(update)와, 설치 없이 더 새 버전이 있는지만 확인하는 점검(version-check) 두 명령."
realizedBy:
  - component.cli
  - component.daemon
implementedIn:
  - cli/src/main.rs
relatesTo:
  - to: integration.github-releases
    type: backed-by
    note: "새 버전 산출물과 최신 버전 정보를 GitHub Releases에서 가져온다."
  - to: component.cli
    type: mutates
    note: "update는 설치된 CLI 바이너리를 새 버전으로 원자 교체한다."
  - to: component.daemon
    type: mutates
    note: "update는 데몬 바이너리도 함께 교체한다(다음 재시작부터 적용)."
governedBy: []
impacts:
  - component.cli
  - component.daemon
  - component.install-gate
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
Clawket의 실행 본체는 컴파일된 두 바이너리(명령줄 도구 CLI와 백그라운드 데몬)다. 시간이 지나면 새 버전이 공개되는데, 이 두 명령은 그 새 버전을 다루는 사용자용 진입점이다.

- `clawket update`: 최신 버전을 내려받아 지금 설치된 CLI와 데몬을 새 것으로 교체한다. 교체는 한 번에 깔끔하게 바꿔치기하는 방식이라 중간에 깨진 상태로 남지 않는다. 이미 돌고 있는 데몬 프로세스는 그대로 계속 동작하다가 다음에 다시 시작될 때부터 새 버전으로 바뀐다.
- `clawket version-check`: 아무것도 설치하지 않고, 지금 쓰는 버전보다 더 새 버전이 나와 있는지만 알려 준다. 업데이트 여부를 사용자가 판단하게 돕는 가벼운 확인 명령이다.

## 요청 / 응답
- update에 보내는 정보: "미리보기만 할지"(실제 설치 없이 무엇을 받을지만 확인), 그리고 "특정 버전으로 고정할지"(예: 특정 릴리스 번호로 명시). 둘 다 생략하면 최신 버전을 그대로 설치한다.
- update가 돌려주는 정보: 무엇을 내려받아 어떻게 교체했는지(미리보기 시에는 무엇을 받을 예정인지)에 대한 결과 보고.
- version-check가 돌려주는 정보: 현재 버전과 가장 최신 버전을 비교한 결과 — 더 새 버전이 있는지 여부.

## 권한 / 제약
- 두 명령 모두 외부 네트워크로 나가 GitHub Releases를 조회한다. 이는 Clawket이 외부와 통신하는 몇 안 되는 경로 중 하나다(평상시 데이터는 로컬에만 머문다).
- update는 설치된 바이너리를 교체하는 작업이므로, 적용 전에 미리보기 모드로 무엇이 바뀔지 확인할 수 있다.
- 버전을 명시해 고정하면 최신이 아닌 특정 버전으로 맞출 수 있다(롤백·재현용).

## provenance
두 명령의 정의·핸들러는 `cli/src/main.rs`에 있다. CLI는 자가 업데이트 처리를 데몬에 위임하고, 새 산출물·최신 버전 정보는 외부 연동(`integration.github-releases`)을 통해 가져온다. 동일한 다운로드·검증 경로는 세션 시작 시의 설치 게이트(`component.install-gate`)와 성격을 공유한다.

## 미확정 (OPEN)
- [ ] OPEN: update가 호출하는 자가 업데이트 적용 경로가 현재 데몬 라우트와 정확히 어떻게 매핑되는지 확인 필요(검사 시점 빌드 바이너리와 데몬 소스 간 명령 표면 차이 가능성). 교체 시 무결성 검증(SHA256 등)이 이 경로에서도 동일하게 적용되는지 확인 필요.
