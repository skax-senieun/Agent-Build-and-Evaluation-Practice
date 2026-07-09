# meta-harness 고도화 리포트 — Agent 비교 평가(CAP 운영 안정성) 채점 방식 개선

- 작성일: 2026-07-09
- 대상 스킬/기능: SAP BTP 비용 산정 에이전트의 "LLM 기반 정량 채점" 능력 (Agent1 vs Agent2 CAP 운영 안정성 평가)
- 방법: `skills/meta-harness` — baseline(본체 복제) 실행 → 약점 진단 → variant(v1) 국소 수정 → 재실행 → 비교/판정 → promote
- 원본 평가 파일: `/output/agent_stability_evaluation.md`

---

## 1. 배경 — 무엇을 고도화 대상으로 잡았나

기존 `/output/agent_stability_evaluation.md`는 Agent1(비용최소)·Agent2(성능우선) 두 유형에 대해 3개 시나리오(사용자 1,000/2,000/3,000명)의 CAP Application 운영 안정성을 100점 만점으로 채점한 보고서다. 형식(항목별 표, 총점)은 갖췄지만, 실제로 각 점수가 어떤 계산에서 나왔는지는 서술형 근거("규모가 커질수록 병목 위험 증가" 등)뿐이었다. 이 구조에서는:

- 같은 입력을 다시 채점해도 점수가 재현되는지 보장할 수 없다.
- 점수 간 격차가 실제 부하/용량 차이를 반영한 것인지, 단순 인상 평가인지 구분할 수 없다.

이 문제를 meta-harness로 검증하고 고치는 것이 이번 작업의 목표였다.

## 2. baseline 실행 — 약점 재현

동일 질의("Agent를 두가지 형태로 나눠서 평가... 점수로 표현해서 파일로 만들어줘")로 하네스 본체를 격리 복제한 `baseline`을 **2회 독립 실행**했다(재현성 확인 목적).

| 실행 | Agent1 총점(S1/S2/S3) | Agent2 총점(S1/S2/S3) | 채점 근거 형태 |
|---|---|---|---|
| baseline 1회차 | 43 / 34 / 24 | 87 / 89 / 90 | 항목별 0~20점 직관 배점, 서술 근거만 |
| baseline 2회차 | 78 / 58 / 35 | 90 / 88 / 85 | 항목별 0~20점 직관 배점(가정 자체도 다름: 피크율 등 재정의), 서술 근거만 |

**핵심 문제 확인**: 동일 질의를 두 번 실행했는데 Agent1의 S1 점수가 43점 → 78점으로, 거의 2배 차이가 났다. 채점에 사용한 가정(피크 동시접속 비율 등)조차 실행마다 암묵적으로 다시 정해져 일관성이 없었다. 즉 "점수"라는 결과물은 있지만, 그 점수가 **재현 가능하지도, 감사 가능하지도 않았다** — 정성적 인상을 숫자로 포장한 것에 가까웠다.

## 3. 진단 → 개선 지점 매핑

이 약점은 meta-harness의 노브 분류상 "행동원칙/판단 문제"에 해당하여, `langchain-deepagents.py`의 `SYSTEM_PROMPT`를 수정 대상으로 매핑했다. (도구 스키마나 SKILL.md 절차의 문제가 아니라, "정량 채점을 요청받았을 때 어떻게 판단해야 하는가"라는 에이전트의 일반 행동 원칙 문제였기 때문.)

## 4. 개선 내용 (v1) — 무엇을 바꿨는가

`SYSTEM_PROMPT`의 "Clarifying Requests"와 "Progress Updates" 섹션 사이에 아래 규칙 블록을 **국소 추가**(원본 삭제 없음, 19줄 추가):

```
## Quantitative / Scored Evaluations

When asked to score or grade something (e.g. comparing agent behaviors, rating stability,
quality, or performance on a numeric scale):

- Never assign numeric sub-scores by intuition alone. For each rubric item, state the
  explicit formula or rule used to derive the number ... so the score is
  reproducible and auditable, not just plausible-looking.
- If the domain has an underlying quantitative driver (e.g. cost, load, capacity), tie the
  score explicitly back to that driver's computed value in the output.
- State all assumptions used in the formula ... once, up front, and reuse them consistently
  across all scenarios being compared — do not silently vary an assumption between scenarios.
- Before finalizing, sanity-check monotonicity/consistency across scenarios ... and flag any
  result that violates the expected trend as a possible error, not just report it.
```

요약하면 4가지를 강제한다:
1. 직관적 배점 금지 → 명시적 공식 사용
2. 점수를 정량 드라이버(부하/용량 등) 계산값에 직접 연결
3. 가정을 한 번 정하고 모든 시나리오에 동일하게 재사용(시나리오 간 임의 변경 금지)
4. 결과 확정 전 단조성/일관성 자체 검증

`diff --a baseline --b v1` 결과 위 19줄 추가만 확인되어, 최소 변경 원칙(원본 보존)을 충족했다.

