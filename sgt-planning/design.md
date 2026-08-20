# 기획 에이전트 시스템 설계

> Phase 0 산출물. 왜 만드는지는 [charter.md](charter.md) 참조. 이 문서는 Phase 1 구현의 명세다.

## 아키텍처 요약

별도 프레임워크 없이 Claude Code의 커스텀 에이전트(`.claude/agents/`)와 스킬(`.claude/skills/`)로 구성한다. 오케스트레이션은 `/plan-cycle` 스킬이 담당하고, **단계 간 인터페이스는 전부 파일**이다 — 각 에이전트는 파일을 읽고 파일을 쓰고 끝난다. 파일 존재 자체가 상태이므로 별도 상태 관리가 없고, 세션이 끊겨도 `ls`만으로 재개된다.

```
0-brief.md ──▶ market-researcher ×2 ──▶ 1-research-{global,domestic}.md
                                            │
roadmap.md, matrix, inventory ──▶ product-planner ──▶ 2-plan.md (+matrix 갱신)
                                            │
                          red-team-critic ──▶ 3-critique.md   (planner와 컨텍스트 비공유)
                                            │
                        [사람: 4-decision.md] ──▶ issue-writer ──▶ issue-drafts/pending/
                                                                       │
                                                  [사람: gh issue create] ──▶ shipped/
```

## 에이전트 roster (4개)

형식은 유일한 선례인 `.claude/agents/sgt-perf-aws.md`를 따른다 (frontmatter: name/description/tools/model:inherit, 본문: 🔴 최우선 규칙 → 입력 → 고정 사실 → 단계 → 최종 보고 → 실패 시).

| 에이전트 | 책임 | 입력 (읽기) | 출력 (쓰기) | tools |
|---|---|---|---|---|
| `market-researcher` | 웹 조사. **브리프 파라미터로 글로벌/국내 겸용** — 정의 파일 1개를 브리프 2종으로 2회 병렬 호출 | 0-brief.md, competitor-matrix.md, sgt-feature-inventory.md | `cycles/<id>/1-research-{global,domestic}.md` | WebSearch, WebFetch, Read, Write |
| `product-planner` | 조사 종합 → 로드맵 델타 제안 + 기능 후보 순위(≤7개) + 매트릭스 갱신 | research 2건, roadmap.md, inventory, `gh issue list --repo ininext/sgt` 최근분, 직전 4-decision.md | `cycles/<id>/2-plan.md`, competitor-matrix.md | Read, Write, Bash |
| `red-team-critic` | 비관 검토: 2인 capacity·폐쇄망·설치형 제약 위반 기각, 기존 이슈 중복 검사, 출처 표본 재검증(**링크 생존이 아니라 인용-본문 일치** 확인 — [references.md](references.md) §3), 시장성 반박 | 2-plan.md, research 원본, roadmap.md — **파일 경로만, planner 대화 컨텍스트 절대 비공유** | `cycles/<id>/3-critique.md` | Read, Bash, WebSearch |
| `issue-writer` | 4-decision.md에서 승인된 후보만 이슈 초안화. PRD 축약형(`docs/temp/PRD_MCP_보안_게이트웨이.md` 구조 기반), 제목은 `feat(scope):` 관례 | 4-decision.md, 2-plan.md | `issue-drafts/pending/NNN-<slug>.md` | **Read, Write만** |

### 설계 결정과 이유

