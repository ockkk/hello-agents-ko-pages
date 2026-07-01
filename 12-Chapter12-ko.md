# 12장: 에이전트 성능 평가

이전 장에서는 다양한 에이전트 패러다임, 도구 시스템, 메모리 메커니즘 및 강화 학습 훈련을 구현하여 HelloAgents 프레임워크의 핵심 기능을 구축했습니다. 에이전트 시스템을 구축할 때 핵심 문제인 **에이전트 성과를 어떻게 객관적으로 평가할 것인가?**도 해결해야 합니다. 특히 다음 질문에 답해야 합니다.

1. 에이전트가 기대되는 능력을 갖추고 있습니까?
2. 다양한 작업에서 어떻게 수행됩니까?
3. 다른 에이전트에 비해 어느 정도 수준인가요?

이번 장에서는 HelloAgents에 **성능 평가 시스템**을 추가하겠습니다. 에이전트 평가의 이론적 기초를 깊이 이해하고 평가 도구를 구현합니다.

## 12.1 에이전트 평가 기본 사항

### 12.1.1 에이전트 평가가 필요한 이유

이제 강력한 추론 및 도구 호출 기능을 이미 보유하고 있는 SimpleAgent가 있습니다. 일반적인 사용 시나리오를 살펴보겠습니다.

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import SearchTool

# Create LLM and agent
llm = HelloAgentsLLM()

# Create a system prompt emphasizing tool use
system_prompt = """You are an AI assistant that can use search tools to obtain the latest information.

When you need to search for information, please use the following format:
[TOOL_CALL:search:search keywords]

For example:
- [TOOL_CALL:search:latest AI news]
- [TOOL_CALL:search:Python programming tutorial]

Please use the search tool to obtain the latest information before answering questions."""

agent = SimpleAgent(name="AI Assistant", llm=llm, system_prompt=system_prompt)

# Add search tool
agent.add_tool(SearchTool())

# Example: Use search tool to answer questions
response = agent.run("What are the latest AI technology development trends?")
print(f"\nAnswer: {response}")
```

이 에이전트는 정상적으로 작동하지만 핵심 문제에 직면합니다. 성능을 객관적으로 평가하는 방법은 무엇입니까? 프롬프트를 최적화하거나 LLM 모델을 변경할 때 실제 개선이 있는지 어떻게 알 수 있습니까? 프로덕션 환경에 배포하기 전에 에이전트 안정성을 어떻게 보장합니까? 이러한 문제는 모두 체계적인 평가를 통해 해결되어야 합니다.

상담원 평가의 핵심 가치는 상담원 능력을 측정하는 표준화된 방법을 제공하는 것입니다. 평가를 통해 특정 수치 지표로 에이전트 성능을 정량화하고, 다양한 설계 솔루션의 장점을 객관적으로 비교하고, 특정 시나리오에서 에이전트 약점을 신속하게 발견하고, 사용자에게 에이전트 신뢰성을 입증할 수 있습니다.

기존 소프트웨어 테스트와 달리 에이전트 평가는 독특한 과제에 직면해 있습니다. 첫 번째는 출력 불확실성입니다. 동일한 질문에 정답이 여러 개 있을 수 있으므로 단순한 옳고 그름을 판단하기가 어렵습니다. 두 번째는 평가 기준의 다양성입니다. 작업마다 서로 다른 평가 방법이 필요합니다. 도구 호출은 함수 서명을 확인해야 하고, Q&A 작업은 의미적 유사성을 평가해야 합니다. 마지막으로 평가 비용이 높습니다. 각 평가에는 수많은 API 호출이 필요하며 잠재적으로 수백 위안 이상의 비용이 소요될 수 있습니다.

이러한 문제를 해결하기 위해 학계와 업계에서는 여러 표준화된 **벤치마크**를 제안했습니다. 이러한 벤치마크는 통합된 데이터 세트, 평가 지표 및 점수 매기기 방법을 제공하므로 동일한 표준 하에서 다양한 에이전트 시스템을 평가하고 비교할 수 있습니다.

12.1.2 주류 평가 벤치마크 개요

에이전트 평가 분야에서는 영향력 있는 여러 벤치마크 테스트가 등장했습니다. 다음은 몇 가지 주요 평가 벤치마크 및 지표입니다.

**(1) 도구 호출 성능 평가**

도구 호출은 에이전트의 핵심 기능 중 하나입니다. 에이전트는 사용자 의도를 이해하고, 적절한 도구를 선택하고, 함수 호출을 올바르게 구성해야 합니다. 관련 평가 벤치마크는 다음과 같습니다.

- **BFCL(Berkeley Function Calling Leaderboard)**<sup>[1]</sup>: UC Berkeley에서 시작되었으며 단순, 다중, 병렬, 관련성 등 4가지 범주를 포괄하는 1120개 이상의 테스트 샘플이 포함되어 있으며 평가를 위해 AST 일치 알고리즘을 사용하고 적당한 데이터 세트 크기, 활성 커뮤니티가 있습니다.
- **ToolBench**<sup>[2]</sup>: Tsinghua University에서 출시되었으며 실제 세계의 복잡한 도구 사용 시나리오를 다루는 16000개 이상의 실제 API 호출 시나리오가 포함되어 있습니다.
- **API-Bank**<sup>[3]</sup>: Microsoft Research에서 출시되었으며 일반적으로 사용되는 53개의 API 도구를 포함하고 에이전트의 API 문서 호출 및 이해를 평가하는 데 중점을 둡니다.

**(2) 종합역량평가**

다단계 추론, 지식 적용, 다중 모드 이해 등을 포함하여 실제 작업에서 에이전트의 종합적인 성능을 평가합니다.

- **GAIA(일반 AI 보조자)**<sup>[4]</sup>: Meta AI와 Hugging Face가 공동으로 출시한 이 문제에는 레벨 1/2/3 난이도로 구분된 466개의 실제 문제가 포함되어 있으며 다단계 추론, 도구 사용, 파일 처리, 웹 검색 기능을 평가하고 Quasi Exact Match 알고리즘을 사용하며 작업이 현실적이고 포괄적입니다.
- **AgentBench**<sup>[5]</sup>: Tsinghua University에서 출시되었으며 다양한 도메인의 8개 작업을 포함하고 에이전트 일반 기능을 종합적으로 평가합니다.
- **WebArena**<sup>[6]</sup>: CMU에서 시작하여 실제 웹 환경에서 에이전트 작업 완료 및 웹 상호 작용 기능을 평가합니다.

**(3) 다중 에이전트 협업 평가**

여러 상담원이 협력하여 작업할 수 있는 능력을 평가합니다.

- **ChatEval**<sup>[7]</sup>: 다중 에이전트 대화 시스템의 품질을 평가합니다.
- **SOTOPIA**<sup>[8]</sup>: 소셜 시나리오에서 에이전트 상호 작용 기능을 평가합니다.
- **맞춤형 협업 시나리오**: 특정 애플리케이션 시나리오에 따라 설계된 평가 작업입니다.

**(4) 공통 평가 지표**

다양한 벤치마크에서는 다양한 평가 측정항목을 사용하며, 일반적인 측정항목은 다음과 같습니다.

- **정확도 측정항목**: 정확성, 정확한 일치, F1 점수는 답변의 정확성을 측정하는 데 사용됩니다.
- **효율성 지표**: 실행 효율성을 측정하는 데 사용되는 응답 시간, 토큰 사용량입니다.
- **강건성 지표**: 오류율, 실패 복구, 내결함성을 측정하는 데 사용됩니다.
- **협업 지표**: 커뮤니케이션 효율성, 작업 완료, 협업 효과를 측정하는 데 사용됩니다.

### 12.1.3 HelloAgents 평가 시스템 설계

학습 곡선과 실용성을 고려하여 이 장에서는 다음 평가 시나리오에 중점을 둘 것입니다.

1. **BFCL**: 도구 호출 기능 평가
   - 선정 근거: 적당한 데이터 세트 규모, 명확한 평가 지표, 활발한 커뮤니티
   - 적용 가능한 시나리오: 에이전트 기능 호출 정확도 평가

2. **GAIA**: 일반 AI 보조 기능 평가
   - 선정근거 : 현실적 과제, 난이도 채점, 강력한 포괄성
   - 적용 가능한 시나리오: 상담원의 종합적인 문제 해결 능력을 평가합니다.

3. **데이터 생성 품질 평가**: LLM에서 생성된 데이터 품질을 평가합니다.
   - 선정근거: 본 사례를 통해 Agent를 활용하여 데이터를 생성하고 평가하는 완전한 실연을 경험한다.
   - 적용 가능한 시나리오: 생성된 학습 데이터 및 테스트 데이터의 품질 평가
   - 평가 방법 : LLM 심사위원, 승률, 수동 검증

이 세 가지 평가 시나리오를 통해 우리는 완전한 평가 시스템을 구축할 것입니다. 그림 12.1은 우리의 평가 시스템 구축 방식을 보여줍니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-1.png" alt="" width="85%"/>
<p>그림 12.1 HelloAgents 평가 시스템 아키텍처</p>
</div>



### 12.1.4 장 학습 목표 및 빠른 경험

먼저 12장의 학습 내용을 살펴보겠습니다.

```
hello_agents/
├── evaluation/                         # Evaluation module
│   └── benchmarks/                     # Evaluation benchmark implementation
│       ├── bfcl/                       # BFCL evaluation implementation
│       │   ├── dataset.py              # BFCL dataset loader
│       │   ├── evaluator.py            # BFCL evaluator (AST matching)
│       │   ├── metrics.py              # BFCL-specific metrics
│       │   └── ast_matcher.py          # AST matching algorithm
│       ├── gaia/                       # GAIA evaluation implementation
│       │   ├── dataset.py              # GAIA dataset loader
│       │   ├── evaluator.py            # GAIA evaluator (quasi exact match)
│       │   ├── metrics.py              # GAIA-specific metrics
│       │   └── quasi_exact_match.py    # Quasi exact match algorithm
│       └── data_generation/            # Data generation evaluation implementation
│           ├── dataset.py              # AIME dataset loader
│           ├── llm_judge.py            # LLM Judge evaluator
│           └── win_rate.py             # Win Rate evaluator
└── tools/builtin/                      # Built-in tools module
    ├── bfcl_evaluation_tool.py         # BFCL evaluation tool
    ├── gaia_evaluation_tool.py         # GAIA evaluation tool
    ├── llm_judge_tool.py               # LLM Judge tool
    └── win_rate_tool.py                # Win Rate tool
```

이 장의 내용에 대한 학습 목표는 평가 도구를 적용하는 능력을 익히는 것입니다. 먼저 개발 환경을 준비하겠습니다.

```bash
# Install HelloAgents framework (Chapter 12 version)
pip install "hello-agents[evaluation]==0.2.7"

# Set environment variables
export HF_TOKEN="your_huggingface_token"     # For GAIA dataset (setup steps will follow)

# Since the official `bfcl-eval` package requires numpy<=2.0.0, which conflicts with HelloAgents main dependencies, separate installation is needed
pip install "numpy==1.26.4" bfcl-eval
```

다음 섹션에서는 각 평가 방법의 자세한 사용법과 소개를 자세히 알아봅니다.

## 12.2 BFCL: 도구 호출 기능 평가

### 12.2.1 BFCL 벤치마크 소개

BFCL(Berkeley Function Calling Leaderboard)은 UC Berkeley<sup>[1]</sup>에서 출시한 함수 호출 성능 평가 벤치마크입니다. 에이전트 시스템에서 도구 호출은 핵심 기능 중 하나입니다. 상담원은 다음 작업을 완료해야 합니다.

1. **작업 요구 사항 이해**: 사용자의 자연어 설명에서 주요 정보 추출
2. **적절한 도구 선택**: 사용 가능한 도구 세트에서 가장 적합한 도구를 선택합니다.
3. **함수 호출 구성**: 함수 이름과 매개변수를 올바르게 입력하세요.
4. **복잡한 시나리오 처리**: 다기능 호출, 병렬 호출과 같은 고급 시나리오 지원

BFCL 벤치마크에는 점점 더 어려워지는 4가지 평가 범주가 포함되어 있습니다. 가장 기본적인 단일 함수 호출(Simple)에서 시작하여 점차적으로 여러 함수 호출이 필요한 시나리오(Multiple), 여러 함수의 병렬 호출이 필요한 복잡한 시나리오(Parallel), 마지막으로 함수 호출이 필요한지 여부를 판단해야 하는 시나리오(Irrelevance)로 증가합니다. 이 네 가지 범주는 표 12.1에 표시된 것처럼 에이전트가 실제 응용 프로그램에서 접할 수 있는 다양한 도구 호출 시나리오를 다룹니다.

<div align="center">
<p>표 12.1 BFCL 벤치마크의 4가지 평가 범주</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-table-1.png" alt="" width="85%"/>
</div>

BFCL 평가 프로세스는 표준 벤치마크 테스트 절차를 따릅니다. 먼저 데이터 세트를 로드하고 평가 범주를 선택한 다음 에이전트를 실행하여 예측 결과를 얻은 다음 예측 결과를 AST(추상 구문 트리)로 구문 분석하고 마지막으로 AST 일치 알고리즘을 통해 예측이 올바른지 판단합니다. 전체 프로세스는 모든 테스트 샘플을 순회하여 궁극적으로 정확성과 같은 평가 지표를 계산하고 평가 보고서를 생성합니다. 전체 평가 프로세스는 그림 12.2에 나와 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-2.png" alt="" width="85%"/>
<p>그림 12.2 BFCL 평가 프로세스 다이어그램</p>
</div>

**(1) BFCL 데이터세트 구조**

BFCL 데이터 세트는 JSON 형식을 사용하며 각 테스트 샘플에는 다음 필드가 포함됩니다.

```json
{
  "id": "simple_001",
  "question": "What's the weather like in Beijing today?",
  "function": [
    {
      "name": "get_weather",
      "description": "Get the current weather for a location",
      "parameters": {
        "type": "object",
        "properties": {
          "location": {
            "type": "string",
            "description": "The city name"
          }
        },
        "required": ["location"]
      }
    }
  ],
  "ground_truth": [
    {
      "name": "get_weather",
      "arguments": {
        "location": "Beijing"
      }
    }
  ]
}
```

**주요 필드 설명:**

- `question`: 사용자의 자연어 요청
- `function`: 사용 가능한 기능 목록(기능 서명 및 설명 포함)
- `ground_truth`: 표준 답변(예상 함수 호출)

**(2) AST 매칭 설명**

BFCL은 핵심 평가 알고리즘으로 **AST Matching(Abstract Syntax Tree Matching)**을 사용하므로 아래의 평가 전략을 이해해 보겠습니다.

BFCL은 단순한 문자열 일치가 아닌 지능형 일치를 위해 AST(추상 구문 트리)를 사용합니다. AST 일치의 핵심 아이디어는 **함수 호출을 구문 트리로 구문 분석한 다음 트리 구조와 노드 값을 비교**하는 것입니다.

예측된 함수 호출 $P$와 표준 답변 $G$가 주어지면 AST 일치 함수는 다음과 같이 정의됩니다.

$$
\text{AST\_Match}(P, G) = \begin{cases}
1 & \text{if } \text{AST}(P) \equiv \text{AST}(G) \\
0 & \text{그렇지 않으면}
\end{사례}
$$

여기서 $\text{AST}(x)$는 추상 구문 트리로의 구문 분석 함수 호출을 나타내고, $\equiv$는 구문 트리 동등성을 나타냅니다.

두 구문 트리는 세 가지 핵심 조건을 충족하는 경우 동일합니다. 함수 이름은 완전히 동일해야 하며(정확히 일치), 매개변수 키-값 쌍 세트는 동일해야 하며(순서 무시), 각 매개변수 값은 의미상 동일합니다((e.g., ⟪P0⟫ is equivalent to ⟪P1⟫)). 특정 일치 프로세스에서 함수 이름 일치에는 정확한 문자열 일치가 필요합니다. 예를 들어 `get_weather` 및 `get_temperature`는 서로 다른 함수로 간주됩니다. 매개변수 일치는 지능적인 비교를 위해 AST를 사용하여 다양한 매개변수 순서(`f(a=1, b=2)`는 `f(b=2, a=1)`와 동일)를 허용하고 동일한 표현(`f(x=2+3)`은 `f(x=5)`과 동일)을 허용하며 다양한 문자열 표현(`f(s="hello")`은 `f(s='hello')`과 동일)도 허용합니다. 다기능 호출 시나리오의 경우 일치 알고리즘은 동일한 수의 함수 호출을 요구하며 각 함수 호출은 일치해야 하지만 호출 순서는 다를 수 있습니다(집합 일치 사용).

**AST 일치 예:**

```python
# Example 1: Different parameter order (match successful)
Prediction: get_weather(city="Beijing", unit="celsius")
Standard: get_weather(unit="celsius", city="Beijing")
Result: ✅ Match successful

