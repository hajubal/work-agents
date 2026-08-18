# feat(owasp): 한국 PII 식별자 유형 커버리지 점검·보강

> 기획 사이클: 2026-08-12-pilot · 2-plan.md 후보 #2 · 승인: 4-decision.md (2026-08-13)

## Problem

경쟁이 KR 식별자 유형 목록을 공개해 영업 자리에서 체크리스트 비교가 시작됐는데, 우리는 "무엇을 잡는가"를 문서 한 장으로 답하지 못한다.

`sgt-feature-inventory.md`는 regex 5종(주민번호·KR전화·KR계좌·카드·IP)이라 적고 있지만 저장소의 패턴 인식기 `recognizer_pattern.py`의 `supported_entity`는 10종이다. 우리 문서가 우리 제품보다 적게 적혀 있다. 손해를 보는 쪽은 현장에서 고객 PM 질문을 받는 영업, 그리고 평가셋 밖 라벨의 실제 품질을 모르는 owasp 담당이다.

## Goals

- KR 식별자 유형 × (인식기 유무 / 평가셋 포함 / 체크섬 검증 유무) 표가 문서로 존재하고 `sgt-feature-inventory.md`와 일치한다
- Theori 공개 8종 각각에 "있음 / 없음 / 평가 밖" 판정이 붙는다
- 라벨을 추가했다면 로컬 폴백 JSON과 admin 시드 값이 같다 (#720과 같은 불일치를 새로 만들지 않는다)
- 추가·편입 후 법령 clean 코퍼스(#729 112종) FPR이 합의 상한을 넘지 않는다

점검 결과가 "이미 커버"로 나와 추가 개발이 0건으로 끝나는 것도 정상 종료다.

## Non-Goals

- **#875** (주소 gazetteer 행정동 편입·금융/카드 브랜드 유한목록) — 이미 있는 라벨의 recall 개선이라 방향이 다르다. 겹치는 것은 재측정 게이트뿐이다
- **#833** (영어 경로 `PresidioPiiRecognizer`의 `supported_entities` 반환 타입 결함) — 원인과 수정 방향이 이미 특정된 버그다. 이 이슈가 흡수하면 티켓 하나가 기획 승인을 기다리게 된다. 별도로 먼저 닫는다
- 오탐 억제용 예외 사전 — #781 트리 소관이다 (같은 사이클 후보 1)
- 신규 모델·외부 데이터셋 반입 없음 → 오프라인 패키징 변경 없음

## Requirements

- [ ] [owasp] `recognizer_pattern.py`의 `supported_entity` 10종(`KR_ID`·`KR_PHONE`·`KR_BANK_ACCOUNT`·`CREDIT_CARD`·`EMAIL_ADDRESS`·`KR_PASSPORT`·`KR_DRIVER_LICENSE`·`KR_BUSINESS_REG`·`KR_CORP_REG`·`KR_MEDICAL_INSURANCE`), `entity_threshold_config.py` 임계값 카논(+`KR_ADDRESS_UNIT`), GLiNER 12 라벨을 실측해 커버리지 표를 만든다
- [ ] [owasp] Theori 공개 8종(주민등록번호·사업자등록번호·건강보험번호·전화번호·이메일·계좌번호·주소·이름)에 각각 판정을 붙인다
- [ ] [owasp] `pii_validators.py`의 체크섬 검증 실제 구현 범위를 확인한다 — 현재 `미확인`이며 `KR_CORP_REG` 관련 서술만 확인됐다
- [ ] [owasp] 검증 측정 13라벨 밖인 라벨(`KR_MEDICAL_INSURANCE` 등)을 평가셋에 편입한다. 이 라벨들은 "없음"이 아니라 "있지만 재본 적 없음"이므로 작업의 실체는 보강이 아니라 측정 편입이다
- [ ] [docs] `sgt-feature-inventory.md`와 영업 자료의 PII 행을 저장소 실측에 맞춘다
- [ ] [owasp] 법령 clean 코퍼스(#729 112종) FPR을 재측정한다
- [ ] [ctl] 작업 없음을 확인한다 — sgtctl은 threshold 키를 파일에서 읽고 하드코딩하지 않으므로 라벨이 늘면 목록에 자동으로 뜬다. `pii_entity_thresholds.json` 기본값만 맞으면 된다
- [ ] **라벨을 실제로 추가하는 경우에만**: [admin] `init_script.sql` 시드(`sgt_app_config`) 갱신 + 수동 DB 마이그레이션 1세트(migration 스크립트 + 문서 + 기동 자가검증), [owasp] 로컬 폴백 JSON 정합(#720), admin↔owasp 설정 왕복(#838) 확인

## 경쟁·시장 근거

- Theori αprism이 KR PII 대상 유형 8종을 명시했다 — 주민등록번호·사업자등록번호·건강보험번호·전화번호·이메일·계좌번호·주소·이름. Micro F1 89.0 / NVIDIA gliner-pii 59.0 / OpenAI privacy-filter 45.0. [마케팅] https://theori.io/ko/blog/korean-pii-detection-benchmark · 확인일 2026-08-12 (D§Q5). 3-critique.md 출처 표본 검증 4번에서 **일치** 판정을 받았다
- 글로벌 제품의 KR 내장 엔티티는 0곳이다. Bedrock의 국가별 PII 엔티티는 USA·Canada·UK뿐이고 주민등록번호는 고객이 커스텀 regex로 직접 정의해야 한다. [공식] https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails-sensitive-filters.html · 확인일 2026-08-12 (G§Q4)
- Bedrock 금칙어(word filters)는 영어·프랑스어·스페인어만 지원하며 한국어는 미지원이다. [공식] https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails-supported-languages.html · 확인일 2026-08-12 (G§Q4)
- **수치는 근거로 쓰지 않는다.** Theori 89.0·이로운 97.88%·파수 93.1%는 데이터셋·과제·기준이 전부 달라 SGT의 strict micro F1 0.8531과 동일 선상 비교가 성립하지 않는다(D§Q5 경고). 이 이슈가 쓰는 것은 **유형 목록**뿐이다

## Open Questions (미해결 사항)

승인은 됐지만 3-critique.md의 반박은 해소되지 않았다. 아래는 4-decision.md에 ⚠️로 병기된 반박이다.

- **critique R5**: [S] 표기 부정확 — 라벨 추가는 admin↔owasp 설정 왕복(#838) + 임계값 시드 정합(#720) + **설치형이라 수동 DB 마이그레이션 1세트** + #875식 문맥 게이트와 FP 재측정을 동반한다. #875·#833과 겹침 판정 없음
- 체크섬 검증 규칙의 실제 구현 범위가 `미확인`이다. 점검의 첫 단계가 이 확인이다
- 검증숫자(체크섬) 규칙이 없는 유형은 오탐이 늘어 후보 1과 세트로만 안전하다. 후보 1이 #781 트리로 접힌 지금, 이 이슈를 단독 착수하면 오탐 증가분을 무엇으로 받을지 정해져 있지 않다
- 4-decision.md 부기: 출처 검증에서 근거는 일치 판정 — Theori KR 유형 목록·89.0 원문 확인됨

## 예상 규모

**[S]** — 2-plan.md 표기 그대로.

읽는 기준은 4-decision.md에서 승인된 2-plan.md 머리의 정의표다. 개발 2인(owasp+모델 1, 나머지 전부 1) 기준 **달력 기간**이며 설정 전파 양쪽 + DB 마이그레이션 1세트 + 문서 + 재측정을 포함하고, 고객 유지보수 창과 외부 시험기관 일정은 뺀다. [S] = 1주 이내(코드 + 단위/회귀 테스트 + 해당 README 갱신).

⚠️ 이 [S]는 **"점검까지가 [S], 보강은 별건"이라는 전제에서만 성립한다.** 라벨을 실제로 추가하면 Requirements 마지막 항목(수동 DB 마이그레이션 1세트·설정 왕복·시드 정합·FP 재측정)이 붙어 같은 정의로 [M]이다. 착수하면 점검 산출물을 먼저 내고, 보강 여부는 그 표를 보고 별건으로 판단한다.
