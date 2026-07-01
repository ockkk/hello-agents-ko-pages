# 11장 Agentic-RL

## 11.1 LLM 교육에서 Agentic RL까지

이전 장에서는 다양한 에이전트 패러다임과 통신 프로토콜을 구현했습니다. 그러나 에이전트가 더 복잡한 작업을 처리하면 성능이 저하되어 자연스럽게 다음과 같은 질문이 제기됩니다. **에이전트가 어떻게 더 강력한 추론 능력을 갖도록 만들 수 있습니까? 상담원이 도구를 더 잘 사용하는 방법을 배우도록 하려면 어떻게 해야 하나요? 상담원의 자기 개선 능력을 어떻게 만들 수 있나요?**

이것이 바로 Agentic RL(강화 학습 기반 에이전트 훈련)이 해결하려는 핵심 문제입니다. 이 장에서는 추론 및 도구 사용과 같은 고급 기능을 사용하여 에이전트를 교육할 수 있도록 HelloAgents 프레임워크에 강화 학습 교육 기능을 소개합니다. LLM 교육의 기본부터 시작하여 SFT(Supervised Fine-Tuning) 및 GRPO(Group Relative Policy Optimization)와 같은 실용적인 기술을 점차적으로 탐구하여 궁극적으로 완전한 에이전트 교육 파이프라인을 구축할 것입니다.

11.1.1 강화 학습에서 Agentic RL까지

2장의 섹션 2.4.2에서는 강화 학습을 기반으로 하는 에이전트를 소개했습니다. 강화 학습(RL)은 순차적인 의사 결정 문제를 해결하는 데 초점을 맞춘 학습 패러다임입니다. 에이전트와 환경 간의 직접적인 상호 작용을 통해 장기적인 보상을 극대화하는 방법을 배우고 "시행 착오"를 통해 학습합니다.

이제 이 프레임워크를 LLM 에이전트에 적용해 보겠습니다. 다음과 같은 질문에 대답해야 하는 수학적 문제 해결 에이전트를 고려해보세요.

```
Question: Janet's ducks lay 16 eggs per day. She eats three for breakfast
every morning and bakes muffins for her friends every day with four.
She sells the remainder at the farmers' market daily for $2 per fresh
duck egg. How much in dollars does she make every day at the farmers' market?
```

이 문제에는 다단계 추론이 필요합니다. 먼저 Janet이 매일 남긴 계란의 수를 계산하고(16 - 3 - 4 = 9) 수입을 계산합니다(9 × 2 = 18). 이 작업을 강화 학습 프레임워크에 매핑할 수 있습니다.

- **에이전트**: LLM 기반 추론 시스템
- **환경**: 수학 문제 및 검증 시스템
- **상태**: 현재 문제 설명 및 기존 추론 단계
- **Action**: 다음 추론 단계 또는 최종 답변 생성
- **보상**: 정답 여부(맞음 +1, 틀림 0)

기존 지도 학습 방법에는 세 가지 핵심 제한 사항이 있습니다. 첫째, 데이터 품질이 훈련 품질을 완전히 결정하고 모델은 훈련 데이터만 모방할 수 있어 이를 능가하기 어렵습니다. 둘째, 탐색 능력이 부족하여 인간이 제공하는 수동적인 학습 경로만 제공됩니다. 셋째, 장기 목표를 최적화하는 데 어려움이 있으며 다단계 추론의 중간 프로세스를 정확하게 최적화할 수 없습니다.

강화 학습은 새로운 가능성을 제공합니다. 에이전트가 여러 후보 답변을 자동으로 생성하고 정확성에 따라 보상을 받을 수 있도록 함으로써 어떤 추론 경로가 더 좋고 어떤 단계가 중요한지 알 수 있으며 인간 주석<sup>[8]</sup>보다 더 나은 문제 해결 방법을 발견할 수도 있습니다. 이것이 Agentic RL의 핵심 아이디어입니다. LLM을 학습 가능한 정책으로 취급하고 이를 에이전트의 인식-결정-실행 루프에 삽입하고 강화 학습을 통해 다단계 작업 성능을 최적화하는 것입니다.

### 11.1.2 LLM 교육 환경

Agentic RL에 대해 알아보기 전에 먼저 LLM 교육의 전체 프로세스를 이해해야 합니다. 강력한 LLM(예: GPT, Claude, Qwen)의 탄생은 일반적으로 사전 교육과 사후 ​​교육이라는 두 가지 주요 단계를 거칩니다. 그림 11.1에서 볼 수 있듯이 이 두 단계는 "언어 모델"에서 "대화 도우미"로의 LLM의 완전한 진화 경로를 구성합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-1.png" alt="" width="85%"/>
  <p>그림 11.1 LLM 교육 환경</p>
</div>

**사전 학습 단계**는 LLM 학습의 첫 번째 단계로, 모델이 기본적인 언어 패턴과 세계 지식을 학습하도록 하는 것을 목표로 합니다. 이 단계에서는 대량의 텍스트 데이터(보통 TB 수준)를 사용하고 이전 컨텍스트에서 다음 단어를 예측하는 등 텍스트 자체에서 훈련 신호가 구성되는 자기 지도 학습을 통해 모델을 훈련합니다. 가장 일반적인 사전 훈련 작업은 다음 토큰 예측이라고도 알려진 인과 언어 모델링입니다.

텍스트 시퀀스 $x_1, x_2, ..., x_t$가 주어지면 모델은 다음 단어 $x_{t+1}$를 예측해야 합니다.

$$
\mathcal{L}_{\text{사전 훈련}} = -\sum_{t=1}^{T} \log P(x_t | x_1, x_2, ..., x_{t-1}; \theta)
$$

여기서 $\theta$는 모델 매개변수이고, $P(x_t | x_1, ..., x_{t-1}; \theta)$는 모델이 예측한 다음 단어의 확률 분포이며, 목표는 음의 로그 우도를 최소화하는 것, 즉 올바른 단어를 예측할 확률을 최대화하는 것입니다. 예를 들어, "The cat sat on the"라는 텍스트가 주어지면 모델은 다음 단어가 "mat"일 가능성이 가장 높다고 예측해야 합니다. 방대한 양의 텍스트에 대한 훈련을 통해 모델은 문법 규칙(어떤 단어 순서가 합법적인지), 의미 지식(단어 간의 관계), 세계 지식(세계에 대한 사실 정보) 및 기본 추론 능력을 점차적으로 학습합니다.

사전 훈련 단계의 특징은 대규모 데이터 볼륨, 높은 계산 비용, 일반적인 언어 이해 및 생성 기능 학습, 수동으로 레이블이 지정된 작업 데이터가 아닌 레이블이 없는 텍스트로 구성된 자체 감독 목표 사용입니다.

**사후 학습 단계**는 사전 학습된 모델의 단점을 해결하는 것을 목표로 합니다. 사전 학습된 모델은 강력한 언어 기능을 갖추고 있지만 "다음 단어 예측" 모델일 뿐이며 인간의 지시를 따르고, 유용하고 무해하며 정직한 답변을 생성하고, 부적절한 요청을 거부하고, 대화 방식으로 인간과 상호 작용하는 방법을 모릅니다. 훈련 후 단계에서는 이러한 문제를 해결하고 모델을 인간의 선호도 및 가치에 맞추는 것을 목표로 합니다.

훈련 후에는 일반적으로 세 단계가 포함됩니다. 첫 번째 단계는 모델이 지침과 대화 형식을 따르는 방법을 학습하도록 하는 것을 목표로 하는 **감독 미세 조정(SFT)**<sup>[15]</sup>입니다. 훈련 데이터는 (프롬프트, 완료) 쌍으로 구성되며 훈련 목표는 사전 훈련과 유사하지만 여전히 올바른 출력 확률을 최대화합니다.

$$
\mathcal{L}_{\text{SFT}} = -\sum_{i=1}^{N} \log P(y_i | x_i; \theta)
$$

여기서 $x_i$는 입력 프롬프트이고 $y_i$는 예상 출력이며 $N$은 훈련 샘플 수입니다. SFT 특성은 다음과 같습니다. 데이터 양이 적고 수동 주석이 필요하며 빠른 결과가 필요하며 주로 작업 형식 및 기본 기능을 학습합니다.

두 번째 단계는 **Reward Modeling(RM)**입니다. SFT 모델은 지침을 따를 수 있지만 생성된 답변의 품질은 다양합니다. 답변의 질을 평가할 수 있는 방법이 필요하며, 이는 보상 모델<sup>[13,14]</sup>의 역할입니다. 보상 모델 훈련 데이터는 동일한 질문에 대한 두 가지 답변, 즉 더 나은 답변(선택)과 더 나쁜 답변(거절)을 포함하는 선호도 비교 데이터로 구성됩니다. 보상 모델 훈련 목표는 인간의 선호도를 학습하는 것입니다.

$$
\mathcal{L}_{\text{RM}} = -\mathbb{E}_{(x, y_w, y_l)} [\log \sigma(r_\phi(x, y_w) - r_\phi(x, y_l))]
$$

여기서 $r_\phi(x, y)$는 보상 모델이고 입력은 (프롬프트, 답변) 쌍, 출력은 품질 점수입니다. $y_w$는 더 나은 답변(선택), $y_l$은 더 나쁜 답변(거절), $\sigma$는 시그모이드 함수이며, 목표는 보상 모델이 더 나은 답변에 더 높은 점수를 주도록 만드는 것입니다.

세 번째 단계는 **강화 학습 미세 조정**입니다. 보상 모델을 사용하면 강화 학습을 사용하여 언어 모델을 최적화하여 더 높은 품질의 답변을 생성할 수 있습니다. 가장 고전적인 알고리즘은 PPO(Proximal Policy Optimization)<sup>[1]</sup>이며 훈련 목표는 다음과 같습니다.

$$
J_{\text{PPO}} = \mathbb{E}_{x, y \sim \pi_\theta} [r_\phi(x, y)] - \beta \cdot D_{KL}(\pi_\theta || \pi_{\text{ref}})
$$

여기서 $\pi_\theta$는 현재 정책, 즉 언어 모델이고, $\pi_{\text{ref}}$는 참조 정책이며, 이 시나리오에서는 SFT 모델이 될 수 있으며, $r_\phi(x, y)$는 보상 모델 점수, $D_{KL}$는 모델이 너무 멀리 벗어나는 것을 방지하기 위한 KL 발산, $\beta$는 균형 계수입니다. 이 목적 함수의 의미는 원래 모델에서 너무 멀리 벗어나지 않으면서 보상을 최대화한다는 것입니다.

전통적인 RLHF(인간 피드백을 통한 강화 학습)<sup>[5]</sup>에는 많은 양의 수동 선호 데이터 주석이 필요하며 이는 비용이 많이 듭니다. 비용을 절감하기 위해 연구원들은 강력한 AI 모델(예: GPT-4)을 사용하여 인간 주석자를 대체하는 RLAIF(Reinforcement Learning from AI Feedback)<sup>[7]</sup>을 제안했습니다. RLAIF 워크플로우는 SFT 모델을 사용하여 여러 후보 답변 생성, 강력한 AI 모델을 사용하여 답변 점수 및 순위 지정, AI 점수를 사용하여 보상 모델 훈련, 강화 학습을 위한 보상 모델 사용입니다. 실험에 따르면 RLAIF의 효율성은 RLHF에 가깝거나 심지어 이를 초과하는 동시에 비용이 크게 절감됩니다<sup>[11]</sup>.

### 11.1.3 Agentic RL의 핵심 철학

LLM의 기본 훈련 과정을 이해한 후 Agentic RL과 기존 훈련 방법의 차이점을 살펴보겠습니다. PBRFT: Preference-Based Reinforcement Fine-Tuning이라고 하는 전통적인 사후 훈련은 주로 단일 전환 대화 품질을 최적화하는 데 중점을 둡니다. 즉, 사용자 질문이 있으면 모델이 답변을 생성한 다음 답변 품질에 따라 보상을 받습니다. 이 접근 방식은 대화 보조자를 최적화하는 데 적합하지만 다단계 추론, 도구 사용 및 장기 계획이 필요한 에이전트 작업에는 부족합니다.

**Agentic RL**은 LLM을 순차적 의사결정 루프에 포함된 학습 가능한 정책으로 취급하는 새로운 패러다임입니다. 이 프레임워크에서 에이전트는 동적 환경에서 외부 세계와 상호 작용하고, 복잡한 작업을 완료하기 위해 다단계 작업을 실행하고, 후속 결정을 안내하기 위한 중간 피드백을 얻고, 단일 단계 보상이 아닌 장기 누적 보상을 최적화해야 합니다.

구체적인 예를 통해 이 차이점을 이해해 보겠습니다. PBRFT 시나리오에서 사용자가 "강화 학습이 무엇인지 설명해 주세요"라고 질문하면 모델이 완전한 답변을 생성한 다음 답변 품질에 따라 직접 점수를 매깁니다. Agentic RL 시나리오에서 사용자가 "이 GitHub 저장소의 코드 품질을 분석하는 데 도움을 주세요"라고 요청하면 에이전트는 여러 단계를 거쳐야 합니다. 먼저 GitHub API를 호출하여 저장소 정보를 얻고, 저장소 구조와 파일 목록을 성공적으로 얻고, +0.1 보상을 받습니다. 그런 다음 메인 코드 파일을 읽고, 코드 내용을 성공적으로 획득하고, +0.1 보상을 받습니다. 코드 품질을 합리적으로 분석하고 +0.2 보상을 받으세요. 최종적으로 고품질의 분석 보고서를 생성하고 +0.6 보상을 받으세요. 총 보상은 모든 단계의 누적입니다: 1.0.

볼 수 있듯이 Agentic RL의 주요 기능은 다단계 상호 작용, 각 작업이 환경 상태를 변경하고 각 단계에서 피드백을 받을 수 있으며 전체 작업 완료 품질을 최적화하는 것입니다.

