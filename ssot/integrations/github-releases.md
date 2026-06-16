---
id: integration.github-releases
kind: Integration
title: GitHub Releases 바이너리 배포
definition: 얇은 플러그인 셸이 자신은 담지 않는 런타임(CLI·데몬·웹, 그리고 미래의 데스크탑) 산출물을 별도 sibling 레포의 GitHub 릴리스에서 내려받아 SHA256SUMS로 무결성을 검증한 뒤 설치하는, Clawket의 유일한 외부 네트워크 경로.
purpose: 플러그인 자체는 훅·프롬프트·스킬·MCP 등록만 담는 얇은 셸이라 컴파일된 무거운 런타임 바이너리를 직접 포함하지 않는다. 그 런타임을 사용자의 머신에 올리려면 어딘가에서 받아와야 하며, 그 단 하나의 수신 경로가 GitHub Releases다. 받아온 바이너리가 변조·손상되지 않았음을 보장하기 위해 무결성 검증이 필수다.
integratesWith:
  - component.install-gate
  - component.claude-adapter
  - component.cli
  - component.daemon
  - component.web-dashboard
implementedIn:
  - clawket/adapters/shared/claude-hooks.cjs
  - clawket/scripts/setup.cjs
impacts:
  - component.install-gate
  - endpoint.cli-update-versioncheck
  - endpoint.cli-doctor
  - component.desktop-app
  - platform.local-first
relatesTo:
  - to: component.install-gate
    type: relates-to
    note: 이 연동을 실제로 호출해 멱등 설치를 수행하는 게이트
  - to: integration.claude-code-hooks
    type: relates-to
    note: 세션 시작 훅이 설치 게이트를 트리거하면서 이 다운로드 경로가 작동한다
governedBy:
  - invariant.local-only-no-egress
  - invariant.path-separation-xdg
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:integration
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로 — 무엇을·왜·누가·어떤 규칙·무슨 데이터. 코드(테이블·필드·경로·SQL) 옮겨적기 금지(식별자는 provenance/근거에만). 코드 분기/의도 불명은 판정 말고 OPEN. -->

## 무엇과 연동하나

상대편은 GitHub의 릴리스 배포 인프라다. Clawket 플러그인 셸은 런타임을 직접 품지 않고, CLI·데몬·웹(그리고 향후 데스크탑)을 각각 별도의 sibling 레포에 릴리스로 올려둔 뒤, 사용자 머신에서 그 릴리스 자산을 내려받는다. 내려받을 버전은 플러그인이 핀으로 고정해 둔 컴포넌트 버전 목록을 따른다.

받는 방식은 단순 HTTPS 다운로드다. 각 자산을 받을 때 스트리밍하면서 동시에 해시를 계산하고, 같은 릴리스에 올라온 체크섬 목록 파일(SHA256SUMS)을 받아 대조한다. 해시가 일치할 때에만 임시 파일을 정식 위치로 원자적으로 옮긴다. 즉 변조·손상된 바이너리가 설치 위치에 도달하는 일이 없도록, 검증 통과가 설치 확정의 선행 조건이다. 체크섬 파일 자체가 막힌 폐쇄망·기업 프록시 환경을 위해 검증을 건너뛰는 명시적 탈출구(환경 변수)가 있으나 권장되지 않는다.

이것이 Clawket이 외부 네트워크로 나가는 유일한 경로다. 작업 데이터·지식·임베딩은 모두 로컬에 머무르고, 외부로 나가는 트래픽은 이 바이너리 수신뿐이다.

## 구현 위치 (provenance)

다운로드·해시 검증·원자적 설치 로직은 공유 핸들러(`clawket/adapters/shared/claude-hooks.cjs`)에 있다. 여기서 컴포넌트별 sibling 레포(환경 변수로 재정의 가능, 기본은 CLI/데몬/웹/데스크탑 각각의 레포)와 릴리스 자산 URL을 구성하고, 체크섬 목록을 받아 대조한 뒤 설치한다. 같은 다운로드·검증 흐름을 셋업 스크립트(`clawket/scripts/setup.cjs`)도 공유한다.

이 연동을 실제로 구동하는 내부 컴포넌트는 설치 게이트(`component.install-gate`)이며, 게이트는 세션 시작 시점에 멱등적으로 호출되어 이미 같은 버전이 깔려 있으면 무동작한다. 수신되는 산출물의 소비처는 CLI(`component.cli`)·데몬(`component.daemon`)·웹 대시보드(`component.web-dashboard`)다.

데스크탑 컴포넌트는 현재 버전 핀이 비어 있어(null) 다운로드 대상에서 빠진다 — 즉 데스크탑은 설치 게이트에서 무동작으로 처리되며, 수동 설치를 위한 힌트만 노출된다.

## 불변식

- 이 다운로드를 제외하면 어떤 데이터도 외부로 나가지 않아야 한다(`invariant.local-only-no-egress`). 바이너리 수신은 외부 egress의 유일한 허용 예외다.
- 받은 자산은 정해진 경로 분리 규칙에 따라 설치되어야 한다(`invariant.path-separation-xdg`) — 데이터 디렉터리와 실행 바이너리·캐시가 섞이지 않는다.
- 무결성 검증이 통과해야만 설치가 확정된다. 검증 우회는 명시적 옵트인일 때만 허용된다.

## 영향 범위

이 경로가 막히거나 검증에 실패하면 설치 게이트(`component.install-gate`)가 런타임을 올리지 못해 CLI·데몬·웹 전체가 부재 상태가 된다. 버전 갱신 명령(`endpoint.cli-update-versioncheck`)과 진단(`endpoint.cli-doctor`)도 이 경로 위에서 동작한다. 데스크탑(`component.desktop-app`)은 버전 핀이 비어 있어 현재 이 경로의 대상이 아니다. 외부로 나가는 트래픽을 이 하나로 한정한다는 점에서 로컬 우선 플랫폼 성격(`platform.local-first`)과 직결된다.

## 미확정 (OPEN)
- [ ] OPEN: 체크섬 목록 파일이 404/네트워크 오류일 때 "검증 생략 후 설치 진행" 폴백의 정확한 경계(언제 경고만 하고 통과시키는지)는 component.install-gate 노드에 위임.
