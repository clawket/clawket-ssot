---
id: decision.docs-english-only
kind: Decision
title: docs 페이지는 i18n 미적용 영문 전용으로 두고 홈만 20 로케일 번역
purpose: docs 본문 페이지도 홈처럼 20 로케일로 다국어화할지, 영문 단일 언어로 둘지
definition: 홈 화면 카피는 20 로케일 사전(dict)으로 다국어화하되, /docs 하위 본문 페이지는 현재 i18n을 적용하지 않고 영문 텍스트를 JSX에 inline으로 작성한다.
relatesTo:
  - to: component.i18n
    type: governs
    note: i18n 적용 범위를 "홈만 포함, docs는 제외"로 규정한다.
  - to: endpoint.route-docs-cli
    type: relates-to
    note: 영문 inline 본문 + per-route useHead를 쓰는 대표 docs 페이지.
  - to: endpoint.route-docs-architecture
    type: relates-to
    note: 동일하게 영문 단일 언어로 작성되는 docs 본문.
  - to: endpoint.route-docs-workflow
    type: relates-to
    note: 동일하게 영문 단일 언어로 작성되는 docs 본문.
  - to: endpoint.route-docs-getting-started
    type: relates-to
    note: 동일하게 영문 단일 언어로 작성되는 docs 본문.
  - to: integration.i18n-locale
    type: relates-to
    note: 추후 docs를 다국어화할 경우 wrapper의 i18n 정책 체계를 따른다.
supersedes: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:decision"]
---

> Decision은 append-only다. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 `supersedes`로 잇는다.

## 맥락 (Context)

랜딩 홈 화면은 20개 로케일을 지원하며, 모든 카피는 로케일별 사전에서 가져온다(JSX에 하드코딩하지 않는다). 반면 /docs 하위 본문 페이지(getting-started, workflow, cli, architecture)는 제품의 CLI·워크플로우·아키텍처를 설명하는 기술 문서다. 이 기술 본문의 기능 주장은 wrapper 레포의 README 정본과 동기화되어야 하며, 빈도 높게 갱신된다.

홈과 동일하게 docs까지 20 로케일로 번역하면, 본문이 바뀔 때마다 모든 로케일을 동시에 갱신해야 하고 정본 동기화 비용이 20배가 된다.

## 결정 (Decision)

docs 본문 페이지에는 현재 i18n을 적용하지 않는다. 본문 텍스트는 영문 단일 언어로 각 페이지의 JSX에 inline으로 작성한다. 다만 각 docs 페이지도 홈과 동일하게 per-route 헤드 갱신(`useHead`)으로 자체 title·description·canonical·og를 설정한다(SEO 처리는 공통).

홈의 다국어 카피 체계(로케일 사전)와 docs의 영문 inline 본문은 분리된 두 작성 경로다.

## 근거와 결과 (Consequences)

- docs 본문은 wrapper README 정본과 동기화되어야 하는 기술 문서라 갱신이 잦다. 영문 단일 언어로 두면 정본과의 동기화가 한 언어로 끝나, 다국어 drift(언어 간 내용 어긋남) 위험을 없앤다.
- 대가로 비영어권 사용자에게 docs는 영문으로만 제공된다. 추후 다국어화로 확장할 경우 wrapper의 i18n 정책을 준수하도록 명시되어 있다(현재는 미적용 상태가 의도된 현 상태다).
- 이 결정은 "홈=다국어 사전, docs=영문 inline"이라는 작성 컨벤션의 경계를 정한다. docs 본문을 dict로 옮기는 일은 본 결정을 대체하는 새 결정이 필요하다.
