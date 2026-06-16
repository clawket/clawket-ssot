---
id: endpoint.asset-uninstall-sh
kind: Endpoint
title: 제거 스크립트(uninstall.sh)
definition: 설치된 Clawket 바이너리·심볼릭 링크를 제거하고 선택적으로 사용자 데이터까지 지우는 셸스크립트 자산으로, install.sh와 동일한 셸스크립트 MIME로 서빙된다.
realizedBy:
  - component.landing-site
implementedIn:
  - landing/public/uninstall.sh
  - landing/vercel.json
relatesTo:
  - to: endpoint.asset-install-sh
    type: depends-on
    note: 설치 스크립트의 짝으로, 동일한 서빙 규칙(셸스크립트 MIME 핀·rewrite 예외)을 공유한다.
governedBy: []
impacts:
  - component.cli
  - component.daemon
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

설치한 Clawket을 되돌리는 진입 스크립트다. 기본 동작은 바이너리(`clawket`·`clawketd`)와 실행 경로의 심볼릭 링크만 제거하는 안전한 제거이고, `--purge` 옵션을 주면 사용자 데이터(SQLite DB·벡터 인덱스·캐시·설정·로그/감사 추적)까지 함께 삭제한다. 제거 스크립트는 install.sh의 짝으로 같은 방식으로 서빙된다.

대상은 직접 설치한 Clawket을 정리하려는 사용자다.

## 요청 / 응답

HTTP로 셸스크립트 본문(텍스트)을 그대로 반환한다. 콘텐츠 타입은 `text/x-shellscript` 로 고정되어 파이프 실행이 가능하다. 스크립트는 인자에 따라 분기한다 — 인자 없음/기본은 바이너리·링크만 제거, `--purge` 는 데이터까지 제거, `--yes` 는 확인 없이 진행한다.

데이터 삭제는 되돌릴 수 없고 별도 백업이 없으므로, `--purge` 는 파이프(비대화형 셸)로 실행하면 기본적으로 거부되며 `--yes` 를 명시해야만 비대화형으로 강행된다. 삭제 대상 경로는 XDG 표준 환경변수로 재정의할 수 있다.

## 권한 / 제약

공개 자산으로 누구나 접근하며, install.sh와 동일하게 사이트의 SPA rewrite 예외에 포함되어 스크립트 파일 그대로 서빙된다. `--purge` 의 비가역성 때문에 비대화형 환경에서의 무인 실행은 명시적 `--yes` 가 있을 때만 허용하는 안전 게이트가 핵심 제약이다.

## provenance

스크립트 본문은 `landing/public/uninstall.sh`, 셸스크립트 MIME·캐시 헤더와 rewrite 예외는 `landing/vercel.json` 에 정의된다.

## 미확정 (OPEN)
- [ ] OPEN: rewrite 예외 패턴은 `uninstall` 토큰을 포함하나, 캐시·MIME 헤더 규칙이 install.sh와 완전 동등하게 적용되는지 vercel 헤더 매칭 우선순위 관점에서 owner 확인 권장.
