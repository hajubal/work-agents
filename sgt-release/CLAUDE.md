# CLAUDE.md — sgt-release

이 폴더는 **SGT 고객사 배포 패키지 에이전트**의 작업 공간이다. 실행 주체는 루트 `.claude/agents/`의 에이전트 2종이다 — `sgt-rel-packager`가 패키지를 만들고, `sgt-rel-reporter`가 기록과 패키지 폴더 실물을 대조해 결과를 요약 보고한다. **경로는 저장소 루트 기준**이며, 이 폴더 파일은 `sgt-release/` 접두어로 가리킨다.

무엇을 하나: 사람이 현행화한 고객사 helm 사본을 입력으로, 배포에 필요한 이미지 tar·helm·(필요 시) models를 **저장소 밖 패키지 폴더** 한 곳에 모은다. 사람은 그 폴더를 확인하고 반출한다.

## 외부 저장소 참조 규약 ⚠️

| 항목 | 위치 | 권한 |
|---|---|---|
| 고객사 배포 설정 | `~/project/sgt-deployments` (GitHub `ininext/sgt-deployments`, private) | **읽기 전용** |
| 제품 릴리즈 | GitHub `ininext/sgt` 릴리즈 — `gh release ... -R ininext/sgt` | 읽기 전용 |
| 이미지 | `ghcr.io/ininext/<이미지명>` | 읽기 전용. `skopeo login ghcr.io`, 토큰은 `sgt-release/.env`의 `GH_TOKEN` |
| 모델 | `s3://sgt-models/` | 읽기 전용 |
| 패키지 출력 | 아래 고객사 표의 베이스 경로 (저장소 밖) | **쓰기는 여기와 `sgt-release/packages/`뿐** |

- `~/project/sgt` 작업 트리는 기준이 아니다. 버전·차트는 전부 **릴리즈**에서 가져온다. 작업 트리는 미릴리즈 상태일 수 있다(sgt-ctl/VERSION 1.8.0 vs 릴리즈 에셋 1.7.0)
- sgt-deployments의 values에는 라이선스·JWT 시크릿이 들어 있다. 에이전트 보고에 **파일 내용을 싣지 않는다** — 파일명만
- `gh` 명령은 cwd가 이 저장소이므로 `-R ininext/sgt`가 반드시 붙는다

## 고객사 표

에이전트가 정보 수집 단계에서 읽는다. customer-id로 유도할 수 없는 값만 둔다. 새 고객사는 한 줄 추가.

| customer-id | 패키지 베이스 경로 | 서버 아키텍처 | 모델 전달 |
|---|---|---|---|
| hmsec | `/Users/ha/프로젝트/현대자동차증권` | amd64 | baked |

- 패키지 폴더명: `<베이스 경로>/sgt-<릴리즈>-<customer-id>` (예: `sgt-3.2.0-hmsec`, 릴리즈 숫자에서 `v`는 뺀다)
- 모델 전달이 `baked`면 models를 넣지 않는다(이미지에 내장). `hostPath`·`s3`면 `s3://sgt-models/` 전체를 sync
- deployments 폴더는 `~/project/sgt-deployments/customers/<customer-id>-<YYYYMM>`. 같은 customer-id가 여럿이면(`kdn-202605`·`kdn-202607`) **YYYYMM 최신**이 현행이다 — 에이전트는 이것을 제안하고 확인받는다

## 패키지 레이아웃 (고정)

```
sgt-<릴리즈>-<id>/
├── images/
│   ├── <이미지명>-<태그>.tar    # oci-archive, 서버 아키텍처. 예: sgt-owasp-1.13.0-baked.tar
│   └── SHA256SUMS
├── helm/                        # deployments 폴더의 helm/ 그대로
├── .env, .env.<env>             # deployments 폴더 루트의 sgtctl 컨텍스트 그대로
├── sgtctl                       # 릴리즈 에셋 sgtctl-<ver>-linux-<arch> — **사람이 넣는다(에이전트 범위 밖)**
└── models/                      # 모델 전달이 baked가 아닐 때만
```

과거 패키지(2.13.3 루트 tar · 2.14.0 단일 tar.gz · 3.2.0 images/)가 제각각이라 위로 고정한다. sgt `docs/offline-package-guide.md`의 `images/<이미지명>-<태그>.tar` 관례와 같다.

## 이 폴더의 규칙

