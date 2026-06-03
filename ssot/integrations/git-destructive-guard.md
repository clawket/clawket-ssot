---
id: integration.git-destructive-guard
kind: Integration
title: git/쉘 파괴적 명령 가드
definition: git reset --hard, rm -rf, DROP TABLE 같은 되돌릴 수 없는 쉘·DB 명령의 정규식 카탈로그를 도구 사용 직전 시점에 대조해, 일치하면 해당 Bash 도구 호출을 하드블록하는 가드 연동.
purpose: 코딩 에이전트가 데이터·이력을 영구히 파괴하는 명령(Clawket SSoT 데이터 삭제, 홈/루트 통째 삭제, DB 테이블 드롭, 커밋 안 된 변경 폐기 등)을 실수로 실행하는 사고를 사전에 차단하기 위해 필요하다. 사용자가 모르는 사이 비가역 손실이 일어나는 것을 막는 안전망이다.
integratesWith:
  - component.claude-adapter
  - component.plugin-shell
implementedIn:
  - clawket/adapters/shared/destructive-patterns.json
  - clawket/adapters/shared/claude-hooks.cjs
impacts:
  - endpoint.hook-pre-tool-use
  - persona.llm-coding-agent
  - concept.audit-log
relatesTo:
  - to: domain.hook-enforcement
    type: realizes
    note: 훅 강제 영역 중 파괴적 명령 차단을 담당하는 부분
  - to: integration.claude-code-hooks
    type: relates-to
    note: 도구 사용 직전 훅 이벤트 위에서 이 가드가 작동한다
governedBy:
  - invariant.destructive-hard-block
  - invariant.preserve-history
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:integration
---

<!-- 작성 고도(methodology §0): 비개발자도 읽는 자연어로 — 무엇을·왜·누가·어떤 규칙·무슨 데이터. 코드(테이블·필드·경로·SQL) 옮겨적기 금지(식별자는 provenance/근거에만). 코드 분기/의도 불명은 판정 말고 OPEN. -->

## 무엇과 연동하나

엄밀히 말하면 외부 시스템이라기보다, 코딩 에이전트가 실행하려는 쉘 명령에 대한 안전 가드다. Claude Code가 Bash 도구를 쓰기 직전에 발화하는 훅 시점에서, 실행하려는 명령 문자열을 미리 정해 둔 "파괴적 명령 카탈로그"의 정규식들과 대조한다.

카탈로그에는 비가역 손실을 부르는 명령 유형이 정리되어 있다 — Clawket 데이터 디렉터리를 노린 `rm -rf`, 홈/루트를 통째로 지우는 `rm -rf`, Clawket DB를 직접 건드리는 `DROP TABLE`/`DELETE`/`TRUNCATE`/`DROP COLUMN`, 하위 태스크·실행 이력을 동반 파괴하는 `--force` 삭제, 커밋되지 않은 변경을 날리는 `git reset --hard`, 볼륨을 지우는 `docker rm -v`, 접근을 봉쇄하는 `chmod 000`, 데이터 디렉터리를 노린 `find ... -delete`, Clawket DB로 향하는 출력 리다이렉트(`>`) 등이다.

어느 하나라도 일치하면 그 Bash 호출은 거부된다. 거부 메시지에는 왜 막혔는지(reason)와 안전한 대안(remediation)이 함께 제시되며, 이 두 문구는 영어·한국어·일본어 로케일 키로 현지화된다. 일부 치명적 패턴(홈/루트 통째 삭제)은 우회 옵션 자체가 없어, 사용자가 별도로 직접 승인하고 직접 실행하는 것 외에는 통과시키지 않는다.

## 구현 위치 (provenance)

차단 대상의 카탈로그는 데이터로 분리되어 있다(`clawket/adapters/shared/destructive-patterns.json`). 각 항목은 하나의 정규식과 컴파일 플래그, 분류(category), 그리고 로케일별 reason·remediation 문구를 갖는다. 카탈로그를 데이터로 빼둔 덕에 새 위험 패턴을 코드 변경 없이 항목 추가로 늘릴 수 있다.

매칭과 차단 로직은 공유 핸들러(`clawket/adapters/shared/claude-hooks.cjs`)에 있다. 핸들러는 카탈로그를 한 번 읽어 정규식을 미리 컴파일해 캐시하고, 명령 문자열을 순회 대조해 첫 일치 항목으로 거부를 내린다. 카탈로그 로드에 실패하면 그 실행 동안 하드블록이 비활성화되며 경고를 남긴다. 차단이 발생하면 베스트-에포트로 데몬의 활동 기록에 남긴다(감사 추적은 관측용이며, 차단 자체가 1차 가드다).

이 연동을 사용하는 내부 컴포넌트는 Claude 어댑터(`component.claude-adapter`), 배포 주체는 플러그인 셸(`component.plugin-shell`)이다.

## 불변식

- 카탈로그에 걸린 파괴적 명령은 하드블록되어야 한다(`invariant.destructive-hard-block`). 경고가 아니라 거부다.
- 작업 이력은 파괴 대신 보존되어야 한다(`invariant.preserve-history`) — 대안 안내가 항상 "상태만 변경(cancel)" 또는 "CLI 경유 정리"로 향하는 이유다.
- 일부 치명 패턴은 우회 경로가 없으며, 사용자의 명시적 별도 승인 없이는 통과하지 않는다.

## 영향 범위

이 가드는 도구 사용 직전 훅(`endpoint.hook-pre-tool-use`) 위에서 작동하므로 그 엔드포인트의 동작 일부를 이룬다. 직접적인 보호 대상은 코딩 에이전트(`persona.llm-coding-agent`)의 실행 행위이며, 차단 사건은 감사 로그(`concept.audit-log`)에 활동으로 기록된다. 가드가 무력화되면(카탈로그 로드 실패 등) 비가역 손실 사고의 방어선이 사라진다.

## 미확정 (OPEN)
- [ ] OPEN: 사용자가 "별도 승인"으로 우회하는 정확한 절차(어떤 신호로 핸들러가 통과를 허용하는지)가 패턴 종류별로 동일한지는 endpoint.hook-pre-tool-use 노드에서 정밀화 필요.
