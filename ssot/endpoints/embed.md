---
id: endpoint.embed
kind: Endpoint
title: 데몬 POST /embed — 임의 텍스트 임베딩 생성
definition: 보낸 텍스트를 온디바이스 임베딩 모델로 벡터화해 그 차원과 벡터 값을 돌려주는 엔드포인트.
realizedBy: [component.daemon, component.embedding-engine]
implementedIn: [daemon/src/routes/embed.rs]
relatesTo:
  - { to: domain.knowledge-rag, type: reads, note: "RAG의 의미 검색에 쓰이는 임베딩과 동일한 온디바이스 모델을 직접 노출" }
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의

아무 텍스트나 보내면 그것을 의미 벡터(임베딩)로 바꿔 돌려주는 엔드포인트다. 평소 태스크·지식의 의미 검색에 쓰이는 그 임베딩 계산을, 저장된 항목과 무관하게 임의의 문자열에 대해 바로 수행해 주는 저수준 도구에 가깝다. 진단·검색 동작 확인·외부 비교 같은 용도로 활용된다.

## 요청 / 응답

요청에는 임베딩할 텍스트 하나를 담는다.

응답은 그 텍스트의 임베딩 벡터와 벡터의 차원 수를 담는다. 즉 "이 문장의 의미를 수치 벡터로 표현하면 이렇다 / 길이는 몇이다"를 돌려준다.

빈 텍스트를 보내면 잘못된 요청으로 거부한다.

## 권한 / 제약

- 텍스트는 비어 있으면 안 된다. 빈 입력은 "잘못된 요청"으로 거부된다.
- 임베딩은 전적으로 온디바이스 모델로 계산된다 — 외부 임베딩 서비스로 텍스트가 나가지 않는다(로컬 우선·프라이버시 원칙과 일치).
- 로컬 데몬에 직접 붙는 호출이므로 로컬에서만 접근 가능하다.

## provenance

임베딩 라우터(`daemon/src/routes/embed.rs`)가 `/embed` 경로의 핸들러를 제공하며, 실제 벡터 계산은 온디바이스 임베딩 엔진(`component.embedding-engine`)에 위임한다. 데몬(`component.daemon`)이 이 라우터를 호스팅한다. 코드가 사라져도 "임의 텍스트를 로컬 모델로 임베딩해 차원과 벡터를 반환 / 빈 입력 거부 / 외부 전송 없음"이라는 의미만으로 재구현이 가능해야 한다.

## 미확정 (OPEN)
- [ ] OPEN: 임베딩 차원·모델 식별·정규화 여부 등 구체 사양은 임베딩 엔진 노드(`component.embedding-engine`)에서 확정한다. 본 노드는 텍스트→벡터 노출 엔드포인트의 의미 범위만 다룬다.