# Example 2: Equivalent expression (match successful)
Prediction: calculate(x=2+3)
Standard: calculate(x=5)
Result: ✅ Match successful

# Example 3: Wrong function name (match failed)
Prediction: get_temperature(city="Beijing")
Standard: get_weather(city="Beijing")
Result: ❌ Match failed

# Example 4: Wrong parameter value (match failed)
Prediction: get_weather(city="Shanghai")
Standard: get_weather(city="Beijing")
Result: ❌ Match failed
```

**(3) BFCL 평가 지표**

BFCL은 다음 측정항목을 사용하여 상담원 성과를 평가합니다.

**1. 정확성**

정확도는 가장 핵심적인 측정항목으로, AST 일치에 성공한 샘플의 비율로 정의됩니다.

$$
\text{정확도} = \frac{1}{N} \sum_{i=1}^{N} \text{AST\_Match}(P_i, G_i)
$$

어디에:
- $N$은 총 샘플 개수입니다.
- $P_i$는 $i$번째 샘플의 예측 결과입니다.
- $G_i$는 $i$번째 샘플의 표준 답변입니다.
- $\text{AST\_Match}(P_i, G_i) \in \{0, 1\}$는 AST 일치 함수입니다.

**2. AST 일치율**

정확성과 동일하며 AST 일치 알고리즘 사용을 강조합니다.

$$
\text{AST 일치율} = \text{정확도}
$$

**3. 카테고리별 정확도**

각 범주 $c \in \{\text{simple}, \text{multiple}, \text{parallel}, \ldots\}$에 대해 해당 범주의 정확도를 계산합니다.

$$
\text{정확도}_c = \frac{1}{|D_c|} \sum_{i \in D_c} \text{AST\_Match}(P_i, G_i)
$$

여기서 $D_c$는 $c$ 카테고리의 샘플 세트이고, $|D_c|$는 해당 카테고리의 샘플 수입니다.

**4. 가중 정확도**

다양한 카테고리의 난이도 가중치를 고려하면 다음과 같습니다.

$$
\text{가중 정확도} = \sum_{c} w_c \cdot \text{정확도}_c
$$

여기서 $w_c$는 $\sum_c w_c = 1$을 만족하는 $c$ 범주의 가중치입니다.

**5. 오류율**

함수를 올바르게 호출하지 못한 샘플의 비율:

$$
\text{오류율} = 1 - \text{정확도} = \frac{1}{N} \sum_{i=1}^{N} (1 - \text{AST\_Match}(P_i, G_i))
$$

**측정항목 해석:**

- **정확도 = 1.0**: 모든 샘플이 완전히 정확함
- **정확도 = 0.8**: 샘플의 80%가 정확하고 샘플의 20%가 부정확함
- **정확도 = 0.0**: 모든 샘플이 정확하지 않습니다.

**범주 정확도 예:**

```python
# Assume evaluation results
simple_accuracy = 0.95      # Simple category: 95% correct
multiple_accuracy = 0.82    # Multiple category: 82% correct
parallel_accuracy = 0.68    # Parallel category: 68% correct

# Weighted accuracy (assuming equal weights)
weighted_accuracy = (0.95 + 0.82 + 0.68) / 3 = 0.817
```

**(4) BFCL 공식 평가 도구**

BFCL은 평가를 위한 공식 CLI 도구를 제공합니다.

```bash
# Install BFCL evaluation tool
pip install bfcl

# Run official evaluation
bfcl evaluate \
    --model-result-path ./results.json \
    --test-category simple_python
```

공식 평가 도구 사용의 장점: 공식 AST 매칭 알고리즘을 사용하고, 평가 결과가 리더보드와 완전히 일치하고, 모든 BFCL v4 카테고리를 지원하며, 자세한 평가 보고서를 자동으로 생성할 수 있습니다.


### 12.2.2 BFCL 데이터 세트 얻기

BFCL 데이터 세트는 다음 방법을 통해 얻을 수 있습니다.

**방법 1: 공식 GitHub 리포지토리에서 복제(권장)**

이는 완전한 데이터 세트와 실제 정보를 얻는 가장 신뢰할 수 있는 방법입니다.

```bash
# Clone BFCL repository
git clone https://github.com/ShishirPatil/gorilla.git temp_gorilla
cd temp_gorilla/berkeley-function-call-leaderboard

# View BFCL v4 dataset
ls bfcl_eval/data/
# Output: BFCL_v4_simple_python.json  BFCL_v4_multiple.json  BFCL_v4_parallel.json  ...

# View ground truth
ls bfcl_eval/data/possible_answer/
# Output: BFCL_v4_simple_python.json  BFCL_v4_multiple.json  ...
```

이 방법을 추천하는 이유: 완전한 정답(표준 답변)을 포함하고, 데이터 형식이 공식 평가 도구와 완전히 일치하며, 공식 평가 스크립트를 직접 사용할 수 있으며, BFCL v4 최신 버전을 지원합니다.

**방법 2: HelloAgents를 사용하여 공식 데이터 로드**

저장소를 복제한 후 HelloAgent를 사용하여 데이터를 로드합니다.

```python
from hello_agents.evaluation import BFCLDataset

# Load BFCL official data
dataset = BFCLDataset(
    bfcl_data_dir="./temp_gorilla/berkeley-function-call-leaderboard/bfcl_eval/data",
    category="simple_python"  # BFCL v4 category
)

# Load data (including test data and ground truth)
data = dataset.load()

print(f"✅ Loaded {len(data)} test samples")
print(f"✅ Loaded {len(dataset.ground_truth)} ground truth")
# Output:
# ✅ Loaded 400 test samples
# ✅ Loaded 400 ground truth
```

이 로더의 작동 원리는 먼저 `bfcl_eval/data/`에서 테스트 데이터를 로드한 다음 `bfcl_eval/data/possible_answer/`에서 Ground Truth를 로드하고, 다음으로 테스트 데이터와 Ground Truth를 자동으로 병합하고 마지막으로 원본 BFCL 데이터 형식을 보존하는 것입니다. BFCL v4 데이터세트 카테고리는 표 12.2에서 볼 수 있습니다.

<div align="center">
<p>표 12.2 BFCL 벤치마크의 4가지 평가 범주</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-table-2.png" alt="" width="85%"/>
</div>

코드를 통해 사용 가능한 카테고리를 볼 수도 있습니다.

```python
# Get all supported categories
categories = dataset.get_available_categories()
print(f"Supported categories: {categories}")
# Output: ['simple_python', 'simple_java', 'simple_javascript', 'multiple', ...]
```

### 12.2.3 HelloAgents에서 BFCL 평가 구현

이제 HelloAgents 프레임워크에서 BFCL 평가를 구현하는 방법을 살펴보겠습니다. 우리는 세 가지 사용 방법을 제공합니다:

**방법 1: BFCLEvaluationTool 사용(권장)**

이는 한 줄의 코드로 평가, 보고서 생성 및 공식 평가를 완료하는 가장 간단한 방법입니다.

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import BFCLEvaluationTool

# 1. Create agent to be evaluated
llm = HelloAgentsLLM()
agent = SimpleAgent(name="TestAgent", llm=llm)

# 2. Create BFCL evaluation tool
bfcl_tool = BFCLEvaluationTool()

# 3. Run evaluation (automatically complete all steps)
results = bfcl_tool.run(
    agent=agent,
    category="simple_python",  # Evaluation category
    max_samples=5              # Number of evaluation samples (0 means all)
)

# 4. View results
print(f"Accuracy: {results['overall_accuracy']:.2%}")
print(f"Correct: {results['correct_samples']}/{results['total_samples']}")
```

**실행 출력:**

```
============================================================
BFCL One-Click Evaluation
============================================================

Configuration:
   Evaluation category: simple_python
   Sample count: 5
   Agent: TestAgent

============================================================
Step 1: Run HelloAgents Evaluation
============================================================
✅ BFCL dataset loaded
   Data directory: ./temp_gorilla/berkeley-function-call-leaderboard/bfcl_eval/data
   Category: simple_python
   Sample count: 400
   Ground truth count: 400

🔧 Starting BFCL evaluation...
   Progress: 1/5
   Progress: 5/5

✅ BFCL evaluation complete
   Overall accuracy: 100.00%
   simple_python: 100.00% (5/5)

📊 Evaluation results:
   Accuracy: 100.00%
   Correct: 5/5

============================================================
Step 2: Export BFCL Format Results
============================================================
✅ BFCL format results exported
   Output file: ./evaluation_results/bfcl_official/BFCL_v4_simple_python_result.json

============================================================
Step 3: Run BFCL Official Evaluation
============================================================
✅ Result file copied to: ./result/Qwen_Qwen3-8B/BFCL_v4_simple_python_result.json

🔄 Running command: bfcl evaluate --model Qwen/Qwen3-8B --test-category simple_python --partial-eval

============================================================
BFCL Official Evaluation Results
============================================================
📊 Evaluation results summary:
Model,Overall Acc,simple_python
Qwen/Qwen3-8B,100.00,100.00

🎯 Final results:
   Accuracy: 100.00%
   Correct: 5/5

============================================================
Step 4: Generate Evaluation Report
============================================================
📄 Report generated: ./evaluation_reports/bfcl_report_20251011_005938.md

Accuracy: 100.00%
Correct: 5/5
```

**자동 생성된 마크다운 보고서:**

평가가 완료되면 다음을 포함한 자세한 Markdown 보고서가 자동으로 생성됩니다.

```markdown
# BFCL Evaluation Report
**Generated**: 2025-10-11 00:59:38

## 📊 Evaluation Overview

- **Agent**: TestAgent
- **Evaluation Category**: simple_python
- **Overall Accuracy**: 100.00%
- **Correct Samples**: 5/5

## 📈 Detailed Metrics

### Category Accuracy

- **simple_python**: 100.00% (5/5)

## 📝 Sample Details

| Sample ID | Question | Prediction | Ground Truth | Correct |
|-----------|----------|------------|--------------|---------|
| simple_python_0 | Find the area of a triangle... | [{'name': 'calculate_triangle_area'...}] | [{'function_name': {'base': [10]...}}] | ✅ |
| simple_python_1 | Calculate the factorial of 5... | [{'name': 'calculate_factorial'...}] | [{'function_name': {'number': [5]}}] | ✅ |
...

## 📊 Accuracy Visualization
Accuracy: ██████████████████████████████████████████████████ 100.00%

## 💡 Recommendations
- ✅ Excellent performance! Agent shows outstanding tool calling capabilities.
```

**방법 2: 원클릭 평가 스크립트 사용**

빠른 명령줄 평가에 적합합니다. 이 장에 포함된 코드 예제에서는 직접적인 명령줄 평가를 지원하는 `04_run_bfcl_evaluation.py`을 제공합니다.

```bash
# Run evaluation script
python chapter12/04_run_bfcl_evaluation.py --category simple_python --samples 10

# Specify model name (for BFCL official evaluation)
python examples/04_run_bfcl_evaluation.py \
    --category simple_python \
    --samples 10 \
    --model-name "Qwen/Qwen3-8B"
```

스크립트는 세 가지 매개변수를 지원합니다. `--category`는 평가 범주를 지정하고(기본값은 simple_python), `--samples`은 평가 샘플 수를 지정합니다(기본값은 5, 0은 모두를 의미함). `--model-name`는 BFCL 공식 평가를 위한 모델 이름을 지정합니다. (default Qwen/Qwen3-8B).

**방법 3: 데이터세트와 평가기를 직접 사용**

맞춤형 평가 프로세스가 필요한 시나리오에 적합합니다.

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.evaluation import BFCLDataset, BFCLEvaluator

# 1. Create agent
llm = HelloAgentsLLM()
agent = SimpleAgent(name="TestAgent", llm=llm)

# 2. Load dataset
dataset = BFCLDataset(
    bfcl_data_dir="./temp_gorilla/berkeley-function-call-leaderboard/bfcl_eval/data",
    category="simple_python"
)
data = dataset.load()

# 3. Create evaluator
evaluator = BFCLEvaluator(
    dataset=dataset,
    category="simple_python",
    evaluation_mode="ast"  # Use AST matching mode
)

# 4. Run evaluation
results = evaluator.evaluate(agent, max_samples=10)

# 5. View results
print(f"Accuracy: {results['overall_accuracy']:.2%}")
print(f"Correct: {results['correct_samples']}/{results['total_samples']}")

# 6. Export BFCL format results (optional)
evaluator.export_to_bfcl_format(
    results,
    output_path="./evaluation_results/my_results.json"
)
```

이 세 가지 방법을 통해 다양한 요구 사항에 따라 적절한 평가 방법을 선택할 수 있습니다. 에이전트 성능을 빠르게 이해하고 싶다면 BFCLEvaluationTool의 원클릭 평가를 사용하는 것이 가장 편리합니다. 일괄 평가 또는 CI/CD 파이프라인 통합이 필요한 경우 명령줄 스크립트를 사용하는 것이 더 적합합니다. 평가 프로세스를 심층적으로 사용자 정의하거나 자체 시스템에 통합해야 하는 경우 Dataset 및 Evaluator를 직접 사용하면 유연성이 극대화됩니다.




### 12.2.4 BFCL 공식 평가 도구 통합

이전에는 HelloAgents에 내장된 평가 기능을 사용하는 방법을 배웠습니다. 실제로 `BFCLEvaluationTool`에는 **BFCL 공식 평가 도구**가 자동으로 통합되어 있어 신뢰할 수 있고 비교 가능한 평가 결과를 얻을 수 있습니다.

전체 평가 프로세스에는 먼저 BFCL v4 데이터 세트의 로드 테스트 데이터, HelloAgents를 사용하여 평가를 실행하고 에이전트 예측 결과를 얻은 다음 결과를 BFCL 공식 형식(JSONL)으로 내보내고 마지막으로 공식 평가 스크립트를 사용하여 최종 점수를 계산하는 네 단계가 포함됩니다. 이 프로세스는 그림 12.3에 표시된 대로 평가 결과가 BFCL 순위표와 완전히 일치하는지 확인합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-3.png" alt="" width="85%"/>
<p>그림 12.3 BFCL 평가 프로세스를 로드하는 HelloAgents</p>
</div>

`BFCLEvaluationTool`을 사용하면 공식 평가가 **자동으로 실행됩니다**(기본적으로 활성화되어 있음):

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import BFCLEvaluationTool

# Create agent
llm = HelloAgentsLLM()
agent = SimpleAgent(name="TestAgent", llm=llm)

# Create evaluation tool
bfcl_tool = BFCLEvaluationTool()

# Run evaluation (automatically runs official evaluation)
results = bfcl_tool.run(
    agent=agent,
    category="simple_python",
    max_samples=5,
    # run_official_eval=True  # Default is True, can be omitted
    model_name="Qwen/Qwen3-8B"  # Optional, specify model name
)
```

도구는 전체 평가 프로세스를 자동으로 실행합니다. 먼저 HelloAgents 평가를 실행하여 예측 결과를 얻은 다음 결과를 BFCL 형식으로 내보내고 `evaluation_results/bfcl_official/` 디렉터리에 저장합니다. 다음으로 결과 파일을 `result/{model_name}/` 디렉터리에 복사하여 공식 평가 도구 요구 사항을 충족합니다. 그런 다음 BFCL 공식 평가 명령을 실행하여 점수를 계산하고 마지막으로 공식 평가 결과를 표시하고 Markdown 형식 평가 보고서를 생성합니다.

**공식 평가 결과 예시:**

```
============================================================
Step 3: Run BFCL Official Evaluation
============================================================

✅ Result file copied to:
   ./result/Qwen_Qwen3-8B/BFCL_v4_simple_python_result.json

🔄 Running command: bfcl evaluate --model Qwen/Qwen3-8B --test-category simple_python --partial-eval

============================================================
BFCL Official Evaluation Results
============================================================

📊 Evaluation results summary:
Model,Overall Acc,simple_python
Qwen/Qwen3-8B,100.00,100.00

🎯 Final results:
   Accuracy: 100.00%
   Correct: 5/5
```

평가 프로세스를 수동으로 제어하려면 자동 공식 평가를 비활성화할 수 있습니다.

