---
id: endpoint.cli-config
kind: Endpoint
title: clawket config (get/set/unset/list)
definition: "사용자별 Clawket 설정 값을 읽고 쓰는 CLI 명령으로, 기본 프로젝트나 데몬 포트 같은 환경 기본값을 키-값으로 보관한다."
realizedBy: [component.cli, component.daemon]
implementedIn:
  - cli/src/main.rs
relatesTo: []
governedBy: []
impacts: []
consumedBy: []
servesPersona: [persona.human-operator, persona.llm-coding-agent]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
`clawket config`는 한 대의 머신에서 Clawket이 기억해야 할 사용자 기본값을 다루는 명령이다. 매번 옵션으로 넘기기 번거로운 값들(예: 어떤 프로젝트를 기본으로 볼지, 데몬이 어느 포트를 쓰는지)을 한 곳에 저장해 두면, 이후 다른 명령들이 그 값을 기본으로 집어 쓴다. 사람 운영자가 환경을 처음 맞출 때, 또는 에이전트가 기본 프로젝트를 확정해 둘 때 사용한다.

네 가지 동작으로 나뉜다.
- 읽기: 특정 키에 저장된 값을 조회한다.
- 쓰기: 키에 값을 새로 지정하거나 덮어쓴다.
- 지우기: 키를 제거해 기본값으로 되돌린다.
- 전체 보기: 현재 저장된 모든 키와 값을 한 번에 나열한다.

## 요청 / 응답
- 읽기/지우기는 "어떤 설정 항목인지"를 가리키는 키 하나를 넘긴다(예: 기본 프로젝트, 데몬 포트).
- 쓰기는 키와 함께 새로 넣을 값을 같이 보낸다.
- 전체 보기는 입력 없이 현재 설정 묶음 전체를 돌려준다.
- 응답은 해당 키의 현재 값, 또는 설정 항목 전체 목록이다. 값은 단순 문자열 형태의 사용자 기본값이다.

## 권한 / 제약
- 설정은 사용자 홈 영역(사용자별 설정 디렉터리)에 보관되며 그 머신 안에서만 의미를 가진다 — 외부로 나가지 않는다.
- 키는 자유 텍스트가 아니라 Clawket이 의미를 부여하는 항목 이름이다. 의미 없는 키를 넣어도 다른 명령이 그 값을 참조하지 않으면 효과가 없다.
- 데이터 자체(프로젝트·태스크 등)를 바꾸지 않으며, 어디까지나 "기본값을 어떻게 잡을지"만 조정한다.

## provenance
CLI 단일 바이너리(`endpoint`는 `component.cli`가 제공)가 진입점이며, 실제 저장·조회는 로컬 데몬(`component.daemon`)의 설정 경로를 통해 처리된다. 명령 정의와 분기는 `cli/src/main.rs`의 Config 서브커맨드(get/set/unset/list)에 있다.

## 미확정 (OPEN)
- [ ] OPEN: 인식되는 설정 키 전체 카탈로그와 각 키가 다른 명령의 동작에 미치는 정확한 기본값 효과(예: 기본 프로젝트 미설정 시 cwd 해석 우선순위)는 데몬 측 설정 라우트를 정독해 확정 필요.
