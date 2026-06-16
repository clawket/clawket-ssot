---
id: invariant.error-code-stability
kind: Invariant
title: 에러 코드 wire contract 안정성
definition: "데몬이 내부 오류를 외부에 알릴 때 쓰는 코드 문자열과 그에 대응하는 HTTP 상태값은 CLI·웹·MCP가 직접 의존하는 외부 약속이므로, 코드 이름이나 상태값을 단독으로 바꾸거나 제거하면 안 된다."
confidence: inferred
lastVerified: 2026-06-04
lifecycle: active
owner: TBD
governs:
  - concept.evidence-required
  - endpoint.daemon-tasks
  - endpoint.daemon-knowledge
  - domain.governance-audit
implementedIn:
  - daemon/src/routes/error.rs
  - daemon/.claude/rules/error-code-stability.md
decidedBy: []
crossesBoundary: true
tags:
  - status:active
  - type:invariant
relatesTo:
  - to: invariant.evidence-required
    type: relates-to
    note: 대표적인 보호 대상 코드가 evidence 누락 거부 코드다.
---

## 제약

데몬은 내부에서 발생한 오류를 외부로 내보낼 때, 오류 메시지 앞에 대문자 코드 단어를 붙여 "이건 어떤 종류의 오류인지"를 기계가 알아볼 수 있게 한다. 데몬을 사용하는 세 주체 — 명령줄 도구(CLI), 웹 대시보드, 그리고 LLM이 쓰는 MCP — 는 이 코드 단어와 그에 딸린 HTTP 응답 번호(예: 잘못된 요청 / 못 찾음 / 충돌)를 직접 보고 동작을 분기한다. 따라서 이 코드 단어와 응답 번호의 짝은 외부와 맺은 계약이며, 한쪽만 임의로 바꾸면 계약 위반이 된다.

지켜야 할 규칙은 세 가지다. 첫째, 기존 코드 단어의 철자를 바꾸거나 없애지 않는다. 둘째, 코드에 붙은 응답 번호를 슬그머니 다른 번호로 바꾸지 않는다. 셋째, 새 오류를 추가할 때는 "잘못된 입력 같으면 대충 잘못된 요청으로 처리" 같은 두루뭉술한 추정 처리에 기대지 않고, 그 오류 전용 분기를 명시적으로 더한다.

## 깨지면 무슨 일이 일어나나

코드 단어를 리네임하거나 삭제하면, 외부에서 그 단어를 보고 분기하던 로직이 더 이상 맞아떨어지지 않아 오류가 일반 서버 오류(500)로 떨어진다. 그러면 명령줄 도구의 재시도·분기 로직이 깨지고, 웹의 안내 메시지가 사라지며, 특히 evidence 누락 거부 같은 사용자 데이터 무결성 가드가 약해져 본래 막아야 할 작업이 통과한다. 응답 번호만 살짝 바꾸는 경우도 마찬가지로, 겉보기엔 멀쩡해 보이지만 그 번호에 의존하던 재시도 흐름이 조용히 어긋난다.

## 코드에서 어떻게 강제되나

데몬의 오류 변환부(daemon/src/routes/error.rs)가 내부 오류 메시지의 앞부분 코드 단어를 차례로 대조해 해당하는 응답 번호와 구조화된 코드 필드로 바꾼다. 어디에도 걸리지 않으면 마지막에 일반 처리로 떨어진다. 이 규칙 자체는 데몬 하위의 규칙 문서(daemon/.claude/rules/error-code-stability.md)에 명문화되어 있고, 오류를 새로 추가·변경할 때 같은 변경 안에서 이 변환부의 분기도 함께 손보라고 요구한다.

다만 강제는 완전하지 않다. 새 코드 단어가 추가됐는데 변환부 분기를 빠뜨린 것을 컴파일 시점에 잡아 주는 장치가 없고, 코드 단어와 응답 번호의 짝을 검증하는 통합 테스트도 부분적이라, 일반 처리로 새어 나간 경우는 조용히 통과할 수 있다. 즉 이 불변식은 현재 코드·규칙 문서와 사람의 주의에 의존하며, 자동 차단까지는 도달하지 못한 상태다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(근거 결정) 연결 필요 — 이 wire contract 안정성을 처음 명문화한 결정 노드가 인벤토리에 없어 미확정.
