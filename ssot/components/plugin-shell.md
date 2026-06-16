---
id: component.plugin-shell
kind: SystemComponent
title: 플러그인 셸(이 레포)
definition: 훅 매니페스트·프롬프트·스킬·MCP 등록 정보(.mcp.json)·컴포넌트 버전 핀(components.json)만 담고, 무거운 런타임(CLI·데몬·웹)은 직접 품지 않은 채 릴리스에서 받아 위임하는 얇은 Claude Code 플러그인 셸.
purpose: Claude Code에 Clawket을 끼워 넣는 설치·통합 단위. 플러그인이 무엇을 어떤 훅 이벤트에 연결하고 어떤 스킬·MCP를 노출하며 어떤 런타임 버전을 쓰는지를 선언하는 책임만 진다. 컴파일된 런타임을 플러그인에 직접 담지 않음으로써 플러그인 설치를 가볍고 빠르게 유지하고, 런타임 교체를 버전 핀 변경만으로 가능하게 한다.
realizedBy:
  - platform.claude-code-plugin
implementedIn:
  - clawket/package.json
  - clawket/components.json
  - clawket/.claude-plugin
  - clawket/hooks/hooks.json
  - clawket/.mcp.json
  - clawket/prompts
  - clawket/skills
dependsOn:
  - component.claude-adapter
  - component.install-gate
  - component.cli
  - component.daemon
  - component.web-dashboard
  - component.skills
  - component.mcp-launcher
integratesWith:
  - integration.claude-code-hooks
  - integration.mcp
  - integration.github-releases
impacts:
  - platform.claude-code-plugin
  - persona.llm-coding-agent
  - persona.human-operator
relatesTo:
  - to: platform.claude-code-plugin
    type: realizes
    note: 이 셸이 곧 Claude Code 플러그인 배포 형태를 구현한 산출물
  - to: component.install-gate
    type: depends-on
    note: 셸이 선언한 버전 핀을 읽어 런타임을 실제로 설치하는 것은 설치 게이트
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

이 레포 자체가 사용자에게 배포되는 "Clawket = Claude Code 플러그인"이다. 다만 이 셸이 직접 들고 있는 것은 무겁게 컴파일된 런타임이 아니라, 통합에 필요한 선언들뿐이다. 구체적으로 (1) 어떤 Claude Code 훅 이벤트를 어떤 핸들러에 연결할지 적은 훅 매니페스트, (2) 세션·작업 규칙을 주입하는 프롬프트 조각, (3) 슬래시 명령으로 노출되는 스킬, (4) MCP stdio 서버를 Claude Code에 등록하는 정보, (5) 어떤 버전의 CLI·데몬·웹 런타임을 쓸지 고정한 버전 핀을 담는다.

핵심 설계는 "얇은 셸 + 위임"이다. CLI·데몬·웹은 별도 레포에서 빌드돼 릴리스로 올라오고, 이 셸은 그것을 직접 포함하지 않은 채 버전만 가리킨다. 그래서 플러그인 설치 시 Rust 툴체인이나 npm 빌드가 돌지 않으며, 런타임 교체·갱신은 버전 핀 변경과 게이트 재설치만으로 끝난다.

## 경계와 의존

런타임 동작은 전부 다른 컴포넌트에 위임한다. 훅 이벤트가 들어오면 Claude 어댑터(`component.claude-adapter`)가 받아 처리하고, 처음 보는 머신이면 설치 게이트(`component.install-gate`)가 버전 핀을 읽어 CLI(`component.cli`)·데몬(`component.daemon`)·웹(`component.web-dashboard`) 바이너리를 받아 설치한다. MCP 등록은 런처(`component.mcp-launcher`)를 통해 CLI의 MCP 서브커맨드로 연결되고, 슬래시 명령은 번들 스킬(`component.skills`)이 제공한다.

외부 경계로는 세 가지 연동을 건다. Claude Code 훅 시스템(`integration.claude-code-hooks`), MCP stdio 등록(`integration.mcp`), 그리고 런타임 바이너리를 받아오는 GitHub Releases 경로(`integration.github-releases`)다.

## 통신 패턴

셸 자체는 상태를 직접 통신하지 않는다. 통신은 위임받은 컴포넌트가 한다 — 어댑터는 데몬 HTTP API를 부르고, MCP 서버는 데몬 포트를 발견해 호출한다. 셸은 그 연결의 "배선도"를 선언하는 역할이다. 버전 핀과 호환 범위 선언을 통해 어떤 런타임 버전이 이 셸과 함께 동작해야 하는지를 명시하고, 그 범위를 벗어나면 설치·진단 단계에서 걸린다.

## 하위 서브패키지 (책임 단위)

- Claude 어댑터(`component.claude-adapter`): 훅 이벤트를 받아 강제·글루 로직을 실행하는 진입점들.
- 설치 게이트(`component.install-gate`): 버전 핀을 읽어 런타임을 멱등 설치.
- MCP 런처(`component.mcp-launcher`): 등록된 MCP 서버를 stdio로 기동.
- 번들 스킬(`component.skills`): 대시보드+PDD 워크플로우 슬래시 명령 진입점.

## 미확정 (OPEN)
- [ ] OPEN: 셸과 런타임 간 호환 버전 범위(compat) 위반 시의 정확한 차단·경고 동작 경계는 component.install-gate / endpoint.cli-doctor 노드에 위임.
