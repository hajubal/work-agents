# 2026-08-pilot 조사 보고서 — global

## 요약 (5줄 이하)

1. **에어갭을 공식 문서로 확인한 곳은 Lakera(현 Check Point) 1곳**, 제품 페이지 문구 수준은 F5(CalypsoAI)·SentinelOne 2곳. Cisco는 관리 평면이 Cisco 호스팅이라 에어갭 미확인, Bedrock은 클라우드 전용으로 폐쇄망 불가.
2. **탐지 기능을 MCP 서버로 노출하는 제품은 6개 중 0곳** — 전부 "MCP를 보호"(게이트웨이·스캐너)일 뿐이다. SGT의 MCP tool 노출은 아직 차별점.
3. **한국 개인정보 유형(주민등록번호) 내장 엔티티를 명시한 제품 없음** — 전부 커스텀 정규식에 위임. Bedrock 금칙어 필터는 한국어 미지원(영/불/스).
4. 도메인 오탐 대응의 업계 표준은 (a)커스텀 정규식 + (c)신뢰도 임계값이며, **고객 데이터 파인튜닝을 공식화한 벤더는 0곳**(전부 미확인).
5. 오탐 관리 기능의 최고 수준은 Lakera의 L1~L4 신뢰도 + Policy Impact Simulator, 멀티 탐지기 오케스트레이션 선례는 NeMo Guardrails.

## 질문별 발견

### Q1. 배포 모델 — SaaS / 하이브리드 / self-hosted / 에어갭 설치 가능 여부

