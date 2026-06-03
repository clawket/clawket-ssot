---
id: endpoint.discover-next-round
kind: Endpoint
title: 다음 검증 라운드 생성
definition: POST /discover-loop/next-round — 직전 라운드의 사이클을 마무리하고, 직전 플랜에서 도메인·영역을 물려받아 다음 라운드(R+1)의 플랜·유닛·사이클을 자동으로 만든다.
realizedBy: [component.daemon]
implementedIn: [daemon/src/routes/discover.rs]
relatesTo:
  - { to: concept.plan, type: reads, note: "직전 라운드 플랜에서 도메인·라운드 번호·프로젝트를 추론한다" }
  - { to: concept.cycle, type: mutates, note: "직전 라운드의 활성 사이클들을 완료 처리한 뒤 새 라운드를 연다" }
  - { to: concept.convergence, type: governs, note: "직전 라운드가 이미 수렴(완료·결함0·시나리오오류0)이면 다음 라운드를 거절한다" }
  - { to: domain.pdd-verification, type: realizes, note: "라운드 간 이어달리기 절차를 구현한다" }
governedBy: []
impacts: [concept.plan, concept.cycle, concept.unit]
consumedBy: [endpoint.cli-discover-loop]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

검증 루프가 한 라운드를 끝내고 아직 수렴하지 않았을 때, 다음 라운드를 자동으로 여는 명령이다. 직전 라운드의 플랜만 가리키면, 그 플랜 제목에서 도메인과 라운드 번호를 읽어내고 그 플랜의 유닛 제목들에서 검증 영역들을 되살려, 다음 라운드(R+1)의 플랜·유닛·사이클을 새로 만든다. 사실상 "라운드 시작" 명령을 직전 라운드 정보로 자동 채워 호출하는 단축 경로다. 검증 루프 자동화의 핵심 이어달리기 단계로, 운영자가 매 라운드 메타데이터를 손으로 다시 입력하지 않아도 된다.

## 요청 / 응답

보내는 정보:
- 직전 라운드의 플랜(필수) — 도메인·영역·프로젝트·라운드 번호 추론의 출발점.
- (선택) 도메인 덮어쓰기 — 생략하면 직전 플랜 제목에서 추론.
- (선택) 검증 영역 목록 덮어쓰기 — 생략하면 직전 플랜의 유닛 제목에서 영역 이름을 떼어내 재사용.
- (선택) 만들 라운드 번호 — 생략하면 직전 라운드 + 1.

돌려주는 정보: "라운드 시작"과 동일한 형태 — 새로 만들어진 활성 플랜, 유닛 목록, 활성 사이클.

## 권한 / 제약

- 직전 플랜을 찾지 못하면 not-found로 거절한다.
- 직전 라운드가 이미 수렴 상태(플랜 완료 + 결함 0 + 시나리오오류 0)이면 다음 라운드를 거절한다. 수렴 후 라운드를 또 여는 것은 퇴행이므로, 그때는 별도의 "결함 수정 플랜"을 명시적으로 만들어야 한다는 정책이다.
- 영역을 직전 플랜에서도, 인자에서도 얻지 못하면 거절한다("영역을 명시적으로 달라").
- 다음 라운드가 직전 라운드의 사이클과 공존하지 않도록, 새 라운드를 만들기 전에 직전 플랜에 속한 활성 사이클들을 먼저 완료 처리한다(완료 실패 시 경고만 남기고 진행).

## provenance

데몬의 `daemon/src/routes/discover.rs`(next_round 핸들러, POST `/discover-loop/next-round`)가 제공하며, 내부적으로 같은 파일의 "라운드 시작" 로직을 추론된 인자로 재사용한다. CLI `clawket discover-loop next-round` 가 사용자 표면이다. "<도메인> Round <번호>" 제목 규칙과 "QA-<도메인> <영역>" 유닛 명명 규칙이 추론의 기반이므로, 이 명명 규칙은 라운드 시작 엔드포인트와 일관되게 유지되어야 한다.

## 미확정 (OPEN)
- [ ] OPEN: 직전 플랜이 완료가 아닌데(미수렴) 활성 사이클이 여러 개 남아 있는 비정상 상태에서 어느 사이클까지 완료 처리하는지의 경계 케이스 확인 필요.
