# CLAUDE.md — sgt-planning

이 폴더는 **SGT 제품 기획 에이전트**의 작업 공간이다. 코드는 없고 전부 마크다운이며, 실행 주체는 저장소 루트 `.claude/`의 에이전트 4종(`market-researcher`·`product-planner`·`red-team-critic`·`issue-writer`)과 `/plan-cycle` 스킬이다. **경로는 저장소 루트 기준이다** — 이 폴더 파일을 가리킬 때는 `sgt-planning/` 접두어를 붙인다.

시작점은 [README.md](README.md)의 상태 보드다. 시스템의 목적과 원칙은 [charter.md](charter.md), 실행 명세는 [design.md](design.md)에 있다.

## SGT 저장소 참조 규약 ⚠️

기획 대상 제품인 SGT는 **별도 저장소**에 있다. 이 저장소는 그 코드를 읽기만 한다.

| 항목 | 값 |
|---|---|
| 로컬 경로 | `~/project/sgt` (이 저장소와 형제 폴더) |
| GitHub | `ininext/sgt` |
| 권한 | **읽기 전용 — SGT 파일을 수정하지 마라.** 기획 산출물은 전부 이 저장소에 쓴다 |

- **SGT 코드·문서 읽기**: 절대 경로로 접근한다 (`~/project/sgt/sgt-owasp/src/...`). 상대 경로 `../sgt/`도 같은 곳이지만, 세션 cwd가 어디든 흔들리지 않는 절대 경로를 권한다
- **SGT 이슈 조회·생성**: cwd가 SGT가 아니므로 `gh` 명령에 **`--repo ininext/sgt`가 반드시 붙는다.** 빠뜨리면 이 저장소를 대상으로 잡거나 실패한다
- **인벤토리 증분 갱신**: `git -C ~/project/sgt log --oneline <기준커밋>..HEAD`

SGT 저장소를 못 찾으면 추측해서 진행하지 말고 경로를 물어라. 기획 산출물의 "SGT 현재" 항목은 저장소 실측이 근거이므로, 못 읽었으면 `미확인`으로 남기는 것이 정답이다.

## 이 폴더의 규칙

- **파일 존재 자체가 상태다.** 진행 상황을 별도로 기록하지 마라 — `ls cycles/<최신>/`에서 **첫 번째로 없는 파일**이 다음 할 일이다
- **`4-decision.md`는 사람 전용.** 없으면 사람 검토 대기 상태이며, 에이전트를 더 돌리지 않는다
- **`roadmap.md`는 팀이 결정한 것만.** 에이전트의 직접 수정 금지 — 갱신 경로는 `4-decision.md` 승인뿐이다
- **이슈 생성은 사람만.** `issue-writer`에 gh 접근 도구가 없는 것이 그 안전장치다
- 문서를 쓸 때는 [writing-style.md](../writing-style.md)를 따른다 (`humanize-korean` 스킬 + 보존·제외 규칙)

## 자주 쓰는 명령

```bash
# 기획 사이클 실행 (Claude Code 안에서)
/plan-cycle

# 재개 지점 확인
ls sgt-planning/cycles/$(ls sgt-planning/cycles | grep -v smoke | tail -1)/

# SGT 백로그 조회
gh issue list --repo ininext/sgt --state open --limit 100
```