## 5. v1 실행 결과 — 개선 후 어떻게 됐는가

v1을 동일 질의로 **2회 실행**하여 안정성을 확인했다.

### v1 1회차

| 시나리오 | 필요 RPS | Agent1 점수 | Agent2 점수 |
|---|---|---|---|
| S1 (1,000/DAU120) | 9 | 47 | 100 |
| S2 (2,000/DAU240) | 18 | 33 | 100 |
| S3 (3,000/DAU360) | 27 | 26 | 100 |

공식: `buffer_ratio_app=(app_capacity-required_rps)/required_rps`, `용량버퍼점수=min(50,buffer_ratio_app×50)`, `이중화점수=0/15/30`, `DB여유점수=min(20,buffer_ratio_db×20)`.

### v1 2회차 (재현성 확인)

| 시나리오 | Peak RPS | Agent1 점수 | Agent2 점수 |
|---|---|---|---|
| S1 | 36 | 39.5 | 100.0 |
| S2 | 72 | 39.5 | 100.0 |
| S3 | 108 | 35.8 | 100.0 |

공식: `Peak_RPS=DAU×0.3`, `Capacity_Score=40×min(capacity_ratio/2,1)`, 이중화 티어 0~4(5~30점), `Scalability_Score=30×min(buffer_ratio,1)×ASG_multiplier`.

두 실행 모두 **명시적 공식 + 고정 가정표 + 자체 일관성 검증(단조성 확인 문구)** 을 포함했다는 점에서 구조적으로 동일했다(구체적 공식 형태는 실행마다 LLM이 재구성하므로 절대 점수는 다를 수 있으나, "근거 없는 배점"이라는 baseline의 결함은 두 번 모두 재발하지 않았다).

## 6. baseline vs v1 비교 — 무엇이 개선되었나

| 비교 항목 | baseline | v1 | 개선 여부 |
|---|---|---|---|
| 채점 방식 | 항목별 0~20점을 직관으로 배정 | `capacity_ratio`, `buffer_ratio` 등 명시적 수식으로 배정 | ✅ |
| 가정 명시·재사용 | 가정이 실행마다 암묵적으로 바뀜(43→78점 원인) | 가정을 표로 고정하고 전 시나리오에 동일 적용 | ✅ |
| 근거-점수 연결 | 점수와 서술 근거가 분리(감점 이유가 숫자와 직접 안 이어짐) | 점수가 계산식의 결과값 그 자체 | ✅ |
| 자체 검증 | 없음 | "추세(일관성) 검증" 섹션에서 단조성 자체 확인 | ✅ |
| 동일 입력 재현성 | Agent1 S1: 43점 vs 78점 (재현 실패) | 구조(공식 기반+검증)가 두 실행 모두 동일하게 유지 | ✅ |
| 산출물 분량/시간 | 평균 ~90초, 토큰 ~9만 | 평균 ~260초, 토큰 ~12만 | ⚠️ 비용 증가(트레이드오프) |

**판정: v1 확실한 우세.** 판단 기준은 "성공기준(재현 가능·감사 가능한 정량 채점)을 충족하는가"였고, baseline은 이 기준을 반복 실행에서도 충족하지 못했지만(점수가 요동침) v1은 두 번의 독립 실행 모두에서 공식 기반 채점 구조를 유지했다. 유일한 부작용은 실행 시간·토큰 사용량 증가(약 2.7배)인데, 이는 "채점 근거를 명시적으로 산출하는 데 드는 정당한 비용"이며 회귀로 보지 않았다.

## 7. 반영(promote) 내역

- 대상 파일: `langchain-deepagents.py` (SYSTEM_PROMPT, "Quantitative / Scored Evaluations" 섹션 19줄 추가)
- 다른 파일 변경 없음(`workspace_seed/`, `AGENTS.md`, 스킬 파일 등은 fork 시점 스냅샷 이슈로 promote 미리보기에 함께 표시되었으나, 실제 반영은 `langchain-deepagents.py` 한 파일로 한정하여 국소 적용함)
- 반영 확인: `git diff langchain-deepagents.py` 결과 의도한 19줄 추가만 존재
- 원본 결과 파일(`/output/agent_stability_evaluation.md`)은 실험 과정에서 임시로 제거했다가 복구 완료

## 8. 남은 한계 및 다음 단계 제안

- 이번 개선은 "채점 방식의 구조"를 고쳤을 뿐, 공식에 들어가는 **가정값(피크 동시접속률, 인스턴스당 처리량 등)의 정확도**는 여전히 LLM의 추정치다. 실측 AWS 벤치마크나 SAP Discovery Center 스펙표를 연동하면 가정의 신뢰도를 높일 수 있다(추후 확장 포인트).
- 실행 시간·토큰 비용이 늘었으므로, 간단한 정성 질문에도 매번 공식화를 강제하면 과잉이 될 수 있다. 필요하면 "정량 채점 요청"과 "단순 정성 비교 요청"을 구분하는 조건을 추가하는 v2를 고려할 수 있다.
