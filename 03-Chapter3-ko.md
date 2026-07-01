# 3장: 대규모 언어 모델의 기초

처음 두 장에서는 에이전트의 정의와 개발 역사를 소개했습니다. 이 장에서는 다음과 같은 주요 질문에 답하기 위해 대규모 언어 모델 자체에 전적으로 초점을 맞출 것입니다: 현대 에이전트는 어떻게 작동합니까? 우리는 언어 모델의 기본 정의부터 시작하여 이러한 원리를 학습함으로써 LLM이 어떻게 강력한 지식 보유량과 추론 능력을 획득하는지 이해하기 위한 탄탄한 기반을 마련할 것입니다.

## 3.1 언어 모델 및 변환기 아키텍처

### 3.1.1 N-gram에서 RNN으로

**언어 모델(LM)**은 자연어 처리의 핵심이며, 기본 작업은 단어 시퀀스(예: 문장)가 나타날 확률을 계산하는 것입니다. 좋은 언어 모델은 어떤 종류의 문장이 유창하고 자연스러운지 알려줄 수 있습니다. 다중 에이전트 시스템에서 언어 모델은 에이전트가 인간의 지시를 이해하고 응답을 생성하는 기반입니다. 이 섹션에서는 고전적인 통계 방법에서 최신 딥 러닝 모델로의 진화를 검토하여 후속 Transformer 아키텍처를 이해하기 위한 견고한 기반을 마련합니다.

**(1) 통계적 언어 모델과 N-gram 아이디어**

딥러닝이 등장하기 전에는 통계적 방법이 언어 모델의 주류였습니다. 핵심 아이디어는 문장이 나타날 확률이 문장에 있는 각 단어의 조건부 확률의 곱과 같다는 것입니다. 단어 $w_1,w_2,\cdots,w_m$으로 구성된 문장 S의 경우 확률 P(S)는 다음과 같이 표현될 수 있습니다.

$$P(S)=P(w_1,w_2,…,w_m)=P(w_1)⋅P(w_2∣w_1)⋅P(w_3∣w_1,w_2)⋯P(w_m∣w_1,…,w_{m−1})$$

이 공식을 확률의 연쇄 법칙이라고 합니다. 그러나 $P(w_m∣w_1,\cdots,w_{m−1})$와 같은 조건부 확률은 코퍼스에서 추정하기가 너무 어렵기 때문에 이 공식을 직접 계산하는 것은 거의 불가능합니다. 단어 시퀀스 $w_1,\cdots,w_{m−1}$가 훈련 데이터에 나타나지 않았을 수 있기 때문입니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/3-figures/1757249275674-0.png" alt="Figure description" width="90%"/>
<p>그림 3.1 마르코프 가정의 개략도</p>
</div>

이 문제를 해결하기 위해 연구자들은 **마르코프 가정**을 도입했습니다. 핵심 아이디어는 다음과 같습니다. 단어의 전체 역사를 추적할 필요가 없습니다. 그림 3.1에서 볼 수 있듯이 단어가 나타날 확률은 그 앞에 있는 제한된 $n−1$ 단어에만 관련되어 있다고 대략적으로 가정할 수 있습니다. 이러한 가정을 바탕으로 구축된 언어 모델을 **N-gram 모델**이라고 합니다. 여기서 "N"은 우리가 고려하는 컨텍스트 창 크기를 나타냅니다. 이 개념을 이해하기 위해 가장 일반적인 몇 가지 예를 살펴보겠습니다.

- **Bigram(N=2인 경우)**: 이는 단어의 모양이 그 앞의 한 단어에만 관련되어 있다고 가정하는 가장 간단한 경우입니다. 따라서 체인 규칙의 복잡한 조건부 확률 $P(w_i∣w_1,\cdots,w_{i−1})$는 보다 쉽게 ​​계산할 수 있는 형식으로 근사화될 수 있습니다.

$$P(w_{i}∣w_{1},…,w_{i−1})≒P(w_{i}∣w_{i−1})$$

- **삼각형(N=3인 경우)**: 마찬가지로 단어의 모양은 그 앞의 두 단어에만 관련되어 있다고 가정합니다.

$$P(w_i∣w_1,…,w_{i−1})≒P(w_i∣w_{i−2},w_{i−1})$$

이러한 확률은 대규모 말뭉치에서 **Maximum Likelihood Estimation(MLE)**을 통해 계산할 수 있습니다. 이 용어는 복잡해 보이지만 그 개념은 매우 직관적입니다. 나타날 가능성이 가장 높은 것은 데이터에서 가장 자주 보는 것입니다. 예를 들어, Bigram 모델의 경우 $w_{i−1}$ 단어가 나타난 후 다음 단어가 $w_i$일 확률 $P(w_i∣w_{i−1})$를 계산하려고 합니다. 최대 우도 추정에 따르면 이 확률은 간단한 계산을 통해 추정할 수 있습니다.

$$P(w_i∣w_{i−1})=\frac{수(w_{i−1},w_i)}{수(w_{i−1})}$$

여기서 `Count()` 함수는 "계산"을 나타냅니다.

- $Count(w_{i−1},w_i)$: 단어 쌍 $(w_{i−1},w_i)$가 말뭉치에 연속적으로 나타나는 총 횟수를 나타냅니다.
- ​​$Count(w_{i−1})$: 단일 단어 $w_{i−1}$가 말뭉치에 나타나는 총 횟수를 나타냅니다.

공식의 의미는 "단어 쌍 $Count(w_{i−1},w_i)$가 나타나는 횟수"를 "단어 $Count(w_{i−1})$가 나타나는 총 횟수"로 나누어 $P(w_i∣w_{i−1})$의 대략적인 추정치로 사용한다는 것입니다.

이 프로세스를 보다 구체적으로 만들기 위해 수동으로 계산을 수행해 보겠습니다. `datawhale agent learns`, `datawhale agent works`라는 두 문장만 포함하는 미니 코퍼스가 있다고 가정합니다. 우리의 목표는 Bigram(N=2) 모델을 사용하여 `datawhale agent learns`라는 문장이 나타날 확률을 추정하는 것입니다. Bigram 가정에 따르면, 우리는 매번 연속적인 단어 쌍(즉, 단어 쌍)을 검사합니다.

**1단계: 첫 번째 단어의 확률 계산** $P(datawhale)$ `datawhale`가 나타나는 횟수를 전체 단어 수로 나눈 값입니다. `datawhale`는 2회 등장하며, 총 단어 수는 6개입니다.

$$P(\text{datawhale}) = \frac{\text{전체 말뭉치의 "datawhale" 수}}{\text{말뭉치의 총 단어 수}} = \frac{2}{6} ≈ 0.333$$

**2단계: 조건부 확률 계산** $P(agent∣datawhale)$ `datawhale agent`라는 단어 쌍이 나타나는 횟수를 `datawhale`가 나타나는 전체 횟수로 나눈 값입니다. `datawhale agent`가 2번 나타나고, `datawhale`가 2번 나타납니다.

$$P(\text{에이전트}|\text{datawhale}) = \frac{\text{개수}(\text{datawhale 에이전트})}{\text{개수}(\text{datawhale})} = \frac{2}{2} = 1$$

**3단계: 조건부 확률 계산** $P(learns∣agent)$ `agent learns`라는 단어 쌍이 나타나는 횟수를 `agent`가 나타나는 총 횟수로 나눈 값입니다. `agent learns`가 1번 나타나고, `agent`가 2번 나타납니다.

$$P(\text{학습}|\text{에이전트}) = \frac{\text{수(에이전트 학습)}}{\text{수(에이전트)}} = \frac{1}{2} = 0.5$$

**마지막으로: 확률을 곱합니다** 따라서 전체 문장의 대략적인 확률은 다음과 같습니다.

$$P(\text{datawhale 에이전트 학습}) \대략 P(\text{datawhale}) \cdot P(\text{에이전트}|\text{datawhale}) \cdot P(\text{학습}|\text{에이전트}) \대략 0.333 \cdot 1 \cdot 0.5 \대략 0.167$$

```Python
import collections

# Example corpus, consistent with the corpus in the case explanation above
corpus = "datawhale agent learns datawhale agent works"
tokens = corpus.split()
total_tokens = len(tokens)

# --- Step 1: Calculate P(datawhale) ---
count_datawhale = tokens.count('datawhale')
p_datawhale = count_datawhale / total_tokens
print(f"Step 1: P(datawhale) = {count_datawhale}/{total_tokens} = {p_datawhale:.3f}")

# --- Step 2: Calculate P(agent|datawhale) ---
# First calculate bigrams for subsequent steps
bigrams = zip(tokens, tokens[1:])
bigram_counts = collections.Counter(bigrams)
count_datawhale_agent = bigram_counts[('datawhale', 'agent')]
# count_datawhale was already calculated in step 1
p_agent_given_datawhale = count_datawhale_agent / count_datawhale
print(f"Step 2: P(agent|datawhale) = {count_datawhale_agent}/{count_datawhale} = {p_agent_given_datawhale:.3f}")

# --- Step 3: Calculate P(learns|agent) ---
count_agent_learns = bigram_counts[('agent', 'learns')]
count_agent = tokens.count('agent')
p_learns_given_agent = count_agent_learns / count_agent
print(f"Step 3: P(learns|agent) = {count_agent_learns}/{count_agent} = {p_learns_given_agent:.3f}")

# --- Finally: Multiply the probabilities ---
p_sentence = p_datawhale * p_agent_given_datawhale * p_learns_given_agent
print(f"Finally: P('datawhale agent learns') ≈ {p_datawhale:.3f} * {p_agent_given_datawhale:.3f} * {p_learns_given_agent:.3f} = {p_sentence:.3f}")

>>>
Step 1: P(datawhale) = 2/6 = 0.333
Step 2: P(agent|datawhale) = 2/2 = 1.000
Step 3: P(learns|agent) = 1/2 = 0.500
Finally: P('datawhale agent learns') ≈ 0.333 * 1.000 * 0.500 = 0.167
```

N-gram 모델은 간단하고 효과적이지만 두 가지 치명적인 결함이 있습니다.

1. **데이터 희소성**: 단어 시퀀스가 ​​코퍼스에 전혀 나타나지 않은 경우 확률 추정치는 0이며 이는 분명히 비합리적입니다. 이는 스무딩 기술을 통해 완화될 수 있지만 근절할 수는 없습니다.
2. **일반화 능력이 낮음**: 모델은 단어 간의 의미적 유사성을 이해할 수 없습니다. 예를 들어 모델이 말뭉치에서 `agent learns`를 여러 번 본 경우에도 이 지식을 의미상 유사한 단어로 일반화할 수 없습니다. `robot learns`의 확률을 계산할 때 `robot`라는 단어가 전혀 나타나지 않거나 `robot learns` 조합이 나타나지 않은 경우 모델에서 계산한 확률도 0이 됩니다. 모델은 `agent`와 `robot` 간의 의미적 유사성을 이해할 수 없습니다.

**(2) 신경망 언어 모델 및 단어 임베딩**

N-gram 모델의 근본적인 결함은 단어를 분리된 개별 기호로 취급한다는 것입니다. 이 문제를 극복하기 위해 연구자들은 신경망으로 전환하여 연속 벡터로 단어를 표현하는 아이디어를 제안했습니다. 2003년 Bengio 등이 제안한 **피드포워드 신경망 언어 모델**. 이 분야의 이정표였습니다<sup>[1]</sup>.

핵심 아이디어는 두 단계로 나눌 수 있습니다.

1. **의미 공간 구축**: 고차원 연속 벡터 공간을 만든 다음 어휘의 각 단어를 해당 공간의 한 지점에 매핑합니다. 이 점(즉, 벡터)을 **단어 임베딩** 또는 단어 벡터라고 합니다. 이 공간에서 의미상 유사한 단어는 위치가 서로 가까운 벡터를 갖습니다. 예를 들어 `agent`와 `robot`의 벡터는 매우 가까운 반면 `agent`와 `apple`의 벡터는 멀리 떨어져 있습니다.
2. **문맥에서 다음 단어까지의 매핑 학습**: 신경망의 강력한 피팅 기능을 활용하여 기능을 학습합니다. 이 함수의 입력은 이전 $n−1$ 단어의 단어 벡터이고, 출력은 현재 문맥 뒤에 나타나는 어휘의 각 단어의 확률 분포입니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/3-figures/1757249275674-1.png" alt="Figure description" width="90%"/>
<p>그림 3.2 신경망 언어 모델 아키텍처의 개략도</p>
</div>

그림 3.2에서 볼 수 있듯이 이 아키텍처에서는 모델 학습 중에 단어 임베딩이 자동으로 학습됩니다. "다음 단어 예측" 작업을 완료하기 위해 모델은 각 단어의 벡터 위치를 지속적으로 조정하여 궁극적으로 이러한 벡터에 풍부한 의미 정보가 포함되도록 만듭니다. 단어를 벡터로 변환하면 수학적 도구를 사용하여 단어 간의 관계를 측정할 수 있습니다. 가장 일반적으로 사용되는 방법은 **코사인 유사성**으로, 두 벡터 사이의 각도의 코사인을 계산하여 유사성을 측정합니다.

