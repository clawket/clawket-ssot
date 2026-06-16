---
id: decision.release-cascade
kind: Decision
title: 데몬 main push는 플러그인 매니페스트 PR을 자동 발행하므로 명시 지시 없이 push하지 않는다
purpose: "데몬 레포의 main 브랜치 push가 일으키는 자동 릴리스·다운스트림 PR 연쇄를 어떻게 취급할지 — 일상적 push로 둘지, 명시적 의사결정 행위로 게이트할지"
definition: "데몬의 main push는 버전 bump·GitHub Release·플러그인 셸 레포의 components.json 갱신 PR까지 자동으로 연쇄(cascade)시키는 릴리스 진원지이므로, 사용자가 명시적으로 push/릴리스를 지시한 경우에만 push하고 활성 태스크가 있다는 이유만으로 자동 push하지 않는다."
relatesTo:
  - to: integration.github-releases
    type: relates-to
    note: cascade의 산출물이 게시되고 다운스트림이 받아가는 배포 경로
  - to: component.daemon
    type: relates-to
    note: 릴리스 순서의 첫 단계이자 cascade의 진원지
  - to: component.plugin-shell
    type: relates-to
    note: cascade가 자동으로 버전 핀 갱신 PR을 발행하는 대상 레포
supersedes: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:decision"]
---

<!-- Decision은 append-only. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 supersedes로 잇는다. -->

> Decision은 append-only다. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 `supersedes`로 잇는다. (스냅샷 원칙 — 히스토리는 여기와 git에만)

## 맥락 (Context)

데몬은 별도(형제) 레포에 있고, 그 main 브랜치에 push가 들어가면 릴리스 워크플로가 자동으로 발사된다. 워크플로는 커밋 prefix로 버전 상승을 판단하고, 빌드·GitHub Release 게시까지 진행한 뒤, 마지막에 플러그인 셸 레포(clawket/clawket)를 받아 버전 핀 파일(components.json)을 새 버전으로 고친 PR을 자동 생성한다. 데몬은 릴리스 순서의 맨 앞(데몬 → CLI → 웹 → 데스크탑 → 플러그인 → 랜딩)이라 잘못된 릴리스가 CLI·웹·MCP·플러그인까지 호환성 영향을 끼친다. 문제는, 활성 태스크가 있으면 에이전트가 "변경 → push"를 자동으로 묶어버려, 사용자도 모르는 사이 다른 레포에 검토 대기 PR이 생기고 릴리스가 발사될 수 있다는 점이다.

## 결정 (Decision)

데몬 main push를 일상적 저장 행위가 아니라 릴리스 의사결정 행위로 취급한다. 사용자가 명시적으로 "push 해"/"릴리스해"라고 지시한 경우에만 push하고, 활성 태스크가 있다는 이유만으로 자동 push하지 않는다. push 전에는 커밋 prefix가 실제로 릴리스를 발사하는 종류인지, 그 결과 핀이 어느 버전으로 올라가며 호환성 범위가 그 버전을 수용하는지, 후속 단계 흐름이 사용자 의도와 일치하는지를 확인한다. 두 레포의 main push를 같은 응답에 묶지 않는다.

## 근거와 결과 (Consequences)

- **의도치 않은 다운스트림 PR 방지**: push 한 번이 다른 레포에 PR을 자동 발행하는 blast radius를 사람이 인지·승인한 뒤에만 발사하게 한다.
- **잘못된 prefix로 인한 릴리스 사고 방지**: 비배포 의도(docs/chore/refactor)인데 잘못된 prefix가 섞여 버전이 올라가고 PR까지 떨어지는 케이스를 push 전 점검으로 막는다.
- **릴리스 순서 직렬화**: 데몬이 진원지이자 첫 단계이므로, 같은 사이클에 CLI·웹도 push 대상이면 데몬의 PR이 머지되어 핀이 갱신된 뒤에 다음 단계를 진행한다 — 그렇지 않으면 stale base에서 분기한 두 PR이 같은 파일을 동시 수정해 한쪽이 다른 쪽을 덮어쓸 위험이 있다.
- **호환성 매트릭스 연동**: 데몬의 응답 모양·SSE 이벤트·에러 코드·스키마 같은 wire contract가 바뀌는 릴리스라면, 같은 사이클에 호환성 범위 갱신이 포함되는지 확인한다.

> 이 결정은 전역 가드레일(활성 태스크 없이 변경 차단)로는 덮이지 않는다. 활성 태스크가 있어도 main push가 다른 레포에 PR을 만들고 릴리스를 발사한다는 데몬 특화 cascade는 별도 인지가 필요하기 때문이다.

provenance: cascade 정의와 push 게이트 규약 — `daemon/.claude/rules/release-cascade-to-plugin-manifest.md`. 워크플로 진입·버전 bump·bump-manifest job(플러그인 레포 clone → components.json 수정 → PR 생성) — `daemon/.github/workflows/release.yml`(`on.push.branches: [main]`, bump/build/publish/bump-manifest jobs).
