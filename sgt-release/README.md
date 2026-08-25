# SGT 배포 패키지 에이전트

🟡 **정의 완료 · 첫 실행 전** — 에이전트 `sgt-rel-packager` 1종. 규칙·고객사 표·사람 체크리스트는 [CLAUDE.md](CLAUDE.md).

사람이 `~/project/sgt-deployments`의 고객사 helm 사본을 릴리즈에 맞춰 현행화하면, 에이전트가 이미지 tar(GHCR→skopeo)·helm·models(baked가 아닐 때, S3)를 고객사 패키지 폴더(저장소 밖)에 모은다. sgtctl은 에이전트가 받지 않는다 — 릴리즈 에셋을 사람이 넣는다. 기록은 `packages/sgt-<릴리즈>-<id>.md` — `ls packages/`가 곧 이력이다.

## 실행

```
sgt 배포 패키지 만들어      # 고객사·릴리즈를 물어보고 진행
hmsec 3.2.0 패키징해
```

## 이 장비 준비물

- `brew install skopeo`
- `sgt-release/.env`에 `GH_TOKEN=<read:packages 권한 토큰>` (gitignore 대상)
- `gh`·`helm`·`aws` 로그인 상태