$$\text{유사성}(\vec{a}, \vec{b}) = \cos(\theta) = \frac{\vec{a} \cdot \vec{b}}{|\vec{a}| |\vec{b}|}$$

이 수식의 의미는 다음과 같습니다.

- 두 벡터가 정확히 동일한 방향을 갖는 경우 각도는 0°이고 코사인 값은 1로 완전한 상관 관계를 나타냅니다.
- 두 벡터가 직교하는 경우 각도는 90°이고 코사인 값은 0으로 관계가 없음을 나타냅니다.
- 두 벡터의 방향이 완전히 반대인 경우 각도는 180°이고 코사인 값은 -1로 완전한 음의 상관 관계를 나타냅니다.

이 방법을 통해 단어 벡터는 "동의어"와 같은 단순한 관계를 캡처할 수 있을 뿐만 아니라 보다 복잡한 유추 관계도 캡처할 수 있습니다.

유명한 예는 단어 벡터로 캡처된 의미 관계를 보여줍니다. `vector('King') - vector('Man') + vector('Woman')` 이 벡터 연산의 결과는 놀랍게도 벡터 공간에서 `vector('Queen')`의 위치에 가깝습니다. 이는 의미론적 번역을 수행하는 것과 같습니다. "왕"이라는 지점에서 시작하여 "남성"의 벡터를 빼고 "여성"의 벡터를 더한 다음 마지막으로 "여왕"의 위치에 도달합니다. 이는 단어 임베딩이 "성별" 및 " 로열티"와 같은 추상적인 개념을 학습할 수 있음을 증명합니다.

```Python
import numpy as np

# Assume we have learned simplified 2D word vectors
embeddings = {
    "king": np.array([0.9, 0.8]),
    "queen": np.array([0.9, 0.2]),
    "man": np.array([0.7, 0.9]),
    "woman": np.array([0.7, 0.3])
}

def cosine_similarity(vec1, vec2):
    dot_product = np.dot(vec1, vec2)
    norm_product = np.linalg.norm(vec1) * np.linalg.norm(vec2)
    return dot_product / norm_product

# king - man + woman
result_vec = embeddings["king"] - embeddings["man"] + embeddings["woman"]

# Calculate similarity between result vector and "queen"
sim = cosine_similarity(result_vec, embeddings["queen"])

print(f"Result vector of king - man + woman: {result_vec}")
print(f"Similarity of this result with 'queen': {sim:.4f}")

>>>
Result vector of king - man + woman: [0.9 0.2]
Similarity of this result with 'queen': 1.0000
```

신경망 언어 모델은 단어 임베딩을 통해 N-gram 모델의 잘못된 일반화 문제를 성공적으로 해결했습니다. 그러나 여전히 N-gram과 유사한 제한 사항이 있습니다. 즉, 컨텍스트 창이 고정되어 있습니다. 그들은 고정된 개수의 선행 단어만 고려할 수 있으며, 이는 임의 길이의 시퀀스를 처리할 수 있는 순환 신경망의 토대를 마련했습니다.

**(3) 순환 신경망(RNN) 및 장단기 기억 네트워크(LSTM)**

이전 섹션의 신경망 언어 모델은 N-gram 모델처럼 일반화 문제를 해결하기 위해 단어 임베딩을 도입했지만 컨텍스트 창의 크기는 고정되어 있습니다. 다음 단어를 예측하려면 이전 n-1 단어만 볼 수 있으며 이전 기록 정보는 삭제됩니다. 이것은 분명히 우리 인간이 언어를 이해하는 방식과 일치하지 않습니다. 고정 창의 한계를 극복하기 위해 매우 직관적인 핵심 아이디어를 갖춘 **반복 신경망(RNN)**이 등장했습니다. 즉, 네트워크<sup>[2]</sup>에 "메모리" 기능을 추가하는 것입니다.

그림 3.3에서 볼 수 있듯이 RNN의 디자인에는 네트워크의 단기 메모리로 이해할 수 있는 **숨겨진 상태** 벡터가 도입되었습니다. 시퀀스를 처리하는 각 단계에서 네트워크는 현재 입력 단어를 읽고 이를 이전 순간의 메모리(즉, 이전 시간 단계의 숨겨진 상태)와 결합한 다음, 다음 순간으로 전달할 새 메모리(즉, 현재 시간 단계의 숨겨진 상태)를 생성합니다. 이 순환 프로세스를 통해 정보가 시퀀스를 통해 지속적으로 뒤로 전파될 수 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/3-figures/1757249275674-2.png" alt="Figure description" width="90%"/>
<p>그림 3.3 RNN 구조 </p>의 개략도
</div>

그러나 표준 RNN에는 실제로 **장기 종속성 문제**라는 심각한 문제가 있습니다. 훈련 중에 모델은 역전파 알고리즘을 통해 출력 끝의 오류를 기반으로 네트워크 깊은 곳의 가중치를 조정해야 합니다. RNN의 경우 시퀀스 길이는 네트워크의 깊이입니다. 시퀀스가 매우 길면 기울기는 역방향 전파 중에 여러 번의 곱셈을 거치게 되며 이로 인해 기울기 값이 빠르게 0에 가까워지거나(**경사 소멸**) 극도로 커집니다(**경사 폭발**). Gradient Vanishing은 모델이 초기 시퀀스 정보가 이후 출력에 미치는 영향을 효과적으로 학습하지 못하게 하여 장거리 종속성을 포착하기 어렵게 만듭니다.

장기 의존성 문제를 해결하기 위해 **장단기 기억(LSTM)**이 <sup>[3]</sup>로 설계되었습니다. LSTM은 RNN의 특별한 유형이며 핵심 혁신은 **Cell State** 및 정교한 **Gating Mechanism**을 도입하는 데 있습니다. 셀 상태는 숨겨진 상태와 독립적인 정보 경로로 볼 수 있으므로 정보가 시간 단계 간에 보다 원활하게 전달될 수 있습니다. 게이팅 메커니즘은 정보를 선택적으로 전달하는 방법을 학습하여 세포 상태에서 정보의 추가 및 제거를 제어할 수 있는 여러 개의 작은 신경망으로 구성됩니다. 이러한 게이트에는 다음이 포함됩니다.

- **Forget Gate**: 이전 순간의 셀 상태에서 어떤 정보를 버릴지 결정합니다.
- **입력 게이트**: 현재 입력에서 셀 상태에 저장할 새로운 정보를 결정합니다.
- **Output Gate**: 현재 셀 상태를 기준으로 Hidden State에 어떤 정보를 출력할지 결정합니다.

### 3.1.2 트랜스포머 아키텍처 분석

이전 섹션에서 우리는 RNN과 LSTM이 순환 구조를 도입하여 순차 데이터를 처리하는 것을 보았고, 이는 장거리 종속성을 캡처하는 문제를 어느 정도 해결했습니다. 그러나 이 반복 계산 방법은 데이터를 순차적으로 처리해야 한다는 새로운 병목 현상도 가져왔습니다. 시간 단계 t에서의 계산은 시작하기 전에 시간 단계 t−1이 완료될 때까지 기다려야 합니다. 이는 RNN이 대규모 병렬 계산을 수행할 수 없고 긴 시퀀스를 처리할 때 비효율적이어서 모델 규모 및 훈련 속도 개선이 크게 제한된다는 것을 의미합니다. Transformer는 2017년<sup>[4]</sup>에서 Google 팀에 의해 제안되었습니다. 순환 구조를 완전히 포기하고 대신 **Attention**이라는 메커니즘에 전적으로 의존하여 시퀀스 내의 종속성을 캡처함으로써 진정한 병렬 계산을 달성했습니다.

**(1) 전체 인코더-디코더 구조**

원래 Transformer 모델은 기계 번역의 엔드투엔드 작업을 위해 설계되었습니다. 그림 3.4에서 볼 수 있듯이 매크로 수준에서는 고전적인 **인코더-디코더** 아키텍처를 따릅니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/3-figures/1757249275674-3.png" alt="Figure description" width="50%"/>
<p>그림 3.4 전체 변압기 아키텍처 다이어그램</p>
</div>

우리는 이 구조를 명확한 업무 분담이 있는 팀으로 이해할 수 있습니다.

1. **인코더**: 입력 문장 전체를 "**이해**"하는 작업입니다. 모든 입력 토큰을 읽고(이 개념은 섹션 3.2.2에서 소개됩니다) 궁극적으로 각 토큰에 대한 문맥 정보가 풍부한 벡터 표현을 생성합니다.
2. **디코더**: 목표 문장을 "**생성**"하는 작업입니다. 이미 생성된 이전 텍스트를 참조하고 인코더의 이해 결과를 "참조"하여 다음 단어를 생성합니다.

Transformer의 작동 방식을 진정으로 이해하려면 가장 좋은 방법은 직접 구현하는 것입니다. 이 섹션에서는 "하향식" 접근 방식을 채택합니다. 먼저 필요한 모든 클래스와 메서드를 정의하는 Transformer의 전체 코드 프레임워크를 구축합니다. 그런 다음 퍼즐을 완성하는 것처럼 이러한 클래스의 특정 기능을 하나씩 구현해 보겠습니다.

```Python
import torch
import torch.nn as nn
import math

# --- Placeholder modules, to be implemented in subsequent subsections ---

class PositionalEncoding(nn.Module):
    """
    Positional encoding module
    """
    def forward(self, x):
        pass

class MultiHeadAttention(nn.Module):
    """
    Multi-head attention mechanism module
    """
    def forward(self, query, key, value, mask):
        pass

class PositionWiseFeedForward(nn.Module):
    """
    Position-wise feed-forward network module
    """
    def forward(self, x):
        pass

# --- Encoder core layer ---

class EncoderLayer(nn.Module):
    def __init__(self, d_model, num_heads, d_ff, dropout):
        super(EncoderLayer, self).__init__()
        self.self_attn = MultiHeadAttention() # To be implemented
        self.feed_forward = PositionWiseFeedForward() # To be implemented
        self.norm1 = nn.LayerNorm(d_model)
        self.norm2 = nn.LayerNorm(d_model)
        self.dropout = nn.Dropout(dropout)

    def forward(self, x, mask):
        # Residual connection and layer normalization will be explained in detail in Section 3.1.2.4
        # 1. Multi-head self-attention
        attn_output = self.self_attn(x, x, x, mask)
        x = self.norm1(x + self.dropout(attn_output))

        # 2. Feed-forward network
        ff_output = self.feed_forward(x)
        x = self.norm2(x + self.dropout(ff_output))

        return x

# --- Decoder core layer ---

class DecoderLayer(nn.Module):
    def __init__(self, d_model, num_heads, d_ff, dropout):
        super(DecoderLayer, self).__init__()
        self.self_attn = MultiHeadAttention() # To be implemented
        self.cross_attn = MultiHeadAttention() # To be implemented
        self.feed_forward = PositionWiseFeedForward() # To be implemented
        self.norm1 = nn.LayerNorm(d_model)
        self.norm2 = nn.LayerNorm(d_model)
        self.norm3 = nn.LayerNorm(d_model)
        self.dropout = nn.Dropout(dropout)

    def forward(self, x, encoder_output, src_mask, tgt_mask):
        # 1. Masked multi-head self-attention (on itself)
        attn_output = self.self_attn(x, x, x, tgt_mask)
        x = self.norm1(x + self.dropout(attn_output))

        # 2. Cross-attention (on encoder output)
        cross_attn_output = self.cross_attn(x, encoder_output, encoder_output, src_mask)
        x = self.norm2(x + self.dropout(cross_attn_output))

        # 3. Feed-forward network
        ff_output = self.feed_forward(x)
        x = self.norm3(x + self.dropout(ff_output))

        return x
```

**(2) Self-Attention에서 Multi-Head Attention으로**(2)

이제 뼈대에서 가장 중요한 모듈인 주의 메커니즘을 채워보겠습니다.

"에이전트는 **지능적이기 때문에 학습합니다."라는 문장을 읽고 있다고 상상해 보세요. 우리가 굵은 글씨의 "**it**"을 읽을 때, 그 의미를 이해하기 위해 우리의 뇌는 무의식적으로 문장 앞부분에 있는 "agent"라는 단어에 더 많은 주의를 기울입니다. **Self-Attention** 메커니즘은 이 현상을 수학적으로 모델링한 것입니다. 이를 통해 모델은 각 단어를 처리할 때 문장의 다른 모든 단어를 고려하고 이러한 단어에 서로 다른 "주의 가중치"를 할당할 수 있습니다. 단어의 가중치가 높을수록 현재 단어와의 연관성이 강해지고 현재 단어 표현에서 해당 정보가 차지하는 비율이 커집니다.

위 프로세스를 구현하기 위해 self-attention 메커니즘은 각 입력 토큰 벡터에 대해 학습 가능한 세 가지 역할을 도입합니다.

