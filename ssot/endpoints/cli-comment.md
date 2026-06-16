---
id: endpoint.cli-comment
kind: Endpoint
title: clawket comment (create/list/delete/update)
definition: 태스크·유닛·플랜에 코멘트를 달고 조회·수정·삭제하는 CLI 명령군으로, 파괴적 삭제 대신 히스토리를 남기는 수단으로도 쓰인다.
realizedBy:
  - component.cli
  - component.daemon
implementedIn:
  - cli/src/main.rs
relatesTo:
  - to: concept.comment
    type: mutates
    note: 코멘트 생성·수정·삭제의 주 진입점
  - to: concept.task
    type: backed-by
    note: 코멘트는 task/unit/plan에 첨부
  - to: concept.audit-log
    type: relates-to
    note: 코멘트 변경이 감사 로그에 기록됨
governedBy:
  - invariant.preserve-history
impacts:
  - domain.governance-audit
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
코멘트는 태스크·유닛·플랜에 붙는 짧은 메모다. 진행 중 결정의 이유, 취소·차단 사유, 사람과 에이전트 사이의 메시지 같은 맥락을 남긴다. Clawket은 작업을 파괴적으로 지우기보다 흔적을 보존하는 쪽을 선호하므로, 태스크를 취소·차단할 때 그 사유가 코멘트로 함께 기록되어 "왜 이 작업이 멈췄는가"의 히스토리가 남는다.

## 요청 / 응답
- **만들기(create)**: 본문(마크다운)과 함께 태스크·유닛·플랜 중 어디에 붙일지, 작성자, 분류용 라벨을 받아 코멘트를 단다.
- **목록(list)**: 태스크·유닛·플랜 기준으로 코멘트 목록을 돌려준다.
- **수정(update)**: 기존 코멘트의 본문을 바꾼다.
- **삭제(delete)**: 코멘트를 제거한다.

## 권한 / 제약
- 코멘트는 반드시 태스크·유닛·플랜 중 하나에 첨부된다.
- 작성자를 명시하지 않으면 기본 주체(main)로 기록된다.
- 변경 이력 보존 원칙상, 취소·차단 등 상태 전이의 사유는 코멘트로 남겨 파괴적 삭제를 대체하는 것이 권장된다.

## provenance
- 명령·옵션 정의: `cli/src/main.rs`의 CommentAction 정의(create/list/delete/update).
- 실제 저장: clawketd 데몬의 comment HTTP API. 태스크 취소·차단 단축 명령이 사유를 코멘트로 함께 남기는 흐름과 연동된다.

## 미확정 (OPEN)
- [ ] OPEN: 코멘트 삭제가 실제로 물리 삭제인지 소프트 삭제(보존)인지, 그리고 라벨이 어디서 쓰이는지 데몬 comment 라우트에서 확인 필요.
