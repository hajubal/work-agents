# 참고 사례 조사 — 유사 에이전트 운용 사례와 근거 데이터

> Phase 0 보조 산출물 (2026-08-12, 웹 조사 3갈래 병렬 수행). 이 문서는 charter.md의 현실성 평가와 design.md의 설계 결정을 외부 사례·연구로 뒷받침/반박하는 근거 모음이다. 자가 보고 수치와 벤더 홍보 자료는 표기해 두었다.

## 1. 유사 운용 사례 (기업·팀·1인)

| 사례 | 핵심 사실 | 우리에게 주는 시사점 |
|---|---|---|
| [Deep Market Researcher AI Agent — The Product Compass](https://www.productcompass.pm/p/deep-market-researcher-ai-agent) (2025-02) | PM이 계획자 1 + 병렬 리서처 최대 12로 시장조사 자동화. 웹 60곳·30초·22p PDF. 자체 기록한 실패: **비주류 도메인에서 답 대신 방법론만 반환** | "계획자+병렬 리서처+종합" 골격 검증. 우리 도메인(국내 폐쇄망 보안)은 니치라 품질 급락을 전제해야 — "미확인" 허용 규칙이 필수인 이유 |
| [AI 경쟁 모니터링 구축기 — Nate Automates](https://nateautomates.com/blog/ai-agent-competitive-intelligence/) (2026-07, 자가 보고) | 경쟁사 5곳을 가격·출시·콘텐츠·채용·소셜 5개 신호로 주간 모니터링. 구축 6h, 유지 월 30분, 모니터링 주 2~3h→15분 | 신호 카테고리 목록(특히 **채용 공고·가격 변경**)을 researcher 브리프에 차용할 것. "수집·요약=에이전트, 해석·대응=사람" 분업 동일 |
| [PM 서브에이전트 사례 — Medium](https://medium.com/all-about-claude/claude-subagents-for-claude-subagents-for-product-managers-how-i-processed-a-quarter-of-research-in-47-minutes-5fa1ecdb67b9) (2026-03) | 병렬 에이전트 3개로 인터뷰 23건+NPS 847건 분석, 2일→47분. 산출물을 명시적으로 "1차 초안"으로 규정 | 산출물의 "초안" 규정이 품질 관리의 출발점 — 우리 issue-drafts/pending/ 개념과 일치 |
| [Polsia 1인 유니콘 — Fortune](https://fortune.com/2026/03/26/the-one-person-unicorn-myth-miracle-future-of-startups-polsia/) (2026-03) / [Rest of World 실사](https://restofworld.org/2026/ai-agent-china-one-person-company/) (2026-04) | "야간 자율 실행 + 아침 요약" 리듬으로 1인 운영. 단 실사 결과: 기본 업무 실패, 에이전트 무단 행동, 창업자의 정보 비대칭 호소 | 비동기 배치 + 검토 창구 단일화 리듬은 이식 가치. **"완전 자율"은 실증적으로 과장** — run-to-gate까지만 무인화하는 우리 결정을 지지 |
| 상용 CI 도구 경계 — [Klue](https://klue.com/blog/how-to-do-competitive-analysis-with-ai) (2025-07, 벤더 자료) / [IndustryLens 비교](https://industry-lens.com/intelligence/competitive-intelligence) (2026-08, 경쟁 벤더라 편향 유의) | Klue·Crayon·AlphaSense·Contify 모두 자동화는 **수집·요약까지**. 큐레이션·배틀카드·해석은 전담 인력 전제, "오너 없으면 셸프웨어" 명시. Klue 스스로 "가격·기능 세부는 그럴듯하게 완전 날조될 수 있다" 경고 | 수억 원짜리 상용 도구도 해석은 자동화 못 했다 — 우리 파이프라인의 자동화 경계(4-decision.md는 사람)가 업계 표준과 일치. 가격·기능 세부의 원문 대조 검증은 벤더도 포기 못 한 필수 단계 |

## 2. 같은 메커니즘(Claude Code) 운용 사례·공식 자료

| 자료 | 핵심 사실 | 시사점 |
|---|---|---|
| [How Anthropic teams use Claude Code](https://claude.com/blog/how-anthropic-teams-use-claude-code) (2025-07) | 그로스 마케팅팀(비개발)이 서브에이전트 2개+파일 입출력으로 광고 운영 자동화("수 시간→수 분") | "서브에이전트 소수 + 파일 인터페이스" 최소 구성이 Anthropic 내부 비개발팀에서 실전 검증됨 |
| [Anthropic multi-agent research system](https://www.anthropic.com/engineering/built-multi-agent-research-system) (2025-06) | 멀티에이전트 리서치가 단일 에이전트 대비 평가 +90.2%, 단 **토큰 ~15×**. 에이전트 정의에 **작업 규모별 노력 상한 명문화**(단순=툴콜 3~10회 등)가 핵심 품질 관리. 산출물은 파일로 저장하고 참조만 반환 권장 | 우리 "검색 ≤25회·분량 상한 하드코딩"과 "파일이 인터페이스" 설계가 공식 방법론과 일치 |
| [Introducing routines in Claude Code](https://claude.com/blog/introducing-routines-in-claude-code) (2026-04) | 루틴 = cron/API/GitHub 이벤트로 클라우드 실행. 일일 한도 Pro 5 / Max 15 / Team·Enterprise 25회. **매회 제로 컨텍스트로 깨어나므로 상태는 repo 파일로 전달** | Phase 3 루틴 전환의 공식 근거. "파일 존재=상태" 설계가 루틴의 전제 조건과 정확히 일치 |
| [Claude Code GitHub Actions — schedule 실행](https://code.claude.com/docs/en/github-actions) | `on: schedule` cron + `--allowedTools` 화이트리스트로 정기 리포트/이슈 파이프라인 예시가 공식 문서에 존재. `--max-turns`·타임아웃이 공식 비용 가드레일 | 루틴의 대안 경로. 단 우리 CI는 셀프호스트 러너 12대 공용이라 API 키·네트워크 검토 필요 — Phase 3에서 루틴과 비교 |
| [The Subagent Tax — Systima](https://systima.ai/blog/subagent-tax) (2026-07, 105회 실측) | 팬아웃은 순차 대비 2.6~5.9× 토큰이고 **더 느림**. 서브에이전트에 저가 모델 고정 시 토큰 -37%·시간 절반 | 진짜 독립적인 단계(글로벌/국내 조사)만 병렬화. 기계적 단계(issue-writer)는 저가 모델 고정 옵션 검토 |
| [ICM: Folder Structure as Agent Architecture — arXiv:2603.16021](https://arxiv.org/html/2603.16021v2) (2026-03) | 프레임워크 없이 **번호 매긴 스테이지 폴더 + 스테이지별 입출력 명세 파일**로 멀티에이전트 오케스트레이션, 사람이 스테이지 사이에서 파일 편집. 통제 실험은 없음(저자 명시) | 우리 파일 기반 그래프와 사실상 동일한 설계의 문서화된 선례 |
| [VoltAgent/awesome-claude-code-subagents](https://github.com/VoltAgent/awesome-claude-code-subagents) (24.2k★) | 서브에이전트 154+개 중 Business & Product 16개, Research & Analysis 11개(market-researcher, competitive-analyst 등) | Phase 1에서 에이전트 정의 작성 시 역할 프롬프트 참고 원본 |

## 3. 설계 가정의 검증 — 지지와 반박

### critic(비관 검토) 에이전트

- **지지**: [Multiagent Debate — Du et al.](https://arxiv.org/abs/2305.14325) (ICML 2024) 외부 비판이 사실성 향상. [Liang et al.](https://arxiv.org/abs/2305.19118) (EMNLP 2024) "Degeneration-of-Thought" — 모델은 확신을 굳히면 자기반성으로 못 깬다(**컨텍스트 격리 근거**). [CriticGPT — OpenAI](https://arxiv.org/abs/2407.00215) (2024-06) 전용 critic이 사람보다 버그를 더 찾고, 인간+critic이 인간 단독보다 60%+ 우수. [ICLR 2024 — Huang et al.](https://arxiv.org/abs/2310.01798) 외부 피드백 없는 자기교정은 오히려 악화.
- **반박**: [Stop Overvaluing Multi-Agent Debate](https://arxiv.org/abs/2502.08788) (2025-02) 동일 모델 debate는 연산만 쓰고 단순 기법을 못 이기는 경우 빈번 — **유일하게 일관된 개선책은 모델 이종성**. [Self-Preference Bias](https://arxiv.org/abs/2410.21819) (NeurIPS 2024 ws) 같은 모델은 자기 스타일 출력에 관대 — 컨텍스트를 분리해도 같은 모델이면 편향이 남는다. [MAST — UC Berkeley](https://arxiv.org/abs/2503.13657) (2025-03) 검증자가 있어도 **부실 검증(superficial verification)이 흔한 실패 모드** — critic의 존재가 아니라 검증 기준 명세가 성패를 가름.
- **역방향 위험**: [Sycophancy — Anthropic](https://arxiv.org/abs/2310.13548) (ICLR 2024) 반박 한 마디에 옳은 답을 98%까지 철회 — critic 반박을 planner에 자동 반영하면 옳은 기획이 죽는다. **우리 흐름(critic 출력은 planner에게 되돌리지 않고 사람이 심판)이 이 위험을 구조적으로 회피.**

### 조사 품질·환각

- [Cited but Not Verified](https://arxiv.org/abs/2605.06635) (2026-05): 프런티어 모델도 링크 생존율 94%+지만 **인용문-본문 일치는 39~77%**, 툴콜 2→150회로 늘면 정확도 ~42% 하락. → 링크 클릭 확인은 검증이 아니다. critic의 표본 검증은 **인용-본문 일치** 기준이어야 하고, 검색 상한(≤25회)은 비용이 아니라 품질 장치다.
- [Deloitte 호주 정부 보고서 환불](https://fortune.com/2025/10/07/deloitte-ai-australia-government-report-hallucinations-technology-290000-refund) (2025-10): 날조 인용이 고객 전달 후 외부인에게 발각 — 환각의 최악 경로. [참고문헌 검증 연구](https://arxiv.org/pdf/2505.18059) (2025-05): 챗봇 8종 중 6종이 참고문헌 39.8% 날조.
- [Deep Research Bench — FutureSearch](https://arxiv.org/abs/2506.06287) (2025-06): 상용 딥리서치 제품의 주요 실패 축 = 환각·툴 오류·장기 트레이스 망각.

### 사람 게이트·검토 부담

- [Workslop — HBR/BetterUp·Stanford](https://hbr.org/2025/09/ai-generated-workslop-is-destroying-productivity) (2025-09): 근로자 41%가 "그럴듯하지만 실질 없는" AI 산출물 수신, **건당 재작업 ~2시간** — 우리 최대 리스크("안 읽히는 보고서")의 정량 실증. 성공 지표를 산출량이 아니라 "검토 시간 대비 채택률"로 잡은 것이 정확히 이 근거와 일치.
- [Human+AI 메타분석 — Nature Human Behaviour](https://www.nature.com/articles/s41562-024-02024-1) (2024-10): 106개 실험 종합 — 인간+AI 조합은 판단 과제에서 평균적으로 최고 단독 성능보다 **나쁨**(g=−0.23). → 사람 게이트는 품질 향상 장치가 아니라 **저가역성 지점의 책임·중단 장치**로 설계해야 한다(이슈 생성 직전 = 정확히 그 지점).
- [Automation bias 리뷰](https://dl.acm.org/doi/10.1007/s00146-025-02422-7) (2025): 정확한 시스템을 반복 접한 검토자는 검증을 멈추고 기본 승인(rubber-stamping)으로 표류. → 4-decision.md 양식에 critique의 미해결 반론을 후보 옆에 강제 병기해야 게이트가 형식화되지 않는다.

### 멀티에이전트 구조 일반

- [Don't Build Multi-Agents — Cognition](https://cognition.com/blog/dont-build-multi-agents) (2025-06) + [후속 완화](https://cognition.com/blog/multi-agents-working) (2026): 실전에서 동작하는 유일한 패턴은 "**읽기(조사·비평)는 병렬, 쓰기(결정)는 단일 스레드**" — 우리 구조(researcher 병렬, planner 단독 쓰기, 사람 단독 결정)와 정확히 일치. 단 critic에 요약본이 아닌 **원본 전체**를 넘겨야 한다(우리는 파일 원문 전달이므로 충족).
- 오류 누적 산술: 단계당 95% 신뢰 × 10단계 = 60%. 단계 수 최소화 압력 — 우리 파이프라인은 4단계로 짧은 편.

## 4. 설계에 반영한 조정 (design.md에 반영됨)

1. **critic의 출처 검증 기준을 "인용-본문 일치"로 명시** — 링크 생존 확인은 검증이 아님 (arXiv:2605.06635)
2. **critic 이종 모델 옵션을 Phase 1 검토 항목으로 추가** — 동일 모델 critic은 컨텍스트를 분리해도 self-preference가 남음 (arXiv:2502.08788, 2410.21819)
3. **4-decision.md 스텁에 critique 미해결 반론을 후보 옆 강제 병기** — rubber-stamping 방지 (automation bias 연구)

반영하지 않은 것: critic→planner 재반영 루프(sycophancy 위험, 사람 심판 구조가 이미 회피), 단계 추가(오류 누적·검토 부담 증가), 전면 무인화(Polsia 실사 사례).

## 변경 이력

| 날짜 | 내용 |
|---|---|
| 2026-08-12 | 최초 작성 — 웹 조사 3갈래(운용 사례 / Claude Code 메커니즘 / 설계 가정 검증) 종합 |
