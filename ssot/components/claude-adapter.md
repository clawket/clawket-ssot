---
id: component.claude-adapter
kind: SystemComponent
title: Claude 어댑터(adapters/)
definition: Claude Code의 각 훅 이벤트를 받는 얇은 진입점들과, 강제 로직·설치 게이트·데몬 글루를 한곳에 모은 공유 fat-handler로 구성된, Claude 통합 전용 어댑터 계층.
purpose: Claude Code가 발생시키는 훅 이벤트(세션 시작·프롬프트 제출·도구 사용 전후·서브에이전트 시작/종료·플랜 모드 종료)를 받아 Clawket의 작업 규칙(활성 태스크 강제, PDD 안티패턴, 파괴적 명령 차단)을 실행하고, 데몬·CLI와의 글루를 제공하기 위해 존재한다. 어댑터 계층을 별도로 두는 이유는 미래에 다른 코딩 에이전트 런타임이 붙더라도 강제·글루 로직(공유 핸들러)을 재사용하고, 런타임별 진입점만 갈아 끼우기 위해서다.
realizedBy:
  - domain.hook-enforcement
  - integration.claude-code-hooks
implementedIn:
  - clawket/adapters/claude
  - clawket/adapters/shared/claude-hooks.cjs
  - clawket/adapters/shared/common.cjs
  - clawket/adapters/shared/session-context.cjs
  - clawket/adapters/shared/locale.cjs
dependsOn:
  - component.cli
  - component.daemon
  - component.install-gate
consumesApi:
  - endpoint.daemon-health
  - endpoint.daemon-activity
  - endpoint.daemon-plans-import-strict
integratesWith:
  - integration.claude-code-hooks
  - integration.git-destructive-guard
  - integration.model-tier-routing
  - integration.i18n-locale
  - integration.github-releases
impacts:
  - domain.hook-enforcement
  - persona.llm-coding-agent
  - concept.run
relatesTo:
  - to: integration.claude-code-hooks
    type: realizes
    note: hooks.json이 가리키는 핸들러 본체가 이 어댑터의 진입점들
  - to: component.install-gate
    type: contains
    note: 설치 게이트는 공유 핸들러(claude-hooks.cjs) 안에 ensureInstalled로 구현됨
governedBy:
  - invariant.active-task-required
  - invariant.evidence-required-on-done
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:systemcomponent
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로 — 무엇을·왜·누가·어떤 규칙·무슨 데이터. 코드(테이블·필드·경로·SQL) 옮겨적기 금지(식별자는 provenance/근거에만). 코드 분기/의도 불명은 판정 말고 OPEN. -->

## 책임

Claude Code가 작업 흐름 중 특정 시점마다 훅 이벤트를 발생시키면, 이 어댑터가 그 이벤트를 받아 Clawket의 규칙을 적용한다. 받는 시점은 세션 시작, 사용자 프롬프트 제출, 도구 사용 직전·직후, 서브에이전트 시작·종료, 그리고 플랜 모드 종료다.

각 이벤트마다 두 부분으로 나뉜다. 하나는 이벤트별로 따로 있는 얇은 진입점(2줄짜리 shim 수준)이고, 다른 하나는 그 진입점들이 공통으로 위임하는 공유 핸들러다. 강제 로직의 단일 진실은 이 공유 핸들러에 모여 있다 — 활성 태스크가 없으면 변경 도구를 막고, PDD 안티패턴(시나리오 연결 누락, 서브에이전트 추론 배치 과대, 완료 시 근거 누락, 동기화 맥락에서의 추론 디스패치)을 해당 이벤트 시점에 검사하며, 파괴적 쉘 명령을 카탈로그 대조로 하드블록한다. 설치 게이트와 데몬 발견·기동 글루도 같은 공유 핸들러 안에 있다.

## 경계와 의존

이 어댑터는 Node.js로 동작하는 플러그인 셸의 일부이며, 무거운 작업관리 로직은 직접 들고 있지 않다. 상태 조회·기록은 데몬(`component.daemon`)의 HTTP API를 호출해 처리하고, 설치가 필요하면 CLI·데몬 바이너리를 받는 설치 게이트(`component.install-gate`)를 구동한다. 데몬이 떠 있지 않거나 도달 불가하면 강제 검사는 차단 대신 우아하게 건너뛴다(graceful skip) — 데몬 부재가 사용자 작업을 막아서는 안 되기 때문이다.

데몬에 대해서는 헬스(`endpoint.daemon-health`), 활동·감사 기록(`endpoint.daemon-activity`), 플랜 모드 종료 시 strict 플랜 형상 검증(`endpoint.daemon-plans-import-strict`)을 호출한다.

## 통신 패턴

훅 이벤트는 Claude Code → 어댑터 진입점 → 공유 핸들러 방향의 단방향 호출이고, 핸들러는 결과로 "허용/차단" 결정을 돌려준다. 부수적으로 데몬 HTTP API를 호출해 컨텍스트를 읽거나 활동을 기록한다. 파괴적 명령이 막히면 그 사실을 데몬 활동 엔드포인트에 best-effort로 남기되, 기록 실패는 차단 자체를 무르지 않는다. 사용자 노출 메시지는 로케일 체인(`integration.i18n-locale`)으로 현지화된다.

## 하위 서브패키지 (책임 단위)

- 이벤트별 진입점(`clawket/adapters/claude/*.cjs`): 각 훅 이벤트당 하나씩, 공유 핸들러로 위임하는 얇은 shim.
- 공유 fat-handler(`clawket/adapters/shared/claude-hooks.cjs`): 강제 검사·설치 게이트·데몬 글루의 단일 진실.
- 공유 보조(common·session-context·locale): 데몬 발견·세션 컨텍스트 구성·현지화 등 공통 유틸.
- 파괴적 패턴 카탈로그: 하드블록 대상 쉘 명령 정의(`integration.git-destructive-guard`로 정렬).

## 미확정 (OPEN)
- [ ] OPEN: 강제 검사별 기본 강도(strict/warn)와 환경변수 오버라이드의 정확한 우선순위·조합 동작은 domain.hook-enforcement 및 각 hook.* 엔드포인트 노드에 위임.
