---
id: endpoint.route-docs-workflow
kind: Endpoint
title: 라우트 /docs/workflow
definition: "GET /docs/workflow — 문서 레이아웃 안에 'Workflow' 문서 페이지를 끼워 렌더하는 중첩 클라이언트 사이드 라우트. Project → Plan → Unit → Cycle → Task 워크플로우와 decompose / contract / execute 루프를 설명한다."
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

문서 영역의 "Workflow" 페이지를 보여 주는 경로다. `/docs` 레이아웃을 그대로 두른 채 본문 자리에 워크플로우 설명을 채운다.

작업이 어떤 순서로 흘러야 하는지를 설명한다: 분해(decompose) → 계약(contract) → 실행(execute)의 세 단계를 순서대로 거치도록 강제하며, 순서를 어기면 작업이 파일에 손을 대기 전 도구 단계에서 거부된다는 점을 강조한다. Project → Plan → Unit → Cycle → Task 계층과 라이프사이클 훅, 모든 자식이 끝났을 때 데몬이 상위(유닛·사이클·플랜) 완료를 자동으로 이어 처리(cascade)하는 동작을 함께 다룬다.

## 요청 / 응답

- 들어오는 정보: 현재 경로가 `/docs/workflow`라는 사실. 별도 질의어·식별자는 없다.
- 돌려주는 결과: 문서 레이아웃 안에 렌더된 "Workflow" 문서 본문. decompose/contract/execute 순서, 워크플로우 계층, 라이프사이클 훅의 역할, 데몬의 완료 cascade 설명을 담는다. 이 페이지는 자신의 제목·설명 등 메타 정보를 스스로 설정한다.

## 권한 / 제약

공개 경로로 인증이 필요 없다. `/docs` 부모 라우트의 중첩 자식이므로 부모 레이아웃 안에서만 렌더되며, 단독 동작하지 않는다.

## provenance

라우팅은 `landing/src/App.tsx`에서 `/docs` 아래 `workflow` 자식 라우트로 선언된 것에서 확인된다. 페이지 본문(3단계 루프·계층·훅·cascade)은 해당 문서 페이지 컴포넌트(Workflow)에 있다. 매칭·렌더는 SPA 라우터(`component.spa-router`)가, 레이아웃은 문서 허브(`screen.docs-hub`)가 제공한다.

## 미확정 (OPEN)
- [ ] 없음 — 라우트 위치(중첩 자식)와 페이지 목적이 코드에서 명확히 확인됨.