- **competitor-analyst는 별도로 두지 않는다.** 시장 조사와 경쟁사 조사는 본질이 같은 "웹 조사 + 정리"이고 차이는 브리프뿐이다. 정의 파일 2개 유지보수보다 1개를 2회 호출하는 게 싸다.
- **SGT 저장소를 읽는 에이전트도 두지 않는다.** 매 사이클 저장소 전체 재독은 토큰 낭비다. SGT 기능 스냅샷은 living doc(`sgt-feature-inventory.md`)으로 유지하고, /plan-cycle 0단계에서 메인 세션이 `git log` 증분으로 갱신한다.
- **red-team-critic의 컨텍스트 격리** (파일 경로만 입력으로 받는 별도 Agent 호출) 이유 3가지:
  1. LLM은 자기 컨텍스트 안의 결론에 자기일관성 편향을 갖는다 — planner의 추론 과정을 본 critic은 그 프레임 안에서만 반박한다
  2. critic이 파일만 보고 계획을 이해하지 못하면 사람도 못 한다 — **파일 단독 이해 가능성의 강제 검증기**를 겸한다
  3. critic은 2-plan.md 주장 상위 N개를 WebSearch로 재검증하는데, 같은 세션이면 planner가 본 검색 결과를 재사용해 검증이 동어반복이 된다
  - 반박 기준은 critic .md의 "고정 사실" 절에 하드코딩한다: 개발 2인(owasp 1 + 나머지 전부 1), 폐쇄망(외부 API 의존 기능 불가), 설치형(SaaS식 상시 업데이트 불가)
  - critic 출력은 planner에게 되돌리지 않고 **사람이 심판**한다 — 반박에 옳은 결론이 자동 철회되는 sycophancy 위험 회피 ([references.md](references.md) §3)
  - Phase 1 검토 항목: critic을 **이종 모델**로 돌리는 옵션 — 같은 모델은 컨텍스트를 분리해도 self-preference 편향이 남는다는 근거가 있다 ([references.md](references.md) §3)
- **issue-writer는 planner에 합치지 않는다.** 독립성이 아니라 **게이트 순서** 때문이다: 승인 전에 초안을 쓰면 기각될 후보에 토큰을 쓰고, 사람이 "초안이 이미 있으니 승인하자"는 앵커링에 걸린다. 4-decision.md 이후에만 실행된다.
- **issue-writer에서 Bash를 뺀 것이 "이슈 생성은 사람만" 원칙의 실제 안전장치다.** `settings.local.json`이 `Bash(gh issue *)`를 이미 허용하므로 권한 시스템은 이를 막아주지 않는다 — 유일하게 이슈를 써야 할 에이전트에서 도구를 빼는 것이 프롬프트 지시보다 확실하다. (중복 검사용 `gh issue list --repo ininext/sgt` 읽기는 critic 담당.)

## 문체 계약

에이전트 4종은 산출 파일을 쓴 **뒤** `im-not-ai` 플러그인의 `humanize-korean` 스킬을 호출해 AI 티를 제거한다(각 에이전트 정의의 **규칙 0**, 이를 위해 `tools`에 `Skill` 추가). 메인 세션이 쓰는 파일(브리프·decision 스텁·로드맵 반영분)도 같은 계약을 진다(SKILL.md 머리말).

[writing-style.md](../writing-style.md)는 그 스킬을 **대체하는 게 아니라 태워 보내는 제약**이다 — 무엇을 보존할지(수치·날짜·URL·이슈 번호·인용문·출처 표기)와 무엇을 적용하지 말지를 정한다. 원본 룰북의 레이아웃 규칙(연속 불릿을 산문으로, 본문 볼드 제거, 숫자 인덱싱·콜론 헤딩 금지, 대시 분해)은 제외한다: 우리 산출물은 칼럼이 아니라 표·불릿·출처 링크가 계약인 기술 보고서다. 스킬이 없는 환경에서는 이 파일의 규칙만 자체 적용하고 그 사실을 보고에 밝힌다.

**윤문이 사실을 바꾸면 윤문을 되돌린다** — 문체보다 사실 보존이 우선이며, 이 순서는 산출물 신뢰의 근간이라 타협 대상이 아니다.

## 산출물 분량 상한 (하드코딩 — "안 읽히는 보고서" 방어)

