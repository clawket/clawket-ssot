---
id: endpoint.cli-knowledge
kind: Endpoint
title: clawket knowledge (create/view/update/list/delete/search/import/export)
definition: 위키성 지식 항목을 만들고 고치고 검색하며 코드 저장소의 docs/와 주고받는 CLI 명령군.
realizedBy:
  - component.cli
  - component.daemon
  - component.embedding-engine
implementedIn:
  - cli/src/main.rs
relatesTo:
  - to: concept.knowledge
    type: mutates
    note: 지식 항목 생성·수정·삭제의 주 진입점
  - to: concept.task
    type: backed-by
    note: 지식 항목을 task/unit/plan에 첨부
  - to: concept.audit-log
    type: relates-to
    note: 지식 변경이 감사 로그에 기록됨
governedBy:
  - invariant.preserve-history
impacts:
  - domain.knowledge-rag
consumedBy:
  - component.skills
  - component.mcp-server
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:endpoint
---

## 정의
지식 항목은 결정·메모·근거·참고문서·명세처럼 작업 과정에서 쌓이는 위키성 문서다. 이 명령군은 그런 항목을 만들고, 보고, 고치고, 지우고, 키워드와 의미를 함께 쓰는 하이브리드 방식으로 검색하며, 코드 저장소의 docs/ 폴더와 양방향으로 주고받게 해 준다. 만들어진 지식은 자동으로 임베딩되어, 세션이 바뀌어도 의미 기반으로 다시 끌어올릴 수 있는 로컬 RAG 캐시의 일부가 된다.

지식은 항상 태스크·유닛·플랜 중 하나 이상에 붙어 "어느 작업의 맥락에서 나온 지식인가"를 잃지 않는다. 부모 지식을 지정하면 위키처럼 계층 구조를 이룰 수 있다.

## 요청 / 응답
- **만들기(create)**: 제목과 종류(문서·결정·참고·메모·명세)를 주고 본문(마크다운/일반텍스트/코드)을 담는다. 적어도 하나의 태스크·유닛·플랜에 첨부하며, 부모 지식을 지정해 위키 계층을 만들 수 있다.
- **보기/목록(view/list)**: 단건 상세 또는 태스크·유닛·플랜·종류로 거른 목록을 돌려준다.
- **수정(update)**: 제목·본문·종류·형식을 바꾸고, 변경 작성자를 감사 추적용으로 남긴다.
- **삭제(delete)**: 항목을 제거한다.
- **검색(search)**: 키워드(FTS5)·의미(임베딩 벡터)·하이브리드 방식으로 위키 전체를 찾으며, 종류 필터와 결과 수 상한을 받는다. 기본값은 하이브리드다.
- **가져오기(import)**: 지정한 작업 디렉터리의 docs/ 파일들을 지식 항목으로 흡수하며, 플랜·유닛에 붙이거나 미리보기(dry-run)만 할 수 있다.
- **내보내기(export)**: 지식 항목을 대상 디렉터리의 docs/로 써 내보내며, 특정 플랜·유닛에 붙은 것만 골라낼 수 있다.
- **위키 트리(wiki-tree)**: 부모-자식 관계를 따라 위키 계층을 펼친다. 임의의 하위 뿌리나 플랜으로 범위를 좁힐 수 있고, 들여쓰기 개요 또는 원시 목록으로 낸다.

## 권한 / 제약
- 지식 검색·임베딩은 데몬과 온디바이스 임베딩 엔진이 처리하며, 모든 처리가 로컬에서 일어난다(외부 전송 없음).
- 변경 이력 보존 원칙에 따라, 과거 결정·기록을 파괴적으로 덮어쓰기보다 새 항목·버전으로 남기는 것이 권장된다.
- import/export는 코드 저장소의 docs/ 디렉터리를 대상으로 하므로 작업 디렉터리(cwd) 지정이 필요하다.

## provenance
- 명령·옵션 정의: `cli/src/main.rs`의 KnowledgeAction 정의(create/view/update/list/delete/search/import/export/wiki-tree).
- 검색·임베딩 처리: clawketd 데몬의 지식 HTTP API + 온디바이스 임베딩 엔진(FTS5 + 벡터 하이브리드). 위 본문의 의미만으로 명령 동작을 재구현할 수 있도록 작성했다.

## 미확정 (OPEN)
- [ ] OPEN: import 시 docs/ 스캔 범위·파일 형식 필터·중복 처리 규칙을 데몬 import 라우트에서 확인 필요.
- [ ] OPEN: 지식 수정 시 버전이 실제로 어떻게 누적·보존되는지(이력 모델) 확인 필요.
