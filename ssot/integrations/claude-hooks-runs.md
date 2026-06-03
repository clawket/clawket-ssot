---
id: integration.claude-hooks-runs
kind: Integration
title: 실행 기록의 변경 파일 — Claude 훅이 채우고 대시보드가 읽는다
purpose: 에이전트가 한 실행에서 실제로 어떤 파일을 바꿨는지를, 별도 저장 칸 없이 실행 기록의 메모(notes)에 실어 보존하고, 대시보드가 그것을 풀어 두 실행을 나란히 비교(diff)할 수 있게 한다.
definition: 실행(run)의 notes 필드는 Claude Code 의 파일 변경 후 훅(PostToolUse)이 바뀐 파일 목록을 구조화된 JSON 으로 채워 넣은 결과다. 웹 대시보드의 실행 비교 화면(RunCompare)은 이 notes 를 방어적으로 파싱해 변경 파일을 꺼내고, 두 실행을 골라 한쪽만/양쪽 공통 파일로 나눠 보여준다. JSON 이 아니거나 손으로 편집된 옛 기록은 "변경 파일 없음"으로 부드럽게 떨어진다.
integratesWith:
  - component.web-dashboard
  - component.claude-adapter
implementedIn:
  - web/src/features/runs/RunCompare.tsx
impacts:
  - concept.run
relatesTo:
  - to: endpoint.hook-post-tool-use
    type: feeds
    note: notes 의 변경 파일 목록을 채워 넣는 주체. 대시보드는 그 산출물을 소비한다.
  - to: integration.claude-code-hooks
    type: relates-to
    note: 이 데이터가 흐르는 더 넓은 Claude 훅 연동의 한 갈래.
  - to: integration.daemon-sse
    type: relates-to
    note: 실행 비교 화면은 실행 SSE 이벤트로 목록을 폴링 없이 갱신한다.
governedBy:
  - invariant.response-shape-backcompat
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:integration
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로 — 무엇을·왜·누가·어떤 규칙·무슨 데이터. 코드 옮겨적기 금지(식별자는 provenance/근거에만). -->

## 무엇과 연동하나

이 연동은 두 주체가 같은 데이터 칸을 사이에 두고 만나는 지점이다. 한쪽은 **Claude Code 의 파일 변경 후 훅**(에이전트가 파일을 편집할 때마다 그 사실을 잡아내는 훅)이고, 다른 쪽은 **웹 대시보드의 실행 비교 화면**이다. 둘을 잇는 데이터 칸은 실행(run) 기록에 달린 자유 메모 필드다.

핵심 사실은 이렇다. 실행 기록에는 "바뀐 파일 목록"을 위한 전용 저장 칸이 따로 없다. 대신 Claude 훅이 한 실행 동안 에이전트가 편집한 파일들을 모아 **구조화된 JSON 형태로 그 메모 칸에 적어 둔다.** 대시보드는 나중에 이 메모를 풀어, 두 실행을 나란히 비교할 때 어느 파일이 한쪽에서만 바뀌었고 어느 파일이 양쪽 공통으로 바뀌었는지 보여준다.

메모 칸은 본래 자유 텍스트라, 손으로 편집됐거나 옛 형식이라 JSON 이 아닌 경우도 있다. 그럴 때 비교 화면은 오류를 내지 않고 "보고된 변경 파일 없음"으로 조용히 떨어진다 — 데이터가 깔끔하지 않아도 화면이 깨지지 않게 하는 방어적 태도다.

## 구현 위치 (provenance)

- `web/src/features/runs/RunCompare.tsx` — 실행 비교 화면. 한 태스크의 모든 실행을 나열하고 정확히 둘을 고르게 한 뒤, 각 실행의 메모를 방어적으로 파싱해 변경 파일 목록을 꺼낸다. 파싱 결과를 한쪽만/양쪽 공통으로 나눠 보여주며, 에이전트 식별자·소요 시간·결과 문자열, 그리고 실행 시작 시점에 유효했던 실행 봉투의 목표 모델도 함께 표시한다. 파일 변경 목록을 채우는 일은 이 화면이 하지 않는다 — 그것은 Claude 훅이 미리 메모에 적어 둔 것을 읽기만 한다.

이 연동을 사용하는 내부 컴포넌트는 웹 대시보드(`component.web-dashboard`)이고, 데이터를 만들어 넣는 쪽은 Claude 어댑터(`component.claude-adapter`)의 파일 변경 후 훅(`endpoint.hook-post-tool-use`)이다.

## 불변식

- 메모(notes)에 적힌 변경 파일 목록의 형태가 약속된 JSON 구조와 호환되어야 대시보드가 풀어 읽을 수 있다 — `invariant.response-shape-backcompat`. 형식이 맞지 않거나 옛 기록이면 "변경 파일 없음"으로 안전하게 폴백한다.

## 영향 범위

이 연동은 실행 기록(`concept.run`)의 의미를 직접 좌우한다. 훅이 채우는 메모 형식이 바뀌면, 같은 데이터를 읽는 실행 비교 화면도 함께 맞춰야 한다 — 즉 "누가 무엇을 적는지"와 "누가 그것을 읽는지"가 한 형식 계약으로 묶여 있다.

## 미확정 (OPEN)
- [ ] OPEN: 변경 파일을 위한 실행 기록의 전용 저장 칸(컬럼)을 신설할지, 메모 기반을 유지할지 확정 필요
- [ ] OPEN: 훅이 적는 JSON 구조의 정식 스키마 정의 위치 확정 필요
