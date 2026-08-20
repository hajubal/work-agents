# SGT 기능 인벤토리 (living doc)

> 경쟁 매트릭스의 "SGT 열"과 planner의 "SGT 현재" 판단의 원천. **고객/시장 관점의 제공 기능** 목록이며 코드 구조 문서가 아니다.
> 갱신: /plan-cycle 단계 0에서 메인 세션이 **SGT 저장소**(`~/project/sgt`)의 `git -C ~/project/sgt log` 증분으로 갱신.
> **기준 커밋: `d2488305` (2026-08-12, sgt develop 기준)** — 기획 시스템이 별도 저장소로 분리되면서, SGT develop에서 도달 가능한 커밋으로 재설정했다(이전 기준 `14c2b09d`는 기획 문서 커밋이라 분리 후 사라진다. 두 커밋 사이에 제품 변경은 없다).

## 제품 한 줄

국내 고객사 **폐쇄망에 설치되는 온프레미스 LLM 보안 게이트웨이** — 한국어 특화 AI 탐지 4종+파일 보안, 종단 간 암호화(국산 암호 지원), RBAC 관리 콘솔.

## AI 탐지 능력 (차별화 핵심)

| 탐지 | 기본 모델 | 카테고리 | 입력 한계 | 정확도 | 비고 |
|---|---|---|---|---|---|
| PII | secureai-ko-pii-detector-v5 (GLiNER v4 hybrid) | GLiNER 12 라벨 + regex 5 라벨(주민번호·KR전화·KR계좌·카드·IP) | 384 형태소 | strict micro F1 0.8531 | 마스킹/익명화/삭제 전략 선택 |
| 독성 | secureai-ko-safe-classifier (mDeBERTa v3 Large) | 4개(혐오/괴롭힘/성적/폭력), **카테고리별 threshold** | 512 tok | — | 대안: Kanana 8B GGUF 7카테고리(threshold 미지원, NAS 조달) |
| 탈옥 | secureai-safeguard-prompt-2.1b | A1 인젝션/A2 유출 | 8192 tok | 97.5% | |
| 금칙어 | 규칙 기반 | 키워드+정규식, 관리자 설정 | — | — | 모델 불필요 |
| 파일 보안 | — | 첨부파일 콘텐츠 추출·민감정보 탐지·체크섬 | — | — | |

