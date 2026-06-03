---
id: concept.demo-seed-chain
kind: Concept
title: 데모 시드 체인(Demo Seed Chain)
definition: "히어로 데모를 촬영하기 전에 CLI로 프로젝트→플랜→유닛→사이클→태스크의 전체 사슬과 보드를 채우는 카드들을 멱등(idempotent)하게 시딩해, 히어로 영상이 비어 있는 데모가 아니라 실제로 굴러가는 워크스페이스처럼 보이게 만드는 사전 상태."
relatesTo:
  - { to: concept.plan, type: mutates, note: "활성 플랜을 찾거나 만들고 active로 보장한다" }
  - { to: concept.unit, type: mutates, note: "플랜 아래 그룹핑 유닛을 보장한다" }
  - { to: concept.cycle, type: mutates, note: "유닛에 묶인 사이클을 만들고 activate한다" }
  - { to: concept.task, type: mutates, note: "촬영 주체 태스크를 in_progress로, 보조 카드들을 각 컬럼에 채운다" }
  - { to: concept.evidence-required, type: relates-to, note: "in_progress 태스크가 있어야 증거 필수 클라이맥스를 찍을 수 있다" }
  - { to: integration.clawket-cli-seed, type: realizes, note: "시딩은 clawket CLI 명령들의 조합으로 수행된다" }
  - { to: screen.web-board, type: leads-to, note: "보드가 todo/in_progress/blocked/done 네 컬럼이 모두 찬 실제 워크스페이스로 보이게 한다" }
governedBy: []
dependsOn:
  - integration.clawket-cli-seed
  - concept.project
implementedIn:
  - landing/scripts/capture/seed-demo-data.sh
  - landing/scripts/capture/capture-scenario.md
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:concept"]
---

<!-- 비개발자도 읽는 자연어. 코드 옮겨적기 금지. -->

## 정의

데모 시드 체인은 히어로 영상을 찍기 직전에 워크스페이스를 "실제로 일하고 있는 모습"으로 미리 꾸며 두는 준비 단계다. 빈 대시보드를 찍으면 마케팅 목업처럼 보이기 때문에, 촬영 전에 프로젝트부터 태스크까지의 전체 작업 사슬과 보드를 채우는 보조 카드들을 자동으로 만들어 둔다.

핵심 성질은 **멱등성**이다 — 시드 스크립트를 몇 번 돌려도 같은 결과로 수렴한다. 프로젝트는 고정된 키(DEMO)로 찾아 없을 때만 만들고, 플랜·유닛·사이클·태스크는 각각 제목으로 기존 것을 먼저 찾아 재사용하며, 없을 때만 새로 만든다. 상태도 마찬가지로 "이미 원하는 상태면 그대로 두고, 아니면 맞춘다." 따라서 재촬영(re-take) 시 매번 새로 시딩할 필요가 없다.

세 가지 운영 모드가 있다:
- 기본(보장): 프로젝트 + 활성 플랜 + 활성 사이클 + in_progress 태스크 + 보드 카드들이 존재하도록 보장.
- 리셋: 기존 태스크를 취소하고 사이클·플랜을 완료 처리한 뒤 처음부터 다시 시딩. (촬영 주체 태스크가 이미 done이 되어 클라이맥스를 못 찍을 때 사용)
- 점검: 아무것도 바꾸지 않고 현재 상태만 보고.

이 사전 상태는 증거 필수 클라이맥스(`concept.evidence-required`)의 전제다 — 진행 중(in_progress) 태스크가 보드에 있어야 그것을 done으로 옮기며 증거 필수 거부 장면을 연출할 수 있기 때문이다.

## 엔티티 (DB)

시드 체인은 새로운 엔티티 종류를 만들지 않는다. 이미 존재하는 작업 관리 엔티티들의 한 인스턴스 묶음을 데몬 SQLite에 채울 뿐이다. 채워지는 사슬은 다음과 같다:

- 프로젝트 1개: 고정 키 DEMO, 데모 전용 작업 디렉터리에 바인딩.
- 플랜 1개: 출시 마케팅 자산 관련 플랜을 active 상태로 보장.
- 유닛 1개: 그 플랜 아래 "촬영" 그룹핑 유닛.
- 사이클 1개: 그 유닛에 묶인 사이클을 active로 활성화하고 목표 문구를 동기화.
- 촬영 주체 태스크 1개: in_progress 상태(클라이맥스용).
- 보조 카드 여러 개: 보드의 네 컬럼(todo·in_progress·blocked·done)을 모두 채우도록, 블록 카드는 차단 사유, done 카드는 증거 문자열까지 부여한다.

이 사슬은 제품의 작업 계층(프로젝트→플랜→유닛→사이클→태스크)과 그 시작 자격 규칙(플랜 active, 사이클 active 등)을 그대로 따른다 — 데모를 위해 규칙을 우회하지 않는다.

## API 표면

시드 체인은 별도 API가 아니라 clawket CLI 명령들의 조합으로 실현된다. 프로젝트·플랜·유닛·사이클·태스크의 생성/조회/상태 전이 명령(create·view·list·approve·activate·update·block·complete 등)을 JSON 출력으로 읽어 가며 find-or-create 방식으로 호출한다. 사전 점검으로 CLI 존재와 데몬 가동을 확인하고, 둘 중 하나라도 없으면 시딩을 멈춘다.

## 불변식

- 멱등성: 같은 입력으로 반복 실행해도 동일한 시드 상태로 수렴한다 — 키/제목으로 기존 엔티티를 재사용하므로 중복 생성이 없다.
- 작업 계층 규칙 준수: 시딩은 플랜을 active로, 사이클을 active로 보장한 뒤에야 태스크를 시작 상태로 옮긴다 — 제품의 시작 자격 불변식을 우회하지 않는다.
- 촬영 전제: 클라이맥스를 찍으려면 촬영 주체 태스크가 진행 중(미종료) 상태여야 한다 — done이 되어 있으면 리셋이 필요하다.

명시적 invariant 노드와의 직접 연결은 확인되지 않았다(OPEN).

## 구현 위치 (provenance)

- `landing/scripts/capture/seed-demo-data.sh`: 프로젝트→플랜→유닛→사이클→태스크 사슬과 보드 보조 카드를 멱등하게 시딩하는 셸 스크립트. 기본/리셋/점검 모드와 종료 코드 규약, find-or-create 로직, 보드 컬럼 채우기 로직이 여기 있다.
- `landing/scripts/capture/capture-scenario.md`: 시드된 상태를 전제로 촬영이 어떻게 진행되는지(엔티티를 만들지 않고 시드된 상태를 투어함)와 재촬영 시 리셋 조건을 명시한 촬영 시나리오 문서.

## 미확정 (OPEN)
- [ ] OPEN: 시드에 쓰이는 정확한 CLI 명령 시그니처는 버전에 따라 달라질 수 있어 CLI 엔드포인트 노드에서 확정 필요.
- [ ] OPEN: 데모 전용 프로젝트 키/디렉터리·보조 카드 목록은 마케팅 카피 변화에 따라 갱신될 수 있음.
- [ ] OPEN: 시드 체인을 강제하는 명시적 invariant 노드 존재 여부 미확인.
