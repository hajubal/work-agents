# 경쟁사 기능 매트릭스 (living doc)

> 회차를 거듭하며 누적되는 핵심 자산. **쓰기: product-planner(와 사람)만.** researcher는 "매트릭스 갱신 제안"으로만 전달한다.
> 규칙: 모든 셀에 출처(등급·URL·확인일) — 출처 없는 값 금지. **확인 못 한 값은 `미확인`으로 두는 것이 정상이다** (그럴듯한 추정치가 최악의 오염). SGT 열은 [sgt-feature-inventory.md](sgt-feature-inventory.md)만을 근거로 한다.
> 출처 등급: `[공식]` 제품 문서 / `[보도]` 언론·애널리스트 / `[마케팅]` 벤더 홍보 / `[커뮤니티]` 포럼·블로그

## 비교 축 (열 정의)

| 열 | 의미 |
|---|---|
| 배포 모델 | SaaS / 하이브리드 / 온프레미스 / **에어갭(완전 폐쇄망) 설치 가능 여부** — "온프레미스 지원" 마케팅 문구와 에어갭 실설치는 다르다 |
| 탐지 기능 | prompt injection·jailbreak / PII / 독성 / 금칙어·custom rule / 파일 스캔 / 기타(환각 등) |
| 한국어 지원 | 한국어 전용 모델 여부 / 다국어 범용 여부 / 한국 개인정보 유형(주민번호 등) 대응 |
| 통합 표면 | REST / SDK 언어 / MCP / 프록시·게이트웨이 방식 |
| 가격 신호 | 공개 가격 / 견적 / 무료 티어 — 대부분 미확인일 것 |
| 국내 신호 | 국내 납품 실적 / 조달(나라장터) / CC인증·보안기능확인서 |
| 최종 확인일 | 이 행을 마지막으로 검증한 날짜 — 오래된 행이 다음 사이클의 갱신 우선순위 |

> ⚠️ **성능 수치 셀 취급 규칙**: 아래 표의 정확도 수치(Theori 89.0, 이로운 97.88%)는 **각 벤더의 자체 데이터셋·자체 과제 기준**이며 SGT의 strict micro F1 0.8531과 데이터셋·과제·판정 기준이 모두 달라 **동일 선상 비교 불가**다. 수치를 인용할 때는 반드시 괄호 안 측정 기준을 함께 옮길 것.

## 매트릭스