**한국어 특화 요소**: 전 모델 자체(salmon113) 한국어 파인튜닝 / PII 법령 문체 학습 + MeCab 사용자 사전 4,745종(#729) / 형태소 단위 청킹 이중 방어 / 한국어 문학 오탐 필터(PersonNameFilter) / 한·영 자동 언어 감지 / 국산 파운데이션(Kanana·kf-deberta) 계열 활용.

**추론 백엔드**: 서비스별 독립 선택 `SGT_{TOXIC,PII,JAILBREAK}_BACKEND` = auto/local/triton/vllm, 호환성 자동 검증. 탐지별 on/off `DISABLE_*_MODEL`. GPU VRAM 기본 ~8.3GB.

## 보안·암호

- 종단 간 암호화: RSA-2048 + AES-256 하이브리드, SHA256withRSA 서명
- **국산 암호**: INICrypto 5.0 Primary(SEED/CBC 128, RSA-OAEP, MGF1 서명) + BouncyCastle 폴백, SDK도 cryptoType 선택(inicrypto/bouncycastle)
- **KeyFix(INISAFENet) 연동**: 자재→values 변환 스크립트, `-f values-keyfix.yaml` 한 줄 활성화
- JWT 스코프 기반 인가(7종 스코프, 최소 권한 강제), 토큰 revocation(해시 원장 + 최소구성용 jti denylist), 2FA(TOTP)
- OWASP 장애 정책 fail-open/fail-close 선택(`OWASP_FAIL_MODE`), 라이선스 RSA 서명 검증·기능별 on/off·클러스터 바인딩

## 게이트웨이·성능

- 다층 보안 게이트웨이(JWT+암호화+서명), 지능형 라우팅+Circuit Breaker(Resilience4j)
- 스트리밍 최적화(백프레셔 토큰 버퍼링, 유니코드 코드포인트 청킹), min-chars 추론 게이트(짧은 입력 스킵)
- 속도 제한·티어(Basic/Premium/Enterprise) 월간 토큰 쿼터·시간 윈도우 제한, OWASP 수평 확장(compose --scale/K8s HPA)

## 관리 콘솔 (admin)

- RBAC(SpEL 동적 역할·계층 역할·속성 기반), SCIM v2 프로비저닝, API 키 관리(스코프 필수)
- 대시보드(실시간 사용 통계·그룹 리포트·워드클라우드·응답시간), 채팅 세션 감사·키워드 검색, 탐지 요청 감사 DB
- AI 채팅 플레이그라운드(RAG 통합), 리소스 관리(Excel 배치 업로드·버전·태그)

## 통합 표면

| 표면 | 내용 |
|---|---|
| REST detect API | `POST /api/v1/detect/**` 5종 (PII·독성·탈옥·금칙어·통합배치), S2S 연동 |
| **MCP 서버** | `/api/v1/mcp` STATELESS streamable HTTP — 탐지 도구 5종, REST와 라이선스·감사 동일 적용, 표준 MCP 클라이언트 무코드 연동 |
| JS SDK (SecureMode) | fetch 자동 래핑 — 앱 코드 수정 없이 정책(glob) 기반 자동 암호화, proxy/chat 2모드 |
| OpenAPI | `/openapi/v1/**` default-deny 스코프 접근, 사용자 자동 프로비저닝(SGT 비마스터 환경), 셀프 API키 재발급 |
| sgt-ctl CLI/TUI | admin 없는 최소구성의 설정 제어(필터 13종·threshold·금칙어·키 발급/폐기) + Loki 로그·Prometheus 지표·버전/라이선스 조회, 정적 바이너리 |

## 배포 스펙트럼

| 형태 | 요지 |
|---|---|
| docker-compose | 전 서비스 단일 명령, 모델 자동 다운로드 프로필 |
| Helm 풀구성 | 전 서비스 + 관측 스택(Loki/Grafana/Prometheus/Tempo/OTel) |
| **Helm 최소구성** | gateway+owasp 2개만(admin/DB/rabbit 없이) — 자체 사용자 마스터 보유 고객용, 라이선스 로컬 검증·오프라인 키 발급 |
| GPU (triton_vllm) | Triton/vLLM 외부 추론, GPU 멀티노드 scale-out |
| **폐쇄망 오프라인 패키지** | 에어갭 2단계(온라인 생성→반입 배포) 절차 문서화 — 이미지·차트·모델 전체 |
| 프리셋 | OpenShift / 온프레미스 / S3 모델 마운트 / 민감로그 비활성 / 이미지 baked |

모델 수급 3경로: HuggingFace / S3 / 사내 NAS·외장하드(폐쇄망).

## 매트릭스 작성 시 유의 (정직한 한계)

- 독성 모델 전환은 무중단 아님(양쪽 설정+재기동), threshold는 기본 mDeBERTa만 지원
- Kanana 8B 등 일부 모델 HF 미배포 — NAS 수동 조달
- 최소구성은 월간 쿼터 fail-open, 탐지 감사 DB·관리 UI 없음
- 성능 공식 수치(레이턴시 SLA) 미측정 — 경쟁사가 수치를 낼 때 SGT는 "미측정"이 정직한 값

## 변경 이력

| 날짜 | 기준 커밋 | 내용 |
|---|---|---|
| 2026-08-12 | `14c2b09d` | 최초 생성 (Phase 1, 저장소 조사 기반) |
| 2026-08-13 | `d2488305` | 기준 커밋 재설정 — 기획 시스템이 별도 저장소로 분리(제품 내용 변경 없음) |
