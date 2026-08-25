# SGT 배포 패키지 에이전트

🟡 **정의 완료 · 첫 실행 전** — 에이전트 2종(`sgt-rel-packager`·`sgt-rel-reporter`). 규칙·고객사 표·사람 체크리스트는 [CLAUDE.md](CLAUDE.md).

사람이 `~/project/sgt-deployments`의 고객사 helm 사본을 릴리즈에 맞춰 현행화하면, 에이전트가 이미지 tar(GHCR→skopeo)·helm·models(baked가 아닐 때, S3)를 고객사 패키지 폴더(저장소 밖)에 모은다. sgtctl은 에이전트가 받지 않는다 — 릴리즈 에셋을 사람이 넣는다. 기록은 `packages/sgt-<릴리즈>-<id>.md` — `ls packages/`가 곧 이력이다.

패키징이 끝나면 `sgt-rel-reporter`가 그 기록과 패키지 폴더 실물을 대조해 무엇이 담겼고 무엇이 빠졌는지 대화로 요약한다. 읽기 전용이라 보고서 파일은 남기지 않는다.

## 실행

```
sgt 배포 패키지 만들어      # 고객사·릴리즈를 물어보고 진행
hmsec 3.2.0 패키징해
```

## 이 장비 준비물

- `brew install skopeo`
- `sgt-release/.env`에 `GH_TOKEN=<read:packages 권한 토큰>` (gitignore 대상)
- `gh`·`helm`·`aws` 로그인 상태
