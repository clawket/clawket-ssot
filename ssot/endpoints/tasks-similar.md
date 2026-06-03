---
id: endpoint.tasks-similar
kind: Endpoint
title: 유사 태스크 조회 (GET /tasks/{id}/similar)
definition: 지정한 태스크의 제목·본문을 의미 벡터로 변환해 의미가 가까운 다른 태스크들을 유사도 점수와 함께 돌려준다.
realizedBy:
  - component.daemon
implementedIn:
  - daemon/src/routes/tasks.rs
relatesTo:
  - to: concept.task
    type: reads
    note: 대상 태스크의 제목·본문을 읽어 검색 시드로 쓰고, 후보 태스크들을 읽어 반환한다
  - to: domain.knowledge-rag
    type: backed-by
    note: 온디바이스 임베딩과 벡터 검색으로 의미 기반 유사도를 계산한다
governedBy: []
governs: []
dependsOn:
  - component.embedding-engine
consumesApi: []
providesApi: []
impacts:
  - domain.knowledge-rag
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의
"지금 이 작업과 비슷한 과거/현재 작업이 뭐가 있지?"에 답하는 조회 기능이다. 대상 태스크의 제목과 본문을 의미 벡터로 임베딩한 뒤, 같은 공간에 임베딩된 다른 태스크들과 의미적으로 가까운 순서로 후보를 찾아 돌려준다. 에이전트가 새 작업을 시작하기 전 비슷한 선례를 끌어와 맥락을 복원하거나, 중복 작업을 발견하는 데 쓰인다. 이 기능은 RAG 풀 별칭 명령과 MCP의 유사 태스크 도구가 내부적으로 의존하는 토대다.

## 요청 / 응답
요청에는 대상 태스크 식별자와 함께 선택적으로 (1) 몇 건까지 받을지(개수), (2) 특정 상태의 태스크만 추릴지(상태 필터)를 지정할 수 있다. 개수는 기본 10건이며 1~30건 범위로 제한된다.

응답은 유사 태스크 목록이다. 각 항목은 해당 태스크 본문 정보와 함께 유사도 정보를 가진다. 의미 유사도(코사인) 점수가 핵심이며, 향후 키워드·태그 일치 점수를 합성하기 위한 자리도 함께 노출된다. 대상 태스크 자신은 결과에서 빠진다. 임베딩을 만들 수 없는 경우(엔진 미가용 등)에는 빈 목록을 돌려준다.

## 권한 / 제약
로컬 데몬 호출자가 사용한다. 존재하지 않는 태스크를 가리키면 찾을 수 없음으로 응답한다. 노이즈 방지를 위해 의미 유사도에 하한(코사인 0.3)이 걸려 있어, 거의 무관한 근접-0 매치는 결과에 포함되지 않는다 — 이는 유사 태스크 MCP 도구가 잡음 섞인 결과를 보지 않도록 보장하기 위한 정책이다. 후보는 요청 개수보다 넉넉히 과다 조회한 뒤 하한과 상태 필터로 걸러 최종 개수만큼만 반환한다.

## provenance
구현은 데몬의 태스크 라우터(`daemon/src/routes/tasks.rs`)의 유사 태스크 핸들러에 있으며, 온디바이스 임베딩으로 시드 벡터를 만든 뒤 벡터 검색으로 근접 태스크를 찾고 유사도 하한으로 필터링한다. 의미 검색은 임베딩 엔진(component.embedding-engine)에 의존하고, 봉투/검색 제공 주체는 데몬(component.daemon)이다.

## 미확정 (OPEN)
- [ ] OPEN: 키워드(BM25)·태그 일치 점수의 실제 산출 방식은 아직 자리만 잡혀 있고 미구현(코드상 TODO)이라 합성 유사도의 최종 정책 확인 필요.