| 제품 | 배포 모델 | 탐지 기능 | 한국어 지원 | 통합 표면 | 가격 신호 | 국내 신호 | 출처 | 최종 확인일 |
|---|---|---|---|---|---|---|---|---|
| **SGT (자사)** | 온프레미스·에어갭 설치형 (SaaS 아님), 최소구성(gateway+owasp) 지원 | jailbreak/PII/독성/금칙어/파일 보안 | 한국어 전용 자체 모델(법령 문체·주민번호 등 KR regex), 한/영 자동 감지 | REST detect·MCP 서버·JS SDK(fetch 래핑)·OpenAPI·CLI | 라이선스 계약 | 국산 암호(SEED)·KeyFix 연동 | [inventory](sgt-feature-inventory.md) | 2026-08-12 |
| Lakera Guard (현 Check Point AI Guardrails) | `[공식]` SaaS + self-hosted(Helm/Docker), **오프라인 배포 공식 지원**("컨테이너를 인터넷 없이 export/load"). 단 **오프라인 라이선스 활성화 절차·모델 로컬 패키징은 미확인**(상세 문서가 고객 전용 포털) | `[공식]` Prompt Defense(injection·jailbreak) / Content Moderation 7카테고리 / DLP(PII·시스템프롬프트 유출) / Malicious Links / Agent Behavior(tool allow-deny) / Custom regex. **정책별 신뢰도 L1~L4**. 파일 스캔 미확인 | `[공식]` 다국어 범용(prompt attack 기준 100+ 언어, 한국어 포함). **한국어 전용 모델 아님**, KR 개인정보 유형 대응 미확인 | `[마케팅]` REST Guard API + 프로젝트별 정책. **MCP 서버 미제공**(고객 MCP 코드에 데코레이터로 API 호출) | `[커뮤니티·스니펫만]` 무료 Community 월 10,000 요청 / self-hosting은 Enterprise 문의 | 미확인 | [research-global](cycles/2026-08-pilot/1-research-global.md) §Q1~Q6 · docs.lakera.ai/docs/selfhosting · /docs/defenses · /docs/prompt-defense · lakera.ai/blog/how-to-secure-mcps-with-lakera-guard · eesel.ai/blog/lakera-pricing · 2026-08-12 | 2026-08-12 |
| Prompt Security (SentinelOne) | `[공식(보도자료)]` SaaS / 브라우저 확장 / **On-Premises 2026-03-23 발표**("fully disconnected environments"). **GA 여부·설치 방식·라이선스 검증 미확인** | 미확인 (공식 문서 확인 실패) | 미확인 | `[마케팅]` AI Gateway·MCP Gateway = 에이전트↔MCP 서버 **리버스 프록시**, 엔드포인트 에이전트, 브라우저 확장. **MCP 서버 제공 아님** | 미확인 | 미확인 | [research-global](cycles/2026-08-pilot/1-research-global.md) §Q1·Q3 · sentinelone.com/press/sentinelone-brings-ai-security-to-on-premise-regulated-sovereign-self-hosted-and-airgapped-environments/ · prompt.security/solutions/agentic-ai-security-and-governance · 2026-08-12 | 2026-08-12 |
| CalypsoAI (F5 AI Guardrails) | `[마케팅]` public cloud / private cloud / on-prem / **"fully air-gapped"** 4종 명시 — **제품 페이지 문구 수준, 설치·오프라인 절차 문서 미확인** | `[마케팅]` prompt injection·jailbreak·data exfiltration·PII 유출·content moderation(toxic·biased·inaccurate)·컴플라이언스 컨트롤(GDPR/HIPAA/EU AI Act/PCI/PHI)·agent tool call 감사, standard+custom categories. **카테고리별 threshold 공개 미확인** | 미확인 (F5 사이트의 한국어는 로케일일 뿐 제품 기능 아님) | 미확인 | 미확인 (견적 문의 방식) | 미확인 | [research-global](cycles/2026-08-pilot/1-research-global.md) §Q1·Q2·Q4·Q6 · f5.com/products/ai-guardrails · f5.com/company/news/press-releases(2025-09 인수, $180M) `[보도]` · 2026-08-12 | 2026-08-12 |
| Cisco AI Defense | `[공식·스니펫만]` 하이브리드 — **Cisco 호스팅 컨트롤 플레인** + 고객 데이터 플레인(SaaS/고객 VPC/온프렘 AI POD), K8s 네이티브. 관리 평면 메타데이터가 Cisco로 전송 → **완전 폐쇄망 여부 미확인** | 미확인 (데이터시트 HTTP 403, 2회 시도) | 미확인 | `[마케팅]` REST API·CI/CD 통합·**MCP Scanner(도구 설명·스키마 정적 스캔 — MCP 서버도 인라인 프록시도 아님)** | 미확인 | 미확인 | [research-global](cycles/2026-08-pilot/1-research-global.md) §Q1·Q3 · cisco.com/c/en/us/products/collateral/security/ai-defense/ai-defense-ds.html · aws.amazon.com/blogs/machine-learning/securing-ai-agents-how-aws-and-cisco-ai-defense-scale-mcp-and-a2a-deployments/ · 2026-08-12 | 2026-08-12 |
| NVIDIA NeMo Guardrails (OSS 기준선) | `[공식]` Apache 2.0 OSS — Python 라이브러리 / `nemoguardrails server` / Docker / K8s(NIM Operator·Helm). NIM·vLLM 로컬 서빙으로 자체 호스팅 가능하나 **공식 에어갭 설치 가이드 미확인** | `[공식]` input/dialog/retrieval/execution/output **5단계 레일**, jailbreak·injection, content safety(Nemotron/Llama Guard 3/ShieldGemma 선택), PII 엔진 플러그인(GLiNER·Presidio·Private AI 등), topical control(Colang) | `[공식]` Nemotron-3.5-Content-Safety가 한국어 포함 **12개 언어** 지원(한국어 전용 아님) | 미확인 | OSS | 미확인 | [research-global](cycles/2026-08-pilot/1-research-global.md) §Q1·Q2·Q4 · github.com/NVIDIA-NeMo/Guardrails · docs.nvidia.com/nemo/guardrails/configure-guardrails/guardrail-catalog/content-safety · huggingface.co/nvidia/Nemotron-3.5-Content-Safety · 2026-08-12 | 2026-08-12 |
| AWS Bedrock Guardrails (기능 하한선) | `[공식]` **AWS 관리형 클라우드 전용, self-host 옵션 없음 → 폐쇄망 적용 불가.** ApplyGuardrail API로 외부·자체호스팅 모델을 보호할 수는 있으나 가드레일 자체는 AWS에서 실행 | `[공식]` content filters(텍스트·이미지) / denied topics / word filters / sensitive information filters(PII 30+ 엔티티 + 커스텀 regex, Block·Mask·None) / contextual grounding / Automated Reasoning. 커스텀 regex는 lookaround 미지원 | `[공식]` 한국어 "Optimized and supported"(content filter·denied topics·PII). **단 word filters(금칙어)는 영/불/스 3개 언어만 = 한국어 미지원**. KR 내장 PII 엔티티 없음(US/CA/UK만, 주민번호는 고객이 regex 정의) | `[공식]` ApplyGuardrail REST API(모델 무관). 공식 MCP 서버는 awslabs/mcp 이슈 #79 = **제안 단계** | `[공식]` 공개가 — 콘텐츠 필터·금지주제 $0.15/1,000 text unit, PII $0.10, 문맥근거 $0.10, Automated Reasoning $0.17(정책당). **regex PII·word filter 무료** | 미확인 | [research-global](cycles/2026-08-pilot/1-research-global.md) §Q1~Q6 · aws.amazon.com/bedrock/guardrails/ · docs.aws.amazon.com/bedrock/latest/userguide/guardrails-sensitive-filters.html · /guardrails-supported-languages.html · github.com/awslabs/mcp/issues/79 · aws.amazon.com/bedrock/pricing/ · 2026-08-12 | 2026-08-12 |
| 이로운앤컴퍼니 세이프엑스 (SAIFE X) | `[보도]` 온프레미스 서술 있으나 **단일 보도 출처, 에어갭 미확인** | `[보도]` 유해성·탈옥성 2축 분리 평가(TAPD), 민감정보 필터링, Jailbreak Filter. 과잉 차단 완화 기법 LUBS 보유 | `[보도]` "한국어 공격 방어", 한국어 탐지 **97.88%** *(유해성/탈옥 축·벤더 자체 발표 — SGT strict micro F1(PII)과 데이터셋·과제 상이, 비교 불가)* | 미확인 | 미확인 | `[보도]` 조달청 디지털서비스몰 등록(2026-05-06), 첫 고객 한국정보보호교육원(2024-12 공급) | [research-domestic](cycles/2026-08-pilot/1-research-domestic.md) §Q2·Q5 · datanet.co.kr/news/articleView.html?idxno=201426 · zdnet.co.kr/view/?no=20250227113526 · dailysecu.com/news/articleView.html?idxno=206617 · byline.network/2025/08/25-468/ · 2026-08-12 | 2026-08-12 |
| 컴트루테크놀로지 Sphinx AI | 미확인 (구축형/SaaS 원문 명시 없음) | `[보도]` 개인정보·기업정보·금융정보 탐지, 입력 차단·마스킹 정책 | 미확인 | `[보도]` 상용 LLM(ChatGPT/Claude/Gemini/Copilot/HyperClovaX) 연계 + 사내 구축 LLM 중계 | 미확인 | `[보도]` 조달청 디지털서비스몰 등록 | [research-domestic](cycles/2026-08-pilot/1-research-domestic.md) §Q2 · dailysecu.com/news/articleView.html?idxno=168547 · 2026-08-12 | 2026-08-12 |
| Theori αprism | 미확인 (AI3 웍스AI에 **API 공급** 형태만 확인, 배포 형태 원문 명시 없음) | `[공식]` 개인정보·기밀 자동 마스킹, jailbreak 탐지·차단, PDF 등 **파일 검사** | `[마케팅]` 한국어 PII Micro F1 **89.0** *(자사 모델을 자사 벤치마크 General set으로 측정 — SGT 수치와 데이터셋·기준 상이, 비교 불가)*. 대상 유형: 주민등록번호·사업자등록번호·건강보험번호·전화번호·이메일·계좌번호·주소·이름 | 미확인 | 미확인 | 미확인 | [research-domestic](cycles/2026-08-pilot/1-research-domestic.md) §Q2·Q5 · theori.io/ko/news/b7552fb9-63c8-4df3-b725-bc6e95374074 · theori.io/ko/blog/korean-pii-detection-benchmark · 2026-08-12 | 2026-08-12 |
| CUBIG LLM Capsule | `[마케팅]` "온프레미스 및 폐쇄망 환경 지원" — **자사 사이트 단독 주장, 제3자 검증 없음** | 미확인 (민감정보 식별→캡슐화→외부 전송→응답 복원 구조만 서술) | 미확인 | 미확인 | 미확인 | `[마케팅]` 조달청 혁신장터 등재 | [research-domestic](cycles/2026-08-pilot/1-research-domestic.md) §Q2 · llmcapsule.ai/resources/learn/public-sector-genai-three-approaches-in-korea · 2026-08-12 | 2026-08-12 |
| 안랩클라우드메이트 시큐어브리지 | `[공식]` "온프레미스, SaaS 등 유연한 서비스 방식을 지원" — **에어갭 미확인**, 출시 연월 미확인 | `[공식]` 중요 데이터 입출력 탐지, 프롬프트 인젝션 방지, 프롬프트 이력 모니터링 및 정책 제어 | 미확인 | 미확인 | 미확인 | 미확인 (납품 사례 기재 없음) | [research-domestic](cycles/2026-08-pilot/1-research-domestic.md) §Q1 · company.ahnlab.com/kr/news/press_release_view.do?seqPressRelease=10147 · 2026-08-12 | 2026-08-12 |
| 파수 AI-R DLP | 미확인 (공식 페이지에 배포 형태 명시 없음) | `[공식]` 생성형 AI 내 민감정보 검출·차단, 차단 정책을 조직 환경별로 설정 | 미확인 | 미확인 | 미확인 | 미확인 | [research-domestic](cycles/2026-08-pilot/1-research-domestic.md) §Q1 · fasoo.ai/document/fasoo-ai-r-dlp · 2026-08-12 | 2026-08-12 |