```python
# Disable official evaluation
results = bfcl_tool.run(
    agent=agent,
    category="simple_python",
    max_samples=5,
    run_official_eval=False  # Disable official evaluation
)

# Then manually run official evaluation
import subprocess
subprocess.run([
    "bfcl", "evaluate",
    "--model", "Qwen/Qwen3-8B",
    "--test-category", "simple_python",
    "--partial-eval"
])
```

보고서를 수동으로 생성할 수도 있습니다.

```python
# Run evaluation
results = bfcl_tool.run(agent, category="simple_python", max_samples=5)

# Manually generate report
report = bfcl_tool.generate_report(
    results,
    output_file="./my_reports/custom_report.md"
)

# Print report content
print(report)
```



### 12.2.5 핵심 구성 요소 구현 세부 사항

이전 섹션에서는 BFCL 평가 도구를 사용하는 방법을 배웠습니다. 이제 HelloAgents 평가 시스템의 핵심 구성 요소가 어떻게 구현되는지 살펴보겠습니다. 이러한 구현 세부 사항을 이해하면 평가 시스템을 더 잘 사용하는 데 도움이 될 뿐만 아니라 필요에 따라 사용자 정의하고 확장할 수도 있습니다.

**(1) BFCLDataset: 데이터세트 로더**

BFCLDataset은 BFCL 데이터세트 로드 및 관리를 담당합니다.

````python
class BFCLDataset:
    """BFCL dataset loader"""

    def __init__(self, category: str = "simple", local_data_path: Optional[str] = None):
        self.category = category
        self.local_data_path = local_data_path
        self.data = []

    def load(self) -> List[Dict[str, Any]]:
        """Load dataset"""
        # Load from local first
        if self.local_data_path:
            return self._load_from_local()
        # Otherwise load from Hugging Face
        return self._load_from_huggingface()
````

BFCL의 데이터 세트는 공식 저장소에 있으므로 여기서 권장되는 접근 방식은 평가를 위해 로컬 복사본을 직접 복제하는 것입니다. 찾을 수 없는 경우에만 Hugging Face에서 로드됩니다.

**(2) BFCLEvaluator: 평가 실행자**

BFCLEvaluator는 평가 프로세스 실행을 담당합니다. 핵심은 전체 평가 프로세스를 조정하는 `evaluate()` 메서드입니다.

````python
class BFCLEvaluator:
    """BFCL evaluator"""

    def evaluate(self, agent: Any, max_samples: Optional[int] = None) -> Dict[str, Any]:
        """Execute evaluation"""
        results = []

        for item in self.dataset[:max_samples]:
            # 1. Construct prompt
            prompt = self._build_prompt(item)

            # 2. Call agent
            response = agent.run(prompt)

            # 3. Extract function calls
            predicted_calls = self._extract_function_calls(response)

            # 4. Compare with ground truth
            is_correct = self._compare_calls(predicted_calls, item["ground_truth"])

            results.append({
                "id": item["id"],
                "prediction": predicted_calls,
                "ground_truth": item["ground_truth"],
                "is_correct": is_correct
            })

        return {"results": results, "total_samples": len(results)}
````

이 평가자의 설계에는 세 가지 핵심 사항이 포함되어 있습니다. 첫 번째는 프롬프트 구성으로, 데이터 세트의 질문과 기능 정의를 상담원이 이해할 수 있는 프롬프트로 변환해야 합니다. 두 번째는 함수 호출 추출로, 에이전트의 응답에서 함수 호출을 추출하고 다양한 형식을 지원해야 합니다 (JSON, code blocks, etc.); 마지막으로 함수 호출 비교를 위해 추상 구문 트리를 사용하는 AST 일치가 있는데, 이는 단순한 문자열 일치보다 더 정확합니다.

함수 호출 추출의 구현을 살펴보겠습니다.

```python
def _extract_function_calls(self, response: str) -> List[Dict[str, Any]]:
    """Extract function calls from response

    Supports multiple formats:
    1. JSON format: {"name": "func", "arguments": {...}}
    2. Code block format: ```python\nfunc(arg1=val1)\n```
    3. Plain text format: func(arg1=val1)
    """
    calls = []

    # Try JSON parsing
    try:
        json_match = re.search(r'\{.*\}', response, re.DOTALL)
        if json_match:
            data = json.loads(json_match.group())
            if isinstance(data, dict) and "name" in data:
                calls.append(data)
            elif isinstance(data, list):
                calls.extend(data)
    except json.JSONDecodeError:
        pass

    # Try code block extraction
    code_blocks = re.findall(r'```(?:python)?\n(.*?)\n```', response, re.DOTALL)
    for code in code_blocks:
        # Parse Python function calls
        parsed_calls = self._parse_python_calls(code)
        calls.extend(parsed_calls)

    return calls
```

**(3) BFCLMetrics: 측정항목 계산기**

BFCLMetrics는 다양한 평가 지표 계산을 담당합니다.

````python
class BFCLMetrics:
    """BFCL metrics calculator"""

    def compute_metrics(self, results: List[Dict[str, Any]]) -> Dict[str, Any]:
        """Compute all metrics"""
        return {
            "accuracy": self._compute_accuracy(results),
            "ast_match_rate": self._compute_ast_match_rate(results),
            "parameter_accuracy": self._compute_parameter_accuracy(results),
            "f1_score": self._compute_f1_score(results),
            "category_statistics": self._compute_category_stats(results)
        }
````

**AST 일치 구현**:

AST 매칭은 BFCL 평가의 핵심 기술입니다. 이는 단순한 문자열 일치보다 더 지능적이며 의미상 동일한 함수 호출을 식별할 수 있습니다.

```python
def _ast_match(self, pred_call: Dict, true_call: Dict) -> bool:
    """Match function calls using AST

    Advantages of AST matching:
    1. Ignore parameter order: func(a=1, b=2) equivalent to func(b=2, a=1)
    2. Recognize equivalent expressions: 2+3 equivalent to 5
    3. Ignore whitespace and format differences
    """
    # 1. Function name must match exactly
    if pred_call.get("name") != true_call.get("name"):
        return False

    # 2. Convert parameters to AST nodes
    pred_args = self._args_to_ast(pred_call.get("arguments", {}))
    true_args = self._args_to_ast(true_call.get("arguments", {}))

    # 3. Compare AST nodes
    return ast.dump(pred_args) == ast.dump(true_args)

def _args_to_ast(self, args: Dict[str, Any]) -> ast.AST:
    """Convert parameter dictionary to AST node"""
    # Construct a virtual function call
    code = f"func({', '.join(f'{k}={repr(v)}' for k, v in args.items())})"
    tree = ast.parse(code)
    return tree.body[0].value  # Return Call node
```

**(4) 도구 캡슐화: BFCLEvaluationTool**

마지막으로 에이전트가 직접 호출할 수 있도록 이러한 구성 요소를 도구로 캡슐화합니다.

````python
class BFCLEvaluationTool(Tool):
    """BFCL evaluation tool"""

    def __init__(self, local_data_path: Optional[str] = None):
        super().__init__(
            name="bfcl_evaluation",
            description="Evaluate agent's tool calling capability"
        )
        self.dataset = None
        self.evaluator = None
        self.metrics_calculator = BFCLMetrics()

    def run(self, parameters: Dict[str, Any]) -> str:
        """Execute evaluation"""
        # 1. Load dataset
        self.dataset = BFCLDataset(...)

        # 2. Create evaluator
        self.evaluator = BFCLEvaluator(...)

        # 3. Run evaluation
        results = self.evaluator.evaluate(...)

        # 4. Calculate metrics
        metrics = self.metrics_calculator.compute_metrics(...)

        # 5. Return JSON results
        return json.dumps(results, ensure_ascii=False)
````

이 도구의 디자인은 세 가지 핵심 원칙을 따릅니다. 먼저 도구 기본 클래스를 상속하여 HelloAgents의 도구 사양을 따르며 프레임워크와의 원활한 통합을 보장합니다. 둘째, 엄격한 매개변수 검증을 수행하고 필수 매개변수를 확인하며 친숙한 오류 프롬프트를 제공하여 사용자 경험을 개선합니다. 마지막으로 결과 형식을 지정하고 쉽게 구문 분석하고 표시할 수 있도록 JSON 문자열을 반환합니다. 이러한 모듈식 설계를 통해 우리는 사용하기 쉽고 유연한 평가 시스템을 구현했습니다. 사용자는 높은 수준의 도구 인터페이스를 직접 사용하여 신속하게 평가를 완료하거나 특별한 요구 사항을 충족하기 위한 사용자 정의를 위해 낮은 수준의 구성 요소를 자세히 살펴볼 수 있습니다.

### 12.2.6 확장 및 최적화 권장 사항

이전 학습을 통해 BFCL 평가를 위해 HelloAgent를 사용하는 방법을 마스터했습니다. 현재 구현은 SimpleAgent를 기반으로 한 간단한 재생산이며 주로 기본 BFCL 평가 기능을 완료한다는 점에 유의해야 합니다. 실제 응용 프로그램에서 BFCL 벤치마크에는 여러 난이도 수준과 시나리오가 포함되어 있습니다. 리더보드에서 더 높은 점수를 얻으려면 추가적인 최적화와 확장이 필요합니다.

**(1) 현재 구현의 한계**

현재 SimpleAgent 구현은 주로 평가 프로세스 구축에 중점을 두고 있으며 도구 호출 기능을 개선할 여지가 있습니다. SimpleAgent는 LLM이 적극적으로 학습하고 사용하는 데 필요한 맞춤형 도구 호출 형식 `[TOOL_CALL:tool_name:parameters]`을 사용합니다. 복잡한 시나리오에서는 성능이 기본 함수 호출을 사용하는 에이전트와 일치하지 않을 수 있습니다. 또한 현재는 simple_python과 같은 기본 카테고리만 테스트합니다. 다중, 병렬, 무관성과 같은 보다 복잡한 시나리오의 경우에도 대상 최적화가 필요합니다.

**(2) BFCL 점수 향상 방향**

BFCL 평가 점수를 더욱 향상시키려면 다음 방향부터 시작할 수 있습니다. 첫 번째는 에이전트의 도구 호출 기능을 최적화하는 것입니다. 기본 함수 호출 (like GPT-4, Claude, etc.)을 지원하는 LLM 사용을 고려하거나 LLM이 도구 호출 형식을 더 잘 이해할 수 있도록 프롬프트를 개선하세요. 두 번째는 도구 라이브러리 확장입니다. BFCL 테스트에는 다양한 유형의 기능이 포함되므로 테스트 데이터 세트 특성을 기반으로 공통 도구 유형을 사전 구현하여 에이전트의 도구 적용 범위를 향상시킬 수 있습니다. 세 번째는 다양한 난이도에 대한 다양한 전략을 설계하는 것입니다. 예를 들어 여러 시나리오에서 상담원은 다단계 도구 호출 순서를 계획해야 하고, 병렬 시나리오에서는 병렬로 실행할 수 있는 도구 호출을 식별해야 하며, 관련 없는 시나리오에서는 도구 호출이 실제로 필요한지 판단해야 합니다.

**(3) 실천 권장사항**

BFCL에서 더 나은 결과를 얻고자 하는 개발자에게는 다음과 같은 실천 전략을 권장합니다. 먼저 간단한 범주에서 시작하여 기본 단일 함수 호출이 안정적으로 작동하는지 확인합니다. 이는 후속 최적화의 기초입니다. 그런 다음 다중, 병렬, 실패 사례 분석, 에이전트의 약점 찾기와 같은 더 복잡한 범주를 점진적으로 테스트합니다. 최적화하는 동안 BFCL 리더보드에서 높은 점수를 받은 모델을 참조하고 해당 모델의 설계 아이디어와 최적화 기술을 배울 수 있습니다. 한편, 검증을 위해 공식 평가 도구를 사용하여 최적화된 결과가 리더보드 표준과 일치하는지 확인하는 것이 좋습니다.

평가 중 추가 처리를 위한 몇 가지 제안 사항은 다음과 같습니다.

**1. 점진적 평가**

작은 샘플부터 시작하여 점차적으로 샘플 수를 늘립니다.

```python
# Step 1: Quick test (5 samples)
results_quick = bfcl_tool.run(agent, category="simple_python", max_samples=5)

# Step 2: Medium-scale test (50 samples)
if results_quick['overall_accuracy'] > 0.8:
    results_medium = bfcl_tool.run(agent, category="simple_python", max_samples=50)

# Step 3: Full evaluation (all samples)
if results_medium['overall_accuracy'] > 0.8:
    results_full = bfcl_tool.run(agent, category="simple_python", max_samples=0)
```

**2. 다중 카테고리 평가**

다양한 난이도의 작업을 평가합니다.

```python
categories = ["simple_python", "multiple", "parallel", "irrelevance"]

for category in categories:
    print(f"\nEvaluating category: {category}")
    results = bfcl_tool.run(agent, category=category, max_samples=10)
    print(f"Accuracy: {results['overall_accuracy']:.2%}")
```

**3. 비교평가**

다양한 구성의 에이전트를 비교합니다.

```python
# Configuration 1: Default prompt
agent1 = SimpleAgent(name="Agent-Default", llm=llm)
results1 = bfcl_tool.run(agent1, category="simple_python", max_samples=10)

# Configuration 2: Optimized prompt
agent2 = SimpleAgent(name="Agent-Optimized", llm=llm)
# ... Set optimized system prompt ...
results2 = bfcl_tool.run(agent2, category="simple_python", max_samples=10)

# Compare results
print(f"Default configuration accuracy: {results1['overall_accuracy']:.2%}")
print(f"Optimized configuration accuracy: {results2['overall_accuracy']:.2%}")
```

평가 결과가 좋으면 BFCL 공식 리더보드에 제출해 보세요!

**1단계: 제출 자료 준비**

1. 모델 설명 문서
2. 평가 결과 파일(전 항목)
3. 모델 접근 방법(API 또는 오픈소스 링크)

**2단계: GitHub에 제출**

BFCL 공식 저장소를 방문하여 지침에 따라 Pull Request를 제출하세요.

- 저장소: https://github.com/ShishirPatil/gorilla
- 제출안내 : `CONTRIBUTING.md` 참조

**3단계: 검토 대기**

BFCL 팀은 귀하의 제출 내용을 검토하고 결과의 정확성을 확인할 것입니다. 승인 후 모델이 공식 리더보드에 표시됩니다!



## 12.3 GAIA: 일반 AI 보조 기능 평가

### 12.3.1 GAIA 벤치마크 소개

GAIA(General AI Assistants)는 Meta AI와 Hugging Face가 공동으로 출시한 평가 벤치마크로, AI 도우미의 **일반 역량**<sup>[2]</sup>을 평가하는 데 중점을 두고 있습니다. 도구 호출에 중점을 두는 BFCL과 달리 GAIA는 실제 작업에서 에이전트의 포괄적인 성능을 평가합니다.

GAIA의 디자인 철학은 다음과 같습니다. **실제 문제에는 종종 여러 기능의 포괄적인 적용이 필요합니다**. 뛰어난 AI 비서는 도구를 호출할 뿐만 아니라 다음도 수행해야 합니다.

- **다단계 추론**: 복잡한 문제를 여러 하위 문제로 분해합니다.
- **지식 응용**: 내장된 지식과 외부 지식 베이스를 활용합니다.
- **다중 모드 이해**: 텍스트, 이미지, 파일과 같은 다중 입력 처리
- **웹 브라우징**: 인터넷에서 최신 정보를 얻습니다.
- **파일 작업**: 다양한 형식의 파일을 읽고 처리합니다.

**(1) GAIA 데이터세트 구조**

GAIA의 평가 철학을 이해한 후, GAIA 데이터 세트의 구체적인 구조를 살펴보겠습니다. GAIA에는 신중하게 설계된 466개의 실제 문제가 포함되어 있습니다. 이러한 문제는 표 12.3과 같이 간단한 제로 단계 추론 작업부터 다단계 복잡한 추론이 필요한 어려운 작업까지 복잡성과 필요한 추론 단계에 따라 세 가지 난이도로 분류되며 에이전트가 실제 응용 프로그램에서 접할 수 있는 다양한 시나리오를 포괄적으로 다룹니다.

<div align="center">
<p>표 12.3 GAIA 데이터 세트 난이도 분포</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-table-3.png" alt="" width="85%"/>
</div>

GAIA 데이터 세트 샘플 예는 아래 코드 스니펫을 참조하세요.

