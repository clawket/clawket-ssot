---
id: endpoint.cli-task
kind: Endpoint
title: clawket task (create/view/list/update/delete/complete/cancel/block/unblock/append-body/search/decompose/tree/ancestors/descendants/stats)
definition: 직접 다루는 유일한 작업 단위인 태스크의 전체 라이프사이클·검색·분해·계보 조회를 담당하는 CLI 명령군.
realizedBy:
  - component.cli
  - component.daemon
implementedIn:
  - cli/src/main.rs
relatesTo:
  - to: concept.task
    type: mutates
    note: 태스크 생성·상태전이·삭제의 주 진입점
  - to: concept.execution-envelope
    type: mutates
    note: create 시 intent/prompt_template/success_criteria로 봉투를 서명, decompose가 봉투의 success_criteria·decomposition_policy를 읽음
  - to: concept.scenario
    type: backed-by
    note: --scenario-id로 시나리오와 태스크를 1:1 연결
  - to: concept.ticket-number
    type: reads
    note: TASK-ULID 외에 CK-N 티켓 번호로도 지정 가능
  - to: concept.tier
    type: reads
    note: --tier(low/med/high)로 요구 모델 등급 지정·필터
  - to: concept.run
    type: backed-by
    note: in_progress 전환 시 훅이 실행 기록을 생성
  - to: concept.audit-log
    type: relates-to
    note: 상태 변경이 감사 로그에 기록됨
governedBy:
  - invariant.evidence-on-done
  - invariant.cycle-required-for-tasks
  - invariant.blocked-reason-required
  - invariant.task-delete-draft-only
  - invariant.start-needs-active-plan-cycle
  - invariant.envelope-required-fields
  - invariant.scenario-id-one-to-one
  - invariant.terminal-auto-cascade
impacts:
  - concept.cycle
  - concept.plan
consumedBy:
  - component.skills
  - component.claude-adapter
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의
태스크는 Clawket에서 사람·LLM 에이전트가 직접 손대는 유일한 작업 단위다. 이 명령군은 그 태스크를 만들고, 진행시키고, 끝내고, 취소하거나 막고, 검색하고, 더 작은 하위 작업으로 쪼개고, 부모-자식 계보를 펼쳐 보는 모든 동작을 한곳에 모은다. 프로젝트·플랜·유닛·사이클은 태스크가 살아갈 틀을 잡아주는 상위 구조이고, 실제 "무슨 일을 했는가"의 기록은 전부 태스크에 모인다.

태스크의 상태는 할 일(todo) → 진행 중(in_progress) → 완료(done) 또는 취소(cancelled)로 흐르며, 외부 의존 때문에 멈춰야 할 때만 차단(blocked) 상태를 거친다. 완료·취소는 종착 상태이고, 한 사이클·플랜의 남은 태스크가 모두 종착 상태가 되면 그 사이클과 플랜은 자동으로 완료 처리된다(유닛은 상태가 없어 영향받지 않는다).

## 요청 / 응답
- **만들기(create)**: 작업 제목과 함께, 어떤 사이클에 속하는지, 우선순위·복잡도·예상 편집 수·종류(버그/기능/리팩터 등)·라벨·요구 모델 등급, 그리고 이 태스크가 무엇을 달성하는지(intent), 실행 에이전트가 돌릴 프롬프트(prompt_template), 성공 기준(success_criteria)을 함께 서명한다. 사이클을 비우면 데몬이 거부하므로 사이클 지정은 사실상 필수다. 부모 태스크가 있으면 그 실행 계약을 물려받아 일부 필수 항목을 생략할 수 있다. 시나리오 연결(scenario_id)과 배치 묶음(batch_id)도 이때 붙는다.
- **보기/목록(view/list)**: 단건 상세 또는 유닛·플랜·프로젝트·상태·담당 에이전트·사이클·라벨·등급·시나리오·배치 기준으로 거른 목록을 돌려준다. "사이클 미배정만(백로그)", "근거가 비어 있는 태스크만" 같은 점검용 필터와 페이지네이션을 지원한다.
- **수정(update)**: 제목·본문·상태·담당·우선순위·사이클 재배정 등을 바꾼다. 사이클에 빈 값을 주면 백로그로 빼낸다. 차단으로 바꿀 때는 차단 사유가, 완료로 바꿀 때는 근거가 필요하다.
- **종료 단축 명령**: 완료(complete)는 근거를 필수로 받는 완료 단축, 취소(cancel)·차단(block)·차단해제(unblock)는 각각 상태 전이의 단축이며 사유·코멘트를 함께 남긴다.
- **본문 덧붙이기(append-body)**: 기존 본문을 지우지 않고 텍스트를 이어 붙인다.
- **검색(search)**: 제목·본문을 키워드/의미/하이브리드 방식으로 찾는다.
- **분해(decompose)**: 실행 계약의 성공 기준과 분해 정책으로부터 하위 태스크 후보를 번호 목록으로 미리 보여주고, 전체 또는 선택 번호만 수락해 자식 태스크를 만든다.
- **계보 조회(tree/ancestors/descendants)**: 한 태스크를 뿌리로 한 하위 트리, 부모 사슬, 자손 목록을 펼친다. 스크립트로 파이프하기 좋은 JSON 출력과 사람이 보기 좋은 트리 출력을 함께 제공한다.
- **배치 통계(stats)**: 한 서브에이전트 디스패치 배치의 QA 판정 분포(통과/결함/시나리오오류)를 집계해, 큰 배치 후반부의 주의 분산을 잡아낸다.

## 권한 / 제약
- 구조화된 작업을 하려면 프로젝트가 등록돼 있고, 플랜이 활성이며, 태스크가 활성 사이클에 배정돼 있어야 한다. 활성 태스크 없이 변경 도구를 쓰려 하면 훅이 막는다.
- 태스크 생성은 사이클 지정이 사실상 필수다(데몬이 사이클 없는 생성을 거부). 실행 계약 필수 항목(intent 등)이 비면 데몬이 생성을 거부하되, 부모 태스크의 활성 계약을 상속하는 경우는 예외다.
- 완료(done) 전환에는 근거(파일:라인 또는 추론 요약)가 반드시 필요하다. 차단(blocked)으로 갈 때는 차단 사유가 필요하다.
- 삭제는 해당 플랜이 아직 초안(draft) 상태일 때만 허용된다. 진행 중인 작업은 삭제 대신 취소로 흔적을 남긴다.
- 종착 상태 도달 시 사이클·플랜 자동 완료가 일어난다(유닛은 무관).

## provenance
- 명령·옵션 정의: `cli/src/main.rs`의 TaskAction 정의(create/view/list/update/delete/append-body/search/complete/cancel/block/unblock/decompose/tree/ancestors/descendants/stats).
- 실제 처리: clawketd 데몬의 태스크 HTTP API(근거 필수·사이클 필수·차단 사유 필수 등 불변식 강제). 위 본문의 정책만으로 명령 의미·제약을 재구현할 수 있도록 작성했다.

## 미확정 (OPEN)
- [ ] OPEN: decompose의 strategy(auto/by-repo/scoped) 힌트가 하위 태스크 생성에 실제로 어떤 차이를 만드는지(주석 표기 외 동작) 데몬 분해 로직 정독 필요.
- [ ] OPEN: tree/ancestors/descendants의 깊이 상한(TREE_NODE_CAP) 구체 값과 초과 시 동작 확인 필요.
