---
id: endpoint.daemon-plans-import-strict
kind: Endpoint
title: 데몬 POST /plans/import/strict
definition: Plan Mode가 내놓은 플랜 마크다운이 정해진 엄격(strict) 형식을 지키는지 검증하고, 통과하면 Clawket 플랜으로 등록 가능한 구조로 파싱해 돌려주는 통로.
realizedBy:
  - component.daemon
implementedIn:
  - clawket/adapters/shared/claude-hooks.cjs
relatesTo:
  - to: concept.plan
    type: mutates
    note: 검증·파싱 결과로 Clawket 플랜 등록을 가능케 한다
  - to: component.daemon
    type: backed-by
    note: 검증·파싱을 수행하는 로컬 데몬
governedBy: []
impacts:
  - concept.plan
consumedBy:
  - endpoint.hook-plan-sync
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의
Plan Mode에서 작성된 플랜 글이 Clawket이 받아들일 수 있는 모양인지 깐깐하게 검사하는 관문이다. Claude가 Plan Mode를 끝내는 순간 plan-sync 훅이 그 플랜 글을 이 관문에 보내, 형식이 맞으면 곧바로 Clawket 플랜으로 등록할 수 있는 구조로 변환해 돌려받는다. 형식이 어긋나면 어디가 왜 틀렸는지 짚어주어 사람이 고칠 수 있게 한다. 즉 "Plan Mode 출력 → Clawket 플랜"을 잇는 검증 다리다.

## 요청 / 응답
보내는 정보는 플랜 마크다운 본문 한 덩어리다(셸 인용 문제를 피하려 호출 측은 본문을 임시 파일에 담아 전송한다). 검증을 통과하면 돌려받는 것은 플랜을 이루는 의미 단위 — 제목, 본문, 그리고 플랜을 구성하는 유닛 목록(각 유닛은 제목·시나리오·예상 사이클 수·모드를 가짐)이다. 이 구조가 그대로 플랜 등록의 입력이 된다. 통과하지 못하면 돌려받는 것은 "엄격 형식 위반"이라는 판정과 그 세부(몇 번째 줄·몇 번째 칸에서, 어떤 종류의 위반인지, 어떻게 고치면 되는지 힌트)다.

## 권한 / 제약
데몬이 로컬 루프백에만 바인딩되므로 같은 기기 안에서만 호출된다. 형식 위반이면 훅은 이를 하드 블록으로 다뤄 Claude에게 실패를 알리고 위반 위치·종류·힌트와 함께 우회 안내(해당 프로젝트의 Clawket 강제 일시 해제 방법)를 보여준다. 데몬이 내려가 있거나 응답하지 못하면(네트워크 오류) 검증 결과를 얻지 못하므로 훅은 엄격 검증 대신 종래의 수동 등록 안내로 넘어간다. 등록 대상 프로젝트는 호출 시점의 작업 디렉터리로부터 결정되며, 강제가 꺼진(disabled) 프로젝트에서는 이 관문 자체를 건너뛴다.

## provenance
호출 측 근거는 `clawket/adapters/shared/claude-hooks.cjs`의 `validateStrictPlan`(포트 파일에서 포트를 읽어 `/plans/import/strict`에 `{content}`를 POST, 200이면 `{title, body, units:[{title, scenarios, expected_cycles, mode}]}` 형태를 파싱, 위반이면 `strict_format_violation`의 line/column/kind/hint를 반환)과 `runPlanSync`의 후속 자동 등록 흐름이다. 검증·파싱 주체는 로컬 데몬(component.daemon)이고, 결과는 Clawket 플랜(concept.plan)으로 이어진다. 호출자는 plan-sync 훅(endpoint.hook-plan-sync)이다. 엄격 형식 스펙 문서 경로는 훅 메시지가 `clawket/cli/docs/plans/strict-format.md`로 안내한다.

## 미확정 (OPEN)
- [ ] OPEN: 엄격 형식의 규칙 본문(필수 섹션·허용 모드 값·시나리오 표기 규약)과 위반 종류(kind)의 전체 목록은 데몬 측 검증 코드와 strict-format 스펙 문서를 정독해 확인 필요.
