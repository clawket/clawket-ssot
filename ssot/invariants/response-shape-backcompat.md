---
id: invariant.response-shape-backcompat
kind: Invariant
title: 응답 JSON 모양은 비파괴 진화만 허용
definition: 데몬이 내보내는 HTTP/SSE 응답 JSON 은 필드 추가만 허용하고, 기존 필드의 제거·이름 변경·타입 변경은 금지한다 — CLI·웹·MCP 클라이언트가 동시에 갱신될 수 없기 때문이다.
governs:
  - component.daemon
  - component.cli
  - component.web-dashboard
  - component.mcp-server
  - endpoint.daemon-events
  - endpoint.daemon-search
decidedBy:
implementedIn:
  - daemon/.claude/rules/response-shape-backwards-compat.md
  - daemon/src/models.rs
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

데몬은 여러 종류의 클라이언트(CLI 바이너리, 웹 대시보드, MCP 도구)에게 같은 응답 JSON 을 내보낸다. 이 클라이언트들은 데몬과 **동시에 업데이트되지 않는다** — 사용자가 데몬만 새 버전으로 올리고 CLI·웹은 옛 버전을 그대로 쓰는 상황이 정상적으로 발생한다. 그래서 응답 JSON 의 "모양(필드 구성)"은 한 방향으로만 진화할 수 있다.

- **허용**: 새 필드를 **추가**하는 것. 단, 옛 클라이언트가 모르는 필드를 무시할 수 있도록, 새 필드는 값이 없을 때 아예 직렬화에서 빠지는 형태(선택적 필드)로 넣는다.
- **금지**: 기존 필드를 **제거**하거나, **이름을 바꾸거나**, **타입을 바꾸는** 것. 여기에는 wire 상의 키 이름(클라이언트가 실제로 읽는 키)을 다른 값으로 덮어쓰는 것, 선택적이던 필드를 필수로 좁히는 것, 문자열을 숫자로 바꾸는 식의 조용한 타입 변환이 포함된다.

의미가 바뀐 필드는 옛 필드를 지우는 대신 **새 이름의 필드를 추가**하고, 옛 필드는 한 릴리스 이상 deprecate 상태로 유지한다.

## 깨지면 무슨 일이 일어나나

옛 클라이언트가 기대하던 필드가 사라지거나 타입이 바뀌면, 그 클라이언트는 응답을 해석하지 못한다 — deserialize 실패나 파싱 무력화, 심하면 패닉으로 이어진다. 사용자 입장에서는 "데몬만 업데이트했는데 CLI 가 갑자기 깨졌다"는, 원인을 찾기 어려운 고장이 된다. 이 제약은 데몬(BE)과 CLI·웹·MCP(여러 FE/소비자) 사이의 경계를 가로지르므로, 한쪽만 보고 모양을 바꾸면 반드시 다른 쪽이 깨진다.

## 코드에서 어떻게 강제되나

이 제약은 **자동으로 강제되지 않는다** — 직렬화 구조체를 바꿔도 컴파일은 통과하고, wire 모양 변경이 조용히 빠져나간다. 룰 문서에 따르면 응답 스키마의 안정성을 보장하는 golden/snapshot 테스트가 없고, 직렬화 설정 변경을 자동으로 막는 수단이 없으며, 클라이언트 측 contract drift 를 감지할 OpenAPI/JSON schema 산출물도 없다. 따라서 현재로서는 코드 리뷰와 작성자의 규율이 유일한 방어선이다.

데몬의 모델 정의에는 이 비파괴 패턴이 적용된 사례가 있다 — 선택적 필드는 값이 없으면 직렬화에서 빠지도록 표시되고, wire 키 이름은 Rust 식별자와 별개로 명시적으로 고정된다(예: 내부적으로 `type_` 이라 부르는 필드를 클라이언트에게는 `type` 키로 노출). 이 키 이름이 곧 클라이언트와의 계약이라, 바꾸면 wire 가 깨진다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요 — 비파괴 진화 정책 채택 결정 기록 미확인
- [ ] OPEN: 응답 스키마 golden test / contract 산출물 도입 여부(현재는 부재로 확인됨) 추적 필요