### 행 미추가 (제품 부재 확인 — 선등록 금지 규칙)

| 시드 | 판정 | 근거 |
|---|---|---|
| SK쉴더스 | 해당 제품 없음 — EQST의 LLM 위협 분석·컨설팅·체크리스트만 확인, "프롬프트 보안 솔루션"은 자사 제품이 아니라 권고 대책 유형 | [research-domestic](cycles/2026-08-pilot/1-research-domestic.md) §Q1 · `[보도·스니펫만]` byline.network/2024/07/2-209/ · 2026-08-12 |
| 이글루코퍼레이션 | 해당 제품 없음(범주 불일치) — `AiR`는 보안관제 로그·이벤트 분석 LLM, 입출력 가드레일 제품 아님 | [research-domestic](cycles/2026-08-pilot/1-research-domestic.md) §Q1 · `[공식·스니펫만]` igloo.co.kr/service/generative-ai-secu/ · 2026-08-12 |
| 지란지교시큐리티 | 해당 제품 없음(사명 불일치) — 관련 제품은 **지란지교데이터**의 AX웍스·PCFILTER | [research-domestic](cycles/2026-08-pilot/1-research-domestic.md) §Q1 · `[보도]` byline.network/2025/08/25-468/ · 2026-08-12 |

## 갱신 이력

| 날짜 | 사이클 | 변경 |
|---|---|---|
| 2026-08-12 | Phase 1 | 뼈대 생성 — SGT 행 + 파일럿 글로벌 대상 6행(미확인) |
| 2026-08-12 | 2026-08-pilot | **12행 변경** — 글로벌 6행 갱신(전 셀 미확인 → 조사값), 국내 6행 신규(세이프엑스·Sphinx AI·αprism·LLM Capsule·시큐어브리지·AI-R DLP). SK쉴더스·이글루·지란지교시큐리티는 제품 부재 확인으로 미추가(위 표에 판정만 기록). 성능 수치 셀 취급 규칙 추가 |
