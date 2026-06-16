---
id: screen.blog-post
kind: Screen
title: 블로그 포스트 화면(정적 HTML)
purpose: 방문자가 제품 변경의 배경 서사 — MCP 서버가 별도 npm 패키지에서 clawket CLI 안으로 통합된 이유·달라진 점·마이그레이션 방법 — 를 읽을 수 있게 한다.
servesPersona:
  - persona.human-operator
realizedBy:
  - component.landing-site
implementedIn:
  - landing/public/blog/2026-04-mcp-integrated.html
  - landing/vercel.json
consumesApi: []
relatesTo:
  - to: integration.mcp
    type: relates-to
    note: MCP가 별도 패키지가 아닌 clawket CLI의 mcp 서브커맨드로 통합된 변경의 배경·근거를 서술한다.
  - to: component.mcp-server
    type: relates-to
  - to: component.cli
    type: relates-to
  - to: screen.spa-index
    type: relates-to
    note: SPA 라우팅 rewrite 규칙에서 /blog 경로가 제외되어 이 정적 HTML이 fallback 없이 직접 서빙된다.
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:screen
---

## 화면 목적
랜딩 사이트의 블로그 포스트 한 편이다. 현재 게시된 글은 "MCP, integrated into clawket"으로, MCP 서버를 별도 npm 래퍼 패키지에서 clawket CLI 바이너리 안의 서브커맨드로 옮긴 변경을 다룬다. 제품 평가자·기존 사용자가 "왜 이렇게 바꿨고, 나에게 무엇이 달라지며, 어떻게 옮겨야 하나"를 이해하도록 돕는 서사형 콘텐츠다.

기술 블로그 성격상 다른 정적 페이지·SPA 페이지와 달리 손으로 작성한 독립 HTML 문서로 관리된다.

## UI 요소 / 입력 필드
입력 요소가 없는 읽기 전용 글이다. 자체 완결형 HTML(인라인 스타일, 다크 테마)로 다음 구조를 가진다.

- 메타 줄: 게시일과 작성 주체("clawket engineering").
- 제목과 도입부: 변경의 핵심 결론(MCP는 이제 같은 Rust 바이너리의 한 서브커맨드이고 기존 별도 npm 패키지는 폐기됨)을 먼저 제시.
- 본문 섹션: "왜 통합했나", "사용자에게 달라진 점", "이것이 가능케 하는 다음 단계"로 나뉜 산문.
- 코드/명령 인라인 표기와 푸터 링크.

## 표시 데이터 / 호출 API
데몬 API를 호출하지 않는 정적 문서다(consumesApi 없음). 전달하는 정보의 의미는 다음과 같다.

- 변경 요지: 기존의 별도 MCP 래퍼(npm 패키지)는 데몬으로 HTTP 호출을 전달하는 얇은 Node 래퍼였고, 이를 CLI 바이너리 안으로 흡수해 한 프로세스·하나의 계약 버전·단일 진실원천으로 정리했다.
- 통합 근거: 계약 표면이 두 곳(Rust 데몬 HTTP 핸들러 + Node 래퍼)으로 갈려 수작업 동기화 비용이 들고, 설치 경로도 둘로 나뉘던 문제를 해소.
- 사용자 영향·마이그레이션: 별도 npm 패키지가 더 이상 갱신되지 않으며 CLI의 `clawket mcp`로 대체된다는 안내.

## 상태 / 엣지케이스
정적 HTML이라 로딩·빈·에러 상태나 권한 분기가 없다. 라우팅 측면의 핵심 동작: 랜딩 사이트의 SPA fallback rewrite 규칙은 `/blog` 경로를 명시적으로 제외하므로, 이 글은 SPA 인덱스로 흡수되지 않고 정적 파일 그대로 서빙된다. 이 때문에 SPA 클라이언트 라우터의 영향을 받지 않는 독립 페이지로 동작한다.

## 미확정 (OPEN)
- [ ] OPEN: 블로그 목록·인덱스 페이지의 존재 여부와, 추가 포스트가 생길 때의 라우팅·내비게이션 규칙을 확인할 것.
