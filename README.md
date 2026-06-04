# clawket-ssot

**Clawket(LLM-네이티브 작업관리 + 로컬 RAG)의 단일 진실(SSOT) 데이터 레포.** 제품·도메인·시스템 맥락을 모듈형 노드 + 관계 엣지 그래프로 외부화한 콘텐츠 저장소다. 코드 레포(플러그인·Rust CLI/데몬·웹/데스크탑)와 분리된 중립 저장소이며, 코드에서 역추론한 자연어 정책·동작·데이터 문서를 담는다. 현재 confidence 는 대부분 `inferred`(코드 추론) — 담당자 검증으로 `high` 로 승급한다.

> 도구는 별도 레포다 — 작성 스킬·MCP 는 [`ssot-studio/ssot-plugin`](https://github.com/ssot-studio/ssot-plugin), 뷰어(웹)는 [`ssot-studio/ssot-web`](https://github.com/ssot-studio/ssot-web). 이 레포는 **데이터(콘텐츠)만** 담는다.

## 구조

```
ssot/
├── _catalog.json     ← 그래프 인덱스 (커밋 = 공유 인덱스. .md 에서 재생성 가능)
├── platform.md       ← Platform 노드
├── personas/ domains/ concepts/ capabilities/ components/
├── integrations/ invariants/ decisions/
├── screens/          ← 화면 (FE 라우트 단위)
└── endpoints/        ← API 엔드포인트
```

제품 전반을 커버하는 노드 그래프로 **Endpoint·Invariant·Capability 가 대다수**이며, 다수는 scaffold 빈 노드(채울 근간)로 점진 보강된다. 정확한 노드 수·kind 분포는 뷰어(웹) 또는 MCP `ssot_list_sources`·`ssot_list_tags` 로 확인한다.

## 노드 규약 (요약)

- **자기완결**: 본문이 원본. 코드가 사라져도 노드만으로 재현 가능.
- **4축 측면**: 비즈니스·제품 / 도메인 개념·관계·불변식 / 시스템·경계·영향 / 거버넌스·근거·생명주기.
- **lifecycle**: `planned`(기획됨·코드없음) / `active`(구현됨) / `deprecated`(폐기예정).
- **confidence**: `high`(사람 검증) / `inferred`(코드 추론) / `unverified`(빈칸).
- 전체 방법론: `ssot-studio/ssot-plugin` 의 `skills/ssot/reference/methodology.md`.

### tags (`namespace:value`)

각 노드는 `namespace:value` 형식의 통제 어휘 태그를 가진다(소문자 kebab). 뷰어(웹) 필터와 조회의 좌표축이다. 현재 채워진 네임스페이스:

| namespace | 적용 | 값 |
|-----------|------|-----|
| `type` | 전 노드 | kind 소문자 — 예: `type:endpoint`, `type:screen`, `type:concept` |
| `status` | 전 노드 | lifecycle 값 — `status:planned` / `status:active` / `status:deprecated` |
| `domain` | 도메인 귀속 노드 | 귀속 도메인 slug |

태그는 플러그인 스킬의 `auto-tag` 가 frontmatter 의 kind·lifecycle·domain 엣지에서 결정적으로 파생한다. `verify` 가 형식(namespace:value, 소문자 kebab)을 검증한다.

## 조회 (AI 질의 — MCP)

[`ssot-studio/ssot-plugin`](https://github.com/ssot-studio/ssot-plugin) 플러그인 설치 후, 로컬 `ssot-sources.json` 에 이 레포를 등록한다:

```jsonc
{
  "sources": [
    {
      "id": "clawket",
      "type": "git",
      "url": "https://github.com/clawket/clawket-ssot.git",
      "ssotPath": "ssot"
    }
    // 로컬 clone 을 쓰면:
    // { "id": "clawket", "type": "local-fs", "dir": "./ssot" }
  ]
}
```

등록 후에는 **평문 질문**만으로 MCP 가 자동 조회한다(스킬 호출 불필요). MCP 도구: `ssot_search`(태그 필터 포함) / `ssot_list_tags` / `ssot_get_node` / `ssot_impact` / `ssot_neighbors` / `ssot_gaps` / `ssot_flag` / `ssot_list_sources`.

## 편집 (제안 → 검토 → 승인)

데이터 변경은 **PR** 로 한다(직접 push 지양). 플러그인 스킬 `propose`(정합→PR / 충돌→이슈+draft / 근간→Decision / 범위외→거부) 가 제안을 생성하고, **사람이 검토·머지**한다. LLM(스킬·MCP)은 항상 보조 — 직접 머지하지 않는다.

## 활용

- **뷰어(웹)**: `main` push(`ssot/**` 변경) → GitHub Actions(`ubuntu-latest`)가 뷰어(`ssot-studio/ssot-web`)를 빌드 + 데이터 주입 → `peaceiris/actions-gh-pages` 가 `gh-pages` 브랜치로 push(인증은 기본 `GITHUB_TOKEN`, 별도 PAT 불필요). **단 현재 이 레포는 private + GitHub Pages 미활성** 이라 gh-pages 산출물이 공개 서빙되지는 않는다 — 서빙하려면 **Settings > Pages** 에서 `gh-pages` 를 소스로 지정해야 하며 private 레포 Pages 는 유료 플랜이 필요하다. 그 전까지 사용 경로는 **로컬 뷰어 또는 아래 MCP 조회**. 뷰어는 `namespace:value` 태그로 노드를 필터한다.
- **로컬 MCP**: 위 조회 config 로 등록 후 질의. git pull 이 갱신 채널(메모리 로드, sqlite 불필요).

## 검증 (로컬)

```bash
# ssot-studio/ssot-plugin 의 스크립트로
node <plugin>/skills/ssot/scripts/build-graph.mjs ./ssot
node <plugin>/skills/ssot/scripts/verify.mjs ./ssot 90 --root <clawket-코드-레포-루트>
```