- **쿼리(Q)**: 정보를 얻기 위해 다른 토큰을 적극적으로 "쿼리"하는 현재 토큰을 나타냅니다.
- **키(K)**: 쿼리할 수 있는 문장 내 토큰의 "레이블" 또는 "인덱스"를 나타냅니다.
- **값(V)**: 토큰 자체가 전달하는 "콘텐츠" 또는 "정보"를 나타냅니다.

이 세 가지 벡터는 모두 원래 단어 임베딩 벡터에 세 가지 학습 가능한 가중치 행렬($W^Q,W^K,W^V$)을 곱하여 얻습니다. 전체 계산 과정은 효율적인 오픈북 시험으로 상상할 수 있는 다음 단계로 나눌 수 있습니다.

- "시험 문제" 및 "자료" 준비: 문장의 각 단어에 대해 가중치 행렬을 통해 $Q,K,V$ 벡터를 생성합니다.
- ​​관련성 점수 계산: 단어 $A$의 새 표현을 계산하려면 단어 $A$의 $Q$ 벡터를 사용하여 문장에 있는 모든 단어($A$ 자체 포함)의 $K$ 벡터와 내적 연산을 수행합니다. 이 점수는 $A$라는 단어를 이해하는 데 있어 다른 단어의 중요성을 반영합니다.
- 안정화 및 정규화: 획득한 모든 점수를 배율 인수 $\sqrt{d_{k}}$($d_{k}$는 $K$ 벡터의 차원)로 나누어 기울기가 너무 작아지는 것을 방지한 다음 Softmax 함수를 사용하여 점수를 합이 1이 되는 가중치로 변환하는 것이 정규화 프로세스입니다.
- 가중치 합: 이전 단계에서 얻은 가중치에 각 단어의 해당 $V$ 벡터를 곱한 다음 모든 결과를 더합니다. 최종 벡터는 전역 문맥 정보를 통합한 후 단어 $A$의 새로운 표현입니다.

이 프로세스는 간결한 공식으로 요약될 수 있습니다.

$$\text{주의}(Q,K,V)=\text{softmax}\left(\frac{QK^{T}}{\sqrt{d_{k}}}\right)V$$

주의 계산이 하나만 수행되는 경우(즉, 단일 헤드) 모델은 한 가지 유형의 연관에만 집중하는 방법만 학습할 수 있습니다. 예를 들어, "it"을 처리할 때 주제에만 집중하는 방법만 학습할 수 있습니다. 그러나 언어의 관계는 복잡하므로 모델이 여러 관계(예: 참조 관계, 긴장 관계, 종속 관계 등)에 동시에 초점을 맞추기를 원합니다. 다중 헤드 주의 메커니즘이 등장했습니다. 아이디어는 간단합니다. 모든 작업을 한 번에 수행하는 대신 여러 그룹으로 나누고 별도로 수행한 다음 병합합니다.

원본 Q, K, V 벡터를 차원을 따라 h 부분으로 분할하고(h는 "헤드" 수) 각 부분은 독립적으로 단일 헤드 주의 계산을 수행합니다. 이는 h명의 서로 다른 "전문가"가 서로 다른 관점에서 문장을 검토하는 것과 같으며, 각 전문가는 서로 다른 특징 관계를 포착합니다. 마지막으로, 이러한 h 전문가의 "의견"(즉, 출력 벡터)을 연결한 다음 선형 변환을 통해 통합하여 최종 출력을 얻습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/3-figures/1757249275674-4.png" alt="Figure description" width="50%"/>
<p>그림 3.5 다중 헤드 주의 메커니즘</p>
</div>

그림 3.5에서 볼 수 있듯이 이 설계를 통해 모델은 다양한 위치와 다양한 표현 하위 공간의 정보에 공동으로 주의를 기울일 수 있어 모델의 표현력이 크게 향상됩니다. 다음은 참고용으로 다중 헤드 어텐션을 간단하게 구현한 것입니다.

```Python
class MultiHeadAttention(nn.Module):
    """
    Multi-head attention mechanism module
    """
    def __init__(self, d_model, num_heads):
        super(MultiHeadAttention, self).__init__()
        assert d_model % num_heads == 0, "d_model must be divisible by num_heads"

        self.d_model = d_model
        self.num_heads = num_heads
        self.d_k = d_model // num_heads

        # Define linear transformation layers for Q, K, V and output
        self.W_q = nn.Linear(d_model, d_model)
        self.W_k = nn.Linear(d_model, d_model)
        self.W_v = nn.Linear(d_model, d_model)
        self.W_o = nn.Linear(d_model, d_model)

    def scaled_dot_product_attention(self, Q, K, V, mask=None):
        # 1. Calculate attention scores (QK^T)
        attn_scores = torch.matmul(Q, K.transpose(-2, -1)) / math.sqrt(self.d_k)

        # 2. Apply mask (if provided)
        if mask is not None:
            # Set positions where mask is 0 to a very small negative number, so they approach 0 after softmax
            attn_scores = attn_scores.masked_fill(mask == 0, -1e9)

        # 3. Calculate attention weights (Softmax)
        attn_probs = torch.softmax(attn_scores, dim=-1)

        # 4. Weighted sum (weights * V)
        output = torch.matmul(attn_probs, V)
        return output

    def split_heads(self, x):
        # Transform input x shape from (batch_size, seq_length, d_model)
        # to (batch_size, num_heads, seq_length, d_k)
        batch_size, seq_length, d_model = x.size()
        return x.view(batch_size, seq_length, self.num_heads, self.d_k).transpose(1, 2)

    def combine_heads(self, x):
        # Transform input x shape from (batch_size, num_heads, seq_length, d_k)
        # back to (batch_size, seq_length, d_model)
        batch_size, num_heads, seq_length, d_k = x.size()
        return x.transpose(1, 2).contiguous().view(batch_size, seq_length, self.d_model)

    def forward(self, Q, K, V, mask=None):
        # 1. Perform linear transformations on Q, K, V
        Q = self.split_heads(self.W_q(Q))
        K = self.split_heads(self.W_k(K))
        V = self.split_heads(self.W_v(V))

        # 2. Calculate scaled dot-product attention
        attn_output = self.scaled_dot_product_attention(Q, K, V, mask)

        # 3. Combine multi-head outputs and perform final linear transformation
        output = self.W_o(self.combine_heads(attn_output))
        return output
```

**(3) 피드포워드 신경망**

각 인코더 및 디코더 레이어에서 다중 헤드 주의 하위 레이어 뒤에는 **위치별 피드 포워드 네트워크(FFN)**가 옵니다. Attention 레이어의 역할이 전체 시퀀스에서 관련 정보를 "동적으로 집계"하는 것이라면 피드포워드 네트워크의 역할은 이 집계된 정보에서 고차 기능을 추출하는 것입니다.

이 이름의 핵심은 '위치별'입니다. 이는 이 피드포워드 네트워크가 시퀀스의 각 토큰 벡터에 대해 독립적으로 작동한다는 것을 의미합니다. 즉, 길이가 `seq_len`인 시퀀스의 경우 이 FFN은 실제로 `seq_len`배라고 불리며 매번 하나의 토큰을 처리합니다. 중요한 것은 모든 포지션이 동일한 네트워크 가중치 세트를 공유한다는 것입니다. 이 설계는 각 위치를 독립적으로 처리하는 기능을 유지하고 모델의 매개변수 수를 크게 줄입니다. 이 네트워크의 구조는 매우 간단하며 두 개의 선형 변환과 ReLU 활성화 함수로 구성됩니다.

$$\mathrm{FFN}(x)=\max\left(0, xW_{1}+b_{1}\right) W_{2}+b_{2}$$

여기서 $x$는 attention 하위 계층의 출력입니다. $W_1,b_1,W_2,b_2$는 학습 가능한 매개변수입니다. 일반적으로 첫 번째 선형 레이어의 출력 차원 `d_ff`는 입력 차원 `d_model`(예: `d_ff = 4 * d_model`)보다 훨씬 크며, ReLU 활성화 후 두 번째 선형 레이어를 통해 `d_model` 차원으로 다시 매핑됩니다. 이 "확장 후 축소" 설계는 모델이 더 풍부한 특징 표현을 학습하는 데 도움이 되는 것으로 여겨집니다.

PyTorch 뼈대에서 다음 코드를 사용하여 이 모듈을 구현할 수 있습니다.

```Python
class PositionWiseFeedForward(nn.Module):
    """
    Position-wise feed-forward network module
    """
    def __init__(self, d_model, d_ff, dropout=0.1):
        super(PositionWiseFeedForward, self).__init__()
        self.linear1 = nn.Linear(d_model, d_ff)
        self.dropout = nn.Dropout(dropout)
        self.linear2 = nn.Linear(d_ff, d_model)
        self.relu = nn.ReLU()

    def forward(self, x):
        # x shape: (batch_size, seq_len, d_model)
        x = self.linear1(x)
        x = self.relu(x)
        x = self.dropout(x)
        x = self.linear2(x)
        # Final output shape: (batch_size, seq_len, d_model)
        return x
```

**(4) 잔여 연결 및 레이어 정규화**

Transformer의 각 인코더 및 디코더 레이어에서 모든 하위 모듈(예: 다중 헤드 주의 및 피드포워드 네트워크)은 `Add & Norm` 작업으로 래핑됩니다. 이 조합을 통해 Transformer는 안정적으로 학습할 수 있습니다.

이 작업은 두 부분으로 구성됩니다.

- ​​**잔여 연결(추가)**: 이 작업은 하위 모듈의 입력 `x`를 하위 모듈의 출력 `Sublayer(x)`에 직접 추가합니다. 이 구조는 심층 신경망의 **Vanishing Gradients** 문제를 해결합니다. 역전파 중에 경사도는 서브모듈을 우회하고 앞으로 직접 전파할 ​​수 있으므로 네트워크에 레이어가 많더라도 모델을 효과적으로 훈련할 수 있습니다. 그 공식은 다음과 같이 표현될 수 있습니다: $\text{Output} = x + \text{Sublayer}(x)$.
- **레이어 정규화(표준)**: 이 작업은 단일 샘플의 모든 기능을 정규화하여 평균을 0으로 만들고 분산을 1로 만듭니다. 이는 모델 학습 중 **내부 공변량 이동** 문제를 해결하고 각 레이어의 입력 분포를 안정적으로 유지하여 모델 수렴을 가속화하고 학습 안정성을 향상시킵니다.

**3.1.2.5 위치 인코딩**

우리는 Transformer의 핵심이 시퀀스에 있는 두 토큰 간의 관계를 계산하여 종속성을 캡처하는 self-attention 메커니즘이라는 것을 이미 이해하고 있습니다. 그러나 이 계산 방법에는 본질적인 문제가 있습니다. 즉, 토큰 순서나 위치에 대한 정보가 포함되어 있지 않습니다. self-attention의 경우 "에이전트 학습"과 "에이전트 학습"이라는 두 시퀀스는 토큰 간의 관계에만 관심을 갖고 배열을 무시하기 때문에 완전히 동일합니다. 이 문제를 해결하기 위해 Transformer에서는 **위치 인코딩**을 도입했습니다.

위치 인코딩의 핵심 아이디어는 절대 및 상대 위치 정보를 나타내는 추가 "위치 벡터"를 입력 시퀀스의 각 토큰 임베딩 벡터에 추가하는 것입니다. 이 위치 벡터는 학습되지 않고 고정된 수학 공식을 통해 직접 계산됩니다. 이렇게 하면 두 개의 토큰(예: 둘 다 `agent`라고 하는 두 개의 토큰)이 동일한 임베딩을 갖고 있더라도 문장에서 서로 다른 위치에 있기 때문에 궁극적으로 Transformer 모델에 입력하는 벡터는 서로 다른 위치 인코딩을 추가하기 때문에 고유하게 됩니다. 원본 논문에서 제안된 위치 인코딩은 사인 및 코사인 함수를 사용하여 생성하며 수식은 다음과 같습니다.

$$PE_{(pos,2i)}=\sin\left(\frac{pos}{10000^{2i/d_{\text{모델}}}}\right)，$$

$$PE_{(pos,2i+1)}=\cos\left(\frac{pos}{10000^{2i/d_{\text{모델}}}}\right)$$

여기서:

- $pos$는 시퀀스에서 토큰의 위치입니다(예: $0$, $1$, $2$, ...).
- $i$는 위치 벡터의 치수 인덱스입니다($0$부터 $d_{\text{model}}/2$까지).
- $d_{\text{model}}$는 단어 임베딩 벡터의 차원입니다(모델에서 정의한 것과 일치).

이제 `PositionalEncoding` 모듈을 구현하고 Transformer 뼈대 코드의 마지막 부분을 완성해 보겠습니다.

