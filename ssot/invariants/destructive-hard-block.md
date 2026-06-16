---
id: invariant.destructive-hard-block
kind: Invariant
title: 카탈로그된 파괴적 명령은 실행 전에 차단된다
definition: "rm -rf로 사용자 홈·루트·Clawket 데이터 경로를 지우거나, git reset --hard, DROP/TRUNCATE/DROP COLUMN으로 DB를 파괴하거나, Clawket DB 파일을 덮어쓰는 등 카탈로그에 등록된 파괴적 명령은 도구 호출이 실행되기 전 정규식 매칭으로 거부된다."
governs:
  - domain.hook-enforcement
  - integration.git-destructive-guard
  - endpoint.hook-pre-tool-use
  - component.claude-adapter
  - concept.audit-log
governs_note: "차단의 목적은 감사 로그·트리거를 보존하는 정상 CLI 경로(예: task delete, project delete)로 사용자를 유도하는 것이다."
implementedIn:
  - clawket/adapters/shared/destructive-patterns.json
  - clawket/adapters/shared/claude-hooks.cjs
decidedBy: []
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:invariant
---

## 제약

Clawket은 되돌릴 수 없는 데이터 파괴를 명령이 실행되기 전에 막는다. 미리 정의된 "파괴적 명령 카탈로그"에 등록된 패턴에 걸리는 쉘 명령은 도구 호출 단계에서 거부되어 아예 실행되지 않는다. 카탈로그가 막는 대표적인 것들은 다음과 같다.

- 사용자 홈 디렉터리나 루트(`/`) 전체, 또는 Clawket의 데이터·캐시·설정·로그·플러그인 데이터 경로를 강제 재귀 삭제(`rm -rf`)하는 명령
- 커밋되지 않은 변경을 영구히 버리는 `git reset --hard`
- Clawket의 단일 진실 저장소인 SQLite DB를 대상으로 한 직접 파괴(테이블 드롭, 전체 행 삭제, DROP DATABASE, TRUNCATE, 컬럼 드롭)
- Clawket DB·데이터 파일을 출력 리다이렉트(`>`)로 덮어쓰는 시도
- Clawket의 플랜·유닛·사이클·프로젝트를 하위 태스크·실행 이력까지 함께 날리는 `--force` 강제 삭제
- 컨테이너 볼륨까지 지우는 `docker rm -v`, 후속 작업을 잠가버리는 `chmod 000`, 보호 데이터 디렉터리를 일괄 삭제하는 `find ... -delete`
- Clawket DB가 아닌 범용 대상이라도 비가역적인 DROP TABLE·TRUNCATE

차단은 단순히 거부에서 끝나지 않고, 각 패턴마다 "대신 이렇게 하라"는 안전한 정상 경로(예: 상태만 바꾸는 `task update --status cancelled`, 감사 로그와 트리거를 보존하는 `task delete`·`project delete`)를 사용자의 로케일에 맞춰 안내한다.

## 깨지면 무슨 일이 일어나나

이 제약이 없으면 한 줄의 잘못된 명령으로 사용자의 모든 작업 이력·지식·감사 추적이 복구 불가능하게 사라질 수 있다. 특히 Clawket은 작업 컨텍스트의 단일 진실 저장소이므로, DB 파일 하나가 지워지면 세션 간 누적된 모든 의사결정·실행 기록이 소실된다. 가장 위험한 카탈로그 항목(홈/루트 전체 `rm -rf` 등)은 우회 옵션 없이 무조건 거부하도록 되어 있어, 사용자가 직접 별도로 승인하고 손수 실행하지 않는 한 에이전트가 절대 수행할 수 없다.

## 코드에서 어떻게 강제되나

파괴적 패턴은 카탈로그 파일(`destructive-patterns.json`)에 정규식·분류·차단 사유·대안 안내로 등록되어 있고, 도구 실행 직전 단계(PreToolUse 훅)의 공유 핸들러가 이 카탈로그를 한 번 읽어 정규식을 컴파일해 두었다가 들어오는 쉘 명령과 대조한다. 매칭되면 그 도구 호출을 거부한다. 이 검사는 정확히 한 곳(cjs 핸들러)에서만 일어나며 데몬이 중복으로 검사하지 않는다 — 데몬 쪽은 상태 변이가 영속될지를 따로 책임진다. 카탈로그 파일 자체가 로드에 실패하면 그 실행 동안 하드블록이 비활성화되며 경고를 남긴다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(파괴적 명령 하드블록 정책의 근거 결정) 연결 필요
