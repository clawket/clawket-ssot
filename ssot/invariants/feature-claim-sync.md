---
id: invariant.feature-claim-sync
kind: Invariant
title: 문서의 버전·스크린샷·기능 주장은 wrapper 정본과 동기화한다
definition: "랜딩 사이트의 문서 페이지가 말하는 버전 숫자, 스크린샷, 제품 기능 주장은 wrapper(clawket 본체) 레포의 정본 — README, 컴포넌트 핀 버전, 호환성 매트릭스 — 와 어긋나서는 안 된다. 문서는 사실의 출처가 아니라 그 정본을 비추는 거울이다."
governs:
  - screen.docs-getting-started
  - screen.docs-workflow
  - screen.docs-cli
  - screen.docs-architecture
  - screen.docs-hub
implementedIn:
  - landing/src/pages/Docs.tsx
  - landing/src/pages/docs
decidedBy: []
crossesBoundary: true
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:invariant
---

## 제약

랜딩 사이트의 문서 영역(허브와 4개 본문 페이지)이 진술하는 세 종류의 사실 — 버전 숫자, 화면 스크린샷, 제품이 무엇을 할 수 있다는 기능 주장 — 은 모두 wrapper 레포(클로켓 본체)의 정본을 따라야 한다. 랜딩은 이 사실들의 원천이 아니라 정본을 옮겨 적는 쪽이다. 따라서 문서가 말하는 내용과 wrapper 정본이 어긋나면, 문서가 틀린 것으로 간주하고 정본 쪽으로 맞춘다.

정본은 세 곳에 나뉘어 있다. 기능 설명과 사실 주장(대시보드 뷰 구성, 감사 로그 동작 등)의 출처는 wrapper의 README이고, CLI·데몬·웹 컴포넌트의 핀 버전 숫자는 wrapper의 컴포넌트 핀 정의에, 컴포넌트 간 버전 범위는 wrapper의 호환성 매트릭스 문서에 있다. 홈 화면 상단의 버전 라벨도 같은 정신으로 wrapper 플러그인 태그의 메이저·마이너를 미러한다.

## 깨지면 무슨 일이 일어나나

문서가 정본보다 앞서거나 뒤처지면 방문자에게 거짓 약속을 하게 된다. 실제 제품이 하지 않는 기능을 문서가 주장하면 신규 사용자는 설치 후 기대와 다른 동작을 만나고, 버전 숫자가 어긋나면 설치 가이드의 명령·산출물과 실제 릴리스가 맞지 않아 온보딩이 막힌다. 스크린샷이 오래되면 화면이 실제와 달라 혼란을 준다. 더 근본적으로, 랜딩이 독자적으로 기능 문구를 만들기 시작하면 "사실의 출처는 wrapper 하나"라는 단일 진실원천 원칙이 무너지고, 같은 제품을 두 곳에서 서로 다르게 설명하는 분기가 생긴다.

## 코드에서 어떻게 강제되나

자동 동기화 장치가 코드에 박혀 있는 것이 아니라, 작성 시 따라야 할 동기화 규칙으로 운영된다. 문서 본문 페이지(`landing/src/pages/docs/` 아래 4개)는 영문 단문으로 화면에 직접 작성되어 있고, 그 기능 주장은 wrapper의 README 정본과 맞추는 것이 합의되어 있다. 랜딩 레포의 `CLAUDE.md`는 "제품 기능 문구를 임의로 만들지 않으며 wrapper README와 어긋나면 사람에게 확인한다", "버전 숫자·스크린샷·기능 주장은 wrapper의 컴포넌트 핀·호환성 매트릭스·README와 동기화한다"고 명문화해 이 거울 관계를 유지한다. 버전 라벨만큼은 릴리스 때 별도 갱신 스크립트로 모든 로케일을 일괄 치환해, 손으로 개별 수정하다 어긋나는 일을 막는다. 이 제약은 랜딩 레포와 wrapper 레포라는 두 시스템 경계를 가로지른다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요 — 동기화 정책을 정한 의사결정 노드가 SSOT에 아직 없음
- [ ] OPEN: 문서 기능 주장과 wrapper README의 drift를 자동 검출하는 CI 게이트는 확인되지 않음 (현재는 작성 관례 + 사람 확인으로 유지)
