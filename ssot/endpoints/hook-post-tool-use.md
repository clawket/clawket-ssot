---
id: endpoint.hook-post-tool-use
kind: Endpoint
title: 훅 PostToolUse(Edit|Write)
definition: "파일 편집·쓰기가 끝난 직후 호출되는 훅으로, 변경된 파일 경로를 활성 태스크(또는 실행 기록)에 자취로 기록하고, 진행 중 태스크에 시나리오가 비어 있으면 경고한다."
realizedBy:
  - component.claude-adapter
implementedIn:
  - clawket/hooks/hooks.json
  - clawket/adapters/claude/post-tool-use.cjs
  - clawket/adapters/shared/claude-hooks.cjs
relatesTo:
  - to: concept.task
    type: mutates
    note: 변경된 파일 경로를 활성 태스크 본문에 자취로 덧붙인다
  - to: concept.run
    type: reads
    note: 현재 세션의 열린 실행 기록을 찾아 그 태스크에 변경을 귀속시킨다
  - to: concept.scenario
    type: evaluates
    note: 변경이 일어난 진행 중 태스크에 시나리오가 비었으면 경고한다(X3)
  - to: endpoint.daemon-activity
    type: relates-to
    note: 열린 실행 기록이 없을 때 변경 자취를 데몬 활동 기록으로 보낸다
governedBy:
  - invariant.scenario-id-one-to-one
impacts: []
consumedBy:
  - integration.claude-code-hooks
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

파일 편집(Edit)이나 쓰기(Write)가 성공적으로 끝난 직후에 발화되는 사후 기록 훅이다. 무엇을 막거나 바꾸는 훅이 아니라, "방금 이 파일을 건드렸다"는 사실을 지금 진행 중인 작업에 자취로 남겨, 나중에 누가·언제·무엇을 바꿨는지 추적할 수 있게 하는 감사·연결 용도다.

부수적으로 검증 워크플로우의 위생 점검도 한다 — 변경이 일어난 진행 중 태스크에 시나리오가 연결돼 있지 않으면 경고를 띄운다. 이 시점은 태스크가 진행 중이고 실제 변경이 막 일어난 것이 확정된 순간이라, 시나리오 누락을 짚기에 적절한 위치다.

## 요청 / 응답

입력은 방금 실행된 도구 이름과 변경 대상 파일 경로, 그리고 현재 세션 식별자다. 응답으로 무언가를 차단하지는 않는다(사후 훅이므로 허용/거부 판정이 없다).

동작은 이렇다. 이번 세션에 열려 있는 실행 기록이 있으면 그 실행이 매인 태스크 본문에 "[Edit] 경로" 또는 "[Write] 경로" 형태의 한 줄을 덧붙인다. 열린 실행 기록이 없으면 변경 사실을 데몬의 활동 기록으로 보내 감사 추적에 남긴다. 두 경우 모두, 관련된 진행 중 태스크에 시나리오가 비어 있으면 경고(비차단)를 함께 남긴다.

## 권한 / 제약

편집·쓰기 도구에 한해 자동 실행되며, 그 외 도구에서는 동작하지 않는다(matcher가 Edit·Write로 한정). 사후 기록이 본질이므로 어떤 작업도 막지 않는다 — 시나리오 누락도 경고일 뿐 차단이 아니다. 파일 경로나 세션 식별자가 없으면 조용히 끝내고, 데몬·조회 실패는 모두 최선 노력(best-effort)으로 삼켜 작업 흐름을 방해하지 않는다.

## provenance

`clawket/hooks/hooks.json`이 PostToolUse를 matcher `Edit|Write`로 어댑터에 연결한다. `clawket/adapters/claude/post-tool-use.cjs`가 공유 핸들러(`clawket/adapters/shared/claude-hooks.cjs`의 `runPostToolUse`)에 위임한다. 그 루틴에서 열린 실행 기록 탐색 → 태스크 본문에 변경 자취 추가, 또는 데몬 활동 기록으로의 전송, 그리고 양쪽 경로에서의 시나리오 누락 경고(X3)를 확인했다.

## 미확정 (OPEN)
- [ ] OPEN: 한 세션에 복수의 열린 실행 기록이 있을 때 어느 태스크에 귀속시키는지의 선택 규칙(가장 첫 미종료 실행 기준으로 보이나 다중 동시 작업 시 정확성)은 추가 정독 필요.
