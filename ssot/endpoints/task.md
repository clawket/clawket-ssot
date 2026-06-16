---
id: endpoint.task
kind: Endpoint
title: clawket task (태스크 전 라이프사이클·검색·분해·트리)
definition: "clawket task — 직접 다루는 유일한 작업 단위인 태스크를 생성·조회·수정·삭제하고, 상태 단축(complete/cancel/block/unblock)·본문 추가·검색·분해·트리/조상/자손 조회·배치 통계를 제공하는 CLI 명령 그룹."
realizedBy: [component.cli, component.daemon]
implementedIn: [cli/src/main.rs]
relatesTo:
  - { to: concept.task, type: mutates, note: "태스크를 생성·수정·삭제하고 todo→in_progress→done/cancelled/blocked 를 전이한다" }
  - { to: concept.execution-envelope, type: mutates, note: "create 시 intent·prompt_template·success_criteria 등 봉투 필드를 서명한다" }
  - { to: concept.scenario, type: reads, note: "--scenario-id 로 시나리오와 1:1 연결한다" }
  - { to: concept.tier, type: reads, note: "--tier low|med|high 로 요구 모델 등급을 정한다" }
  - { to: concept.backlog, type: reads, note: "--cycle \"\" 로 사이클을 떼면 백로그로 이동한다" }
  - { to: concept.tsv-evidence, type: reads, note: "stats 는 배치의 qa_status(pass/defect/scenario_error) 히스토그램을 집계한다" }
  - { to: endpoint.daemon-search, type: reads, note: "search 는 데몬의 FTS5/벡터/하이브리드 검색을 호출한다" }
governedBy: [invariant.evidence-on-done, invariant.blocked-reason-required, invariant.envelope-required-fields, invariant.cycle-required-for-tasks, invariant.task-delete-draft-only, invariant.terminal-auto-cascade, invariant.scenario-id-one-to-one]
impacts: [domain.work-management, domain.pdd-verification]
consumedBy: [endpoint.hook-subagent-stop, endpoint.hook-post-tool-use, component.claude-adapter]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

`clawket task` 는 Clawket에서 **직접 다루는 유일한 원자적 작업 단위** 인 태스크를 처음부터 끝까지 관리하는 명령 그룹이다. 태스크는 `todo → in_progress → done/cancelled` 를 거치며, 외부 의존이 있으면 `blocked` 로 둔다. 에이전트와 사람이 실제 일을 잡고·닫고·근거를 남기는 표면이며, 검증 루프의 시나리오가 1:1로 매핑되는 실행 대상이기도 하다.

제공하는 동작:
- **만들기(create)**: 제목으로 태스크를 만든다. 소속 유닛·사이클은 활성 플랜/사이클에서 자동 추론될 수 있다. 담당자, 우선순위, 복잡도, 의존 태스크, 부모 태스크, 타입(task/bug/feature 등), 라벨, 요구 모델 등급(tier), 시나리오 ID, 배치 ID, 그리고 **실행 봉투 필드**(intent·prompt_template·success_criteria 등)를 함께 서명한다.
- **보기/목록(view/list)**: 단일 태스크 상세, 또는 유닛·플랜·프로젝트·상태·담당 에이전트·사이클·라벨·tier·시나리오·배치 등 다양한 축으로 거른 목록을 본다. "근거(evidence)가 비어 있는 태스크만" 같은 점검 필터, 페이지네이션(limit/offset)도 있다.
- **수정(update)**: 제목·본문·상태·담당자·우선순위·복잡도·사이클·tier·시나리오·근거·배치·차단 사유 등을 바꾼다. 사이클에 빈 값을 주면 백로그로 떼어낸다.
- **삭제(delete)**: 플랜이 아직 draft 일 때만 허용된다.
- **본문 추가(append-body)**: 기존 본문을 지우지 않고 텍스트를 덧붙인다.
- **상태 단축**: done(complete, 근거 필수)·cancelled(cancel)·blocked(block, 사유)·todo 복귀(unblock). 종료 상태에 도달하면 마지막 비종료 태스크일 때 사이클·플랜이 자동 완료된다.
- **검색(search)**: 제목·본문에 대해 keyword(FTS5)/semantic(벡터)/hybrid 로 찾는다.
- **분해(decompose)**: 봉투의 success_criteria 와 decomposition_policy 로부터 하위 태스크를 제안하고(미리보기), `--accept ALL` 또는 `--accept 1,3` 으로 실제 생성한다.
- **트리/조상/자손(tree/ancestors/descendants)**: 서브트리를 유니코드 트리나 평면 배열로 보고, 부모 체인·자손을 DFS/BFS로 조회한다(스크립팅용 json/yaml/table).
- **배치 통계(stats)**: 한 서브에이전트 배치의 qa_status 히스토그램(pass/defect/scenario_error)을 집계한다 — 큰 배치 후반의 주의 분산을 감지한다.

