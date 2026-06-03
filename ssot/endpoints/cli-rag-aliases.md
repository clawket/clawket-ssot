---
id: endpoint.cli-rag-aliases
kind: Endpoint
title: clawket find-similar / get-task-context / get-recent-decisions
definition: "세션 간 컨텍스트 복원을 돕는 세 가지 최상위 단축 명령 — 질의와 비슷한 태스크 찾기(find-similar), 한 태스크의 전체 맥락 묶음 가져오기(get-task-context), 최근 결정 기록 모아 보기(get-recent-decisions)."
realizedBy:
  - component.cli
  - component.daemon
implementedIn:
  - cli/src/main.rs
relatesTo:
  - to: concept.task
    type: reads
    note: "find-similar는 태스크를 의미 기반으로 검색하고, get-task-context는 한 태스크의 본문·실행기록·코멘트·첨부 지식을 한데 모은다."
  - to: concept.knowledge
    type: reads
    note: "get-recent-decisions는 결정(decision) 유형의 지식 항목을 최신순으로 모은다."
  - to: concept.run
    type: reads
    note: "get-task-context가 태스크의 실행 기록을 함께 묶어 반환한다."
  - to: concept.comment
    type: reads
    note: "get-task-context가 태스크의 코멘트를 함께 묶어 반환한다."
  - to: component.embedding-engine
    type: depends-on
    note: "find-similar의 의미 검색은 온디바이스 임베딩에 기반한 유사도 비교에 의존한다."
governedBy: []
impacts: []
consumedBy:
  - persona.llm-coding-agent
  - persona.human-operator
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의
LLM 에이전트는 세션이 바뀌면 직전까지의 작업 맥락을 잊는다. 이 세 명령은 그 맥락을 다시 끌어와(RAG 풀) 새 세션이 과거 작업 위에서 이어 일하게 돕는 빠른 진입점이다. 같은 일을 MCP 도구로도 할 수 있지만, 이 명령들은 명령줄에서 곧장 쓰는 최상위 별칭이다.

- `clawket find-similar`: 입력한 질의 문장과 의미가 비슷한 태스크를 찾는다(제목·본문 기준 의미 검색). "예전에 비슷한 걸 한 적 있나?"를 묻는 용도다.
- `clawket get-task-context`: 특정 태스크 하나의 맥락을 한 덩어리로 모아 준다 — 본문, 실행 기록, 코멘트, 첨부된 지식까지. 이 결과를 그대로 LLM 프롬프트에 넣어 세션을 복원하는 데 쓴다.
- `clawket get-recent-decisions`: 최근의 의사결정 기록(결정 유형 지식 항목)을 최신순으로 모아 준다. "지금까지 무엇을 어떻게 정했나?"를 빠르게 되짚는 용도다.

## 요청 / 응답
- find-similar: 보내는 정보는 찾고 싶은 질의 문장, 최대 몇 건까지 받을지, 어느 프로젝트로 한정할지. 돌려주는 정보는 질의와 의미가 가까운 순서의 태스크 목록.
- get-task-context: 보내는 정보는 태스크 식별자 하나. 돌려주는 정보는 그 태스크의 본문·실행 기록·코멘트·첨부 지식을 한 페이로드로 묶은 맥락 묶음.
- get-recent-decisions: 보내는 정보는 어느 프로젝트로 한정할지와 최대 몇 건까지 받을지. 돌려주는 정보는 결정 유형 지식 항목을 최신순으로 정렬해 상한 건수만큼 자른 목록.

## 권한 / 제약
- 모두 읽기 전용 조회다 — 상태를 바꾸지 않는다.
- 로컬 데몬이 살아 있어야 동작한다. 의미 검색(find-similar)은 온디바이스 임베딩 기반 유사도에 의존한다.
- get-recent-decisions는 "결정" 유형 항목만 골라 최신순으로 정렬·상한 적용한다.

## provenance
세 명령의 정의·핸들러는 `cli/src/main.rs`에 있다. CLI는 유사 태스크 검색·태스크 맥락 조회·결정 지식 조회를 데몬에 요청하며, find-similar의 의미 검색은 임베딩 엔진(`component.embedding-engine`)이 만든 벡터에 기반한다. 같은 기능군은 MCP read-only 도구로도 노출된다.

## 미확정 (OPEN)
- [ ] OPEN: 각 명령이 호출하는 데몬 조회 경로가 현재 데몬 라우트와 정확히 어떻게 매핑되는지 확인 필요(검사 시점 빌드 바이너리와 데몬 소스 간 명령 표면 차이 가능성).
