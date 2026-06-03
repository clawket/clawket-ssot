---
id: endpoint.asset-install-sh
kind: Endpoint
title: 설치 스크립트(install.sh)
definition: 랜딩 사이트가 셸스크립트로 서빙하는 "curl | sh" 한 줄 설치 진입점으로, GitHub 릴리스에서 CLI·데몬 바이너리를 받아 설치한다.
realizedBy:
  - component.landing-site
implementedIn:
  - landing/public/install.sh
  - landing/vercel.json
relatesTo:
  - to: integration.github-releases
    type: depends-on
    note: 최신 릴리스 태그를 조회하고 OS·아키텍처별 바이너리 압축본을 내려받아 설치한다.
  - to: capability.copy-install-command
    type: backed-by
    note: 홈 화면이 안내하는 설치 명령이 이 스크립트를 가리킨다.
governedBy: []
impacts:
  - component.cli
  - component.daemon
consumedBy:
  - screen.home
integratesWith:
  - integration.github-releases
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

사용자가 터미널에서 `curl ... /install.sh | sh` 한 줄로 Clawket을 설치할 수 있게 하는 진입 스크립트다. 랜딩 사이트의 정적 자산으로 제공되며, 실행되면 현재 운영체제·CPU 아키텍처를 감지하고 GitHub 릴리스에서 최신(또는 환경변수로 핀한) CLI(`clawket`)와 데몬(`clawketd`) 바이너리를 내려받아 사용자 홈의 표준 경로(`~/.local/share/clawket/bin`)에 설치한 뒤 실행 경로(`~/.local/bin`)로 심볼릭 링크를 건다. 설치 후에는 검증·데몬 시작·제거 방법을 안내한다.

대상은 플러그인 없이 CLI·데몬을 직접 설치하려는 사용자다.

## 요청 / 응답

이 자산은 HTTP로 셸스크립트 본문(텍스트)을 그대로 반환한다. 응답 헤더에서 콘텐츠 타입을 `text/x-shellscript` 로 고정해 브라우저가 다운로드·해석하지 않고 파이프로 흘려보낼 수 있게 한다. 캐시는 짧게(약 5분) 잡혀 스크립트 갱신이 빠르게 반영된다.

스크립트 동작은 환경변수로 조정 가능하다 — CLI/데몬 버전 핀, 설치 루트, 심볼릭 링크 대상 디렉터리. 설치에 실패하면(네트워크·지원되지 않는 OS/아키텍처·GitHub API 레이트리밋) 빨간 에러 메시지와 함께 비정상 종료한다. 미지원 OS는 Windows(WSL2 사용 안내)와 그 외이며, macOS·Linux의 arm64/x86_64만 지원한다.

## 권한 / 제약

공개 자산으로 누구나 접근한다. 사이트의 SPA rewrite 규칙은 이 경로를 단일 진입 문서로 재작성하지 않도록 예외 처리되어 있어, 실제 스크립트 파일이 그대로 서빙된다. 설치에는 `curl`·`tar`·`uname` 명령이 필요하며 없으면 즉시 실패한다.

## provenance

스크립트 본문은 `landing/public/install.sh`, 셸스크립트 MIME·캐시 헤더와 rewrite 예외는 `landing/vercel.json` 에 정의된다. 바이너리 출처는 GitHub Releases 연동이며, 결과적으로 CLI·데몬 컴포넌트가 사용자 머신에 설치된다.

## 미확정 (OPEN)
- [ ] OPEN: 스크립트가 가리키는 릴리스 레포 기본값(clawket/cli, clawket/daemon)과 배포 URL 도메인의 운영 안정성은 owner 확인 필요.
