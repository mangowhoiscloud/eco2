# Eco²: RSI roadmap and current scope

[한국어](#한국어) · [English](#english) · [Landing](https://mangowhoiscloud.github.io/eco2/#rsi-roadmap)

## 한국어

### 현재 위치

Eco²는 **작업 내 답변 개선과 운영 관측 기반을 구현한 제품 시스템**입니다.
개발 과정에서 에이전트가 문서·Skills·코드 변경을 남기는 것과, 제품이 스스로
개선 전략을 바꾸는 것은 별개의 주장입니다. 현재는 후자를 입증한 RSI 시스템으로
표시하지 않습니다. 서비스는 종료됐으며, 아래는 2026-09-21에 확인한 공개 코드의
범위입니다. 새 실험이나 운영 중인 클러스터의 검증 결과가 아닙니다.

기준 문헌은 Duan et al., [The Last AI Built by Humans: Toward Genuine Recursive
Self-Improvement](https://arxiv.org/abs/2609.11873v2), v2 (2026-09-15)입니다.
§1.4·Figure 2, §2.2, §3을 용어와 연구 방향의 가이드로 사용합니다.
아래 대응은 프로젝트 자체 해석이며, 논문 저자의 Eco² 평가나 인증이 아닙니다.

### 같은 이름을 다른 수준으로 읽지 않기

| 표현 | Eco²에서 뜻하는 것 | 뜻하지 않는 것 |
|---|---|---|
| B0 / task-local refinement | 현재 답변을 평가하고 제한된 횟수로 재생성 | 다음 독립 과제의 agent 정책이 개선됐다는 증거 |
| Eval tier 1 / 2 / 3 | Code grader / LLM grader / calibration monitor. 기존 소스의 `L1/L2/L3` | RSI의 개선 자율성 L1–L3 |
| Infrastructure layer | Edge / Service / Integration / Persistence / Platform | RSI의 L1–L5 |
| System state | 대화·job 복구 상태와 개선되어 계승되는 정책 revision을 구분 | Redis checkpoint나 로그가 있다는 이유만으로 학습 완료 |
| Experience | 다음 변경에 실제 사용된 평가·실패·운영 기록 | 수집했지만 후속 변경에서 읽지 않은 telemetry |
| Improver / strategy | 현재는 개발자와 개발 에이전트가 변경 후보와 방법을 선택 | KEDA autoscaling이나 ArgoCD sync가 개선 전략을 발명 |
| Verifier / acceptance | 답변 grader, 테스트, 변경 검토가 각각 자기 범위를 검사 | 답변 점수 하나가 코드 병합과 배포를 승인 |
| Improvement / successor | 채택된 변경 revision을 이후 실행이 이어받음 | fallback 응답 하나를 생성하거나 job을 재개한 것 |
| Reward | Scan의 캐릭터 지급 판단 | 학습용 reward model 또는 agent 개선의 성공 지표 |

기존 API·schema·설정 필드 이름은 바꾸지 않습니다. 공개 설명에는 `eval tier`,
`infrastructure layer`, `RSI level`을 붙여 서로 다른 분류임을 밝힙니다.

### 단계별 로드맵

| RSI 범위 | 현재 코드 근거 | 다음 검증 조건 |
|---|---|---|
| B0 · 작업 내 수정 | Chat graph에 answer → eval → answer/END 분기가 있습니다. 평가·재생성 flag와 서비스 주입에 따라 활성화됩니다. | 활성 설정·grader 결과·재생성 전후 답변을 한 실행에 연결 |
| L1 · 개선 실행 | 개발 문서·Skills·CI·GitOps는 사람이 정한 변경 절차의 기반입니다. 제품의 자율 개선 수준을 입증하지는 않습니다. | 실패 → 후보 diff → 테스트 → 채택 revision → 다음 실행의 연결 기록 |
| L2 · 개선 전략 | 운영자·개발 에이전트의 진단과 제품 런타임을 구분합니다. 자동 후보 탐색·채택의 성능 실험은 이 감사에서 확인하지 않았습니다. | 고정 평가·동일 예산에서 고정 전략 대비 후보 선택 효과 측정 |
| L3 · 경험 선택 | 이벤트·평가·trace 수집은 기반입니다. 다음 학습 경험을 시스템이 선택한다는 뜻은 아닙니다. | 실패 표본 선택 정책, 제외 근거, held-out 평가를 분리 |
| L4 · 배포 적응 | KEDA·State KV·GitOps는 자원 조정·복구·선언 상태 동기화입니다. 운영 경험에서 새로운 agent 정책을 채택한 증거와 다릅니다. | 운영 신호 → 검토된 영속 변경 → 이후 독립 작업의 이득·회귀 추적 |
| L5 · 개선 메커니즘 계승 | 연구 방향입니다. improver·verifier 자체를 개선하고 계승한 효과를 주장하지 않습니다. | 변경된 개선 메커니즘을 다음 라운드가 사용했는지와 독립 평가 성능 검증 |

### 코드에서 확인한 경계

감사 revision: [`eco2-team/backend@a072127`](https://github.com/eco2-team/backend/tree/a0721271ac569679f1e4f19dd0745ab634a0c115).

1. [Chat graph factory](https://github.com/eco2-team/backend/blob/a0721271ac569679f1e4f19dd0745ab634a0c115/apps/chat_worker/infrastructure/orchestration/langgraph/factory.py)는 평가 flag가 켜진 경우에만 eval subgraph를 연결합니다.
2. [EvalConfig](https://github.com/eco2-team/backend/blob/a0721271ac569679f1e4f19dd0745ab634a0c115/apps/chat_worker/application/dto/eval_config.py)의 기본값은 `enable_eval_pipeline=False`, `eval_regeneration_enabled=False`입니다. 구현 존재를 기본 활성화나 운영 실측으로 서술하지 않습니다.
3. [Eval subgraph](https://github.com/eco2-team/backend/blob/a0721271ac569679f1e4f19dd0745ab634a0c115/apps/chat_worker/infrastructure/orchestration/langgraph/eval_graph_factory.py)는 C등급·flag 활성·retry count 조건으로 재생성을 최대 한 번 요청합니다. Grader 미주입 시 passthrough 경로가 있으므로 route 이름 `pass`를 검증 통과와 동일하게 읽지 않습니다.
4. [서비스 구조와 운영 기록](https://github.com/eco2-team/backend/blob/a0721271ac569679f1e4f19dd0745ab634a0c115/README.md)은 Redis Streams·Pub/Sub·State KV, 관측 스택, GitOps와 서비스 종료 상태를 설명합니다. 기록의 수집·재전송이 학습 정책을 수정하는 경로는 아닙니다.

### 다음에 검증할 최소 단위

운영 실패 유형 하나를 선택해 고정된 재현 입력과 독립 평가를 먼저 만듭니다.
개발 에이전트의 후보 변경, 사람의 검토, 테스트 결과, 채택된 revision을 연결하고
다음 실행에서 그 revision이 사용됐는지 확인합니다. 동일 예산 대조군에서 이득과
회귀를 측정한 뒤에야 전략 자율성의 주장을 넓힙니다. 이 항목은 계획이며 실행 결과가 아닙니다.

[GEODE의 동일 기준 로드맵](https://mangowhoiscloud.github.io/geode/docs/explanation/rsi-roadmap/)
과 비교하되, GEODE의 후보 탐색 구현을 Eco² 제품 기능으로 옮겨 적지 않습니다.

## English

### Current scope

Eco² implements **task-local answer refinement and operational observation infrastructure**.
Developer-agent code, documents, and Skills are distinct from a product autonomously revising
its improvement strategy. We do not claim the latter has been demonstrated. The service is
closed; this is a public-source audit dated 2026-09-21, not a new experiment or a live-cluster result.

The design reference is Duan et al., [The Last AI Built by Humans: Toward Genuine Recursive
Self-Improvement](https://arxiv.org/abs/2609.11873v2), v2 (2026-09-15), §1.4/Figure 2,
§2.2 and §3. The mapping is our interpretation, not an assessment or certification by the authors.

### Terminology and roadmap

| Scope | Current evidence | Next evidence needed |
|---|---|---|
| B0 · In-task refinement | Conditional answer → eval → answer/END path | Bind enabled configuration, actual grader results, and before/after answers |
| L1 · Improvement execution | Developer instructions, Skills, CI and GitOps support externally specified changes | Connect a failure, candidate diff, tests, acceptance and next-run revision |
| L2 · Improvement strategy | Operator/developer-agent diagnosis is separate from product execution; automated search performance was not established in this audit | Compare candidate selection with a fixed strategy at equal budget and evaluation |
| L3 · Experience acquisition | Events, evaluations and traces supply observation infrastructure | Test failure-sampling decisions separately from held-out evaluation |
| L4 · Deployment adaptation | Scaling, state recovery and GitOps sync are operational controls, not evidence of a learned agent-policy update | Trace deployment feedback into a reviewed persistent update and later independent results |
| L5 · Recursive inheritance | Research direction, not a demonstrated capability | Show reuse of a revised improvement mechanism and gains on independent tests |

The source’s eval `L1/L2/L3` mean **code grader, LLM grader, calibration monitor**.
Infrastructure layers describe deployment topology. Neither is the RSI autonomy scale.
Conversation/job state supports continuity; an inherited policy revision supports an improvement
claim. Stored logs become experience for this analysis only when a later update consumes them.
An improver proposes changes; a strategy selects how to search. A verifier and acceptance rule
judge a bounded result. CI integration, candidate acceptance, and production deployment are
different decisions. Scan’s character reward is a product incentive, not an RL reward model.
Existing API and schema names remain unchanged.

### Source boundaries and next experiment

The pinned source links above reference `eco2-team/backend@a072127`:

- The Chat factory attaches evaluation conditionally. `EvalConfig` defaults both evaluation
  and regeneration to **off**; code presence does not prove deployment activation.
- The eval subgraph requests at most one regeneration when its grade, flag and retry conditions
  permit it. Missing grader injections can take passthrough paths. A route named `pass` is not
  by itself a successful verification receipt.
- Redis Streams, Pub/Sub, State KV, observability and GitOps deliver, inspect, recover or
  synchronize work. They do not by themselves demonstrate autonomous agent-policy learning.

Next, select one reproducible failure family, freeze inputs and independent evaluation, then
link candidate, human review, test result and the accepted revision to the next execution.
Measure gains and regressions against an equal-budget control. This is a proposed experiment,
not an observed outcome. [GEODE’s roadmap](https://mangowhoiscloud.github.io/geode/docs/explanation/rsi-roadmap/?lang=en)
uses the same criteria without implying that Eco² includes GEODE’s search implementation.
