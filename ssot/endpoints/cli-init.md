---
id: endpoint.cli-init
kind: Endpoint
title: clawket init
definition: 신규 사용자가 약 5분 안에 첫 태스크를 끝까지 닫아볼 수 있도록 프로젝트·플랜·유닛·사이클·첫 태스크를 한 번에 스캐폴딩하는 멱등(여러 번 실행해도 중복을 만들지 않는) 온보딩 명령.
realizedBy: [component.cli, component.daemon]
implementedIn: [cli/src/init.rs, cli/src/main.rs]
relatesTo:
  - { to: concept.project, type: creates-instance-of, note: "튜토리얼 프로젝트(Hello Clawket)를 cwd 기준으로 찾거나 새로 만든다" }
  - { to: concept.plan, type: creates-instance-of, note: "Hello Clawket 플랜을 만들고 즉시 approve" }
  - { to: concept.unit, type: creates-instance-of, note: "Onboarding 유닛 생성" }
  - { to: concept.cycle, type: creates-instance-of, note: "Sprint 0 사이클을 만들고 activate" }
  - { to: concept.task, type: creates-instance-of, note: "첫 태스크를 만들고 in_progress 로 시작" }
governedBy: []
impacts: [domain.work-management]
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
`clawket init`은 처음 쓰는 사람을 위한 온보딩 명령이다. 빈 상태에서 "어디서부터 손대야 하나"를 고민하지 않도록, Clawket의 작업 계층(프로젝트 → 플랜 → 유닛 → 사이클 → 태스크) 한 벌을 자동으로 깔아주고 첫 태스크를 진행 중 상태까지 만들어 둔다. 사용자는 그 태스크를 닫아보는 것만으로 전체 작업 흐름을 한 바퀴 경험한다.

현재 이 명령은 `--tutorial` 모드로만 동작한다. 옵션 없이 호출하면 "튜토리얼 모드로 실행하라"는 안내를 내고 멈춘다. 작업 디렉터리는 기본적으로 현재 위치를 쓰며, 지정한 경로가 없으면 새로 만들어준다.

## 요청 / 응답
입력은 "어느 작업 디렉터리에 튜토리얼을 깔 것인지"(기본값은 현재 폴더)와 튜토리얼 모드 여부다. 명령은 그 디렉터리에 이미 등록된 프로젝트가 있으면 재사용하고, 없으면 "Hello Clawket"이라는 튜토리얼 프로젝트를 만든다.

응답으로는 6단계 진행 로그를 화면에 보여준다 — (1) 프로젝트 확보, (2) 플랜 생성·승인, (3) 유닛 생성, (4) 사이클 생성·활성화, (5) 첫 태스크 생성(읽어볼 만한 튜토리얼 링크가 본문에 담김), (6) 그 태스크를 진행 중으로 시작. 끝에는 "이제 무엇을 하면 되는지"(대시보드 열기, 태스크 닫기, 전체 가이드 보기)를 안내한다.

## 권한 / 제약
- 멱등성: 해당 디렉터리의 프로젝트에 이미 플랜이 하나라도 있으면 새로 만들지 않고, 기존 스캐폴딩(플랜 목록과 상태)을 그대로 보여준 뒤 종료한다. 같은 폴더에서 여러 번 실행해도 중복 플랜·사이클·태스크가 쌓이지 않는다.
- 첫 태스크가 곧장 시작될 수 있는 이유는, init이 그 전에 플랜을 승인하고 사이클을 활성화해 두기 때문이다. 즉 이 명령은 "태스크를 시작하려면 활성 플랜과 활성 사이클이 있어야 한다"는 작업 규칙을 어기지 않고, 오히려 그 전제를 자동으로 갖춰 준다.
- 로컬 데몬이 떠 있어야 동작한다(프로젝트·플랜·태스크 생성이 모두 데몬 API 호출이므로).

## provenance
구현은 CLI의 `cli/src/init.rs`(스캐폴딩 시퀀스 전체)와 `cli/src/main.rs`(명령 진입·인자 파싱)에 있다. 모든 단계는 로컬 데몬의 HTTP API(프로젝트·플랜·유닛·사이클·태스크 생성과 승인·활성화·시작)를 호출해 수행되므로 제공 주체는 CLI 바이너리와 데몬이다.

## 미확정 (OPEN)
- [ ] OPEN: `--tutorial` 외 다른 init 모드(예: 비튜토리얼 빈 프로젝트 부트스트랩)를 향후 추가할 계획이 있는지 — 현재는 명시적으로 미지원.
