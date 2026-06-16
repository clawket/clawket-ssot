---
id: endpoint.plans-approve
kind: Endpoint
title: 데몬 POST /plans/{id}/approve
definition: "초안 플랜을 활성(active)으로 승인해 그 플랜 아래 태스크 시작 자격을 부여하는 전용 승인 엔드포인트."
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/plans.rs]
relatesTo:
  - { to: concept.plan, type: mutates, note: "플랜 상태를 active 로 전환" }
governedBy: [invariant.one-active-plan, invariant.error-code-stability]
impacts: [concept.task]
consumedBy: [component.web-dashboard, endpoint.cli-plan]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

플랜을 "초안" 상태에서 "활성" 상태로 올리는 승인 동작이다. Clawket의 워크플로우에서 태스크는 활성 플랜 아래에서만 시작할 수 있으므로, 이 승인은 사실상 "이 로드맵으로 일을 시작해도 된다"는 게이트를 여는 행위다. 사람이 웹 대시보드에서 플랜을 승인하거나, CLI `plan approve`로 같은 일을 할 때 이 경로가 호출된다. 일반적인 부분 수정(PATCH)으로는 초안을 활성으로 바꿀 수 없고, 반드시 이 전용 승인 경로를 거쳐야 한다.

## 요청 / 응답

요청에는 승인할 플랜의 식별자만 담는다. 성공하면 활성으로 바뀐 플랜의 갱신된 형상을 돌려주고, 동시에 "플랜이 변경됨" 이벤트를 데몬 이벤트 버스로 흘려 대시보드 등 구독자가 즉시 화면을 갱신할 수 있게 한다. 식별자에 해당하는 플랜이 없으면 "찾을 수 없음"으로 응답한다.

## 권한 / 제약

같은 프로젝트 안에는 활성 플랜이 하나만 존재할 수 있다는 불변식(한 프로젝트=한 활성 플랜)이 여기서 강제된다. 승인하려는 플랜 외에 이미 다른 활성 플랜이 같은 프로젝트에 있으면 승인은 거부되며, 단순 실패가 아니라 "single_active_plan"이라는 안정된 오류 코드와 함께 충돌 중인 기존 활성 플랜의 식별자를 응답 본문에 담아 돌려준다. 호출자는 그 식별자로 곧장 충돌 플랜으로 이동하거나 사람의 판단을 구할 수 있다. 이 오류 코드와 본문 형태는 클라이언트가 사람이 읽는 메시지를 파싱하지 않고도 분기할 수 있도록 안정적으로 유지된다. 로컬 데몬에 붙은 클라이언트만 호출할 수 있다.

## provenance

라우터 정의와 핸들러는 `daemon/src/routes/plans.rs`의 `/plans/{id}/approve` POST 경로(`approve`)에 있으며, 활성 플랜 충돌 검사 후 플랜 저장소(`plans::approve`)를 호출하고 변경 이벤트를 발행한다. 충돌 응답 형태는 같은 파일의 `single_active_plan_conflict` 빌더가 만든다. 제공 주체는 로컬 clawketd 데몬(component.daemon)이다.

## 미확정 (OPEN)
- (없음)
