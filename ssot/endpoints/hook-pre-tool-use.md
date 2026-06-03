---
id: endpoint.hook-pre-tool-use
kind: Endpoint
title: 훅 PreToolUse
definition: "변경·에이전트 도구를 실행하기 직전에 호출되는 핵심 게이트 훅으로, 활성 태스크가 없으면 변경을 차단하고, 파괴적 쉘 명령을 하드블록하며, PDD 안티패턴(시나리오 누락·배치 초과·근거 누락·동기화 추론)과 모델 티어 적합성을 검사한다."
realizedBy:
  - component.claude-adapter
implementedIn:
  - clawket/hooks/hooks.json
  - clawket/adapters/claude/pre-tool-use.cjs
  - clawket/adapters/shared/claude-hooks.cjs
relatesTo:
  - to: concept.task
    type: reads
    note: 진행 중 태스크 유무로 변경 도구 허용/차단을 결정한다
  - to: concept.scenario
    type: evaluates
    note: 태스크 생성·갱신 시 시나리오 식별자의 존재·형식을 검사한다(X3)
  - to: concept.evidence
    type: evaluates
    note: done·blocked·cancelled 등 종료 전이 시 근거가 채워졌는지 검사한다(X8)
  - to: concept.batch
    type: evaluates
    note: 배치 검증 명령의 시나리오 수가 상한(30)을 넘지 않는지 검사한다(X7)
  - to: concept.tier
    type: evaluates
    note: 태스크가 요구하는 모델 등급과 현재 모델이 맞는지 게이트한다
governedBy:
  - invariant.active-task-required
  - invariant.destructive-hard-block
  - invariant.scenario-id-one-to-one
  - invariant.evidence-on-done
  - invariant.batch-size-30
  - invariant.sync-purity
impacts: []
consumedBy:
  - integration.claude-code-hooks
  - integration.git-destructive-guard
  - integration.model-tier-routing
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의

에이전트가 실제로 무언가를 바꾸거나 다른 에이전트를 부르려는 도구(파일 편집·쓰기·쉘 실행·노트북 편집·에이전트 스폰·팀 생성·메시지 전송, 그리고 태스크 생성·갱신)를 실행하기 직전에 발화되는 가장 무거운 가드레일 훅이다. Clawket의 "활성 태스크 없이는 변경 불가"라는 원칙이 실제로 강제되는 지점이며, 파괴적 명령과 검증 워크플로우의 안티패턴을 막는 최후의 방어선이기도 하다.

읽기 전용 도구(읽기·검색·브라우저 스냅샷 등)는 곧장 통과시키고, 변경·에이전트·태스크 계열 도구에 대해서만 검사를 적용한다.

## 요청 / 응답

입력은 실행하려는 도구 이름과 그 도구에 넘길 인자(예: 쉘 명령 문자열, 편집 대상 파일, 태스크 갱신 필드)다. 응답은 "허용" 또는 "거부 + 사유"다. 거부 시에는 사람·에이전트가 읽고 고칠 수 있는 구체적 사유 메시지를 함께 돌려준다.

검사는 여러 층이다.

- 파괴적 쉘 명령(되돌릴 수 없는 데이터 파괴 류)은 어떤 우회 경로보다 먼저, 무조건 하드블록한다. 이 차단에는 우회 환경변수가 없으며, 정말 필요하면 사람이 Claude 밖 셸에서 직접 실행하는 수밖에 없다. 오탐은 로컬 우회가 아니라 파괴 명령 카탈로그 자체를 고치는 방식으로만 해소하도록 설계됐다.
- 검증 워크플로우의 무결성 검사(PDD 안티패턴): 태스크에 시나리오 식별자가 비었거나 형식이 어긋나면(X3), 배치 검증 한 묶음의 시나리오 수가 상한을 넘으면(X7), 종료 상태로의 전이인데 근거가 비어 있으면(X8), 동기화 스크립트가 추론 로직을 품고 있으면(X9) 차단한다.
- 모델 티어 게이트: 태스크가 요구하는 등급(낮음/중간/높음)과 현재 모델이 맞지 않으면 차단하고, 모델 전환이나 티어 조정을 안내한다. 통과 시에는 채택된 티어를 후속 단계가 읽을 수 있게 환경에 실어 보낸다.
- 마지막으로 위 검사를 모두 통과해도, 등록·활성 프로젝트에서 진행 중 태스크가 하나도 없으면 변경 도구를 거부한다(활성 태스크 요구). 비활성 프로젝트는 이 워크플로우 강제를 면제받지만, 파괴적 명령 보호만은 면제되지 않는다.

## 권한 / 제약

도구 실행 직전마다 자동 실행되며 짧은 응답 예산 안에서 끝나야 한다. 읽기 전용 도구는 검사 없이 통과한다. 비활성 프로젝트·미등록 디렉터리는 활성 태스크 요구 등 워크플로우 강제를 우회하지만, 파괴적 명령 하드블록은 프로젝트 상태와 무관하게 항상 적용된다 — 사용자 데이터 무결성은 정책 상태보다 우선이기 때문이다.

## provenance

`clawket/hooks/hooks.json`이 PreToolUse를 matcher `Agent|TeamCreate|SendMessage|Edit|Write|Bash`로 어댑터에 연결한다. `clawket/adapters/claude/pre-tool-use.cjs`가 공유 핸들러(`clawket/adapters/shared/claude-hooks.cjs`의 `runPreToolUse`)에 위임한다. 그 루틴에서 읽기 전용 통과, 파괴 명령 우선 하드블록, X9/X7/X3/X8 검사, 티어 게이트, 활성 태스크 없음 거부의 순서와 분기를 확인했다. 안티패턴 검사 본체는 같은 파일의 `checkX3ScenarioId`·`checkX7BatchSize`·`checkX8Evidence`·`checkX9SyncReasoning`이고 파괴 패턴 카탈로그는 `clawket/adapters/shared/destructive-patterns.json`이다.

## 미확정 (OPEN)
- [ ] OPEN: 티어 다운그레이드가 현재 차단(deny)인지 경고(advisory)인지의 정확한 강도와, 모델 등급 판정 기준(모델명→티어 매핑)의 경계 케이스는 추가 정독 필요.
