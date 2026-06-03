---
id: invariant.no-remote-telemetry
kind: Invariant
title: 원격 텔레메트리를 일절 기록·전송하지 않는다
definition: Clawket은 분석·통계·사용량 등 어떤 원격 텔레메트리도 수집·전송하지 않는다. 유일하게 남기는 관측 데이터는 로컬 감사 추적(audit_log)뿐이며, 그 항목은 어느 것도 사용자 기계 밖으로 나가지 않는다.
governs:
  - domain.governance-audit
  - concept.audit-log
  - component.daemon
  - component.sqlite-storage
implementedIn:
  - clawket/PRIVACY.md
  - clawket/README.md
decidedBy:
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: 2026-06-04
tags:
  - status:active
  - type:invariant
---

<!-- 작성 고도: 비개발자도 읽는 자연어. 식별자는 provenance/근거에만. -->

## 제약

Clawket은 사용 행태를 외부로 보고하는 텔레메트리·애널리틱스·집계 전송을 전혀 하지 않는다. 어떤 종류의 클라우드 통신도, 사용량 핑도 없다. 시스템이 관측 목적으로 남기는 유일한 기록은 로컬 감사 추적이다. 이 추적은 엔티티가 누구에 의해 어떻게 바뀌었는지를 변조 탐지가 가능한 형태로 로컬 데이터베이스에 적어 두는 것이며, 그 어떤 항목도 사용자의 기계를 떠나지 않는다.

다시 말해 "관측은 하되, 그 관측 데이터는 전적으로 로컬에 머문다"가 불변식이다.

## 깨지면 무슨 일이 일어나나

이 제약이 깨져 어떤 사용량·에러·행태 데이터라도 원격으로 전송되기 시작하면, 사용자가 무엇을 작업하고 어떤 결정을 내렸는지의 메타데이터가 동의 없이 외부로 흘러나간다. 로컬 우선(local-first) 불변식과 함께 Clawket의 프라이버시 약속을 이루는 두 축 중 하나가 무너지는 것이며, 개인·사내 코드 작업 도구로서의 신뢰를 잃는다. 또한 감사 추적이 외부로 나간다면 그 자체가 기밀 변경 이력의 유출이 된다.

## 코드에서 어떻게 강제되나

프라이버시 정책과 README의 텔레메트리 절이 "원격 텔레메트리 없음 / 외부 서버와 데이터 공유 없음"을 명시적 약속으로 선언한다. 구현상으로는 데몬이 외부로 향하는 네트워크 요청 경로를 갖지 않고(로컬 우선 불변식과 동일한 바인딩·온디바이스 처리 형상), 관측 데이터는 로컬 감사 추적 테이블에만 적힌다. 과거 기록을 보려면 사용자가 직접 로컬 데이터베이스를 조회해야 하며, 이 추적은 어떤 경로로도 전송되지 않는다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(이 불변식을 도입한 결정 근거) 연결 필요