- **Lakera Guard (현 Check Point AI Guardrails)** — self-hosted 지원. "Full offline deployment support for the most restrictive environments", "Containers can be exported and loaded without internet access", 설치는 공식 Helm 차트 또는 Docker. 단 "A valid AI Guardrails Enterprise license" 필요하며 **오프라인 라이선스 활성화 절차·모델 패키징 여부는 문서에 없음(미확인)**. 상세 self-hosted 문서는 고객 전용 포털(self-hosted.docs.lakera.ai) — [공식] [https://docs.lakera.ai/docs/selfhosting](https://docs.lakera.ai/docs/selfhosting) (2026-08-12)
- **Prompt Security (SentinelOne)** — "Prompt Security On-Premises" 2026-03-23 발표, "adds these same protections to the world of AI, even in fully disconnected environments". **GA 여부·설치 방식·라이선스 검증 방식 미확인**(보도자료 외 설치 문서 확인 실패) — [공식] (보도자료) [https://www.sentinelone.com/press/sentinelone-brings-ai-security-to-on-premise-regulated-sovereign-self-hosted-and-airgapped-environments/](https://www.sentinelone.com/press/sentinelone-brings-ai-security-to-on-premise-regulated-sovereign-self-hosted-and-airgapped-environments/) (2026-08-12)
- **CalypsoAI (F5 인수, 현 F5 AI Guardrails)** — 제품 페이지에 "Maintain full functionality in on-prem or fully air-gapped environments", public cloud / private cloud / on-prem 4종 배포 명시. **설치 문서·오프라인 절차는 미확인**(제품 마케팅 페이지 문구 수준) — [마케팅] (공식 도메인 제품 페이지) [https://www.f5.com/products/ai-guardrails](https://www.f5.com/products/ai-guardrails) (2026-08-12)
- **Cisco AI Defense** — 하이브리드. Cisco 호스팅 컨트롤 플레인 + 고객 데이터 플레인(SaaS / 고객 VPC / 온프렘 AI POD), Kubernetes 네이티브(EKS/AKS/GKE/OpenShift). "Only metadata required for management-plane operations is sent to Cisco" → **관리 평면 아웃바운드가 전제되므로 완전 폐쇄망 여부는 미확인**. 데이터시트 원문 2회 403으로 열지 못함 — [공식·스니펫만] [https://www.cisco.com/c/en/us/products/collateral/security/ai-defense/ai-defense-ds.html](https://www.cisco.com/c/en/us/products/collateral/security/ai-defense/ai-defense-ds.html) (2026-08-12)
- **NVIDIA NeMo Guardrails** — Apache 2.0 OSS. Python 라이브러리 / `nemoguardrails server` / Docker / K8s(NIM Operator·Helm). 기본은 외부 LLM 호출 의존이나 NIM·vLLM 로컬 서빙으로 자체 호스팅 가능. **공식 에어갭 설치 가이드는 미확인** — [공식] [https://github.com/NVIDIA-NeMo/Guardrails/blob/develop/README.md](https://github.com/NVIDIA-NeMo/Guardrails/blob/develop/README.md) , [https://docs.nvidia.com/nemo/guardrails/latest/about/overview.html](https://docs.nvidia.com/nemo/guardrails/latest/about/overview.html) (2026-08-12)
- **AWS Bedrock Guardrails** — AWS 관리형 클라우드 전용, **self-hosted/온프레미스 옵션 없음**. 단 `ApplyGuardrail` API는 "with any foundation model whether hosted on Amazon Bedrock or self-hosted models, including third-party models" — 즉 보호 대상은 자체 모델이어도 **가드레일 자체는 AWS 클라우드에서 실행** → 폐쇄망 적용 불가 — [공식] [https://aws.amazon.com/bedrock/guardrails/](https://aws.amazon.com/bedrock/guardrails/) (2026-08-12)



### Q2. 탐지 항목 및 정책 조정 단위

- **Lakera** — 6종: Prompt Defense(jailbreak·prompt injection) / Content Moderation(Crime, Hate, Profanity, Sexual, Violence, Weapons, Self Harm 7카테고리) / Data Leakage Prevention(PII·시스템 프롬프트 유출) / Malicious Links / Agent Behavior Defense(Off-Task Action, Tool Allow/Deny List) / Custom Guardrails(정규식). **정책별 신뢰도 4단계 L1(Lenient)~L4(Paranoid)** 조정. **파일 스캔은 문서에 언급 없음** — [공식] [https://docs.lakera.ai/docs/defenses](https://docs.lakera.ai/docs/defenses) , [https://docs.lakera.ai/docs/content-moderation](https://docs.lakera.ai/docs/content-moderation) (2026-08-12)
- **AWS Bedrock Guardrails** — 6 정책: content filters(텍스트·이미지) / denied topics / word filters / sensitive information filters(PII 30+ 엔티티 + 커스텀 regex, Block·Mask 2모드) / contextual grounding(환각) / Automated Reasoning checks. 커스텀 regex는 **lookaround 미지원** — [공식] [https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails-sensitive-filters.html](https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails-sensitive-filters.html) (2026-08-12)
- **NeMo Guardrails** — input/dialog/retrieval/execution/output 5단계 레일. jailbreak·injection 탐지, content safety(Nemotron Content Safety / Llama Guard 3 / ShieldGemma), PII(GLiNER·Presidio·Private AI·Polygraf·AutoAlign·GuardrailsAI 중 선택), topical control(Colang) — [공식] [https://docs.nvidia.com/nemo/guardrails/configure-guardrails/guardrail-catalog/content-safety](https://docs.nvidia.com/nemo/guardrails/configure-guardrails/guardrail-catalog/content-safety) (2026-08-12)
- **F5 (CalypsoAI)** — prompt injection, jailbreak, data exfiltration, PII 유출, content moderation(toxic·biased·inaccurate), GDPR/HIPAA/EU AI Act/PCI/PHI 컴플라이언스 컨트롤, "standard and custom categories", agent tool call 감사. **카테고리별 threshold 공개 여부 미확인** — [마케팅] [https://www.f5.com/products/ai-guardrails](https://www.f5.com/products/ai-guardrails) (2026-08-12)
- **Prompt Security** — sensitive data, secrets, prompt injection, topics detector, customer guardrails, shadow MCP 탐지, MCP 서버 risk scoring, 사용자/서버/액션 단위 allow-block 정책. **탐지 카테고리 전체 목록·threshold는 공식 문서 확인 실패(미확인)** — [마케팅] [https://prompt.security/](https://prompt.security/) , [https://prompt.security/solutions/agentic-ai-security-and-governance](https://prompt.security/solutions/agentic-ai-security-and-governance) (2026-08-12)
- **Cisco AI Defense** — prompt injection·adversarial attack 대응 언급 외 **탐지 카테고리 전체 목록 미확인**(데이터시트 403)



### Q3. 통합 표면 — MCP 서버를 제공하는 제품이 있는가

**결론: 탐지 기능을 MCP tool로 노출하는 제품은 확인된 6개 중 0곳.** 전부 "MCP 트래픽·서버를 보호"하는 방향이며, SGT처럼 자사 탐지를 MCP 서버로 제공하는 사례는 확인되지 않았다.

- **Lakera** — MCP 서버 미제공. 고객이 자기 MCP 서버 코드에 데코레이터(`@guard_content`)를 붙여 Guard API를 호출하는 방식. "Adding a single line ... to your tools, prompts or resources you can start guarding them" — [마케팅] [https://www.lakera.ai/blog/how-to-secure-mcps-with-lakera-guard](https://www.lakera.ai/blog/how-to-secure-mcps-with-lakera-guard) (2026-08-12). Zapier가 제3자 래퍼 MCP를 제공하나 Lakera 자사 제품 아님 [커뮤니티] [https://zapier.com/mcp/lakera-ai-guardrails](https://zapier.com/mcp/lakera-ai-guardrails)
- **Prompt Security** — AI Gateway/MCP Gateway가 에이전트↔MCP 서버 **사이의 프록시**로 요청·응답 검사. MCP 서버 제공 아님 — [마케팅] [https://prompt.security/solutions/agentic-ai-security-and-governance](https://prompt.security/solutions/agentic-ai-security-and-governance) (2026-08-12)
- **Cisco AI Defense** — "the Cisco AI Defense MCP Scanner performs deep analysis of tool descriptions and schema" — 레지스트리 등록 시 **정적 스캔**이지 MCP 서버도 인라인 프록시도 아님. 표면은 REST API + CI/CD 통합 — [마케팅] (AWS 벤더 블로그) [https://aws.amazon.com/blogs/machine-learning/securing-ai-agents-how-aws-and-cisco-ai-defense-scale-mcp-and-a2a-deployments/](https://aws.amazon.com/blogs/machine-learning/securing-ai-agents-how-aws-and-cisco-ai-defense-scale-mcp-and-a2a-deployments/) (2026-08-12)
- **AWS Bedrock** — 공식 MCP 서버는 awslabs/mcp **이슈 #79(제안 단계)**, 정식 제공 여부 미확인. 커뮤니티 구현은 존재(관리 API 노출용). 1급 표면은 `ApplyGuardrail` REST API — [공식] [https://github.com/awslabs/mcp/issues/79](https://github.com/awslabs/mcp/issues/79) (2026-08-12)
- **NeMo Guardrails** — Python SDK / REST 서버 / Docker. **MCP 서버 미확인**
- **Lakera 통합 표면 보강**: REST Guard API + 프로젝트 단위 정책, APISIX 게이트웨이 플러그인 제안 존재 [커뮤니티] [https://github.com/apache/apisix/issues/13291](https://github.com/apache/apisix/issues/13291)



### Q4. 한국어·다국어 대응

- **AWS Bedrock Guardrails** — 한국어가 **"Optimized and supported"**: content filters(Standard tier)·denied topics·sensitive information filters(PII) 3종. **단 word filters(금칙어)는 English/French/Spanish 3개 언어만 — 한국어 미지원**. contextual grounding도 영/불/스만 — [공식] [https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails-supported-languages.html](https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails-supported-languages.html) (2026-08-12)
- **Bedrock PII 엔티티에 한국 특화 없음** — 국가별 엔티티는 USA(SSN·ITIN·여권 등)·Canada(SIN·Health Number)·UK(NINO·NHS·UTR)뿐. **주민등록번호는 커스텀 regex로 고객이 직접 정의해야 함** — [공식] [https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails-sensitive-filters.html](https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails-sensitive-filters.html) (2026-08-12)
- **Lakera** — "Our detector models have been specifically trained to screen content prompt attacks in over 100+ major languages and scripts", "Asian languages (Chinese, Japanese, Korean, Vietnamese, Thai, etc.)". 이는 **prompt attack 스크리닝 기준**이며 PII·한국 식별자 대응은 미확인 — [공식] [https://docs.lakera.ai/docs/prompt-defense](https://docs.lakera.ai/docs/prompt-defense) (2026-08-12)
- **NVIDIA Nemotron-3.5-Content-Safety** — "Supported languages include English, Arabic, German, Spanish, French, Hindi, Japanese, Thai, Dutch, Italian, Korean and Chinese"(12개). 라이선스 OpenMDW-1.1 + Gemma Terms of Use — [공식] [https://huggingface.co/nvidia/Nemotron-3.5-Content-Safety](https://huggingface.co/nvidia/Nemotron-3.5-Content-Safety) (2026-08-12)
- **F5(CalypsoAI) / Cisco / Prompt Security** — 한국어 지원 명시 **미확인**(F5 페이지의 한국어는 사이트 로케일일 뿐 제품 기능 아님)
- **종합**: 확인 범위에서 **한국어 전용 모델을 명시한 글로벌 제품은 없음** — 전부 다국어 범용 모델이며, 한국 개인정보 유형은 커스텀 정규식에 위임.



### Q5. 도메인 특화 오탐 대응 방식 (핵심 질문)

- **(a) allowlist/커스텀 엔티티** — Lakera: "Using policies, you can create custom regular expression based detectors for content moderation"(Content Moderation·DLP 대상, 예시로 경쟁사명·현지 슬랭). 단 **오탐 억제용 allowlist·예외 사전 메커니즘은 문서에 없음(미확인)** — [공식] [https://docs.lakera.ai/docs/content-moderation](https://docs.lakera.ai/docs/content-moderation) (2026-08-12). Bedrock: 커스텀 regex(BLOCK/ANONYMIZE/NONE) + PII 엔티티 개별 on/off, **NONE 액션으로 탐지만 하고 조치 안 함** 가능 — [공식] guardrails-sensitive-filters (2026-08-12). F5: "standard and custom categories" [마케팅]
- **(b) 고객 데이터 파인튜닝·커스텀 모델** — **6개 벤더 모두 공식 서술 확인 못함 → 전부 미확인.** NeMo만 OSS 구조상 고객이 모델을 직접 교체·파인튜닝 가능(단 NVIDIA가 제공하는 서비스가 아님). **에어갭에서 클라우드 재학습에 의존하는 방식은 어느 벤더에서도 확인되지 않았고, 반대로 로컬 파인튜닝 지원을 광고하는 벤더도 없다.**
- **(c) 신뢰도 임계값 조정** — **Lakera가 가장 구체적**: 정책별 L1(Lenient, "very few false positives") ~ L4(Paranoid, "higher false positives but very few false negatives") 4단계 + "The Policy Impact Simulator in the dashboard provides a visual way to compare flagging rates across all sensitivity levels using your historical traffic" — [공식] [https://docs.lakera.ai/docs/policies](https://docs.lakera.ai/docs/policies) , [https://docs.lakera.ai/docs/defenses](https://docs.lakera.ai/docs/defenses) (2026-08-12). Bedrock의 카테고리별 강도 등급은 이번 조사에서 원문 미확인.
- **(d) 다중 탐지기 오케스트레이션** — **NeMo Guardrails가 유일한 명시적 선례**: 5단계 레일에 PII 엔진(GLiNER/Presidio/Private AI/Polygraf/AutoAlign/GuardrailsAI)과 content safety 모델(Nemotron/Llama Guard 3/ShieldGemma), 서드파티(ActiveFence·Prompt Security·Pangea 등)를 **플러그인으로 조합**하는 구조 — [공식] [https://docs.nvidia.com/nemo/guardrails/latest/about/overview.html](https://docs.nvidia.com/nemo/guardrails/latest/about/overview.html) (2026-08-12). 브리프 고객 맥락 1("도메인마다 별도 모델 vs 여러 모델 오케스트레이션")에 대한 업계 답은 **"단일 도메인 모델 교체가 아니라 조합형 파이프라인 + 임계값·정규식 튜닝"** 쪽이다.
- **에어갭 적용 가능성**: (a)(c)(d)는 전부 로컬 설정 파일·정책으로 동작하므로 폐쇄망에서도 성립. (b)는 어느 벤더도 확인되지 않아 **SGT가 "고객 도메인 사전·예외 처리"로 승부할 여지가 열려 있음**.



### Q6. 가격 신호

- **AWS Bedrock Guardrails** — 공개 가격: content filters(텍스트) **$0.15 / 1,000 text units**, denied topics $0.15, sensitive information filters(PII) **$0.10**, contextual grounding $0.10, Automated Reasoning $0.17(정책당). **정규식 기반 PII 필터와 word filters는 무료.** 1 text unit = 최대 1,000자 — [공식] [https://aws.amazon.com/bedrock/pricing/](https://aws.amazon.com/bedrock/pricing/) (2026-08-12)
- **NeMo Guardrails** — Apache 2.0 무료. 단 모델 라이선스 별도(Nemotron-3.5-Content-Safety = OpenMDW-1.1 + Gemma Terms) — [공식] GitHub LICENSE (2026-08-12)
- **Lakera** — 무료 Community 티어(월 10,000 요청) 존재, self-hosting은 Enterprise 문의. **pricing 페이지 원문 렌더 실패로 3자 요약 기반** — [커뮤니티·스니펫만] [https://www.eesel.ai/blog/lakera-pricing](https://www.eesel.ai/blog/lakera-pricing) (2026-08-12)
- **F5(CalypsoAI) / Cisco AI Defense / Prompt Security** — **미확인**(전부 견적·데모 문의 방식, 공개 가격 없음)



## 매트릭스 갱신 제안


| 제품                 | 열         | 제안 값                                                                                                                                                               | 출처(등급·URL·확인일)                                                                  |
| ------------------ | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------- |
| Lakera Guard       | 제품명       | **Check Point AI Guardrails로 리브랜딩**(문서가 "Check Point AI Security" 표기) — 행 제목 갱신 검토                                                                                 | [공식] docs.lakera.ai/docs/defenses · 2026-08-12                                  |
| Lakera Guard       | 배포 모델     | SaaS + self-hosted(Helm/Docker), **오프라인 배포 공식 지원**("Containers can be exported and loaded without internet access"). 오프라인 라이선스 활성화 절차 미확인                          | [공식] docs.lakera.ai/docs/selfhosting · 2026-08-12                               |
| Lakera Guard       | 탐지 기능     | Prompt Defense / Content Moderation(7종) / DLP(PII·시스템프롬프트) / Malicious Links / Agent Behavior / Custom regex. 파일 스캔 미확인. 정책별 신뢰도 L1~L4                             | [공식] docs.lakera.ai/docs/defenses · 2026-08-12                                  |
| Lakera Guard       | 한국어 지원    | 다국어 범용(100+ 언어, prompt attack 기준 한국어 포함). 한국어 전용 모델 아님, KR 개인정보 유형 미확인                                                                                             | [공식] docs.lakera.ai/docs/prompt-defense · 2026-08-12                            |
| Lakera Guard       | 통합 표면     | REST Guard API + 프로젝트별 정책. **MCP 서버 미제공**(고객 MCP에 데코레이터로 API 호출)                                                                                                   | [마케팅] lakera.ai/blog/how-to-secure-mcps-with-lakera-guard · 2026-08-12          |
| Lakera Guard       | 가격 신호     | 무료 Community 월 10,000 요청 / Enterprise 문의 `[스니펫만]`                                                                                                                  | [커뮤니티] eesel.ai/blog/lakera-pricing · 2026-08-12                                |
| Prompt Security    | 배포 모델     | SaaS / 브라우저 확장 / **On-Premises(2026-03-23 발표, "fully disconnected environments")**. GA·설치 방식 미확인                                                                   | [공식] sentinelone.com/press/...airgapped-environments · 2026-08-12               |
| Prompt Security    | 통합 표면     | AI Gateway·MCP Gateway(리버스 프록시), 엔드포인트 에이전트, 브라우저 확장. **MCP 서버 제공 아님**                                                                                             | [마케팅] prompt.security/solutions/agentic-ai-security-and-governance · 2026-08-12 |
| Prompt Security    | 탐지/한국어/가격 | 미확인                                                                                                                                                                | —                                                                               |
| CalypsoAI (F5)     | 배포 모델     | public cloud / private cloud / **on-prem·"fully air-gapped"**(제품 페이지 문구, 설치 문서 미확인)                                                                                | [마케팅] f5.com/products/ai-guardrails · 2026-08-12                                |
| CalypsoAI (F5)     | 탐지 기능     | prompt injection·jailbreak·data exfiltration·PII·content moderation·컴플라이언스(GDPR/HIPAA/EU AI Act/PCI/PHI)·agent tool call 감사, custom categories                     | [마케팅] f5.com/products/ai-guardrails · 2026-08-12                                |
| CalypsoAI (F5)     | 한국어/가격    | 미확인 (F5가 2025-09 인수, $180M)                                                                                                                                        | [보도] f5.com/company/news/press-releases · 2026-08-12                            |
| Cisco AI Defense   | 배포 모델     | 하이브리드 — **Cisco 호스팅 컨트롤 플레인** + 데이터 플레인(SaaS/고객 VPC/온프렘 AI POD), K8s 네이티브. 관리 평면 메타데이터가 Cisco로 전송 → **에어갭 미확인**                                                    | [공식·스니펫만] cisco.com/.../ai-defense-ds.html · 2026-08-12                         |
| Cisco AI Defense   | 통합 표면     | REST API·CI/CD·**MCP Scanner(정적 스캔, MCP 서버 아님)**                                                                                                                   | [마케팅] aws.amazon.com/blogs/machine-learning/securing-ai-agents-... · 2026-08-12 |
| Cisco AI Defense   | 탐지/한국어/가격 | 미확인 (데이터시트 403)                                                                                                                                                    | —                                                                               |
| NeMo Guardrails    | 배포 모델     | Apache 2.0 OSS — Python 라이브러리 / 서버 / Docker / NIM Operator. NIM·vLLM 로컬 서빙 가능, **공식 에어갭 가이드 미확인**                                                                  | [공식] github.com/NVIDIA-NeMo/Guardrails · 2026-08-12                             |
| NeMo Guardrails    | 탐지 기능     | input/dialog/retrieval/execution/output 5레일, jailbreak·injection, content safety(Nemotron/Llama Guard 3/ShieldGemma), PII 플러그인(GLiNER·Presidio 등), topical(Colang) | [공식] docs.nvidia.com/nemo/guardrails/.../content-safety · 2026-08-12            |
| NeMo Guardrails    | 한국어 지원    | Nemotron-3.5-Content-Safety가 한국어 포함 12개 언어 지원(전용 아님)                                                                                                               | [공식] huggingface.co/nvidia/Nemotron-3.5-Content-Safety · 2026-08-12             |
| Bedrock Guardrails | 배포 모델     | **AWS 관리형 클라우드 전용, self-host 없음** → 폐쇄망 불가. ApplyGuardrail API로 외부·자체호스팅 모델 보호는 가능                                                                                 | [공식] aws.amazon.com/bedrock/guardrails/ · 2026-08-12                            |
| Bedrock Guardrails | 탐지 기능     | content filters / denied topics / word filters / sensitive info(PII 30+ 엔티티 + 커스텀 regex, Block·Mask·None) / contextual grounding / Automated Reasoning             | [공식] docs.aws.amazon.com/.../guardrails-sensitive-filters.html · 2026-08-12     |
| Bedrock Guardrails | 한국어 지원    | 한국어 "Optimized and supported"(content filter·denied topics·PII). **word filters는 영/불/스만 = 한국어 금칙어 미지원**. KR 개인정보 내장 엔티티 없음(US/CA/UK만)                              | [공식] docs.aws.amazon.com/.../guardrails-supported-languages.html · 2026-08-12   |
| Bedrock Guardrails | 통합 표면     | ApplyGuardrail REST API(모델 무관). 공식 MCP 서버는 awslabs/mcp 이슈 #79 = 제안 단계                                                                                              | [공식] github.com/awslabs/mcp/issues/79 · 2026-08-12                              |
| Bedrock Guardrails | 가격 신호     | 공개가 — 콘텐츠 필터·금지주제 $0.15/1,000 text unit, PII $0.10, 문맥근거 $0.10, Automated Reasoning $0.17. **regex PII·word filter 무료**                                            | [공식] aws.amazon.com/bedrock/pricing/ · 2026-08-12                               |




## 미확인·미조사 목록

- **Lakera 오프라인 라이선스 활성화 방식·모델 로컬 패키징 여부** — 상세 문서가 고객 전용 포털(self-hosted.docs.lakera.ai)이라 접근 불가. SGT와의 실질 비교에 가장 중요한 항목이므로 다음 사이클 최우선.
- **Cisco AI Defense 탐지 카테고리 전체·에어갭 가능 여부·가격** — 데이터시트/레퍼런스 아키텍처 문서 모두 HTTP 403(2회 시도). 스니펫 외 원문 검증 실패.
- **Prompt Security 탐지 카테고리 목록·threshold 조정 단위·On-Premises GA 여부** — 공식 문서 사이트 확인 실패, 보도자료·마케팅 페이지만 확보.
- **F5(CalypsoAI) 에어갭의 실체** — 제품 페이지 문구뿐, 설치·오프라인 절차 문서 미확인.
- **고객 데이터 파인튜닝(Q5-b)** — 6개 벤더 전부 공식 근거 없음. "없다"가 아니라 "확인 못 함"임에 주의.
- **한국 개인정보 유형 대응** — Bedrock만 엔티티 목록 원문 확인(KR 없음). 나머지 5개는 PII 엔티티 목록 자체를 확보하지 못함.
- **성능·레이턴시 수치 비교** — Lakera의 "sub-50ms / 98%+ 탐지" 등은 3자 요약 스니펫이라 원문 미검증, 이번 보고서에서 제외.
- **미조사**: 각 제품의 파일 스캔 기능 유무(Q2 항목 중 하나), 감사·로그 보존 기능.



## 사용한 검색 (19회 / 상한 25회)

Lakera(자체호스팅·탐지·MCP·다국어·가격) 5 / Prompt Security(온프렘·탐지·문서) 3 / CalypsoAI·F5 2 / Cisco AI Defense 2 / NeMo Guardrails 3 / Bedrock(언어·MCP) 2 / 횡단 질문(Q5 오탐 대응) 1 · 원문 검증 WebFetch 18회(성공 13 / 403·404 5)