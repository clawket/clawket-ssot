---
id: concept.execution-envelope
kind: Concept
title: 실행 봉투(19-field execution envelope)
definition: 리프 태스크 하나하나에 서명되는 실행 계약. 무엇을 왜 바꾸는지(의도), 어떤 저장소를 건드려도 되는지, 성공을 어떻게 판정하는지, 더 잘게 쪼갤지 말지를 한 묶음으로 못 박아, "쪼개고(decompose) → 계약하고(contract) → 실행한다(execute)" 순서를 강제하는 장치다. 동의어: task contract, 태스크 계약, envelope.
relatesTo:
  - to: concept.task
    type: belongs-to
    note: 봉투는 태스크에 종속된 부속물이며 태스크당 버전 관리되는 계약으로 첨부된다.
  - to: concept.run
    type: relates-to
    note: 태스크 실행이 시작되면 그 시점의 봉투가 스냅샷으로 얼려져 실행 기록에 보존된다.
  - to: capability.decompose-task
    type: relates-to
    note: 봉투의 분해 정책(쪼갬 한도)이 하위 태스크 트리를 어디까지 만들지 결정한다.
governedBy: []
governs: []
dependsOn: []
relatesTo: []
implementedIn:
  - daemon/schemas/envelope-v1.schema.json
  - daemon/src/envelope/mod.rs
  - daemon/src/envelope/sign.rs
  - daemon/src/envelope/validate.rs
  - daemon/src/models.rs
  - clawket/README.md
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-03
tags: [status:active, "type:concept"]
---

## 정의

실행 봉투는 작업 하나를 "어떻게 끝내야 하는지"를 사전에 못 박는 계약서다. Clawket의 기여 흐름은 항상 세 단계를 순서대로 거친다 — 일을 태스크 트리로 **쪼개고(decompose)**, 트리의 말단(리프) 태스크마다 **계약을 서명하고(contract)**, 그 계약에 맞춰 **실행한다(execute)**. 봉투는 이 가운데 두 번째 단계의 산출물이며, 세 번째 단계(실제 코드 변경)는 봉투가 갖춰지지 않으면 시작 자체가 막힌다. 건너뛸 수 있는 우회 플래그는 없고, 막혔을 때의 올바른 대응은 "되돌아가 계약을 마저 채우는 것"이다.

봉투가 담는 핵심 약속은 다음과 같다.

- **의도(intent)**: 이 태스크가 무엇을 왜 바꾸는지 한 문단으로 적은 선언. 비어 있으면 안 된다.
- **대상 저장소(target_repo)**: 이 태스크가 변경해도 되는 저장소(또는 로컬 경로). 작업 범위를 명시적으로 가둔다.
- **성공 기준(success_criteria)**: 확인 가능한 결과들의 목록. 각 항목은 링크·명령·단언처럼 검증 가능한 형태여야 하며 최소 1개는 있어야 한다.
- **검증 명령(verification_cmd)**: 성공을 자동 판정하는 단일 셸 명령. 종료 코드 0이면 성공으로 보고, 출력은 실행 기록에 첨부된다.
- **분해 정책(decomposition_policy)**: 이 태스크를 더 쪼갤지 말지의 규칙. "더는 쪼개지 않음", "트리 형태로 깊이 N까지", "단계적으로 N스텝까지" 같은 형태로 표현한다.
- **컨텍스트 참조(context_refs)**: 에이전트가 참고해도 되는 다른 태스크·산출물·결정 등에 대한 안정적 참조 목록.

이 위에 선택적으로 더 정교한 약속들을 얹을 수 있다 — 게이팅에 쓰는 수용 기준(acceptance_criteria), 실행 전·후 조건, 표준 롤백 절차, 선언적 모델 등급, 재시도 정책, 비밀값 조회 순서, 프로젝트 간 RAG 허용 여부(기본 꺼짐), 깨지기 쉬운(flaky) 테스트 격리 임계치 등. 이렇게 모인 약속의 묶음을 제품 문서는 "19-field 실행 봉투"라고 부른다.

