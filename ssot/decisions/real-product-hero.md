---
id: decision.real-product-hero
kind: Decision
title: hero 비디오는 마케팅 목업이 아니라 시딩된 실제 대시보드 6뷰 녹화로 만든다
purpose: 랜딩 hero 비디오를 손으로 그린 마케팅 목업으로 만들지, 실제 동작하는 제품 화면을 녹화해 만들지
definition: hero 비디오를 실제 시딩 데이터를 띄운 대시보드의 6개 뷰(Summary→Plans 트리→Backlog→Timeline→Wiki→Board)를 Playwright로 녹화해 만들고, Board의 EVIDENCE_REQUIRED 게이트 순간을 클라이맥스로 배치한다.
relatesTo:
  - to: component.capture-pipeline
    type: realizes
    note: 이 결정이 캡처 파이프라인(시딩 + Playwright 녹화 + 트랜스코드)의 존재 이유다.
  - to: endpoint.asset-hero-video
    type: realizes
    note: 캡처 결과물인 hero 비디오 자산이 이 결정의 산출물이다.
  - to: concept.evidence-required
    type: relates-to
    note: 녹화의 클라이맥스가 EVIDENCE_REQUIRED 게이트 — 제품 핵심 동작을 hero가 직접 보여준다.
  - to: screen.web-board
    type: relates-to
    note: 클라이맥스 프레임이 녹화되는 화면이 Board의 상태변경·근거입력 모달.
  - to: concept.demo-seed-chain
    type: depends-on
    note: 녹화 전 Plan→Unit→Cycle→Task 체인이 시딩 스크립트로 미리 준비되어야 한다.
  - to: capability.watch-demo
    type: relates-to
    note: 방문자가 hero에서 12~14초 데모를 보는 행위가 이 결정의 수혜.
supersedes: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:decision"]
---

> Decision은 append-only다. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 `supersedes`로 잇는다.

## 맥락 (Context)

랜딩 hero는 첫 방문자가 제품의 정체를 판단하는 가장 결정적인 순간이다. 마케팅용으로 손으로 그린 목업 영상은 "실제로 동작하는가"를 의심하게 만들 수 있다. Clawket의 핵심 가치는 "에이전트의 작업 완료를 근거 없이 done으로 선언하지 못하게 막는다"는 점이며, 이 가치는 화면에서 직접 보여야 설득력이 있다.

## 결정 (Decision)

hero 비디오를 실제 제품 화면 녹화로 만든다. 녹화 전에 Plan→Unit→Cycle→Task 체인을 시딩 스크립트로 미리 채워 두고(녹화 자체는 엔티티를 만들지 않고 시딩된 상태를 투어한다), Playwright로 대시보드 6개 뷰를 순서대로 돌며 녹화한다: Summary → 사이드바 Plans 트리 → Backlog → Timeline → Wiki → Board.

마지막 Board에서 클라이맥스를 잡는다: 태스크 상태를 done으로 바꾸려 할 때 근거(evidence) 입력이 비어 있으면 제출 버튼이 비활성화되는 EVIDENCE_REQUIRED 순간이다. 이 비활성화 + 안내 힌트 프레임이 hero·OG 이미지·포스터 정지 프레임이 모두 겨냥하는 단일 윈도우다.

## 근거와 결과 (Consequences)

- 실제 시딩 상태를 투어함으로써 hero가 "마케팅 목업이 아니라 진짜 제품 표면"으로 읽힌다. 첫 방문자가 대시보드의 실제 모양을 본다.
- 클라이맥스를 EVIDENCE_REQUIRED 게이트에 둠으로써, "done은 자기 선언이 아니다 — 제품이 근거를 강제한다"는 제품의 핵심 메시지를 영상이 직접 증명한다.
- 대가로 hero가 실제 제품 화면의 셀렉터(data-testid·역할·텍스트)에 의존한다. 웹 대시보드의 해당 컴포넌트가 바뀌면 캡처가 깨지므로, 다음 녹화 전 셀렉터 계약을 재검증해야 한다. 포스터 정지 프레임은 모달 열림·제출 비활성·근거 필드 빈 상태가 함께 보이는 좁은 시간 구간 안에서 잡혀야 한다.