```json
{
  "task_id": "gaia_001",
  "Question": "What is the total population of the top 3 most populous cities in California?",
  "Level": 2,
  "Final answer": "12847521",
  "file_name": "",
  "file_path": "",
  "Annotator Metadata": {
    "Steps": [
      "Search for most populous cities in California",
      "Get population data for top 3 cities",
      "Sum the populations"
    ],
    "Number of steps": 3,
    "How long did this take?": "5 minutes",
    "Tools": ["web_search", "calculator"]
  }
}
```

**주요 필드 설명:**
- `Question`: 질문 설명
- `Level`: 난이도(1~3)
- `Final answer`: 표준 답변(숫자, 텍스트, 파일일 수 있음)
- `file_name/file_path`: 첨부파일(있는 경우)
- `Annotator Metadata`: Annotator (reasoning steps, required tools, etc.)에서 제공하는 메타데이터

**(2) 준완전 일치 소개**

GAIA는 GAIA가 공식적으로 정의한 평가 표준인 **Quasi Exact Match** 평가 알고리즘을 사용합니다. 이 알고리즘의 핵심 아이디어는 **먼저 답변을 정규화한 다음 정확한 일치를 수행**하는 것입니다.

예측 답변 $A_{\text{pred}}$ 및 표준 답변 $A_{\text{true}}$가 주어지면 준정확 일치 함수는 다음과 같이 정의됩니다.

$$
\text{Quasi\_Exact\_Match}(A_{\text{pred}}, A_{\text{true}}) = \begin{cases}
1 & \text{if } \mathcal{N}(A_{\text{pred}}) = \mathcal{N}(A_{\text{true}}) \\
0 & \text{그렇지 않으면}
\end{사례}
$$

여기서 $\mathcal{N}(\cdot)$는 정규화 함수로, 답변 유형에 따라 다른 규칙을 적용합니다.

정규화 함수는 답변 유형에 따라 다른 규칙을 적용합니다. 숫자 유형의 경우 쉼표 구분 기호(`1,000` → `1000`) 및 단위 기호(`$100` → `100`, `50%` → `50`)를 제거합니다. 예를 들어 `"$1,234.56"`는 `"1234.56"`로 정규화됩니다. 문자열 유형의 경우 소문자로 변환(`"Apple"` → `"apple"`), 관사 제거(`"the apple"` → `"apple"`), 추가 공백 제거(`"hello  world"` → `"hello world"`), 후행 구두점 제거(`"hello."` → `"hello"`) 등이 있습니다. `"The United States"`은 `"united states"`로 정규화됩니다. 목록 유형의 경우 요소를 쉼표로 분할하고, 각 요소에 문자열 정규화를 적용하고, 알파벳순으로 정렬한 다음 다시 결합합니다. 예를 들어 `"Paris, London, Berlin"`은 `"berlin,london,paris"`로 정규화됩니다.

**정규화 예:**

```python
# Numeric answer
Original answer: "$1,234.56"
Normalized: "1234.56"

# String answer
Original answer: "The United States of America"
Normalized: "united states of america"

# List answer
Original answer: "Paris, London, Berlin"
Normalized: "berlin, london, paris"
```

**(3) GAIA 평가 지표**

GAIA는 다음 측정항목을 사용하여 상담사 성과를 평가합니다.

**1. 정확한 일치율**

완전 일치율은 GAIA의 핵심 측정항목으로, 준정확 일치에 성공한 샘플의 비율로 정의됩니다.

$$
\text{정확한 일치율} = \frac{1}{N} \sum_{i=1}^{N} \text{Quasi\_Exact\_Match}(A_{\text{pred},i}, A_{\text{true},i})
$$

어디에:
- $N$은 총 샘플 개수입니다.
- $A_{\text{pred},i}$는 $i$번째 샘플의 예상 답변입니다.
- $A_{\text{true},i}$는 $i$번째 샘플의 표준 답변입니다.
- $\text{Quasi\_Exact\_Match}(\cdot, \cdot) \in \{0, 1\}$는 준정확 일치 함수입니다.

**2. 레벨별 정확도**

각 난이도 $\ell \in \{1, 2, 3\}$에 대해 해당 레벨의 정확도를 계산합니다.

$$
\text{정확도}_\ell = \frac{1}{|D_\ell|} \sum_{i \in D_\ell} \text{Quasi\_Exact\_Match}(A_{\text{pred},i}, A_{\text{true},i})
$$

여기서 $D_\ell$은 난이도 $\ell$의 샘플 세트이고, $|D_\ell|$은 해당 레벨의 샘플 수입니다.

**3. 난이도 진행 드롭률**

난이도가 증가함에 따라 에이전트의 성능 저하를 측정합니다.

$$
\text{드롭율}_{\ell \to \ell+1} = \frac{\text{정확도}_\ell - \text{정확도}_{\ell+1}}{\text{정확도}_\ell}
$$

- $\text{Drop Rate}_{1 \to 2}$: 1레벨에서 2레벨까지 드랍율
- $\text{Drop Rate}_{2 \to 3}$: 2레벨에서 3레벨까지 드랍율

**4. 평균 추론 단계**

에이전트가 작업을 완료하는 데 필요한 평균 단계 수를 평가합니다.

$$
\text{평균 단계} = \frac{1}{N_{\text{올바른}}} \sum_{i \in \text{올바른}} \text{단계}_i
$$

여기서 $N_{\text{corright}}$는 정답 샘플 수이고, $\text{steps}_i$는 $i$번째 샘플에 대한 추론 단계 수입니다.

**측정항목 해석:**

- **정확한 일치율 = 1.0**: 모든 샘플이 완전히 정확함
- **정확한 일치율 = 0.5**: 샘플의 50%가 정확하고 샘플의 50%가 부정확함
- **드롭율 = 0.3**: 난이도 증가로 인해 정확도가 30% 감소합니다.
- **드롭율 = 0.0**: 난이도 증가는 정확도에 영향을 미치지 않습니다(이상적인 경우).

**평가 예시:**

10개의 샘플을 평가했다고 가정하면 결과는 표 12.4에서 참조할 수 있습니다.

<div align="center">
<p>표 12.4 GAIA 데이터세트 난이도 분포</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-table-4.png" alt="" width="85%"/>
</div>

이 사례에 대한 측정항목을 계산하려면 아래 Python 스크립트를 참조하세요.

```python
# 1. Exact match rate
total_samples = 10
correct_samples = 7  # Samples 1,2,3,5,6,8,9
exact_match_rate = correct_samples / total_samples = 0.70  # 70%

# 2. Level-wise accuracy
level_1_correct = 3  # Samples 1,2,3
level_1_total = 3
level_1_accuracy = 3 / 3 = 1.00  # 100%

level_2_correct = 2  # Samples 5,6
level_2_total = 3
level_2_accuracy = 2 / 3 = 0.67  # 67%

level_3_correct = 2  # Samples 8,9
level_3_total = 4
level_3_accuracy = 2 / 4 = 0.50  # 50%

# 3. Difficulty progression drop rate
drop_rate_1_to_2 = (1.00 - 0.67) / 1.00 = 0.33  # 33%
drop_rate_2_to_3 = (0.67 - 0.50) / 0.67 = 0.25  # 25%

print(f"Exact match rate: {exact_match_rate:.2%}")  # 70.00%
print(f"Level 1 accuracy: {level_1_accuracy:.2%}")  # 100.00%
print(f"Level 2 accuracy: {level_2_accuracy:.2%}")  # 66.67%
print(f"Level 3 accuracy: {level_3_accuracy:.2%}")  # 50.00%
print(f"Level 1→2 drop rate: {drop_rate_1_to_2:.2%}")  # 33.00%
print(f"Level 2→3 drop rate: {drop_rate_2_to_3:.2%}")  # 25.00%
```

**결과 분석:**

- **전체 성능**: 일치율 70%, 우수한 성능
- **난이도 감도**: 레벨 1에서 레벨 2로 33% 감소하여 중간 난이도 작업에서 상당한 성능 저하를 나타냅니다.
- **능력 경계**: 레벨 3 정확도는 50%로, 복잡한 작업에서 개선의 여지가 있음을 나타냅니다.

드롭률이 높을수록 복잡한 작업을 처리할 때 에이전트의 성능 저하가 더욱 뚜렷해집니다.

**(4) GAIA 공식 시스템 프롬프트**

GAIA에서는 모델 출력이 평가 형식을 준수하는지 확인하기 위해 특정 시스템 프롬프트를 사용해야 합니다.

```python
GAIA_SYSTEM_PROMPT = """You are a general AI assistant. I will ask you a question. Report your thoughts, and finish your answer with the following template: FINAL ANSWER: [YOUR FINAL ANSWER].

YOUR FINAL ANSWER should be a number OR as few words as possible OR a comma separated list of numbers and/or strings.

If you are asked for a number, don't use comma to write your number neither use units such as $ or percent sign unless specified otherwise.

If you are asked for a string, don't use articles, neither abbreviations (e.g. for cities), and write the digits in plain text unless specified otherwise.

If you are asked for a comma separated list, apply the above rules depending of whether the element to be put in the list is a number or a string."""
```

GAIA에는 답변 형식에 대한 엄격한 요구사항이 있습니다. 답변은 `FINAL ANSWER: [answer]` 형식으로 제공되어야 합니다. 숫자 답변에는 쉼표 구분 기호와 단위 기호를 사용하지 마세요. 문자열 답변의 경우 관사와 약어를 사용하지 마세요. 목록 답변의 경우 쉼표로 구분하고 알파벳순으로 정렬하세요.

### 12.3.2 GAIA 데이터세트 얻기

**중요 사항**: GAIA는 **Gated Dataset**이므로 HuggingFace에 대한 액세스 권한을 얻으려면 사전 신청이 필요합니다.

**1단계: 접근 권한 신청**

1. https://huggingface.co/datasets/gaia-benchmark/GAIA 방문
2. '액세스 요청' 버튼을 클릭하세요.
3. 신청서 작성(보통 몇 초 안에 승인됨)
4. HuggingFace 토큰 받기: https://huggingface.co/settings/tokens

**2단계: 환경 변수 구성**

HuggingFace 토큰을 `.env` 파일에 추가하세요:

```bash
# HuggingFace API configuration
HF_TOKEN=hf_your_token_here
```

**방법 1: HelloAgents를 이용한 자동 다운로드(권장)**

HelloAgents는 GAIA 데이터 세트 다운로드 및 캐싱을 자동으로 처리합니다.

```python
from hello_agents.evaluation import GAIADataset
import os

# Ensure HF_TOKEN is set, this line is not needed if .env is configured
os.environ["HF_TOKEN"] = "hf_your_token_here"

# Automatically download to ./data/gaia/
dataset = GAIADataset(
    dataset_name="gaia-benchmark/GAIA",
    split="validation",  # or "test"
    level=1  # Optional: 1, 2, 3, None(all)
)
items = dataset.load()

print(f"Loaded {len(items)} test samples")
# Output: Loaded 53 test samples (Level 1)
```

**작동 원리**:

- 처음 실행 시 `snapshot_download`를 사용하여 전체 데이터세트를 `./data/gaia/`에 다운로드합니다.
- 데이터세트에는 114개의 파일이 포함되어 있습니다. (questions, images, PDFs, etc.)
- 후속 사용은 로컬에서 직접 로드되며 매우 빠릅니다.

**데이터세트 디렉터리 구조**:
```
./data/gaia/
├── 2023/
│   ├── validation/
│   │   ├── metadata.jsonl  (165 questions)
│   │   ├── *.png, *.pdf, *.csv, *.xlsx  (attachment files)
│   └── test/
│       ├── metadata.jsonl  (301 questions)
│       └── ... (attachment files)
├── GAIA.py
└── README.md
```

**방법 2: 수동 다운로드**

데이터세트를 수동으로 다운로드하려면 다음 안내를 따르세요.

```python
from huggingface_hub import snapshot_download
import os

# Set Token
os.environ["HF_TOKEN"] = "hf_your_token_here"

# Download dataset
snapshot_download(
    repo_id="gaia-benchmark/GAIA",
    repo_type="dataset",
    local_dir="./data/gaia",
    token=os.getenv("HF_TOKEN")
)
```

**데이터세트 통계 보기**:

```python
# View dataset statistics
stats = dataset.get_statistics()
print(f"Total samples: {stats['total_samples']}")
print(f"Level distribution: {stats['level_distribution']}")
# Output:
# Total samples: 165
# Level distribution: {1: 53, 2: 62, 3: 50}
```


### 12.3.3 HelloAgents에서 GAIA 평가 구현

BFCL과 유사하게 두 가지 평가 방법을 제공하며 **방법 1**을 권장합니다.

**방법 1: GAIAEvaluationTool을 사용한 원클릭 평가**

이는 데이터 세트 다운로드, 평가 실행, 결과 내보내기 및 보고서 생성을 자동으로 완료하는 가장 간단한 방법입니다.

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import GAIAEvaluationTool

# GAIA official system prompt (from paper)
GAIA_SYSTEM_PROMPT = """You are a general AI assistant. I will ask you a question. Report your thoughts, and finish your answer with the following template: FINAL ANSWER: [YOUR FINAL ANSWER].

YOUR FINAL ANSWER should be a number OR as few words as possible OR a comma separated list of numbers and/or strings.

If you are asked for a number, don't use comma to write your number neither use units such as $ or percent sign unless specified otherwise.

If you are asked for a string, don't use articles, neither abbreviations (e.g. for cities), and write the digits in plain text unless specified otherwise.

If you are asked for a comma separated list, apply the above rules depending of whether the element to be put in the list is a number or a string."""

# 1. Create agent (using GAIA official system prompt)
llm = HelloAgentsLLM()
agent = SimpleAgent(
    name="TestAgent",
    llm=llm,
    system_prompt=GAIA_SYSTEM_PROMPT  # Key: Use GAIA official prompt
)

# 2. Create GAIA evaluation tool
gaia_tool = GAIAEvaluationTool()

# 3. One-click run evaluation
results = gaia_tool.run(
    agent=agent,
    level=1,  # Level 1: Simple tasks
    max_samples=5,  # Evaluate 5 samples
    export_results=True,  # Export GAIA format results
    generate_report=True  # Generate evaluation report
)

# 4. View results
print(f"Exact match rate: {results['exact_match_rate']:.2%}")
print(f"Partial match rate: {results['partial_match_rate']:.2%}")
print(f"Correct: {results['exact_matches']}/{results['total_samples']}")
```

**실행 결과:**

```
============================================================
GAIA One-Click Evaluation
============================================================

Configuration:
   Agent: TestAgent
   Difficulty level: 1
   Sample count: 5

============================================================
Step 1: Run HelloAgents Evaluation
============================================================
   Downloading from HuggingFace: gaia-benchmark/GAIA
   📥 Downloading GAIA dataset...
   ✓ Dataset download complete
   ✓ Loaded 165 samples
✅ GAIA dataset loaded
   Data source: gaia-benchmark/GAIA
   Split: validation
   Level: 1
   Sample count: 53

🌟 Starting GAIA evaluation...
   Sample count: 5
   Progress: 5/5
✅ GAIA evaluation complete
   Exact match rate: 80.00%
   Partial match rate: 80.00%

============================================================
Step 2: Export GAIA Format Results
============================================================
✅ GAIA format results exported
   Output file: evaluation_results\gaia_official\gaia_level1_result_20251011_012648.jsonl
   Sample count: 5
   Includes reasoning trace: True
📄 Submission guide generated: evaluation_results\gaia_official\SUBMISSION_GUIDE_20251011_012648.md

============================================================
Step 3: Generate Evaluation Report
============================================================
📄 Report generated: evaluation_reports\gaia_report_20251011_012648.md

============================================================
🎯 Final Results
============================================================
   Exact match rate: 80.00%
   Partial match rate: 80.00%
   Correct: 4/5
