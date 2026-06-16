---
id: integration.clawket-cli-seed
kind: Integration
title: clawket CLI 데모 시딩(랜딩 캡처용)
purpose: "히어로 영상이 '실제 제품이 돌아가는 진짜 워크스페이스'로 읽히도록, 녹화 직전에 보드를 채운 데모 작업 상태를 자동으로 만들고, 녹화 드라이버가 그 상태를 조회해 정확한 대상을 짚게 한다."
definition: "랜딩 저장소의 시딩 스크립트와 녹화 드라이버가 clawket CLI 를 외부 프로세스로 호출해(프로젝트·플랜·유닛·사이클·태스크의 list/create/update/complete/block 등) 데모 상태를 멱등하게 구성·조회하는 연동."
integratesWith:
  - component.cli
implementedIn:
  - landing/scripts/capture/seed-demo-data.sh
  - landing/scripts/capture/capture.mjs
impacts: []
relatesTo:
  - { to: endpoint.cli-project, type: depends-on, note: "데모 프로젝트를 키(DEMO)로 찾거나 없으면 만들고, 키로 프로젝트 식별자를 되찾는다." }
  - { to: endpoint.cli-plan, type: depends-on, note: "데모 플랜을 만들고 approve 로 active 전환해 태스크 시작 자격을 갖춘다." }
  - { to: endpoint.cli-unit, type: depends-on, note: "플랜 안에 캡처용 유닛을 만들어 사이클·태스크를 묶는다." }
  - { to: endpoint.cli-cycle, type: depends-on, note: "유닛에 묶인 사이클을 만들고 activate 로 active 전환한다 — active 사이클이 있어야 태스크가 시작 가능하다." }
  - { to: endpoint.cli-task, type: depends-on, note: "녹화 주인공 태스크를 in_progress 로 두고, 보드의 나머지 칸(todo·blocked·done)을 채우는 카드들을 create/update/block/complete 로 만든다." }
  - { to: endpoint.cli-dashboard, type: depends-on, note: "--check 모드에서 만들어진 데모 상태를 대시보드로 점검 출력한다." }
  - { to: concept.evidence, type: relates-to, note: "보드를 채우는 done 카드는 완료 시 근거(evidence) 문자열을 함께 넣어 완성하므로, evidence 필수 규칙을 그대로 따른다." }
governedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:integration"]
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드/식별자는 provenance·근거에만. -->

## 무엇과 연동하나

히어로 영상을 찍으려면 녹화 화면에 비칠 작업 데이터가 미리 있어야 한다. 빈 보드를 찍으면 "실제 제품" 으로 보이지 않기 때문이다. 그래서 랜딩 저장소는 clawket 의 명령줄 도구(CLI)를 외부 프로세스로 직접 불러, 녹화 직전에 데모 워크스페이스를 만들어 둔다.

연동은 두 갈래로 일어난다.

- **시딩(상태 만들기)**: 시딩 스크립트가 CLI 를 호출해 "Demo Launch" 라는 데모 프로젝트를 만들고, 그 안에 플랜·유닛·사이클·태스크를 연쇄로 세운다. 플랜은 승인해 활성화하고, 사이클도 활성화하며, 녹화 주인공 태스크는 진행 중 상태로 둔다. 여기에 더해 보드의 나머지 칸이 비지 않도록 할 일·진행 중·차단·완료 상태의 보조 카드들을 채워 넣는다. 이 스크립트는 멱등하다 — 이미 있으면 다시 만들지 않고 상태만 맞춰 주며, 처음부터 다시 깔거나(reset) 손대지 않고 점검만(check) 하는 모드도 있다.
- **조회(대상 짚기)**: 녹화 드라이버가 CLI 를 다시 호출해 데모 프로젝트의 식별자, 활성 사이클, 녹화 주인공 태스크의 식별자를 되찾아, 브라우저로 정확한 보드·태스크 화면을 열 수 있게 한다.

## 구현 위치 (provenance)

상태를 만드는 쪽은 시딩 스크립트(`landing/scripts/capture/seed-demo-data.sh`)이고, 만들어진 상태를 조회해 녹화하는 쪽은 녹화 드라이버(`landing/scripts/capture/capture.mjs`)다. 둘 다 clawket CLI 바이너리를 별도 프로세스로 실행하며, 그 CLI 가 바로 본 제품의 명령줄 컴포넌트(component.cli)다 — 이것이 이 연동이 사용하는 내부 컴포넌트다.

시딩이 다루는 작업 엔티티(프로젝트·플랜·유닛·사이클·태스크)와 그 상태 전이(플랜 승인→활성, 사이클 활성화, 태스크 진행/완료/차단)는 clawket 의 작업 관리 워크플로우 그대로다. 데모용이라 해서 다른 우회 경로를 쓰지 않고, 사용자가 평소 쓰는 것과 같은 CLI 명령과 같은 규칙을 따른다(각 명령 엔드포인트는 relatesTo 참조).

## 불변식

- 시딩이 만드는 상태 연쇄는 clawket 의 워크플로우 규칙을 그대로 지킨다 — 플랜이 활성이어야 태스크를 시작할 수 있고, 활성 사이클에 배정되어야 태스크가 시작 가능하며, 완료 카드는 근거(evidence) 없이는 완료 처리되지 않는다.
- 시딩은 멱등해야 한다 — 반복 실행해도 같은 데모 상태로 수렴하고, 의도치 않은 중복이 쌓이지 않는다.
- 녹화 드라이버는 시딩이 보장한 상태(활성 사이클·진행 중 주인공 태스크)가 없으면 빠르게 실패한다.

## 영향 범위

이 연동이 다루는 데이터는 캡처 전용 데모 워크스페이스이며, 본 제품의 운영 데이터와는 분리된 임시 상태다. 변경은 히어로 영상이 비추는 보드 구성에 파급되지만, 본 SSOT 인벤토리 등재 노드 중 직접 영향을 받는 대상은 확인되지 않았다(impacts 는 OPEN). 의존하는 CLI 명령과 워크플로우 규칙은 relatesTo 로 연결했다.

## 미확정 (OPEN)
- [ ] OPEN: 영향 대상(impacts) 확정 필요 — 데모 시딩 변경이 파급하는 인벤토리 등재 대상 미확인