```Python
class PositionalEncoding(nn.Module):
    """
    Add positional encoding to word embedding vectors of input sequence.
    """
    def __init__(self, d_model: int, dropout: float = 0.1, max_len: int = 5000):
        super().__init__()
        self.dropout = nn.Dropout(p=dropout)

        # Create a sufficiently long positional encoding matrix
        position = torch.arange(max_len).unsqueeze(1)
        div_term = torch.exp(torch.arange(0, d_model, 2) * (-math.log(10000.0) / d_model))

        # pe (positional encoding) size is (max_len, d_model)
        pe = torch.zeros(max_len, d_model)

        # Even dimensions use sin, odd dimensions use cos
        pe[:, 0::2] = torch.sin(position * div_term)
        pe[:, 1::2] = torch.cos(position * div_term)

        # Register pe as buffer, so it won't be treated as model parameter but will move with the model (e.g., to(device))
        self.register_buffer('pe', pe.unsqueeze(0))

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        # x.size(1) is the current input sequence length
        # Add positional encoding to input vector
        x = x + self.pe[:, :x.size(1)]
        return self.dropout(x)
```

이 하위 섹션은 주로 Transformer의 매크로 구조와 각 내부 모듈의 작동 세부 사항을 이해하는 데 도움이 됩니다. 에이전트 학습에서 대형 모델의 지식 시스템을 보완하기 위한 것이므로 더 이상 구현하지 않을 것입니다. 이 시점에서 우리는 최신 대규모 언어 모델을 이해하기 위한 견고한 아키텍처 기반을 마련했습니다. 다음 섹션에서는 디코더 전용 아키텍처를 살펴보고 이것이 Transformer의 아이디어를 기반으로 어떻게 발전했는지 살펴보겠습니다.

### 3.1.3 디코더 전용 아키텍처

이전 섹션에서는 다양한 엔드투엔드 시나리오에서 탁월한 성능을 발휘하는 완전한 Transformer 모델을 직접 구축했습니다. 그러나 작업이 사람과 대화하고, 생성하고, 에이전트의 두뇌 역할을 할 수 있는 일반 모델을 구축하는 것으로 옮겨지면 아마도 그러한 복잡한 구조는 필요하지 않을 것입니다.

Transformer의 디자인 철학은 '먼저 이해한 후 생성'입니다. 인코더는 전체 입력 문장을 깊이 이해하여 전역 정보를 포함하는 상황별 메모리를 형성하고, 디코더는 이 메모리를 기반으로 번역을 생성합니다. 하지만 OpenAI가 **GPT(Generative Pre-trained Transformer)**를 개발하면서 더 간단한 아이디어를 제안했습니다.<sup>[5]</sup>: **다음으로 가능성이 가장 높은 단어를 예측하는 것이 언어의 핵심 작업이 아닌가요?**

질문에 답하든, 스토리를 작성하든, 코드를 생성하든 기본적으로 기존 텍스트 시퀀스 뒤에 가장 합리적인 콘텐츠를 단어별로 추가하는 것입니다. 이 아이디어를 바탕으로 GPT는 대담한 단순화를 시도했습니다. **인코더를 완전히 버리고 디코더 부분만 유지했습니다.** 이것이 **Decoder-Only** 아키텍처의 기원입니다.

디코더 전용 아키텍처의 작업 모드를 **Autoregressive**라고 합니다. 전문적으로 들리는 이 용어는 실제로 매우 간단한 프로세스를 설명합니다.

1. 모델에 시작 텍스트를 제공합니다(예: "Datawhale Agent is").
2. 모델은 다음으로 가능성이 가장 높은 단어(예: "a")를 예측합니다.
3. 모델은 방금 생성한 단어 "a"를 입력 텍스트 끝에 추가하여 새 입력("Datawhale Agent is a")을 형성합니다.
4. 이 새로운 입력을 기반으로 모델은 다음 단어(예: "powerful")를 다시 예측합니다.
5. 완전한 문장이 생성되거나 중지 조건에 도달할 때까지 이 과정을 계속 반복합니다.

모델은 이미 작성된 내용을 지속적으로 "검토"한 후 다음 단어가 무엇인지 생각하는 "단어 체인" 게임을 하는 것과 같습니다.

다음과 같이 질문할 수 있습니다. 훈련 중에 모델에 완전한 텍스트 시퀀스가 ​​한 번에 제공되는 경우가 많은데, 다음 토큰을 예측하는 방법을 학습할 때 나중에 답변을 "살펴보기"하지 않도록 어떻게 보장합니까?

정답은 **가면을 쓴 자기 주의**입니다. 디코더 전용 아키텍처에서는 이 메커니즘이 매우 중요합니다. 작동 원리는 매우 영리합니다.

훈련 중에 전체 텍스트 시퀀스가 ​​모델에 병렬로 공급될 수 있지만 self-attention 메커니즘이 attention 점수 행렬(즉, 다른 모든 단어에 대한 각 단어의 attention 점수)을 계산한 후 Softmax 정규화 전에 모델은 인과 마스크를 적용합니다. 이 마스크는 현재 위치 이후의 모든 토큰에 해당하는 점수를 매우 큰 음수로 대체합니다. 이 행렬이 Softmax를 거치면 해당 위치의 확률은 0이 됩니다. 이렇게 하면 모델이 어떤 위치에서든 표현을 계산할 때 수학적으로 해당 위치 이후의 정보에 주의하는 것을 방지할 수 있습니다.

생성 중에 상황은 훨씬 더 직접적입니다. 향후 토큰이 아직 생성되지 않았으므로 모델은 이미 생성된 콘텐츠만 컨텍스트로 사용하고 다음 토큰을 단계별로 예측할 수 있습니다. Masked self-attention은 훈련 목표를 자동 회귀 생성과 일관되게 유지하여 모델이 항상 현재 위치 이전의 정보에만 의존하도록 보장합니다.

**디코더 전용 아키텍처의 장점**

단순해 보이는 이 아키텍처는 다음과 같은 이점을 통해 엄청난 성공을 가져왔습니다.

- **통합 학습 목표**: 모델의 유일한 작업은 "다음 단어를 예측"하는 것입니다. 이는 레이블이 지정되지 않은 대규모 텍스트 데이터에 대한 사전 학습에 매우 적합한 간단한 목표입니다.
- **간단한 구조, 손쉬운 확장**: 구성 요소가 적다는 것은 확장이 더 쉽다는 것을 의미합니다. 오늘날의 GPT-4, Llama 및 수천억, 심지어는 수조 개의 매개변수를 가진 기타 거대 모델은 모두 이 간결한 아키텍처를 기반으로 합니다.
- **생성 작업에 자연스럽게 적합**: 자동 회귀 작업 모드는 모든 생성 작업(대화, 쓰기, 코드 생성 등)과 완벽하게 일치하며, 이는 일반 에이전트 구축의 기반이 될 수 있는 핵심 이유이기도 합니다.

요약하자면, Transformer의 디코더에서 발전한 Decoder-Only 아키텍처는 "다음 단어 예측"이라는 단순한 패러다임을 통해 오늘날 우리가 살고 있는 대규모 언어 모델의 시대를 열었습니다.

## 3.2 대규모 언어 모델과 상호작용

### 3.2.1 신속한 엔지니어링

대규모 언어 모델을 매우 유능한 "뇌"에 비유한다면 **Prompt**는 이 "뇌"와 통신하는 데 사용하는 언어입니다. 프롬프트 엔지니어링은 우리가 기대하는 응답을 생성하도록 모델을 안내하는 정확한 프롬프트를 설계하는 방법에 대한 연구입니다. 에이전트 구축의 경우 신중하게 설계된 프롬프트를 사용하면 에이전트 간의 협업과 작업 분할을 효율적으로 수행할 수 있습니다.

**(1) 모델 샘플링 매개변수**

대형 모델을 사용하다 보면 `Temperature`와 같은 구성 가능한 매개변수를 자주 보게 됩니다. 그들의 본질은 특정 시나리오 요구 사항에 맞게 "확률 분포"에 대한 모델의 샘플링 전략을 조정하는 것입니다. 적절한 매개변수를 구성하면 특정 시나리오에서 에이전트 성능을 향상시킬 수 있습니다.

전통적인 확률 분포는 Softmax 공식 $p_i = \frac{e^{z_i}}{\sum_{j=1}^k e^{z_j}}$로 계산됩니다. 샘플링 매개변수의 본질은 다양한 전략을 기반으로 분포를 "재조정"하거나 "절단"하여 대규모 모델에 의한 다음 토큰 출력을 변경하는 것입니다.

`Temperature`: 온도는 모델 출력의 "임의성"과 "결정성"을 제어하는 ​​핵심 매개변수입니다. 그 원리는 온도 계수 $T\gt0$를 도입하여 Softmax를 $p_i^{(T)} = \frac{e^{z_i / T}}{\sum_{j=1}^k e^{z_j / T}}$로 다시 작성하는 것입니다.

T가 감소하면 분포가 "가파르게" 되고 확률이 높은 항목 가중치가 더욱 증폭되어 반복률이 더 높은 "보수적인" 텍스트가 생성됩니다. T가 증가하면 분포가 "더 평평해지고" 확률이 낮은 항목 가중치가 증가하여 더 "다양하지만" 일관되지 않은 콘텐츠가 생성됩니다.

- 저온(0 $\leqslant$ 온도 $\lt$ 0.3): 출력이 더 "정확하고 결정적"입니다. 적용 가능한 시나리오: 실제 작업: Q&A, 데이터 계산, 코드 생성 등; 엄격한 시나리오: 법률 텍스트 해석, 기술 문서 작성, 학문적 개념 설명 등

- 중간 온도(0.3 $\leqslant$ 온도 $\lt$ 0.7): 출력이 "균형 있고 자연스럽습니다." 적용 가능한 시나리오: 일상 대화: 고객 서비스 상호 작용, 챗봇 등; 정기작성 : 이메일 작성, 제품 카피, 간단한 스토리 작성 등.

- 고온(0.7 $\leqslant$ 온도 $\lt$ 2): 출력이 "혁신적이고 다양합니다." 적용 가능한 시나리오: 창의적인 작업: 시 창작, 공상 과학 이야기 구상, 광고 슬로건 브레인스토밍, 예술적 영감과 같은; 다양한 사고.

`Top-k`: 그 원리는 모든 토큰을 높은 확률에서 낮은 확률로 정렬하고, 상위 k 토큰을 가져와 "후보 세트"를 구성한 다음 필터링된 k 토큰의 확률을 "정규화"하는 것입니다. $ \hat{p}_i = \frac{p_i}{\sum_{j \in \text{후보 세트}} p_j}$

- 온도 샘플링과의 차이점 및 연결: 온도 샘플링은 후보 토큰의 수를 변경하지 않고(여전히 모든 N을 고려함) 온도 T를 통해 모든 토큰(부드럽거나 가파른)의 확률 분포를 조정합니다. Top-k 샘플링은 k 값을 통해 후보 토큰의 수(확률이 높은 상위 k개 토큰만 유지)를 제한한 다음 그로부터 샘플링합니다. k=1이면 출력은 완전히 결정적이며 "탐욕스러운 샘플링"으로 변질됩니다.

`Top-p`: 그 원리는 모든 토큰을 확률에 따라 높은 것부터 낮은 것까지 정렬하는 것입니다. 정렬 후 첫 번째 토큰부터 시작하여 누적 합계가 임계값 p: $\sum_{i \in S} p_{(i)} \geq p$에 도달하거나 초과할 때까지 점진적으로 확률을 누적합니다. 이 시점에서 축적과정에 포함된 모든 토큰은 '핵집합'을 형성하고 최종적으로 핵집합이 정규화된다.

- Top-k와의 차이점 및 연결: 잘림 크기가 고정된 Top-k와 비교하여 Top-p는 다양한 분포의 "롱테일" 특성에 동적으로 적응할 수 있으며, 고르지 않은 확률 분포의 극단적인 경우에 대한 더 나은 적응성을 제공합니다.

텍스트 생성에서 Top-p, Top-k 및 온도 계수가 동시에 설정되면 이러한 매개변수는 온도 조정 → Top-k → Top-p 우선 순위에 따라 계층화된 필터링 방식으로 함께 작동합니다. 온도는 분포의 전반적인 경사도를 조정하고, Top-k는 먼저 가장 높은 확률을 가진 k개의 후보를 유지한 다음 Top-p는 Top-k 결과에서 누적 확률 ≥ p인 최소 집합을 최종 후보 집합으로 선택합니다. 그러나 일반적으로 Top-k 또는 Top-p 중 하나를 선택하면 충분합니다. 둘 다 설정된 경우 실제 후보 세트는 둘의 교차점입니다.
온도가 0으로 설정되면 가장 가능성이 높은 토큰이 다음 예측 토큰이 되기 때문에 Top-k와 Top-p는 관련이 없게 됩니다. Top-k가 1로 설정되면 단 하나의 토큰만이 Top-k 기준을 통과하고 다음 예측 토큰이 되기 때문에 온도와 Top-p도 관련이 없게 됩니다.

**(2) 제로샷, 원샷 및 퓨샷 프롬프트**

모델에 제공하는 예제(Exemplars)의 수에 따라 프롬프트는 세 가지 유형으로 나눌 수 있습니다. 이를 더 잘 이해하기 위해 모델이 텍스트의 감정적 어조(예: 긍정적, 부정적 또는 중립)를 판단하도록 하는 것을 목표로 감정 분류 작업을 예로 들어 보겠습니다.