```

평가가 완료되면 세 가지 유형의 파일이 자동으로 생성됩니다. 첫 번째는 JSONL 형식(한 줄에 하나의 JSON 개체)을 사용하는 GAIA 형식 결과 파일(`evaluation_results/gaia_official/gaia_level1_result_*.jsonl`)이며 GAIA 리더보드에 제출하는 데 직접 사용할 수 있습니다. 두 번째는 자세한 제출 단계, 결과 파일 형식 설명 및 메모가 포함된 제출 가이드 파일(`evaluation_results/gaia_official/SUBMISSION_GUIDE_*.md`)입니다. 마지막으로 평가 보고서(`evaluation_reports/gaia_report_*.md`)가 있는데, 평가 결과 요약, 세부 지표, 샘플 세부 정보, 시각화 차트가 포함되어 있습니다.

**참고**: 생성된 평가 결과가 만족스럽지 못한 경우((e.g., low accuracy)) 이는 정상적인 현상입니다. 레벨 1은 1단계 추론 작업이지만 에이전트는 질문에 올바르게 답하기 위해 여전히 도구 호출 기능 (like search engine, calculator, etc.)이 필요합니다. 현재 SimpleAgent는 평가 프로세스를 시연하는 데 주로 사용되며 도구 호출 기능을 개선할 여지가 있습니다.

**방법 2: 데이터세트 + 평가기 사용(유연한 사용자 정의)**

보다 세밀한 제어가 필요한 경우 하위 수준 구성 요소를 직접 사용할 수 있습니다.

```python
from hello_agents.evaluation import GAIADataset, GAIAEvaluator

# 1. Load dataset
dataset = GAIADataset(level=1)
items = dataset.load()
print(f"Loaded {len(items)} samples")

# 2. Create evaluator
evaluator = GAIAEvaluator(dataset=dataset, level=1)

# 3. Run evaluation
results = evaluator.evaluate(agent, max_samples=5)

# 4. Export GAIA format results
evaluator.export_to_gaia_format(
    results,
    "gaia_results.jsonl",
    include_reasoning=True
)
```

생성된 평가 보고서(`gaia_report_*.md`)는 아래 파일을 참조할 수 있습니다.

```markdown
# GAIA Evaluation Report

**Generated**: 2025-10-11 01:26:48

## 📊 Evaluation Overview

- **Agent**: TestAgent
- **Difficulty Level**: 1
- **Total Samples**: 2
- **Exact Matches**: 1
- **Partial Matches**: 1
- **Exact Match Rate**: 50.00%
- **Partial Match Rate**: 50.00%

## 📈 Detailed Metrics

### Level-wise Accuracy

- **Level 1**: 50.00% exact / 50.00% partial (1/2)

## 📝 Sample Details (First 10)

| Task ID | Level | Predicted Answer | Correct Answer | Exact Match | Partial Match |
|---------|-------|------------------|----------------|-------------|---------------|
| e1fc63a2-da7a-432f-be78-7c4a95598703 | 1 | 24000 | 17 | ❌ | ❌ |
| 8e867cd7-cff9-4e6c-867a-ff5ddc2550be | 1 | 3 | 3 | ✅ | ✅ |

## 📊 Accuracy Visualization

Exact match: █████████████████████████░░░░░░░░░░░░░░░░░░░░░░░░░ 50.00%
Partial match: █████████████████████████░░░░░░░░░░░░░░░░░░░░░░░░░ 50.00%


## 💡 Recommendations

- ⚠️ Average performance, needs improvement.
- 💡 Suggest checking tool usage and multi-step reasoning capabilities.
```

**생성된 GAIA 형식 결과(`gaia_level1_result_*.jsonl`):**

```json
{"task_id": "e1fc63a2-da7a-432f-be78-7c4a95598703", "model_answer": "24000", "reasoning_trace": "24000"}
{"task_id": "8e867cd7-cff9-4e6c-867a-ff5ddc2550be", "model_answer": "3", "reasoning_trace": "3"}
```

### 12.3.4 GAIA 공식 리더보드에 결과 제출

GAIAEvaluationTool을 사용하여 평가를 실행하면 제출에 필요한 파일과 자세한 제출 지침이 `evaluation_results/gaia_official/` 디렉터리에 생성됩니다.

1. **GAIA 형식 결과 파일**: `gaia_level1_result_*.jsonl`
   ```json
   {"task_id": "xxx", "model_answer": "answer", "reasoning_trace": "reasoning process"}
   {"task_id": "yyy", "model_answer": "answer", "reasoning_trace": "reasoning process"}
   ```

2. **제출 가이드 파일**: `SUBMISSION_GUIDE_*.md`

전체 제출 가이드가 포함된 자동으로 생성된 `SUBMISSION_GUIDE_*.md` 파일을 엽니다.

구체적으로 브라우저를 열고 다음을 방문하세요.
```
https://huggingface.co/spaces/gaia-benchmark/leaderboard
```

그림 12.4와 같이 제출 양식에 정보를 입력합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-4.png" alt="" width="85%"/>
<p>그림 12.4 BFCL 평가 프로세스 다이어그램</p>
</div>

제출하기 전에 생성된 JSONL 파일을 수동으로 확인할 수 있습니다.

```python
import json

# Read result file
with open("evaluation_results/gaia_official/gaia_level1_result_*.jsonl", "r") as f:
    for line in f:
        result = json.loads(line)
        print(f"Task ID: {result['task_id']}")
        print(f"Answer: {result['model_answer']}")
        print(f"Reasoning: {result['reasoning_trace']}")
        print("-" * 50)
```

### 12.3.5 핵심 구성 요소 구현 세부 사항

GAIA 평가 시스템 구현은 BFCL과 유사하지만 일반적인 능력 평가를 위한 몇 가지 특별한 설계가 있습니다.

**(1) GAIADataset: 다중 모드 데이터 로더**

GAIA 데이터세트의 특별한 특징은 다중 모드 데이터 (text, files, images, etc.)를 포함한다는 것입니다.

````python
class GAIADataset:
    """GAIA dataset loader

    Supports loading GAIA dataset from HuggingFace (gated dataset)
    """

    def __init__(
        self,
        level: Optional[int] = None,
        split: str = "validation",
        local_data_dir: Optional[str] = None
    ):
        self.level = level
        self.split = split
        self.local_data_dir = local_data_dir or "./data/gaia"
        self.data = []

    def load(self) -> List[Dict[str, Any]]:
        """Load dataset"""
        # Download from HuggingFace
        items = self._load_from_huggingface()

        # Filter by level
        if self.level:
            items = [item for item in items if item.get("level") == self.level]

        self.data = items
        return items

    def _load_from_huggingface(self) -> List[Dict[str, Any]]:
        """Download GAIA dataset from HuggingFace"""
        from huggingface_hub import snapshot_download
        import json

        # Download dataset
        repo_id = "gaia-benchmark/GAIA"
        local_dir = snapshot_download(
            repo_id=repo_id,
            repo_type="dataset",
            local_dir=self.local_data_dir,
            local_dir_use_symlinks=False
        )

        # Load JSONL file
        data_file = Path(local_dir) / "2023" / self.split / "metadata.jsonl"
        items = []
        with open(data_file, 'r', encoding='utf-8') as f:
            for line in f:
                item = json.loads(line)
                items.append(self._standardize_item(item))

        return items
````

**(2) GAIAEvaluator: GAIA 공식 평가 알고리즘 구현**

GAIA 평가에서는 **Quasi Exact Match** 알고리즘을 사용하며 특수 답변 정규화 및 일치 논리가 필요합니다.

````python
class GAIAEvaluator:
    """GAIA evaluator

    Implements GAIA official Quasi Exact Match evaluation algorithm
    """

    def evaluate(self, agent: Any, max_samples: Optional[int] = None) -> Dict[str, Any]:
        """Execute evaluation"""
        dataset_items = self.dataset.load()

        if max_samples:
            dataset_items = dataset_items[:max_samples]

        results = []
        for i, item in enumerate(dataset_items, 1):
            # 1. Construct prompt
            prompt = self._build_prompt(item["question"], item)

            # 2. Call agent
            response = agent.run(prompt)

            # 3. Extract answer (GAIA format: FINAL ANSWER: [answer])
            predicted_answer = self._extract_answer(response)

            # 4. Normalize answer (GAIA official rules)
            normalized_pred = self._normalize_answer(predicted_answer)
            normalized_truth = self._normalize_answer(item["final_answer"])

            # 5. Quasi exact match
            exact_match = (normalized_pred == normalized_truth)

            results.append({
                "task_id": item["task_id"],
                "predicted": predicted_answer,
                "expected": item["final_answer"],
                "exact_match": exact_match,
                "level": item.get("level", 0)
            })

        return self._format_results(results)
````

GAIA는 특정 정규화 규칙을 사용하여 다양한 유형의 답변을 처리합니다.

```python
def _normalize_answer(self, answer: str) -> str:
    """Normalize answer string (GAIA official normalization rules)

    Rules:
    1. Numbers: Remove comma separators and unit symbols
    2. Strings: Remove articles, convert to lowercase, remove extra spaces
    3. Lists: Comma-separated, sorted alphabetically
    """
    if not answer:
        return ""

    answer = answer.strip()

    # Check if it's a comma-separated list
    if ',' in answer:
        parts = [self._normalize_single_answer(p.strip()) for p in answer.split(',')]
        parts.sort()  # GAIA requires alphabetical sorting
        return ','.join(parts)
    else:
        return self._normalize_single_answer(answer)

def _normalize_single_answer(self, answer: str) -> str:
    """Normalize single answer (answer without commas)"""
    answer = answer.strip().lower()

    # Remove common articles
    articles = ['the', 'a', 'an']
    words = answer.split()
    if words and words[0] in articles:
        words = words[1:]
        answer = ' '.join(words)

    # Remove currency symbols and percent signs
    answer = answer.replace('$', '').replace('%', '').replace('€', '').replace('£', '')

    # Remove comma separators in numbers
    answer = re.sub(r'(\d),(\d)', r'\1\2', answer)

    # Remove extra spaces
    answer = ' '.join(answer.split())

    # Remove trailing punctuation
    answer = answer.rstrip('.,;:!?')

    return answer
```

GAIA에서는 모델 출력 형식이 `FINAL ANSWER: [answer]`이어야 합니다.

```python
def _extract_answer(self, response: str) -> str:
    """Extract answer from response (GAIA format)

    GAIA requires answer format: FINAL ANSWER: [answer]
    """
    # First try to extract GAIA official format answer
    final_answer_pattern = r'FINAL ANSWER:\s*(.+?)(?:\n|$)'
    match = re.search(final_answer_pattern, response, re.IGNORECASE | re.MULTILINE)
    if match:
        answer = match.group(1).strip()
        # Remove possible brackets
        answer = answer.strip('[]')
        return answer

    # Fallback: Look for other answer markers
    answer_patterns = [
        r'答案[：:]\s*(.+)',
        r'最终答案[：:]\s*(.+)',
        r'Final answer[：:]\s*(.+)',
        r'Answer[：:]\s*(.+)',
    ]

    for pattern in answer_patterns:
        match = re.search(pattern, response, re.IGNORECASE)
        if match:
            return match.group(1).strip()

    # If no marker found, return last non-empty line
    lines = response.strip().split('\n')
    for line in reversed(lines):
        line = line.strip()
        if line and not line.startswith('#'):
            return line

    return response.strip()
```

평가가 완료된 후 GAIA 공식에서 요구하는 JSONL 형식으로 내보낼 수 있습니다.

```python
def export_to_gaia_format(
    self,
    results: Dict[str, Any],
    output_path: Union[str, Path],
    include_reasoning: bool = True
) -> None:
    """Export to GAIA official format (JSONL)

    GAIA required format:
    {"task_id": "xxx", "model_answer": "answer", "reasoning_trace": "reasoning process"}
    """
    output_path = Path(output_path)
    output_path.parent.mkdir(parents=True, exist_ok=True)

    with open(output_path, 'w', encoding='utf-8') as f:
        for result in results.get("detailed_results", []):
            entry = {
                "task_id": result["task_id"],
                "model_answer": result["predicted"]
            }

            if include_reasoning:
                entry["reasoning_trace"] = result.get("response", result["predicted"])

            f.write(json.dumps(entry, ensure_ascii=False) + '\n')
```

**(3) GAIAEvaluationTool: 원클릭 평가 도구**

GAIAEvaluationTool은 완전한 평가 프로세스를 캡슐화하여 원클릭 평가 기능을 제공합니다.

````python
class GAIAEvaluationTool(Tool):
    """GAIA evaluation tool

    Provides one-click evaluation functionality:
    1. Run HelloAgents evaluation
    2. Export GAIA format results
    3. Generate evaluation report
    4. Generate submission guide
    """

    def run(
        self,
        agent: Any,
        level: Optional[int] = None,
        max_samples: Optional[int] = None,
        local_data_dir: Optional[str] = None,
        export_results: bool = True,
        generate_report: bool = True
    ) -> Dict[str, Any]:
        """Execute GAIA one-click evaluation"""
        # Step 1: Run HelloAgents evaluation
        results = self._run_evaluation(agent, level, max_samples, local_data_dir)

        # Step 2: Export GAIA format results
        if export_results:
            self._export_results(results)

        # Step 3: Generate evaluation report
        if generate_report:
            self.generate_report(results)

        return results
````

GAIAEvaluationTool은 자동으로 평가 보고서를 생성합니다.

```python
def generate_report(
    self,
    results: Dict[str, Any],
    output_file: Optional[Union[str, Path]] = None
) -> str:
    """Generate evaluation report"""
    report = f"""# GAIA Evaluation Report

**Generated**: {datetime.now().strftime("%Y-%m-%d %H:%M:%S")}

## 📊 Evaluation Overview

- **Agent**: {results.get("agent_name", "Unknown")}
- **Difficulty Level**: {results.get("level_filter") or 'All'}
- **Total Samples**: {results.get("total_samples", 0)}
- **Exact Matches**: {results.get("exact_matches", 0)}
- **Exact Match Rate**: {results.get("exact_match_rate", 0):.2%}

## 📈 Detailed Metrics

### Level-wise Accuracy

{self._format_level_metrics(results.get("level_metrics", {}))}

## 📝 Sample Details (First 10)

{self._format_sample_details(results.get("detailed_results", [])[:10])}

## 📊 Accuracy Visualization

{self._format_visualization(results.get("exact_match_rate", 0))}

## 💡 Recommendations

{self._format_suggestions(results.get("exact_match_rate", 0))}
"""

    # Save report
    if output_file is None:
        output_dir = Path("./evaluation_reports")
        output_dir.mkdir(parents=True, exist_ok=True)
        output_file = output_dir / f"gaia_report_{datetime.now().strftime('%Y%m%d_%H%M%S')}.md"

    with open(output_file, 'w', encoding='utf-8') as f:
        f.write(report)

    return report
