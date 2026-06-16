---
id: decision.mcp-enforce-deferred
kind: Decision
title: 훅 강제는 단일 MCP 도구가 아니라 어댑터 강제 계층(cjs fat-handler)으로 유지한다
purpose: "Claude Code 훅의 강제(가드레일) 로직을 하나의 MCP 도구로 옮길지, 지금의 어댑터 강제 계층에 그대로 둘지"
definition: "단일 MCP 도구(clawket.enforce)로의 이전은 평가 후 보류하고, 어댑터의 cjs 강제 계층을 의도된 단일 진실 강제 지점으로 유지한다."
relatesTo:
  - to: domain.hook-enforcement
    type: governs
    note: 이 결정이 훅 강제 영역의 구현 위치(어댑터 cjs)를 못박는다
  - to: component.claude-adapter
    type: relates-to
    note: 강제 로직이 머무는 어댑터 계층(cjs fat-handler)
  - to: integration.mcp
    type: relates-to
    note: 검토 대상이었던 MCP 경로 — 현재는 read-only 지식 도구만 제공
  - to: endpoint.hook-pre-tool-use
    type: relates-to
    note: 가장 큰 영향 범위를 가진 강제 지점으로, 이전 시 마지막에 옮기기로 한 핸들러
supersedes: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-03
tags: [status:active, "type:decision"]
---

<!-- Decision은 append-only. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 supersedes로 잇는다. -->

> Decision은 append-only다. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 `supersedes`로 잇는다.

## 맥락 (Context)

Clawket의 가드레일(활성 태스크 없으면 변경 차단, 파괴적 명령 차단, PDD 규칙 검사, 모델 티어 게이트, 플랜 형상 검증 등)은 Claude Code가 발생시키는 훅 이벤트마다 동작한다. 이 강제 판정이 어디에 살아야 하는가에 대해, "강제 로직 전부를 하나의 MCP 도구(가칭 clawket.enforce)로 모아 상주 프로세스로 돌리자"는 대안 설계가 검토되었다. 이 대안은 두 가지 문제가 있다는 전제에서 출발했다 — (1) 훅 이벤트마다 새 프로세스가 차갑게 뜨는 비용, (2) 강제 로직이 어댑터 계층과 데몬에 중복된다는 가정.

검토 결과 두 전제 모두 기술된 형태로는 성립하지 않았다. (1) 콜드 스타트 비용은 실재하나 지배적 비용은 프로세스 기동이 아니라 핸들러 내부에서 CLI를 서너 번 외부 호출하는 부분이었고, 이는 MCP로 옮기지 않아도 어댑터에서 데몬을 HTTP로 직접 부르는 훨씬 작은 변경으로 해소된다. (2) 중복은 사실이 아니었다 — 각 게이트는 정확히 한 곳에만 산다. 어댑터 계층은 "이 도구 호출을 진행해도 되는가"를 읽기 전용 조회로 판정하고, 데몬은 "이 상태 변경을 저장해도 되는가"를 제약으로 막는다. 둘은 중복이 아니라 상보적이다.

## 결정 (Decision)

단일 MCP 강제 도구로의 이전은 **보류(deferred)** 한다. 어댑터의 강제 계층(cjs fat-handler)을 훅 강제의 **의도된 단일 진실 지점**으로 유지한다. 이 계층은 설치 게이트·스킬 무결성 테스트·훅 회귀 테스트에 의해 고정(pinned)되어 있어, 임의로 흔들리지 않도록 보호된다.

향후 누군가 이 대안을 다시 검토할 경우를 위해, MCP 강제 도구의 입력/반환 형상, 점진 이전 경로, 위험 목록은 별도 설계 메모로 남겨두되 — 그 어느 것도 로드맵 약속이 아니다.

## 근거와 결과 (Consequences)

- **단일 진실의 위치 보존**: 강제 게이트가 어댑터 한 곳에 있고 데몬이 이를 중복하지 않으므로, "어느 계층이 판정의 주인인가"가 명확하다. MCP로 옮기면 훅 의미가 데몬 릴리스 주기에 묶여 이 경계가 흐려진다.
- **롤백 안전성**: 어댑터 cjs 계층은 플러그인 설치 게이트가 매번 재구축하는 영역(삭제·재생성 가능)이라, 회귀가 나도 되돌리기 쉽다. 강제 로직을 사용자 데이터 도메인(데몬, 스키마 버전 관리)으로 옮기면 되돌리기가 더 어려워진다.
- **빅뱅 이전 거부**: 훅 계층은 모든 세션의 임계 경로에 있어 회귀 한 번이 사용자를 망가뜨린다. 만약 이전한다면 감사 전용이라 도구 호출을 막지 못하는 PostToolUse부터 파일럿하고, 영향 범위가 가장 큰 PreToolUse를 마지막에 옮기는 점진 방식만 허용한다.
- **데몬 부재 시 동작 차이**: MCP 경로는 데몬이 닿지 않으면 fail-closed(차단)로 동작하게 되는데, 이는 지금의 어댑터 계층이 데몬 장애 시 경고만 남기고 통과(fail-open)시키는 것과 다른 동작 변경이다.

[근거 provenance] clawket/docs/HOOK_ENFORCEMENT.md (Status: Considered, Not Adopted), adapters/shared/claude-hooks.cjs
