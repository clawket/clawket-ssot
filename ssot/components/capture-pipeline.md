---
id: component.capture-pipeline
kind: SystemComponent
title: Hero 데모 캡처 파이프라인 (scripts/capture)
definition: 실제 대시보드 6개 화면을 Playwright 로 녹화하고 ffmpeg 로 트랜스코딩해 홈 화면용 hero 데모 영상을 만드는, 빌드와 분리된 데모 생성 파이프라인이다.
purpose: 가짜 목업이 아닌 실제 제품 화면을, 특히 "증거 없이는 완료 불가" 클라이맥스를 12~14초 영상으로 보여 주어 랜딩의 설득력을 높인다.
realizedBy:
  - capability.watch-demo
implementedIn:
  - landing/scripts/capture/capture.mjs
  - landing/scripts/capture/seed-demo-data.sh
  - landing/scripts/capture/capture-scenario.md
dependsOn:
  - component.web-dashboard
  - component.cli
consumesApi: []
providesApi: []
integratesWith:
  - integration.playwright
  - integration.clawket-cli-seed
impacts:
  - screen.home
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags: [status:active, "type:systemcomponent"]
relatesTo:
  - to: concept.demo-seed-chain
    type: reads
    note: 녹화 전에 CLI 로 데모 프로젝트와 보드 카드를 미리 시딩해 hero 가 실제 워크스페이스처럼 보이게 한다.
  - to: concept.evidence-required
    type: reads
    note: 녹화 시나리오의 핵심 순간이 증거 없이 완료 전환 시 제출이 거부되는 장면이다.
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 코드 옮겨적기 금지. -->

## 책임

이 파이프라인은 홈 화면에 박히는 데모 영상을 만든다. 핵심 원칙은 "실제 제품을 그대로 녹화한다"이다. 따로 만든 애니메이션이나 목업이 아니라, 진짜로 도는 웹 대시보드를 헤드리스 브라우저가 자동으로 돌아다니며 화면을 찍는다.

녹화 동선은 정해진 시나리오를 따른다. 요약 화면에서 시작해 플랜 트리·백로그·타임라인·위키를 차례로 둘러본 뒤, 마지막에 보드와 태스크 상세로 들어가 클라이맥스를 연출한다. 클라이맥스는 태스크를 완료로 바꾸려 할 때 증거가 없으면 제출 버튼이 잠겨 거부되는 장면이고, 증거를 채워 넣자 제출이 활성화되어 완료로 넘어가는 흐름까지 보여 준다. 이 한 장면이 Clawket 의 핵심 약속을 시청자에게 각인시킨다.

녹화가 끝나면 원본 영상을 두고, 별도 인코딩 단계가 이를 웹용 포맷 두 가지(webm·mp4)와 포스터 정지 이미지로 변환한다. 최종 산출물만 사이트 정적 자산으로 커밋되고, 녹화 중간 산출물은 버전 관리에서 제외된다.

## 경계와 의존

이 파이프라인은 사이트 빌드의 일부가 아니다. 평소 빌드·배포 경로에는 들어가지 않고, 데모를 새로 찍어야 할 때만 수동으로 돌리는 보조 도구다. 빌드를 결정론적으로 유지하기 위해 일부러 분리되어 있다.

동작하려면 두 가지가 갖춰져야 한다. 첫째, 실제로 떠 있는 웹 대시보드와 데몬이 필요하다 — 그래야 녹화할 화면이 존재한다. 둘째, 녹화 직전에 CLI 로 데모용 프로젝트·플랜·유닛·사이클·태스크와 보드 카드들을 미리 채워 두어야 화면이 비어 보이지 않는다. 이 사전 시딩은 멱등하게 동작해 여러 번 돌려도 안전하다.

## 통신 패턴

헤드리스 브라우저가 로컬에서 떠 있는 대시보드 주소로 접속해 화면을 조작·녹화한다. 시딩과 화면 식별(프로젝트·태스크 찾기)은 CLI 명령을 호출해 처리한다. 외부 네트워크로 나가는 통신은 없다.

## 하위 서브패키지 (책임 단위)

세 부분이다. 데이터 시딩 스크립트(녹화 전 워크스페이스 상태 구성), 녹화 드라이버(시나리오대로 화면을 돌며 원본 영상 생성), 그리고 시나리오 문서(어떤 동선·타이밍으로 찍을지 정의)다. 원본 영상을 웹 포맷으로 바꾸는 인코딩은 후속 단계로 분리되어 있다.

## 미확정 (OPEN)
- [ ] OPEN: 인코딩 단계(encode.sh)의 자동화 수준 — 현재 수동 실행으로 추정.
