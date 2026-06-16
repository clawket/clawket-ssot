---
id: decision.push-is-production
kind: Decision
title: main push가 곧 production 배포이므로 명시 지시 없는 commit/push 금지·사람 리뷰가 유일 backstop
purpose: 랜딩 레포에서 main 푸시의 production blast radius를 어떻게 게이트할지 — 자동 배포를 막는 기술 장치를 둘지, 사람 리뷰를 게이트로 둘지
definition: 랜딩 레포의 main 푸시는 Vercel Git 통합이 감지해 즉시 production 배포를 트리거하므로, 자동·반자동 흐름에서 사용자의 명시적 지시 없이는 commit/push/merge를 하지 않고 PR 경유 사람 리뷰를 유일한 backstop으로 둔다.
relatesTo:
  - to: component.landing-site
    type: governs
    note: 랜딩 레포의 변경 반영(배포) 흐름을 이 결정이 규정한다.
  - to: integration.vercel
    type: relates-to
    note: main push를 감지해 자체 워커에서 빌드·발행하는 주체가 Vercel Git 통합이다.
  - to: integration.i18n-sync-ci
    type: relates-to
    note: CI(빌드 artifact 업로드·i18n drift 검사)는 빌드 검증만 하고 push를 막지 않는다는 점에서 본 결정과 함께 읽힌다.
supersedes: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:decision"]
---

> Decision은 append-only다. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 `supersedes`로 잇는다.

## 맥락 (Context)

랜딩 레포에는 Vercel로의 직접 배포 워크플로가 없다. 대신 Vercel Git 통합이 외부에서 main 브랜치 push를 직접 감지해 자체 워커에서 빌드 후 발행한다. 즉 `git push origin main` 한 줄이 곧 공개 production alias로의 배포다. SSO Deployment Protection은 꺼져 있어 production·preview alias가 모두 공개 접근이다.

검토되지 않은 카피·잘못된 버전 라벨·잘못된 응답 헤더 핀이 사람 리뷰 없이 통과하면, 그대로 공개 표면에 노출되고 회복에는 롤백 PR + Vercel 재빌드(수 분)가 든다. 게다가 자동화된 작업 게이트(활성 태스크 검사 훅)는 대상 브랜치를 알지 못하고, branch protection·required reviews 같은 GitHub 정책도 이 레포에 설정되어 있지 않다.

## 결정 (Decision)

이 레포에서 "push = production"임을 모든 변경 흐름에서 명시적으로 인지·게이트한다.

- 사용자가 명시적으로 "push 해 / 머지해"라고 지시한 경우에만 push·merge를 실행한다. 지시 없이 main으로 직접 commit하거나 push하지 않는다.
- main 변경은 PR 경유로만 수행하고, PR 본문에 production 영향 범위(hero / SEO / 응답 헤더·rewrites / 설치 스크립트 중 무엇이 바뀌는지)를 명시한다 — PR 가시성이 사람 게이트의 핵심이다.
- "CI가 통과하므로 안전"이라는 가정으로 머지를 자가 결정하지 않는다. CI는 빌드만 검증하고 배포를 막지 않으며, Vercel 빌드는 CI와 분리되어 CI가 red여도 진행될 수 있다.
- 머지 직후 공개 자산(예: 설치 스크립트 응답)이 정상인지 검증하고 결과를 사용자에게 보고한다.

## 근거와 결과 (Consequences)

- 자동 배포를 막는 기술 장치(branch protection·required reviews)가 없고 CI는 배포 게이트가 아니므로, 사람 리뷰가 유일한 backstop이다. 이 사실을 명문화해 "CI 통과=안전"이라는 오해로 인한 자가 머지를 차단한다.
- 글로벌 가드레일과의 관계: 글로벌 작업 게이트 훅은 활성 태스크 유무만 검사할 뿐 대상 브랜치의 production blast radius를 표현하지 못한다. 글로벌 commit 룰("명시 지시 없이 커밋/푸시 안 함")은 모든 레포 공통의 일반 룰이라, "이 레포에서는 push가 곧 production"이라는 추가 가중을 담지 못한다. 그래서 이 결정이 레포 특화로 명문화된다.
- production hotfix 명목으로도 응답 헤더·설치 스크립트·hero 버전 라벨을 즉시 변경 후 push하지 않는다 — 사용자 승인이 우선이다.
