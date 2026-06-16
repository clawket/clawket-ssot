---
id: component.embedding-engine
kind: SystemComponent
title: 온디바이스 임베딩 엔진
definition: 태스크와 지식 문서의 텍스트를 의미 검색이 가능한 384차원 숫자 벡터로 바꾸는, 사용자 기기 안에서만 동작하는 임베딩 계산기다. 다국어를 지원하는 사전학습 문장 임베딩 모델을 로컬에서 실행하므로 어떤 텍스트도 외부 서비스로 보내지 않는다.
purpose: "태스크·지식 텍스트를 의미 벡터로 변환해 로컬 RAG(유사도·하이브리드 검색)의 토대를 만든다. 외부 임베딩 API에 의존하지 않고 온디바이스로만 처리해 로컬 우선 원칙을 지킨다."
realizedBy:
  - domain.knowledge-rag
  - platform.local-first
implementedIn:
  - daemon/src/embeddings.rs
dependsOn: []
consumesApi: []
providesApi: []
integratesWith: []
impacts:
  - component.sqlite-storage
  - domain.knowledge-rag
  - capability.rag-pull
  - capability.wiki-knowledge
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:systemcomponent"]
relatesTo:
  - to: component.daemon
    type: belongs-to
    note: 임베딩 엔진은 데몬 프로세스 안에 내장되어 동작한다 — 별도 프로세스가 아니다.
  - to: component.sqlite-storage
    type: feeds
    note: 엔진이 산출한 벡터는 저장소의 벡터 가상 테이블에 적재된다.
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 분기/의도 불명은 OPEN. -->

## 책임

이 구성 요소는 글자로 된 내용(태스크 제목과 본문, 지식 문서의 제목과 내용)을 컴퓨터가 "의미가 비슷한지"를 견줄 수 있는 숫자 벡터로 바꾸는 일을 맡는다. 각 텍스트는 384개의 숫자로 이루어진 벡터 하나로 표현되고, 의미가 가까운 글일수록 벡터도 가깝게 위치한다. 이 벡터들이 모여 Clawket의 의미 검색·유사 항목 찾기·세션 간 컨텍스트 복원을 가능하게 한다.

핵심 특징은 "온디바이스"라는 점이다. 임베딩 계산은 외부 임베딩 API를 호출하지 않고 사용자 컴퓨터 안에서 직접 이루어진다. 이를 위해 다국어를 지원하는 사전학습 문장 임베딩 모델(paraphrase-multilingual-MiniLM-L12-v2 계열, 한국어·일본어 포함 50개 이상 언어 지원, 384차원)을 로컬에서 실행한다. 모델 파일 자체는 처음 사용할 때 한 번 내려받아 캐시한다.

## 경계와 의존

엔진은 데몬(`component.daemon`) 프로세스 안에 함께 들어가 있는 내부 모듈이다. 독립 서비스가 아니라 데몬의 일부로 동작하므로, 외부에서 직접 호출하는 인터페이스를 노출하지 않는다. 태스크나 지식이 만들어지거나 바뀔 때 데몬이 내부적으로 엔진을 불러 벡터를 계산하고, 그 결과를 SQLite 저장소(`component.sqlite-storage`)의 벡터 가상 테이블에 적재한다.

입력 텍스트에는 길이 상한이 있다. 너무 긴 글은 잘려서 처리되며(글자 수와 토큰 수 양쪽에서 제한), 빈 텍스트는 벡터를 만들지 않는다. 계산은 CPU에서 수행된다 — 별도의 GPU가 필요하지 않다.

## 통신 패턴

엔진은 평소 어떤 외부 통신도 하지 않는다. 단 한 번의 예외는 모델 파일을 처음 받아올 때다 — 사전학습 모델의 가중치·토크나이저·설정 파일을 모델 허브에서 최초 1회 내려받아 로컬에 캐시한다. 이후로는 캐시된 모델을 그대로 써서 완전히 오프라인으로 동작한다. 사용자가 만든 태스크·지식 텍스트 자체는 절대 외부로 전송되지 않는다.

## 하위 서브패키지 (책임 단위)

- 모델 로더: 다국어 임베딩 모델의 가중치·토크나이저·설정을 받아 메모리에 한 번만 올려 두고 재사용한다(지연 초기화).
- 토크나이저·전처리: 입력 텍스트를 모델이 이해하는 토큰으로 변환하고, 상한을 넘는 길이는 잘라낸다.
- 임베딩 계산기: 토큰을 모델에 통과시켜 384차원 벡터를 산출한다.

## 미확정 (OPEN)
- [ ] OPEN: 모델 파일을 받아오는 최초 1회 다운로드 경로가 GitHub Releases 배포 경로와 별개인지(모델 허브 경유) — 외부 네트워크 경로 정합성은 integration 측에서 추가 확인 필요.
- [ ] OPEN: 일괄(batch) 임베딩·재임베딩 트리거의 세부 조건은 데몬 라우트 측에서 별도 추적.
