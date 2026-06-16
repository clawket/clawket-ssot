---
id: endpoint.handoff
kind: Endpoint
title: 데몬 GET /handoff
definition: 작업 디렉터리(cwd)를 기준으로 그 프로젝트의 현재 진행 상황을 하나의 인수인계 요약 문서로 집계해 돌려준다.
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/handoff.rs]
relatesTo:
  - { to: concept.project, type: reads, note: "cwd로 프로젝트를 찾아 요약 대상으로 삼는다" }
  - { to: concept.plan, type: reads, note: "프로젝트의 활성 플랜을 골라 요약 범위로 삼는다" }
  - { to: concept.unit, type: reads, note: "활성 플랜 아래 유닛들을 순회한다" }
  - { to: concept.task, type: reads, note: "진행 중·차단·다음 할 일 태스크를 분류해 진척도를 계산한다" }
governedBy: []
impacts: []
consumedBy: [capability.handoff]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
세션이 끊기거나 사람·다른 에이전트에게 작업을 넘길 때, "지금 어디까지 왔고 다음에 무엇을 해야 하는가"를 한 장의 읽기 좋은 요약으로 만들어 주는 인수인계용 엔드포인트다. 호출 시점의 작업 디렉터리를 단서로 해당 프로젝트를 자동으로 찾아, 그 프로젝트의 활성 플랜을 중심으로 현재 상태를 정리한다.

## 요청 / 응답
- 요청: 현재 작업 디렉터리 경로를 선택적으로 넘긴다. 경로로 프로젝트를 특정하지 못하더라도 등록된 프로젝트가 딱 하나뿐이면 그 프로젝트를 자동으로 대상으로 삼는다.
- 응답: 사람이 바로 읽을 수 있는 마크다운 형식의 요약 문서 한 덩어리를 돌려준다. 요약은 다음을 순서대로 담는다.
  - 프로젝트 이름과 생성 시각(헤더)
  - 진척도: 활성 플랜에 속한 전체 태스크 중 완료된 비율(완료/전체 및 백분율)
  - 진행 중인 작업 목록(담당자가 있으면 함께 표시)
  - 막혀 있는(blocked) 작업 목록
  - 다음에 착수할 후보(아직 시작 전인 todo 태스크, 최대 10건)
  - 답을 기다리는 미해결 질문 목록
  - 그동안 내려진 설계 결정 요약(결정 유형 지식 항목의 제목과 앞부분 발췌)
- 대상 프로젝트를 끝내 찾지 못하면 "프로젝트 없음" 안내를, 프로젝트는 있으나 활성 플랜이 없으면 "활성 플랜 없음" 안내를 담아 돌려준다.

## 권한 / 제약
- 로컬 데몬에 붙은 클라이언트가 호출하는 읽기 전용 집계 엔드포인트다. 상태를 바꾸지 않는다.
- 요약 범위는 "활성 플랜 하나"다. 활성으로 간주하는 범위에는 진행 중·승인됨·초안 상태의 플랜이 포함되며, 그중 하나를 골라 그 아래 유닛·태스크만 집계한다.
- 유닛은 상태가 없는 순수 그룹핑이므로 "완료된 유닛" 개념으로 묶지 않고, 완료 여부는 태스크 단위로만 계산한다.
- 다음 할 일·질문 목록 등은 과도하게 길어지지 않도록 상한(예: todo 10건)을 둔다.

## provenance
- 라우터/핸들러: `daemon/src/routes/handoff.rs`
- 제공 컴포넌트: 로컬 데몬(`component.daemon`)
- 소비처: 세션 인수인계 역량(`capability.handoff`)

## 미확정 (OPEN)
- [ ] OPEN: 활성 플랜 후보가 여럿일 때 어느 하나를 고르는 우선순위 규칙(현재는 목록에서 처음 매칭되는 것)이 의도된 정책인지 owner 확인 필요.