**제로샷 프롬프트** 이는 모델에 어떠한 예시도 제공하지 않고 지침에 따라 작업을 완료하도록 직접 요청한다는 의미입니다. 이는 대규모 데이터에 대한 사전 학습 후 획득한 모델의 강력한 일반화 능력의 이점을 활용합니다.

사례: 모델에 직접 지침을 제공하여 감정 분류 작업을 완료하도록 요구합니다.

```Python
Text: Datawhale's AI Agent course is excellent!
Sentiment: Positive
```

**원샷 프롬프트** 작업 형식과 예상되는 출력 스타일을 보여주는 하나의 완전한 예를 모델에 제공합니다.

사례: 먼저 모델에 완전한 "질문-답변" 쌍을 시연으로 제공한 다음 새로운 질문을 제기합니다.

```Python
Text: This restaurant's service is too slow.
Sentiment: Negative

Text: Datawhale's AI Agent course is excellent!
Sentiment:
```

모델은 주어진 예제 형식을 모방하고 두 번째 텍스트에 대해 "Positive"를 완성합니다.

**퓨샷 프롬프트** 모델이 작업의 세부 사항, 경계 및 뉘앙스를 더 정확하게 이해할 수 있도록 하는 여러 가지 예를 제공하여 더 나은 성능을 달성합니다.

사례: 다양한 상황을 다루는 여러 예를 제공하여 모델이 작업을 보다 포괄적으로 이해할 수 있도록 합니다.

```Python
Text: This restaurant's service is too slow.
Sentiment: Negative

Text: This movie's plot is very bland.
Sentiment: Neutral

Text: Datawhale's AI Agent course is excellent!
Sentiment:
```

모델은 모든 예를 종합하여 마지막 문장의 감정을 "긍정적"으로 보다 정확하게 분류합니다.

**(3) 명령어 튜닝의 영향**

초기 GPT 모델(예: GPT-3)은 주로 "텍스트 완성" 모델이었습니다. 그들은 이전 텍스트를 기반으로 텍스트를 계속하는 데 능숙했지만 인간의 지시를 이해하고 실행하는 데 반드시 능숙하지는 않았습니다.

**명령 조정**은 사전 학습된 모델을 추가로 교육하기 위해 대량의 "명령-응답" 형식 데이터를 사용하는 미세 조정 기술입니다. 명령어 조정 후 모델은 사용자 명령어를 더 잘 이해하고 따를 수 있습니다. 오늘날 우리가 일상 업무와 학습에 사용하는 모든 모델(예: `ChatGPT`, `DeepSeek`, `Qwen`)은 해당 모델 계열의 교육 조정 모델입니다.

- **"텍스트 완성" 모델에 대한 프롬프트(모델에 수행할 작업을 "교육"하려면 몇 번의 프롬프트를 사용해야 함):**

```Plain
This is a program that translates English to Chinese.
English: Hello
Chinese: 你好
English: How are you?
Chinese:
```

- **"명령 조정" 모델에 대한 프롬프트(직접 지침을 제공할 수 있음):**

```Plain
Please translate the following English to Chinese:
How are you?
```

명령 조정의 출현으로 모델과 상호 작용하는 방식이 크게 단순화되어 직접적이고 명확한 자연어 명령이 가능해졌습니다.

**(4) 기본 프롬프트 기법**

**역할극** 모델에 특정 역할을 할당함으로써 응답 스타일, 어조 및 지식 범위를 안내하여 특정 시나리오 요구 사항에 더 적합한 출력을 만들 수 있습니다.

```Plain
# Case
You are now a senior Python programming expert. Please explain what GIL (Global Interpreter Lock) is in Python in a way that even a beginner can understand.
```

**상황 내 예** 이는 퓨샷 프롬프트 아이디어와 일치합니다. 프롬프트에 명확한 입력-출력 예제를 제공함으로써 모델에 요청 처리 방법을 "교육"합니다. 이는 복잡한 형식이나 특정 스타일 작업을 처리할 때 특히 효과적입니다.

```Plain
# Case
I need you to extract product names and user sentiment from product reviews. Please output strictly in the JSON format below.

Review: The screen display of this "Stardust" laptop is amazing, but I don't really like its keyboard feel.
Output: {"product_name": "Stardust Laptop", "sentiment": "Mixed"}

Review: The "SoundMove" headphones I just bought have great sound quality, and the battery life exceeded my expectations!
Output:
```

**(5) 생각의 연쇄**

논리적 추론이나 계산, 다단계 사고가 필요한 복잡한 문제의 경우 모델에게 직접 답변을 요구하면 오류가 발생하는 경우가 많습니다. **CoT(사고 사슬)**은 "단계별로 생각"하도록 안내하여 복잡한 작업에 대한 모델의 추론 능력을 향상시키는 강력한 프롬프트 기술입니다.

CoT 구현의 핵심은 프롬프트에 "단계적으로 생각해 보세요" 또는 "단계적으로 생각해보자"와 같은 간단한 안내 문구를 추가하는 것입니다.

```Plain
# Chain-of-Thought Prompt
A basketball team won 60% of their 80 games in one season. In the next season, they played 15 games and won 12. What is the total winning percentage for both seasons?
Please think step by step and solve.

>>>
(The model might output)
Okay, let's calculate step by step.
Step 1: Calculate the number of games won in the first season.
80 games * 60% = 48 games.
Step 2: Calculate the total number of games and total wins for both seasons.
Total games = 80 + 15 = 95 games.
Total wins = 48 + 12 = 60 games.
Step 3: Calculate the total winning percentage.
Total winning percentage = (Total wins / Total games) * 100% = (60 / 95) * 100% ≈ 63.16%.
So, the total winning percentage for both seasons is approximately 63.16%.
```

추론 과정을 명시적으로 표시함으로써 모델은 더 쉽게 정답에 도달할 뿐만 아니라 응답을 더 신뢰할 수 있고 확인하고 수정하기가 더 쉬워집니다.

### 3.2.2 텍스트 토큰화

우리는 컴퓨터가 기본적으로 숫자만 이해할 수 있다는 것을 알고 있습니다. 따라서 대규모 언어 모델에 자연어 텍스트를 제공하기 전에 먼저 모델이 처리할 수 있는 숫자 형식으로 변환해야 합니다. 텍스트 시퀀스를 숫자 시퀀스로 변환하는 프로세스를 **토큰화**라고 합니다. **토큰나이저**의 역할은 원시 텍스트를 **토큰**이라고 하는 최소 단위로 분할하는 규칙 세트를 정의하는 것입니다.

**3.2.2.1 토큰화가 필요한 이유**

초기 자연어 처리 작업은 간단한 토큰화 전략을 채택할 수 있습니다.

- **단어 기반**: 공백이나 구두점을 사용하여 문장을 단어로 직접 분할합니다. 이 방법은 직관적이지만 다음과 같은 중요한 과제에 직면해 있습니다.
- **어휘 폭발과 OOV**: 언어의 어휘는 방대합니다. 각 단어가 독립적인 토큰으로 처리되면 어휘 관리가 어려워집니다. 더 나쁜 것은 모델이 어휘에 나타나지 않는 단어(예: "DatawhaleAgent")를 처리할 수 없다는 것입니다. 이 현상을 "OOV(어휘 부족)" 문제라고 합니다.
- **의미 연관성 부족**: 모델은 형태학적으로 유사한 단어 간의 의미 관계를 포착하는 데 어려움을 겪습니다. 예를 들어, "look", "looks" 및 " looking"은 공통 핵심 의미를 공유함에도 불구하고 완전히 다른 세 가지 토큰으로 취급됩니다. 마찬가지로 훈련 데이터에서 빈도가 낮은 단어의 의미는 발생 빈도가 낮기 때문에 완전히 학습할 수 없습니다.

- **문자 기반**: 텍스트를 개별 문자로 분할합니다. 이 방법은 매우 작은 어휘(예: 영문자, 숫자, 구두점)를 사용하므로 OOV 문제를 피할 수 있습니다. 그러나 단점은 개별 문자가 대부분 독립적인 의미적 의미가 부족하다는 것입니다. 모델은 문자를 의미 있는 단어로 결합하는 방법을 학습하는 데 더 많은 노력을 기울여야 하므로 학습이 비효율적입니다.

어휘 크기와 의미 표현의 균형을 맞추기 위해 최신 대규모 언어 모델은 **하위 단어 토큰화** 알고리즘을 널리 채택합니다. 핵심 아이디어는 일반적인 단어(예: "에이전트")를 하나의 완전한 토큰으로 유지하면서 흔하지 않은 단어(예: "토큰화")를 의미 있는 하위 단어 조각(예: "토큰" 및 "화")으로 분해하는 것입니다. 이 접근 방식은 어휘의 크기를 제어할 뿐만 아니라 모델이 하위 단어를 결합하여 새로운 단어를 이해하고 생성할 수 있도록 합니다.

**3.2.2.2 바이트 쌍 인코딩 알고리즘 분석**

BPE(바이트 쌍 인코딩)는 GPT 시리즈 모델에 채택된 가장 주류인 하위 단어 토큰화 알고리즘 <sup>[6]</sup> 중 하나입니다. 핵심 아이디어는 매우 간결하며 "탐욕스러운" 병합 프로세스로 이해될 수 있습니다.

1. **초기화**: 말뭉치에 등장하는 모든 기본 문자로 어휘를 초기화합니다.
2. **반복적 병합**: 코퍼스에서 인접한 모든 토큰 쌍의 빈도를 계산하고 빈도가 가장 높은 쌍을 찾아서 새 토큰으로 병합하고 어휘에 추가합니다.
3. **반복**: 어휘 크기가 미리 설정된 임계값에 도달할 때까지 2단계를 반복합니다.

**사례 시연:** 미니 코퍼스가 `{"hug": 1, "pug": 1, "pun": 1, "bun": 1}`이고 크기 10의 어휘를 구축한다고 가정합니다. BPE 훈련 프로세스는 표 3.1로 나타낼 수 있습니다.

<div align="center">
<p>Table 3.1 BPE 알고리즘 병합 프로세스의 예</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/3-figures/1757249275674-5.png" alt="Figure description" width="90%"/>
</div>

훈련이 끝난 후 어휘 크기가 10에 도달하면 새로운 토큰화 규칙을 얻습니다. 이제 보이지 않는 단어 "bug"에 대해 토크나이저는 먼저 "bug"가 어휘에 있는지 확인하고 그렇지 않은 것을 찾습니다. 그런 다음 "bu"를 확인하고 그렇지 않은 것을 찾으십시오. 마지막으로 "b"와 "ug"를 확인하고 둘 다 포함되어 있는지 찾아 `['b', 'ug']`로 분할합니다.

아래에서는 간단한 Python 코드를 사용하여 위 프로세스를 시뮬레이션합니다.

```Python
import re, collections

def get_stats(vocab):
    """Count token pair frequencies"""
    pairs = collections.defaultdict(int)
    for word, freq in vocab.items():
        symbols = word.split()
        for i in range(len(symbols)-1):
            pairs[symbols[i],symbols[i+1]] += freq
    return pairs

def merge_vocab(pair, v_in):
    """Merge token pairs"""
    v_out = {}
    bigram = re.escape(' '.join(pair))
    p = re.compile(r'(?<!\S)' + bigram + r'(?!\S)')
    for word in v_in:
        w_out = p.sub(''.join(pair), word)
        v_out[w_out] = v_in[word]
    return v_out

# Prepare corpus, add </w> at the end of each word to indicate ending, and split characters
vocab = {'h u g </w>': 1, 'p u g </w>': 1, 'p u n </w>': 1, 'b u n </w>': 1}
num_merges = 4 # Set number of merges

for i in range(num_merges):
    pairs = get_stats(vocab)
    if not pairs:
        break
    best = max(pairs, key=pairs.get)
    vocab = merge_vocab(best, vocab)
    print(f"Merge {i+1}: {best} -> {''.join(best)}")
    print(f"New vocabulary (partial): {list(vocab.keys())}")
    print("-" * 20)

>>>
Merge 1: ('u', 'g') -> ug
New vocabulary (partial): ['h ug </w>', 'p ug </w>', 'p u n </w>', 'b u n </w>']
--------------------
Merge 2: ('ug', '</w>') -> ug</w>
New vocabulary (partial): ['h ug</w>', 'p ug</w>', 'p u n </w>', 'b u n </w>']
--------------------
Merge 3: ('u', 'n') -> un
New vocabulary (partial): ['h ug</w>', 'p ug</w>', 'p un </w>', 'b un </w>']
--------------------
Merge 4: ('un', '</w>') -> un</w>
New vocabulary (partial): ['h ug</w>', 'p ug</w>', 'p un</w>', 'b un</w>']
--------------------
```

이 코드는 BPE 알고리즘이 가장 높은 빈도의 인접 토큰 쌍을 반복적으로 병합하여 어휘를 점차적으로 구축하고 확장하는 방법을 명확하게 보여줍니다.