## 요청 / 응답

- **만들 때** 보내는 정보의 의미: "무엇을 할 일인지(제목·본문)", "어디 소속인지(유닛·사이클, 자동 추론 가능)", "누가·어떤 등급으로(담당자·tier)", "어떤 시나리오를 실현하는지(scenario_id)", 그리고 **실행 계약**(이 일이 달성하려는 한 문장 intent, 실행 프롬프트 본문 prompt_template, 성공 기준 success_criteria). 응답은 만들어진 태스크의 식별 정보(ID·티켓 번호)와 추론된 소속을 담는다.
- **완료(complete)** 는 태스크 ID와 **근거(evidence)** 를 받는다 — file:line 참조나 자유 서술 요약. 근거 없는 done 은 거부된다.
- **검색**은 질의·모드·개수를 받아, 일치한 태스크들을 반환한다.
- **stats** 는 배치 ID를 받아 `{batch_id, total, pass, defect, scenario_error}` 형태의 집계를 반환한다.
- (필드의 정확한 직렬화 형태는 데몬 API 시점에 결정된다 — 여기서는 의미만 기술한다.)

## 권한 / 제약

- done 전환에는 근거가 필수 — 데몬이 EVIDENCE 를 강제(`invariant.evidence-on-done`).
- blocked 전환에는 차단 사유가 필수 — `invariant.blocked-reason-required`.
- 태스크 생성에는 봉투 필수 필드(intent·prompt_template·success_criteria)가 있어야 한다(부모로부터 active 봉투를 상속받는 경우 생략 가능) — `invariant.envelope-required-fields`.
- 태스크에는 사이클이 필요하다 — 데몬이 사이클 ID 없으면 거부(`invariant.cycle-required-for-tasks`).
- 삭제는 플랜이 draft 일 때만 — `invariant.task-delete-draft-only`. (이미 진행 중인 작업의 파괴적 삭제 대신 cancel 로 히스토리를 보존한다.)
- 종료(done/cancelled) 도달 시 마지막 비종료 태스크라면 사이클·플랜이 자동 완료로 cascade — `invariant.terminal-auto-cascade`.
- 시나리오는 태스크와 1:1로 연결된다 — `invariant.scenario-id-one-to-one`.
- 태스크 시작(in_progress) 자체는 active 플랜·active 사이클·활성 태스크 컨텍스트라는 상위 게이트를 통과해야 한다.

## provenance

- 명령 정의·핸들러: `cli/src/main.rs`(Commands::Task, TaskAction — create/view/list/update/delete/append-body/search/complete/cancel/block/unblock/decompose/tree/ancestors/descendants/stats). 상태 전이·검증·검색·분해·트리·집계는 데몬 HTTP API로 위임된다. (참고: v3.0에서 TaskAction::Envelope 는 제거되어 봉투 관리는 태스크 PATCH 본문으로 통합.)
- 제공 컴포넌트: `component.cli` + `component.daemon`. 검색은 `endpoint.daemon-search` 를 공용.
- 재구현 시 보존해야 할 의미: 상태 머신과 각 전이의 필수 조건(근거·사유·봉투·사이클), 자동 cascade, 시나리오 1:1, 분해가 봉투 정책에 종속된다는 점, 백로그 detach 의미.

## 미확정 (OPEN)

- [ ] OPEN: 봉투 필수 필드의 "부모 상속" 해석 규칙(어디까지의 조상 체인을 보는지)은 데몬 봉투 resolve 로직 정독으로 확인 필요.
- [ ] OPEN: decompose 의 max_depth 가 봉투 decomposition_policy.max_depth 와 충돌할 때의 우선순위는 분해 라우트 확인 필요.
