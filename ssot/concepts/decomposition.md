---
id: concept.decomposition
kind: Concept
title: 태스크 분해(Decomposition) 제안
definition: "부모 태스크의 실행 봉투에 적힌 성공 기준(success_criteria)과 분해 정책(decomposition_policy)을 입력으로, 규칙 기반으로 하위 태스크 후보 목록과 정책 위반 경고를 만들어내는 결정적(deterministic) 제안 로직. 규칙의 단일 소유자는 데몬이며 MCP·웹은 같은 결과를 받아 보여주기만 하는 얇은 소비자다."
relatesTo:
  - to: concept.task
    type: reads
    note: 부모 태스크를 받아 하위 태스크 후보를 만든다 — 실제 트리 생성은 후보를 사람이 수락할 때 일어난다.
  - to: concept.execution-envelope
    type: reads
    note: 입력은 해석·상속 병합이 끝난 부모의 실행 봉투(성공 기준·분해 정책)다.
  - to: capability.decompose-task
    type: relates-to
    note: 이 개념(데몬 규칙)을 사람·에이전트가 실제로 호출해 트리를 만드는 행위가 분해 capability 다 — capability 쪽이 이 개념을 실현(realizes)한다.
governedBy: []
governs: []
dependsOn:
  - concept.task
  - concept.execution-envelope
implementedIn:
  - daemon/src/decomposition/mod.rs
  - daemon/src/decomposition/suggest.rs
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:concept"]
---

<!-- 작성 고도: 자연어 서술. 식별자는 provenance/근거에만. -->

## 정의

분해 제안은 "이 큰 태스크를 어떤 하위 태스크들로 쪼개면 되는가"를 자동으로 제안하는 개념이다. 핵심 원천은 부모 태스크의 실행 봉투에 적힌 성공 기준 목록이다. 성공 기준 한 줄이 곧 하위 태스크 후보 하나가 된다 — "무엇이 완료의 조건인가"가 곧 "무엇을 따로 해내야 하는가"라는 발상이다.

이 로직에는 두 가지 성격이 못박혀 있다. 첫째, LLM 호출도 DB 접근도 하지 않는 순수 규칙 기반이며 같은 입력에 항상 같은 결과를 낸다(결정적). 둘째, 규칙은 데몬이 단독으로 소유한다 — MCP 도구로 부르든 웹 제안 패널에서 보든 모두 같은 데몬 경로를 거치므로 표면마다 규칙이 갈라지지 않는다. 이는 검증기(validator)에서 쓰던 "데몬이 규칙을 갖고, 클라이언트는 얇게"라는 패턴을 그대로 따른 것이다.

분해에는 전략(strategy)이라는 힌트가 있다. 자동(auto)·표면별 범위(scoped: api/ui/infra)·레포별(by-repo: cli/daemon/web 분리)을 지정할 수 있고, 모르는 값을 주면 자동으로 떨어진다. 전략은 후보의 범위 힌트로 표시될 뿐, 후보 개수 자체는 성공 기준 줄 수가 결정한다.

## 엔티티 (DB)

분해 제안은 저장되는 엔티티가 아니라 그때그때 계산해 돌려주는 결과다. 자체 테이블이 없으며 DB 에 쓰지 않는다(부모 봉투를 떠오는 일과 자식 개수 세기는 호출하는 쪽이 맡고, 제안 로직 자체는 순수 계산만 한다).

결과는 부모 식별, 정규화된 전략, 적용된 최대 깊이, 기존 자식 개수, 제안된 하위 태스크 목록, 정책 위반 목록으로 구성된다. 후보 한 건에는 제목·근거(어느 성공 기준에서 나왔는지)·범위 힌트·상속할 봉투 키 목록이 담긴다. 화면이 깨지지 않도록 제목·근거 길이에는 상한이 걸려 잘린다.

## API 표면

부모 태스크에 대해 분해를 호출하면 위의 결과 구조를 JSON 으로 돌려준다. 같은 데몬 경로를 MCP 의 분해 도구와 웹 제안 패널이 공유한다. 후보를 실제 하위 태스크로 만드는 것은 별개의 수락 단계(capability.decompose-task)이며, 제안 자체는 아무것도 생성하지 않는다.

## 불변식

- 분해 제안은 LLM 을 부르지 않고 DB 를 건드리지 않는 순수·결정적 계산이다.
- 규칙의 단일 소유자는 데몬이다 — MCP·웹은 같은 결과를 받아 표시만 한다(표면 간 규칙 분기 금지).
- 후보 개수는 부모 봉투의 성공 기준 줄 수에서 나온다. 성공 기준이 하나도 파싱되지 않으면 후보 없이 오류 수준 위반을 낸다.
- 분해 정책의 한도 위반은 경고/오류로 보고된다: 최소 하위 개수 미달과 최대 개수 초과는 경고(초과분은 잘림), 요청 깊이가 정책 최대 깊이를 넘으면 오류, 정책 자체가 없으면 "기본값 사용" 경고.
- 성공 기준은 줄바꿈 문자열이든 문자열 배열이든 모두 받아들이며 불릿 기호·빈 줄은 정규화로 제거한다.

## 구현 위치 (provenance)

모듈 진입점과 "데몬이 규칙 소유, MCP·웹은 얇은 클라이언트"라는 설계 메모는 `daemon/src/decomposition/mod.rs`, 실제 규칙 기반 제안 계산(성공 기준 파싱·후보 생성·정책 위반 판정·JSON 직렬화)은 `daemon/src/decomposition/suggest.rs` 에 있다. 코드 주석은 이 기능을 LM-87 / RL-U6-04, 그리고 검증기 패턴(LM-151)과 연결한다.

## 미확정 (OPEN)
- [ ] OPEN: 분해를 트리거하는 HTTP 라우트와 MCP 도구 이름의 정확한 결선(부모 봉투 해석·자식 개수 수집을 담당하는 라우트 핸들러 위치) 확인 필요.
- [ ] OPEN: "기존 자식 개수"를 활용하는 후속 정책(이미 분해된 경우 제안 생략 등)이 현재 활성인지 미확인(코드상 향후 확장용으로만 노출됨).
