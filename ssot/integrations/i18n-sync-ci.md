---
id: integration.i18n-sync-ci
kind: Integration
title: i18n 문서 동기화 검사 워크플로(랜딩 CI)
purpose: "랜딩 저장소의 다국어 문서(영문 .md 와 그 한국어 번역 .ko.md)가 서로 어긋나지 않도록 변경 시점에 자동으로 검사해, 번역 누락·표류(drift)를 막는다."
definition: "GitHub Actions 워크플로가 main push 와 PR 의 문서 변경에 반응해, 정본 저장소(wrapper)에 게시된 표준 검사 스크립트를 내려받아 실행하는 연동. 영문/한국어 문서 짝의 표류를 검사한다."
integratesWith: []
implementedIn:
  - landing/.github/workflows/i18n-sync.yml
impacts: []
relatesTo:
  - { to: integration.i18n-locale, type: relates-to, note: "런타임 로케일 fallback 체인(제품 안의 다국어 동작)과 달리, 이쪽은 문서(.md/.ko.md) 번역 표류를 막는 CI 검사다 — 둘 다 다국어 정책 표면이지만 적용 대상이 다르다." }
governedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:integration"]
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드/식별자는 provenance·근거에만. -->

## 무엇과 연동하나

랜딩 저장소에는 영문 문서와 그 한국어 번역 문서가 짝으로 존재할 수 있다. 둘이 따로 수정되다 보면 한쪽만 바뀌어 내용이 어긋나는 표류(drift)가 생긴다. 이를 막기 위해 GitHub Actions 워크플로가 문서가 바뀔 때마다 자동으로 검사한다.

작동 방식은 이렇다. main 브랜치 push 와 PR 에서 문서 파일(영문 `.md` 또는 한국어 `.ko.md`)이 바뀌면 워크플로가 깨어난다. 워크플로는 검사 로직을 직접 들고 있지 않고, **정본 저장소(wrapper, 즉 clawket 본 저장소)에 게시된 표준 검사 스크립트를 인터넷에서 내려받아** 실행한다 — 검사 규칙의 단일 출처를 한 곳에 두고 여러 위성 저장소가 그것을 공유하는 구조다. 랜딩의 경우 번역은 선택 사항(필수 아님)이라, 영문을 1차로 두고 한국어를 2차 권장으로 검사한다.

## 구현 위치 (provenance)

워크플로 정의는 랜딩 저장소의 CI 설정(`landing/.github/workflows/i18n-sync.yml`)에 있다. 검사 본체 스크립트는 이 저장소가 아니라 clawket 정본 저장소(wrapper)의 표준 검사 스크립트에 있고, 워크플로는 그것을 실행 시점에 내려받아 돌린다 — 검사 로직의 정본은 위성 저장소가 아니라 정본 저장소에 있다.

이 연동은 CI 단계(코드 머지·푸시 시점)에서 동작하며, clawket 제품 런타임 컴포넌트가 호출하는 연동이 아니다. 그래서 사용하는 내부 컴포넌트(integratesWith)는 비워 둔다(OPEN).

런타임에서 제품 화면을 다국어로 보여 주는 로케일 fallback 동작(별도 연동)과는 적용 대상이 다르다 — 이쪽은 어디까지나 문서 번역 짝의 표류 검사다(relatesTo 참조).

## 불변식

- 검사 규칙의 정본은 정본(wrapper) 저장소의 표준 스크립트 한 곳이다 — 위성 저장소(랜딩)는 그것을 내려받아 쓸 뿐, 자체 검사 로직을 따로 두지 않는다.
- 랜딩에서 번역은 필수가 아니라 선택이다 — 영문이 1차이고 한국어가 2차 권장이라는 정책이 검사 메시지에 명시되어 있다.

## 영향 범위

이 연동의 변경(검사 트리거 조건·정본 스크립트 주소)은 랜딩 저장소의 문서 머지 게이트에 파급된다. 그 대상은 랜딩 저장소의 CI 흐름이며, 본 SSOT 인벤토리 등재 노드 중 직접 영향을 받는 대상은 확인되지 않았다(impacts 는 OPEN). 같은 다국어 정책 표면의 런타임 측면은 별도 로케일 연동이 담당한다.

## 미확정 (OPEN)
- [ ] OPEN: CI 워크플로를 호출하는 내부 컴포넌트 노드가 SSOT 인벤토리에 없어 integratesWith 미배선
- [ ] OPEN: 영향 대상(impacts) 확정 필요 — 인벤토리 등재 노드와의 직접 연결 미확인