```

## 12.4 데이터 생성 품질 평가

AI 시스템 개발에 있어서 고품질의 훈련 데이터는 시스템 성능의 기초입니다. 이 섹션에서는 AIME(American Invitational Mathematics Examination)<sup>[9]</sup> 스타일 수학 문제 생성을 예로 들어 HelloAgents 프레임워크를 사용하여 생성된 데이터의 품질을 평가하는 방법을 소개합니다.

AIME은 미국수학협회(MAA)가 주최하는 중간 난이도 수학 경시대회로 AMC 10/12와 미국 수학 올림피아드(USAMO) 사이에 위치합니다. AIME 문제는 고유한 특성을 가지고 있습니다. 각 문제의 답은 0에서 999 사이의 정수이고, 문제는 대수학, 기하학, 정수론, 조합론, 확률을 포함한 여러 수학 영역을 다루고, 다단계 추론이 필요하지만 고급 이론을 포함하지 않으며, 중간 정도의 난이도를 갖습니다(AIME 문제 6-9와 동일). 이러한 특성으로 인해 AIME 문제는 수학 문제 생성 품질을 평가하기 위한 이상적인 벤치마크가 됩니다. 통합된 답변 형식은 자동화된 평가를 용이하게 하며 중간 난이도는 대규모 생성에 적합합니다. 우리는 HuggingFace의 `TianHongZXY/aime-1983-2025` 데이터세트를 참조로 사용합니다. 여기에는 1983년부터 2025년까지 900개 이상의 AIME 실제 문제가 포함되어 있으며 생성 및 평가를 위한 풍부한 참조 샘플을 제공합니다.

12.4.1 평가 방법 개요

데이터 생성 품질 평가에서는 LLM 심사위원, 승률, 수동 검증이라는 세 가지 보완적인 평가 방법을 채택합니다. 이 세 가지 방법을 선택한 데에는 두 가지 중요한 이유가 있습니다. 첫째, 방법론적 관점에서 볼 때, 이는 현재 에이전트 분야에서 일반적으로 사용되는 자동 평가 방식과 많은 학술 논문의 주류 관행으로 폭넓은 인지도와 실용적인 기반을 갖추고 있습니다. 둘째, 적용 가능성 관점에서 볼 때 이 세 가지 방법은 우리의 평가 시나리오에 자연스럽게 적합합니다. LLM 판사와 승률은 문제 생성 품질 (multi-dimensional evaluation from correctness, clarity, difficulty matching, etc.)을 평가하는 데 사용되는 반면 수동 확인은 답변 생성 품질(인간 전문가를 통해 답변의 정확성 확인)을 평가하는 데 사용됩니다. 이러한 업무 분업은 매우 합리적이고 이해하기 쉽습니다.

아래에서는 이 세 가지 평가 방법의 구체적인 구현을 자세히 소개합니다. 전체 사례의 구현 흐름은 그림 12.5에 나와 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-5.png" alt="" width="85%"/>
<p>그림 12.5 데이터 생성 품질 평가 흐름도</p>
</div>

**(1) LLM 심사위원 평가**

**디자인 동기 부여**: 데이터 생성 품질 평가에서는 생성된 수많은 문제의 품질을 신속하고 일관되게 평가해야 합니다. 전통적인 수동 평가는 정확하기는 하지만 비용이 많이 들고 비효율적이므로 대규모 데이터 생성 요구 사항을 충족하기 어렵습니다. LLM Judge는 대규모 언어 모델을 심사위원으로 사용하여 생성된 데이터의 품질을 다차원에서 자동으로 평가할 수 있어 평가 효율성을 크게 향상시킬 뿐만 아니라 평가 기준의 일관성을 유지합니다. 더 중요한 것은 LLM 심사위원이 자세한 채점 이유와 개선 제안을 제공하여 생성된 데이터의 강점과 약점을 이해하고 후속 최적화 방향을 제시할 수 있다는 것입니다.

우리의 구현에서 LLM Judge는 다음 네 가지 주요 차원에서 AIME 문제 품질을 평가합니다.

<div align="center">
<p>표 12.5 AIME 문제에 대한 LLM 판사 평가 차원</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-table-5.png" alt="" width="85%"/>
</div>

4가지 차원에서 점수를 얻은 후 이러한 점수를 전체 평가 지표로 집계해야 합니다. 우리는 생성된 문제의 품질 수준을 측정하기 위해 세 가지 주요 지표를 정의합니다.

**평가 지표**:

**1. 평균 점수**: 생성된 문제의 전반적인 품질 수준을 반영하여 4개 차원에 걸쳐 모든 문제의 평균 점수를 계산합니다.
$$
\text{평균 점수} = \frac{1}{N} \sum_{i=1}^{N} \frac{\sum_{d=1}^{4} S_{i,d}}{4}
$$

**2. 합격률**: 생성된 문제에 대한 기본 품질 보증을 반영하여 평균 점수가 3.5 이상인 문제의 비율을 계산합니다.

$$
\text{합격률} = \frac{|\{i : \text{점수}_i \geq 3.5\}|}{N}
$$

**3. 우수율**: 발생된 문제 중 품질이 높은 비율을 반영하여 평균 점수가 4.5 이상인 문제의 비율을 계산합니다.

$$
\text{우수한 비율} = \frac{|\{i : \text{점수}_i \geq 4.5\}|}{N}
$$

어디에:
- $N$은 평가된 총 문제 수입니다.
- $S_{i,d}$는 $d$번째 차원(1-5점)의 $i$번째 문제의 점수입니다.
- $\text{Score}_i$는 $i$번째 문제의 평균 점수입니다(4개 차원 점수의 평균).

이 세 가지 지표는 다양한 각도에서 생성 품질을 반영합니다. 평균 점수는 전체 수준을 나타내고 합격률은 기본 품질을 보장하며 우수한 비율은 고품질 출력 기능을 측정합니다.

**(2) 승률 평가**

**디자인 동기 부여**: LLM 심사위원은 다차원적인 절대 점수를 제공할 수 있지만 생성된 문제와 실제 문제 사이의 품질 격차를 측정하려면 상대적인 평가 지표도 필요합니다. 쌍별 비교를 통한 승률 평가를 통해 LLM은 생성된 문제와 실제 문제 중 어느 것이 더 나은지 직접 판단할 수 있습니다. 이러한 상대적 비교는 절대 채점보다는 인간의 판단 습관에 더 부합하며, 생성된 문제의 상대적인 장단점을 더 쉽게 발견할 수 있습니다. 이상적으로 생성된 문제의 품질이 실제 문제에 가까울 경우 승률은 약 50% (i.e., generated problems and real problems each have 50% win rate)여야 합니다. 이 측정법은 간단하고 직관적이므로 발전 시스템의 전반적인 품질 수준을 빠르게 판단할 수 있습니다.

우리 구현에서는 그림 12.6에 표시된 흐름을 통해 승률 평가가 수행됩니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-6.png" alt="" width="85%"/>
<p>그림 12.6 데이터 생성 품질 평가 흐름도</p>
</div>

쌍별 비교 평가에서 각 비교는 생성된 문제 승리(Win), 실제 문제 승리(Los) 또는 동점(Tie)의 세 가지 가능한 결과를 생성합니다. 우리는 다음 세 가지 결과의 비율을 계산하여 생성된 문제의 품질을 평가합니다.

**평가 지표**:

**1. 승률**: 실제 문제에 비해 생성된 문제의 장점을 반영하여 더 나은 것으로 판단된 생성된 문제의 비율입니다.

$$
\text{승률} = \frac{\text{승수}}{\text{총 비교 수}}
$$

**2. 손실률**: 실제 문제에 비해 생성된 문제의 단점을 반영하여 더 나은 것으로 판단된 실제 문제의 비율입니다.

$$
\text{손실률} = \frac{\text{손실}}{\text{총 비교}}
$$

**3. 동률**: 생성된 문제와 실제 문제 간의 유사성을 반영하여 동등한 품질로 판단되는 비율입니다.

$$
\text{동점률} = \frac{\text{동점}}{\text{총 비교}}
$$

여기서 Total Comparisons는 총 비교 횟수이고, Wins, Losses, Ties는 각각 생성된 문제의 Wins, Losses, Ties 수입니다. 이 세 가지 측정항목은 다음을 충족합니다. 승률 + 패율 + 동률 = 100%.

**이상적인 결과**: 승률 ≒ 50%(생성 품질이 실제 문제에 가깝다는 것을 나타냄) 승률이 50%보다 현저히 낮으면 생성된 문제 품질이 실제 문제보다 열등하고 생성 전략에 최적화가 필요함을 나타냅니다. 승률이 50%보다 훨씬 높으면 생성된 문제가 일부 측면에서 실제 문제를 능가하거나 평가 기준에 편향이 있음을 나타낼 수 있습니다.

**(3) 수동 확인**

**설계 동기**: LLM 심사위원과 승률은 자동으로 문제 품질을 평가할 수 있지만, 엄격한 논리적 추론이 필요한 수학적 문제의 경우 여전히 수동 검증이 필수적입니다. 특히 답변 생성 품질을 평가할 때 답변의 정확성, 해결 단계 완전성 및 수학적 추론의 엄격함을 검증하려면 인간 전문가가 필요합니다. 또한 수동 검증을 통해 문제 혁신 및 관심도와 같은 주관적인 요소 등 자동 평가에서 놓칠 수 있는 문제를 발견할 수 있습니다. 수동 검증 효율성과 경험을 향상시키기 위해 검증자가 편리하게 문제를 찾아보고, 점수를 매기고, 상태에 주석을 달고, 의견을 추가할 수 있는 Gradio 기반 웹 인터페이스를 개발하여 수동 검증에 대한 장벽을 크게 낮췄습니다.

구현 시 수동 확인은 다음 단계를 통해 수행됩니다.

1. 문제, 답, 해결방안 읽기
2. 점수(1~5점): 정확성, 명확성, 난이도 매칭, 완전성
3. 상태에 주석을 답니다:
   - ✅ 승인(통과)
   - ❌ 거부됨 (거부됨)
   - 🔄 need_revision (개정 필요)
4. 댓글 추가

### 12.4.2 시스템 아키텍처

데이터 생성 및 평가 시스템은 모듈식 설계를 채택합니다.

```
data_generation/
├── aime_generator.py              # AIME problem generator
├── human_verification_ui.py       # Manual verification interface
├── run_complete_evaluation.py     # Complete evaluation flow
│
├── generated_data/                # Generated data
│   ├── aime_generated_XXXXXX.json
│   └── generation_report_XXXXXX.md
│
└── evaluation_results/            # Evaluation results
    └── XXXXXX/
        ├── llm_judge/
        ├── win_rate/
        └── comprehensive_report.md
```

시스템에는 네 가지 핵심 구성 요소가 포함되어 있습니다. 첫 번째는 AIMEGenerator(문제 생성기)로, HelloAgents 프레임워크를 사용하여 AIME 스타일 문제를 생성하고 일괄 생성 및 진행률 저장을 지원하며 API 속도 제한을 자동으로 처리합니다. 두 번째는 4차원 품질 평가를 제공하고 JSON 결과 및 마크다운 보고서를 자동으로 생성하는 LLMJudgeTool(LLM 심사위원 평가 도구)입니다. 세 번째는 WinRateTool(승률 평가 도구)로, 쌍별 비교 평가를 통해 승률, 패률, 동점률을 계산합니다. 마지막으로 Gradio 웹 인터페이스를 기반으로 채점 및 상태 주석을 지원하는 HumanVerificationUI(수동 검증 인터페이스)입니다.

### 12.4.3 AIME 문제 생성기 구현

```python
class AIMEGenerator:
    """AIME Problem Generator"""

    def __init__(
        self,
        llm: HelloAgentsLLM = None,
        delay_seconds: float = 1.0,
        use_reference_examples: bool = True,
        reference_dataset: str = "TianHongZXY/aime-1983-2025"
    ):
        self.llm = llm or HelloAgentsLLM()
        self.agent = SimpleAgent(
            name="AIME Generator",
            llm=self.llm,
            system_prompt="You are a professional mathematics competition problem designer."
        )
        self.delay_seconds = delay_seconds
        self.use_reference_examples = use_reference_examples

        # Load reference examples from 900+ AIME problems (1983-2025)
        if use_reference_examples:
            dataset = load_dataset(reference_dataset, split="test")
            self.reference_examples = list(dataset)
```

우리의 목표는 유사한 스타일 데이터 세트를 생성하는 것이므로 900개 이상의 AIME 실제 문제(1983-2025)에서 참조 예제를 무작위로 선택합니다.

세대 프롬프트 디자인(영어):

```python
GENERATION_PROMPT = """You are a professional mathematics competition problem designer, skilled in creating AIME (American Invitational Mathematics Examination) style problems.

【Reference Example】(For style reference only, please generate a completely different problem)
Problem: {example_problem}
Answer: {example_answer}

AIME Problem Characteristics:
1. Answer: An integer between 0 and 999
2. Topics: Algebra, Geometry, Number Theory, Combinatorics, Probability, etc.
3. Style: Requires multi-step reasoning, but no advanced theory
4. Difficulty: Medium to hard (similar to AIME problems 6-9)

Please generate a **completely different** AIME-style mathematics problem, including:
1. Problem statement (clear and complete, different from the reference)
2. Answer (an integer between 0 and 999, different from the reference)
3. Detailed solution (including all reasoning steps)
4. Topic classification (Algebra/Geometry/Number Theory/Combinatorics/Probability)

Please output in the following JSON format:
{
    "problem": "Problem statement in English",
    "answer": 123,
    "solution": "Detailed solution steps in English",
    "topic": "Algebra"
}
"""
```

우리는 네 가지 중요한 이유 때문에 영어로 문제를 생성하기로 결정했습니다. 첫 번째는 AIME 실제 문제와의 일관성(AIME은 영어 경쟁이므로 영어 문제 생성이 더 합리적임), 두 번째는 평가 공정성 보장(LLM 심사위원 평가는 영어 대 영어일 때 더 공정함), 세 번째는 국제화 촉진(영어 문제는 더 널리 사용될 수 있음), 마지막으로 번역 문제 방지(중국어-영어 번역의 정확성에 대해 걱정할 필요 없음)입니다.

일괄 생성 구현:

```python
def generate_and_save(self, num_problems: int = 30, output_dir: str = "data_generation/generated_data"):
    """Generate and save problems with intelligent delay"""
    # Clean old checkpoints
    for file in os.listdir(output_dir):
        if file.startswith("checkpoint_") and file.endswith(".json"):
            os.remove(os.path.join(output_dir, file))

    # Generate with tqdm progress bar
    with tqdm(total=num_problems, desc="Generating AIME problems", unit="problem") as pbar:
        last_call_time = 0

        for i in range(num_problems):
            # Ensure minimum delay between API calls
            if last_call_time > 0:
                elapsed = time.time() - last_call_time
                if elapsed < self.delay_seconds:
                    wait_time = self.delay_seconds - elapsed
                    time.sleep(wait_time)

            # Generate problem (randomly select reference example)
            start_time = time.time()
            problem = self.generate_single()
            last_call_time = time.time()
            generation_time = last_call_time - start_time

            # Update progress bar
            pbar.set_postfix({
                "topic": problem.get('topic', 'N/A'),
                "answer": problem.get('answer', 'N/A'),
                "time": f"{generation_time:.1f}s"
            })
            pbar.update(1)

    return generated_data_path
```

LaTeX 수학 공식 지원:

생성된 AIME 문제에는 특별한 JSON 구문 분석 처리가 필요한 LaTeX 수학 공식(예: `$\frac{a}{b}$`, `$\sqrt{x}$`)이 포함되어 있습니다.

```python
def _parse_response(self, response: str) -> Dict[str, Any]:
    """Parse LLM response (supports LaTeX mathematical formulas)"""
    import re

    # Extract JSON part
    if "```json" in response:
        json_str = response.split("```json")[1].split("```")[0].strip()
    else:
        json_str = response.strip()

    try:
        problem_data = json.loads(json_str)
    except json.JSONDecodeError:
        # Fix LaTeX escape issue: convert \frac to \\frac
        # Regular expression: find unescaped backslashes
        fixed_json_str = re.sub(r'(?<!\\)\\(?!["\\/bfnrtu])', r'\\\\', json_str)
        problem_data = json.loads(fixed_json_str)

    return problem_data
```

LaTeX 수식의 백슬래시(예: `\frac`, `\sqrt`)는 JSON에서 잘못된 이스케이프 문자이므로 구문 분석 오류가 발생합니다.
```
Invalid \escape: line 4 column 185 (char 375)
```

정규식을 사용하여 이스케이프되지 않은 백슬래시를 이중 백슬래시로 대체하여 JSON에서 합법적으로 만듭니다.

12.4.4 LLM 심사위원 평가 도구

LLM 판사 도구는 LLM을 판사로 사용하여 생성된 문제에 대한 다차원 평가를 수행합니다.

```python
class LLMJudgeTool(Tool):
    """LLM Judge evaluation tool"""

    def run(self, params: Dict[str, Any]) -> str:
        """Run LLM Judge evaluation"""
        # 1. Load generated data
        gen_dataset = AIDataset(dataset_type="generated", data_path=params["generated_data_path"])
        gen_problems = gen_dataset.load()

        # 2. Load reference data (AIME 2025)
        ref_dataset = AIDataset(dataset_type="real", year=2025)
        ref_problems = ref_dataset.load()

        # 3. Create evaluator
        evaluator = LLMJudgeEvaluator(llm=self.llm, judge_model=params.get("judge_model", "gpt-4o"))

        # 4. Run evaluation
        results = evaluator.evaluate_batch(gen_problems, max_samples=params.get("max_samples"))

        # 5. Save results
        evaluator.export_results(results, result_file)

        # 6. Generate report
        self._generate_report(results, report_file)

        return json.dumps({"status": "success", "metrics": results["metrics"]})
```

**평가 프롬프트**:

```python
EVALUATION_PROMPT = """Please evaluate the quality of the following AIME mathematics problem.

Problem:
{problem}

Answer: {answer}

Solution:
{solution}

Please score from the following 4 dimensions (1-5 points):

1. **Correctness**: Is the mathematical logic correct, is the answer accurate
2. **Clarity**: Is the problem statement clear, is the solution easy to understand
3. **Difficulty Match**: Does the difficulty match AIME standards (medium to hard)
4. **Completeness**: Are the solution steps complete, does it include necessary reasoning

Please output in the following JSON format:
{
    "correctness": 5,
    "clarity": 4,
    "difficulty_match": 4,
    "completeness": 5,
    "comments": "Evaluation reason"
}
"""
```

**평가 보고서 예**:

```markdown
# LLM Judge Evaluation Report

## Overall Score

- **Average Total Score**: 4.2/5.0
- **Pass Rate**: 85.0% (≥3.5 points)
- **Excellent Rate**: 40.0% (≥4.5 points)

