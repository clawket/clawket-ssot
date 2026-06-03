---
id: capability.defect-fix
kind: Capability
title: 결함 수정 태스크 등록·적용
definition: 검증 라운드가 "코드가 시나리오의 의도를 어긴다(defect)"고 판정한 행마다, 도메인별로 공유되는 결함해결 플랜의 그 라운드 유닛 아래 수정 태스크를 하나씩 등록하고, 그 태스크 안에서만 코드를 고쳐 다음 라운드에 외부에서 검증 가능한 완료 상태를 만드는 일이다.
purpose: 검증에서 드러난 각 결함을 추적 가능한 수정 태스크로 등록하고, QA 플랜과 분리된 결함해결 플랜 안에서 코드를 고쳐 다음 라운드 통과를 외부에서 확인 가능하게 만든다.
servesPersona:
  - persona.llm-coding-agent
  - persona.human-operator
realizedBy:
  - component.cli
  - component.daemon
  - component.skills
implementedIn:
  - clawket/skills/clawket-defect-fix/SKILL.md
relatesTo:
  - { to: concept.task, type: mutates, note: "결함마다 FIX 태스크를 생성하고 in_progress→done 으로 진행한다" }
  - { to: concept.scenario, type: reads, note: "어긴 의도를 가리키는 scenario_id 가 수정 태스크에 필수다" }
  - { to: endpoint.cli-task, type: calls, note: "task create / update 로 수정 태스크 라이프사이클을 다룬다" }
  - { to: endpoint.cli-plan, type: calls, note: "도메인당 하나의 결함해결 플랜을 확보·승인한다" }
  - { to: endpoint.cli-unit, type: calls, note: "라운드당 하나의 Round R 유닛을 결함해결 플랜 안에 만든다" }
  - { to: endpoint.cli-comment, type: calls, note: "원본 QA 태스크에 수정 태스크 ID·근거를 코멘트로 연결한다" }
  - { to: domain.pdd-verification, type: relates-to, note: "defect 행에 대한 라운드 내 후속 처리로 검증 루프가 호출한다" }
  - { to: capability.run-verification-loop, type: relates-to, note: "검증 라운드가 defect 를 보고하면 이 역량이 호출되고, 완료 후 R+1 로 제어를 넘긴다" }
impacts:
  - concept.task
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:capability"]
---

## 사용자가 할 수 있는 일

검증 라운드가 "코드가 시나리오의 의도를 어긴다"고 판정한 항목(defect)을 실제로 고치는 일이다. 핵심 규율은 두 가지다. 첫째, QA(검증) 플랜과 결함해결 플랜을 분리한다 — QA 플랜은 추론만 하는 곳이라 코드를 절대 고치지 않고, 모든 코드 수정은 결함해결 플랜의 수정 태스크 안에서만 일어난다. 둘째, 결함 하나당 수정 태스크 하나(1:1)를 등록해, 어떤 결함이 어떤 코드 변경으로 처리됐는지 추적 흔적을 남긴다.

수정 태스크의 완료(Done) 정의는 "자기 참조"여서는 안 된다 — "수정 완료" "구현 끝" 같은 표현은 거부되고, "다음 라운드(R+1)에서 이 시나리오 판정이 pass" 또는 "Given→When→Then 추적이 기대 결과에 깨끗이 도달한다"처럼 외부에서 검증 가능한 명제로 다시 써야 한다.

## 행위

- 도메인당 하나의 결함해결 플랜(`<도메인> QA 이슈 해결`)을 확보하고, 없으면 만들어 승인한다(모든 라운드가 이 한 플랜을 공유).
- 라운드당 하나의 `Round R` 유닛을 그 플랜 안에 만든다(없으면 생성, sequential 모드).
- defect 행마다 수정 태스크를 하나씩 등록한다 — 제목은 `FIX:`로 시작하고, scenario_id와 evidence(file:line)가 필수이며, 외부 검증 가능한 완료 조건을 명시한다.
- 원본 QA 태스크에 수정 태스크 ID·근거·evidence를 코멘트로 연결해 양방향 추적을 만든다.
- 수정 태스크를 in_progress로 전환한 뒤 evidence의 file:line에서 시작해 최소 범위로 코드를 고치고, done으로 닫으면서 수정 위치·내용·완료 검증 근거를 남긴다.

## 시스템 흐름

검증 라운드가 defect 행을 산출하면 이 역량이 호출된다. 먼저 결함해결 플랜과 해당 라운드 유닛을 확보하고(없으면 생성·승인), defect 행마다 수정 태스크를 등록한다. 코드 변경은 오직 수정 태스크가 in_progress인 동안에만 일어나며(가드레일이 이를 강제), 변경 후에는 "이 시나리오가 R+1에서 pass로 판정될까"를 코드 추론으로 확인한 뒤 태스크를 done으로 닫는다. 모든 결함의 수정 태스크가 done이 되면 제어를 검증 루프(R+1 서브에이전트 디스패치)로 넘기고, R+1은 같은 시나리오들을 새 태스크로 다시 평가한다.

## 어디에 구현되어 있나

수정 태스크 등록·적용 절차는 번들 스킬 `clawket/skills/clawket-defect-fix/SKILL.md`에 정의되어 있다. 플랜·유닛·태스크·코멘트의 실제 생성·전환은 CLI의 plan·unit·task·comment 명령을 통해 데몬·SQLite로 영구 기록된다. 코드 변경이 활성 수정 태스크 안에서만 가능하도록 막는 것은 훅 강제 영역의 책임이다.

## 미확정 (OPEN)
- [ ] OPEN: 수정 태스크가 어긴 시나리오의 scenario_id·evidence를 태스크 필드로 받는 정확한 CLI 인자 매핑은 endpoint.cli-task 노드에서 교차 확인 필요.