많은 후속 알고리즘은 BPE를 기반으로 한 최적화입니다. 그 중 구글이 개발한 워드피스(WordPiece)와 센텐스피스(SentencePiece)가 가장 영향력이 크다.

- **WordPiece**: Google의 BERT 모델<sup>[7]</sup>에서 채택한 알고리즘입니다. BPE와 매우 유사하지만 토큰 병합 기준은 '최고 빈도'가 아니라 '코퍼스의 언어 모델 확률 향상을 극대화'하는 것입니다. 간단히 말해서, 전체 코퍼스의 "유창성" 향상을 극대화할 수 있는 토큰 쌍 병합을 우선시합니다.
- ​​**SentencePiece**: Llama 시리즈 모델에 채택된 Google<sup>[8]</sup>의 오픈 소스 토큰화 도구입니다. 가장 큰 특징은 공백을 일반 문자(보통 밑줄 `_`로 표시)로 처리한다는 것입니다. 이를 통해 토큰화 및 디코딩 프로세스를 완전히 되돌릴 수 있고 특정 언어와 독립적으로 만들 수 있습니다(예를 들어 중국어는 단어 분할에 공백을 사용하지 않는다는 사실을 알 필요가 없습니다).

**3.2.2.3 개발자를 위한 토크나이저의 중요성**

토큰화 알고리즘의 세부 사항을 이해하는 것이 목표는 아니지만 에이전트 개발자로서 토크나이저의 실제 영향을 이해하는 것은 에이전트 성능, 비용 및 안정성과 직접적인 관련이 있기 때문에 중요합니다.

- **컨텍스트 창 제한**: 모델의 컨텍스트 창(예: 8K, 128K)은 문자 수나 단어 수가 아닌 **토큰 수**로 계산됩니다. 동일한 텍스트라도 언어(예: 중국어, 영어)나 토크나이저에 따라 토큰 수가 크게 다를 수 있습니다. 입력 길이를 정확하게 관리하고 컨텍스트 제한을 초과하지 않는 것이 장기 기억 에이전트 구축의 기초입니다.
- **API 비용**: 대부분의 모델 API는 토큰 수에 따라 요금을 청구합니다. 텍스트가 어떻게 토큰화되는지 이해하는 것은 상담원 운영 비용을 추정하고 제어하는 ​​핵심 단계입니다.
- **모델 성능 이상**: 때때로 토큰화로 인해 이상한 모델 동작이 발생합니다. 예를 들어, 모델은 `2 + 2`를 계산하는 데는 능숙할 수 있지만 `2+2`(공백 없이)에서는 실수를 범할 수 있습니다. 왜냐하면 `2+2`가 토크나이저에 의해 독립적이고 흔하지 않은 토큰으로 처리될 수 있기 때문입니다. 마찬가지로, 첫 글자의 대문자가 다른 단어는 완전히 다른 토큰 시퀀스로 분할되어 모델의 이해에 영향을 미칠 수 있습니다. 프롬프트를 디자인하고 모델 출력을 구문 분석할 때 이러한 "트랩"을 고려하면 에이전트 견고성을 향상시키는 데 도움이 됩니다.

### 3.2.3 오픈 소스 대형 언어 모델 호출

이 책의 1장에서는 에이전트를 구동하기 위해 API를 통해 대규모 언어 모델과 상호작용했습니다. 이것은 빠르고 편리한 방법이지만 유일한 방법은 아닙니다. 민감한 데이터 처리, 오프라인 작업 또는 정밀한 비용 제어가 필요한 많은 시나리오의 경우 대규모 언어 모델을 로컬로 직접 배포하는 것이 중요합니다.

**Hugging Face Transformers**는 사전 훈련된 수만 개의 모델을 로드하고 사용할 수 있는 표준화된 인터페이스를 제공하는 강력한 오픈 소스 라이브러리입니다. 이 연습을 완료하는 데 이를 사용하겠습니다.

**환경 구성 및 모델 선택**: 대부분의 독자가 개인용 컴퓨터에서 원활하게 작동할 수 있도록 의도적으로 작지만 강력한 모델인 `Qwen/Qwen1.5-0.5B-Chat`를 선택했습니다. Alibaba DAMO Academy에서 오픈소스로 제공하는 약 5억 개의 매개변수를 갖춘 대화 모델입니다. 크기가 작고 성능이 뛰어나 입문 학습 및 로컬 배포에 매우 적합합니다.

먼저 필요한 라이브러리를 설치했는지 확인하세요.

```Plain
pip install transformers torch
```

`transformers` 라이브러리에서는 일반적으로 `AutoModelForCausalLM` 및 `AutoTokenizer` 클래스를 사용하여 모델과 일치하는 가중치 및 토크나이저를 자동으로 로드합니다. 다음 코드는 Hugging Face Hub에서 필수 모델 파일과 토크나이저 구성을 자동으로 다운로드하며, 네트워크 속도에 따라 다소 시간이 걸릴 수 있습니다.

```Python
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer

# Specify model ID
model_id = "Qwen/Qwen1.5-0.5B-Chat"

# Set device, prioritize GPU
device = "cuda" if torch.cuda.is_available() else "cpu"
print(f"Using device: {device}")

# Load tokenizer
tokenizer = AutoTokenizer.from_pretrained(model_id)

# Load model and move it to specified device
model = AutoModelForCausalLM.from_pretrained(model_id).to(device)

print("Model and tokenizer loaded!")
```

대화 프롬프트를 만들어 보겠습니다. Qwen1.5-Chat 모델은 특정 대화 템플릿을 따릅니다. 그런 다음 이전 단계에서 로드된 `tokenizer`를 사용하여 텍스트 프롬프트를 모델이 이해할 수 있는 숫자 ID(즉, 토큰 ID)로 변환할 수 있습니다.

```Python
# Prepare dialogue input
messages = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Hello, please introduce yourself."}
]

# Use tokenizer's template to format input
text = tokenizer.apply_chat_template(
    messages,
    tokenize=False,
    add_generation_prompt=True
)

# Encode input text
model_inputs = tokenizer([text], return_tensors="pt").to(device)

print("Encoded input text:")
print(model_inputs)

>>>
{'input_ids': tensor([[151644, 8948, 198, 2610, 525, 264,  10950, 17847, 13,151645, 198, 151644, 872, 198, 108386, 37945, 100157, 107828,1773, 151645, 198, 151644, 77091, 198]], device='cuda:0'), 'attention_mask': tensor([[1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1]],
       device='cuda:0')}
```

이제 모델의 `generate()` 메서드를 호출하여 답변을 생성할 수 있습니다. 모델은 답변을 나타내는 일련의 토큰 ID를 출력합니다.

마지막으로 토크나이저의 `decode()` 메서드를 사용하여 이러한 숫자 ID를 사람이 읽을 수 있는 텍스트로 다시 변환해야 합니다.

```Python
# Use model to generate answer
# max_new_tokens controls the maximum number of new Tokens the model can generate
generated_ids = model.generate(
    model_inputs.input_ids,
    max_new_tokens=512
)

# Truncate the input part from generated Token IDs
# This way we only decode the newly generated part by the model
generated_ids = [
    output_ids[len(input_ids):] for input_ids, output_ids in zip(model_inputs.input_ids, generated_ids)
]

# Decode generated Token IDs
response = tokenizer.batch_decode(generated_ids, skip_special_tokens=True)[0]

print("\nModel's answer:")
print(response)

>>>
My name is Tongyi Qianwen, a pre-trained language model developed by Alibaba Cloud. I can answer questions, create text, express opinions, and write code. My main functions are to provide help in multiple fields, including but not limited to: language understanding, text generation, machine translation, question-answering systems, etc. Is there anything I can help you with?
```

모든 코드를 실행한 후 로컬 컴퓨터에서 Qwen 모델에 대한 모델 생성 소개를 볼 수 있습니다. 축하합니다. 오픈 소스 대규모 언어 모델을 로컬에 성공적으로 배포하고 실행했습니다!

### 3.2.4 모델 선택

이전 섹션에서는 소규모 오픈 소스 언어 모델을 로컬에서 성공적으로 실행했습니다. 이는 자연스럽게 에이전트 개발자에게 중요한 질문을 제기합니다. 수백 개의 모델이 등장하는 현재 상황에서 특정 작업에 가장 적합한 모델을 어떻게 선택해야 할까요?

언어 모델을 선택하는 것은 단순히 "가장 크고 가장 강력한" 것을 추구하는 것이 아니라 성능, 비용, 속도 및 배포 방법의 균형을 맞추는 의사 결정 프로세스입니다. 이 섹션에서는 먼저 모델 선택을 위한 몇 가지 주요 고려 사항을 정리한 다음 현재 주류 폐쇄 소스 및 오픈 소스 모델을 검토합니다.

대규모 언어 모델 기술은 새로운 모델과 버전이 지속적으로 등장하고 극도로 빠른 반복을 통해 급속한 개발 단계에 있기 때문에 이 섹션에서는 작성 시 현재 주류 모델에 대한 개요와 선택 고려 사항을 제공하려고 노력하지만 독자는 언급된 특정 모델 버전과 성능 데이터가 시간이 지남에 따라 변경될 수 있으며 포괄적이지 않고 일부 작업만 나열된다는 점에 유의해야 합니다. 우리는 에이전트 개발에 있어 핵심적인 기술적 특성, 개발 동향 및 일반적인 선택 원칙을 소개하는 데 더 중점을 둡니다.

**3.2.4.1 모델 선택 시 주요 고려 사항**

에이전트에 대한 대규모 언어 모델을 선택할 때 다음 차원에서 종합적으로 평가할 수 있습니다.

- **성능 및 기능**: 이것이 핵심 고려 사항입니다. 다양한 모델이 다양한 작업에 탁월합니다. 일부는 논리적 추론과 코드 생성에 능숙하고, 다른 일부는 창의적인 글쓰기나 다국어 번역에 능숙합니다. 일부 공개 벤치마크 순위표(예: LMSys Chatbot Arena 순위표)를 참조하여 모델의 포괄적인 기능을 평가할 수 있습니다.
- **비용**: 비공개 소스 모델의 경우 비용은 주로 토큰 수에 따라 부과되는 API 호출 요금으로 나타납니다. 오픈 소스 모델의 경우 비용은 로컬 배포에 필요한 하드웨어(GPU, 메모리) 및 작업으로 나타납니다. 애플리케이션의 예상 사용량과 예산에 따라 선택해야 합니다.
- **속도(지연)**: 실시간 상호 작용이 필요한 에이전트(예: 고객 서비스, 게임 NPC)의 경우 모델 응답 속도가 중요합니다. 일부 경량 또는 최적화된 모델(예: GPT-3.5 Turbo, Claude 3.5 Sonnet)은 대기 시간이 더 좋습니다.
- **컨텍스트 창**: 모델이 한 번에 처리할 수 있는 토큰 수의 상한입니다. 긴 문서를 이해하거나, 코드 저장소를 분석하거나, 장기간 대화 메모리를 유지해야 하는 에이전트의 경우 컨텍스트 창이 더 큰 모델(예: 128K 토큰 이상)을 선택해야 합니다.
- **배포 방법**: API를 사용하는 것이 가장 간단하고 편리하지만 데이터를 제3자에게 전송해야 하며 서비스 제공업체 약관이 적용됩니다. 로컬 배포는 데이터 개인 정보 보호와 최고 수준의 자율성을 보장할 수 있지만 기술 및 하드웨어 요구 사항이 더 높습니다.
- ​​**생태계 및 툴체인**: 모델의 인기도 주변 생태계의 성숙도를 결정합니다. 주류 모델은 일반적으로 개발을 크게 가속화하고 난이도를 줄일 수 있는 풍부한 커뮤니티 지원, 튜토리얼, 사전 훈련된 모델, 미세 조정 도구 및 호환 가능한 개발 프레임워크(예: LangChain, LlamaIndex, Hugging Face Transformers)를 갖추고 있습니다. 활발한 커뮤니티와 완전한 도구 체인이 있는 모델을 선택하면 문제가 발생할 때 솔루션과 리소스를 더 쉽게 찾을 수 있습니다.
- **미세 조정 가능성 및 사용자 정의**: 도메인별 데이터를 처리하거나 특정 작업을 수행해야 하는 에이전트의 경우 모델 미세 조정 기능이 중요합니다. 일부 모델은 편리한 미세 조정 인터페이스와 도구를 제공하므로 개발자는 자체 데이터 세트를 사용하여 교육을 사용자 정의할 수 있으므로 특정 시나리오에서 모델 성능과 정확도가 크게 향상됩니다. 오픈 소스 모델은 일반적으로 이와 관련하여 더 큰 유연성을 제공합니다.
- **안전 및 윤리**: 대규모 언어 모델이 널리 적용됨에 따라 잠재적인 안전 위험과 윤리적 문제가 점점 더 두드러지고 있습니다. 모델을 선택할 때 편견, 독성, 환각 등에 대한 성능과 모델 안전 및 책임 있는 AI에 대한 서비스 제공업체 또는 오픈 소스 커뮤니티의 투자를 고려하세요. 대중을 대상으로 하거나 민감한 정보를 포함하는 애플리케이션의 경우 모델 안전성과 윤리적 준수는 무시할 수 없는 고려 사항입니다.

