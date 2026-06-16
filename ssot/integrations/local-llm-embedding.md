---
id: integration.local-llm-embedding
kind: Integration
title: 로컬 LLM 임베딩(Hugging Face 모델 + 온디바이스 추론)
definition: 데몬이 Hugging Face Hub에서 다국어 문장 임베딩 모델을 한 번 내려받아, 외부 LLM API 호출 없이 기기 안에서 직접 추론해 태스크·지식 텍스트를 의미 벡터로 바꾸는 연동이다.
purpose: 세션을 넘는 의미 검색(하이브리드 RAG)을 외부 임베딩 API에 의존하지 않고 제공하기 위함 — 어떤 본문 텍스트도 외부로 나가지 않게 하면서 한국어·일본어 등 50여 개 언어를 같은 벡터 공간에서 검색한다.
integratesWith:
  - component.embeddings
  - component.embedding-engine
  - component.daemon
implementedIn:
  - daemon/src/embeddings.rs
impacts:
  - domain.knowledge-rag
  - concept.task
  - concept.knowledge
relatesTo:
  - to: domain.knowledge-rag
    type: relates-to
    note: 임베딩이 하이브리드(키워드+의미) 검색의 의미 축을 채운다
  - to: invariant.local-only-no-egress
    type: relates-to
    note: 모델 가중치 다운로드를 제외하면 추론은 전부 기기 내에서 일어난다
governedBy:
  - invariant.local-only-no-egress
  - invariant.no-remote-telemetry
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:integration
---

## 무엇과 연동하나

이 연동은 두 단계로 외부와 만난다. 첫 번째는 모델을 처음 받아오는 순간이다. 데몬은 Hugging Face Hub에 접속해 다국어 문장 임베딩 모델(paraphrase 계열의 multilingual MiniLM, 384차원, 50여 개 언어 지원)의 설정 파일·토크나이저·가중치 세 가지를 한 번 내려받는다. 인증은 필요 없는 공개 모델이며, 다운로드는 최초 1회 게으르게(lazy) 일어나고 이후에는 받아둔 파일을 재사용한다.

두 번째 단계부터는 외부와 전혀 통신하지 않는다. 받아온 가중치를 기기 안의 추론 런타임(candle)으로 올려 두고, 들어온 텍스트를 토큰화한 뒤 모델을 통과시켜 평균 풀링·정규화한 의미 벡터를 만든다. 즉 "임베딩을 만든다"는 행위 자체는 어떤 외부 LLM API도 호출하지 않는 온디바이스 추론이다. 텍스트가 너무 길면 일정 글자 수에서 안전하게 잘라내고, 토큰 길이도 상한으로 제한한다. 빈 텍스트는 벡터를 만들지 않는다.

## 구현 위치 (provenance)

추론 엔진과 모델 다운로드 로직은 `daemon/src/embeddings.rs`에 있다. 이 모듈은 모델을 프로세스 수명 동안 한 번만 메모리에 적재해 두고(전역 슬롯), 이후 임베딩 요청은 그 적재된 모델을 공유한다. 데몬(component.daemon)이 지식·태스크를 저장하거나 갱신할 때 이 모듈을 불러 벡터를 만들고, 그 벡터는 의미 검색에 쓰인다. SSOT에서 이 추론 엔진을 가리키는 컴포넌트가 임베딩 엔진(component.embedding-engine)과 임베딩 모듈(component.embeddings)이다.

핵심은 "모델 파일을 받는 통로"와 "추론하는 통로"가 분리되어 있다는 점이다. 코드가 사라져도 본문만으로 재현하려면 이렇게 기억하면 된다 — 공개 다국어 임베딩 모델을 1회 받아 캐시하고, 그다음부터는 기기 안에서만 벡터를 만든다.

## 불변식

이 연동에 걸린 제약은 데이터 유출 금지다. 모델 가중치를 받는 최초 다운로드를 제외하면, 사용자의 태스크·지식 본문은 임베딩을 위해 외부로 절대 나가지 않는다(invariant.local-only-no-egress). 또한 임베딩 과정에서 별도의 원격 텔레메트리도 보내지 않는다(invariant.no-remote-telemetry). 이 두 제약이 "로컬 우선" 약속의 실질적 근거 중 하나다 — 외부 임베딩 API를 쓰지 않기로 한 선택이 곧 프라이버시 보장이 된다.

## 영향 범위

이 연동이 닿는 곳은 지식·로컬 RAG 영역(domain.knowledge-rag) 전체다. 임베딩 품질·차원·지원 언어가 바뀌면 의미 검색 결과가 달라지고, 태스크(concept.task)와 지식(concept.knowledge)이 검색되는 방식에 직접 영향을 준다. 모델 차원이 바뀌면 저장된 벡터의 호환성 문제도 생기므로, 모델 교체는 RAG 영역 전체를 다시 검토해야 하는 변경이다.

## 미확정 (OPEN)
- [ ] OPEN: 모델 가중치 다운로드 실패 시(오프라인·Hub 접근 불가) 의미 검색이 어떻게 우아하게 저하되는지(키워드 검색만으로 폴백하는지)의 정책 확인 필요