## Dimension Scores

| Dimension | Average Score | Rating |
|------|--------|------|
| Correctness | 4.3/5.0 | Good ⭐⭐⭐⭐ |
| Clarity | 4.1/5.0 | Good ⭐⭐⭐⭐ |
| Difficulty Match | 4.0/5.0 | Good ⭐⭐⭐⭐ |
| Completeness | 4.4/5.0 | Good ⭐⭐⭐⭐ |
```

### 12.4.5 승률 평가 도구

승률 도구는 쌍별 비교를 통해 실제 문제와 관련하여 생성된 데이터의 품질을 평가합니다.

```python
class WinRateTool(Tool):
    """Win Rate evaluation tool"""

    def run(self, params: Dict[str, Any]) -> str:
        """Run Win Rate evaluation"""
        # 1. Load generated data
        gen_dataset = AIDataset(dataset_type="generated", data_path=params["generated_data_path"])
        gen_problems = gen_dataset.load()

        # 2. Load reference data (AIME 2025)
        ref_dataset = AIDataset(dataset_type="real", year=2025)
        ref_problems = ref_dataset.load()

        # 3. Create evaluator
        evaluator = WinRateEvaluator(llm=self.llm, judge_model=params.get("judge_model", "gpt-4o"))

        # 4. Run evaluation
        results = evaluator.evaluate_win_rate(gen_problems, ref_problems, num_comparisons=params.get("num_comparisons"))

        # 5. Save results and report
        evaluator.export_results(results, result_file)
        self._generate_report(results, report_file)

        return json.dumps({"status": "success", "metrics": results["metrics"]})
```

AIDataset은 생성된 데이터와 AIME 실제 문제 데이터를 로드하는 역할을 담당하며 다음 두 가지 데이터 유형을 지원합니다.

```python
class AIDataset:
    """AI dataset loader

    Supports two data types:
    1. generated: Generated data (JSON format)
    2. real: AIME real problems (loaded from HuggingFace)
    """

    def __init__(
        self,
        dataset_type: str = "generated",
        data_path: Optional[str] = None,
        year: Optional[int] = None
    ):
        self.dataset_type = dataset_type
        self.data_path = data_path
        self.year = year  # Only for real type, default 2025

    def load(self) -> List[Dict[str, Any]]:
        """Load dataset"""
        if self.dataset_type == "generated":
            return self._load_generated_data()
        elif self.dataset_type == "real":
            return self._load_real_data()

    def _load_real_data(self) -> List[Dict[str, Any]]:
        """Load AIME 2025 real problems from HuggingFace"""
        from huggingface_hub import snapshot_download

        # Use AIME 2025 dataset
        repo_id = "math-ai/aime25"

        # Download dataset
        local_dir = snapshot_download(
            repo_id=repo_id,
            repo_type="dataset"
        )

        # Read JSONL file
        data_file = list(Path(local_dir).glob("*.jsonl"))[0]
        data = []
        with open(data_file, 'r', encoding='utf-8') as f:
            for line in f:
                if line.strip():
                    data.append(json.loads(line))

        # Unify data format (AIME 2025 uses lowercase field names)
        problems = []
        for idx, item in enumerate(data):
            problem = {
                "problem_id": item.get("id", f"aime_2025_{idx}"),
                "problem": item.get("problem", ""),
                "answer": item.get("answer", ""),
                "solution": item.get("solution", ""),  # AIME 2025 has no solution field
            }
            problems.append(problem)

        return problems
```

우리는 네 가지 이유로 AIME 2025 데이터 세트만 사용하기로 결정했습니다. 첫 번째는 데이터 적시성(2025는 최신 AIME 경쟁 데이터), 두 번째는 유지 관리 단순화(단일 데이터 세트만 유지, 코드는 더 간결함), 세 번째는 통합 형식(JSONL 형식, 필드 이름을 소문자로 통합), 마지막으로 충분한 대표성(30개의 문제는 생성 품질을 평가하기에 충분함)입니다.

**비교 프롬프트**:

```python
COMPARISON_PROMPT = """Please compare the quality of the following two AIME mathematics problems and judge which is better.

【Problem A - Generated Problem】
Problem: {problem_a}
Answer: {answer_a}
Solution: {solution_a}

【Problem B - AIME Real Problem】
Problem: {problem_b}
Answer: {answer_b}
Solution: {solution_b}

Please compare from the following aspects:
1. Rigor of mathematical logic
2. Clarity of problem statement
3. Reasonableness of difficulty
4. Completeness of solution

Please output in the following JSON format:
{
    "winner": "A" or "B" or "Tie",
    "reason": "Judgment reason"
}
"""
```

**평가 보고서 예**:

```markdown
# Win Rate Evaluation Report

## Win Rate Statistics

| Metric | Value | Percentage |
|------|------|--------|
| Generated Data Wins | 9 times | 45.0% |
| AIME Real Problems Win | 8 times | 40.0% |
| Tie | 3 times | 15.0% |

**Win Rate**: 45.0%

✅ **Good**: Generated data quality is close to reference data (gap <10%).
```

### 12.4.6 수동 검증 인터페이스

Gradio를 사용하여 생성된 문제의 수동 검증을 지원하는 웹 인터페이스를 만듭니다.

```python
class HumanVerificationUI:
    """Manual verification interface"""

    def launch(self, share: bool = False):
        """Launch Gradio interface"""
        with gr.Blocks(title="AIME Problem Manual Verification") as demo:
            gr.Markdown("# 🎯 AIME Problem Manual Verification System")

            with gr.Row():
                with gr.Column(scale=2):
                    # Problem display area
                    problem_text = gr.Textbox(label="Problem Description", lines=5, interactive=False)
                    answer_text = gr.Textbox(label="Answer", interactive=False)
                    solution_text = gr.Textbox(label="Solution Process", lines=10, interactive=False)

                with gr.Column(scale=1):
                    # Scoring area
                    correctness_slider = gr.Slider(1, 5, value=3, step=1, label="Correctness")
                    clarity_slider = gr.Slider(1, 5, value=3, step=1, label="Clarity")
                    difficulty_slider = gr.Slider(1, 5, value=3, step=1, label="Difficulty Match")
                    completeness_slider = gr.Slider(1, 5, value=3, step=1, label="Completeness")

                    # Status selection
                    status_radio = gr.Radio(
                        choices=["approved", "rejected", "needs_revision"],
                        value="approved",
                        label="Status"
                    )

                    # Verification button
                    verify_btn = gr.Button("✅ Submit Verification", variant="primary")

            demo.launch(share=share, server_name="127.0.0.1", server_port=7860)
```

**사용 방법**:

```bash
# Launch manual verification interface
python data_generation/human_verification_ui.py data_generation/generated_data/aime_generated_XXXXXX.json

# Open browser and visit
http://127.0.0.1:7860
```

최종 효과는 그림 12.7에서 참조할 수 있습니다. 문제의 정확성을 위해서는 직접 검토하는 것이 가장 좋습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-7.png" alt="" width="85%"/>
<p>그림 12.7 AIME 문제 매뉴얼 확인 페이지</p>
</div>

**확인 프로세스**:

1. 브라우저에서 확인 인터페이스 열기
2. 문제, 답, 해결책 읽기
3. 4가지 차원에서 점수를 매깁니다(1~5점).
4. 인증상태 선택 (approved/rejected/needs_revision)
5. 댓글 추가(선택)
6. "인증서 제출"을 클릭하세요.
7. 다음 문제 보기

**검증 결과 저장**:

검증 결과는 자동으로 `<data_path>_verifications.json`로 저장됩니다:

```json
{
  "gen_aime_1": {
    "problem_id": "gen_aime_1",
    "scores": {
      "correctness": 5,
      "clarity": 4,
      "difficulty_match": 4,
      "completeness": 5
    },
    "total_score": 4.5,
    "status": "approved",
    "comments": "Problem quality is very good, logic is rigorous",
    "verified_at": "2025-01-10T12:00:00"
  }
}
```

### 12.4.7 전체 평가 흐름

모든 평가 방법을 완전한 흐름으로 통합합니다.

```python
def run_complete_evaluation(
    num_problems: int = 30,
    delay_seconds: float = 3.0
):
    """
    Run complete evaluation flow

    Args:
        num_problems: Number of problems to generate
        delay_seconds: Delay between each generation (seconds), avoid API rate limit
    """
    # Step 1: Generate AIME problems
    generator = AIMEGenerator(delay_seconds=delay_seconds)
    generated_data_path = generator.generate_and_save(
        num_problems=num_problems,
        output_dir="data_generation/generated_data"
    )

    # Step 2: Evaluation
    # Create evaluation result directory
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    evaluation_dir = f"data_generation/evaluation_results/{timestamp}"
    os.makedirs(evaluation_dir, exist_ok=True)
    os.makedirs(os.path.join(evaluation_dir, "llm_judge"), exist_ok=True)
    os.makedirs(os.path.join(evaluation_dir, "win_rate"), exist_ok=True)

    # Create LLM
    llm = HelloAgentsLLM()

    # Step 2.1: LLM Judge evaluation
    llm_judge_result = None
    try:
        llm_judge_tool = LLMJudgeTool(llm=llm)
        llm_judge_result_json = llm_judge_tool.run({
            "generated_data_path": generated_data_path,
            "reference_year": 2025,
            "max_samples": num_problems,
            "output_dir": os.path.join(evaluation_dir, "llm_judge"),
            "judge_model": "gpt-4o"
        })
        llm_judge_result = json.loads(llm_judge_result_json)
    except Exception as e:
        print(f"❌ LLM Judge evaluation failed: {e}")

    # Step 2.2: Win Rate evaluation
    win_rate_result = None
    try:
        win_rate_tool = WinRateTool(llm=llm)
        win_rate_result_json = win_rate_tool.run({
            "generated_data_path": generated_data_path,
            "reference_year": 2025,
            "num_comparisons": min(num_problems, 20),
            "output_dir": os.path.join(evaluation_dir, "win_rate"),
            "judge_model": "gpt-4o"
        })
        win_rate_result = json.loads(win_rate_result_json)
    except Exception as e:
        print(f"❌ Win Rate evaluation failed: {e}")

    # Step 3: Generate comprehensive report
    comprehensive_report_path = None
    if llm_judge_result or win_rate_result:
        comprehensive_report_path = os.path.join(evaluation_dir, "comprehensive_report.md")
        report = generate_comprehensive_report(
            generated_data_path,
            llm_judge_result,
            win_rate_result
        )
        with open(comprehensive_report_path, 'w', encoding='utf-8') as f:
            f.write(report)

    return {
        "generated_data_path": generated_data_path,
        "llm_judge_result": llm_judge_result,
        "win_rate_result": win_rate_result,
        "comprehensive_report_path": comprehensive_report_path
    }
```

**실행 방법**:

```bash
# Basic usage (default 3 second delay)
python data_generation/run_complete_evaluation.py 30

# Custom delay (recommended 3-5 seconds, avoid API rate limit)
python data_generation/run_complete_evaluation.py 30 3.0

# Parameter explanation:
# - 30: Number of problems to generate
# - 3.0: Delay between each generation (seconds)

# Explanation:
# - Generation phase: Randomly select reference examples from 900+ AIME real problems (1983-2025)
# - Evaluation phase: Quality comparison with AIME 2025 real problems
# - Dataset source: math-ai/aime25 (JSONL format)
```

**출력 예**:

```
================================================================================
🚀 AIME Data Generation and Evaluation Complete Flow
================================================================================

Configuration:
  - Number of problems to generate: 30
  - API delay: 3.0 seconds/problem
  - Generation reference data: TianHongZXY/aime-1983-2025 (900+ problems)
  - Evaluation reference: AIME 2025 real problems

================================================================================
📝 Step 1: Generate AIME Problems
================================================================================
📚 Load AIME real problem dataset: TianHongZXY/aime-1983-2025
   ✓ Loaded 963 reference problems

🎯 Start generating AIME problems
   Target quantity: 30
   Generation model: gpt-4o
   Delay setting: 3.0 seconds/problem

Generating AIME problems:  100%|██████████| 30/30 [01:30<00:00, 3.00s/problem, topic=Algebra, answer=123, time=3.0s]

✅ Step 1 complete! Generated data saved at: data_generation/generated_data/aime_generated_20250110_120000.json

🎯 Step 2.1: LLM Judge Evaluation (vs AIME 2025)

✅ LLM Judge evaluation complete!
   Average total score: 4.2/5.0
   Pass rate: 85.0%

🏆 Step 2.2: Win Rate Evaluation (vs AIME 2025)

✅ Win Rate evaluation complete!
   Win Rate: 45.0%

================================================================================
📊 Step 3: Generate Comprehensive Report
================================================================================

✅ Comprehensive report saved: data_generation/evaluation_results/20250110_120000/comprehensive_report.md

================================================================================
🎉 Complete Evaluation Flow Finished!
================================================================================

📁 Output Files:
   - Generated data: data_generation/generated_data/aime_generated_20250110_120000.json
   - Evaluation result directory: data_generation/evaluation_results/20250110_120000
   - LLM Judge report: data_generation/evaluation_results/20250110_120000/llm_judge/llm_judge_report_20250110_120000.md
   - Win Rate report: data_generation/evaluation_results/20250110_120000/win_rate/win_rate_report_20250110_120000.md
   - Comprehensive report: data_generation/evaluation_results/20250110_120000/comprehensive_report.md

💡 Next Steps:
   1. View comprehensive report: data_generation/evaluation_results/20250110_120000/comprehensive_report.md
   2. Run manual verification: python data_generation/human_verification_ui.py data_generation/generated_data/aime_generated_20250110_120000.json
```

12.4.8 종합평가 보고서

시스템은 모든 평가 결과를 요약한 종합 평가 보고서를 자동으로 생성합니다. 다음은 보고서 예시입니다.

```markdown
# AIME Data Generation and Evaluation Comprehensive Report

## 1. Basic Information

- **Generation Time**: 2025-01-10 12:00:00
- **Number of Generated Problems**: 30
- **Reference AIME Year**: 2025

## 2. Data Generation Statistics

### Topic Distribution

| Topic | Quantity | Proportion |
|------|------|------|
| Algebra | 10 | 33.3% |
| Geometry | 8 | 26.7% |
| Number Theory | 7 | 23.3% |
| Combinatorics | 3 | 10.0% |
| Probability | 2 | 6.7% |

## 3. LLM Judge Evaluation Results

### Overall Score

- **Average Total Score**: 4.2/5.0
- **Pass Rate**: 85.0% (≥3.5 points)
- **Excellent Rate**: 40.0% (≥4.5 points)

### Dimension Scores

| Dimension | Average Score | Rating |
|------|--------|------|
| Correctness | 4.3/5.0 | Good ⭐⭐⭐⭐ |
| Clarity | 4.1/5.0 | Good ⭐⭐⭐⭐ |
| Difficulty Match | 4.0/5.0 | Good ⭐⭐⭐⭐ |
| Completeness | 4.4/5.0 | Good ⭐⭐⭐⭐ |

## 4. Win Rate Evaluation Results

### Win Rate Statistics

| Metric | Value | Percentage |
|------|------|--------|
| Generated Data Wins | 9 times | 45.0% |
| AIME Real Problems Win | 8 times | 40.0% |
| Tie | 3 times | 15.0% |

**Win Rate**: 45.0%

✅ **Good**: Generated data quality is close to reference data (gap <10%).

## 5. Comprehensive Conclusion

Based on the results of LLM Judge and Win Rate evaluation methods:

1. **LLM Judge Evaluation**: Average quality of generated data is **4.2/5.0**
2. **Win Rate Evaluation**: Win rate of generated data relative to AIME 2025 real problems is **45.0%**

✅ **Conclusion**: Generated data quality is **excellent**, reaching or exceeding AIME real problem level. Can be used for practical applications.

## 6. Improvement Suggestions

- ✅ Continue maintaining current generation strategy
- ✅ Can consider increasing generation quantity
- ✅ Recommend manual verification to ensure quality

## 7. Next Steps

1. **Manual Verification**: Run `python data_generation/human_verification_ui.py <data_path>` for manual verification
2. **View Detailed Results**:
   - LLM Judge detailed report
   - Win Rate detailed report
