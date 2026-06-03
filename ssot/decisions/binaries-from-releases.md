---
id: decision.binaries-from-releases
kind: Decision
title: 플러그인은 얇은 셸로 두고 런타임 산출물은 GitHub Releases에서 받는다
purpose: "런타임 소스(CLI·데몬·웹)를 플러그인 레포에 함께 둘지, 별도 레포에 두고 컴파일된 산출물만 배포받을지"
definition: "플러그인 레포는 얇은 셸로 두고 CLI·데몬·웹 소스는 형제(sibling) 레포에 두며, 사용자는 컴파일된 산출물을 GitHub Releases에서 받아 설치 시 Rust 툴체인·npm install 없이 동작하게 한다."
relatesTo:
  - to: component.plugin-shell
    type: relates-to
    note: 이 결정이 정의하는 얇은 셸 — 훅·프롬프트·스킬·설정만 담는다
  - to: component.install-gate
    type: relates-to
    note: 산출물을 릴리스에서 받아 무결성 검증 후 설치하는 메커니즘
  - to: integration.github-releases
    type: relates-to
    note: 산출물 배포·다운로드 경로
  - to: component.cli
    type: relates-to
    note: 형제 레포에서 빌드되어 릴리스로 배포되는 Rust CLI 산출물
  - to: component.daemon
    type: relates-to
    note: 형제 레포에서 빌드되어 릴리스로 배포되는 데몬 산출물
  - to: component.web-dashboard
    type: relates-to
    note: 형제 레포에서 빌드되어 릴리스로 배포되는 웹 번들
  - to: component.desktop-app
    type: relates-to
    note: 핀이 null이라 첫 릴리스 전까지 설치에서 제외되는 데스크탑 산출물
supersedes: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-03
tags: [status:active, "type:decision"]
---

<!-- Decision은 append-only. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 supersedes로 잇는다. -->

> Decision은 append-only다. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 `supersedes`로 잇는다.

## 맥락 (Context)

Clawket의 실제 런타임은 Rust로 작성된 CLI와 데몬, 그리고 React 웹 대시보드로 이루어진다. 이들을 플러그인 레포에 소스째 함께 두면, 사용자가 플러그인을 설치할 때마다 Rust 툴체인으로 빌드하고 웹 의존성을 설치해야 한다. 이는 "Claude Code 플러그인을 켜면 바로 쓸 수 있다"는 경험과 어긋나고, 설치 진입 장벽과 실패 지점을 키운다. 한편 컴파일된 산출물을 어딘가에서 받아 쓰려면, 그 산출물이 위변조되지 않았음을 보장하고 버전을 고정·동기화할 방법이 필요하다.

## 결정 (Decision)

플러그인 레포는 **얇은 셸**로 유지한다 — 훅 매니페스트, 프롬프트, 스킬, MCP 등록 설정, 그리고 각 런타임 산출물의 버전을 고정한 핀 목록만 담는다. CLI·데몬·웹(및 데스크탑) 소스는 형제 레포(clawket GitHub org 산하 cli/daemon/web/desktop)에 분리해 둔다.

사용자 설치 시에는 세션 시작 훅의 설치 게이트가 핀에 적힌 버전의 컴파일된 산출물을 GitHub Releases에서 내려받아, 체크섬으로 무결성을 검증한 뒤 원자적으로 설치한다. 버전 마커가 일치하면 이후 실행은 무동작(no-op)이 되어 멱등적이다. 그 결과 일반 사용자는 Rust 툴체인이나 웹 의존성 설치 없이 동작하고, 소스 빌드는 CLI·데몬을 직접 개발하려는 경우에만 필요하다.

데스크탑 산출물의 핀은 첫 릴리스가 나오기 전까지 비워(null) 두어 설치 게이트가 해당 항목을 건너뛰게 한다.

## 근거와 결과 (Consequences)

- **설치 진입 장벽 제거**: 사용자는 Node.js만 있으면 되고 Rust 툴체인·npm install이 불필요하다. 플러그인을 켜는 즉시 런타임이 갖춰진다.
- **무결성·버전 고정**: 산출물은 체크섬으로 검증되고 핀 목록으로 버전이 고정되므로, 플러그인·CLI·데몬·웹이 서로 맞는 버전 조합으로만 설치된다. GitHub Releases가 외부 네트워크가 닿는 유일한 경로다.
- **관심사 분리**: 플러그인(통합·강제·워크플로우 진입점)과 런타임(상태·검색·API)이 레포 단위로 분리되어, 각자의 릴리스 주기를 따른다.
- **단계적 컴포넌트 도입**: 핀을 null로 두는 방식으로 아직 릴리스되지 않은 컴포넌트(데스크탑)를 설치 흐름에서 안전하게 제외할 수 있다.
- **대가**: 산출물 다운로드가 GitHub Releases 가용성에 의존하며, 핀과 형제 레포 릴리스가 어긋나면 설치가 실패할 수 있어 버전 동기화 규율이 필요하다.

[근거 provenance] clawket/README.md (얇은 플러그인 셸·sibling 레포·설치 게이트 서술), clawket/components.json (daemon/cli/web 버전 핀, desktop: null)
