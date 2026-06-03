---
id: capability.copy-install-command
kind: Capability
title: 설치 안내·설치 스크립트 제공
definition: 랜딩 홈의 설치 버튼과 문서의 시작하기 페이지가 방문자에게 Clawket 설치 경로를 안내하며, 별도 정적 설치 스크립트는 한 줄 명령으로 CLI와 데몬 바이너리를 내려받아 설치한다.
purpose: 방문자가 막힘 없이 설치 단계로 진입하도록, 사람이 따라 할 수 있는 설치 진입로를 제공한다.
servesPersona:
  - persona.human-operator
  - persona.llm-coding-agent
realizedBy:
  - component.landing-site
implementedIn:
  - landing/public/install.sh
  - landing/src/pages/Home.tsx
  - landing/src/pages/docs/GettingStarted.tsx
relatesTo:
  - to: endpoint.asset-install-sh
    type: relates-to
    note: 한 줄 설치 명령이 가리키는 정적 스크립트가 이 엔드포인트로 제공된다
  - to: integration.github-releases
    type: relates-to
    note: 설치 스크립트가 바이너리를 받는 곳이 GitHub Releases다
  - to: endpoint.route-docs-getting-started
    type: relates-to
    note: 권장 설치 절차(플러그인 설치)는 시작하기 문서에 안내된다
impacts:
  - endpoint.asset-install-sh
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:capability
---

## 사용자가 할 수 있는 일

방문자는 두 가지 설치 진입로를 만난다. 첫째, 홈의 주요 버튼과 문서의 시작하기 페이지를 통해 권장 설치 절차(Claude Code에서 마켓플레이스 추가 후 플러그인 설치)를 안내받는다. 둘째, 플러그인 경로 대신 직접 바이너리를 깔고 싶은 사람은 사이트가 제공하는 정적 설치 스크립트를 한 줄 명령으로 받아 실행해 CLI와 데몬 바이너리를 자신의 사용자 디렉터리에 설치할 수 있다.

설치 스크립트는 운영체제와 CPU 종류(맥의 Apple Silicon/Intel, 리눅스 arm64/x86_64)를 자동 감지해 알맞은 바이너리를 받고, 사용자 PATH에 바이너리가 노출되도록 링크를 만들며, 끝나면 설치 확인·데몬 시작·제거 방법을 함께 안내한다. 윈도우 사용자에게는 WSL2 사용을 권한다.

## 행위

- 안내(홈): 홈의 주요 버튼이 제품 소스 저장소로 연결되고, 설치 흐름의 진입점을 강조한다.
- 안내(문서): 시작하기 페이지가 마켓플레이스 추가·플러그인 설치 명령과 첫 세션의 설치 게이트 동작을 설명한다.
- 제공(스크립트): 정적 설치 스크립트가 플랫폼 감지 → 최신 릴리스 버전 확인 → 바이너리 다운로드·압축 해제 → 사용자 bin에 설치·링크 → 설치 후 안내를 수행한다.

## 시스템 흐름

권장 경로에서는 방문자가 시작하기 문서를 읽고 Claude Code 안에서 플러그인을 설치하면, 첫 세션의 설치 게이트가 바이너리를 받는다(이 게이트는 플러그인 측 기능). 직접 경로에서는 방문자가 한 줄 명령으로 사이트의 정적 설치 스크립트(`endpoint.asset-install-sh`)를 받아 실행하고, 스크립트가 GitHub Releases(`integration.github-releases`)에서 CLI·데몬 바이너리를 플랫폼에 맞게 내려받아 설치한 뒤 PATH 링크와 사용 안내를 출력한다.

## 어디에 구현되어 있나

- `landing/public/install.sh` — 플랫폼 감지, 최신 릴리스 확인, 다운로드·압축 해제, 사용자 bin 설치·링크, 설치 후 안내를 담은 정적 설치 스크립트.
- `landing/src/pages/Home.tsx` — 홈의 설치/저장소 버튼.
- `landing/src/pages/docs/GettingStarted.tsx` — 마켓플레이스 추가·플러그인 설치 절차와 첫 세션 설치 게이트 설명.

이 기능은 랜딩 사이트(`component.landing-site`)에서 실현된다.

## 미확정 (OPEN)
- [ ] OPEN: 홈의 "설치" 버튼은 코드상 제품 소스 저장소로 연결되며, 클립보드로 명령을 복사하는 동작은 코드에서 확인되지 않음(노드 제목의 "copy"는 인벤토리 명칭, 실제 동작은 안내·스크립트 제공 중심).