| 산출물 | 상한 |
|---|---|
| 1-research-*.md | ≤150줄, 모든 사실 주장에 URL+확인일, 출처 등급("공식 문서"/"마케팅 주장") 구분, **"미확인" 명시 허용·권장(추측 금지)** |
| 2-plan.md | 기능 후보 ≤7개, **후보당 15~40줄** — 사람이 이 문서만 읽고 승인/기각할 수 있어야 하므로 개수는 줄이고 각 후보를 깊게 쓴다(문제·제품 형상·동작 시나리오·완료 판정·규모 근거·기존 이슈 흡수/대체/분리 판정). 40줄 초과는 후보가 아니라 설계 문서 |
| 3-critique.md | 반박 상위 5건 |
| 4-decision.md | 체크박스 양식 (에세이 아님) |

## /plan-cycle 스킬 실행 흐름

```
0. 준비(메인 세션): 직전 사이클 4-decision.md 존재 확인(없으면 시작 거부)
   → cycles/<id>/ 생성, 0-brief.md 초안 작성 → 사람이 대화 중 확정   [터치포인트 A, ~5분]
   → git log 증분으로 sgt-feature-inventory.md 갱신
1. 병렬: market-researcher ×2 (글로벌 브리프 / 국내 브리프)
2. 순차: product-planner → 2-plan.md + competitor-matrix.md 갱신
3. 순차: red-team-critic → 3-critique.md
   (1→3은 사람 개입 없이 연속 실행 — "run-to-gate")
4. 게이트: 사람이 2-plan.md + 3-critique.md를 읽고 4-decision.md 작성
   (스킬이 후보별 승인/보류/기각 체크박스 스텁을 미리 생성하되,
    critique의 미해결 반론을 각 후보 옆에 강제 병기 — rubber-stamping 방지)  [터치포인트 B, ~1시간]
5. 4-decision.md 감지 → 승인 델타를 roadmap.md에 반영(메인 세션, 기계적 작업)
   → issue-writer가 승인 후보만 pending/에 초안 작성
6. 사람이 초안 확인 → gh issue create --label product-planning → shipped/ 이동
   (터치포인트 B와 같은 세션에서 연속 처리 가능 — 사람 세션은 사이클당 실질 2회)
```

- **0-brief.md에는 사람이 최근 고객 요청·영업 이벤트 3줄을 필수로 넣는다.** 폐쇄망 제품이라 이것이 에이전트가 접할 수 있는 유일한 고객 ground truth다.
- **research 검토 게이트는 평시엔 없다.** 나쁜 조사 위에 planner+critic이 도는 낭비는 몇 달러지만 게이트 하나는 사람 30분+대기 하루다. 파일럿에서만 1→2 사이에 품질 보정용 검토를 1회 넣고 평시엔 제거한다 (critic이 조사 품질 문제를 어차피 잡는다).
- **재개**: 스킬 시작 시 최신 cycles/ 폴더를 `ls`해서 첫 번째로 없는 파일부터 실행한다. 4-decision.md 부재 = 사람 대기 상태.
- **주기 실행 판단은 Phase 3으로 미룬다.** 사람 게이트 2개가 있는 한 전체 무인 실행은 대기만 쌓고, 적정 주기는 파일럿 실측 없이 결정 근거가 없다. 선도 후보 메커니즘은 **Claude 루틴(스케줄 클라우드 에이전트)** — 월 1회 repo를 clone해 1~3단계(run-to-gate)만 실행하고 cycles/ 산출물을 PR로 push+알림, 4-decision.md 이후는 사람이 트리거. 전제: private repo의 GitHub 연동, 에이전트·스킬의 develop 머지(Phase 3 완료 후 자연 충족), 상시 brief 파일(무인 실행엔 고객 맥락 3줄 주입자가 없으므로 사람이 평소 유지). 4-decision.md 미결 시 루틴 실행은 no-op — 이 가드는 무인화에서 더 중요해진다.

## 파일 계약 (누가 무엇을 쓰는가)

