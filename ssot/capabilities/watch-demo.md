---
id: capability.watch-demo
kind: Capability
title: 데모 영상 보기
definition: 랜딩 홈 화면 중앙에 실제 제품 대시보드의 작동 모습을 담은 짧은 데모 영상이 자동으로 재생되어, 방문자가 설치 없이도 Clawket이 근거 없는 작업 완료를 거부하는 흐름을 눈으로 확인할 수 있다.
purpose: 글 설명만으로는 와닿지 않는 "근거 없이는 완료 안 됨"이라는 핵심 가치를 실제 화면으로 즉시 보여줘 방문자를 설득한다.
servesPersona:
  - persona.human-operator
realizedBy:
  - component.landing-site
implementedIn:
  - landing/src/pages/Home.tsx
  - landing/public/hero.webm
  - landing/public/hero.mp4
  - landing/public/hero-poster.png
relatesTo:
  - to: endpoint.asset-hero-video
    type: relates-to
    note: 홈이 재생하는 영상 자산이 이 정적 엔드포인트로 제공된다
  - to: decision.real-product-hero
    type: relates-to
    note: 합성/목업이 아닌 실제 제품 화면을 데모로 쓰기로 한 결정에 따른 기능이다
impacts:
  - endpoint.asset-hero-video
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:capability
---

## 사용자가 할 수 있는 일

랜딩 홈에 들어온 방문자는 데모 영역에서 제품의 실제 동작 영상을 별도 클릭 없이 바로 볼 수 있다. 영상은 자동으로 재생되고, 소리는 꺼져 있으며, 끝나면 다시 처음부터 반복된다. 이 영상은 합성 화면이 아니라 실제 대시보드에서 작업이 진행되고, 근거 없이 완료를 시도하면 거부되는 흐름을 그대로 담고 있어, 방문자는 설치 전에도 제품이 무엇을 강제하는지 직접 확인할 수 있다.

## 행위

- 읽기/재생: 홈 진입 시 데모 영상을 자동 재생한다(음소거, 반복).
- 형식 선택: 브라우저가 지원하는 형식을 우선 사용한다 — webm을 먼저 시도하고, 지원하지 않으면 mp4로 대체한다.
- 사전 표시: 영상이 로드되기 전에는 정지 화면(포스터 이미지)을 보여줘 빈 영역을 막는다.
- 접근성: 화면 낭독기 사용자를 위해 영상의 의미를 설명하는 대체 텍스트를 제공한다.

이 기능은 정적 영상 자산만 사용하며 데몬 API나 CLI를 호출하지 않는다.

## 시스템 흐름

방문자가 홈을 열면 데모 영역의 영상 요소가 자동 재생 상태로 그려진다. 브라우저는 webm 소스를 먼저 시도하고, 재생 불가하면 mp4 소스로 넘어간다. 영상 데이터가 준비되기 전에는 포스터 이미지가 표시되고, 준비되면 영상이 반복 재생된다. 영상 파일은 사이트의 정적 자산 경로(`endpoint.asset-hero-video`)에서 내려받는다.

## 어디에 구현되어 있나

- `landing/src/pages/Home.tsx` — 데모 영역과 자동 재생·음소거·반복·소스 우선순위(webm→mp4)·포스터·대체 텍스트 설정.
- `landing/public/hero.webm`, `landing/public/hero.mp4` — 데모 영상 자산(형식별).
- `landing/public/hero-poster.png` — 로드 전 표시되는 정지 이미지.

이 기능은 랜딩 사이트(`component.landing-site`)에서 실현된다.

## 미확정 (OPEN)
- [ ] OPEN: 영상 길이(12~14초)는 인벤토리 설명에 기재된 값으로, 소스 코드에는 길이 지정이 없어 코드만으로 검증되지 않음(자산 자체의 메타데이터 확인 필요).