자식 태스크는 부모의 봉투를 물려받는다. 자식은 바꾸고 싶은 부분만 담은 얇은 차분(delta)을 가질 수 있고, 나머지 약속은 부모 체인에서 채워진다. 검증과 보존은 항상 이렇게 "합성된 최종 봉투" 기준으로 이뤄진다.

## 엔티티 (DB)

봉투는 태스크에 버전 단위로 첨부되는 별도 레코드로 보존된다. 한 레코드는 어느 태스크의 봉투인지, 몇 번째 버전인지, 봉투 본문(JSON), 서명 시각과 서명 주체, 그리고 이 봉투를 대체한 새 봉투가 있다면 그 참조를 담는다. 새 버전을 서명할 때마다 이전 버전을 덮어쓰지 않고 새 버전으로 쌓아 계약 이력이 남는다.

봉투 본문의 형상은 스키마(envelope-v1.schema.json)로 정의된다. 다만 "반드시 채워져야 통과하는 필수 칸"은 두 층위가 있다. 스키마가 정한 필수 항목과, 서명 시점의 런타임 검증이 실제로 막는 항목이 다르다 — 후자는 의도·성공 기준에 더해 프롬프트 템플릿(prompt_template)을 비어 있지 않은 값으로 요구한다. 권장이지만 누락해도 막지는 않는 항목(예: 대상 모델, 최대 턴 수)은 경고(advisory)로만 표시된다. 이 "막음/경고" 구분의 정확한 경계는 OPEN으로 둔다(아래 참조).

## API 표면

봉투를 새로 서명하거나 갱신하는 모든 경로 — 태스크 생성, 태스크 수정, 하위 태스크 생성, 플랜 가져오기 — 는 단일 서명 관문(envelope::sign)을 반드시 통과한다. 이 관문은 부모 체인을 합성한 "최종 봉투"를 만든 뒤 검증하고, 통과하면 새 버전으로 저장한다. 검증에서 오류(Severity::Error)가 하나라도 나오면 저장을 중단하고 위반 목록을 돌려준다.

웹 대시보드는 별도 검증 엔드포인트(태스크 봉투 validate)로 같은 검증기를 호출해 폼에 경고를 띄운다. 서명 경로와 검증 경로가 같은 검증기를 쓰도록 강제하는 이유는, 만약 둘이 어긋나면 화면이 "데몬이 실제로는 강제하지 않는 계약"을 사용자에게 안내하는 거짓 UI가 되기 때문이다.

## 불변식

- 코드 변경(execute) 단계는 쪼갬·계약(decompose·contract)이 끝나지 않으면 PreToolUse 단계에서 하드 블록된다. 건너뛰는 플래그는 없다.
- 의도·프롬프트 템플릿·성공 기준은 비어 있으면 서명이 거부된다(런타임 검증 기준).
- 검증은 항상 부모 체인을 합성한 최종 봉투에 대해 수행된다 — 자식의 얇은 차분만 보고 판정하지 않는다.
- 봉투를 만드는 모든 경로는 동일한 단일 서명 관문을 통과해야 하며, 검증 오류가 있으면 태스크와 봉투가 한 트랜잭션 안에서 함께 거부되어 고아 태스크가 남지 않는다.

## 구현 위치 (provenance)

- `daemon/schemas/envelope-v1.schema.json` — 봉투 본문의 형상(필드·제약)을 정의하는 스키마.
- `daemon/src/envelope/mod.rs`, `daemon/src/envelope/sign.rs`, `daemon/src/envelope/validate.rs` — 서명 관문, 부모 체인 합성, 검증 로직.
- `daemon/src/models.rs` — 봉투 레코드(태스크 참조·버전·본문·서명 정보) 구조.
- `clawket/README.md` — "쪼개고·계약하고·실행한다"는 세 단계 흐름과 19-field 봉투 명칭, PreToolUse 하드 블록 정책.

## 미확정 (OPEN)
- [ ] OPEN: "19개 필드"라는 명칭과 실제 스키마 필드 수, 그리고 스키마 필수 항목과 런타임 검증 필수 항목(의도·프롬프트 템플릿·성공 기준)의 차이가 의도된 것인지 확정 필요.
- [ ] OPEN: 경고(advisory)로만 처리되는 항목과 서명을 막는 오류 항목의 정확한 경계.
