---
id: component.install-gate
kind: SystemComponent
title: 설치 게이트(ensureInstalled)
definition: 세션이 시작될 때 CLI·데몬·웹 바이너리가 올바른 버전으로 깔려 있는지 보장하는 멱등적 설치 절차다. components.json에 고정된 버전을 GitHub Releases에서 받아 SHA256으로 무결성을 검증하고, 검증을 통과한 뒤에만 원자적으로 최종 위치에 옮기며, 버전 마커가 일치하면 재실행을 아무 일도 하지 않는 동작(no-op)으로 만든다.
purpose: "사용자가 Rust 툴체인 없이도 플러그인 설치 즉시 올바른 버전의 런타임 바이너리를 갖추게 한다. 다운로드 무결성·원자성·재실행 멱등성을 보장해, 부분 설치된 손상 바이너리가 정상 설치로 위장하지 못하게 막는다."
realizedBy:
  - platform.claude-code-plugin
implementedIn:
  - clawket/adapters/shared/claude-hooks.cjs
  - clawket/scripts/setup.cjs
  - clawket/components.json
dependsOn:
  - component.plugin-shell
consumesApi: []
providesApi: []
integratesWith:
  - integration.github-releases
impacts:
  - component.cli
  - component.daemon
  - component.web-dashboard
  - component.desktop-app
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:systemcomponent"]
relatesTo:
  - to: integration.github-releases
    type: depends-on
    note: 바이너리·번들과 SHA256SUMS를 받아오는 유일한 외부 네트워크 경로다.
  - to: endpoint.hook-session-start
    type: relates-to
    note: SessionStart 훅이 게이트를 실행한다(이미 최신이면 no-op) — 게이트는 그 훅에 의해 트리거되는 쪽이다.
  - to: component.desktop-app
    type: relates-to
    note: components.json의 desktop 핀이 null이면 데스크탑 설치는 건너뛴다(조건부 설치 대상).
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 분기/의도 불명은 OPEN. -->

## 책임

이 구성 요소는 "사용자가 플러그인을 깔면 알아서 올바른 런타임이 갖춰져 있게" 만드는 설치 절차다. 핵심 가치는 세 가지다. 첫째, Rust 툴체인이나 별도 빌드 없이 미리 컴파일된 바이너리(CLI, 데몬, 웹 번들)를 받아 쓰게 한다. 둘째, 멱등성 — 이미 올바른 버전이 깔려 있으면 다시 실행해도 아무 일도 일어나지 않는다. 셋째, 무결성·원자성 — 받은 파일이 손상되었거나 도중에 끊겼을 때 그것이 정상 설치로 위장하지 못하게 막는다.

어떤 버전을 깔지는 플러그인 릴리스마다 `components.json`에 못 박혀 있다(예: 데몬·CLI·웹은 특정 버전, 데스크탑은 아직 null). 게이트는 이 핀 버전과 실제 설치된 버전을 버전 마커 파일로 비교해, 일치하면 건너뛰고 어긋나면 다시 받는다. 데스크탑처럼 핀이 null인 구성 요소는 설치 자체를 건너뛴다.

## 경계와 의존

게이트는 플러그인 셸(`component.plugin-shell`)의 공용 핸들러(`claude-hooks.cjs`)와 수동/CI 진입점(`setup.cjs`) 안에 구현되어 있고, Node.js로 동작한다(설치 훅에만 Node가 필요). 외부로는 GitHub Releases 배포(`integration.github-releases`)에만 의존한다 — 이것이 Clawket의 유일한 외부 네트워크 경로다. 게이트는 자신이 설치하는 대상인 CLI·데몬·웹·데스크탑 구성 요소들에게 영향을 준다(존재 여부와 버전을 좌우).

무결성 검증은 같은 릴리스의 `SHA256SUMS` 동반 파일을 받아, 내려받은 아카이브의 SHA256 해시와 대조하는 방식이다. 폐쇄망·기업 프록시 환경처럼 SHA256SUMS 자체가 막힐 수 있는 경우를 위해 검증을 건너뛰는 환경변수 우회가 있고, 이 경우 경고를 남기며 통과시킨다.

## 통신 패턴

다운로드는 바이트를 흘려보내며 동시에 해시를 계산하는 스트리밍 방식이고, GitHub의 리다이렉트(릴리스 에셋 호스트로의 이동)를 그대로 따라간다. 받은 파일은 우선 임시 경로(`.tmp`/`.staging`)에 떨어뜨린 뒤, 해시 검증을 통과한 다음에야 최종 위치로 원자적으로 이름을 바꾼다(rename). 디렉터리형 산출물(웹 번들)은 스테이징 디렉터리에 풀어 둔 뒤 통째로 옮긴다. 동시 실행(여러 Claude 세션이 동시에 SessionStart를 거는 경우)에 대비해 구성 요소별 잠금 파일로 마커 갱신과 rename을 직렬화한다. macOS에서는 추출된 바이너리의 격리 속성(quarantine xattr)을 제거하려 시도한다.

## 하위 서브패키지 (책임 단위)

- 다운로드+해시 파이프라인: 스트리밍 다운로드, 리다이렉트 추종, GitHub 레이트리밋 처리, SHA256 계산.
- 무결성 검증: SHA256SUMS를 받아 파싱·대조하고, 환경변수로 우회 시 경고 처리.
- 버전 마커 관리: 설치된 버전을 마커 파일에 기록하고 핀 버전과 비교(멱등성의 근거), 잠금 파일로 동시 갱신 보호.
- 원자적 설치: 임시 경로 → 검증 → rename으로 부분 설치가 정상으로 위장하지 못하게 보장(바이너리 복사·디렉터리 추출 모두).
- 컴포넌트 매니페스트 로더: components.json에서 핀 버전을 읽고 null 핀은 건너뜀.

## 미확정 (OPEN)
- [ ] OPEN: 설치 게이트가 데몬 헬스 핑(설치 직후 기동 확인)까지 책임지는 범위와 데몬 자동 기동 글루의 경계 구분은 어댑터 측에서 추가 확인 필요.
- [ ] OPEN: 임베딩 모델 파일의 최초 다운로드는 이 게이트가 아니라 데몬이 첫 사용 시 수행하는 별개 경로(GitHub Releases 외부)임 — 외부 경로 정합성은 integration 측에서 확인.
