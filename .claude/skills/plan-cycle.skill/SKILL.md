---
name: plan-cycle
description: SGT 제품 기획 사이클을 오케스트레이션한다. 브리프 확정 → 시장조사(글로벌/국내 병렬) → 기획 종합 → 비관 검토까지 무인 연속 실행(run-to-gate)하고, 사람이 4-decision.md를 작성하면 로드맵 반영과 이슈 초안 작성을 이어간다. 직전 사이클의 4-decision.md가 없으면 새 사이클 시작을 거부한다. 사용자가 "기획 사이클", "/plan-cycle"을 요청할 때 실행된다.
---

# /plan-cycle — SGT 제품 기획 사이클 오케스트레이션

이 스킬은 `sgt-planning/design.md`의 실행 흐름을 구동한다. **기획 대상 SGT는 별도 저장소(`~/project/sgt`, GitHub `ininext/sgt`)이며 읽기 전용이다** — 산출물은 전부 이 저장소에 쓰고, `gh` 명령에는 `--repo ininext/sgt`를 붙인다. 단계 간 인터페이스는 전부 파일이고, **파일 존재 자체가 상태다** — 상태를 다른 곳에 기록하지 마라.

**문체**: 메인 세션이 쓰는 파일(브리프·4-decision.md 스텁·로드맵 반영분)도 다 쓴 뒤 `humanize-korean` 스킬(im-not-ai)로 윤문한다. 호출할 때 `writing-style.md`의 보존·제외 규칙을 함께 넘기고, **윤문이 수치·URL·인용을 바꿨으면 되돌린다.** 에이전트 4종은 자기 정의 파일의 규칙 0으로 같은 계약을 진다.
⚠️ **이미 다른 문서가 인용 중인 파일은 윤문하지 마라** — 예를 들어 3-critique.md는 2-plan.md의 문장을 그대로 인용하므로, plan을 뒤늦게 윤문하면 그 반박이 허공을 가리킨다. 윤문은 **파일을 처음 쓸 때 그 자리에서** 끝낸다.

## 시작 전 검사 (위반 시 시작 거부)

1. **직전 사이클의 4-decision.md가 없으면 새 사이클을 만들지 마라.** `ls sgt-planning/cycles/`의 최신 폴더(`smoke-*` 폴더는 무시)에 `4-decision.md`가 없으면: "직전 사이클(<id>)이 미결입니다. 2-plan.md와 3-critique.md를 검토하고 4-decision.md를 작성해 주세요"라고 안내하고 중단한다. 이 가드는 읽지 않는 보고서가 쌓이는 것을 막는 핵심 장치다 — 우회하지 마라.
2. **재개 판정**: 최신 사이클 폴더에 4-decision.md 이외의 파일이 일부만 있으면 새 사이클이 아니라 **재개**다. 아래 "파일 → 다음 단계" 표에서 첫 번째로 없는 파일부터 실행하라.

| 없는 첫 파일 | 실행할 단계 |
|---|---|
| 0-brief.md | 0 (브리프 작성) |
| 1-research-global.md / 1-research-domestic.md | 1 (조사 — 없는 쪽만) |
| 2-plan.md | 2 (기획 종합) |
| 3-critique.md | 3 (비관 검토) |
| 4-decision.md | **사람 대기 — 에이전트를 더 돌리지 마라.** 4-decision.md 스텁이 없으면 스텁만 생성하고 안내 후 종료 |
| (4-decision.md 있음, 초안 미작성) | 5~6 (로드맵 반영 + 이슈 초안) |

## 단계 0 — 준비 [사람 터치포인트 A]

1. 사이클 폴더 생성: `sgt-planning/cycles/<YYYY-MM>/` (파일럿은 `<YYYY-MM>-pilot`, 임시 사이클은 `<YYYY-MM>-<슬러그>`).
2. `0-brief.md` 초안을 쓴다. 구조:
   ```markdown
   # <사이클 ID> 브리프
   ## 고객·영업 맥락 (사람 입력 — 최근 고객 요청/문의/입찰 3줄, 없으면 "없음"이라고 명시)
   ## 글로벌 조사 질문 (global scope)
   ## 국내 조사 질문 (domestic scope)
   ## 제한 (검색 상한 등 — 기본: 스코프당 WebSearch 25회)
   ```
   조사 질문 기본값: 매트릭스(`sgt-planning/competitor-matrix.md`)의 빈칸·마지막 확인일이 오래된 행 갱신 + 브리프의 고객 맥락에서 파생되는 질문.
