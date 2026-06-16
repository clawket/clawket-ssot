---
id: screen.docs-getting-started
kind: Screen
title: 설치 가이드 화면(Getting Started)
definition: /docs/getting-started 경로의 영문 설치 가이드 화면. 플러그인 설치(마켓플레이스 추가 + 설치) → 프로젝트 등록 → 구조화된 워크플로우로 첫 태스크 생성·시작 → --evidence 로 완료까지를 단계별 코드 예시와 함께 안내하고, 마지막에 어떤 훅이 무엇을 했는지 요약한다.
purpose: 신규 사용자가 별도 SaaS 계정·API 키 없이 로컬 환경에서 Clawket 플러그인을 설치하고 첫 태스크를 근거와 함께 완료하기까지의 전 과정을 따라 할 수 있게 한다.
servesPersona: [persona.human-operator, persona.llm-coding-agent]
realizedBy: [component.landing-site]
implementedIn: [landing/src/pages/docs/GettingStarted.tsx]
consumesApi: []
relatesTo:
  - { to: capability.register-project, type: relates-to, note: "clawket project create 로 작업 디렉터리를 프로젝트로 등록하는 절차 안내" }
  - { to: capability.plan-and-approve, type: relates-to, note: "plan create/approve 로 플랜을 active 전환하는 절차 안내" }
  - { to: capability.manage-task-lifecycle, type: relates-to, note: "task create→in_progress→done(--evidence) 라이프사이클 안내" }
  - { to: concept.execution-envelope, type: relates-to, note: "task create 시 intent·prompt_template·success_criteria 등 봉투 필드가 필요함을 설명" }
  - { to: screen.docs-workflow, type: leads-to, note: "구조화된 워크플로우의 자세한 규칙은 Workflow 페이지로" }
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:screen"]
---

## 화면 목적

처음 Clawket을 쓰는 사람을 위한 손에 잡히는 설치 안내다. "계정도 API 키도 없고, 모든 것이 로컬 SQLite에서 돈다"는 전제를 먼저 못 박은 뒤, 플러그인 설치부터 첫 태스크를 근거와 함께 완료하는 데까지를 복사해 붙일 수 있는 명령 블록으로 차례차례 보여 준다.

## UI 요소 / 입력 필드

순서대로 다음 절(section)이 나열된 영문 문서다.

- 머리말: Clawket이 Claude Code 플러그인으로 배포되며 첫 세션에서 GitHub Releases의 사전 빌드 바이너리(CLI+데몬+웹 번들)를 받아 설치하므로 Rust/Node 툴체인이 설치 시점에 필요 없다는 설명.
- 사전 준비물: Claude Code CLI 로그인, PATH의 Node.js 20+(플러그인 setup 훅 용도), macOS/Linux/Windows 지원, Rust 불필요.
- 설치: 마켓플레이스 추가와 플러그인 설치 명령. 첫 SessionStart 훅이 멱등 설치 게이트를 돌려 핀된 CLI·데몬·웹 번들을 받고, 이후 세션은 버전 마커가 맞으면 건너뛰며, 온디바이스 임베딩 모델(384차원, 50+개 언어)은 데몬이 첫 사용 시 지연 로딩한다는 설명.
- 프로젝트 등록: 변경 도구(Edit/Write/Bash/Agent/TeamCreate/SendMessage) 실행 전에 작업 디렉터리를 반드시 등록해야 하며, 에이전트가 project create 를 실행하고, 등록된 프로젝트가 localhost:19400 웹 대시보드 사이드바에 나타난다는 설명.
- 첫 태스크: Project → Plan(approve) → Unit → Cycle(activate) → Task 구조와, 플랜·사이클이 active가 되기 전엔 태스크가 todo를 벗어날 수 없다는 규칙, PreToolUse 훅이 활성 태스크 없는 변경을 우회 플래그 없이 하드블록한다는 점, 그리고 plan/unit/cycle/task 생성과 in_progress 전환까지의 명령 예시.
- --evidence로 완료: done 전환 시 데몬이 EVIDENCE_REQUIRED를 HTTP 400으로 강제하므로 검증 가능한 근거(테스트 출력·diff 요약·curl 응답) 없는 완료 주장은 API 계층에서 거부되며, 자식이 모두 종료되면 데몬이 유닛/사이클/플랜 완료를 자동 cascade한다는 설명과 명령 예시.
- "방금 무슨 일이 일어났나": SessionStart/UserPromptSubmit/PreToolUse/PostToolUse/done 각 단계가 한 일을 한 줄씩 회고.

코드 블록은 읽기용이며 사용자 입력 필드는 없다.

## 표시 데이터 / 호출 API

내용은 화면에 하드코딩된 영문 가이드 텍스트와 코드 예시다. 화면 자체는 데몬·DB를 호출하지 않는다(consumesApi 없음). 다만 본문이 설명하는 명령들은 사용자가 자기 터미널에서 실행하면 CLI를 통해 데몬에 작업을 만든다 — 화면은 그 절차를 가르칠 뿐 직접 호출하지는 않는다.

## 상태 / 엣지케이스

- 정적 문서라 로딩/에러 상태가 없다.
- 페이지 진입 시 자기 SEO 메타데이터(title/description/canonical)를 직접 설정한다.

## 미확정 (OPEN)
- 없음
