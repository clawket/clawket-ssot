---
id: endpoint.route-docs-cli
kind: Endpoint
title: 라우트 /docs/cli
definition: "GET /docs/cli — 문서 레이아웃 안에 'CLI Reference' 문서 페이지를 끼워 렌더하는 중첩 클라이언트 사이드 라우트. clawket 명령들의 길잡이 지도이며 --help가 정식 레퍼런스임을 안내한다."
realizedBy: [component.spa-router]
implementedIn:
  - landing/src/App.tsx
relatesTo:
  - { to: endpoint.route-docs, type: depends-on, note: "/docs 부모 라우트의 중첩 자식. 부모 레이아웃 본문에 끼워 렌더된다." }
  - { to: component.landing-site, type: belongs-to, note: "랜딩 사이트 SPA 문서 영역의 한 페이지." }
governedBy: []
impacts: []
consumedBy: [screen.docs-hub]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

문서 영역의 "CLI Reference" 페이지를 보여 주는 경로다. `/docs` 레이아웃을 그대로 두른 채 본문 자리에 CLI 명령 안내를 채운다.

이 페이지의 입장은 분명하다: 모든 옵션의 정식 레퍼런스는 `--help`이며, 이 페이지는 그 위를 빠르게 훑어보게 해 주는 길잡이 지도다. `clawket` 바이너리가 로컬 데몬과 유닉스 소켓(루프백 TCP 폴백)으로 통신하는 단일 실행 파일이라는 점, 그리고 project·plan·cycle·task·dashboard·doctor·mcp 같은 상위 하위 명령이 무엇을 하는지를 짧게 정리한다.

## 요청 / 응답

- 들어오는 정보: 현재 경로가 `/docs/cli`라는 사실. 별도 질의어·식별자는 없다.
- 돌려주는 결과: 문서 레이아웃 안에 렌더된 "CLI Reference" 문서 본문. `--help` 3단 사용법(전체 / 하위 명령 / 하위 명령의 동작)과 상위 하위 명령들의 한 줄 설명 목록을 담는다. 이 페이지는 자신의 제목·설명 등 메타 정보를 스스로 설정한다.

## 권한 / 제약

공개 경로로 인증이 필요 없다. `/docs` 부모 라우트의 중첩 자식이므로 부모 레이아웃 안에서만 렌더되며, 단독 동작하지 않는다. 이 페이지는 명령 레퍼런스의 원본(SoT)이 아니라 길잡이임을 스스로 명시한다 — 권위 있는 출처는 실제 CLI의 `--help` 출력이다.

## provenance

라우팅은 `landing/src/App.tsx`에서 `/docs` 아래 `cli` 자식 라우트로 선언된 것에서 확인된다. 페이지 본문(`--help` 안내·하위 명령 지도)은 해당 문서 페이지 컴포넌트(CLI)에 있다. 매칭·렌더는 SPA 라우터(`component.spa-router`)가, 레이아웃은 문서 허브(`screen.docs-hub`)가 제공한다.

## 미확정 (OPEN)
- [ ] 없음 — 라우트 위치(중첩 자식)와 페이지 목적이 코드에서 명확히 확인됨.
