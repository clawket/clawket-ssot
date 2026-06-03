---
id: invariant.spa-rewrite-exclusions
kind: Invariant
title: SPA fallback 라우팅 제외 목록
definition: "랜딩 사이트의 경로 재작성 규칙은 정적 자산·설치/제거 스크립트·블로그·튜토리얼·확장자가 붙은 경로를 fallback 대상에서 제외하고, 그 외 나머지 경로만 엔트리 HTML 로 보내 클라이언트 라우터가 처리하게 해야 한다."
governs:
  - component.spa-router
  - integration.vercel
  - screen.blog-post
  - screen.tutorial
  - capability.copy-install-command
governs_note: ""
implementedIn:
  - landing/vercel.json
  - landing/CLAUDE.md
decidedBy:
  - decision.csr-spa-no-ssr
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:invariant"]
---

## 제약

이 사이트는 클라이언트에서 라우팅하는 단일 페이지 앱이라, 호스팅은 알 수 없는 경로 요청을 엔트리 HTML 로 돌려보내 브라우저 안의 라우터가 화면을 그리게 한다("fallback"). 단, 이 fallback 이 모든 경로를 가로채면 안 된다. 정적 자산(번들·이미지·hero 미디어·OG 이미지), 설치/제거 스크립트, 정적 블로그·튜토리얼 페이지, 그리고 파일 확장자가 붙은 경로는 fallback 대상에서 반드시 제외되어야 한다. 즉 이 항목들은 진짜 파일 그대로 서빙되고, 그 외 나머지 경로만 엔트리 HTML 로 보내 라우터가 처리한다.

새 라우트나 새 정적 경로를 추가할 때는, 이 제외 목록이 새 경로를 의도치 않게 가로채는지(또는 가로채야 하는데 빠뜨렸는지) 반드시 확인해야 한다.

## 깨지면 무슨 일이 일어나나

제외 목록이 잘못되면 두 방향으로 깨진다. 정적 자산이나 설치 스크립트·블로그 같은 항목이 제외에서 빠지면, 그 요청이 엔트리 HTML 로 돌려보내져 실제 파일(스크립트·이미지·정적 페이지) 대신 앱 셸이 응답한다 — 설치 한 줄 명령이 셸 스크립트 대신 HTML 을 받아 깨지고, 블로그·튜토리얼이 안 열린다. 반대로 클라이언트 라우터가 처리해야 할 새 경로가 제외 목록에 잘못 걸리면, 그 화면이 fallback 을 못 받아 직접 접속 시 404 가 된다.

## 코드에서 어떻게 강제되나

이 제약은 랜딩 사이트의 호스팅 설정(`landing/vercel.json`)의 경로 재작성 규칙에 정의되어 있다. 재작성 규칙은 제외 패턴(정적 자산·설치/제거·블로그·튜토리얼·일반적인 정적 파일 이름·확장자가 붙은 경로)을 빼고 나머지를 엔트리 HTML 로 보낸다. 새 라우트 추가 시 제외 여부를 확인하라는 작업 지침은 작성 규칙(`landing/CLAUDE.md`)에 명시되어 있다.

자동 강제 장치는 없다. 새 경로가 제외 목록과 충돌하는지를 검사하는 CI 가 없으므로, 라우트·정적 경로를 추가할 때 사람이 재작성 규칙을 직접 점검하는 절차에 의존한다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy 는 CSR SPA 채택 결정에 연결했다(클라이언트 라우팅이 이 제외 목록을 필요하게 만든 근원). 별도 전용 결정이 필요하면 추가.