3. **Data Usage**: If quality is satisfactory, generated data can be used for training or testing
```

실제 사용 경험을 바탕으로 다음 내용을 요약합니다.

데이터 생성에서는 적절한 지연 시간(2~3초)을 사용하여 API 속도 제한을 피하고, 체크포인트 저장을 활성화하여 중단 손실을 방지하고, 먼저 작은 배치(10개)로 테스트하여 대규모 생성 전에 문제가 없는지 확인하고, 정기적으로 생성 품질을 확인하여 제때에 프롬프트를 조정합니다. 평가 전략에서는 LLM Judge와 Win Rate 방법을 결합하는 것이 좋습니다. 여기서 LLM Judge는 절대 품질 평가에 사용되고 Win Rate는 상대적 품질 비교에 사용되며 최종 품질 관리에는 수동 검증이 사용됩니다. 품질 기준에 대해서는 LLM 심사위원 평균 점수 4.0/5.0 이상, 승률 45% 이상(50%에 가까움), 합격률 80% 이상, 수동 검증 합격률 90% 이상을 권장합니다. 반복 최적화에서는 평가 결과를 기반으로 생성 프롬프트를 조정하고, 점수가 낮은 문제의 공통 문제를 분석하고, 점수가 높은 문제의 장점을 참조하고, 생성 전략을 지속적으로 개선합니다.

이 섹션을 학습함으로써 우리는 LLM 심사위원 평가, 승률 평가 및 수동 검증의 세 가지 방법을 포함하여 데이터 생성 품질 평가를 위해 HelloAgents 프레임워크를 사용하는 방법을 마스터했습니다. 이 완전한 평가 시스템은 생성된 데이터의 고품질을 보장하고 AI 시스템 교육 및 테스트를 위한 안정적인 데이터 지원을 제공합니다.

LLM 심사위원 및 승률 평가를 위해 HelloAgents는 도구를 통합하고 완전한 예제 코드를 제공했습니다. 이 두 가지 평가 방법의 구체적인 구현 세부 사항에 관심이 있는 경우 예제 코드를 참조할 수도 있습니다.

## 12.5 장 요약

이 장에서는 HelloAgents 프레임워크에 대한 완전한 성능 평가 시스템을 구축했습니다. 학습한 핵심 내용을 검토해 보겠습니다.

**(1) 평가제도 개요**

우리는 에이전트의 다양한 역량 차원을 포괄적으로 다루는 3단계 평가 시스템을 구축했습니다. 첫 번째는 도구 호출 능력 평가(BFCL)로, 정밀한 평가를 위해 AST 매칭 기술을 사용하여 단순, 다중, 병렬, 무관성 4가지 범주를 포함한 에이전트 기능 호출 정확도를 평가하는 데 중점을 둡니다. 두 번째는 GAIA(일반 능력 평가)로, 다단계 추론, 도구 사용, 파일 처리 및 기타 기능에 중점을 두고 466개 실제 문제의 3가지 난이도를 포함하여 에이전트의 종합적인 문제 해결 능력을 평가합니다. 세 번째는 데이터 생성 품질 평가(AIME)로, LLM 판단 및 승률 방법을 사용하여 LLM에서 생성된 데이터 품질을 평가하고 수동 검증 및 포괄적인 보고서 생성을 지원하여 생성된 데이터가 참조 데이터 품질 표준에 도달하도록 보장합니다.

**(2) 핵심 기술 포인트 **

기술 구현에서는 6가지 핵심 기술 포인트를 채택했습니다. 첫 번째는 모듈식 설계이며, 평가 시스템은 데이터 계층(데이터 로드 및 관리를 담당하는 데이터 세트), 평가 계층(평가 흐름 실행을 담당하는 평가자), 메트릭 계층(다양한 평가 메트릭 계산을 담당하는 메트릭)의 3단계 아키텍처를 채택합니다. 두 번째는 도구 캡슐화입니다. 모든 평가 기능은 도구로 캡슐화되며, 에이전트가 직접 호출하거나 워크플로에 통합하거나 통합 인터페이스를 통해 사용할 수 있습니다. 세 번째는 AST 일치 기술로, 함수 호출에 추상 구문 트리 일치를 사용하고 단순한 문자열 일치보다 더 지능적이며 매개변수 순서를 무시하고 동등한 표현식을 인식하며 형식 차이를 무시할 수 있습니다. 넷째는 다중 모드 지원입니다. GAIA 평가는 텍스트 질문, 첨부 파일, 이미지 입력 ​​및 기타 다중 모드 데이터를 지원합니다. 다섯 번째는 LLM 판사 평가로, LLM을 판사로 사용하여 생성된 데이터 품질을 평가하고 다차원 채점(정확성, 명확성, 난이도 일치, 완전성), 자동화된 평가 흐름, 세부 평가 보고서를 제공하고 맞춤형 평가 차원 및 표준을 지원합니다. 여섯째는 승률 비교 평가로, 쌍별 비교(생성된 데이터 대 참조 데이터)를 통해 생성 품질을 평가하며, LLM은 어느 쪽이 더 나은지 판단하고 승률 통계를 계산하며, 50%에 가까울수록 동등한 품질을 나타냅니다.

**(3) 확장 방향**

본 장의 평가 체계를 바탕으로 네 가지 방향으로 확장할 수 있다. 첫 번째는 새로운 평가 벤치마크를 추가하고, BFCL 및 GAIA 구현 패턴을 참조하고, Dataset, Evaluator, Metrics 세 가지 구성 요소를 구현하고 사용할 도구로 캡슐화할 수 있습니다. 두 번째는 사용자 정의 평가 지표이며, Metrics 클래스에 새로운 지표 계산 방법을 추가하고, 특정 애플리케이션 시나리오에 따라 지표를 설계합니다. 세 번째는 CI/CD 흐름에 통합하고, 코드 커밋에 대한 평가를 자동으로 실행하고, 성능 저하를 방지하기 위해 성능 임계값을 설정하고, 평가 보고서를 생성하고 보관하는 것입니다. 넷째는 데이터 생성 평가 확장, 더 많은 데이터 유형 지원((code, dialogue, documents, etc.)), 더 많은 평가 차원 추가((innovation, diversity, etc.)), 더 많은 참조 데이터 세트 통합, 다중 모델 비교 평가 지원입니다.

**12장을 완료한 것을 축하합니다!** 🎉

평가는 에이전트 개발의 중요한 부분이며 이를 통해 다음을 수행할 수 있습니다.

- 상담원 능력을 객관적으로 측정
- 문제 발견 및 수정
- 지속적인 시스템 개선

다음 장에서는 실제 프로젝트에 HelloAgents 프레임워크를 적용하는 방법을 살펴보겠습니다.

**계속하세요!** 💪

## 연습

> **힌트**: 일부 연습에는 표준 답변이 없으며 에이전트 성능 평가에 대한 학습자의 포괄적인 이해와 실무 능력을 키우는 데 중점을 둡니다.

1. 이 장에서는 여러 에이전트 평가 벤치마크를 소개했습니다. 분석해 주십시오:

- 섹션 12.1.2에서는 BFCL, GAIA, AgentBench 및 기타 평가 벤치마크가 도입되었습니다. BFCL과 GAIA를 비교해 보세요. 각각 에이전트의 어떤 핵심 기능을 평가합니까? BFCL은 AST 일치 알고리즘을 사용하는 반면 GAIA는 Quasi Exact Match를 사용하는 이유는 무엇입니까? 이 두 가지 평가 방법의 장점과 단점은 무엇입니까?
   - 다음 기능을 평가해야 하는 "지능형 고객 서비스 시스템"을 구축한다고 가정해 보겠습니다. (1) 사용자 의도를 이해하는 정확성; (2) 백엔드 API 호출의 정확성; (3) 답변의 친절함과 전문성; (4) 예외적인 상황을 처리하는 견고성. 각 기능에 대해 적절한 평가 지표와 방법을 선택하거나 설계하십시오.
   - 12.1.1절에서는 에이전트 평가가 "출력 불확실성", "평가 표준 다양성", "높은 평가 비용"이라는 세 가지 주요 과제에 직면해 있다고 언급했습니다. 각 과제에 대한 구체적인 솔루션을 제안하고 솔루션의 타당성과 한계를 분석하십시오.

2. BFCL(Berkeley Function Calling Leaderboard)은 도구 호출 기능을 평가하는 중요한 벤치마크입니다. 섹션 12.2 내용을 바탕으로 깊이 생각해 보십시오.

> **힌트**: 실습 문제이므로 실제 작동을 권장합니다.

- 12.2.3절의 AST 매칭 알고리즘에서는 추상 구문 트리를 비교하여 함수 호출이 올바른지 판단합니다. 분석해 보십시오. 단순 문자열 일치보다 AST 일치가 더 적합한 이유는 무엇입니까? 어떤 상황에서 AST 일치가 잘못된 판단(위양성 또는 위음성)을 초래할 수 있나요? 정확도를 높이기 위해 AST 매칭 알고리즘을 개선하는 방법은 무엇입니까?
   - BFCL 데이터 세트에는 단순, 다중, 병렬, 무관의 네 가지 범주가 포함됩니다. 해당 카테고리에서 경계 사례 또는 오류가 발생하기 쉬운 시나리오를 테스트하는 기능이 필요한 각 카테고리에 대해 2-3개의 새로운 테스트 샘플을 설계하십시오.
   - 섹션 12.2.4의 코드를 기반으로 BFCL 평가기를 확장하여 다음 기능을 추가하십시오. (1) 도구 호출의 실행 순서 평가 지원(종속성이 있는 여러 도구 호출의 경우); (2) 도구 호출 효율성(예: 최소 호출 횟수가 사용되었는지 여부)을 평가합니다. (3) 자세한 오류 분석 보고서(가장 일반적인 오류 유형 등)를 생성합니다.

3. GAIA(General AI Assistants)는 에이전트의 종합적인 기능을 평가합니다. 섹션 12.3 내용을 바탕으로 다음 확장 실습을 완료하세요.

> **힌트**: 실습 문제이므로 실제 작동을 권장합니다.

- 섹션 12.3.2에서는 GAIA (Level 1/2/3)의 세 가지 난이도가 도입되었습니다. 분석해 보십시오: 작업 복잡성, 필요한 역량, 평가 표준 등에서 이 세 가지 수준의 차이점은 무엇입니까? 레벨 4(초고난이도)를 설계하는 경우 어떤 유형의 작업을 포함해야 합니까?
   - GAIA는 "Quasi Exact Match" 알고리즘을 사용하여 답변의 정확성을 평가합니다. 분석해 보세요. 이 방법은 답변 다양성 (such as "42", "forty-two", "42.0" should all be considered correct)을 어떻게 처리합니까? 어떤 상황에서 준정확 일치만으로는 충분하지 않을 수 있나요? 의미상 동일한 답변을 처리할 수 있는 보다 지능적인 답변 일치 알고리즘을 설계하십시오.
   - 섹션 12.3.4의 코드를 기반으로 "맞춤형 GAIA 평가 세트"를 구현하십시오. 특정 영역(예: 의료, 법률, 금융)을 선택하고 10개의 실제 질문을 설계하고 전체 평가 흐름을 구현하십시오. 다양한 난이도를 다루는 질문을 요구하고 표준 답변과 채점 기준을 제공합니다.

4. LLM 판사는 평가를 위해 대규모 언어 모델을 사용하는 새로운 방법입니다. 섹션 12.4 내용을 바탕으로 심층 분석해 보십시오.

- 섹션 12.4.2에서는 에이전트 응답 품질을 평가하기 위해 GPT-4를 판사로 사용했습니다. 분석해 보십시오: LLM Judge는 기존 규칙 일치 또는 측정항목 계산과 비교하여 어떤 이점이 있습니까? 어떤 잠재적인 편향이나 한계가 있습니까(예: 특정 응답 스타일에 대한 선호, 길이에 대한 민감도)?
   - LLM 판사 채점 기준 설계가 중요합니다. 다음 세 가지 평가 시나리오에 대한 자세한 채점 기준(점수 차원, 가중치, 예 포함)을 설계하십시오. (1) 코드 생성 품질 평가; (2) 창의적 글쓰기 품질 평가; (3) 기술 문서 품질 평가.
   - 섹션 12.4.3에서는 "배심원 스타일" 평가를 위해 여러 LLM 심사위원을 사용할 수 있다고 언급되었습니다. "다중 심판 평가 시스템"을 설계해 주십시오. 3~5명의 서로 다른 LLM(예: GPT-4, Claude, Qwen)을 심사위원으로 사용하여 점수를 집계하는 방법은 무엇입니까? 판사들 사이의 의견 차이는 어떻게 처리하나요? 비정상적인 점수를 감지하고 필터링하는 방법은 무엇입니까?

5. 대리인 평가의 실제 적용은 여러 측면을 고려해야 합니다. 생각해 보십시오:

- 실제 프로젝트에서는 평가 시 '평가 비용'과 '평가 품질'의 균형이 필요한 경우가 많습니다. "계층형 평가 전략"을 설계하십시오: (1) 빠른 평가(낮은 비용, 일일 개발 반복용); (2) 표준 평가(중간 비용, 시험판의 경우); (3) 종합적인 평가(주요 업데이트 또는 공개 릴리스의 경우 높은 비용). 각 계층에는 어떤 평가 항목이 포함되어야 하나요? 평가 흐름을 어떻게 설계하나요?
   - 에이전트 성능은 시간이 지남에 따라 변경될 수 있습니다(예: 종속 외부 API 변경, 사용자 요구 사항 변경). 정기적으로 자동 평가를 실행하고 에이전트 성능 변화 추세를 모니터링하며 성능이 저하될 때 적시에 경고할 수 있는 "지속적인 평가 시스템"을 설계하십시오. 이 시스템에는 어떤 구성 요소가 포함되어야 합니까? 경고 규칙을 디자인하는 방법은 무엇입니까?
   - 평가 결과는 다양한 청중(예: 개발자, 제품 관리자, 사용자)에게 명확하게 제시되어야 합니다. 대상 유형에 따라 다양한 세부 수준의 보고서를 자동으로 생성할 수 있는 "평가 보고서 생성 시스템"을 설계하십시오. 개발자 보고서에는 어떤 기술적 세부정보가 포함되어야 하나요? 제품 관리자 보고서에서는 어떤 비즈니스 지표를 강조해야 합니까? 사용자 보고서를 어떻게 단순화하고 시각화해야 합니까?

## 참고자료

[1] Patil, S. G., Zhang, T., Wang, X., & Gonzalez, J. E. (2023). Gorilla: 대규모 API와 연결된 대규모 언어 모델. arXiv 사전 인쇄 arXiv:2305.15334.

[2] Qin, Y., Liang, S., Ye, Y., Zhu, K., Yan, L., Lu, Y., ... & Sun, M. (2023). ToolLLM: 16000개 이상의 실제 API를 마스터하기 위한 대규모 언어 모델 촉진. arXiv 사전 인쇄 arXiv:2307.16789.

[3] Li, M., Zhao, Y., Yu, B., Song, F., Li, H., Yu, H., ... & Li, Y. (2023). Api-bank: 도구로 강화된 LLM에 대한 포괄적인 벤치마크입니다. arXiv 사전 인쇄 arXiv:2304.08244.

[4] Mialon, G., Dessì, R., Lomeli, M., Nalmpantis, C., Pasunuru, R., Raileanu, R., ... & Scialom, T. (2023). GAIA: 일반 AI 보조자에 대한 벤치마크입니다. arXiv 사전 인쇄 arXiv:2311.12983.

[5] Liu, X., Yu, H., Zhang, H., Xu, Y., Lei, X., Lai, H., ... & Zhang, D. (2023). AgentBench: LLM을 에이전트로 평가합니다. arXiv 사전 인쇄 arXiv:2308.03688.

[6] Zhou, S., Xu, F. F., Zhu, H., Zhou, X., Lo, R., Sridhar, A., ... & Neubig, G. (2023). WebArena: 자율 에이전트 구축을 위한 현실적인 웹 환경. arXiv 사전 인쇄 arXiv:2307.13854.

[7] Chan, C. M., Chen, W., Su, Y., Yu, J., Xue, W., Zhang, S., ... & Liu, Z. (2023). ChatEval: 다중 에이전트 토론을 통해 더 나은 LLM 기반 평가자를 향하여. arXiv 사전 인쇄 arXiv:2308.07201.

[8] Zhou, X., Zhu, H., Mathur, L., Zhang, R., Yu, H., Qi, Z., ... & Neubig, G. (2023). SOTOPIA: 언어 에이전트의 사회 지능에 대한 대화형 평가. arXiv 사전 인쇄 arXiv:2310.11667.

[9] 미국 수학 협회. (2024). 미국 초청 수학 시험(AIME). https://www.maa.org/math-competitions/invitational-competitions/aime에서 검색됨

