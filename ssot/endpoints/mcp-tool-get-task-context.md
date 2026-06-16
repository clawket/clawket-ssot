---
id: endpoint.mcp-tool-get-task-context
kind: Endpoint
title: MCP 도구 clawket_get_task_context (구현)
definition: "특정 태스크 하나와 그 주변 맥락(관련 지식·태스크 관계·코멘트·활동 이력)을 한 번의 호출로 묶어 돌려주는 읽기 전용 MCP 도구. \"이 티켓이 어떤 배경에서 만들어졌는지\"를 LLM이 한 페이로드로 파악하게 한다."
realizedBy: [component.mcp-server]
implementedIn: ["cli/src/mcp.rs"]
relatesTo:
  - { to: concept.task, type: reads, note: "조회 대상이 되는 태스크 본체(상태·우선순위·유형·소속 유닛/사이클·본문·시각 정보)를 읽어 반환한다" }
  - { to: concept.knowledge, type: reads, note: "기본 포함 섹션으로 해당 태스크에 첨부된 지식 항목 목록을 스니펫과 함께 읽는다" }
  - { to: concept.comment, type: reads, note: "comments 섹션을 명시하면 태스크 코멘트를 읽어 반환한다" }
  - { to: endpoint.mcp-get-task-context, type: relates-to, note: "동일 도구의 README 기준 인벤토리 노드 — 본 노드는 mcp.rs 구현 기준 상세" }
  - { to: endpoint.daemon-activity, type: relates-to, note: "history 섹션은 데몬 활동(activity) 기록을 태스크 단위로 조회해 채운다" }
governedBy: []
impacts: [capability.rag-pull]
consumedBy: [persona.llm-coding-agent]
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:endpoint"]
---

## 정의
LLM 코딩 에이전트가 작업 중인(또는 참조하려는) 태스크 하나를 지목하면, 그 태스크의 기본 정보와 함께 주변 맥락을 한 덩어리로 받아오는 읽기 전용 도구다. 여러 번에 나눠 "이 태스크 정보 줘", "여기 붙은 지식 줘", "관련 태스크 줘"를 묻는 대신, 한 번의 호출로 필요한 맥락을 모아 받아 "이 티켓이 무슨 배경으로 생겼는지"를 빠르게 이해하는 용도다. 키워드 검색이 아니라 특정 태스크를 ID로 지목하는 조회라는 점이 검색 계열 도구와 다르다.

## 요청 / 응답
요청으로는 맥락을 볼 대상 태스크의 식별자(사람이 읽는 티켓 번호 또는 내부 ID 둘 다 가능)와, 함께 받아올 맥락 섹션의 선택 목록을 보낸다. 선택 가능한 섹션은 네 가지다: 첨부 지식, 태스크 간 관계, 코멘트, 활동 이력. 아무것도 지정하지 않으면 지식과 관계 두 섹션이 기본으로 포함되고, 코멘트와 이력은 명시적으로 요청해야 따라온다.

응답은 항상 태스크 본체를 먼저 담는다. 본체에는 그 태스크가 어떤 상태이고(할 일/진행/완료 등), 우선순위·유형이 무엇이며, 어느 유닛·사이클에 속하는지, 본문 내용과 생성·시작·완료 시각이 무엇인지가 들어간다. 그 뒤로 요청된 섹션이 의미별로 붙는다:
- 지식: 태스크에 달린 지식 항목들의 제목·유형과 내용 앞부분 요약(스니펫). 전체 본문이 아니라 미리보기 길이로 잘라 토큰을 아낀다.
- 관계: 태스크 간 연결을 의미별로 분류해서 준다. "이 태스크가 막는 것 / 이 태스크를 막는 것 / 단순 관련 / 중복" 네 갈래로 나눠 묶는다.
- 코멘트: 작성자·작성 시각·본문.
- 이력: 이 태스크에 대해 누적된 활동 기록(최근 분량으로 제한해서).

## 권한 / 제약
읽기 전용 도구로 어떤 상태도 바꾸지 않는다. 동작하려면 로컬 데몬이 떠 있어야 하며, 도구는 데몬의 태스크·지식·관계·코멘트·활동 조회 경로를 모아 호출한 결과를 합쳐 돌려준다. 지정한 태스크가 없으면 오류 형태의 응답을 준다. 지식·코멘트의 본문은 미리보기 길이로 잘라 반환하므로 전체 원문이 필요하면 별도 조회가 필요하다. 모든 데이터는 로컬에 머무르며 외부로 나가지 않는다.

## provenance
구현은 CLI 바이너리에 내장된 MCP 서버 안에 있다(`cli/src/mcp.rs`의 `clawket_get_task_context`). 요청 형태는 같은 파일의 인자 구조(대상 태스크 ID와 포함 섹션 목록)로 정의되며, 실제 데이터는 데몬의 태스크 조회·지식 목록·태스크 관계·코멘트·활동 이력 경로를 병렬로 호출해 합친다. 제공 컴포넌트는 임베디드 MCP stdio 서버(component.mcp-server)다.

## 미확정 (OPEN)
- [ ] OPEN: 응답에 포함되는 태스크 관계의 분류 어휘(막음/막힘/관련/중복)가 데몬·CLI의 관계 모델과 1:1로 정합한지, 그리고 새 관계 유형이 추가될 때 이 도구의 버킷이 어떻게 확장되는지 owner 확인 필요.