**3.2.4.2 비공개 소스 모델 개요**

비공개 소스 모델은 일반적으로 현재 AI 기술의 최첨단을 대표하며 안정적이고 사용하기 쉬운 API 서비스를 제공하므로 고성능 에이전트 구축을 위한 첫 번째 선택입니다.

1. **OpenAI GPT 시리즈**: 대규모 모델 시대를 연 GPT-3부터 RLHF(Reinforcement Learning from Human Feedback)를 도입하고 인간의 의도와 정렬을 달성한 ChatGPT, 멀티모달 시대를 연 GPT-4에 이르기까지 OpenAI는 계속해서 산업 발전을 선도하고 있습니다. 최신 GPT-5는 멀티모달 기능과 일반 지능을 새로운 차원으로 끌어올려 텍스트, 오디오 및 이미지 입력을 원활하게 처리하고 해당 출력을 생성하며 응답 속도와 자연스러움이 크게 향상되었으며 특히 실시간 음성 대화에서 탁월합니다.
2. **Google Gemini 시리즈**: Google DeepMind의 Gemini 시리즈 모델은 텍스트, 코드, 오디오/비디오, 이미지를 포함한 여러 양식의 통합 처리의 핵심 기능과 매우 긴 컨텍스트 창을 통한 대량 정보 처리의 이점을 갖춘 네이티브 다중 양식의 대표 모델입니다. Gemini Ultra는 매우 복잡한 작업에 적합한 가장 강력한 모델입니다. Gemini Pro는 높은 성능과 효율성을 제공하여 광범위한 작업에 적합합니다. Gemini Nano는 온디바이스 배포에 최적화되어 있습니다. Gemini 2.5 Pro 및 Gemini 2.5 Flash와 같은 최신 Gemini 2.5 시리즈 모델은 추론 기능과 컨텍스트 창을 더욱 개선합니다. 특히 Gemini 2.5 Flash는 더 빠른 추론 속도와 비용 효율성을 갖추고 있어 빠른 응답이 필요한 시나리오에 적합합니다.
3. **Anthropic Claude 시리즈**: Anthropic은 AI 안전과 책임 있는 AI에 중점을 둔 회사입니다. Claude 시리즈 모델은 설계 단계부터 AI 안전을 최우선으로 삼았으며, 긴 문서 처리, 유해한 출력 감소, 지침 준수에 대한 신뢰성으로 유명하며 기업 애플리케이션에서 큰 선호를 받고 있습니다. Claude 3 시리즈에는 Claude 3 Opus(가장 지능적이고 강력한 성능), Claude 3 Sonnet(성능과 속도의 균형 잡힌 선택), Claude 3 Haiku(가장 빠르고 컴팩트한 모델, 거의 실시간 상호 작용에 적합)가 포함됩니다. Claude 4 Opus와 같은 최신 Claude 4 시리즈 모델은 일반 지능, 복잡한 추론 및 코드 생성 분야에서 상당한 발전을 이루었으며 긴 컨텍스트 및 다중 모드 작업을 처리하는 기능이 더욱 향상되었습니다.
4. **국내 주류 모델**: 중국은 Baidu ERNIE Bot, Tencent Hunyuan, Huawei Pangu-α, iFlytek SparkDesk 및 Moonshot AI로 대표되는 대규모 언어 모델 분야에서 경쟁력 있는 많은 비공개 소스 모델을 선보였습니다. 이러한 국내 모델은 중국 가공 분야에서 자연적인 이점을 갖고 있으며 현지 산업에 큰 힘을 실어줍니다.

**3.2.4.3 오픈 소스 모델 개요**

오픈 소스 모델은 개발자에게 최고 수준의 유연성, 투명성 및 자율성을 제공하여 번영하는 커뮤니티 생태계를 촉진합니다. 이를 통해 개발자는 로컬로 배포하고, 맞춤형 미세 조정을 수행하고, 완전한 모델 제어를 할 수 있습니다.

- **Meta Llama 시리즈**: Meta의 Llama 시리즈는 오픈 소스 대규모 언어 모델에서 중요한 이정표입니다. 이 시리즈는 탁월한 종합 성능, 공개 라이센스 계약 및 강력한 커뮤니티 지원을 통해 많은 파생 프로젝트 및 연구의 기반이 되었습니다. Llama 4 시리즈는 2025년 4월에 출시되었으며, 이는 특정 작업을 처리하는 데 필요한 모델 부분만 활성화하여 계산 효율성을 크게 향상시키는 MoE(Mixture of Experts) 아키텍처를 채택한 Meta의 첫 번째 모델입니다. 이 시리즈에는 뚜렷이 구분되는 세 가지 모델이 포함되어 있습니다. Llama 4 Scout는 긴 문서 분석 및 모바일 배포를 위해 설계된 1천만 개의 토큰 컨텍스트 창을 지원합니다. Llama 4 Maverick은 다중 모드 기능에 중점을 두고 코딩, 복잡한 추론 및 다국어 지원에 뛰어납니다. Llama 4 Behemoth는 여러 STEM 벤치마크에서 경쟁사를 능가하며 현재 Meta의 가장 강력한 모델입니다.
- **Mistral AI 시리즈**: 프랑스의 Mistral AI는 "작은 크기, 고성능" 모델 디자인으로 유명합니다. 최신 모델인 Mistral Medium 3.1은 2025년 8월에 출시되었으며, 코드 생성, STEM 추론, 도메인 간 Q&A 등의 작업에서 정확도와 응답 속도가 크게 향상되었으며, Claude Sonnet 3.7 및 Llama 4 Maverick 및 기타 유사 모델보다 벤치마크 성능이 뛰어났습니다. 기본 멀티모달 기능이 있고, 혼합된 이미지와 텍스트 입력을 동시에 처리할 수 있으며, 기업이 보다 쉽게 ​​브랜드에 맞는 출력을 달성할 수 있도록 지원하는 "톤 적응 레이어"가 내장되어 있습니다.
- **국내 오픈소스 세력**: Alibaba의 **Qwen(Tongyi Qianwen)** 시리즈, Tsinghua University와 Zhipu AI의 **ChatGLM** 시리즈 협력 등 국내 제조업체 및 연구 기관에서도 오픈소스를 적극적으로 수용하고 있습니다. 그들은 강력한 중국 역량을 제공하고 주변에 활발한 커뮤니티를 구축했습니다.

에이전트 개발자에게 비공개 소스 모델은 '즉시 사용 가능한' 편의성을 제공하는 반면, 오픈 소스 모델은 '사용자 정의의 자유'를 제공합니다. 두 진영의 특성과 대표 모델을 이해하는 것이 에이전트 프로젝트를 위한 현명한 기술 선택의 첫 번째 단계입니다.

## 3.3 확장 법칙과 대규모 언어 모델의 한계

LLM(대형 언어 모델)은 기능 경계가 지속적으로 확장되고 애플리케이션 시나리오가 더욱 풍부해지면서 최근 몇 년간 눈에 띄는 발전을 이루었습니다. 그러나 이러한 성과 뒤에는 모델 규모, 데이터 양 및 계산 리소스, 즉 **확장 법칙** 간의 관계에 대한 깊은 이해가 있습니다. 한편, 새로운 기술로서 LLM은 많은 도전과 한계에 직면해 있습니다. 이 섹션에서는 독자가 LLM의 기능 경계를 포괄적으로 이해하여 에이전트를 구축할 때 강점을 활용하고 약점을 피할 수 있도록 돕는 것을 목표로 이러한 핵심 개념을 깊이 탐구할 것입니다.

### 3.3.1 스케일링 법칙

**확장 법칙**은 최근 몇 년 동안 대규모 언어 모델 분야에서 가장 중요한 발견 중 하나입니다. 그들은 모델 성능과 모델 매개변수 수, 훈련 데이터 양 및 계산 리소스 사이에 예측 가능한 거듭제곱 관계가 있음을 보여줍니다. 이 발견은 대규모 언어 모델의 지속적인 개발을 위한 이론적 지침을 제공하며, 리소스 투자를 늘리면 모델 성능이 체계적으로 향상될 수 있다는 기본 논리를 명확히 합니다.

연구에 따르면 로그-로그 좌표계에서 모델 성능(일반적으로 손실로 측정)은 매개변수 수, 데이터 볼륨 및 계산<sup>[9]</sup>의 세 가지 요소 모두와 원활한 거듭제곱 관계를 보여줍니다. 간단히 말해서, 이 세 가지 요소를 지속적으로 비례적으로 증가시키는 한 모델 성능은 명백한 병목 현상 없이 예측 가능하고 원활하게 향상됩니다. 이 발견은 대규모 모델 설계 및 교육에 대한 명확한 지침을 제공합니다. 리소스 제약 내에서 모델 규모와 교육 데이터 볼륨을 최대한 최대화합니다.

초기 연구는 모델 매개변수 수를 늘리는 데 더 중점을 두었지만 2022년에 제안된 DeepMind의 '친칠라 법칙'은 <sup>[10]</sup>에 중요한 수정 사항을 적용했습니다. 이 법칙은 주어진 계산 예산 하에서 최적의 성능을 달성하기 위해 **모델 매개변수 수와 훈련 데이터 볼륨 사이에 최적의 비율이 있음**을 지적합니다. 특히 최적의 모델은 이전에 일반적으로 믿어졌던 것보다 작아야 하지만 훨씬 더 많은 데이터를 사용하여 훈련해야 합니다. 예를 들어, 700억 개의 매개변수가 있는 Chinchilla 모델은 GPT-3(1,750억 개의 매개변수)보다 4배 더 많은 데이터로 훈련되었기 때문에 실제로 GPT-3보다 성능이 뛰어납니다. 이 발견은 "더 클수록 좋다"는 일방적인 인식을 바로잡았고, 데이터 효율성의 중요성을 강조했으며, 이후의 많은 효율적인 대형 모델(예: Llama 시리즈)의 설계를 안내했습니다.

확장법칙의 가장 놀라운 산물은 '능력발현'입니다. 소위 능력 출현은 모델 규모가 특정 임계값에 도달하면 소규모 모델에는 존재하지 않거나 성능이 떨어지는 완전히 새로운 능력이 갑자기 나타나는 것을 의미합니다. 예를 들어 **생각의 사슬**, **지침 따르기**, 다단계 추론, 코드 생성 및 기타 기능은 모두 모델 매개변수 수가 수백억 또는 심지어 수천억에 도달한 후에야 크게 나타났습니다. 이러한 현상은 대규모 언어 모델이 단순히 암기하고 낭독하는 것이 아니라는 것을 나타냅니다. 학습 중에 더 깊은 수준의 추상화 및 추론 능력을 형성했을 수도 있습니다. 에이전트 개발자에게 기능 출현은 충분히 큰 규모의 모델을 선택하는 것이 복잡한 자율적 의사 결정 및 계획 기능을 달성하기 위한 전제 조건임을 의미합니다.

### 3.3.2 모델 환각

**모델 환각**은 일반적으로 객관적인 사실, 사용자 입력 또는 상황 정보와 모순되거나 존재하지 않는 사실, 개체 또는 이벤트를 생성하는 대규모 언어 모델에서 생성된 콘텐츠를 나타냅니다. 환각의 본질은 모델이 정확한 검색이나 추론보다는 생성 중에 정보를 지나치게 "조작"한다는 것입니다. 발현 형태에 따라 환각은 다음과 같은 여러 유형<sup>[11]</sup>로 나눌 수 있습니다.

- **사실적 환각**: 모델은 실제 사실과 일치하지 않는 정보를 생성합니다.
- **신실한 환각**: 텍스트 요약 및 번역과 같은 작업에서 생성된 콘텐츠는 원본 텍스트 의미를 충실하게 반영하지 못합니다.
- **내재적 환각**: 모델 생성 콘텐츠는 입력 정보와 직접적으로 모순됩니다.

환각은 여러 요인이 함께 작용하여 생성됩니다. 첫째, 훈련 데이터에는 오류가 있거나 모순되는 정보가 포함될 수 있습니다. 둘째, 모델의 자동 회귀 생성 메커니즘은 내장된 사실 확인 모듈 없이 다음으로 가능성이 가장 높은 토큰만 예측한다고 결정합니다. 마지막으로, 복잡한 추론이 필요한 작업에 직면할 때 모델은 논리적 체인에서 오류를 범하여 잘못된 결론을 "조작"할 수 있습니다. 예를 들어 여행 계획 담당자가 존재하지 않는 관광지를 추천하거나 잘못된 항공편 번호로 항공권을 예약할 수 있습니다.

또한 대규모 언어 모델은 지식 적시성이 부족하고 교육 데이터의 편향과 같은 문제에 직면합니다. 대규모 언어 모델 기능은 교육 데이터에서 비롯됩니다. 이는 모델이 보유한 지식이 훈련 데이터를 수집할 당시의 최신 자료임을 의미합니다. 이 날짜 이후에 발생하는 이벤트, 새로 등장한 개념 또는 최신 사실의 경우 모델이 인식하거나 정확하게 대답할 수 없습니다. 한편, 훈련 데이터에는 인간 사회의 다양한 편견과 고정관념이 포함되어 있는 경우가 많습니다. 모델이 이 데이터를 학습할 때 필연적으로 이러한 편향<sup>[12]</sup>을 흡수하고 반영합니다.

