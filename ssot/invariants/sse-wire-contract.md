---
id: invariant.sse-wire-contract
kind: Invariant
title: SSE 이벤트 wire contract 동기화
definition: "데몬이 실시간으로 내보내는 이벤트 이름은 구독하는 쪽이 직접 의존하는 약속이므로, 이벤트를 발행하는 자리와 그 이름을 해석하는 정적 매핑이 항상 같은 변경 안에서 함께 갱신되어야 한다."
confidence: inferred
lastVerified: 2026-06-04
lifecycle: active
owner: TBD
governs:
  - endpoint.events-sse
  - integration.daemon-sse
  - concept.knowledge
implementedIn:
  - daemon/src/state.rs
  - daemon/.claude/rules/sse-event-wire-contract.md
decidedBy: []
crossesBoundary: true
tags:
  - status:active
  - type:invariant
relatesTo:
  - to: invariant.sse-payload-id-only
    type: relates-to
    note: 같은 SSE 스트림의 페이로드 형태를 제약하는 자매 불변식.
---

## 제약

데몬은 작업·사이클·실행 등의 변화가 생기면 "어떤 것에 무슨 일이 일어났다"를 실시간 이벤트로 흘려보낸다(예: 작업이 만들어짐, 지식이 삭제됨). 웹 대시보드·명령줄 감시 도구·MCP가 이 이벤트 이름을 직접 보고 화면을 갱신하거나 후속 처리를 한다. 그래서 이벤트 이름은 구독자와 맺은 약속이다.

데몬 안에서 이 약속은 두 곳에 나뉘어 있다. 하나는 실제로 이벤트를 내보내는 자리(라우트), 다른 하나는 그 이름을 "어떤 대상의 어떤 변화"로 풀어 주는 정적 해석표(daemon/src/state.rs)다. 제약은 이 둘이 항상 짝을 맞춰야 한다는 것이다 — 한쪽에 이벤트를 추가·변경·삭제하면 같은 변경 안에서 다른 쪽도 함께 손봐야 한다. 또한 이벤트 이름은 "대상:변화" 형식을 유지하고, 페이로드에 들어가는 정보는 추가만 허용하며 기존 항목의 제거·이름 변경은 금지한다.

## 깨지면 무슨 일이 일어나나

라우트에 새 이벤트를 추가했는데 해석표 갱신을 잊으면, 그 이벤트는 "알 수 없음"으로 흘러가 대상·변화 라벨이 사라진다. 구독자는 무슨 일이 일어났는지 알 수 없어 조용히 무시한다 — 에러가 나는 게 아니라 아무 일도 안 일어난 것처럼 새어 나간다. 반대로 이벤트 이름을 슬그머니 바꾸면, 옛 이름을 듣고 있던 구독자가 다음 릴리스 사이에 갑자기 끊긴다. 해석표만 지우고 발행 자리는 남겨 두는 역방향 실수도 같은 결과를 낳는다. 이런 어긋남은 겉으로 드러나지 않고 조용히 누락되는 것이 가장 위험하다.

## 코드에서 어떻게 강제되나

이름 해석 책임은 데몬의 상태 모듈(daemon/src/state.rs)에 있는 정적 해석표가 진다. 알려진 이름은 대상·변화 한 쌍으로 풀어 주고, 표에 없는 이름은 마지막에 "알 수 없음"으로 떨어진다. 규칙 자체는 데몬 하위 규칙 문서(daemon/.claude/rules/sse-event-wire-contract.md)에 명문화되어 있으며, 이벤트 관련 라우트를 바꾸면 같은 변경 안에서 해석표도 함께 갱신하라고 요구한다.

강제는 완전하지 않다. 발행 자리에 새 이벤트가 추가됐을 때 해석표 누락을 컴파일 시점에 잡아 주는 장치가 없고, 새 이벤트와 해석의 짝을 검증하는 테스트도 강제되지 않는다. 이름이 그대로 흘러가 "알 수 없음" 처리까지 통과하므로, 이 불변식은 현재 사람의 주의와 규칙 문서에 의존한다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요 — 이중 정의 패턴과 동기화 규칙을 정한 결정 노드가 인벤토리에 없어 미확정.