- **패키지 기록은 `sgt-release/packages/sgt-<릴리즈>-<id>.md` 한 장.** 파일이 있으면 그 패키지는 끝난 것이고, 다음 패키징 때 "이미 전달한 이미지"의 **유일한** 기준이다 — 기록 없는 패키지 폴더의 tar는 이력이 아니다(중단된 실행일 수 있다). 에이전트 이전 수작업 패키지를 이력에 넣으려면 같은 양식의 기록을 손으로 추가한다. 다른 상태 파일은 두지 않는다
- **보고는 파일로 남기지 않는다.** 패키징이 끝나면 `sgt-rel-reporter`가 기록과 실물을 대조해 대화로 요약한다. reporter는 읽기 전용으로 동작한다(Write 도구 없음 + 프롬프트 규칙) — 기록은 위 한 장뿐이다
- **이미지 목록은 values를 파싱하지 않고 렌더로 얻는다.** `.env.<env>`의 `HELM_VALUES_FILES` 순서대로 `helm template`한 결과의 `image:`가 유일한 근거다. values 태그와 GHCR 태그가 다른 사례가 실제로 있다(`v0.18.1-baked` vs `0.18.1-baked`) — 태그가 GHCR에 없으면 추측하지 말고 후보를 보여주고 묻는다
- **사람 단계가 끝났는지는 `helm/Chart.yaml`의 `appVersion`으로 판단한다.** 선택한 릴리즈와 다르면 에이전트는 멈춘다
- 문서를 쓸 때는 [writing-style.md](../writing-style.md)를 따른다

## 사람이 하는 일

### 최초 1회 — 신규 고객사

1. `~/project/sgt-deployments`에 고객사 폴더 생성: `cp -r templates/customer customers/<id>-<YYYYMM>` (규칙은 그 저장소의 `docs/conventions.md`)
2. `helm/`: 릴리즈 차트(`helm-v<릴리즈>.tar.gz` 에셋)를 풀어 넣고, 고객사가 쓰지 않는 `values-*.yaml`은 뺀다. 고객 의존 설정은 `values-<id>.yaml`(+ `values-<env>.yaml`)에
3. 폴더 루트에 `.env`, `.env.<env>`: sgtctl 컨텍스트. `HELM_VALUES_FILES`가 곧 렌더 레이어 순서다 — 반드시 채운다
4. 라이선스: 라이선스 서버에 고객사 등록 → 배포 버전 등록 → env별(dev·prod) 발급 → `values-<env>.yaml`에 기입
5. 시크릿 생성(필요한 것만): JWT secret · DB password · keyfix(`helm/gen-keyfix-values.sh`) · RabbitMQ password
6. 이미지 레지스트리 주소(`global.imageRegistry`): 초기 설치 때 알아와서 기입
7. 위 고객사 표에 한 줄 추가
8. 커밋·PR (`[<id>] <릴리즈> install`)

이미지 export·models 복사는 사람이 하지 않는다 — 에이전트가 GHCR·S3에서 가져온다. **sgtctl은 사람 몫이다**(아래 "에이전트 실행 후").

### 업데이트마다 — 에이전트 호출 전

1. `helm/` 현행화: 새 릴리즈 차트를 `values-*.yaml` 제외하고 통째 복사 → 고객사 README의 전용 수정 재적용 → `values-*.yaml`의 `versions:` 태그 갱신
2. `helm/Chart.yaml`의 `appVersion`이 배포할 릴리즈와 같은지 확인 (에이전트의 게이트)
3. `scripts/render-smoke.sh`로 렌더 확인, 커밋

그다음 "sgt 배포 패키지 만들어"로 에이전트를 부른다.

### 에이전트 실행 후 — sgtctl 넣기

에이전트가 만든 패키지 폴더 루트에 릴리즈 에셋 sgtctl을 사람이 넣는다. 반출 전 마지막 단계다.

```bash
gh release download <태그> -R ininext/sgt -p 'sgtctl-*-linux-<arch>' -O <pkg>/sgtctl --clobber && chmod +x <pkg>/sgtctl
```

## 자주 쓰는 명령

```bash
# 최신 릴리즈
gh release list -R ininext/sgt --limit 3

# 고객사 현행 렌더 이미지 (예: hmsec prod)
cd ~/project/sgt-deployments/customers/hmsec-202607 && helm template sgt ./helm \
  $(sed -n 's/^HELM_VALUES_FILES=//p' .env.prod | tr ':,' '\n' | sed 's/^/-f /') | grep 'image:' | sort -u

# GHCR 태그 확인
gh api '/orgs/ininext/packages/container/sgt-owasp/versions?per_page=10' --jq '.[].metadata.container.tags[]'
```