대규모 언어 모델의 신뢰성을 향상시키기 위해 연구원과 개발자는 환각을 감지하고 완화하는 다양한 방법을 적극적으로 탐색하고 있습니다.

1. **데이터 수준**: 고품질 데이터 정리, 사실적 지식 도입, RLHF(인간 피드백을 통한 강화 학습)<sup>[13]</sup>를 통해 소스에서 환각을 줄입니다.
2. **모델 수준**: 새로운 모델 아키텍처를 탐색하거나 모델이 생성된 콘텐츠에 대한 불확실성을 표현할 수 있도록 합니다.
3. **추론 및 생성 수준**:
1. **RAG(Retrieval-Augmented Generation)**<sup>[14]</sup>: 이것은 현재 환각을 완화하는 효과적인 방법 중 하나입니다. RAG 시스템은 생성 전에 외부 지식 기반(예: 문서 데이터베이스, 웹 페이지)에서 관련 정보를 검색한 다음 검색된 정보를 컨텍스트로 사용하여 모델이 사실 기반 답변을 생성하도록 안내합니다.
2. **다단계 추론 및 검증**: 모델이 다단계 추론을 수행하고 각 단계에서 자체 점검 또는 외부 검증을 수행하도록 안내합니다.
3. **외부 도구 소개**: 모델이 외부 도구(예: 검색 엔진, 계산기, 코드 해석기)를 호출하여 실시간 정보를 얻거나 정확한 계산을 수행할 수 있도록 허용합니다.

환각 문제는 단기적으로 완전히 제거하기는 어렵지만 위의 전략을 통해 발생 빈도와 영향을 크게 줄여 실제 응용 프로그램에서 대규모 언어 모델의 신뢰성과 실용성을 향상시킬 수 있습니다.

## 3.4 장 요약

이 장에서는 핵심 구성 요소인 LLM(대형 언어 모델)에 중점을 두고 에이전트 구축에 필요한 기본 지식을 소개했습니다. 초기 언어 모델 개발에서 시작된 콘텐츠는 Transformer 아키텍처를 자세히 설명하고 LLM과 상호 작용하는 방법을 소개했습니다. 마지막으로 이 장에서는 현재의 주류 모델 생태계, 개발 패턴 및 고유한 한계를 정리했습니다.

**핵심 지식 검토:**

- **모델 진화 및 핵심 아키텍처**: 이 장에서는 통계 언어 모델(N-gram)부터 신경망 모델(RNN, LSTM), 최신 LLM의 기반을 마련한 Transformer 아키텍처까지 추적했습니다. "하향식" 코드 구현을 통해 이 장에서는 Transformer의 핵심 구성 요소를 분석하고 병렬 계산 및 장거리 종속성 캡처에서 self-attention 메커니즘의 핵심 역할을 설명했습니다.
- **모델과의 상호 작용 방법**: 이 장에서는 LLM과 상호 작용하는 두 가지 핵심 측면인 프롬프트 엔지니어링과 토큰화를 소개했습니다. 전자는 모델 동작을 안내하고 후자는 모델 입력 처리를 이해하기 위한 기초입니다. 오픈소스 모델을 로컬에 배포하고 실행하는 실습을 통해 이론적 지식을 실제 운영에 적용했습니다.
- **모델 생태계 및 선택**: 이 장에서는 에이전트 모델을 선택할 때 중요하게 생각하는 핵심 요소와 OpenAI GPT 및 Google Gemini로 대표되는 비공개 소스 모델과 Llama 및 Mistral로 대표되는 오픈 소스 모델의 특성과 포지셔닝을 개략적으로 정리했습니다.
- **법칙 및 제한 사항**: 이 장에서는 LLM 기능 개선을 주도하는 확장 법칙을 살펴보고 기본 원칙을 설명했습니다. 한편, 이 장에서는 신뢰할 수 있고 강력한 에이전트를 구축하는 데 중요한 사실적 환각 및 오래된 지식과 같은 모델의 고유한 한계도 분석했습니다.

**LLM 기초부터 빌딩 에이전트까지:**

이 장의 LLM 기초는 주로 모든 사람이 대형 모델의 탄생 및 개발 과정을 더 잘 이해하는 데 도움이 되며 여기에는 에이전트 설계에 대한 몇 가지 생각도 포함되어 있습니다. 예를 들어 에이전트 계획 및 의사 결정을 안내하는 효과적인 프롬프트를 디자인하는 방법, 작업 요구 사항에 따라 적절한 모델을 선택하는 방법, 모델 환각을 피하기 위해 에이전트 워크플로에 확인 메커니즘을 추가하는 방법 등 이러한 문제에 대한 솔루션은 모두 이 장의 기초를 바탕으로 구축되었습니다. 이제 이론에서 실습으로 전환할 준비가 되었습니다. 다음 장에서는 이 장에서 배운 지식을 실제 에이전트 설계에 적용하여 고전적인 에이전트 패러다임 구성을 탐색하기 시작합니다.

## 연습

1. 자연어 처리에서 언어 모델은 통계 모델에서 신경망 모델로 진화했습니다.

- Bigram 모델에서 `agent works` 문장의 확률을 계산하려면 본 장에서 제공되는 미니 코퍼스(`datawhale agent learns`, `datawhale agent works`)를 사용하십시오.
- N-gram 모델의 핵심 가정은 Markov 가정입니다. 이 가정의 의미와 N-gram 모델의 근본적인 한계에 대해 설명해 주세요.
- 신경망 언어 모델(RNN/LSTM)과 Transformer는 각각 N-gram 모델 한계를 어떻게 극복합니까? 각각의 장점은 무엇입니까?

2. Transformer 아키텍처<sup>[4]</sup>는 최신 대규모 언어 모델의 기초입니다. 그 중에는:

> **힌트**: 이해를 돕기 위해 이 장의 섹션 3.1.2의 코드 구현을 결합할 수 있습니다.

- Self-Attention 메커니즘의 핵심 아이디어는 무엇입니까?
- RNN은 직렬로 처리해야 하는데 Transformer는 시퀀스를 병렬로 처리할 수 있는 이유는 무엇입니까? 위치 인코딩은 어떤 역할을 합니까?
- 디코더 전용 아키텍처와 전체 인코더-디코더 아키텍처의 차이점은 무엇입니까? 현재 주류 대형 언어 모델이 모두 디코더 전용 아키텍처를 채택하는 이유는 무엇입니까?

3. 텍스트 하위 단어 토큰화 알고리즘은 텍스트를 모델이 처리할 수 있는 토큰 시퀀스로 변환하는 대규모 언어 모델의 핵심 기술입니다. 왜 "문자"나 "단어"를 모델 입력 단위로 직접 사용할 수 없나요? BPE(바이트 쌍 인코딩) 알고리즘은 어떤 문제를 해결합니까?

4. 이 장의 섹션 3.2.3에서는 오픈 소스 대규모 언어 모델을 로컬로 배포하는 방법을 소개했습니다. 다음 연습과 분석을 완료하십시오.

> **힌트**: 실습 연습 문제입니다. 실제 작동을 권장합니다

- 이 장의 지침에 따라 경량 오픈 소스 모델을 로컬로 배포하고([Qwen3-0.6B](https://modelscope.cn/models/Qwen/Qwen3-0.6B) 권장) 샘플링 매개변수를 조정해 보고 출력에 미치는 영향을 관찰합니다.
- 특정 작업(예: 텍스트 분류, 정보 추출, 코드 생성 등)을 선택하고 다양한 프롬프트 전략(예: Zero-shot, Few-shot, Chain-of-Thought)과 출력 결과에 미치는 영향 차이를 설계 및 비교합니다.
- 성능, 비용, 제어 가능성, 개인 정보 보호 등의 측면에서 비공개 소스 모델과 오픈 소스 모델을 비교합니다.
- 엔터프라이즈급 고객 서비스 에이전트를 구축하려면 어떤 유형의 모델을 선택하시겠습니까? 어떤 요소를 고려해야 합니까?

5. 모델 환각<sup>[11]</sup>는 현재 대규모 언어 모델의 주요 제한 사항 중 하나입니다. 이 장에서는 환각을 완화하는 방법(예: 검색 증강 생성, 다단계 추론, 외부 도구 호출)을 소개했습니다.

- 하나를 선택하고 작동 원리와 적용 가능한 시나리오를 설명하십시오.
- 최첨단 연구 및 논문을 조사합니다. 모델 환각을 완화할 수 있는 다른 방법이 있습니까? 그리고 그 방법에는 어떤 개선 사항과 이점이 있습니까?

6. 논문 연구의 핵심 내용 요약, 논문에 대한 질문에 대한 답변, 핵심 정보 추출, 다양한 논문의 관점 비교 등 연구자가 학술 논문을 빠르게 읽고 이해하는 데 도움이 되는 논문 지원 읽기 에이전트를 설계한다고 가정해 보겠습니다. 답변해 주세요.

- 에이전트를 설계할 때 기본 모델로 어떤 모델을 선택하시겠습니까? 선택할 때 어떤 요소를 고려해야 합니까?
- ​​학술 논문을 더 잘 이해할 수 있도록 모델을 안내하는 프롬프트를 디자인하는 방법은 무엇입니까? 학술 논문은 일반적으로 매우 길며 모델의 컨텍스트 창 제한을 초과할 수 있습니다. 이 문제를 어떻게 해결하시겠습니까?
- 학문적 연구는 엄격합니다. 즉, 에이전트가 생성한 정보가 정확하고 객관적이며 원본 텍스트에 충실하도록 보장해야 합니다. 이 요구 사항을 더 잘 달성하려면 시스템에 어떤 디자인을 추가해야 한다고 생각하십니까?

## 참고자료

[1] Bengio, Y., Ducharme, R., Vincent, P., & Jauvin, C. (2003). 신경 확률적 언어 모델. *머신러닝 연구 저널*, 3, 1137-1155.

[2] 엘만, J. L.(1990). 시간에 맞춰 구조를 찾아보세요. *인지과학*, 14(2), 179-211.

[3] Hochreiter, S., & Schmidhuber, J. (1997). 장단기 기억. *신경 계산*, 9(8), 1735-1780.

[4] Vaswani, A., Shazeer, N., Parmar, N., Uszkoreit, J., Jones, L., Gomez, A. N., ... & Polosukhin, I. (2017). 주의가 필요한 전부입니다. *신경 정보 처리 시스템의 발전*(pp. 5998-6008).

[5] Radford, A., Narasimhan, K., Salimans, T., & Sutskever, I. (2018). 생성적 사전 훈련을 통해 언어 이해를 향상시킵니다. 오픈AI.

[6] 게이지, P.(1994). 데이터 압축을 위한 새로운 알고리즘입니다. *C 사용자 저널*, *12*(2), 23-38.

[7] Schuster, M., & Nakajima, K. (2012, 3월). 일본어와 한국어 음성 검색. *2012 음향, 음성 및 신호 처리에 관한 IEEE 국제 회의(ICASSP)*(pp. 5149-5152). IEEE.

[8] Kudo, T., & Richardson, J. (2018). SentencePiece: 신경 텍스트 처리를 위한 간단하고 언어 독립적인 하위 단어 토크나이저 및 디토크나이저입니다. *arXiv 사전 인쇄본 arXiv:1808.06226*.

[9] Kaplan, J., McCandlish, S., Henighan, T., Brown, T. B., Chess, B., Child, R., ... & Amodei, D. (2020). 신경 언어 모델의 확장 법칙. arXiv 사전 인쇄 arXiv:2001.08361.

[10] Hoffmann, J., Borgeaud, E., Mensch, A., Buchatskaya, E., Cai, T., Rutherford, R., ... & Sifre, L. (2022). 컴퓨팅 최적의 대형 언어 모델 교육. arXiv 사전 인쇄 arXiv:2203.07678.

[11] Ji, Z., Lee, N., Fries, R., Yu, T., & Su, D. (2023). 대규모 언어 모델의 환각 조사.

[12] Bender, E. M., Gebru, T., McMillan-Major, A. 및 Mitchell, M. (2021). 확률론적 앵무새의 위험성: 언어 모델이 너무 클 수 있습니까? .

[13] Christiano, P., Leike, J., Brown, T. B., Martic, M., Legg, S., & Amodei, D. (2017). 인간의 선호도를 바탕으로 한 심층 강화 학습. *arXiv 사전 인쇄 arXiv:1706.03741*.

[14] Lewis, P., Perez, E., Piktus, A., Petroni, F., Karpukhin, V., Goswami, N., ... & Kiela, D.(2020). 지식 집약적인 NLP 작업을 위한 검색 강화 생성입니다. *신경 정보 처리 시스템의 발전*(pp. 9459-9474).

