---
id: endpoint.verify
kind: Endpoint
title: clawket verify
definition: "설치 직후 스모크 체크 — 데몬 헬스를 확인한 뒤 일회용 프로젝트를 만들었다 지워서 전체 쓰기 경로가 동작하는지 증명한다(--dry-run은 데몬 접촉 없이 단계 목록만 출력)."
realizedBy: [component.cli, component.daemon-client]
implementedIn: [cli/src/main.rs, cli/src/verify.rs]
relatesTo:
  - { to: concept.project, type: mutates, note: "검증용 일회용 프로젝트를 생성했다가 즉시 삭제" }
  - { to: endpoint.health, type: reads, note: "쓰기 전에 데몬 헬스부터 핑" }
  - { to: endpoint.doctor, type: relates-to, note: "doctor는 읽기 전용, verify는 실제 쓰기 경로까지 확인하는 보완 관계" }
governedBy: []
impacts: []
consumedBy: [integration.clawketd]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
설치가 끝난 직후 "정말 끝까지 동작하는가?"를 한 번에 증명하는 스모크 체크다. 읽기 전용 진단(doctor)과 달리 이 명령은 실제로 데이터를 쓴다 — 진짜 사용자 데이터를 건드리기 전에, 임시 이름의 일회용 프로젝트를 만들었다가 곧바로 삭제해서 데몬의 쓰기 경로(생성·삭제)가 제대로 연결됐음을 보여 준다. 설치 스크립트가 바이너리를 내려받은 뒤 성공 게이트로 이 명령을 돌린다.

진행 단계는 네 가지다: CLI 버전 자가 확인 → 데몬 헬스 핑 → 일회용 프로젝트 생성 → 정리(그 프로젝트 삭제, 관련 플랜·태스크까지 연쇄 삭제). 미리보기 모드를 켜면 데몬에 접촉하지 않고 이 단계 목록만 출력하므로, 상태를 띄우지 않고도 바이너리가 정상 파싱되는지 CI·문서·설치 스크립트에서 확인할 수 있다.

## 요청 / 응답
- 보낼 수 있는 것: 미리보기 모드 여부(켜면 데몬 접촉·쓰기 없이 단계 목록만 출력).
- 돌려주는 것: 단계별 진행 표시(버전, 데몬 헬스 상태, 생성된 일회용 프로젝트 식별자, 삭제 완료)와 최종 결과 표시. 정리(삭제) 단계가 실패하면 "일회용 프로젝트가 남아 있다"는 경고와 함께 실패로 끝나, 잔재가 조용히 방치되지 않게 한다.

## 권한 / 제약
로컬에서 그 사용자 계정으로 실행하며, 일회용 프로젝트는 임시 디렉터리 아래 시각 기반 고유 이름으로 만들어져 실제 작업과 충돌하지 않는다. 라이브 모드는 데몬이 떠 있어야 하고, 헬스 핑이 실패하면 데몬을 먼저 시작하라는 안내와 함께 중단한다. 정상 흐름이면 생성한 일회용 프로젝트는 반드시 삭제되어 부수 효과를 남기지 않는다.

## provenance
명령 정의와 미리보기 옵션은 CLI 진입점(`cli/src/main.rs`)에, 4단계 스모크 흐름(헬스 핑·생성·삭제·정리 실패 처리)은 `cli/src/verify.rs`에 있다. 제공 주체는 CLI 바이너리이며 데몬 HTTP 호출은 데몬 클라이언트 컴포넌트를 통한다.

## 미확정 (OPEN)
- [ ] OPEN: 삭제 시 연쇄(cascade) 범위가 프로젝트 산하 어디까지인지(플랜/유닛/사이클/태스크 전부 vs 일부) 정확한 경계 확인 필요.
