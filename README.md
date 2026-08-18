# 제품 기획 에이전트 시스템

개발 2인 체제에서 공백인 시장 리서치·경쟁사 비교·로드맵·기능 기획을 Claude Code 에이전트 파이프라인으로 보완한다. **이 저장소가 시스템의 전체 기록이며, 이 README가 유일한 상태 문서다** (상태를 다른 곳에 중복 기록하지 않는다).

기획 대상인 SGT 제품은 **별도 저장소**(`~/project/sgt` · GitHub `ininext/sgt`)에 있고 **읽기 전용**이다. 참조 규약은 [CLAUDE.md](CLAUDE.md)에 있다.

## 상태 보드

| 항목 | 값 |
|---|---|
| 현재 Phase | **Phase 2 — 파일럿 사이클 `cycles/2026-08-12-pilot/` 진행 중. 4-decision.md 작성 완료 → 로드맵 반영·이슈 초안까지 끝, 실제 이슈 생성 대기** |
| 다음 액션 | 팀 회의([5-meeting-agenda.md](cycles/2026-08-12-pilot/5-meeting-agenda.md)) → 이슈·코멘트 생성 → Phase 3 회고 |
| 다음 액션 보조 | pending/ 초안 6건 대기 — 신규 이슈 2건은 생성 가능, 코멘트 4건은 담당자 확인 후 |
| SGT 저장소 | `~/project/sgt` (GitHub `ininext/sgt`) — 읽기 전용 |

## 재개 절차 (세션이 끊겼을 때)

1. 이 README의 상태 보드를 읽는다
2. `ls cycles/` 최신 폴더에서 **첫 번째로 없는 파일**이 다음 할 일이다 (파일 존재 자체가 상태)
3. `4-decision.md`가 없으면 사람 검토 대기 상태다 — 에이전트를 더 돌리지 말 것

## Phase 개요

| Phase | 내용 | 게이트(사람 검토) | 상태 |
|---|---|---|---|
| 0 | 차터+설계 (charter.md, design.md) | 게이트 1: 두 문서 승인 | ✅ 2026-08-12 |
| 1 | 구현 — 에이전트 4종 + /plan-cycle 스킬 + living doc 초기화 | 게이트 2: 드라이런 확인 | ✅ 2026-08-12 |
| 2 | 파일럿 1사이클 (cycles/2026-08-12-pilot/) | 사이클 내 게이트 3~4회 | 🔵 4-decision.md 대기 |
| 3 | 회고·운영 전환 (retro.md, ops.md) + develop PR | 게이트 3: 운영 여부 결정 | ⚪ |

## 문서 인덱스

| 문서 | 설명 | 생성 Phase |
|---|---|---|
| [charter.md](charter.md) | 목적·비목표·현실성 평가·운영 원칙·성공 지표·비용 | 0 |
| [design.md](design.md) | 에이전트 roster·파일 계약·/plan-cycle 흐름·게이트 정의 | 0 |
| [references.md](references.md) | 유사 에이전트 운용 사례·연구 근거 조사 (설계 가정의 지지/반박) | 0 |
| roadmap.md | 제품 로드맵 living doc — **팀이 결정한 항목만**, 사이클 승인으로만 갱신 | 1 |
| idea-pool.md | 검증 안 된 후보 풀 (구 xlsx 로드맵 — AI 임의 생성 판명으로 강등) | 1 |
| competitor-matrix.md | 경쟁사 기능 매트릭스 living doc — 회차 누적 핵심 자산 | 1 |
| sgt-feature-inventory.md | SGT 자체 기능 스냅샷 (매트릭스 SGT 열의 원천) | 1 |
| cycles/<회차>/ | 회차별 기획 사이클 기록 (brief→research→plan→critique→decision) | 2~ |
| issue-drafts/pending/ | 승인된 기능 후보의 이슈 초안 대기열 | 2~ |
| issue-drafts/shipped/ | gh 이슈 생성 완료분 (파일 머리에 이슈 번호 기록) | 2~ |
| ops.md | 운영 주기·게이트 규칙 (파일럿 회고 후 확정) | 3 |
