# 업무 에이전트

회사 업무를 Claude Code 에이전트로 보완하는 저장소다. 에이전트마다 최상위 폴더 하나를 갖고, 그 폴더가 해당 에이전트의 문서·산출물·상태를 전부 담는다. 에이전트·스킬 정의만 루트 `.claude/`에 모여 있다.

| 에이전트 | 폴더 | 상태 | 진입점 |
|---|---|---|---|
| SGT 기획 에이전트 | [sgt-planning/](sgt-planning/README.md) | 🔵 파일럿 사이클 진행 중 | `/plan-cycle` |
| SGT 배포 패키지 에이전트 | [sgt-release/](sgt-release/README.md) | 🟡 정의 완료 · 첫 실행 전 | "sgt 배포 패키지 만들어" |

각 에이전트의 현재 상태는 그 폴더의 README에 있다. 여기에 중복 기록하지 않는다.

## 구조

```
.claude/agents/     서브에이전트 정의 (전 에이전트 공용, 루트에만 둔다)
.claude/skills/     스킬 정의
writing-style.md    문체 규칙 — 에이전트 공용
<에이전트>/          폴더 하나 = 에이전트 하나 (CLAUDE.md · README.md · 산출물)
```

규약과 새 에이전트 추가 절차는 [CLAUDE.md](CLAUDE.md)에 있다.