강화 학습은 일반적으로 MDP(Markov Decision Process) 프레임워크를 통해 공식화됩니다. MDP는 상태 공간 $S$, 행동 공간 $A$, 상태 전이 함수 $P(s'|s,a)$, 보상 함수 $R(s,a)$, 할인 계수 $\gamma$ 등 5개의 튜플 $(S, A, P, R, \gamma)$로 정의됩니다. 아래 표에서는 MDP 관점에서 PBRFT와 Agentic RL을 비교합니다.

<div align="center">
  <p>표 11.1 PBRFT와 Agentic RL</p>의 비교
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-1.png" alt="" width="85%"/>
</div>

상태 측면에서 PBRFT의 상태 $s_0$는 사용자 프롬프트로만 구성되며 시간 범위 $T=1$(단일 단계), 상태는 변경되지 않으며 $s_0 = \text{prompt}$로 나타낼 수 있습니다. Agentic RL의 상태 $s_t$에는 기록 관찰 및 컨텍스트가 포함되어 있지만 시간 범위 $T \gg 1$(다단계), 상태는 작업과 함께 발전하고 $s_t = (\text{prompt}, o_1, o_2, ..., o_t)$로 표시될 수 있습니다. 여기서 $o_t$는 $t$ 단계의 관찰입니다(예: 도구 반환 결과, 환경 피드백 등).

액션 측면에서 PBRFT의 액션 공간에는 $a = y \sim \pi_\theta(y|s_0)$로 표시되는 텍스트 생성, 단일 액션 유형만 있습니다. Agentic RL의 작업 공간에는 $a_t \in \{a_t^{\text{text}}, a_t^{\text{tool}}\}$로 표시되는 텍스트 생성, 도구 호출, 환경 작업 및 기타 유형이 포함되어 있지만, 예를 들어 $a_t^{\text{text}}$는 사고 프로세스 또는 답변을 생성하고 $a_t^{\text{tool}}$는 계산기, 검색 엔진 및 기타 도구를 호출합니다.

전환 기능 측면에서 PBRFT에는 $P(s'|s,a) = \delta(s' - s_{\text{terminal}})$로 표시되는 상태 전환이 없습니다. Agentic RL의 상태는 $s_{t+1} \sim P(s_{t+1}|s_t, a_t)$로 표시되는 작업 및 환경에 따라 동적으로 변경되지만, 예를 들어 검색 도구를 호출한 후 상태에는 검색 결과가 포함됩니다.

보상 측면에서 PBRFT는 작업 종료 시에만 제공되는 단일 단계 보상 $r(s_0, a)$만 가지며 $R_{\text{PBRFT}} = r(s_0, y)$로 표시되며 일반적으로 보상 모델 $r(s_0, y) = r_\phi(s_0, y)$로 제공됩니다. Agentic RL에는 다단계 보상 $r(s_t, a_t)$이 있지만 다음과 같이 중간 단계에서 부분 보상을 제공할 수 있습니다.

$$
R_{\text{에이전트}} = \sum_{t=0}^{T} \gamma^t r(s_t, a_t)
$$

$\gamma \in [0,1]$이 할인 요인인 경우, $r(s_t, a_t)$는 희소 보상(정답 +1과 같이 작업 완료 시에만 제공됨), 조밀한 보상(성공적인 도구 호출 +0.1과 같이 각 단계에서 제공됨) 또는 둘의 조합일 수 있습니다.

목적 함수 측면에서 PBRFT는 단일 단계 기대 보상을 극대화합니다.

$$
J_{\text{PBRFT}}(\theta) = \mathbb{E}_{s_0, y \sim \pi_\theta} [r(s_0, y)]
$$

Agentic RL은 누적 할인 보상을 극대화합니다.

$$
J_{\text{Agentic}}(\theta) = \mathbb{E}_{\tau \sim \pi_\theta} \left[\sum_{t=0}^{T} \gamma^t r(s_t, a_t)\right]
$$

여기서 $\tau = (s_0, a_0, s_1, a_1, ..., s_T)$는 전체 궤적입니다.

이러한 변화는 단지 기술적인 세부 사항의 차이가 아니라 사고의 근본적인 변화입니다. PBRFT 사고는 "모델이 더 나은 단일 답변을 생성하는 방법"에 초점을 맞추고 답변 품질을 최적화하며 언어 표현에 중점을 두고 단일 단계 결정을 내립니다. Agentic RL 사고는 "에이전트가 복잡한 작업을 완료하도록 하는 방법"에 초점을 맞추고 작업 완료를 최적화하며 실행 전략에 집중하고 다단계 계획을 세웁니다. 이러한 변화를 통해 LLM은 "대화 도우미"에서 "자율 에이전트"로 발전할 수 있으며, 적극적으로 정보를 검색하고, 외부 도구를 언제 어떻게 사용할지 알고, 궁극적인 목표를 위해 겉보기에 "우회"하는 중간 단계를 기꺼이 실행하고, 실수로부터 배울 수 있습니다.

Agentic RL은 그림 11.2에 표시된 대로 LLM 에이전트에 6가지 핵심 기능을 부여하는 것을 목표로 합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-2.png" alt="" width="85%"/>
  <p>그림 11.2 Agentic RL</p>의 6가지 핵심 기능
</div>

**추론**은 주어진 정보로부터 논리적으로 결론을 도출하는 과정을 말하며, 이는 에이전트의 핵심 역량입니다. 전통적인 CoT 프롬프트 방법은 일반화 능력이 제한된 소수의 예시에 의존합니다. SFT는 훈련 데이터의 추론 패턴만 모방할 수 있어 혁신이 어렵습니다. 강화학습의 장점은 시행착오를 통해 효과적인 추론 전략을 학습하고, 훈련 데이터에 없는 추론 경로를 발견하고, 깊은 사고가 필요할 때와 빠른 답변이 가능한 때를 학습하는 것입니다. 추론 작업은 순차적 결정 문제로 모델링될 수 있습니다. 질문 $q$가 주어지면 에이전트는 추론 체인 $c = (c_1, c_2, ..., c_n)$과 최종 답변 $a$를 생성해야 합니다. 보상 함수는 일반적으로 훈련 목표 $\max_\theta \mathbb{E}_{q, (c,a) \sim \pi_\theta} [r(q, c, a)]$를 사용하여 $a = a^*$ else $0$인 경우 $r(q, c, a) = 1$로 설계됩니다. 이 접근 방식을 통해 모델은 단순히 답변을 암기하는 것이 아니라 고품질 추론 체인을 생성하는 방법을 학습합니다.

**도구 사용**은 작업을 완료하기 위해 외부 도구를 호출하는 에이전트의 기능을 의미합니다. 도구 사용 작업에서 행동 공간은 $a_t \in \{a_t^{\text{think}}, a_t^{\text{tool}}\}$로 확장됩니다. 여기서 $a_t^{\text{think}}$는 사고 과정을 생성하고 $a_t^{\text{tool}} = (\text{tool\_name}, \text{arguments})$는 도구를 호출합니다. 강화 학습을 통해 에이전트는 언제 도구를 사용해야 하는지, 어떤 도구를 선택해야 하는지, 여러 도구를 결합하는 방법을 배울 수 있습니다. 예를 들어, 수학 문제를 풀 때 상담원은 언제 계산기를 사용해야 하는지, 언제 코드 해석기를 사용해야 하는지, 언제 직접 추론해야 하는지 배워야 합니다.

**메모리**는 에이전트가 과거 정보를 유지하고 재사용하는 능력을 말하며 이는 장기적인 작업에 매우 중요합니다. LLM의 컨텍스트 창은 제한되어 있으며 RAG와 같은 정적 검색 전략은 작업에 최적화될 수 없습니다. 강화 학습을 통해 에이전트는 기억할 가치가 있는 정보, 메모리 업데이트 시기, 오래된 정보 삭제 시기 등을 결정하는 메모리 관리 전략을 학습할 수 있습니다. 이는 인간의 작업 기억과 유사합니다. 즉, 우리는 뇌의 정보를 적극적으로 관리하여 중요한 정보를 유지하고 관련 없는 정보는 잊어버립니다.

**계획**은 목표를 달성하기 위해 행동 순서를 공식화하는 능력을 말합니다. 전통적인 CoT는 선형적 사고 방식이므로 되돌릴 수 없습니다. 프롬프트 엔지니어링은 새로운 상황에 적응하기 어려운 정적 계획 템플릿을 사용합니다. 강화 학습을 통해 에이전트는 시행착오를 통해 효과적인 작업 순서를 발견하고 단기 및 장기 이점의 균형을 맞추는 방법을 학습하는 동적 계획을 학습할 수 있습니다. 예를 들어, 다단계 작업에서 에이전트는 최종적으로 작업을 완료하기 전에 정보 수집과 같이 외관상 "우회" 단계를 먼저 실행해야 할 수도 있습니다.

**자체 개선**은 에이전트가 자체 출력을 검토하고 오류를 수정하며 전략을 최적화하는 능력을 의미합니다. 강화 학습을 통해 에이전트는 자신의 오류 식별, 실패 원인 분석, 전략 조정 등 자기 성찰을 학습할 수 있습니다. 이 기능을 통해 에이전트는 인간의 "실수로부터 학습"하는 것과 유사하게 인간의 개입 없이 지속적으로 개선할 수 있습니다.

**지각**은 다양한 정보를 이해하는 능력을 말합니다. 예를 들어, 강화 학습은 시각적 추론 능력을 향상시켜 모델이 시각적 도구를 사용하고 시각적 계획을 배울 수 있도록 해줍니다. 이를 통해 에이전트는 텍스트를 이해할 뿐만 아니라 시각적 세계에서도 이해하고 작업할 수 있습니다.

### 11.1.4 HelloAgents의 Agentic RL 설계

Agentic RL의 핵심 철학을 이해한 후 HelloAgents 프레임워크에서 이러한 기능을 구현하는 방법을 살펴보겠습니다.

기술 선택 측면에서는 TRL(Transformer Reinforcement Learning) 프레임워크<sup>[9]</sup>을 통합하고 Qwen3-0.6B 모델<sup>[10]</sup>을 선택했습니다. TRL은 Hugging Face의 강화 학습 라이브러리로, 성숙하고 안정적이며 기능이 완벽하고 통합이 쉽습니다. Qwen3-0.6B는 Alibaba Cloud의 소규모 언어 모델로, 일반 GPU 훈련에 적합한 0.6B 매개변수, 뛰어난 성능, 오픈 소스 및 무료를 갖추고 있습니다.

HelloAgents의 Agentic RL 모듈은 그림 11.3과 같이 4계층 아키텍처 설계를 채택합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-3.png" alt="" width="85%"/>
  <p>그림 11.3 HelloAgents Agentic RL 아키텍처</p>
</div>

맨 아래 계층은 **데이터 세트 계층**으로, `GSM8KDataset` 클래스, `create_sft_dataset()` 함수 및 `create_rl_dataset()` 함수를 포함하며 데이터 로드 및 형식 변환을 담당합니다. 두 번째 레이어는 **보상 함수 레이어**로, `MathRewardFunction` 기본 클래스, `AccuracyReward` 정확도 보상, `LengthPenaltyReward` 길이 페널티, `StepReward` 단계 보상, 그리고 좋은 행동이 무엇인지 정의하는 편리한 생성 함수 `create_*_reward()`을 포함합니다. 세 번째 레이어는 **트레이너 레이어**입니다. `SFTTrainerWrapper` 및 `GRPOTrainerWrapper`이 포함되어 있으며 특정 트레이닝 로직과 LoRA 지원을 담당합니다. 최상위 계층은 **통합 인터페이스 레이어**로, `RLTrainingTool` 통합 훈련 도구를 제공하고 `action="train"`(훈련 모델), `action="load_dataset"`(데이터 세트 로드), `action="create_reward"`(보상 기능 생성), `action="evaluate"`(모델 평가)의 네 가지 작업을 지원합니다.

### 11.1.5 빠른 시작 예

학습에 들어가기 전에 전체 훈련 과정을 빠르게 경험해 봅시다. 이번 장에는 이론적인 내용이 많고 실제적인 디버깅이 상당히 지루하기 때문에 도구를 구성하기보다는 적용하는 방법을 배우는 데 중점을 둡니다. 먼저 HelloAgents 프레임워크를 설치합니다.

```bash
# Install HelloAgents framework (Chapter 11 version)
pip install "hello-agents[rl]==0.2.5"

# Or install from source
cd HelloAgents
pip install -e ".[rl]"
```

그런 다음 빠른 훈련 예제를 실행하십시오.

```python
import sys
import json

from hello_agents.tools import RLTrainingTool

# Create RL training tool
rl_tool = RLTrainingTool()

# 1. Quick test: SFT training (10 samples, 1 epoch)
sft_result_str = rl_tool.run({
    "action": "train",
    "algorithm": "sft",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/quick_test_sft",
    "max_samples": 10,      # Only use 10 samples for quick test
    "num_epochs": 1,        # Only train 1 epoch
    "batch_size": 2,
    "use_lora": True        # Use LoRA to accelerate training
})

sft_result = json.loads(sft_result_str)
print(f"\n✓ SFT training completed, model saved at: {sft_result['output_dir']}")

# 2. GRPO training (5 samples, 1 epoch)
grpo_result_str = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",  # Use base model
    "output_dir": "./models/quick_test_grpo",
    "max_samples": 5,       # Only use 5 samples for quick test
    "num_epochs": 1,
    "batch_size": 2,        # Must be divisible by num_generations(8), use 2
    "use_lora": True
})

grpo_result = json.loads(grpo_result_str)
print(f"\n✓ GRPO training completed, model saved at: {grpo_result['output_dir']}")

# 3. Evaluate model
eval_result_str = rl_tool.run({
    "action": "evaluate",
    "model_path": "./models/quick_test_grpo",
    "max_samples": 10,      # Evaluate on 10 test samples
    "use_lora": True
})

eval_result = json.loads(eval_result_str)
print(f"\n✓ Evaluation completed:")
print(f"  - Accuracy: {eval_result['accuracy']}")
print(f"  - Average reward: {eval_result['average_reward']}")
print(f"  - Test samples: {eval_result['num_samples']}")

print("\n" + "=" * 50)
print("🎉 Congratulations! You have completed training your first Agentic RL model!")
print("=" * 50)
print(f"\nModel paths:")
print(f"  SFT model: {sft_result['output_dir']}")
print(f"  GRPO model: {grpo_result['output_dir']}")
```

이 빠른 예는 전체 훈련 프로세스를 보여줍니다. SFT 훈련을 통해 모델은 기본 추론 형식과 대화 패턴을 학습할 수 있고, GRPO 훈련은 강화 학습을 통해 추론 전략을 최적화하여 정확성을 향상시키며, 모델 평가는 테스트 세트에 대한 훈련 효과를 평가합니다. 또한 모델이 훈련 샘플의 0.7%만 확인했고 한 에포크 동안만 실행되었기 때문에 실행 후 정확도가 매우 낮은 것이 일반적입니다.

## 11.2 데이터세트와 보상 함수

데이터 세트와 보상 함수는 강화 학습 훈련의 두 가지 초석입니다. 데이터 세트는 에이전트가 학습해야 하는 작업을 정의하고 보상 기능은 좋은 행동이 무엇인지 정의합니다. 이번 섹션에서는 훈련 데이터를 준비하고 보상 함수를 설계하는 방법을 알아봅니다.

### 11.2.1 GSM8K 수학적 추론 데이터 세트

수학적 추론은 LLM 추론 능력을 평가하는 데 이상적인 작업입니다. 첫째, 수학 문제에는 수동 주석이나 복잡한 보상 모델 없이 자동으로 평가할 수 있는 명확한 정답이 있습니다. 둘째, 수학 문제를 해결하려면 문제 분해와 단계별 도출이 필요하며 이는 다단계 추론의 일반적인 시나리오입니다. 마지막으로, 학습된 추론 능력은 강력한 일반화를 통해 다른 영역으로 이전될 수 있습니다. 이와 대조적으로 개방형 Q&A 작업(예: "프로그래밍을 배우는 방법")은 객관적으로 평가하기 어렵고 광범위한 수동 주석이 필요한 답변 품질을 갖습니다.

GSM8K(Grade School Math 8K)<sup>[4]</sup>은 고품질 초등학교 수학 단어 문제 데이터세트입니다. 표 11.2에서 볼 수 있듯이 데이터 세트에는 7,473개의 훈련 샘플과 1,319개의 테스트 샘플이 포함되어 있으며 초등학교 수학 수준(2~8학년)에서는 어려움이 있고 문제 유형은 단어 문제이므로 답에 도달하기 위해 2~8단계의 추론이 필요합니다.

<div align="center">
  <p>표 11.2 GSM8K 데이터 세트 통계</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-2.png" alt="" width="85%"/>
</div>

일반적인 GSM8K 문제를 살펴보겠습니다.

```
Question: Natalia sold clips to 48 of her friends in April, and then she sold half
      as many clips in May. How many clips did Natalia sell altogether in April
      and May?

Answer: Natalia sold 48/2 = <<48/2=24>>24 clips in May.
      Natalia sold 48+24 = <<48+24=72>>72 clips altogether in April and May.
      #### 72

Final Answer: 72
```

이 문제에는 두 단계의 추론이 필요합니다. 먼저 5월에 판매된 수량(48개의 절반)을 계산한 다음 총계(4월 + 5월)를 계산합니다. 답변의 `@@PH0@@>`은 중간 계산 단계를 나타내는 표시이고, `#### 72`은 최종 답변을 표시합니다.

그림 11.4에 표시된 것처럼 GSM8K 데이터 세트는 다양한 훈련 방법에 적응하기 위해 다양한 형식으로 변환되어야 합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-4.png" alt="" width="85%"/>
  <p>그림 11.4 GSM8K 데이터 형식 변환</p>
</div>


원본 형식은 사람이 읽기에 적합한 질문과 답변(해결 단계 포함)을 포함하는 데이터 세트에서 직접 제공됩니다. SFT 형식은 감독된 미세 조정에 사용되며 질문을 대화 형식 프롬프트로 변환하고 완전한 솔루션을 완성합니다. 예를 들면:

```python
{
    "prompt": "<|im_start|>user\nNatalia sold clips to 48 of her friends...<|im_end|>\n<|im_start|>assistant\n",
    "completion": "Let me solve this step by step.\n\nStep 1: ...\n\nFinal Answer: 72<|im_end|>"
}
```

핵심 사항은 모델의 대화 템플릿(예: Qwen의 `@@PH0@@` 마커)을 사용하고, 프롬프트에는 사용자 질문이 포함되고, 완료에는 전체 솔루션 프로세스와 답변이 포함된다는 것입니다. 이러한 방식으로 모델은 출력 형식을 지정하는 방법과 단계별로 추론하는 방법을 학습할 수 있습니다.

강화학습에는 RL 형식이 사용되며, 문제 해결 과정이 아닌 질문과 정답만 제공됩니다. 예를 들면:

```python
{
    "prompt": "<|im_start|>user\nNatalia sold clips to 48 of her friends...<|im_end|>\n<|im_start|>assistant\n",
    "ground_truth": "72"
}
```

핵심 사항은 프롬프트가 SFT와 동일하지만 ground_truth에는 최종 답(보상 계산에 사용됨)만 포함되어 있으며 모델은 완전한 추론 프로세스 자체를 생성해야 합니다. 이 설계는 모델이 단순히 답을 암기하는 것이 아니라 자율적인 추론을 학습하도록 합니다.

표 11.3에 표시된 대로 세 가지 형식에는 각각 용도가 있습니다.

<div align="center">
  <p>표 11.3 데이터 형식 비교</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-3.png" alt="" width="85%"/>
</div>

HelloAgents는 편리한 데이터세트 로딩 기능을 제공합니다. 코드를 통해 데이터세트를 로드하고 살펴보겠습니다.

```python
from hello_agents.tools import RLTrainingTool
import json

# Create tool
rl_tool = RLTrainingTool()

# 1. Load SFT format dataset
sft_result = rl_tool.run({
    "action": "load_dataset",
    "format": "sft",
    "max_samples": 5  # Only load 5 samples to view
})
sft_data = json.loads(sft_result)

print(f"Dataset size: {sft_data['dataset_size']}")
print(f"Data format: {sft_data['format']}")
print(f"Sample keys: {sft_data['sample_keys']}")

# 2. Load RL format dataset
rl_result = rl_tool.run({
    "action": "load_dataset",
    "format": "rl",
    "max_samples": 5
})
rl_data = json.loads(rl_result)

print(f"Dataset size: {rl_data['dataset_size']}")
print(f"Data format: {rl_data['format']}")
print(f"Sample keys: {rl_data['sample_keys']}")
```

볼 수 있듯이 SFT 형식에는 지도 학습을 위한 완전한 솔루션 프로세스가 포함되어 있습니다. RL 형식에는 최종 답변만 포함되며 모델은 추론 프로세스 자체를 생성해야 합니다. `max_samples` 매개변수는 로드된 샘플 수를 제어하므로 빠른 테스트에 편리합니다.

### 11.2.2 보상 기능 설계

보상 기능은 "좋은 행동"이 무엇인지 정의하는 강화 학습의 핵심입니다. 좋은 보상 기능은 에이전트가 올바른 전략을 배우도록 안내할 수 있는 반면, 나쁜 보상 기능은 훈련 실패 또는 잘못된 행동 학습으로 이어질 수 있습니다.

강화 학습에서 보상 함수 $r(s, a)$ 또는 $r(s, a, s')$는 에이전트의 각 행동에 수치적 보상을 할당합니다. 에이전트의 목표는 누적 보상을 최대화하는 것입니다.

$$
J(\theta) = \mathbb{E}_{\tau \sim \pi_\theta} \left[\sum_{t=0}^{T} \gamma^t r(s_t, a_t)\right]
$$

수학적 추론 작업의 경우 다음과 같이 단순화할 수 있습니다.

$$
r(q, a) = f(a, a^*)
$$

여기서 $q$는 질문이고, $a$는 모델에서 생성된 답변이고, $a^*$는 정답이고, $f$는 평가 함수입니다.

보상 기능 설계는 훈련 효과에 직접적인 영향을 미칩니다. 좋은 보상 함수는 성공이 무엇인지 명확하게 정의하고, 기울기 신호를 제공하며, 과도한 분산을 생성하지 않고, 조정 및 결합이 쉬워야 합니다. 열악한 보상 기능은 중간 피드백 없이 작업 종료 시에만 보상을 제공할 수 있고, 에이전트가 높은 보상을 얻기 위해 "속임수" 방법을 찾는 보상 해킹이 있거나, 여러 가지 상충되는 목표가 있거나, 수렴을 방해하는 과도한 분산이 있을 수 있습니다.

HelloAgents는 그림 11.5와 같이 개별적으로 또는 조합하여 사용할 수 있는 세 가지 기본 보상 기능을 제공합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-5.png" alt="" width="85%"/>
  <p>그림 11.5 보상 기능 설계</p>
</div>

**(1) 정확도 보상**

정확도 보상(AccuracyReward)은 가장 기본적인 보상 기능으로, 정답 여부에만 신경을 씁니다. 수학적 정의:

$$
r_{\text{acc}}(a, a^*) = \begin{cases}
1 & \text{if } a = a^* \\
0 & \text{그렇지 않으면}
\end{사례}
$$

여기서 $a$는 모델에 의해 생성된 답이고 $a^*$는 정답입니다. 이는 정답에 대해 1점을 얻고 오답에 대해 0점을 얻는 이진 보상 함수입니다.

구현을 위해서는 답변 추출 및 비교를 처리해야 합니다. 모델 출력에는 많은 양의 텍스트가 포함될 수 있으므로 최종 답을 추출해야 합니다. 일반적인 추출 방법에는 "최종 답변:" 뒤의 숫자 찾기, "####" 표시 뒤의 숫자 찾기, 정규식을 사용하여 마지막 숫자 추출 등이 있습니다. 답변 비교는 숫자 정밀도(예: 72.0과 72는 동일한 것으로 간주되어야 함), 단위 변환(예: 1000 및 1k) 및 형식 차이(예: "72" 및 "seventy-two")를 처리해야 합니다.

사용 예:

```python
from hello_agents.tools import RLTrainingTool
import json
rl_tool = RLTrainingTool()

# Create accuracy reward function
reward_result = rl_tool.run({
    "action": "create_reward",
    "reward_type": "accuracy"
})
reward_data = json.loads(reward_result)

print(f"Reward type: {reward_data['reward_type']}")
print(f"Description: {reward_data['description']}")

# Note: The create_reward operation of RLTrainingTool returns configuration information,
# the actual reward function will be automatically created and used during training
```

출력:

```json
Prediction: 72, Ground truth: 72, Reward: 1.0
Prediction: 72.0, Ground truth: 72, Reward: 1.0
Prediction: 73, Ground truth: 72, Reward: 0.0
```

정확도 보상의 장점: 간단하고 직접적이며 이해 및 구현이 쉽고 명확한 정답이 있는 작업에 적합합니다. 단점: 보상이 희박하고 완전히 정답인 경우에만 보상을 받을 수 있으며 "정답에 가까움"과 "완전히 틀림"을 구분할 수 없으며 초기 교육에서 효과적인 피드백이 부족할 수 있습니다.

**(2) 길이 페널티**

길이 페널티(LengthPenaltyReward)는 모델이 장황함을 피하면서 간결한 답변을 생성하도록 권장합니다. 수학적 정의:

$$
r_{\text{길이}}(a, a^*, l) = r_{\text{acc}}(a, a^*) - \alpha \cdot \max(0, l - l_{\text{대상}})
$$

여기서 $l$은 생성된 텍스트의 길이(문자 수 또는 토큰 수)이고 $l_{\text{target}}$는 대상 길이이며 $\alpha$는 페널티 계수(기본값 0.001)입니다. 길이 페널티는 답변이 올바른 경우에만 적용되며, 페널티를 줄이기 위해 모델이 잘못된 짧은 답변을 생성하는 것을 방지합니다.

설계 근거: 답변이 틀리면 보상은 0입니다(길이에 관계 없음). 대답이 정확하고 길이가 합리적이면 보상은 1입니다. 답이 정확하지만 너무 길면 보상은 $1 - \alpha \cdot (l - l_{\text{target}})$입니다. 예를 들어, 목표 길이가 200자, 실제 길이가 500자, 페널티 계수가 0.001이면 보상은 $1 - 0.001 \times (500 - 200) = 0.7$입니다.

사용 예:

```python
# Create length penalty reward function
reward_result = rl_tool.run({
    "action": "create_reward",
    "reward_type": "length_penalty",
    "max_length": 1024,      # Maximum length
    "penalty_weight": 0.001  # Penalty weight
})
reward_data = json.loads(reward_result)

print(f"Reward type: {reward_data['reward_type']}")
print(f"Description: {reward_data['description']}")
print(f"Max length: {reward_data['max_length']}")
print(f"Penalty weight: {reward_data['penalty_weight']}")
```

출력:

```
Prediction: 72, Ground truth: 72, Length: 50, Reward: 1.000
Prediction: 72, Ground truth: 72, Length: 200, Reward: 1.000
Prediction: 72, Ground truth: 72, Length: 500, Reward: 0.700
Prediction: 73, Ground truth: 72, Length: 50, Reward: 0.000
```

길이 페널티의 장점: 간결한 표현을 장려하고, 모델이 중복되는 콘텐츠를 생성하는 것을 방지하며, 추론 비용을 제어할 수 있습니다(출력이 짧을수록 토큰 소비가 적음을 의미함). 단점: 자세한 추론을 억제할 수 있고 페널티 계수를 신중하게 조정해야 하며 최적의 길이는 작업마다 크게 다릅니다.

**(3) 단계 보상**

단계 보상(StepReward)은 모델이 명확한 추론 단계를 생성하도록 장려하여 해석 가능성을 향상시킵니다. 수학적 정의:

$$
r_{\text{단계}}(a, a^*, s) = r_{\text{acc}}(a, a^*) + \beta \cdot s
$$

여기서 $s$는 감지된 추론 단계 수이고 $\beta$는 단계 보상 계수(기본값 0.1)입니다. 마찬가지로, 답이 맞는 경우에만 걸음 보상이 주어집니다.

단계 감지 방법에는 "1단계:", "2단계:" 마커 찾기, 개행 문자 계산, 추론 패턴 일치를 위한 정규식 사용 등이 포함됩니다. 예를 들어, 명확한 3단계의 정답은 $1 + 0.1 \times 3 = 1.3$의 보상을 받습니다.

사용 예:

```python
# Create step reward function
reward_result = rl_tool.run({
    "action": "create_reward",
    "reward_type": "step",
    "step_bonus": 0.1  # 0.1 reward per step
})
reward_data = json.loads(reward_result)

print(f"Reward type: {reward_data['reward_type']}")
print(f"Description: {reward_data['description']}")
print(f"Step bonus: {reward_data['step_bonus']}")
```

출력:

```
Prediction: 72, Ground truth: 72, Steps: 0, Reward: 1.00
Prediction: 72, Ground truth: 72, Steps: 2, Reward: 1.20
Prediction: 72, Ground truth: 72, Steps: 5, Reward: 1.50
Prediction: 73, Ground truth: 72, Steps: 5, Reward: 0.00
```

단계 보상의 장점: 해석 가능한 추론을 장려하고, 생성된 답변을 확인 및 디버그하기가 더 쉽고, 모델이 체계적인 사고를 학습하는 데 도움이 됩니다. 단점: 모델이 더 많은 보상을 얻기 위해 중복 단계를 생성하도록 유도할 수 있고 단계 수량과 답변 품질의 균형을 맞춰야 하며 단계 감지가 부정확할 수 있습니다.

실제 적용에서 우리는 일반적으로 다양한 목표의 균형을 맞추기 위해 여러 보상 기능을 결합합니다. 일반적인 조합 전략은 다음과 같습니다.

**정확도 + 길이 페널티**: 대화 시스템 및 Q&A 시스템에 적합한 간결한 정답을 권장합니다. 공식:

$$
r = r_{\text{acc}} - \alpha \cdot \max(0, l - l_{\text{대상}})
$$

**정확도 + 단계 보상**: 교육 시나리오 및 설명 가능한 AI에 적합한 상세한 추론 프로세스를 장려합니다. 공식:

$$
r = r_{\text{acc}} + \beta \cdot s
$$

**3방향 균형**: 답변 품질, 간결성 및 해석 가능성을 종합적으로 최적화합니다. 공식:

$$
r = r_{\text{acc}} - \alpha \cdot \max(0, l - l_{\text{대상}}) + \beta \cdot s
$$

하나의 목표가 지나치게 지배적이지 않도록 가중치 $\alpha$ 및 $\beta$를 주의 깊게 조정해야 합니다.

사용 예:

```python
# Combined reward function: accuracy + length penalty + step reward
# Note: RLTrainingTool currently supports single reward type
# Combined rewards need to be specified through reward_fn parameter in training configuration
# This shows how to configure different types of reward functions

# Accuracy reward
accuracy_result = rl_tool.run({
    "action": "create_reward",
    "reward_type": "accuracy"
})
print("Accuracy reward:", json.loads(accuracy_result)['description'])

# Length penalty reward
length_result = rl_tool.run({
    "action": "create_reward",
    "reward_type": "length_penalty",
    "max_length": 1024,
    "penalty_weight": 0.001
})
print("Length penalty reward:", json.loads(length_result)['description'])

# Step reward
step_result = rl_tool.run({
    "action": "create_reward",
    "reward_type": "step",
    "step_bonus": 0.1
})
print("Step reward:", json.loads(step_result)['description'])
```

출력:

```
Combined reward: 1.200
  - Accuracy: 1.0
  - Length penalty: -0.100
  - Step reward: +0.3
```

표 11.4에서 볼 수 있듯이 다양한 보상 기능은 다양한 애플리케이션 시나리오에 적합합니다.

<div align="center">
  <p>표 11.4 보상 기능 비교</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-4.png" alt="" width="85%"/>
</div>

### 11.2.3 사용자 정의 데이터세트 및 보상 함수

HelloAgents는 GSM8K 데이터 세트와 일반적인 보상 기능을 제공하지만 실제 애플리케이션에서는 자체 데이터 세트를 사용하거나 특정 보상 기능을 설계해야 할 수도 있습니다. 이 섹션에서는 프레임워크를 확장하는 방법을 소개합니다.

사용자 정의 데이터 세트를 사용하기 전에 두 가지 훈련 형식에 대한 데이터 요구 사항을 이해해야 합니다.

**SFT 형식**: 감독된 미세 조정에 사용되며 다음 필드를 포함해야 합니다.
- `prompt`: 입력 프롬프트(시스템 및 사용자 메시지 포함)
- `completion`: 예상되는 출력
- `text`: 완전한 대화 텍스트(선택 사항)

**RL 형식**: 강화 학습에 사용되며 다음 필드를 포함해야 합니다.
- `question`: 원래 질문
- `prompt`: 입력 프롬프트(시스템 및 사용자 메시지 포함)
- `ground_truth` : 정답
- `full_answer`: 완전한 답변(추론 과정 포함)

**(1) format_math_dataset로 변환**

가장 간단한 방법은 `question` 및 `answer` 필드가 포함된 원시 데이터를 준비한 다음 자동 변환을 위해 `format_math_dataset()` 기능을 사용하는 것입니다.

```python
from datasets import Dataset
from hello_agents.rl import format_math_dataset

# 1. Prepare raw data
custom_data = [
    {
        "question": "What is 2+2?",
        "answer": "2+2=4. #### 4"
    },
    {
        "question": "What is 5*3?",
        "answer": "5*3=15. #### 15"
    },
    {
        "question": "What is 10+7?",
        "answer": "10+7=17. #### 17"
    }
]

# 2. Convert to Dataset object
raw_dataset = Dataset.from_list(custom_data)

# 3. Convert to SFT format
sft_dataset = format_math_dataset(
    dataset=raw_dataset,
    format_type="sft",
    model_name="Qwen/Qwen3-0.6B"
)
print(f"SFT dataset: {len(sft_dataset)} samples")
print(f"Fields: {sft_dataset.column_names}")

# 4. Convert to RL format
rl_dataset = format_math_dataset(
    dataset=raw_dataset,
    format_type="rl",
    model_name="Qwen/Qwen3-0.6B"
)
print(f"RL dataset: {len(rl_dataset)} samples")
print(f"Fields: {rl_dataset.column_names}")
```

**(2) 사용자 정의 데이터세트 직접 전달**

RLTrainingTool을 사용하면 `custom_dataset` 매개변수를 통해 사용자 정의 데이터세트를 직접 전달할 수 있습니다.

```python
from hello_agents.tools import RLTrainingTool

rl_tool = RLTrainingTool()

# SFT training
result = rl_tool.run({
    "action": "train",
    "algorithm": "sft",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/custom_sft",
    "num_epochs": 3,
    "batch_size": 4,
    "use_lora": True,
    "custom_dataset": sft_dataset  # Directly pass custom dataset
})

# GRPO training
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/custom_grpo",
    "num_epochs": 2,
    "batch_size": 2,
    "use_lora": True,
    "custom_dataset": rl_dataset  # Directly pass custom dataset
})
```

**(3) 사용자 정의 데이터세트 등록(권장)**

여러 번 사용해야 하는 데이터세트의 경우 등록을 권장합니다.

```python
# 1. Register dataset
rl_tool.register_dataset("my_math_dataset", rl_dataset)

# 2. Use registered dataset
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "dataset": "my_math_dataset",  # Use registered dataset name
    "output_dir": "./models/custom_grpo",
    "num_epochs": 2,
    "use_lora": True
})
```

보상 함수는 모델에서 생성된 답변의 품질을 평가하는 데 사용됩니다. 사용자 정의 보상 함수는 다음 서명을 따라야 합니다.

```python
from typing import List
import re

def custom_reward_function(
    completions: List[str],
    **kwargs
) -> List[float]:
    """
    Custom reward function

    Args:
        completions: List of completion texts generated by the model
        **kwargs: Other parameters, typically including:
            - ground_truth: List of correct answers
            - Other dataset fields

    Returns:
        List of reward values (each value between 0.0-1.0)
    """
    ground_truths = kwargs.get("ground_truth", [])
    rewards = []

    for completion, truth in zip(completions, ground_truths):
        reward = 0.0

        # Extract answer
        numbers = re.findall(r'-?\d+\.?\d*', completion)
        if numbers:
            try:
                pred = float(numbers[-1])
                truth_num = float(truth)
                error = abs(pred - truth_num)

                # Give different rewards based on error
                if error < 0.01:
                    reward = 1.0  # Completely correct
                elif error < 1.0:
                    reward = 0.8  # Very close
                elif error < 5.0:
                    reward = 0.5  # Close

                # Extra reward: encourage showing reasoning steps
                if "step" in completion.lower() or "=" in completion:
                    reward += 0.1

            except ValueError:
                reward = 0.0

        rewards.append(min(reward, 1.0))  # Limit maximum value to 1.0

    return rewards
```

맞춤 보상 기능을 사용하는 방법에는 두 가지가 있습니다.

**(1) 직접 패스**

```python
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/custom_grpo",
    "custom_dataset": rl_dataset,
    "custom_reward": custom_reward_function  # Directly pass reward function
})
```

**(2) 등록(권장)**

```python
# 1. Register reward function
rl_tool.register_reward_function("my_reward", custom_reward_function)

# 2. Use registered reward function
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "dataset": "my_math_dataset",
    "output_dir": "./models/custom_grpo"
    # Reward function will automatically use registered function with same name as dataset
})
```

다음은 사용자 정의 데이터 세트 및 보상 함수의 전체 예입니다.

```python
from datasets import Dataset
from hello_agents.tools import RLTrainingTool
from hello_agents.rl import format_math_dataset
import re
from typing import List

# 1. Prepare custom data
custom_data = [
    {"question": "What is 2+2?", "answer": "2+2=4. #### 4"},
    {"question": "What is 5+3?", "answer": "5+3=8. #### 8"},
    {"question": "What is 10+7?", "answer": "10+7=17. #### 17"}
]

# 2. Convert to training format
raw_dataset = Dataset.from_list(custom_data)
rl_dataset = format_math_dataset(raw_dataset, format_type="rl")

# 3. Define custom reward function
def tolerant_reward(completions: List[str], **kwargs) -> List[float]:
    """Reward function with tolerance"""
    ground_truths = kwargs.get("ground_truth", [])
    rewards = []

    for completion, truth in zip(completions, ground_truths):
        numbers = re.findall(r'-?\d+\.?\d*', completion)
        if numbers:
            try:
                pred = float(numbers[-1])
                truth_num = float(truth)
                error = abs(pred - truth_num)

                if error < 0.01:
                    reward = 1.0
                elif error < 5.0:
                    reward = 0.5
                else:
                    reward = 0.0
            except ValueError:
                reward = 0.0
        else:
            reward = 0.0

        rewards.append(reward)

    return rewards

# 4. Create tool and register
rl_tool = RLTrainingTool()
rl_tool.register_dataset("my_dataset", rl_dataset)
rl_tool.register_reward_function("my_dataset", tolerant_reward)

# 5. Train
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",
    "dataset": "my_dataset",
    "output_dir": "./models/custom_grpo",
    "num_epochs": 2,
    "batch_size": 2,
    "use_lora": True
})
```

## 11.3 SFT 훈련

SFT(Supervised Fine-Tuning)는 강화학습 훈련의 첫 번째 단계이자 가장 중요한 기초입니다. SFT를 통해 모델은 작업의 기본 형식, 대화 패턴 및 예비 추론 기능을 학습할 수 있습니다. SFT 기반이 없으면 강화 학습을 직접 수행하는 것은 모델이 기본 출력 형식조차 알지 못하기 때문에 실패하는 경우가 많습니다.

11.3.1 SFT가 필요한 이유

강화학습을 시작하기 전에 먼저 SFT 훈련을 진행해야 합니다. 이는 사전 학습된 모델이 강력한 언어 기능을 갖추고 있음에도 불구하고 특정 작업을 완료하는 방법을 모르기 때문입니다. 사전 훈련된 모델의 훈련 목표는 수학 문제를 해결하거나 도구를 사용하는 것이 아니라 다음 단어를 예측하는 것입니다. 사전 학습된 모델의 출력 형식은 자유 텍스트인 반면, 구조화된 출력(예: "1단계: ..., 2단계: ..., 최종 답변: ...")이 필요합니다. 사전 훈련된 모델은 작업 관련 데이터를 본 적이 없으며 "좋은" 추론 프로세스가 무엇인지 모릅니다.

SFT의 역할은 모델에 작업의 기본 규칙을 가르치는 것입니다. 먼저, 출력 형식을 학습하여 모델에 답변 구성 방법을 알려줍니다(예: "1단계", "최종 답변" 마커 사용). 둘째, 추론 패턴을 학습하고, 예시를 통해 문제를 분해하고 도출하는 방법을 단계별로 학습합니다. 셋째, 기본 기능을 설정하여 후속 강화 학습을 위한 합리적인 시작점을 제공합니다. 마지막으로 탐색 공간을 줄이면 강화 학습은 처음부터 시작할 필요가 없으며 SFT를 기반으로 최적화할 수 있습니다.

비교 실험을 통해 SFT의 중요성을 이해해 봅시다. GSM8K 문제를 해결하기 위해 사전 훈련된 모델을 직접 사용한다고 가정해 보겠습니다.

```python
from transformers import AutoTokenizer, AutoModelForCausalLM

# Load pretrained model
model_name = "Qwen/Qwen3-0.6B"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(model_name)

# Test question
question = """Natalia sold clips to 48 of her friends in April, and then she sold half as many clips in May. How many clips did Natalia sell altogether in April and May?"""

# Construct input
prompt = f"<|im_start|>user\n{question}<|im_end|>\n<|im_start|>assistant\n"
inputs = tokenizer(prompt, return_tensors="pt")

# Generate answer
outputs = model.generate(**inputs, max_new_tokens=200)
response = tokenizer.decode(outputs[0], skip_special_tokens=False)

print("Pretrained model's answer:")
print(response)
```

사전 학습된 모델의 출력은 다음과 같습니다.

```bash
<|im_start|>user
Natalia sold clips to 48 of her friends in April, and then she sold half as many clips in May. How many clips did Natalia sell altogether in April and May?<|im_end|>
<|im_start|>assistant
I can help you with that. Let me think about this problem. Natalia sold clips to her friends. In April she sold 48 clips. In May she sold half as many. So we need to find out how many she sold in May and then add them together. But I'm not sure how to calculate half of 48. Maybe I should use a calculator? Or maybe I can just estimate...
```

볼 수 있듯이, 사전 훈련된 모델이 질문을 이해하더라도 출력은 매우 장황하고 구조가 부족하며 명확한 대답이 없으며 추론 프로세스가 혼란스럽습니다. 이러한 출력은 답변을 추출할 수 없고 품질을 평가할 수 없으며 효과적인 보상 신호를 제공할 수 없기 때문에 강화 학습에 사용할 수 없습니다.

이제 SFT 이후의 모델을 살펴보겠습니다.

```python
# Load SFT model (assuming already trained)
sft_model_path = "./models/sft_model"
sft_model = AutoModelForCausalLM.from_pretrained(sft_model_path)

# Use the same question
outputs = sft_model.generate(**inputs, max_new_tokens=200)
response = tokenizer.decode(outputs[0], skip_special_tokens=False)

print("SFT model's answer:")
print(response)
```

SFT 모델의 출력:

```bash
<|im_start|>user
Natalia sold clips to 48 of her friends in April, and then she sold half as many clips in May. How many clips did Natalia sell altogether in April and May?<|im_end|>
<|im_start|>assistant
Let me solve this step by step.

Step 1: Calculate clips sold in May
Natalia sold half as many clips in May as in April.
Clips in May = 48 / 2 = 24

Step 2: Calculate total clips
Total = April + May = 48 + 24 = 72

Final Answer: 72<|im_end|>
```

볼 수 있듯이 SFT 모델의 출력은 명확한 구조("1단계", "2단계", "최종 답변" 마커 사용), 올바른 추론, 명확한 답변 및 통일된 형식을 가지고 있습니다. 이러한 출력은 답변을 추출하고, 보상을 계산하고, 전략을 최적화할 수 있으므로 강화 학습에 사용될 수 있습니다.

그림 11.6에서 볼 수 있듯이 SFT는 사전 훈련된 모델에서 강화 학습으로의 다리입니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-6.png" alt="" width="85%"/>
  <p>그림 11.6 훈련 파이프라인에서 SFT의 역할</p>
</div>

11.3.2 LoRA: 매개변수 효율적인 미세 조정

전체 모델을 직접 미세 조정하려면 상당한 계산 리소스와 메모리가 필요합니다. Qwen3-0.6B(0.6B 매개변수)의 경우 전체 미세 조정에는 약 12GB 메모리(FP16) 또는 24GB 메모리(FP32)가 필요합니다. 대형 모델(예: 7B, 13B)의 경우 소비자급 GPU에서는 전체 미세 조정이 거의 불가능합니다.

LoRA(Low-Rank Adaptation)<sup>[3]</sup>은 원래 모델 매개변수를 고정한 상태로 유지하면서 소수의 추가 매개변수만 훈련하는 매개변수 효율적인 미세 조정 방법입니다. LoRA의 핵심 아이디어는 모델 미세 조정 중 매개변수 변경을 하위 행렬로 표현할 수 있다는 것입니다.

원래 모델의 가중치 행렬이 $W \in \mathbb{R}^{d \times k}$이고 미세 조정된 가중치가 $W' = W + \Delta W$라고 가정합니다. LoRA는 $\Delta W$가 두 개의 하위 행렬의 곱으로 분해될 수 있다고 가정합니다.

$$
\델타 W = BA
$$

여기서 $B \in \mathbb{R}^{d \times r}$, $A \in \mathbb{R}^{r \times k}$, $r \ll \min(d, k)$는 순위입니다.

순방향 전파 동안 출력은 다음과 같습니다.

$$
h = Wx + \델타 Wx = Wx + BAx
$$

원래 모델 매개변수 $W$는 고정된 상태로 유지되며 $B$ 및 $A$만 훈련합니다.

매개변수 수 비교: 원래 모델 매개변수 수는 $d \times k$, LoRA 매개변수 수는 $d \times r + r \times k = r(d + k)$입니다. $r\ll\min(d, k)$일 때 LoRA 매개변수 개수는 원래 모델보다 훨씬 적습니다. 예를 들어 $d=4096, k=4096, r=8$의 경우 원래 모델 매개변수 개수는 $4096 \times 4096 = 16,777,216$이고 LoRA 매개변수 개수는 $8 \times (4096 + 4096) = 65,536$이므로 매개변수가 256배 감소합니다!

따라서 LoRA의 장점을 크게 메모리 사용량 감소, 학습 속도 향상, 배포 용이성, 과적합 방지 등으로 요약할 수 있습니다. 그러나 훈련 효과는 일반적으로 전체 매개변수 조정보다 다소 나쁩니다.

표 11.5에 표시된 것처럼 다양한 모델 규모에서 LoRA 효과를 비교합니다.

<div align="center">
  <p>표 11.5 LoRA와 전체 미세 조정 비교</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-5.png" alt="" width="85%"/>
</div>

LoRA의 주요 하이퍼파라미터는 다음과 같습니다: 순위(r), LoRA 행렬의 순위 제어, 클수록 표현력이 강해지지만 매개변수가 더 많다는 의미, 일반적인 값은 4-64, 기본값은 8입니다. 알파($\alpha$), LoRA 스케일링 계수, 실제 업데이트는 $\Delta W = \frac{\alpha}{r} BA$, LoRA의 영향력 강도를 제어하며 일반적인 값은 순위와 같습니다. LoRA를 적용할 레이어를 지정하고 일반적으로 attention 레이어(q_proj, k_proj, v_proj, o_proj)를 선택하는 target_modules에는 MLP 레이어(gate_proj, up_proj, down_proj)도 포함될 수 있습니다.

### 11.3.3 SFT 훈련 실습

이제 HelloAgent를 사용하여 SFT 교육을 수행해 보겠습니다. 전체 훈련 프로세스에는 데이터 세트 준비, LoRA 구성, 훈련 매개변수 설정, 훈련 시작 및 모델 저장이 포함됩니다.

기본 훈련 예시:

```python
from hello_agents.tools import RLTrainingTool

# Create training tool
rl_tool = RLTrainingTool()

# SFT training
result = rl_tool.run({
    # Training configuration
    "action": "train",
    "algorithm": "sft",

    # Model configuration
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/sft_model",

    # Data configuration
    "max_samples": 100,     # Use 100 samples for quick test

    # Training parameters
    "num_epochs": 3,        # Train for 3 epochs
    "batch_size": 4,        # Batch size
    "learning_rate": 5e-5,  # Learning rate

    # LoRA configuration
    "use_lora": True,       # Use LoRA
    "lora_rank": 8,         # LoRA rank
    "lora_alpha": 16,       # LoRA alpha
})

print(f"\n✓ Training completed!")
print(f"  - Model save path: {result['model_path']}")
print(f"  - Training samples: {result['num_samples']}")
print(f"  - Training epochs: {result['num_epochs']}")
print(f"  - Final loss: {result['final_loss']:.4f}")
```

학습 중에 손실이 점차 감소하면 모델이 학습 중임을 나타냅니다.

**(1) 훈련 매개변수 세부사항**

각 트레이닝 매개변수의 의미와 튜닝 제안을 자세히 이해해 보겠습니다.

**데이터 매개변수**:

- `max_samples`: 사용할 학습 샘플 수입니다. 빠른 테스트를 위해 100-1000개의 샘플을 사용하십시오. 완전한 훈련을 위해서는 모든 데이터(7473개 샘플)를 사용하는 것이 좋습니다. 일반적으로 데이터가 많을수록 더 나은 결과를 얻을 수 있지만 훈련 시간도 길어집니다.
- `split`: 데이터 세트 분할, 기본 "트레인". 처음 1000개의 샘플만 사용하려면 "train[:1000]"으로 설정할 수 있습니다.

**훈련 매개변수**:

- `num_epochs`: 학습 에포크 횟수입니다. 1 에포크는 전체 데이터 세트를 한 번 탐색한다는 의미입니다. 너무 적으면(1-2 epoch) 과소적합될 수 있고, 너무 많으면(>10 epoch) 과적합될 수 있습니다. 3개 에포크부터 시작하는 것이 좋습니다. 손실 곡선을 관찰하고 조정하세요.
- `batch_size`: 업데이트당 사용되는 샘플 수입니다. 클수록 더 안정적이지만 더 많은 메모리를 사용합니다. 메모리를 기준으로 조정하는 것이 좋습니다. 4GB 메모리 사용 Batch_size=1~2, 8GB 메모리 사용 Batch_size=4~8, 16GB 메모리 사용 Batch_size=8~16.
- `learning_rate`: 학습 속도, 매개변수 업데이트 단계 크기를 제어합니다. 너무 작으면(1e-6) 천천히 수렴하고, 너무 크면(1e-3) 수렴하지 않을 수 있습니다. SFT에서는 5e-5를 권장하고 LoRA는 약간 더 클 수 있습니다(1e-4).

**LoRA 매개변수**:

- `use_lora`: LoRA 사용 여부. 메모리가 충분하지 않은 한 항상 활성화하는 것이 좋습니다.
- `lora_rank`: LoRA 등급, 표현력을 제어합니다. 4~8은 소규모 작업에 적합하고, 16~32는 복잡한 작업에 적합하며, 64는 대규모 미세 조정에 적합합니다.
- `lora_alpha`: LoRA 스케일링 인자, 일반적으로 랭크의 2배로 설정됩니다. 순위=8, 알파=16일 때; 순위=16, 알파=32일 때.

**최적화 매개변수**:

- `optimizer`: 최적화 유형, 기본값은 "adamw"입니다. AdamW는 가장 일반적으로 사용되는 선택이며 "sgd" 또는 "adafactor"를 사용해 볼 수도 있습니다.
- `weight_decay`: 가중치 감소, 과적합을 방지합니다. 기본값은 0.01이며 0.001-0.1을 시도할 수 있습니다.
- `warmup_ratio`: 학습 속도 준비 비율입니다. 학습률은 첫 번째 Warmup_ratio 단계에서 선형적으로 증가한 다음 선형적으로 감소합니다. 기본값은 0.1(처음 10% 단계에 대한 준비)입니다.

**(2) 전체 훈련 예시**

모든 데이터와 모범 사례를 사용하여 완전한 SFT 교육을 수행해 보겠습니다.

```python
from hello_agents.tools import RLTrainingTool

rl_tool = RLTrainingTool()

# Complete SFT training
result = rl_tool.run({
    "action": "train",
    "algorithm": "sft",

    # Model configuration
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/sft_full",

    # Data configuration
    "max_samples": None,    # Use all data (7473 samples)

    # Training parameters
    "num_epochs": 3,
    "batch_size": 8,
    "learning_rate": 5e-5,
    "warmup_ratio": 0.1,
    "weight_decay": 0.01,

    # LoRA configuration
    "use_lora": True,
    "lora_rank": 16,        # Use larger rank
    "lora_alpha": 32,
    "lora_target_modules": ["q_proj", "k_proj", "v_proj", "o_proj"],

    # Other configurations
    "save_steps": 500,      # Save every 500 steps
    "logging_steps": 100,   # Log every 100 steps
    "eval_steps": 500,      # Evaluate every 500 steps
})

print(f"Training completed! Model saved at: {result['model_path']}")
```

이 구성은 8GB 메모리를 갖춘 GPU 교육에 적합하며 약 30~60분 정도 소요됩니다.

**(3) 훈련 모니터링 및 디버깅**

훈련 중에는 세 가지 주요 지표를 모니터링해야 합니다. 손실은 점차 감소해야 합니다. 감소하지 않으면 학습률이 너무 작거나 데이터에 문제가 있을 수 있습니다. 감소했다가 증가하면 학습률이 너무 크거나 과적합이 발생할 수 있습니다. Gradient Norm은 0.1-10의 합리적인 범위에 있어야 합니다. 너무 크면(>100) 기울기 폭발을 나타내며 학습 속도를 줄여야 합니다. 너무 작으면(<0.01) 그라데이션이 사라지는 것을 나타내며 모델 구성을 확인해야 합니다. 학습률은 준비 전략에 따라 변경되어야 하며 처음 10% 단계에서는 선형적으로 증가한 다음 0으로 선형적으로 감소합니다.

교육 및 해결 방법 중 일반적인 문제: 메모리가 부족할 경우 배치_크기 또는 최대 길이를 줄이고 기울기 누적 또는 더 작은 모델을 사용합니다. 훈련 속도가 느린 경우, 배치 크기를 늘리거나, 로깅 빈도를 줄이거나, 혼합 정밀도 훈련을 사용하십시오. 손실이 감소하지 않으면 학습률을 높이거나 데이터 형식을 확인하거나 훈련 에포크를 늘립니다. 과대적합 시에는 Weight_decay를 늘리고, 훈련 에포크를 줄이거나, 더 많은 데이터를 사용하세요.

11.3.4 모델 평가

훈련이 완료된 후에는 모델의 효율성을 평가해야 합니다. 평가 지표에는 다음이 포함됩니다.

- **정확도**: 완전히 정답의 비율, 가장 직접적인 측정항목, 범위 0-1, 높을수록 좋습니다.

- **평균 보상**: 정확도, 길이, 단계 및 기타 요소를 종합적으로 고려하여 모든 샘플의 평균 보상, 범위는 보상 기능 설계에 따라 다릅니다.

- **추론 품질**: 추론 프로세스의 명확성과 논리에는 수동 평가 또는 전문 평가 모델이 필요합니다.

HelloAgent를 사용하여 모델 평가:

```python
from hello_agents.tools import RLTrainingTool

rl_tool = RLTrainingTool()

# Evaluate SFT model
eval_result = rl_tool.run({
    "action": "evaluate",
    "model_path": "./models/sft_full",
    "max_samples": 100,     # Evaluate on 100 test samples
    "use_lora": True,
})

eval_data = json.loads(eval_result)
print(f"\nEvaluation results:")
print(f"  - Accuracy: {eval_data['accuracy']}")
print(f"  - Average reward: {eval_data['average_reward']}")
print(f"  - Test samples: {eval_data['num_samples']}")
```

Qwen3-0.6B와 같은 소형 모델의 경우 SFT 후 GSM8K에서 40-50% 정확도를 달성하는 것이 정상입니다. 강화학습을 통해 60~70%까지 더 향상시킬 수 있습니다.

SFT의 효율성을 더 잘 이해하기 위해 다양한 단계에서 모델을 비교할 수 있습니다.

```python
# Evaluate pretrained model (without SFT)
base_result = rl_tool.run({
    "action": "evaluate",
    "model_path": "Qwen/Qwen3-0.6B",
    "max_samples": 100,
    "use_lora": False,
})
base_data = json.loads(base_result)

# Evaluate SFT model
sft_result = rl_tool.run({
    "action": "evaluate",
    "model_path": "./models/sft_full",
    "max_samples": 100,
    "use_lora": True,
})
sft_data = json.loads(sft_result)

# Compare results
print("Model comparison:")
print(f"Pretrained model accuracy: {base_data['accuracy']}")
print(f"SFT model accuracy: {sft_data['accuracy']}")
```

이번 섹션에서는 SFT의 중요성(학습 형식, 기준선 설정), LoRA 원리(하위 분해, 매개변수 효율성), SFT 훈련 실습(매개변수 구성, 훈련 모니터링), 모델 평가(정확도, 비교 분석)에 대해 알아보았습니다.

## 11.4 GRPO 교육

SFT 훈련을 마친 후 구조화된 답변을 생성할 수 있는 모델을 얻었습니다. 그러나 SFT 모델은 훈련 데이터에서 추론 과정을 "모방"하는 방법만 학습했을 뿐 실제로 "생각하는" 방법은 배우지 않았습니다. 강화 학습을 통해 모델은 시행착오를 통해 추론 전략을 최적화할 수 있으므로 훈련 데이터의 품질을 뛰어넘을 수 있습니다.

11.4.1 PPO에서 GRPO로

강화학습 분야에서 PPO(Proximal Policy Optimization)<sup>[1]</sup>은 가장 고전적인 알고리즘 중 하나입니다. PPO는 정책 업데이트의 규모를 제한하여 교육 안정성을 보장합니다. 그러나 PPO는 LLM 교육에 몇 가지 문제가 있습니다. 가치 모델 교육이 필요하고 교육 복잡성과 메모리 사용량이 증가합니다. 4가지 모델(정책 모델, 참조 모델, 가치 모델, 보상 모델)을 동시에 유지해야 하므로 엔지니어링 구현이 복잡해집니다. 훈련은 불안정하고 보상이 붕괴되거나 정책이 저하되기 쉽습니다.

GRPO(그룹 상대 정책 최적화)<sup>[2]</sup>은 LLM을 위해 특별히 설계된 단순화된 PPO 변형입니다. GRPO의 핵심 아이디어는 다음과 같습니다. 절대 보상 대신 그룹 상대적 보상을 사용하여 가치 모델이 필요하지 않습니다. 정책 모델과 참조 모델만 필요한 단순화된 교육 프로세스 훈련 안정성이 향상되어 보상이 붕괴될 위험이 줄어듭니다.

GRPO의 원리를 수학 공식을 통해 이해해 봅시다. PPO의 목적 함수는 다음과 같습니다.

$$
J_{\text{PPO}}(\theta) = \mathbb{E}_{s,a \sim \pi_\theta} \left[ \min\left( \frac{\pi_\theta(a|s)}{\pi_{\text{old}}(a|s)} A(s,a), \text{클립}\left(\frac{\pi_\theta(a|s)}{\pi_{\text{old}}(a|s)}, 1-\epsilon, 1+\epsilon\right) A(s,a) \right) \right]
$$

여기서 $A(s,a)$는 가치 모델이 다음을 추정하도록 요구하는 이점 함수입니다.

$$
A(s,a) = Q(s,a) - V(s) = r(s,a) + \gamma V(s') - V(s)
$$

GRPO의 목적 함수는 다음과 같이 단순화됩니다.

$$
J_{\text{GRPO}}(\theta) = \mathbb{E}_{s,a \sim \pi_\theta} \left[ \frac{\pi_\theta(a|s)}{\pi_{\text{ref}}(a|s)} \cdot (r(s,a) - \bar{r}_{\text{group}}) \right] - \beta \cdot D_{KL}(\pi_\theta || \pi_{\text{ref}})
$$

여기서 $\bar{r}_{\text{group}}$는 그룹 평균 보상이고 $\beta$는 KL 발산 페널티 계수입니다. 주요 차이점은 다음과 같습니다. GRPO는 장점 함수 $A(s,a)$ 대신 $r(s,a) - \bar{r}_{\text{group}}$를 사용하므로 가치 모델이 필요하지 않습니다. GRPO는 그룹 상대적 보상을 사용하여 보상 차이를 줄입니다. GRPO는 KL 발산 페널티를 추가하여 정책이 너무 멀리 벗어나는 것을 방지합니다.

그림 11.7에서 볼 수 있듯이 PPO와 GRPO 교육 프로세스를 비교합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-7.png" alt="" width="85%"/>
  <p>그림 11.7 PPO와 GRPO 교육 프로세스</p>
</div>

보시다시피 GRPO는 가치 모델 교육을 제거하여 프로세스를 크게 단순화합니다.

표 11.6에는 PPO와 GRPO를 자세히 비교한 내용이 나와 있습니다.

<div align="center">
  <p>표 11.6 PPO와 GRPO 비교</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-6.png" alt="" width="85%"/>
</div>



LLM 교육의 경우 GRPO가 더 간단하고 안정적이며 메모리 사용량이 적기 때문에 더 나은 선택입니다.

11.4.2 GRPO 교육 실습

이제 HelloAgents를 사용하여 GRPO 교육을 수행해 보겠습니다. GRPO에는 합리적인 초기 정책이 필요하므로 GRPO 교육의 전제 조건은 SFT 교육을 완료하는 것입니다.

기본 GRPO 교육 예시:

```python
from hello_agents.tools import RLTrainingTool

# Create training tool
rl_tool = RLTrainingTool()

# GRPO training
result = rl_tool.run({
    # Training configuration
    "action": "train",
    "algorithm": "grpo",

    # Model configuration
    "model_name": "./models/sft_full",  # Start from SFT model
    "output_dir": "./models/grpo_model",

    # Data configuration
    "max_samples": 100,     # Use 100 samples for quick test

    # Training parameters
    "num_epochs": 3,
    "batch_size": 4,
    "learning_rate": 1e-5,  # GRPO learning rate usually smaller than SFT

    # GRPO-specific parameters
    "num_generations": 4,   # Generate 4 answers per question
    "kl_coef": 0.05,        # KL divergence penalty coefficient

    # LoRA configuration
    "use_lora": True,
    "lora_rank": 16,
    "lora_alpha": 32,

    # Reward function configuration
    "reward_type": "accuracy",  # Use accuracy reward
})

print(f"\n✓ Training completed!")
print(f"  - Model save path: {result['model_path']}")
print(f"  - Training samples: {result['num_samples']}")
print(f"  - Training epochs: {result['num_epochs']}")
print(f"  - Average reward: {result['average_reward']:.4f}")
```

GRPO 훈련 중 평균 보상이 점차 증가하고 KL 차이가 합리적인 범위에 유지되면 훈련이 정상적으로 진행되고 있음을 나타냅니다.

GRPO에는 이해하고 조정해야 하는 몇 가지 특정 매개변수가 있습니다.

**세대 매개변수**:

- `num_generations`: 질문당 생성할 답변 수입니다. 많을수록 좋지만 계산 비용도 높아집니다. 일반적인 값은 4-8입니다. 다중 답변을 생성하는 목적은 그룹 상대적 보상을 계산하고 훈련 신호의 다양성을 높이는 것입니다.
- `max_new_tokens`: 답변당 생성할 최대 토큰 수입니다. 너무 적으면 답변이 잘릴 수 있고 너무 많으면 계산이 낭비됩니다. 256-512를 추천합니다.
- `temperature`: 생성 온도, 무작위성을 제어합니다. 0은 그리디 디코딩을 의미하고, 1은 표준 샘플링을 의미합니다. GRPO는 일부 탐색을 유지하면서 0.7-1.0을 권장합니다.

**최적화 매개변수**:

- `learning_rate`: GRPO의 학습률은 SFT 모델에서 너무 멀리 벗어나는 것을 원하지 않기 때문에 일반적으로 SFT보다 작습니다. 1e-5에서 5e-5를 권장합니다.
- `kl_coef`: KL 발산 페널티 계수, 정책 업데이트의 크기를 제어합니다. 너무 작으면(0.01) 정책이 너무 멀리 벗어날 수 있고, 너무 크면(0.5) 학습이 제한될 수 있습니다. 0.05-0.1을 권장합니다.
- `clip_range`: PPO의 엡실론과 유사한 정책 비율 클리핑 범위입니다. 0.2를 추천합니다.

**보상 매개변수**:

- `reward_type`: 보상 함수 유형은 "accuracy", "length_penalty", "step" 또는 "combined"일 수 있습니다.
- `reward_config`: 길이 페널티에 대한 목표 길이, 단계 보상에 대한 계수 등 보상 함수에 대한 추가 구성입니다.

모든 데이터와 모범 사례를 사용하여 완전한 GRPO 교육을 수행해 보겠습니다.

```python
from hello_agents.tools import RLTrainingTool

rl_tool = RLTrainingTool()

# Complete GRPO training
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",

    # Model configuration
    "model_name": "./models/sft_full",
    "output_dir": "./models/grpo_full",

    # Data configuration
    "max_samples": None,    # Use all data

    # Training parameters
    "num_epochs": 3,
    "batch_size": 4,
    "learning_rate": 1e-5,
    "warmup_ratio": 0.1,

    # GRPO-specific parameters
    "num_generations": 4,
    "max_new_tokens": 512,
    "temperature": 0.8,
    "kl_coef": 0.05,
    "clip_range": 0.2,

    # LoRA configuration
    "use_lora": True,
    "lora_rank": 16,
    "lora_alpha": 32,

    # Reward function configuration
    "reward_type": "combined",
    "reward_config": {
        "components": [
            {"type": "accuracy", "weight": 1.0},
            {"type": "length_penalty", "weight": 0.5, "target_length": 200},
            {"type": "step", "weight": 0.3, "step_bonus": 0.1}
        ]
    },

    # Other configurations
    "save_steps": 500,
    "logging_steps": 100,
})

print(f"Training completed! Model saved at: {result['model_path']}")
```

11.4.3 GRPO 훈련 과정 분석

GRPO의 훈련 과정을 깊이 이해하고 각 단계에서 어떤 일이 일어나는지 살펴보겠습니다.

**(1) 훈련 루프**

GRPO의 훈련 루프에는 다음 단계가 포함됩니다.

1. **샘플링 단계**: 각 질문에 대해 현재 정책을 사용하여 여러 답변을 생성합니다(`num_generations`). 이러한 답변은 상대적인 보상을 계산하기 위한 "그룹"을 형성합니다.

2. **보상 계산**: 생성된 각 답변에 대한 보상 $r_i$를 계산합니다. 보상은 정확도, 길이 페널티, 단계 보상 또는 이들의 조합일 수 있습니다.

3. **상대적 보상**: 그룹 평균 보상 $\bar{r} = \frac{1}{N}\sum_{i=1}^{N} r_i$을 계산한 다음 상대 보상 $\hat{r}_i = r_i - \bar{r}$을 계산합니다. 이것의 이점은 보상 변동을 줄이고 훈련을 더욱 안정적으로 만드는 것입니다.

4. **정책 업데이트**: 상대 보상을 사용하여 정책을 업데이트하는 동시에 정책이 참조 모델에서 너무 멀리 벗어나는 것을 방지하기 위해 KL 발산 페널티를 추가합니다.

5. **반복**: 모든 훈련 에포크가 완료될 때까지 위 단계를 반복합니다.

구체적인 예를 통해 이해해 보겠습니다.

```python
# Assume we have a question
question = "What is 48 + 24?"

# Generate 4 answers
answers = [
    "48 + 24 = 72. Final Answer: 72",      # Correct
    "48 + 24 = 72. Final Answer: 72",      # Correct
    "48 + 24 = 70. Final Answer: 70",      # Incorrect
    "Let me think... 72. Final Answer: 72" # Correct but verbose
]

# Calculate rewards (assuming using accuracy + length penalty)
rewards = [1.0, 1.0, 0.0, 0.8]  # 4th answer penalized for verbosity

# Calculate group average reward
avg_reward = (1.0 + 1.0 + 0.0 + 0.8) / 4 = 0.7

# Calculate relative rewards
relative_rewards = [
    1.0 - 0.7 = 0.3,   # Correct and concise, positive relative reward
    1.0 - 0.7 = 0.3,   # Correct and concise, positive relative reward
    0.0 - 0.7 = -0.7,  # Incorrect, negative relative reward
    0.8 - 0.7 = 0.1    # Correct but verbose, smaller relative reward
]

# Policy update: increase probability of first two answers, decrease probability of third answer
```

볼 수 있듯이 상대적 보상 메커니즘은 모델이 단순히 높은 보상을 추구하기보다는 "평균보다 나은" 답변을 생성하도록 장려합니다. 이는 보상 변동을 줄이고 훈련 안정성을 향상시킬 수 있습니다.

**(2) KL 다이버전스 페널티**

KL 발산 페널티는 GRPO의 핵심 구성 요소로, 정책이 참조 모델에서 너무 벗어나는 것을 방지합니다. KL 발산은 다음과 같이 정의됩니다.

$$
D_{KL}(\pi_\theta || \pi_{\text{ref}}) = \mathbb{E}_{s,a \sim \pi_\theta} \left[ \log \frac{\pi_\theta(a|s)}{\pi_{\text{ref}}(a|s)} \right]
$$

실제로는 각 토큰에 대한 KL 차이를 계산한 후 다음과 같이 합산합니다.

$$
D_{KL} = \sum_{t=1}^{T} \log \frac{\pi_\theta(a_t|s, a_{<t})}{\pi_{\text{ref}}(a_t|s, a_{<t})}
$$

KL 발산이 클수록 현재 정책과 참조 모델 간의 차이가 커집니다. KL 발산 페널티 항 $-\beta\cdot D_{KL}$을 추가함으로써 정책 업데이트의 규모를 제한하여 SFT 단계에서 학습한 지식을 "잊는" 것을 방지합니다.

`kl_coef`($\beta$)의 선택이 중요합니다.

- 너무 작음(0.01): 정책이 너무 많이 벗어나 출력 형식이 혼동되거나 품질이 저하될 수 있습니다.
- 너무 큼(0.5): 정책 업데이트가 제한적이고 학습이 느리며 SFT 모델을 능가하기 어렵습니다.
- 권장(0.05-0.1): 탐색과 안정성의 균형

**(3) 훈련 모니터링**

GRPO 교육 중에 다음 측정항목을 모니터링해야 합니다.

- **평균 보상**: 점차 증가해야 합니다. 보상이 증가하지 않으면 학습률이 너무 작거나, KL 페널티가 너무 크거나, 보상 함수 설계가 불합리할 수 있습니다. 보상이 증가한 후 감소하면 과적합이거나 보상이 붕괴될 수 있습니다.

- **KL 발산**: 합리적인 범위(0.01-0.1)를 유지해야 합니다. KL 발산이 너무 크면(>0.5) 정책이 너무 멀리 벗어나므로 kl_coef를 높이거나 학습 속도를 줄여야 합니다. KL 발산이 너무 작으면(<0.001) 정책이 거의 업데이트되지 않으므로 kl_coef를 줄이거나 학습 속도를 높여야 합니다.

- **정확도**: 점진적으로 개선되어야 합니다. 이는 모델의 실제 성능을 반영하는 가장 직관적인 측정항목입니다.

- **생성 품질**: 올바른 형식과 명확한 추론을 보장하기 위해 생성된 답변을 수동으로 검사해야 합니다.

HelloAgents는 두 가지 주요 교육 모니터링 도구인 Weights & Biases(wandb)와 TensorBoard를 통합합니다.

**방법 1: 가중치 및 편향 사용(권장)**

Weights & Biases는 현재 가장 널리 사용되는 기계 학습 실험 추적 플랫폼으로, 강력한 시각화 및 실험 관리 기능을 제공합니다.

```python
import os

# 1. Set up wandb (need to register account first: https://wandb.ai)
os.environ["WANDB_PROJECT"] = "hello-agents-grpo"  # Project name
os.environ["WANDB_LOG_MODEL"] = "false"            # Don't upload model files

# 2. Enable wandb in training configuration
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/grpo_monitored",
    "num_epochs": 2,
    "batch_size": 2,
    "use_lora": True,
    # wandb will automatically log all training metrics
})

# After training completes, visit https://wandb.ai to view training curves
```

wandb는 다음 측정항목을 자동으로 기록합니다.
- `train/reward`: 평균 보상
- `train/kl`: KL 발산
- `train/loss`: 훈련 손실
- `train/learning_rate`: 학습률
- `train/epoch`: 훈련 에포크

**방법 2: 텐서보드 사용**

TensorBoard는 TensorFlow에서 제공하는 시각화 도구이며 PyTorch 교육도 지원합니다.

```python
# 1. TensorBoard logs will be automatically created in output_dir during training
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/grpo_tb",
    "num_epochs": 2,
    "batch_size": 2,
    "use_lora": True,
})

# 2. Launch TensorBoard to view training curves
# Run in command line:
# tensorboard --logdir=./models/grpo_tb
# Then visit http://localhost:6006
```

**방법 3: 오프라인 모니터링(외부 도구 필요 없음)**

wandb 또는 TensorBoard를 사용하지 않으려면 훈련 로그를 통해 모니터링할 수도 있습니다.

```python
# Training process will print detailed logs
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/grpo_simple",
    "num_epochs": 2,
    "batch_size": 2,
    "use_lora": True,
})

# Log example:
# Epoch 1/2 | Step 100/500 | Reward: 0.45 | KL: 0.023 | Loss: 1.234
# Epoch 1/2 | Step 200/500 | Reward: 0.52 | KL: 0.031 | Loss: 1.156
# ...
```

GRPO 교육 중에 몇 가지 문제가 발생할 수 있습니다. 보상이 증가하지 않는 경우 학습률이 너무 작거나 KL 페널티가 너무 커서 정책 업데이트를 제한하거나 보상 기능 설계가 불합리하거나 SFT 모델 품질이 너무 낮은 것일 수 있습니다. 이 경우 학습률을 높이거나(1e-5에서 5e-5로), kl_coef를 낮추거나(0.1에서 0.05로), 보상 기능을 확인하거나 SFT 모델을 다시 학습시키세요.

KL 발산이 폭발적으로 증가하여(0.5 또는 1.0 초과) 생성된 답변 형식 혼란을 일으키는 경우 이는 일반적으로 학습률이 너무 크거나 KL 페널티가 너무 작거나 보상 기능이 너무 공격적이기 때문입니다. 학습률을 낮추거나(5e-5에서 1e-5로) kl_coef를 높이거나(0.05에서 0.1로) 보상 기능을 조정하거나 그래디언트 클리핑을 사용할 수 있습니다.

생성 품질이 저하되면(정확도는 향상되지만 형식이 혼란스럽고 추론이 불분명함) 보상 기능이 다른 품질 측정항목을 무시하고 정확도에만 초점을 맞추거나 KL 페널티가 너무 작아서 모델이 SFT에서 너무 멀리 벗어나거나 과적합이 발생할 수 있습니다. 이 경우 결합된 보상 함수를 사용하여 여러 측정항목을 동시에 최적화하고, kl_coef를 늘려 일관성을 유지하고, 학습 에포크를 줄이거나, 학습 데이터를 늘립니다.

GRPO 교육은 여러 답변을 동시에 생성하고 참조 모델 출력을 저장해야 하며 OOM이 발생하기 쉬우므로 SFT보다 메모리 사용량이 높습니다. num_세대(8에서 4로), 배치_크기(4에서 2로) 또는 max_new_tokens(512에서 256으로)를 줄이거나 그라데이션 체크포인트 및 혼합 정밀도 교육을 사용하여 완화할 수 있습니다.

## 11.5 모델 평가 및 분석

학습이 완료된 후에는 정확도를 단일 지표로 보는 것뿐만 아니라 모델의 추론 품질, 오류 패턴, 일반화 능력 등을 심층적으로 분석하여 모델 성능을 종합적으로 평가해야 합니다. 이 섹션에서는 Agentic RL 모델을 체계적으로 평가하고 분석하는 방법을 소개합니다.

11.5.1 평가 지표 시스템

좋은 평가 시스템은 다양한 각도에서 모델 기능을 측정하는 다차원적이어야 합니다. 우리는 평가 지표를 정확도 지표, 효율성 지표, 품질 지표의 세 가지 범주로 나눕니다.

**(1) 정확도 측정항목**

정확도 측정항목은 모델이 정답에 도달할 수 있는지 여부를 측정합니다.

**정확도**: 가장 기본적인 측정항목으로 완전히 정답의 비율입니다. 계산 공식:
$$
\text{정확도} = \frac{\text{정답 수}}{\text{총 질문 수}}
$$

장점은 간단하고 직관적이며 이해하고 비교하기 쉽습니다. 단점은 "거의 옳은 것"과 "완전히 틀린 것"을 구별할 수 없다는 점이며, 복잡한 작업에는 너무 조잡할 수 있습니다.

**상위 K 정확도**: K개의 답변을 생성하고, 하나라도 맞으면 정답으로 계산합니다. 계산 공식:
$$
\text{정확도@K} = \frac{\text{정답이 하나 이상인 질문 수}}{\text{총 질문 수}}
$$

이 측정항목은 모델의 '잠재력', 즉 다중 샘플링을 통해 정답을 찾을 수 있는지 여부를 반영합니다.

**수치 오류**: 수학적 문제의 경우 예측 값과 실제 값 사이의 오류를 계산할 수 있습니다. 계산 공식:

$$
\text{오류} = \frac{1}{N} \sum_{i=1}^{N} |y_i - \hat{y}_i|
$$

이 측정항목은 "거의 정확한"(예: 예측 72.5, 실제 72)과 "완전히 틀린"(예: 예측 100, 실제 72)을 구별할 수 있습니다.

**(2) 효율성 지표**

효율성 지표는 답변 생성 비용을 측정합니다.

**평균 길이**: 생성된 답변의 평균 토큰 수입니다. 계산 공식:

$$
\text{평균 길이} = \frac{1}{N} \sum_{i=1}^{N} |y_i|
$$

답변이 짧을수록 추론 비용이 낮아지고 응답 속도가 빨라집니다.

**추론 단계**: 답변에 포함된 추론 단계의 수입니다. 계산 공식:

$$
\text{평균 단계} = \frac{1}{N} \sum_{i=1}^{N} s_i
$$

적절한 수의 단계(2~5단계)는 모델이 문제를 체계적으로 분해할 수 있음을 나타냅니다. 단계가 너무 많으면 추론이 중복되었음을 나타낼 수 있습니다.

**추론 시간**: 하나의 답변을 생성하는 데 필요한 시간입니다. 이 지표는 실제 배포에서 중요하며 사용자 경험에 영향을 미칩니다.

**(3) 품질 지표**

품질 지표는 답변의 가독성과 설명 가능성을 측정합니다.

**형식 정확성**: 답변이 예상 형식(예: "1단계", "최종 답변"과 같은 표시 포함)을 준수하는지 여부입니다. 계산 공식:
$$
\text{형식 정확성} = \frac{\text{올바른 형식의 답변 수}}{\text{총 답변 수}}
$$

올바른 형식은 기본 요구 사항입니다. 혼란스러운 형식의 답변은 결과가 정확하더라도 사용하기 어렵습니다.

**추론 일관성**: 추론 단계가 논리적으로 일관성이 있는지 여부입니다. 이 측정항목에는 일반적으로 수동 평가 또는 특수 평가 모델이 필요합니다.

**설명성**: 답변을 이해하고 확인하기 쉬운지 여부입니다. 명확한 단계가 포함된 답변은 결과를 직접적으로 제공하는 답변보다 더 설명하기 쉽습니다.

표 11.7에 표시된 것처럼 다양한 측정항목을 비교합니다.

<div align="center">
  <p>표 11.7 평가 지표 비교</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-7.png" alt="" width="85%"/>
</div>


11.5.2 평가 실습

HelloAgents는 여러 지표를 한 번에 계산할 수 있는 포괄적인 평가 기능을 제공합니다.

```python
from hello_agents.tools import RLTrainingTool

rl_tool = RLTrainingTool()

# Comprehensive evaluation
print("=" * 50)
print("Comprehensive GRPO Model Evaluation")
print("=" * 50)

result = rl_tool.run({
    "action": "evaluate",
    "model_path": "./models/grpo_full",
    "max_samples": 200,
    "use_lora": True,

    # Evaluation configuration
    "metrics": [
        "accuracy",           # Accuracy
        "accuracy_at_k",      # Top-K accuracy
        "average_length",     # Average length
        "average_steps",      # Average steps
        "format_correctness", # Format correctness
    ],
    "k": 3,  # Top-3 accuracy
})

# Parse results
eval_data = json.loads(result)

# Print results
print(f"\nEvaluation results:")
print(f"  Accuracy: {eval_data['accuracy']}")
print(f"  Average reward: {eval_data['average_reward']}")
print(f"  Test samples: {eval_data['num_samples']}")
```

사전 훈련된 모델, SFT 모델 및 GRPO 모델의 성능을 비교할 수 있습니다.

```python
# Evaluate three models
models = [
    ("Pretrained Model", "Qwen/Qwen3-0.6B", False),
    ("SFT Model", "./models/sft_full", True),
    ("GRPO Model", "./models/grpo_full", True),
]

results = []
for name, path, use_lora in models:
    print(f"\nEvaluating {name}...")
    result = rl_tool.run({
        "action": "evaluate",
        "model_path": path,
        "max_samples": 200,
        "use_lora": use_lora,
        "metrics": ["accuracy", "average_length", "format_correctness"],
    })
    results.append((name, result))

# Print comparison table
print("\n" + "=" * 70)
print(f"{'Model':<15} {'Accuracy':<12} {'Avg Length':<15} {'Format Correct':<12}")
print("=" * 70)
for name, result in results:
    print(f"{name:<15} {result['accuracy']:<12.2%} {result['average_length']:<15.1f} {result['format_correctness']:<12.2%}")
print("=" * 70)
```

11.5.3 오류 분석

정확성을 아는 것만으로는 충분하지 않습니다. 모델에서 오류가 발생하기 쉬운 문제 유형을 심층 분석하여 후속 개선을 유도해야 합니다. 모델 오류는 다음 네 가지 범주로 나눌 수 있습니다. 계산 오류(추론 단계는 정확하지만 계산이 잘못됨(예: "48/2=25", 수치 계산 능력 부족을 나타냄)), 추론 오류(잘못된 문제 해결 접근 방식으로 이어지는 추론 논리 오류, 예: 먼저 나누는 대신 먼저 더한 다음 나누기, 논리적 추론 능력이 부족함을 나타냄), 이해 오류(문제를 올바르게 이해하지 못함(예: 질문에서 "전체""를 요구하지만 계산된 부분만 표시하여 언어가 부족함을 나타냄) 이해 능력), 형식 오류(정답이지만 형식이 요구 사항을 충족하지 않음, 예: "최종 답변:" 표시 누락, 형식 학습이 부족함을 나타냄).

오류 분석 예:

```python
from hello_agents.tools import RLTrainingTool

rl_tool = RLTrainingTool()

# Evaluate and collect error samples
result = rl_tool.run({
    "action": "evaluate",
    "model_path": "./models/grpo_full",
    "max_samples": 200,
    "use_lora": True,
    "return_details": True,  # Return detailed results
})

# Analyze error samples
errors = result['errors']  # Error sample list
print(f"Total errors: {len(errors)}")

# Classify by error type
error_types = {
    "Calculation Error": 0,
    "Reasoning Error": 0,
    "Comprehension Error": 0,
    "Format Error": 0,
}

for error in errors:
    question = error['question']
    prediction = error['prediction']
    ground_truth = error['ground_truth']

    # Simple error classification logic (may need more complex analysis in practice)
    if "Final Answer:" not in prediction:
        error_types["Format Error"] += 1
    elif "Step" in prediction:
        # Has reasoning steps, may be calculation or reasoning error
        # More detailed analysis needed here
        error_types["Calculation Error"] += 1
    else:
        error_types["Comprehension Error"] += 1

# Print error distribution
print("\nError type distribution:")
for error_type, count in error_types.items():
    percentage = count / len(errors) * 100
    print(f"  {error_type}: {count} ({percentage:.1f}%)")
```

출력 예:

```bash
Total errors: 76

Error type distribution:
  Calculation Error: 32 (42.1%)
  Reasoning Error: 18 (23.7%)
  Comprehension Error: 22 (28.9%)
  Format Error: 4 (5.3%)
```

보는 바와 같이 계산오류가 주요 오류 유형(42.1%)으로 나타나 모델의 수치계산 능력 강화가 필요함을 알 수 있다. 형식 오류는 거의 발생하지 않으며(5.3%), 이는 SFT 훈련이 효과적임을 나타냅니다. 또한 다양한 난이도의 문제에 대한 모델의 성능을 분석할 수도 있습니다.

```python
# Group by number of reasoning steps
step_groups = {
    "Easy (1-2 steps)": [],
    "Medium (3-4 steps)": [],
    "Hard (5+ steps)": [],
}

for sample in result['details']:
    steps = sample['ground_truth_steps']  # Number of steps in true answer
    correct = sample['correct']

    if steps <= 2:
        step_groups["Easy (1-2 steps)"].append(correct)
    elif steps <= 4:
        step_groups["Medium (3-4 steps)"].append(correct)
    else:
        step_groups["Hard (5+ steps)"].append(correct)

# Calculate accuracy for each group
print("\nAccuracy at different difficulty levels:")
for group_name, results in step_groups.items():
    if len(results) > 0:
        accuracy = sum(results) / len(results)
        print(f"  {group_name}: {accuracy:.2%} ({len(results)} samples)")
```

출력 예:

```bash
Accuracy at different difficulty levels:
  Easy (1-2 steps): 78.50% (85 samples)
  Medium (3-4 steps): 58.30% (96 samples)
  Hard (5+ steps): 31.60% (19 samples)
```

보시다시피, 모델은 쉬운 문제(78.5%)에서는 잘 수행되지만 어려운 문제(31.6%)에서는 제대로 수행되지 않습니다. 이는 모델의 다단계 추론 능력이 향상되어야 함을 나타냅니다.

11.5.4 개선방향

평가 및 분석 결과를 바탕으로 그림 11.8과 같이 모델의 개선 방향을 결정할 수 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-8.png" alt="" width="85%"/>
  <p>그림 11.8 모델 개선 반복 프로세스</p>
</div>

이는 모델 학습 → 성능 평가 → 오류 분석 → 문제 식별 → 개선 방향 선택 → 재학습의 지속적인 반복 프로세스입니다. 여러 번의 반복을 통해 모델 성능이 지속적으로 향상됩니다.

## 11.6 훈련 파이프라인 실습 완료

이전 섹션에서는 데이터 준비, SFT 훈련, GRPO 훈련 및 모델 평가에 대해 별도로 배웠습니다. 이제 이 지식을 통합하여 엔드투엔드 Agentic RL 훈련 파이프라인을 완성해 보겠습니다.

11.6.1 엔드투엔드 훈련 파이프라인

전체 Agentic RL 교육 파이프라인에는 데이터 준비, SFT 교육, SFT 평가, GRPO 교육, GRPO 평가 및 모델 배포 단계가 포함됩니다. 그림 11.9에 나와 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-9.png" alt="" width="85%"/>
  <p>그림 11.9 엔드투엔드 훈련 파이프라인</p>
</div>

완전한 스크립트를 통해 이 파이프라인을 구현해 보겠습니다.

```python
"""
Complete Agentic RL Training Pipeline
End-to-end example from data preparation to model deployment
"""

from hello_agents.tools import RLTrainingTool
import json
from datetime import datetime

class AgenticRLPipeline:
    """Agentic RL Training Pipeline"""

    def __init__(self, config_path="config.json"):
        """
        Initialize training pipeline

        Args:
            config_path: Configuration file path
        """
        self.rl_tool = RLTrainingTool()
        self.config = self.load_config(config_path)
        self.results = {}

    def load_config(self, config_path):
        """Load configuration file"""
        with open(config_path, 'r') as f:
            return json.load(f)

    def log(self, message):
        """Log message"""
        timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        print(f"[{timestamp}] {message}")

    def stage1_prepare_data(self):
        """Stage 1: Data Preparation"""
        self.log("=" * 50)
        self.log("Stage 1: Data Preparation")
        self.log("=" * 50)

        # Load and check dataset
        result = self.rl_tool.run({
            "action": "load_dataset",
            "format": "sft",
            "max_samples": self.config["data"]["max_samples"],
        })

        # Parse JSON result
        dataset_info = json.loads(result)

        self.log(f"✓ Dataset loaded")
        self.log(f"  - Samples: {dataset_info['dataset_size']}")
        self.log(f"  - Format: {dataset_info['format']}")
        self.log(f"  - Data columns: {', '.join(dataset_info['sample_keys'])}")

        self.results["data"] = dataset_info

        return dataset_info

    def stage2_sft_training(self):
        """Stage 2: SFT Training"""
        self.log("\n" + "=" * 50)
        self.log("Stage 2: SFT Training")
        self.log("=" * 50)

        sft_config = self.config["sft"]

        result = self.rl_tool.run({
            "action": "train",
            "algorithm": "sft",
            "model_name": self.config["model"]["base_model"],
            "output_dir": sft_config["output_dir"],
            "max_samples": self.config["data"]["max_samples"],
            "num_epochs": sft_config["num_epochs"],
            "batch_size": sft_config["batch_size"],
            "use_lora": True,
            # Training monitoring configuration
            "use_wandb": self.config.get("monitoring", {}).get("use_wandb", False),
            "use_tensorboard": self.config.get("monitoring", {}).get("use_tensorboard", True),
            "wandb_project": self.config.get("monitoring", {}).get("wandb_project", None),
        })

        # Parse JSON result
        result_data = json.loads(result)

        self.log(f"✓ SFT training completed")
        self.log(f"  - Model path: {result_data['output_dir']}")
        self.log(f"  - Status: {result_data['status']}")

        self.results["sft_training"] = result_data

        return result_data["output_dir"]

    def stage3_sft_evaluation(self, model_path):
        """Stage 3: SFT Evaluation"""
        self.log("\n" + "=" * 50)
        self.log("Stage 3: SFT Evaluation")
        self.log("=" * 50)

        result = self.rl_tool.run({
            "action": "evaluate",
            "model_path": model_path,
            "max_samples": self.config["eval"]["max_samples"],
            "use_lora": True,
        })
        eval_data = json.loads(result)

        self.log(f"✓ SFT evaluation completed")
        self.log(f"  - Accuracy: {eval_data['accuracy']}")
        self.log(f"  - Average reward: {eval_data['average_reward']}")

        self.results["sft_evaluation"] = eval_data

        return eval_data

    def stage4_grpo_training(self, sft_model_path):
        """Stage 4: GRPO Training"""
        self.log("\n" + "=" * 50)
        self.log("Stage 4: GRPO Training")
        self.log("=" * 50)

        grpo_config = self.config["grpo"]

        result = self.rl_tool.run({
            "action": "train",
            "algorithm": "grpo",
            "model_name": sft_model_path,
            "output_dir": grpo_config["output_dir"],
            "max_samples": self.config["data"]["max_samples"],
            "num_epochs": grpo_config["num_epochs"],
            "batch_size": grpo_config["batch_size"],
            "use_lora": True,
            # Training monitoring configuration
            "use_wandb": self.config.get("monitoring", {}).get("use_wandb", False),
            "use_tensorboard": self.config.get("monitoring", {}).get("use_tensorboard", True),
            "wandb_project": self.config.get("monitoring", {}).get("wandb_project", None),
        })

        # Parse JSON result
        result_data = json.loads(result)

        self.log(f"✓ GRPO training completed")
        self.log(f"  - Model path: {result_data['output_dir']}")
        self.log(f"  - Status: {result_data['status']}")

        self.results["grpo_training"] = result_data

        return result_data["output_dir"]

    def stage5_grpo_evaluation(self, model_path):
        """Stage 5: GRPO Evaluation"""
        self.log("\n" + "=" * 50)
        self.log("Stage 5: GRPO Evaluation")
        self.log("=" * 50)

        result = self.rl_tool.run({
            "action": "evaluate",
            "model_path": model_path,
            "max_samples": self.config["eval"]["max_samples"],
            "use_lora": True,
        })
        eval_data = json.loads(result)

        self.log(f"✓ GRPO evaluation completed")
        self.log(f"  - Accuracy: {eval_data['accuracy']}")
        self.log(f"  - Average reward: {eval_data['average_reward']}")

        self.results["grpo_evaluation"] = eval_data

        return eval_data

    def stage6_save_results(self):
        """Stage 6: Save Results"""
        self.log("\n" + "=" * 50)
        self.log("Stage 6: Save Results")
        self.log("=" * 50)

        # Save training results
        results_path = "training_results.json"
        with open(results_path, 'w') as f:
            json.dump(self.results, f, indent=2)

        self.log(f"✓ Results saved to: {results_path}")

    def run(self):
        """Run complete pipeline"""
        try:
            # Stage 1: Data preparation
            self.stage1_prepare_data()

            # Stage 2: SFT training
            sft_model_path = self.stage2_sft_training()

            # Stage 3: SFT evaluation
            self.stage3_sft_evaluation(sft_model_path)

            # Stage 4: GRPO training
            grpo_model_path = self.stage4_grpo_training(sft_model_path)

            # Stage 5: GRPO evaluation
            self.stage5_grpo_evaluation(grpo_model_path)

            # Stage 6: Save results
            self.stage6_save_results()

            self.log("\n" + "=" * 50)
            self.log("✓ Training pipeline completed!")
            self.log("=" * 50)

        except Exception as e:
            self.log(f"\n✗ Training failed: {str(e)}")
            raise

# Usage example
if __name__ == "__main__":
    # Create configuration file
    config = {
        "model": {
            "base_model": "Qwen/Qwen3-0.6B"
        },
        "data": {
            "max_samples": 1000  # Use 1000 samples
        },
        "sft": {
            "output_dir": "./models/sft_model",
            "num_epochs": 3,
            "batch_size": 8,
        },
        "grpo": {
            "output_dir": "./models/grpo_model",
            "num_epochs": 3,
            "batch_size": 4,
        },
        "eval": {
            "max_samples": 200,
            "sft_accuracy_threshold": 0.40  # SFT accuracy threshold
        },
        "monitoring": {
            "use_wandb": False,  # Whether to use Wandb
            "use_tensorboard": True,  # Whether to use TensorBoard
            "wandb_project": "agentic-rl-pipeline"  # Wandb project name
        }
    }

    # Save configuration
    with open("config.json", 'w') as f:
        json.dump(config, f, indent=2)

    # Run training pipeline
    pipeline = AgenticRLPipeline("config.json")
    pipeline.run()
```

이 스크립트를 실행하면 전체 학습 프로세스가 표시됩니다.

달리기 팁:

**작게 시작**: 모든 데이터를 한꺼번에 사용하여 학습을 시작하지 마세요. 먼저 빠른 반복을 위해 100~1000개의 샘플을 사용하고, 프로세스와 매개변수를 검증하고, 효과를 확인한 후 규모를 확대합니다. 이를 통해 상당한 시간과 계산 리소스를 절약할 수 있습니다.

**데이터 품질 확인**: 훈련 전에 데이터 품질을 확인하고 올바른 형식, 정확한 답, 중복 샘플이 없는지 확인하세요. 다음 코드를 사용할 수 있습니다.

```python
def check_data_quality(dataset):
    """Check data quality"""
    issues = []

    # Check required fields
    required_fields = ["prompt", "completion"]
    for field in required_fields:
        if field not in dataset.column_names:
            issues.append(f"Missing field: {field}")

    # Check null values
    for i, sample in enumerate(dataset):
        if not sample["prompt"] or not sample["completion"]:
            issues.append(f"Sample {i} contains null values")

    # Check duplicates
    prompts = [s["prompt"] for s in dataset]
    duplicates = len(prompts) - len(set(prompts))
    if duplicates > 0:
        issues.append(f"Found {duplicates} duplicate samples")

    return issues

# Usage
issues = check_data_quality(dataset)
if issues:
    print("Data quality issues:")
    for issue in issues:
        print(f"  - {issue}")
else:
    print("✓ Data quality check passed")
```

**데이터 확대**: 데이터 양이 부족한 경우 질문 다시 작성(답변을 변경하지 않고 유지), 유사한 질문 생성 또는 역번역과 같은 데이터 확대를 고려합니다. 하지만 데이터 품질을 유지하고 노이즈가 발생하지 않도록 주의하세요.

### 11.6.2 하이퍼파라미터 튜닝

초매개변수 조정은 모델 성능을 향상시키는 데 핵심입니다. 다음은 일반적으로 사용되는 몇 가지 튜닝 전략입니다.

**(1) 그리드 검색**

그리드 검색은 모든 매개변수 조합을 탐색하고 최상의 세트를 선택하는 가장 간단한 튜닝 방법입니다.

```python
# Define parameter grid
param_grid = {
    "learning_rate": [1e-5, 5e-5, 1e-4],
    "lora_rank": [8, 16, 32],
    "kl_coef": [0.05, 0.1, 0.2],
}

best_accuracy = 0
best_params = None

# Traverse all combinations
for lr in param_grid["learning_rate"]:
    for rank in param_grid["lora_rank"]:
        for kl in param_grid["kl_coef"]:
            print(f"Testing parameters: lr={lr}, rank={rank}, kl={kl}")

            # Train model
            result = rl_tool.run({
                "action": "train",
                "algorithm": "grpo",
                "learning_rate": lr,
                "lora_rank": rank,
                "kl_coef": kl,
                # Other parameters...
            })

            # Evaluate model
            eval_result = rl_tool.run({
                "action": "evaluate",
                "model_path": result["model_path"],
            })

            # Update best parameters
            if eval_result["accuracy"] > best_accuracy:
                best_accuracy = eval_result["accuracy"]
                best_params = {"lr": lr, "rank": rank, "kl": kl}

print(f"Best parameters: {best_params}")
print(f"Best accuracy: {best_accuracy:.2%}")
```

그리드 검색의 장점은 간단하고 직접적이며 전역 최적을 찾을 수 있습니다. 단점은 계산 비용이 높고 매개변수가 많으면 실용적이지 않다는 것입니다.

**(2) 무작위 검색**

무작위 검색은 매개변수 조합을 무작위로 샘플링하므로 그리드 검색보다 더 효율적입니다.

```python
import random

# Define parameter ranges
param_ranges = {
    "learning_rate": (1e-6, 1e-4),  # Log-uniform distribution
    "lora_rank": [4, 8, 16, 32, 64],
    "kl_coef": (0.01, 0.5),
}

best_accuracy = 0
best_params = None

# Random sampling N times
N = 10
for i in range(N):
    # Randomly sample parameters
    lr = 10 ** random.uniform(-6, -4)  # Log-uniform
    rank = random.choice(param_ranges["lora_rank"])
    kl = random.uniform(0.01, 0.5)

    print(f"[{i+1}/{N}] Testing parameters: lr={lr:.2e}, rank={rank}, kl={kl:.3f}")

    # Train and evaluate (same as above)
    # ...

print(f"Best parameters: {best_params}")
print(f"Best accuracy: {best_accuracy:.2%}")
```

무작위 검색의 장점은 효율성이 높고 매개변수 공간이 큰 경우에 적합합니다. 단점은 최적의 솔루션을 놓칠 수 있다는 것입니다.

**(3) 베이지안 최적화**

베이지안 최적화는 확률 모델을 사용하여 검색을 보다 지능적으로 안내합니다. Optuna와 같은 라이브러리를 사용할 수 있습니다.

```python
import optuna

def objective(trial):
    """Optimization objective function"""
    # Sample parameters
    lr = trial.suggest_loguniform("learning_rate", 1e-6, 1e-4)
    rank = trial.suggest_categorical("lora_rank", [8, 16, 32])
    kl = trial.suggest_uniform("kl_coef", 0.01, 0.5)

    # Train model
    result = rl_tool.run({
        "action": "train",
        "algorithm": "grpo",
        "learning_rate": lr,
        "lora_rank": rank,
        "kl_coef": kl,
        # Other parameters...
    })

    # Evaluate model
    eval_result = rl_tool.run({
        "action": "evaluate",
        "model_path": result["model_path"],
    })

    return eval_result["accuracy"]

# Create study
study = optuna.create_study(direction="maximize")
study.optimize(objective, n_trials=20)

# Print best parameters
print(f"Best parameters: {study.best_params}")
print(f"Best accuracy: {study.best_value:.2%}")
```

베이지안 최적화의 장점은 높은 샘플 효율성이며, 좋은 매개변수를 빠르게 찾을 수 있다는 것입니다. 단점은 구현이 복잡하고 추가 라이브러리가 필요하다는 것입니다.

표 11.8은 다양한 튜닝 방법을 비교한 것입니다.

<div align="center">
  <p>표 11.8 하이퍼파라미터 튜닝 방법 비교</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-8.png" alt="" width="85%"/>
</div>

11.6.3 분산 훈련

데이터 볼륨과 모델 규모가 증가하면 단일 GPU 교육이 매우 느려집니다. 이 시점에서는 훈련 프로세스를 가속화하기 위해 분산 훈련을 사용해야 합니다. HelloAgents는 TRL과 Hugging Face Accelerate를 기반으로 다중 GPU 및 다중 노드 분산 학습을 자연스럽게 지원합니다.

**솔루션 선택 권장사항**:

- **단일 머신 다중 GPU(카드 2~8개)**: DDP 사용, 간단하고 효율적
- **대형 모델(>7B)**: DeepSpeed ZeRO-2 또는 ZeRO-3 사용
- **다중 노드 클러스터**: DeepSpeed ZeRO-3 + 오프로드 사용

**(1) 가속 구성**

먼저 Accelerate 구성 파일을 생성해야 합니다. 다음 명령을 실행하십시오.

```bash
accelerate config
```

프롬프트에 따라 구성을 선택하십시오.

```
In which compute environment are you running?
> This machine

Which type of machine are you using?
> multi-GPU

How many different machines will you use?
> 1

Do you wish to optimize your script with torch dynamo?
> NO

Do you want to use DeepSpeed?
> YES

Which DeepSpeed config file do you want to use?
> ZeRO-2

How many GPU(s) should be used for distributed training?
> 4
```

그러면 `~/.cache/huggingface/accelerate/default_config.yaml`에 구성 파일이 생성됩니다.

**(2) DDP를 통한 교육**

**데이터 병렬(DDP)**은 가장 간단한 분산 솔루션으로, 각 GPU는 완전한 모델 복사본을 보유하고 데이터는 GPU 전체에 분할됩니다.

**구성 파일 가속화**(`multi_gpu_ddp.yaml`):

```yaml
compute_environment: LOCAL_MACHINE
distributed_type: MULTI_GPU
num_processes: 4  # Number of GPUs
machine_rank: 0
num_machines: 1
gpu_ids: all
mixed_precision: fp16
```

**교육 스크립트**(수정 필요 없음):

```python
from hello_agents.tools import RLTrainingTool

rl_tool = RLTrainingTool()

# Training code remains unchanged
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/grpo_ddp",
    "num_epochs": 3,
    "batch_size": 4,  # Batch size per GPU
    "use_lora": True,
})
```

**교육 개시**:

```bash
# Using configuration file
accelerate launch --config_file multi_gpu_ddp.yaml train_script.py

# Or directly specify parameters
accelerate launch --num_processes 4 --mixed_precision fp16 train_script.py
```

**(3) DeepSpeed ZeRO를 사용한 훈련**

**DeepSpeed ZeRO**는 최적화 프로그램 상태, 기울기 및 모델 매개변수를 샤딩하여 더 큰 모델과 배치 크기를 지원함으로써 메모리 사용량을 크게 줄입니다.

**ZeRO-2 구성 파일**(`deepspeed_zero2.yaml`):

```yaml
compute_environment: LOCAL_MACHINE
distributed_type: DEEPSPEED
num_processes: 4
machine_rank: 0
num_machines: 1
gpu_ids: all
mixed_precision: fp16
deepspeed_config:
  gradient_accumulation_steps: 4
  gradient_clipping: 1.0
  offload_optimizer_device: none
  offload_param_device: none
  zero3_init_flag: false
  zero_stage: 2  # ZeRO-2
```

**ZeRO-3 구성 파일**(`deepspeed_zero3.yaml`):

```yaml
compute_environment: LOCAL_MACHINE
distributed_type: DEEPSPEED
num_processes: 4
machine_rank: 0
num_machines: 1
gpu_ids: all
mixed_precision: fp16
deepspeed_config:
  gradient_accumulation_steps: 4
  gradient_clipping: 1.0
  offload_optimizer_device: cpu  # Offload optimizer states to CPU
  offload_param_device: cpu      # Offload parameters to CPU
  zero3_init_flag: true
  zero_stage: 3  # ZeRO-3
```

**교육 개시**:

```bash
# ZeRO-2
accelerate launch --config_file deepspeed_zero2.yaml train_script.py

# ZeRO-3
accelerate launch --config_file deepspeed_zero3.yaml train_script.py
```

표 11.9에 표시된 것처럼 이는 다양한 방법으로 Qwen3-0.6B 모델을 훈련하기 위한 메모리 비교입니다.

<div align="center">
  <p>표 11.9 메모리 비교 (Qwen3-0.6B 모델)</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-9.png" alt="" width="85%"/>
</div>

**(4) 다중 노드 훈련**

초대형 훈련의 경우 여러 노드(머신)를 사용할 수 있습니다.

**기본 노드 구성**(`multi_node_main.yaml`):

```yaml
compute_environment: LOCAL_MACHINE
distributed_type: DEEPSPEED
num_processes: 16  # 4 nodes x 4 GPUs
machine_rank: 0    # Main node
num_machines: 4
main_process_ip: 192.168.1.100  # Main node IP
main_process_port: 29500
gpu_ids: all
mixed_precision: fp16
deepspeed_config:
  zero_stage: 3
  offload_optimizer_device: cpu
  offload_param_device: cpu
```

**작업자 노드 구성**(`machine_rank`을 1, 2, 3으로 수정):

```yaml
machine_rank: 1  # Worker node 1
# Other configurations same
```

**교육 개시**:

```bash
# On main node
accelerate launch --config_file multi_node_main.yaml train_script.py

# On worker node 1
accelerate launch --config_file multi_node_worker1.yaml train_script.py

# On worker node 2
accelerate launch --config_file multi_node_worker2.yaml train_script.py

# On worker node 3
accelerate launch --config_file multi_node_worker3.yaml train_script.py
```

**(5) 분산 교육 모범 사례**

**1. 배치 크기 조정**

분산 학습에서 총 배치 크기 = `per_device_batch_size × num_gpus × gradient_accumulation_steps`

```python
# Single GPU: batch_size=4, gradient_accumulation=4, total_batch=16
# 4GPU DDP: batch_size=4, gradient_accumulation=1, total_batch=16 (keep consistent)
```

**2. 학습률 조정**

선형 확장 규칙 사용: `lr_new = lr_base × sqrt(total_batch_size_new / total_batch_size_base)`

```python
# Baseline: single GPU, batch=16, lr=5e-5
# 4GPU: batch=64, lr=5e-5 × sqrt(64/16) = 1e-4
```

**3. 모니터링 및 디버깅**

```python
# Enable verbose logging
export ACCELERATE_LOG_LEVEL=INFO

# Enable NCCL debugging (multi-node)
export NCCL_DEBUG=INFO

# Check GPU utilization
watch -n 1 nvidia-smi
```

### 11.6.4 프로덕션 배포

훈련이 완료된 후에는 모델을 프로덕션 환경에 배포해야 합니다. 다음은 몇 가지 배포 권장 사항입니다.

**(1) 모델 내보내기**

더 쉬운 배포를 위해 LoRA 가중치를 기본 모델에 병합합니다.

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
from peft import PeftModel

# Load base model
base_model = AutoModelForCausalLM.from_pretrained("Qwen/Qwen3-0.6B")

# Load LoRA weights
model = PeftModel.from_pretrained(base_model, "./models/grpo_model")

# Merge weights
merged_model = model.merge_and_unload()

# Save merged model
merged_model.save_pretrained("./models/merged_model")

# Save tokenizer
tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen3-0.6B")
tokenizer.save_pretrained("./models/merged_model")

print("✓ Model exported to: ./models/merged_model")
```

**(2) 추론 최적화**

추론을 가속화하기 위해 양자화 및 최적화 기술을 사용합니다.

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
import torch

# Load model (using 8-bit quantization)
model = AutoModelForCausalLM.from_pretrained(
    "./models/merged_model",
    load_in_8bit=True,  # 8-bit quantization
    device_map="auto",  # Auto device allocation
)

tokenizer = AutoTokenizer.from_pretrained("./models/merged_model")

# Inference
def generate_answer(question):
    prompt = f"<|im_start|>user\n{question}<|im_end|>\n<|im_start|>assistant\n"
    inputs = tokenizer(prompt, return_tensors="pt").to(model.device)

    outputs = model.generate(
        **inputs,
        max_new_tokens=512,
        temperature=0.7,
        do_sample=True,
    )

    response = tokenizer.decode(outputs[0], skip_special_tokens=False)
    return response

# Test
question = "What is 48 + 24?"
answer = generate_answer(question)
print(answer)
```

**(3) API 서비스**

FastAPI를 사용하여 추론 서비스를 생성합니다.

```python
from fastapi import FastAPI
from pydantic import BaseModel
from transformers import AutoModelForCausalLM, AutoTokenizer

app = FastAPI()

# Load model
model = AutoModelForCausalLM.from_pretrained("./models/merged_model")
tokenizer = AutoTokenizer.from_pretrained("./models/merged_model")

class Question(BaseModel):
    text: str
    max_tokens: int = 512

class Answer(BaseModel):
    text: str
    confidence: float

@app.post("/generate", response_model=Answer)
def generate(question: Question):
    """Generate answer"""
    prompt = f"<|im_start|>user\n{question.text}<|im_end|>\n<|im_start|>assistant\n"
    inputs = tokenizer(prompt, return_tensors="pt")

    outputs = model.generate(
        **inputs,
        max_new_tokens=question.max_tokens,
        temperature=0.7,
        return_dict_in_generate=True,
        output_scores=True,
    )

    response = tokenizer.decode(outputs.sequences[0], skip_special_tokens=False)

    # Calculate confidence (simplified version)
    confidence = 0.8  # Should actually be calculated based on output probabilities

    return Answer(text=response, confidence=confidence)

# Run: uvicorn api:app --host 0.0.0.0 --port 8000
```



## 11.7 장 요약

이 장에서는 기본 개념부터 학습 파이프라인 완성, 데이터 준비부터 모델 배포까지 Agentic RL의 이론과 실습을 체계적으로 배웠습니다. 이 장의 주요 내용을 검토해 보겠습니다.

**(1) Agentic RL의 본질**

Agentic RL은 LLM을 학습 가능한 정책으로 취급하여 이를 에이전트의 인식-결정-실행 루프에 삽입하고 강화 학습을 통해 다단계 작업에서 에이전트 성능을 최적화합니다. 기존 PBRFT(Preference-Based Reinforcement Fine-Tuning)와의 핵심 차이점은 다음과 같습니다.

- **작업 성격**: 단일 회전 대화 최적화부터 다단계 순차적 의사 결정까지
- **상태 공간**: 정적 프롬프트에서 동적으로 진화하는 환경 상태까지
- **Action Space**: 순수 텍스트 생성부터 텍스트 + 도구 + 환경 작업까지
- **보상 설계**: 단일 단계 품질 평가부터 장기 누적 수익률까지
- **최적화 목표**: 단기적인 대응 품질부터 장기적인 작업 성공까지

**(2) 6가지 핵심 기능**

Agentic RL은 에이전트의 6가지 핵심 기능을 향상시키는 것을 목표로 합니다.

1. **추론**: 다단계 논리적 추론, 추론 전략 학습
2. **도구 사용**: API/도구 호출, 사용 시기 및 방법 학습
3. **기억**: 장기 정보 보유, 학습 기억 관리
4. **계획**: 동작 순서 계획, 동적 계획 학습
5. **자기 개선**: 자기 성찰 최적화, 실수로부터 배우기
6. **지각**: 다중 모드 이해, 시각적 추론 및 도구 사용

**(3) 훈련 파이프라인**

완전한 Agentic RL 교육 파이프라인에는 다음이 포함됩니다.

1. **사전 학습**: 대규모 텍스트에 대한 언어 지식 학습(보통 기존 사전 학습 모델 사용)
2. **Supervised Fine-Tuning (SFT)**: 학습 과제 형식 및 기본 추론 능력
3. **강화 학습(RL)**: 시행착오를 통해 추론 전략을 최적화하고 훈련 데이터 품질을 능가합니다.

이 중 SFT가 기초이고 RL이 향상입니다. SFT 기반이 없으면 RL은 성공하기 어렵습니다. RL 최적화가 없으면 모델은 훈련 데이터만 모방할 수 있습니다.

Agentic RL을 자세히 배우고 싶다면 다음 경로를 따르는 것이 좋습니다.

**기초 단계**

1. **강화 학습 기본**: MDP, 정책 변화도, PPO와 같은 기본 개념을 알아보세요.
2. **LLM 기초**: Transformer, 사전 훈련, 미세 조정과 같은 기술 이해
3. **HelloAgents 연습**: 이 장의 예제 코드를 실행하고 전체 파이프라인을 이해합니다.

**고급 단계**

1. **TRL 심층 분석**: TRL 라이브러리 구현을 배우고 SFT 및 GRPO와 같은 알고리즘의 세부 사항을 이해합니다.
2. **맞춤형 데이터세트**: 자체 데이터세트를 사용하여 모델 학습
3. **맞춤형 보상 기능**: 업무에 적합한 보상 기능 설계
4. **매개변수 조정**: 하이퍼 매개변수를 체계적으로 조정하고 모델 성능을 향상시킵니다.

**전문가 단계**

1. **다단계 추론**: 긴 순서 추론 작업 연구
2. **도구 학습**: 상담원이 도구 사용을 배울 수 있도록 지원
3. **다중 에이전트**: 다중 에이전트 협업 연구
4. **최첨단 논문**: 최신 연구 논문을 읽고 개척 발전을 따르세요.



이 장이 Agentic RL 기술을 이해하고 숙달하고, 이 지식을 자신의 프로젝트에 적용하고, 보다 지능적인 Agent 시스템을 구축하는 데 도움이 되기를 바랍니다.



## 참고자료

[1] Schulman, J., Wolski, F., Dhariwal, P., Radford, A., & Klimov, O. (2017). 근위 정책 최적화 알고리즘. *arXiv 사전 인쇄본 arXiv:1707.06347*.

[2] Shao, Z., Wang, P., Zhu, Q., Xu, R., Song, J., Zhang, M., ... & Guo, D. (2024). DeepSeekMath: 개방형 언어 모델에서 수학적 추론의 한계를 뛰어넘습니다. *arXiv 사전 인쇄본 arXiv:2402.03300*.

[3] Hu, E. J., Shen, Y., Wallis, P., Allen-Zhu, Z., Li, Y., Wang, S., ... & Chen, W. (2021). LoRA: 대규모 언어 모델의 낮은 순위 적응. *arXiv 사전 인쇄본 arXiv:2106.09685*.

[4] Cobbe, K., Kosaraju, V., Bavarian, M., Chen, M., Jun, H., Kaiser, L., ... & Schulman, J. (2021). 수학 단어 문제를 해결하기 위한 훈련 검증기. *arXiv 사전 인쇄본 arXiv:2110.14168*.

[5] Ouyang, L., Wu, J., Jiang, X., Almeida, D., Wainwright, C., Mishkin, P., ... & Lowe, R. (2022). 인간의 피드백을 통해 지침을 따르도록 언어 모델을 훈련합니다. *신경 정보 처리 시스템의 발전*, 35, 27730-27744.

[6] Rafailov, R., Sharma, A., Mitchell, E., Ermon, S., Manning, C. D., & Finn, C. (2023). 직접 선호 최적화: 언어 모델은 비밀리에 보상 모델입니다. *arXiv 사전 인쇄본 arXiv:2305.18290*.

[7] Lee, H., Phatale, S., Mansoor, H., Lu, K., Mesnard, T., Bishop, C., ... & Rastogi, A. (2023). RLAIF: AI 피드백을 사용하여 인간 피드백에서 강화 학습 확장. *arXiv 사전 인쇄본 arXiv:2309.00267*.

[8] Wei, J., Wang, X., Schuurmans, D., Bosma, M., Ichter, B., Xia, F., ... & Zhou, D. (2022). 생각의 연쇄 유도는 대규모 언어 모델에서 추론을 이끌어냅니다. *신경 정보 처리 시스템의 발전*, 35, 24824-24837.

[9] von Werra, L., Belkada, Y., Tunstall, L., Beeching, E., Thrush, T., Lambert, N., & Huang, S. (2020). TRL: 변압기 강화 학습. *GitHub 저장소*. https://github.com/huggingface/trl

[10] Qwen 팀. (2025). Qwen3 기술 보고서. *arXiv 사전 인쇄본 arXiv:2505.09388*.

[11] Bai, Y., Jones, A., Ndousse, K., Askell, A., Chen, A., DasSarma, N., ... & Kaplan, J. (2022). 인간 피드백을 통한 강화 학습을 통해 유용하고 무해한 보조자를 훈련합니다. *arXiv 사전 인쇄본 arXiv:2204.05862*.

[12] Wang, X., Wei, J., Schuurmans, D., Le, Q., Chi, E., Narang, S., ... & Zhou, D. (2022). 자기 일관성은 언어 모델의 사고 추론을 향상시킵니다. *arXiv 사전 인쇄본 arXiv:2203.11171*.

[13] Christiano, P. F., Leike, J., Brown, T., Martic, M., Legg, S., & Amodei, D. (2017). 인간 선호도로부터의 심층 강화 학습. *신경 정보 처리 시스템의 발전*, 30.

[14] Stiennon, N., Ouyang, L., Wu, J., Ziegler, D., Lowe, R., Voss, C., ... & Christiano, P. F. (2020). 인간의 피드백을 통해 요약하는 방법을 학습합니다. *신경 정보 처리 시스템의 발전*, 33, 3008-3021.

[15] Ziegler, D. M., Stiennon, N., Wu, J., Brown, T. B., Radford, A., Amodei, D., ... & Irving, G. (2019). 인간의 선호에 따른 언어 모델 미세 조정. *arXiv 사전 인쇄 arXiv:1909.08593*.

## 연습

> **참고**: 일부 연습에는 표준 답변이 없습니다. Agentic RL 및 에이전트 교육에 대한 학습자의 포괄적인 이해와 실무 능력을 배양하는 데 중점을 두고 있습니다.

1. 이 장에서는 LLM 교육에서 Agentic RL로의 발전을 소개했습니다. 분석해 주십시오:

   - 섹션 11.1.3의 Table 11.1에서는 MDP 프레임워크 하에서 PBRFT(Preference-Based Reinforcement Fine-Tuning)와 Agentic RL의 차이점을 비교합니다. 자세히 설명해주세요. Agentic RL의 상태 공간 $s_t = (\text{prompt}, o_1, o_2, ..., o_t)$에는 기록 관찰이 포함되는 반면, PBRFT의 상태 $s_0 = \text{prompt}$에는 초기 프롬프트만 포함되는 이유는 무엇입니까? 이러한 차이가 훈련 과정과 최종 결과에 어떤 영향을 미치나요?
   - (1) 버그를 찾기 위해 코드를 분석하고; (2) API 사용법을 이해하려면 문서를 참조하세요. (3) 코드를 수정합니다. (4) 수정 효과를 확인하기 위해 테스트를 실행합니다. 이 작업을 강화 학습 프레임워크에 매핑하여 상태 공간, 행동 공간, 보상 기능 및 상태 전환 기능을 명확하게 정의하십시오.
   - 섹션 11.1.1에서는 전통적인 지도 학습이 "장기 목표를 최적화하기 어렵다"는 한계가 있다고 언급했습니다. 지도 학습이 중간 단계를 최적화하는 데 어려움을 겪는 이유를 보여주는 특정 다단계 추론 작업(예: 수학적 증명, 복잡한 문제 해결)을 설계하고 강화 학습은 지연된 보상을 통해 이 문제를 해결할 수 있습니다.

2. SFT(Supervised Fine-Tuning)와 GRPO(Group Relative Policy Optimization)는 이 장의 두 가지 핵심 훈련 방법입니다. 섹션 11.2 및 11.3을 바탕으로 깊이 생각해 보십시오.

   > **참고**: 이것은 실습 연습 문제이므로 실제 작동을 권장합니다.

- 섹션 11.2.4의 SFT 훈련 코드에서는 훈련 매개변수를 줄이기 위해 LoRA(Low-Rank Adaptation) 기술을 사용했습니다. 분석해 보세요: LoRA의 핵심 아이디어는 무엇입니까? 적은 수의 매개변수(예: 0.16%)로 전체 매개변수 미세 조정에 가까운 효과를 얻을 수 있는 이유는 무엇입니까? 어떤 상황에서 전체 매개변수 미세 조정 대신 LoRA를 선택해야 합니까?
   - GRPO 알고리즘(11.3절)은 기존 PPO 알고리즘과 비교하여 어떤 장점이 있습니까? GRPO가 "그룹 상대적 보상"을 통해 훈련 과정을 단순화하고 안정성을 향상시키는 방법을 분석하여 두 가지의 훈련 과정을 비교하십시오. GRPO를 다른 작업(예: 코드 생성, 대화 최적화)에 적용하는 경우 어떤 조정이 필요합니까?
   - 섹션 11.2.5의 코드를 기반으로 SFT 훈련 파이프라인을 확장하여 다음 기능을 추가하십시오: (1) 다중 회전 대화 데이터 훈련 지원; (2) 데이터 확대 전략(예: 동의어 재작성, 난이도 조정)을 추가합니다. (3) 훈련 과정의 시각화 모니터링(예: 손실 곡선, 샘플 품질 평가)을 구현합니다.

3. 보상 기능 설계는 Agentic RL의 핵심 과제입니다. 섹션 11.3.3을 바탕으로 다음 확장 연습을 완료하세요.

   > **참고**: 이것은 실습 연습 문제이므로 실제 작동을 권장합니다.

   - 섹션 11.3.3에서는 GSM8K 수학 문제(올바른 +1, 잘못된 0)에 대한 간단한 이진 보상을 설계했습니다. (1) 부분적으로 정답에 대해 부분적인 보상을 제공할 수 있는 보다 세련된 보상 기능을 설계하십시오. (2) 추론 과정의 합리성을 평가합니다. (3) 지나치게 장황하거나 비효율적인 솔루션 경로에 불이익을 줍니다. 이 보상 기능은 어떻게 구현되어야 합니까?
   - 보상 기능 설계에는 도메인 지식이 필요한 경우가 많습니다. 다음 세 가지 에이전트 작업에 대한 보상 기능을 설계하십시오. (1) 코드 생성 보조(코드 정확성, 가독성, 효율성을 고려해야 함); (2) 고객 서비스 대화 에이전트(문제 해결률, 사용자 만족도, 응답 시간을 고려해야 함) (3) 게임 AI(승률, 전략 다양성, 적대적 견고성 고려 필요)
   - 실제 적용에서 보상 기능에는 "보상 해킹" 문제가 있을 수 있습니다. 에이전트는 높은 보상을 얻기 위한 지름길을 찾지만 실제로 작업을 완료하지는 않습니다. 이러한 현상의 예를 들어보고 보상해킹을 방지하기 위한 방어 메커니즘을 설계해 주세요.

4. 섹션 11.4의 "수학적 추론 에이전트 훈련" 사례에서 우리는 완전한 훈련 파이프라인을 보았습니다. 심층적으로 분석해 보세요.

- 훈련과 평가를 위해 GSM8K 데이터세트를 사용한 사례입니다. 분석해 보세요. 이 데이터세트의 특징은 무엇인가요? 어떤 종류의 추론 능력이 훈련에 적합합니까? 보다 복잡한 수학적 문제(예: 고급 수학, 수학적 증명)를 처리할 수 있는 에이전트를 교육하는 경우 데이터 세트와 교육 방법을 어떻게 확장해야 합니까?
   - 11.4.3절의 훈련 결과에서 훈련 세트의 정확도 향상이 관찰되었으나 과적합 위험이 있을 수 있습니다. "일반화 능력 평가" 계획을 설계해 주십시오. 모델이 훈련 데이터를 암기하는 것이 아니라 실제로 수학적 추론을 학습했는지 테스트하는 방법은 무엇입니까? 정규화, 데이터 증대 및 기타 기술을 통해 일반화 능력을 향상시키는 방법은 무엇입니까?
   - 해당 사례의 교육은 오프라인입니다(사전 수집된 데이터 세트 사용). "온라인 학습" 계획을 설계하십시오. 에이전트는 실제 사용 중에 지속적으로 사용자 피드백을 수집하고 자동으로 모델을 업데이트합니다. 이 계획에서는 어떤 기술적 과제(예: 데이터 품질 관리, 치명적인 망각, 안전 보장)를 고려해야 합니까?

5. Agentic RL의 중요한 응용 프로그램은 에이전트가 도구 사용을 배울 수 있도록 하는 것입니다. 생각해 보십시오:

   - 섹션 11.1.3에서는 Agentic RL이 "다단계 추론, 도구 사용, 장기 계획이 필요한" 작업을 최적화하는 데 적합하다고 언급했습니다. "도구 학습" 교육 계획을 설계하십시오. 도구 세트(예: 검색 엔진, 계산기, 코드 실행기)가 제공되면 에이전트가 적절한 시기에 적절한 도구를 선택하는 방법을 배우도록 교육하는 방법은 무엇입니까? 보상 기능은 어떻게 설계되어야 합니까?
   - 도구 사용에는 종종 복잡한 종속성이 포함됩니다(예: "도구 B를 호출하기 전에 정보를 얻으려면 먼저 도구 A를 호출해야 합니다"). "계층적 강화 학습" 계획을 설계하십시오: 작업 계획을 담당하는 상위 수준 정책, 도구 호출을 담당하는 하위 수준 정책. 이 계층 구조를 훈련하는 방법은 무엇입니까? 높은 수준과 낮은 수준의 최적화 목표를 조정하는 방법은 무엇입니까?
   - 실제 응용 프로그램에서는 도구 수가 매우 많을 수 있으며(예: 50개 이상의 API) 직접 교육은 "낮은 탐색 효율성" 문제에 직면할 수 있습니다. "커리큘럼 학습" 계획을 설계하십시오. 간단한 작업(몇 가지 도구 사용)부터 훈련을 시작하고 점차적으로 작업 난이도와 도구 수를 늘리십시오. 이 계획은 커리큘럼 순서를 어떻게 설계해야 합니까? 상담원이 다음 단계에 들어갈 준비가 되었는지 어떻게 평가하나요?

