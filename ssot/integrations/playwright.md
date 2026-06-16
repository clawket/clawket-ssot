---
id: integration.playwright
kind: Integration
title: Playwright 브라우저 자동화(랜딩 자산 생성)
purpose: "랜딩 페이지에 쓰이는 시각 자산(히어로 데모 영상·소셜 카드 이미지)을 사람이 화면을 직접 녹화·캡처하지 않고 재현 가능하게 생성하고, 발행 전에 페이지가 실제로 떠는지 자동으로 점검한다."
definition: "헤드리스 브라우저(Chromium)를 코드로 조종하는 Playwright 자동화. 실제 제품 화면을 띄워 영상으로 녹화하고, 템플릿 페이지를 정해진 크기로 스크린샷하며, 빌드된 랜딩을 열어 핵심 요소가 렌더되는지 검사한다."
integratesWith: []
implementedIn:
  - landing/scripts/capture/capture.mjs
  - landing/scripts/og/render-og.mjs
  - landing/scripts/og/landing-smoke.mjs
impacts: []
relatesTo:
  - { to: integration.clawket-cli-seed, type: depends-on, note: "히어로 영상 녹화는 CLI 시딩이 미리 만들어 둔 프로젝트→플랜→유닛→사이클→태스크 데모 상태 위에서 실제 웹 대시보드를 띄워 진행한다 — 시딩 없이는 녹화가 실패한다." }
  - { to: concept.evidence, type: relates-to, note: "히어로 영상의 클라이맥스는 태스크를 done 으로 바꾸려 할 때 evidence 없이는 제출이 막히는 장면(EVIDENCE_REQUIRED)을 의도적으로 담는다 — clawket 의 핵심 규칙을 시연한다." }
governedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:integration"]
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드/식별자는 provenance·근거에만. -->

## 무엇과 연동하나

랜딩 페이지에 필요한 시각 자산을 만들고 점검하기 위해 **Playwright** 라는 브라우저 자동화 도구를 쓴다. Playwright 는 사람이 클릭·이동하는 대신, 코드로 실제 브라우저(Chromium)를 띄워 화면을 조종한다. 여기서는 세 가지 용도로 쓰인다.

- **히어로 데모 영상 녹화**: 실제 clawket 웹 대시보드를 브라우저로 띄우고, 화면을 화면 녹화하면서 여섯 개 화면(요약·플랜 트리 사이드바·백로그·타임라인·위키·보드)을 순서대로 둘러본다. 마지막 보드 화면에서는 태스크를 완료로 바꾸려 시도해, 근거(evidence) 없이는 완료 제출이 막히는 장면을 클라이맥스로 보여 준 뒤 근거를 채워 완료시킨다. 약 14초 길이로 호흡이 맞춰져 있고, 결과 원본 영상은 이후 별도 변환을 거쳐 랜딩의 히어로 영상이 된다.
- **소셜 카드 이미지 렌더**: 소셜 공유용 카드(OG 이미지) 템플릿 HTML 을 띄워 정해진 크기(1200×630)로 스크린샷해 이미지 파일로 저장한다.
- **랜딩 스모크 점검**: 빌드해 미리보기로 띄운 랜딩 페이지를 열어, 히어로 영상 요소가 실제로 존재하고 재생 소스가 붙어 있는지 확인하고, 콘솔 오류를 감시하며 점검 스크린샷을 남긴다.

## 구현 위치 (provenance)

세 용도는 각각 별도의 자동화 스크립트로 구현되어 있다 — 히어로 녹화 드라이버(`landing/scripts/capture/capture.mjs`), 소셜 카드 렌더러(`landing/scripts/og/render-og.mjs`), 랜딩 스모크 점검(`landing/scripts/og/landing-smoke.mjs`). 셋 모두 헤드리스 Chromium 을 띄워 동작한다.

이들은 랜딩 페이지 산출물을 만드는 빌드-타임/제작-타임 도구이며, clawket 본 제품(CLI·데몬·웹 대시보드) 내부 컴포넌트가 사용하는 런타임 연동이 아니다. 그래서 사용하는 내부 컴포넌트 노드(integratesWith)는 본 인벤토리에 대응 항목이 없어 비워 둔다(OPEN). 다만 히어로 녹화는 실제 제품 웹 대시보드를 띄워 진행하므로, 녹화가 비추는 대상은 실제 clawket 화면이라는 점이 중요하다.

## 불변식

- 히어로 영상 녹화는 미리 시딩된 데모 상태(활성 플랜·활성 사이클·진행 중 태스크)가 존재해야만 성공한다 — 화면 앵커가 뜨지 않으면 빠르게 실패하도록 되어 있어, 라우트가 바뀌거나 화면이 깨지면 영상이 잘못 만들어지지 않는다.
- 녹화 클라이맥스는 "근거 없이는 완료 불가" 라는 제품 규칙을 사실대로 시연해야 한다 — 즉 자산이 제품의 실제 동작과 어긋나지 않아야 한다.

## 영향 범위

이 연동의 산출물은 랜딩 페이지의 히어로 영상·소셜 카드 이미지이며, 변경은 랜딩의 시각 표면에 파급된다. 그 표면은 랜딩 산출물 내부이고, 본 SSOT 인벤토리 등재 노드와의 직접 연결은 확인되지 않았다(impacts 는 OPEN). 다만 녹화 시나리오가 의존하는 데모 상태는 CLI 시딩 연동이 책임지고, 시연 대상 규칙은 evidence 개념과 닿아 있다(relatesTo 참조).

## 미확정 (OPEN)
- [ ] OPEN: 랜딩 자산 제작 스크립트에 대응하는 내부 컴포넌트 노드가 SSOT 인벤토리에 없어 integratesWith 미배선
- [ ] OPEN: 영향 대상(impacts) 확정 필요 — 현재 인벤토리 등재 노드와의 직접 연결 미확인