| 파일 | 쓰기 권한 | 비고 |
|---|---|---|
| roadmap.md | planner 제안 → **사람 승인 후 메인 세션이 반영** | 원본 xlsx는 동결(머리말에 명시) |
| competitor-matrix.md | planner 단독 | 셀마다 출처가 cycles/ 회차 파일을 가리킴 |
| sgt-feature-inventory.md | 메인 세션 (git log 증분) | |
| cycles/<id>/* | 각 에이전트가 자기 산출물만 | researcher는 cycles/ 밖에 쓰지 않음 |
| 4-decision.md | **사람 전용** | |
| issue-drafts/pending/ | issue-writer | shipped/ 이동은 사람(gh 실행 후) |

## 폴더 구조

```

├── README.md                  # 유일한 상태 문서
├── charter.md / design.md / ops.md
├── roadmap.md                 # living
├── competitor-matrix.md       # living — 회차 누적 핵심 자산
├── sgt-feature-inventory.md   # living
├── cycles/<YYYY-MM[-이름]>/
│   ├── 0-brief.md → 1-research-global.md / 1-research-domestic.md
│   ├── 2-plan.md → 3-critique.md → 4-decision.md(사람) → 9-cost.md
│   └── retro.md (파일럿 한정)
└── issue-drafts/{pending,shipped}/
```

## Phase 1 구현 목록 (게이트 1 승인 후 착수)

1. `.claude/agents/` 4종: market-researcher.md, product-planner.md, red-team-critic.md, issue-writer.md
2. `.claude/skills/plan-cycle.skill/SKILL.md` — 4-decision.md 체크박스 양식 내장
3. `roadmap.md` — 확정 항목만 담는 빈 베이스라인 (최초 xlsx 이관본은 AI 임의 생성으로 판명되어 `idea-pool.md`로 강등 — 2026-08-12 리셋)
4. `sgt-feature-inventory.md` — 저장소 읽고 최초 생성 (사람 검수)
5. `competitor-matrix.md` — 열 스키마 뼈대만: 제품 / 배포모델 / 탐지기능 / 온프레미스 여부 / 가격신호 / 출처 / 확인일
6. `issue-drafts/{pending,shipped}/.gitkeep`
7. GitHub 라벨 `product-planning` 생성 — 성과 측정("에이전트발 이슈 중 실제 구현된 비율") 필터용
8. 별도 템플릿 파일 없음 — 이슈 초안 양식은 issue-writer.md 본문에, decision 양식은 SKILL.md에 내장 (양식과 사용처가 한 파일에 있어야 어긋나지 않는다)

**게이트 2 (Phase 1 완료 기준)**: market-researcher를 "검색 3회 제한" 스모크 브리프로 1회 드라이런 → 파일이 규약 위치에 생기는지 확인 → 커밋.

## 파일럿(Phase 2) 범위

- **글로벌 6개 고정**: Lakera Guard, Prompt Security(SentinelOne), CalypsoAI, Cisco AI Defense, NeMo Guardrails(OSS 기준선), AWS Bedrock Guardrails(기능 하한선). 6개 제한 이유: 파일럿의 목적은 완전한 지도가 아니라 **프로세스 검증 + 매트릭스 스키마 확정**이다. 나머지는 2회차 이후 증분 추가
- **국내는 "시드 + 발굴 임무"**: 시드(파수, 지란지교시큐리티, SK쉴더스, 안랩, 이글루코퍼레이션 — researcher가 실재·현황 필수 검증) + 발굴 질문("국내 공공/금융 폐쇄망에 LLM 보안 게이트웨이를 실제 납품한 업체는 누구인가, 나라장터 조달·CC/보안기능확인서 신호는 무엇인가")
- 산출: cycles/2026-08-pilot/ 전체 + 매트릭스 1차 채움 + 이슈 초안 ≥2건 + 실제 gh 이슈 ≥1건 + 9-cost.md
- 판정: [charter.md](charter.md)의 성공 지표 표

## 변경 이력

| 날짜 | 내용 |
|---|---|
| 2026-08-12 | 최초 작성 (Phase 0) |