3. **사람에게 브리프 확정을 받는다** — 특히 "고객·영업 맥락" 절은 사람이 직접 채워야 한다(에이전트가 지어낼 수 없는 유일한 ground truth). 대화 중 수정·확정 후 다음 단계로.
4. 인벤토리 증분 갱신: **SGT 저장소에서** `git -C ~/project/sgt log --oneline <inventory의 마지막 갱신 커밋>..HEAD`를 훑어 제품 기능 변화(새 API·모델 교체·배포 프리셋 추가)가 있으면 `sgt-planning/sgt-feature-inventory.md`에 반영하고 머리말의 기준 커밋을 갱신한다.

## 단계 1 — 조사 (병렬)

Agent tool로 `market-researcher`를 **2개 병렬** 실행한다 (한 메시지에 두 호출):

- 호출 1: `brief_path=<cycle>/0-brief.md, scope=global, output_path=<cycle>/1-research-global.md`
- 호출 2: `brief_path=<cycle>/0-brief.md, scope=domestic, output_path=<cycle>/1-research-domestic.md`

**파일럿 사이클 한정**: 두 보고서가 나오면 여기서 멈추고 사람에게 조사 품질 확인을 받는다(브리프 보정용 게이트 — 평시엔 이 게이트 없음, 바로 단계 2로).

## 단계 2 — 기획 종합 (순차)

Agent tool로 `product-planner` 1개 실행: `cycle_dir=<cycle>`, 직전 사이클 4-decision.md가 있으면 `prev_decision_path`로 전달. 산출: `<cycle>/2-plan.md` + `sgt-planning/competitor-matrix.md` 갱신.

## 단계 3 — 비관 검토 (순차, 컨텍스트 격리)

Agent tool로 `red-team-critic` 1개 실행. **planner의 실행 결과·대화 내용을 프롬프트에 넣지 마라 — 파일 경로만 전달한다**: `plan_path=<cycle>/2-plan.md, research_paths=<cycle>/1-research-*.md, output_path=<cycle>/3-critique.md`.

## 단계 4 — 사람 게이트 [사람 터치포인트 B]

`<cycle>/4-decision.md` **스텁을 생성**하고, 2-plan.md·3-critique.md와 함께 검토를 안내한 뒤 **종료한다** (사람이 작성할 때까지 사이클은 열린 상태). 스텁 양식 — critique의 반박을 각 후보 옆에 반드시 병기한다(무비판 승인 방지):

```markdown
# <사이클 ID> 결정 (사람 작성)

## 기능 후보
- [ ] 승인 / [ ] 보류 / [ ] 기각 — **후보 1: <이름>** [규모 X]
  - ⚠️ critique: <이 후보를 겨냥한 반박 요지 (없으면 "반박 없음")>
  - 결정 메모:
(후보 수만큼 반복)

## 로드맵 델타
- [ ] 승인 / [ ] 기각 — <델타 요지> — ⚠️ critique: <해당 반박>

## 미해결 반론 확인
- [ ] 3-critique.md의 반박 5건을 모두 읽었음
```

## 단계 5 — 결정 반영 (4-decision.md 작성 후 재개 시)

1. 4-decision.md에서 승인된 로드맵 델타를 `sgt-planning/roadmap.md`에 반영한다(기계적 반영 — 재해석 금지). 변경 이력에 사이클 ID 한 줄 추가.
2. Agent tool로 `issue-writer` 실행: `decision_path=<cycle>/4-decision.md, plan_path=<cycle>/2-plan.md`. 산출: `sgt-planning/issue-drafts/pending/*.md`.

## 단계 6 — 이슈 생성 (사람 실행 지원)

pending/ 초안 목록을 보여주고, 사람이 확인한 것만 메인 세션이 `gh issue create --repo ininext/sgt --title "<초안 제목>" --body-file <초안> --label product-planning --assignee @me`로 생성한다. 생성 후:
1. 초안 파일 머리에 `> GitHub: #<번호> (YYYY-MM-DD)` 추가
2. `git mv`로 `sgt-planning/issue-drafts/shipped/`로 이동

## 마무리 (매 사이클 공통)

- `<cycle>/9-cost.md`에 단계별 비용을 기록한다: 각 단계 종료 시 `/cost` 출력(또는 최종 보고의 토큰 수)을 사람이 붙여넣도록 스텁을 만들어 둔다.
- `sgt-planning/README.md` 상태 보드의 "현재 Phase/다음 액션"을 갱신한다.
- 커밋은 사람 확인 후: `docs(planning): <사이클 ID> 기획 사이클` 형식.

## 스모크 모드 (Phase 1 드라이런·동작 검증용)

"스모크"로 요청되면: 시작 전 검사를 건너뛰고 `sgt-planning/cycles/smoke-<날짜>/`에 최소 브리프(질문 1개, **검색 상한 3회** 명시)를 만들어 market-researcher 1개(global)만 실행하고, 산출 파일이 규약 위치에 생겼는지 확인 후 결과를 보고한다. 스모크 폴더는 검증 후 삭제해도 된다.
