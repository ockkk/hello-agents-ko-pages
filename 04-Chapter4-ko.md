# 4장: 클래식 에이전트 패러다임 구축

이전 장에서 우리는 현대 에이전트의 "두뇌"인 대규모 언어 모델을 깊이 탐구했습니다. 내부 Transformer 아키텍처, 상호 작용 방법 및 기능 경계에 대해 배웠습니다. 이제 이 이론적 지식을 실습으로 전환하고 직접 에이전트를 구축할 때입니다.

현대 에이전트의 핵심 기능은 대규모 언어 모델의 추론 능력을 외부 세계와 연결하는 능력에 있습니다. 코드 해석기, 검색 엔진, API 등 일련의 '도구'를 호출하여 정보를 얻고 작업을 실행함으로써 자동으로 사용자 의도를 이해하고 복잡한 작업을 분해하며 목표를 달성할 수 있습니다. 그러나 에이전트는 전능하지 않습니다. 또한 에이전트의 기능 경계를 구성하는 대규모 모델에 내재된 "환각" 문제, 복잡한 작업의 잠재적인 추론 루프, 잘못된 도구 사용으로 인한 문제에 직면합니다.

에이전트의 "사고" 및 "행동" 프로세스를 더 잘 구성하기 위해 업계에서는 여러 가지 고전적인 아키텍처 패러다임이 등장했습니다. 이 장에서는 가장 대표적인 세 가지에 중점을 두고 처음부터 단계별로 구현해 보겠습니다.

- **ReAct(Reasoning and Acting):** "생각"과 "행동"을 긴밀하게 결합한 패러다임으로 에이전트가 행동하면서 생각하고 동적으로 조정할 수 있습니다.
- **계획 및 해결:** 상담원이 먼저 완전한 실행 계획을 생성한 다음 엄격하게 실행하는 "행동하기 전에 생각하기" 패러다임입니다.
- **반성:** 상담원에게 "반성" 기능을 부여하고 자기 비판과 교정을 통해 결과를 최적화하는 패러다임입니다.

이러한 사항을 이해한 후에는 다음과 같이 질문할 수 있습니다. 이미 사용 가능한 LangChain 및 LlamaIndex와 같은 훌륭한 프레임워크가 있는데 왜 "바퀴를 재발명"해야 합니까? 대답은 성숙한 프레임워크가 엔지니어링 효율성 측면에서 상당한 이점을 갖고 있지만 고도로 추상화된 도구를 직접 사용하는 것은 기본 설계 메커니즘이 작동하는 방식이나 제공되는 이점을 이해하는 데 도움이 되지 않는다는 사실에 있습니다. 둘째, 이 프로세스는 프로젝트의 엔지니어링 문제를 노출시킵니다. 프레임워크는 모델 출력 형식 구문 분석, 실패한 도구 호출 재시도, 에이전트가 무한 루프에 빠지는 것을 방지하는 등 많은 문제를 처리합니다. 이러한 문제를 직접 처리하는 것이 시스템 설계 역량을 키우는 가장 직접적인 방법입니다. 마지막으로 가장 중요한 것은 디자인 원칙을 완전히 익히면 프레임워크 "사용자"에서 지능형 애플리케이션 "작성자"로 진정한 변화를 이룰 수 있다는 것입니다. 표준 구성 요소가 복잡한 요구 사항을 충족할 수 없는 경우 심층적으로 사용자 정의하거나 처음부터 완전히 새로운 에이전트를 구축할 수도 있습니다.

## 4.1 환경 준비 및 기본 도구 정의

빌드를 시작하기 전에 개발 환경을 설정하고 몇 가지 기본 구성요소를 정의해야 합니다. 이는 나중에 다른 패러다임을 구현할 때 반복적인 작업을 피하고 핵심 로직에 더 집중하는 데 도움이 될 것입니다.

### 4.1.1 종속성 설치

이 책의 실무 부분에서는 Python 언어를 주로 사용하며, Python 3.10 이상을 권장합니다. 먼저, 대규모 언어 모델과 상호작용하기 위한 'openai' 라이브러리와 API 키를 안전하게 관리하기 위한 'python-dotenv' 라이브러리를 설치했는지 확인하세요.

터미널에서 다음 명령을 실행하세요.

```bash
pip install openai python-dotenv
```

### 4.1.2 API 키 구성

코드를 보다 보편적으로 만들기 위해 환경 변수에 모델 서비스 관련 정보(모델 ID, API 키, 서비스 주소)를 일률적으로 구성하겠습니다.

1. 프로젝트 루트 디렉터리에 '.env'라는 파일을 생성합니다.
2. 이 파일에 다음 내용을 추가합니다. 필요에 따라 OpenAI의 공식 서비스 또는 OpenAI 인터페이스와 호환되는 로컬/타사 서비스를 가리킬 수 있습니다.
3. 정말 어떻게 구해야 할지 모르겠다면 [환경 설정](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra07-环境配置.md)을 참고하세요.

```bash
# .env file
LLM_API_KEY="YOUR-API-KEY"
LLM_MODEL_ID="YOUR-MODEL"
LLM_BASE_URL="YOUR-URL"
```

우리 코드는 이 파일에서 이러한 구성을 자동으로 로드합니다.

### 4.1.3 기본 LLM 호출 기능 캡슐화

코드 구조를 더 명확하고 재사용 가능하게 만들기 위해 전용 LLM 클라이언트 클래스를 정의해 보겠습니다. 이 클래스는 모델 서비스와의 상호 작용에 대한 모든 세부 정보를 캡슐화하여 기본 논리가 에이전트 구성에 더 집중할 수 있도록 합니다.

```python
import os
from openai import OpenAI
from dotenv import load_dotenv
from typing import List, Dict

# Load environment variables from .env file
load_dotenv()

class HelloAgentsLLM:
    """
    A customized LLM client for the book "Hello Agents".
    It is used to call any service compatible with the OpenAI interface and uses streaming responses by default.
    """
    def __init__(self, model: str = None, apiKey: str = None, baseUrl: str = None, timeout: int = None):
        """
        Initialize the client. Prioritize passed parameters; if not provided, load from environment variables.
        """
        self.model = model or os.getenv("LLM_MODEL_ID")
        apiKey = apiKey or os.getenv("LLM_API_KEY")
        baseUrl = baseUrl or os.getenv("LLM_BASE_URL")
        timeout = timeout or int(os.getenv("LLM_TIMEOUT", 60))
        
        if not all([self.model, apiKey, baseUrl]):
            raise ValueError("Model ID, API key, and service address must be provided or defined in the .env file.")

        self.client = OpenAI(api_key=apiKey, base_url=baseUrl, timeout=timeout)

    def think(self, messages: List[Dict[str, str]], temperature: float = 0) -> str:
        """
        Call the large language model to think and return its response.
        """
        print(f"🧠 Calling {self.model} model...")
        try:
            response = self.client.chat.completions.create(
                model=self.model,
                messages=messages,
                temperature=temperature,
                stream=True,
            )
            
            # Handle streaming response
            print("✅ Large language model response successful:")
            collected_content = []
            for chunk in response:
                if not chunk.choices:
                    continue
                content = chunk.choices[0].delta.content or ""
                print(content, end="", flush=True)
                collected_content.append(content)
            print()  # Newline after streaming output ends
            return "".join(collected_content)

        except Exception as e:
            print(f"❌ Error occurred when calling LLM API: {e}")
            return None

# --- Client Usage Example ---
if __name__ == '__main__':
    try:
        llmClient = HelloAgentsLLM()
        
        exampleMessages = [
            {"role": "system", "content": "You are a helpful assistant that writes Python code."},
            {"role": "user", "content": "Write a quicksort algorithm"}
        ]
        
        print("--- Calling LLM ---")
        responseText = llmClient.think(exampleMessages)
        if responseText:
            print("\n\n--- Complete Model Response ---")
            print(responseText)

    except ValueError as e:
        print(e)


>>>
--- Calling LLM ---
🧠 Calling xxxxxx model...
✅ Large language model response successful:
Quicksort is a very efficient sorting algorithm...
```



## 4.2 리액트

After preparing the LLM client, we will build the first and most classic agent paradigm: **ReAct (Reason + Act)**. ReAct was proposed by Shunyu Yao in 2022<sup>[1]</sup>. Its core idea is to mimic how humans solve problems by explicitly combining **Reasoning** and **Acting** to form a "think-act-observe" loop.

### 4.2.1 ReAct 작업 흐름

ReAct가 등장하기 전에 주류 방법은 두 가지 범주로 나눌 수 있었습니다. 하나는 모델이 복잡한 논리적 추론을 수행하도록 안내할 수 있지만 외부 세계와 상호 작용할 수 없고 사실적 환각을 일으키기 쉬운 **사고 사슬**과 같은 "순수한 사고" 유형입니다. 다른 하나는 모델이 실행할 작업을 직접 출력하지만 계획 및 오류 수정 기능이 부족한 "순수 작업" 유형입니다.

ReAct의 독창성은 **생각과 행동이 상호 보완적**이라는 점을 인식하는 데 있습니다. 생각은 행동을 이끌고, 행동은 올바른 생각을 낳습니다. 이를 위해 ReAct 패러다임은 특별한 프롬프트 엔지니어링을 사용하여 출력의 각 단계가 고정된 궤적을 따르도록 모델을 안내합니다.

- **생각(Thinking):** 에이전트의 '내면의 독백'입니다. 현재 상황을 분석하고, 업무를 분해하고, 다음 계획을 수립하거나, 이전 단계의 결과를 반영합니다.
- **Action (Acting):** 상담원이 취하기로 결정한 구체적인 액션으로, 일반적으로 'Search['Huawei's late Phone']'과 같은 외부 도구를 호출합니다.
- **Observing(Observing):** 검색 결과 요약이나 API 반환 값 등 `Action`을 실행한 후 외부 도구에서 반환되는 결과입니다.

에이전트는 이 **생각 -> 동작 -> 관찰** 루프를 계속 반복하여 기록에 새로운 관찰 결과를 추가하여 '생각'에서 최종 답을 찾았다고 판단하고 결과를 출력할 때까지 지속적으로 성장하는 컨텍스트를 형성합니다. 이 프로세스는 강력한 시너지 효과를 형성합니다. **추론은 행동을 더욱 목적 있게 만들고, 행동은 추론을 위한 사실 기반을 제공합니다.**

그림 4.1과 같이 이 프로세스를 공식적으로 표현할 수 있습니다. 특히, 각 시간 단계 $t$에서 에이전트의 정책(예: 대규모 언어 모델 $\pi$)은 초기 질문 $q$와 모든 이전 "작업 관찰" 단계 $((a_1,o_1),\dots,(a_{t-1},o_{t-1}))$의 기록 궤적을 기반으로 현재 생각 $th_t$ 및 작업 $a_t$를 생성합니다.

$$\left(th_t,a_t\right)=\pi\left(q,(a_1,o_1),\ldots,(a_{t-1},o_{t-1})\right)$$

이어서 환경의 $T$ 도구는 $a_t$ 작업을 실행하고 새로운 관찰 결과 $o_t$를 반환합니다.

$$o_t = T(a_t)$$

이 루프는 모델이 $th_t$ 작업이 완료되었다고 판단할 때까지 새로운 $(a_t,o_t)$ 쌍을 기록에 추가하면서 계속됩니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/4-figures/4-1.png" alt="Think-Act-Observe synergistic loop in ReAct paradigm" width="90%"/>
  <p>그림 4.1 ReAct 패러다임의 Think-Act-Observe 시너지 루프</p>
</div>

이 메커니즘은 특히 다음 시나리오에 적합합니다.

- **외부 지식이 필요한 업무** : 실시간 정보(날씨, 뉴스, 주가) 조회, 전문 분야 지식 검색 등
- **정확한 계산이 필요한 작업**: LLM 계산 오류를 방지하기 위해 수학 문제를 계산기 도구에 위임합니다.
- **API 상호작용이 필요한 작업**: 데이터베이스 운영, 특정 기능을 완료하기 위한 서비스의 API 호출 등.

따라서 우리는 대규모 언어 모델이 자체 지식 기반만으로는 직접 답변할 수 없는 질문에 답변하기 위해 **외부 도구를 사용**하는 기능을 갖춘 ReAct 에이전트를 구축할 것입니다. 예: "화웨이의 최신 휴대폰은 무엇입니까? 주요 판매 포인트는 무엇입니까?" 이 질문을 하려면 상담원은 온라인으로 검색하고, 결과를 검색하기 위한 도구를 호출하고, 답변을 요약해야 한다는 점을 이해해야 합니다.

### 4.2.2 도구 정의 및 구현

대규모 언어 모델이 에이전트의 두뇌라면 **도구**는 외부 세계와 상호작용하기 위한 '손과 발'입니다. ReAct 패러다임을 사용하여 우리가 설정한 문제를 실제로 해결하려면 에이전트에 외부 도구를 호출하는 기능이 필요합니다.

이 섹션에서 설정한 목표("Huawei의 최신 휴대폰"에 대한 질문에 답변)를 위해 상담원에게 웹 검색 도구를 제공해야 합니다. 여기서는 API를 통해 구조화된 Google 검색 결과를 제공하고 "답변 요약 상자" 또는 정확한 지식 그래프 정보를 직접 반환할 수 있는 **SerpApi**를 선택합니다.

먼저 라이브러리를 설치해야 합니다.

```bash
pip install google-search-results
```

동시에 [SerpApi 공식 웹사이트](https://serpapi.com/)로 이동하여 무료 계정을 등록하고 API 키를 얻은 후 프로젝트 루트 디렉터리의 `.env` 파일에 추가해야 합니다.

```bash
# .env file
# ... (Keep previous LLM configuration)
SERPAPI_API_KEY="YOUR_SERPAPI_API_KEY"
```

다음으로 코드를 통해 이 도구를 정의하고 관리하겠습니다. 우리는 단계별로 진행할 것입니다. 먼저 도구의 핵심 기능을 구현한 다음 일반 도구 관리자를 구축합니다.

(1) 검색 도구의 핵심 로직 구현

잘 정의된 도구에는 다음 세 가지 핵심 요소가 포함되어야 합니다.

1. **이름**: '검색'과 같은 '작업'에서 호출할 에이전트의 간결하고 고유한 식별자입니다.
2. **설명**: 이 도구의 목적을 설명하는 명확한 자연어 설명입니다. **이것은 전체 메커니즘에서 가장 중요한 부분**입니다. 왜냐하면 대규모 언어 모델은 이 설명에 의존하여 언제 어떤 도구를 사용할지 결정하기 때문입니다.
3. **실행 로직**: 실제로 작업을 수행하는 함수 또는 메서드입니다.

첫 번째 도구는 쿼리 문자열을 받은 다음 검색 결과를 반환하는 '검색' 기능입니다.

```python
from serpapi import SerpApiClient

def search(query: str) -> str:
    """
    A practical web search engine tool based on SerpApi.
    It intelligently parses search results, prioritizing direct answers or knowledge graph information.
    """
    print(f"🔍 Executing [SerpApi] web search: {query}")
    try:
        api_key = os.getenv("SERPAPI_API_KEY")
        if not api_key:
            return "Error: SERPAPI_API_KEY not configured in .env file."

        params = {
            "engine": "google",
            "q": query,
            "api_key": api_key,
            "gl": "cn",  # Country code
            "hl": "zh-cn", # Language code
        }
        
        client = SerpApiClient(params)
        results = client.get_dict()
        
        # Intelligent parsing: prioritize finding the most direct answer
        if "answer_box_list" in results:
            return "\n".join(results["answer_box_list"])
        if "answer_box" in results and "answer" in results["answer_box"]:
            return results["answer_box"]["answer"]
        if "knowledge_graph" in results and "description" in results["knowledge_graph"]:
            return results["knowledge_graph"]["description"]
        if "organic_results" in results and results["organic_results"]:
            # If no direct answer, return summaries of the first three organic results
            snippets = [
                f"[{i+1}] {res.get('title', '')}\n{res.get('snippet', '')}"
                for i, res in enumerate(results["organic_results"][:3])
            ]
            return "\n\n".join(snippets)
        
        return f"Sorry, no information found about '{query}'."

    except Exception as e:
        return f"Error occurred during search: {e}"
```

위 코드에서는 먼저 'answer_box'(구글의 답변 요약 상자) 또는 'knowledge_graph'(지식 그래프) 정보가 있는지 확인합니다. 그렇다면 가장 정확한 답변을 직접 반환합니다. 그렇지 않은 경우 처음 세 개의 일반 검색 결과에 대한 요약을 반환합니다. 이 "지능형 구문 분석"은 LLM에 대한 고품질 정보 입력을 제공할 수 있습니다.

(2) 일반 도구 실행자 구축

에이전트가 여러 도구를 사용해야 하는 경우(예: 검색 외에도 계산, 데이터베이스 쿼리 등이 필요할 수 있음) 이러한 도구를 등록하고 파견하려면 통합 관리자가 필요합니다. 이를 위해 `ToolExecutor` 클래스를 만듭니다.

```python
from typing import Dict, Any

class ToolExecutor:
    """
    A tool executor responsible for managing and executing tools.
    """
    def __init__(self):
        self.tools: Dict[str, Dict[str, Any]] = {}

    def registerTool(self, name: str, description: str, func: callable):
        """
        Register a new tool in the toolbox.
        """
        if name in self.tools:
            print(f"Warning: Tool '{name}' already exists and will be overwritten.")
        self.tools[name] = {"description": description, "func": func}
        print(f"Tool '{name}' registered.")

    def getTool(self, name: str) -> callable:
        """
        Get a tool's execution function by name.
        """
        return self.tools.get(name, {}).get("func")

    def getAvailableTools(self) -> str:
        """
        Get a formatted description string of all available tools.
        """
        return "\n".join([
            f"- {name}: {info['description']}" 
            for name, info in self.tools.items()
        ])

```

(3) 테스트

이제 `ToolExecutor`에 `search` 도구를 등록하고 호출을 시뮬레이션하여 전체 프로세스가 제대로 작동하는지 확인하겠습니다.

```python
# --- Tool Initialization and Usage Example ---
if __name__ == '__main__':
    # 1. Initialize tool executor
    toolExecutor = ToolExecutor()

    # 2. Register our practical search tool
    search_description = "A web search engine. Use this tool when you need to answer questions about current events, facts, and information not found in your knowledge base."
    toolExecutor.registerTool("Search", search_description, search)

    # 3. Print available tools
    print("\n--- Available Tools ---")
    print(toolExecutor.getAvailableTools())

    # 4. Agent's Action call, this time we ask a real-time question
    print("\n--- Execute Action: Search['What is NVIDIA's latest GPU model'] ---")
    tool_name = "Search"
    tool_input = "What is NVIDIA's latest GPU model"

    tool_function = toolExecutor.getTool(tool_name)
    if tool_function:
        observation = tool_function(tool_input)
        print("--- Observation ---")
        print(observation)
    else:
        print(f"Error: Tool named '{tool_name}' not found.")

>>>
Tool 'Search' registered.

--- Available Tools ---
- Search: A web search engine. Use this tool when you need to answer questions about current events, facts, and information not found in your knowledge base.

--- Execute Action: Search['What is NVIDIA's latest GPU model'] ---
🔍 Executing [SerpApi] web search: What is NVIDIA's latest GPU model
--- Observation ---
[1] GeForce RTX 50 Series Graphics Cards
GeForce RTX™ 50 Series GPUs are powered by NVIDIA Blackwell architecture, bringing new gameplay for gamers and creators. RTX 50 Series has powerful AI computing power, bringing upgraded experience and more realistic graphics.

[2] Compare GeForce Series Latest Generation and Previous Generation Graphics Cards
Compare the latest RTX 30 series graphics cards with previous RTX 20 series, GTX 10 and 900 series graphics cards. View specifications, features, technical support, etc.

[3] GeForce Graphics Cards | NVIDIA
DRIVE AGX. Powerful in-vehicle computing power for AI-driven intelligent vehicle systems · Clara AGX. AI computing for innovative medical devices and imaging. Gaming and Creation. GeForce. Explore graphics cards, gaming solutions, AI ...
```

지금까지 우리는 실제 인터넷에 연결하는 '검색' 도구를 에이전트에 장착하여 후속 ReAct 루프를 위한 견고한 기반을 제공했습니다.



### 4.2.3 ReAct Agent 코딩 구현

이제 LLM 클라이언트와 도구 실행기 등 모든 독립 구성 요소를 조립하여 완전한 ReAct 에이전트를 구축하겠습니다. ReActAgent 클래스를 통해 핵심 로직을 캡슐화하겠습니다. 이해를 돕기 위해 이 클래스의 구현 프로세스를 다음과 같은 주요 부분으로 나누어 설명하겠습니다.

(1) 시스템 프롬프트 디자인

프롬프트는 전체 ReAct 메커니즘의 초석이며 대규모 언어 모델에 대한 운영 지침을 제공합니다. 사용 가능한 도구, 사용자 질문, 중간 단계의 상호 작용 기록을 동적으로 삽입하는 템플릿을 신중하게 디자인해야 합니다.

```bash
# ReAct Prompt Template
REACT_PROMPT_TEMPLATE = """
Please note that you are an intelligent assistant capable of calling external tools.

Available tools are as follows:
{tools}

Please respond strictly in the following format:

Thought: Your thinking process, used to analyze problems, decompose tasks, and plan the next action.
Action: The action you decide to take, must be in one of the following formats:
- {{tool_name}}[{{tool_input}}]`: Call an available tool.
- `Finish[final answer]`: When you believe you have obtained the final answer.
- When you have collected enough information to answer the user's final question, you must use `Finish[final answer]` after the Action: field to output the final answer.

Now, please start solving the following problem:
Question: {question}
History: {history}
"""
```

이 템플릿은 상담원과 LLM 간의 상호 작용 사양을 정의합니다.

- **역할 정의**: "당신은 외부 도구를 호출할 수 있는 지능형 보조자입니다."는 LLM의 역할을 설정합니다.
- **도구 목록(`{tools}`)**: LLM에 사용 가능한 "손과 발"이 무엇인지 알려줍니다.
- **형식 규칙(`Thought`/`Action`)**: 이는 가장 중요한 부분으로, LLM의 출력을 구조화하여 코드를 통해 그 의도를 정확하게 구문 분석할 수 있도록 합니다.
- **동적 컨텍스트(`{question}`/`{history}`)**: 사용자의 원래 질문과 지속적으로 축적된 상호 작용 내역을 주입하여 LLM이 완전한 컨텍스트를 기반으로 결정을 내릴 수 있도록 합니다.

(2) 코어 루프 구현

ReActAgent의 핵심은 작업이 완료되거나 최대 단계 제한에 도달할 때까지 "프롬프트 형식 지정 -> LLM 호출 -> 작업 실행 -> 결과 통합"을 지속적으로 수행하는 루프입니다.

```python
class ReActAgent:
    def __init__(self, llm_client: HelloAgentsLLM, tool_executor: ToolExecutor, max_steps: int = 5):
        self.llm_client = llm_client
        self.tool_executor = tool_executor
        self.max_steps = max_steps
        self.history = []

    def run(self, question: str):
        """
        Run the ReAct agent to answer a question.
        """
        self.history = [] # Reset history for each run
        current_step = 0

        while current_step < self.max_steps:
            current_step += 1
            print(f"--- Step {current_step} ---")

            # 1. Format prompt
            tools_desc = self.tool_executor.getAvailableTools()
            history_str = "\n".join(self.history)
            prompt = REACT_PROMPT_TEMPLATE.format(
                tools=tools_desc,
                question=question,
                history=history_str
            )

            # 2. Call LLM to think
            messages = [{"role": "user", "content": prompt}]
            response_text = self.llm_client.think(messages=messages)

            if not response_text:
                print("Error: LLM failed to return a valid response.")
                break

            # ... (Subsequent parsing, execution, integration steps)

```

`run` 메소드는 에이전트의 진입점입니다. 'while' 루프는 ReAct 패러다임의 본체를 구성하며, 'max_steps' 매개변수는 에이전트가 무한 루프에 빠지고 리소스가 고갈되는 것을 방지하는 중요한 안전 밸브입니다.

(3) 출력 파서 구현

LLM은 일반 텍스트를 반환하므로 여기서 '생각'과 '행동'을 정확하게 추출해야 합니다. 이는 일반적으로 정규식을 사용하는 여러 보조 구문 분석 기능을 통해 수행됩니다.

```python
# (These methods are part of the ReActAgent class)
    def _parse_output(self, text: str):
        """Parse LLM output to extract Thought and Action.
        """
        # Thought: match until Action: or end of text
        thought_match = re.search(r"Thought:\s*(.*?)(?=\nAction:|$)", text, re.DOTALL)
        # Action: match until end of text
        action_match = re.search(r"Action:\s*(.*?)$", text, re.DOTALL)
        thought = thought_match.group(1).strip() if thought_match else None
        action = action_match.group(1).strip() if action_match else None
        return thought, action

    def _parse_action(self, action_text: str):
        """Parse Action string to extract tool name and input.
        """
        match = re.match(r"(\w+)\[(.*)\]", action_text, re.DOTALL)
        if match:
            return match.group(1), match.group(2)
        return None, None
```

- `_parse_output`: LLM의 전체 응답에서 `Thought`와 `Action` 두 가지 주요 부분을 분리하는 역할을 담당합니다.
- `_parse_action`: `Action` 문자열을 추가로 구문 분석하는 역할을 담당합니다. 예를 들어 도구 이름 `Search`를 추출하고 `Search[Huawei's 최신 전화]`에서 도구 입력 `Huawei's 최신 전화`를 추출합니다.

(4) 도구 호출 및 실행

```python
# (This logic is inside the while loop of the run method)
            # 3. Parse LLM output
            thought, action = self._parse_output(response_text)

            if thought:
                print(f"Thought: {thought}")

            if not action:
                print("Warning: Failed to parse valid Action, process terminated.")
                break

            # 4. Execute Action
            if action.startswith("Finish"):
                # If it's a Finish instruction, extract the final answer and end
                final_answer = re.match(r"Finish\[(.*)\]", action).group(1)
                print(f"🎉 Final Answer: {final_answer}")
                return final_answer

            tool_name, tool_input = self._parse_action(action)
            if not tool_name or not tool_input:
                # ... Handle invalid Action format ...
                continue

            print(f"🎬 Action: {tool_name}[{tool_input}]")

            tool_function = self.tool_executor.getTool(tool_name)
            if not tool_function:
                observation = f"Error: Tool named '{tool_name}' not found."
            else:
                observation = tool_function(tool_input) # Call real tool

```

이 코드는 `Action`의 실행 중심입니다. 먼저 'Finish' 명령인지 확인합니다. 그렇다면 프로세스가 종료됩니다. 그렇지 않으면 `tool_executor`를 통해 해당 도구 기능을 얻은 후 이를 실행하여 `observation`을 얻습니다.

(5) 관찰 결과의 통합

닫힌 루프를 형성하는 마지막 단계이자 핵심은 도구 실행 후 'Action' 자체와 'Observation'을 기록에 다시 추가하여 다음 루프에 대한 새로운 컨텍스트를 제공하는 것입니다.

```python
# (This logic follows tool invocation, at the end of the while loop)
            print(f"👀 Observation: {observation}")

            # Add this round's Action and Observation to history
            self.history.append(f"Action: {action}")
            self.history.append(f"Observation: {observation}")

        # Loop ends
        print("Maximum steps reached, process terminated.")
        return None
```

`self.history`에 `Observation`을 추가함으로써 에이전트는 다음 라운드에서 프롬프트를 생성할 때 이전 작업의 결과를 "보고" 그에 따라 새로운 사고와 계획을 수행할 수 있습니다.

(6) 인스턴스 실행 및 분석

위의 모든 부분을 결합하여 완전한 `ReActAgent` 클래스를 얻습니다. 전체 코드 실행 인스턴스는 이 책과 함께 제공되는 코드 저장소의 `code` 폴더에서 찾을 수 있습니다.

아래는 실제 실행 기록입니다.

```
Tool 'Search' registered.

--- Step 1 ---
🧠 Calling xxxxxx model...
✅ Large language model response successful:
Thought: To answer this question, I need to search for Huawei's latest released phone model and its main features. This information may be outside my existing knowledge base, so I need to use a search engine to obtain the latest data.
Action: Search[Huawei latest phone model and main selling points]
🤔 Thought: To answer this question, I need to search for Huawei's latest released phone model and its main features. This information may be outside my existing knowledge base, so I need to use a search engine to obtain the latest data.
🎬 Action: Search[Huawei latest phone model and main selling points]
🔍 Executing [SerpApi] web search: Huawei latest phone model and main selling points
👀 Observation: [1] Huawei Phones - Huawei Official Website
Smartphones ; Mate Series. Extraordinary Flagship · HUAWEI Mate XTs. Extraordinary Master ; Pura Series. Pioneer Imaging · HUAWEI Pura 80 Pro+ ; Pocket Series. Aesthetic New Chapter. HUAWEI Pocket 2 ; nova Series. Professional Portrait.

[2] Which Huawei phone has the best value in 2025? Huawei phone recommendations and market analysis ...
Now the biggest selling point of Huawei phones is only the HarmonyOS system, and the controversial brand faith. Here are several Huawei series phones worth buying, choose according to different budgets:. Huawei's currently most popular, also equipped with ...

[3] Which new Huawei phone has the best value in 2025? 10 new Huawei phone recommendations
Choosing Huawei mainly recommends high-end phones, Mate 70 and Pura 70 series are the latest flagship models. HUAWEI Mate 70. Advantages are, photography configuration is still top-level, full focal length coverage, suitable for professional photography, excellent workmanship, outdoor drop resistance ...

--- Step 2 ---
🧠 Calling xxxxxx model...
✅ Large language model response successful:
Thought: Based on the search results, Huawei's latest flagship models include Mate 70 and Pura 80 Pro+. To determine the latest model and its main selling points, I will focus on this information. From the provided links, both Mate 70 series and Pura 80 Pro+ are recently released products, but which one is "latest" needs further confirmation. Meanwhile, I can extract their main selling points from this information.
Action: Finish[According to the latest information, Huawei's latest phones may be HUAWEI Pura 80 Pro+ or HUAWEI Mate 70. Among them, HUAWEI Mate 70's main selling points include top-level photography configuration, full focal length coverage, suitable for professional photography, excellent workmanship, and good outdoor drop resistance. While HUAWEI Pura 80 Pro+ emphasizes pioneer imaging technology.]
🤔 Thought: Based on the search results, Huawei's latest flagship models include Mate 70 and Pura 80 Pro+. To determine the latest model and its main selling points, I will focus on this information. From the provided links, both Mate 70 series and Pura 80 Pro+ are recently released products, but which one is "latest" needs further confirmation. Meanwhile, I can extract their main selling points from this information.
🎉 Final Answer: According to the latest information, Huawei's latest phones may be HUAWEI Pura 80 Pro+ or HUAWEI Mate 70. Among them, HUAWEI Mate 70's main selling points include top-level photography configuration, full focal length coverage, suitable for professional photography, excellent workmanship, and good outdoor drop resistance. While HUAWEI Pura 80 Pro+ emphasizes pioneer imaging technology.
```

위의 출력에서 ​​에이전트가 생각의 사슬을 명확하게 보여주는 것을 볼 수 있습니다. 먼저 지식이 부족하여 검색 도구를 사용해야 한다는 것을 깨닫습니다. 그런 다음 검색 결과를 바탕으로 추론하고 요약하여 두 단계 내에 최종 답변에 도달합니다.

모델의 지식과 인터넷 정보는 지속적으로 업데이트되기 때문에, 여러분의 달리기 결과가 이와 정확히 일치하지 않을 수도 있다는 점을 참고하시기 바랍니다. 이 섹션이 작성된 2025년 9월 8일 현재 검색 결과에 언급된 HUAWEI Mate 70 및 HUAWEI Pura 80 Pro+는 실제로 당시 Huawei의 최신 플래그십 시리즈 휴대폰이었습니다. 이는 시간에 민감한 문제를 처리하는 데 있어서 ReAct 패러다임의 강력한 기능을 완전히 보여줍니다.

### 4.2.4 ReAct의 특징, 한계, 디버깅 기법

ReAct 에이전트를 직접 구현함으로써 우리는 워크플로를 마스터했을 뿐만 아니라 내부 메커니즘에 대해서도 더 깊이 이해해야 합니다. 모든 기술 패러다임에는 하이라이트와 개선 영역이 있습니다. 이 섹션에서는 ReAct를 요약합니다.

(1) ReAct의 주요 특징

1. **높은 해석성**: ReAct의 가장 큰 장점 중 하나는 투명성입니다. '생각' 체인을 통해 각 단계에서 에이전트의 '정신적 여정'(이 도구를 선택한 이유와 다음에 수행할 계획)을 명확하게 확인할 수 있습니다. 이는 에이전트 동작을 이해하고, 신뢰하고, 디버깅하는 데 중요합니다.
2. **동적 계획 및 오류 수정 기능**: 한 번에 완전한 계획을 생성하는 패러다임과 달리 ReAct는 "한 단계 진행, 한 단계 살펴보기"입니다. 각 단계에서 외부 세계로부터 얻은 '관찰'을 기반으로 후속 '생각'과 '행동'을 동적으로 조정합니다. 이전 검색 결과가 만족스럽지 못한 경우 다음 단계에서 검색어를 수정하고 다시 시도할 수 있습니다.
3. **도구 시너지 기능**: ReAct 패러다임은 대규모 언어 모델의 추론 기능과 외부 도구의 실행 기능을 자연스럽게 결합합니다. LLM은 전략 수립(계획 및 추론)을 담당하고, 도구는 특정 문제 해결(검색, 계산)을 담당하며, 두 가지 작업이 시너지 효과를 발휘하여 지식 적시성, 계산 정확성 등에서 단일 LLM의 본질적인 한계를 극복합니다.

(2) ReAct의 본질적인 한계

1. **LLM 자체 기능에 대한 강한 의존성**: ReAct 프로세스의 성공은 기본 LLM의 포괄적인 기능에 크게 좌우됩니다. LLM의 논리적 추론 능력, 지시 따르기 능력, 형식화된 출력 능력이 부족하면 '생각' 단계에서 잘못된 계획을 세우거나 '행동' 단계에서 형식에 맞지 않는 지시를 생성하여 전체 프로세스가 중단되기 쉽습니다.
2. **실행 효율성 문제**: 단계별 특성으로 인해 작업을 완료하려면 일반적으로 여러 LLM 호출이 필요합니다. 각 호출에는 네트워크 대기 시간과 계산 비용이 수반됩니다. 많은 단계가 필요한 복잡한 작업의 경우 이 연속적인 "생각-행동" 루프로 인해 총 시간과 비용이 높아질 수 있습니다.
3. **프롬프트 취약성**: 전체 메커니즘의 안정적인 작동은 신중하게 설계된 프롬프트 템플릿을 기반으로 구축되었습니다. 템플릿의 사소한 변경, 심지어 문구의 차이도 LLM 동작에 영향을 미칠 수 있습니다. 또한 모든 모델이 사전 설정된 형식을 일관되게 따를 수 있는 것은 아니므로 실제 적용 시 불확실성이 증가합니다.
4. **로컬 최적 상태에 빠질 수 있음**: 단계별 의사 결정 모드는 에이전트에 글로벌 장기 계획이 부족함을 의미합니다. 단기적으로는 올바른 것처럼 보이지만 즉각적인 '관찰'로 인해 장기적으로는 최적이 아닌 경로를 선택하거나 경우에 따라 '제자리 회전' 루프에 빠질 수도 있습니다.

(3) 디버깅 기술

빌드된 ReAct 에이전트가 예기치 않게 작동하는 경우 다음 측면에서 디버깅할 수 있습니다.

- **완료 프롬프트 확인**: 각 LLM 호출 전에 모든 기록이 포함된 최종 형식의 전체 프롬프트를 인쇄합니다. 이는 LLM 결정의 출처를 추적하는 가장 직접적인 방법입니다.
- **원시 출력 분석**: 출력 구문 분석에 실패하는 경우(예: 정규 표현식이 'Action'과 일치하지 않음) LLM에서 반환한 처리되지 않은 원시 텍스트를 인쇄해야 합니다. 이는 LLM이 형식을 따르지 않았는지 또는 구문 분석 논리가 잘못되었는지 확인하는 데 도움이 될 수 있습니다.
- **도구 입력 및 출력 확인**: 에이전트가 생성한 'tool_input'이 도구 기능에서 예상하는 형식인지 확인하고, 도구에서 반환된 '관찰'이 에이전트가 이해하고 처리할 수 있는 형식인지 확인하세요.
- **프롬프트에서 예제 조정(Few-shot Prompting)**: 모델이 자주 오류를 범하는 경우 프롬프트에 하나 또는 두 개의 완전한 성공적인 "생각-행동-관찰" 사례를 추가하여 모델이 예제를 통해 지침을 더 잘 따르도록 안내할 수 있습니다.
- **다른 모델 또는 매개변수 사용해 보기**: 보다 성능이 뛰어난 모델로 전환하거나 '온도' 매개변수(일반적으로 출력 결정성을 보장하기 위해 0으로 설정)를 조정하면 문제를 직접 해결할 수 있는 경우가 있습니다.

## 4.3 계획 및 해결

이 반응형 단계별 의사 결정 에이전트 패러다임인 ReAct를 마스터한 후 다음으로 매우 다른 스타일이지만 똑같이 강력한 방법인 **계획 및 해결**을 탐색합니다. 이름에서 알 수 있듯이 이 패러다임은 작업 처리를 **먼저 계획하고 그 다음 해결**이라는 두 단계로 명시적으로 나눕니다.

ReAct가 현장의 단서를 바탕으로 차근차근 추론하고(Observation) 수사 방향을 수시로 조정하는 노련한 탐정과 같다면, 그렇다면 Plan-and-Solve는 건설을 시작하기 전에 먼저 완전한 청사진(Plan)을 그려야 하고 그런 다음 청사진에 따라 엄격하게 구축(Solve)해야 하는 건축가와 비슷합니다. 실제로 우리가 사용하는 많은 대규모 모델 도구의 에이전트 모드에는 이제 이 디자인 패턴이 통합되어 있습니다.

### 4.3.1 계획 ​​및 해결의 작동 원리

Plan-and-Solve Prompting was proposed by Lei Wang in 2023<sup>[2]</sup>. Its core motivation is to solve the problem that chain-of-thought easily "goes off track" when handling multi-step, complex problems.

각 단계에서 사고와 행동을 통합하는 ReAct와 달리 Plan-and-Solve는 그림 4.2와 같이 전체 프로세스를 두 가지 핵심 단계로 분리합니다.

1. **계획 단계**: 먼저 상담원은 사용자의 완전한 질문을 받습니다. 첫 번째 임무는 문제를 직접 해결하거나 도구를 호출하는 것이 아니라 **문제를 분해하고 명확한 단계별 실행 계획을 수립**하는 것입니다. 이 계획 자체는 대규모 언어 모델 호출의 산물입니다.
2. **해결 단계**: 에이전트는 완전한 계획을 얻은 후 실행 단계에 들어갑니다. **계획의 단계에 따라 하나씩 엄격하게 실행됩니다**. 각 단계의 실행은 계획의 모든 단계가 완료되고 최종 답변을 얻을 때까지 독립적인 LLM 호출 또는 이전 단계 결과의 처리일 수 있습니다.

이 "행동 전 계획" 전략을 통해 에이전트는 장기 계획이 필요한 복잡한 작업을 처리할 때 중간 단계에서 길을 잃지 않고 더 높은 목표 일관성을 유지할 수 있습니다.

우리는 이 2단계 과정을 공식적으로 표현할 수 있습니다. 먼저, 계획 모델 $\pi_{\text{plan}}$은 원래 질문 $q$를 기반으로 $n$ 단계를 포함하는 $P = (p_1, p_2, \dots, p_n)$ 계획을 생성합니다.

$$
P = \pi_{\text{계획}}(q)
$$

이후 실행 단계에서 실행 모델 $\pi_{\text{solve}}$는 계획의 단계를 하나씩 완료합니다. $i$-번째 단계의 경우 해당 솔루션 $s_i$의 생성은 원래 질문 $q$, 전체 계획 $P$ 및 모든 이전 단계 $(s_1, \dots, s_{i-1})$의 실행 결과에 따라 달라집니다.

$$
s_i = \pi_{\text{풀이}}(q, P, (s_1, \dots, s_{i-1}))
$$

최종 답변은 마지막 단계 $s_n$의 실행 결과입니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/4-figures/4-2.png" alt="Two-stage workflow of Plan-and-Solve paradigm" width="90%"/>
  <p>그림 4.2 계획-해결 패러다임의 2단계 작업 흐름</p>
</div>

Plan-and-Solve는 다음과 같이 명확하게 분해될 수 있는 강력한 구조를 가진 복잡한 작업에 특히 적합합니다.

- **다단계 수학 단어 문제**: 먼저 계산 단계를 나열한 다음 하나씩 풀어야 합니다.
- **여러 정보원을 통합한 보고서 작성**: 먼저 보고서 구조(서론, 데이터소스A, 데이터소스B, 요약)를 계획한 후 내용을 하나씩 채워야 합니다.
- **코드 생성 작업**: 함수, 클래스, 모듈의 구조를 먼저 구상한 후 하나씩 구현해야 합니다.

### 4.3.2 계획 단계

구조화된 추론 작업에서 계획 및 해결 패러다임의 장점을 강조하기 위해 도구를 사용하지 않고 신속한 설계를 통해 추론 작업을 완료합니다.

이러한 유형의 작업의 특징은 단일 쿼리나 계산을 통해 답변을 얻을 수 없다는 것입니다. 문제는 먼저 논리적으로 일관된 일련의 하위 단계로 분해된 다음 순서대로 해결되어야 합니다. 이는 "먼저 계획하고 나중에 실행"하는 Plan-and-Solve의 핵심 기능을 정확하게 활용합니다.

**우리의 목표 문제는 다음과 같습니다.** "과일 가게에서 월요일에 사과 15개를 판매했습니다. 화요일에 판매된 사과 수는 월요일의 두 배였습니다. 수요일에 판매된 수는 화요일보다 5개 적었습니다. 이 3일 동안 총 몇 개의 사과가 판매되었습니까?"

이 문제는 대규모 언어 모델의 경우 특별히 어렵지 않지만 참조할 수 있는 명확한 논리적 체인이 포함되어 있습니다. 일부 실제 논리적 퍼즐의 경우 대형 모델이 고품질로 정확한 답변을 추론할 수 없는 경우 이 디자인 패턴을 참조하여 작업을 완료하기 위해 자신만의 에이전트를 설계할 수 있습니다. 상담원은 다음을 수행해야 합니다.

1. **계획 단계**: 먼저 문제를 세 가지 독립적인 계산 단계(화요일 매출 계산, 수요일 매출 계산, 총 매출 계산)로 분해합니다.
2. **실행 단계**: 그런 다음 계획을 엄격히 따르고 단계별로 계산을 실행하며 각 단계의 결과를 다음 단계의 입력으로 사용하여 최종적으로 합계를 얻습니다.

계획 단계의 목표는 대규모 언어 모델이 원래 문제를 수신하고 명확한 단계별 실행 계획을 출력하도록 하는 것입니다. 이 계획은 코드가 하나씩 쉽게 구문 분석하고 실행할 수 있도록 구조화되어야 합니다. 따라서 우리가 디자인하는 프롬프트는 모델의 역할과 임무를 명확하게 알려주고 출력 형식의 예를 제공해야 합니다.

````python
PLANNER_PROMPT_TEMPLATE = """
You are a top AI planning expert. Your task is to decompose complex problems posed by users into an action plan consisting of multiple simple steps.
Please ensure that each step in the plan is an independent, executable subtask and is strictly arranged in logical order.
Your output must be a Python list, where each element is a string describing a subtask.

Question: {question}

Please strictly output your plan in the following format, with ```python and ``` as prefix and suffix being necessary:
```python
["1단계", "2단계", "3단계", ...]
```
"""
````

이 프롬프트는 다음 사항을 통해 출력 품질과 안정성을 보장합니다.
- **역할 설정**: "최고의 AI 기획 전문가"는 모델의 전문적 역량을 활성화합니다.
- **작업 설명**: "문제 분해"의 목표를 명확하게 정의합니다.
- **형식 제약**: 출력을 Python 목록 형식의 문자열로 강제하여 후속 코드 구문 분석 작업을 크게 단순화하여 자연어 구문 분석보다 더 안정적이고 신뢰할 수 있게 만듭니다.

다음으로, 이 프롬프트 로직을 플래너이기도 한 'Planner' 클래스에 캡슐화합니다.

```python
# Assume the HelloAgentsLLM class in llm_client.py is already defined
# from llm_client import HelloAgentsLLM

class Planner:
    def __init__(self, llm_client):
        self.llm_client = llm_client

    def plan(self, question: str) -> list[str]:
        """
        Generate an action plan based on user question.
        """
        prompt = PLANNER_PROMPT_TEMPLATE.format(question=question)

        # To generate a plan, we build a simple message list
        messages = [{"role": "user", "content": prompt}]

        print("--- Generating Plan ---")
        # Use streaming output to get the complete plan
        response_text = self.llm_client.think(messages=messages) or ""

        print(f"✅ Plan Generated:\n{response_text}")

        # Parse the list string output by LLM
        try:
            # Find content between ```python and ```
            plan_str = response_text.split("```python")[1].split("```")[0].strip()
            # Use ast.literal_eval to safely execute the string and convert it to a Python list
            plan = ast.literal_eval(plan_str)
            return plan if isinstance(plan, list) else []
        except (ValueError, SyntaxError, IndexError) as e:
            print(f"❌ Error parsing plan: {e}")
            print(f"Raw response: {response_text}")
            return []
        except Exception as e:
            print(f"❌ Unknown error occurred while parsing plan: {e}")
            return []
```

### 4.3.3 실행자와 상태 관리

플래너(`Planner`)가 명확한 작업 청사진을 생성한 후 계획의 작업을 하나씩 완료하려면 실행자(`Executor`)가 필요합니다. 실행자는 각 하위 문제를 해결하기 위해 대규모 언어 모델을 호출하는 역할을 담당할 뿐만 아니라 **상태 관리**라는 중요한 역할도 수행합니다. 각 단계의 실행 결과를 기록하고 이를 후속 단계의 컨텍스트로 제공하여 전체 작업 체인에서 정보가 원활하게 흐르도록 해야 합니다.

집행자의 프롬프트는 기획자의 프롬프트와 다릅니다. 그 목표는 문제를 분해하는 것이 아니라 **기존 컨텍스트를 기반으로 현재 단계를 해결하는 데 집중**하는 것입니다. 따라서 프롬프트에는 다음과 같은 주요 정보가 포함되어야 합니다.

- **원래 질문**: 모델이 항상 최종 목표를 이해하는지 확인하세요.
- **완전한 계획**: 모델이 전체 작업에서 현재 단계의 위치를 ​​이해하도록 합니다.
- **역사적 단계 및 결과**: 현재 단계에 대해 현재까지 완료된 작업을 직접 입력하여 제공합니다.
- **현재 단계**: 지금 해결해야 하는 특정 작업이 무엇인지 모델에 명확하게 지시합니다.

```python
EXECUTOR_PROMPT_TEMPLATE = """
You are a top AI execution expert. Your task is to strictly follow the given plan and solve the problem step by step.
You will receive the original question, the complete plan, and the steps and results completed so far.
Please focus on solving the "current step" and only output the final answer for that step, without any additional explanations or dialogue.

# Original Question:
{question}

# Complete Plan:
{plan}

# Historical Steps and Results:
{history}

# Current Step:
{current_step}

Please only output the answer for the "current step":
"""
```

실행 로직을 `Executor` 클래스로 캡슐화합니다. 이 클래스는 계획을 반복하고, LLM을 호출하고, 기록(상태)을 유지합니다.

```python
class Executor:
    def __init__(self, llm_client):
        self.llm_client = llm_client

    def execute(self, question: str, plan: list[str]) -> str:
        """
        Execute step by step according to the plan and solve the problem.
        """
        history = "" # String to store historical steps and results

        print("\n--- Executing Plan ---")

        for i, step in enumerate(plan):
            print(f"\n-> Executing step {i+1}/{len(plan)}: {step}")

            prompt = EXECUTOR_PROMPT_TEMPLATE.format(
                question=question,
                plan=plan,
                history=history if history else "None", # If it's the first step, history is empty
                current_step=step
            )

            messages = [{"role": "user", "content": prompt}]

            response_text = self.llm_client.think(messages=messages) or ""

            # Update history for the next step
            history += f"Step {i+1}: {step}\nResult: {response_text}\n\n"

            print(f"✅ Step {i+1} completed, result: {response_text}")

        # After the loop ends, the last step's response is the final answer
        final_answer = response_text
        return final_answer
```

이제 '계획'을 담당하는 'Planner'와 '실행'을 담당하는 'Executor'를 별도로 구축했습니다. 마지막 단계는 이 두 구성 요소를 통합 에이전트 'PlanAndSolveAgent'에 통합하고 완전한 문제 해결 기능을 제공하는 것입니다. 우리는 LLM 클라이언트를 수신하고 내부 플래너와 실행기를 초기화하며 전체 프로세스를 시작하기 위한 간단한 `run` 메소드를 제공하는 책임이 매우 명확한 기본 클래스 `PlanAndSolveAgent`를 생성할 것입니다.

```python
class PlanAndSolveAgent:
    def __init__(self, llm_client):
        """
        Initialize the agent and create planner and executor instances.
        """
        self.llm_client = llm_client
        self.planner = Planner(self.llm_client)
        self.executor = Executor(self.llm_client)

    def run(self, question: str):
        """
        Run the agent's complete process: plan first, then execute.
        """
        print(f"\n--- Starting to Process Question ---\nQuestion: {question}")

        # 1. Call planner to generate plan
        plan = self.planner.plan(question)

        # Check if plan was successfully generated
        if not plan:
            print("\n--- Task Terminated --- \nUnable to generate valid action plan.")
            return

        # 2. Call executor to execute plan
        final_answer = self.executor.execute(question, plan)

        print(f"\n--- Task Completed ---\nFinal Answer: {final_answer}")
```

이 `PlanAndSolveAgent` 클래스의 디자인은 "상속보다 구성" 원칙을 구현합니다. 복잡한 논리 자체를 포함하지는 않지만 오케스트레이터 역할을 하여 작업을 완료하기 위해 내부 구성 요소를 명확하게 호출합니다.

### 4.3.4 인스턴스 실행 및 분석

전체 코드는 이 책과 함께 제공되는 코드 저장소의 `code` 폴더에서도 찾을 수 있습니다. 여기서는 최종 결과만 보여드리겠습니다.

````bash
--- Starting to Process Question ---
Question: A fruit store sold 15 apples on Monday. The number of apples sold on Tuesday was twice that of Monday. The number sold on Wednesday was 5 fewer than Tuesday. How many apples were sold in total over these three days?
--- Generating Plan ---
🧠 Calling xxxx model...
✅ Large language model response successful:
```python
["월요일 사과 판매량 계산: 15", "화요일 사과 판매량 계산: 월요일 수량 × 2 = 15 × 2 = 30", "수요일 사과 판매량 계산: 화요일 수량 - 5 = 30 - 5 = 25", "3일간 총 판매량 계산: 월요일 + 화요일 + 수요일 = 15 + 30 + 25 = 70"]
```
✅ Plan Generated:
```python
["월요일 사과 판매량 계산: 15", "화요일 사과 판매량 계산: 월요일 수량 × 2 = 15 × 2 = 30", "수요일 사과 판매량 계산: 화요일 수량 - 5 = 30 - 5 = 25", "3일간 총 판매량 계산: 월요일 + 화요일 + 수요일 = 15 + 30 + 25 = 70"]
```

--- Executing Plan ---

-> Executing step 1/4: Calculate Monday's apple sales: 15
🧠 Calling xxxx model...
✅ Large language model response successful:
15
✅ Step 1 completed, result: 15

-> Executing step 2/4: Calculate Tuesday's apple sales: Monday's quantity × 2 = 15 × 2 = 30
🧠 Calling xxxx model...
✅ Large language model response successful:
30
✅ Step 2 completed, result: 30

-> Executing step 3/4: Calculate Wednesday's apple sales: Tuesday's quantity - 5 = 30 - 5 = 25
🧠 Calling xxxx model...
✅ Large language model response successful:
25
✅ Step 3 completed, result: 25

-> Executing step 4/4: Calculate total sales for three days: Monday + Tuesday + Wednesday = 15 + 30 + 25 = 70
🧠 Calling xxxx model...
✅ Large language model response successful:
70
✅ Step 4 completed, result: 70

--- Task Completed ---
Final Answer: 70
````

위의 출력 로그에서 계획 및 해결 패러다임의 워크플로를 명확하게 볼 수 있습니다.

1. **계획 단계**: 에이전트는 먼저 `Planner`를 호출하고 복잡한 단어 문제를 4개의 논리적 단계가 포함된 Python 목록으로 성공적으로 분해합니다. 이 구조화된 계획은 후속 실행을 위한 기반을 마련합니다.
2. **실행 단계**: `Executor`는 생성된 계획에 따라 단계별로 엄격하게 실행합니다. 각 단계에서는 기록 결과를 컨텍스트로 사용하여 올바른 정보 전송을 보장합니다. 예를 들어 2단계에서는 1단계의 결과 "15"를 올바르게 사용하고 3단계에서는 2단계의 결과 "30"도 올바르게 사용합니다.
3. **결과**: 전체 프로세스는 명시적인 단계로 논리적으로 명확하며 에이전트는 정답 "70"에 정확하게 도달합니다.

## 4.4 반사

우리가 이미 구현한 ReAct 및 Plan-and-Solve 패러다임에서는 에이전트가 작업을 완료하면 워크플로가 종료됩니다. 그러나 행동 궤적이든 최종 결과이든 그들이 생성하는 초기 답변에는 오류가 포함되거나 개선의 여지가 있을 수 있습니다. Reflection 메커니즘의 핵심 아이디어는 에이전트에 **사후 자체 수정 루프**를 도입하여 인간처럼 작업을 검토하고 결함을 발견하고 반복적으로 최적화할 수 있도록 하는 것입니다.

### 4.4.1 반사 메커니즘의 핵심 아이디어

The inspiration for the Reflection mechanism comes from the human learning process: we proofread after completing a first draft and verify after solving a math problem. This idea is embodied in multiple studies, such as the Reflexion framework proposed by Shinn, Noah in 2023<sup>[3]</sup>. Its core workflow can be summarized as a concise three-step loop: **Execute -> Reflect -> Refine**.

1. **실행**: 먼저 에이전트는 익숙한 방법(예: ReAct 또는 계획 및 해결)을 사용하여 작업을 완료하려고 시도하여 예비 솔루션 또는 작업 궤적을 생성합니다. 이것은 "첫 번째 초안"으로 볼 수 있습니다.
2. **반사**: 다음으로 에이전트는 반성 단계에 들어갑니다. 이는 "검토자" 역할을 수행하기 위해 독립적인 대형 언어 모델 인스턴스 또는 특별한 프롬프트가 있는 인스턴스를 호출합니다. 이 "검토자"는 첫 번째 단계에서 생성된 "첫 번째 초안"을 검사하고 다음과 같은 다양한 차원에서 평가합니다.
- **사실적 오류**: 상식이나 알려진 사실과 모순되는 내용이 있나요?
- **논리적 결함**: 추론 과정에 불일치나 모순이 있습니까?
- **효율성 문제**: 작업을 완료하기 위한 보다 직접적이고 간결한 경로가 있습니까?
- **정보 누락**: 일부 핵심 제약 조건이나 문제의 측면이 간과되었습니까? 평가를 바탕으로 체계적인 **피드백**을 생성하여 구체적인 문제와 개선 제안을 지적합니다.
3. **정제**: 마지막으로 에이전트는 "첫 번째 초안"과 "피드백"을 새로운 컨텍스트로 사용하고 대규모 언어 모델을 다시 호출하고 피드백 내용을 기반으로 첫 번째 초안을 수정하도록 요청하여 보다 완전한 "수정된 초안"을 생성합니다.

그림 4.3에 표시된 것처럼 이 루프는 반사 단계가 더 이상 새로운 문제를 발견하지 않거나 미리 설정된 반복 제한에 도달할 때까지 여러 번 반복될 수 있습니다. 우리는 이러한 반복적인 최적화 프로세스를 공식적으로 표현할 수 있습니다. $O_i$가 $i$ 번째 반복에 의해 생성된 출력이라고 가정하면($O_0$은 초기 출력) 반사 모델 $\pi_{\text{reflect}}$는 $O_i$에 대한 피드백 $F_i$를 생성합니다.
$$
F_i = \pi_{\text{반영}}(\text{작업}, O_i)
$$
그 후, 개선 모델 $\pi_{\text{refine}}$은 원래 작업, 이전 버전의 출력 및 피드백을 결합하여 새 버전의 출력 $O_{i+1}$을 생성합니다.
$$
O_{i+1} = \pi_{\text{정제}}(\text{작업}, O_i, F_i)
$$



<div align="center">
<img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/4-figures/4-3.png" alt="Execute-Reflect-Refine iterative loop in Reflection mechanism" width="70%"/>
<p>그림 4.3 반사 메커니즘의 실행-반사-수정 반복 루프</p>
</div>



이전 두 패러다임과 비교하여 Reflection의 가치는 다음과 같습니다.

- 에이전트에 내부 오류 수정 루프를 제공하여 더 이상 외부 도구 피드백(ReAct의 관찰)에 완전히 의존하지 않게 하여 더 높은 수준의 논리적, 전략적 오류를 수정할 수 있습니다.
- 일회성 작업 실행을 지속적인 최적화 프로세스로 전환하여 복잡한 작업에 대한 최종 성공률과 답변 품질을 크게 향상시킵니다.
- 에이전트를 위한 임시 **"단기 기억"**을 구축합니다. 전체 "실행-반영-수정" 궤적은 귀중한 경험 기록을 형성합니다. 에이전트는 최종 답변을 알 뿐만 아니라 결함이 있는 첫 번째 초안에서 최종 버전까지 어떻게 반복되었는지도 기억합니다. 또한 이 메모리 시스템은 **다중 모드**가 가능하므로 에이전트가 텍스트(예: 코드, 이미지 등) 이상의 출력을 반영하고 수정할 수 있어 보다 강력한 다중 모드 에이전트를 구축하기 위한 기반을 마련할 수 있습니다.

### 4.4.2 케이스 세팅 및 메모리 모듈 설계

실제로 Reflection 메커니즘을 구현하기 위해 메모리 관리 메커니즘을 소개하겠습니다. 왜냐하면 Reflection은 일반적으로 정보 저장 및 검색에 해당하기 때문입니다. 컨텍스트가 충분히 길면 "검토자"가 모든 정보를 직접 얻은 다음 반영하여 중복된 정보가 많이 발생하는 경우가 많습니다. 이 실무 단계에서는 주로 **코드 생성 및 반복 최적화**를 완료합니다.

이 단계의 목표 작업은 "1과 n 사이의 모든 소수를 찾는 Python 함수를 작성합니다."입니다.

이 작업은 Reflection 메커니즘을 테스트하기 위한 훌륭한 시나리오입니다.

1. **명확한 최적화 경로 존재**: 대규모 언어 모델에 의해 처음 생성된 코드는 단순하지만 비효율적인 재귀 구현일 가능성이 높습니다.
2. **명확한 반성점**: 반성을 통해 "지나치게 높은 시간 복잡도" 또는 "중복 계산"과 같은 문제를 발견할 수 있습니다.
3. **명확한 최적화 방향**: 피드백을 바탕으로 보다 효율적인 반복 버전이나 메모이제이션 패턴을 사용하는 버전으로 최적화할 수 있습니다.

Reflection의 핵심은 반복에 있으며, 반복의 전제조건은 이전 시도와 받은 피드백을 기억하는 능력입니다. 따라서 이 패러다임을 구현하려면 "단기 기억" 모듈이 필수적입니다. 이 메모리 모듈은 각 "실행-반사" 루프의 전체 궤적을 저장하는 역할을 합니다.

```python
from typing import List, Dict, Any, Optional

class Memory:
    """
    A simple short-term memory module for storing the agent's action and reflection trajectory.
    """

    def __init__(self):
        """
        Initialize an empty list to store all records.
        """
        self.records: List[Dict[str, Any]] = []

    def add_record(self, record_type: str, content: str):
        """
        Add a new record to memory.

        Parameters:
        - record_type (str): Type of record ('execution' or 'reflection').
        - content (str): Specific content of the record (e.g., generated code or reflection feedback).
        """
        record = {"type": record_type, "content": content}
        self.records.append(record)
        print(f"📝 Memory updated, added a '{record_type}' record.")

    def get_trajectory(self) -> str:
        """
        Format all memory records into a coherent string text for building prompts.
        """
        trajectory_parts = []
        for record in self.records:
            if record['type'] == 'execution':
                trajectory_parts.append(f"--- Previous Attempt (Code) ---\n{record['content']}")
            elif record['type'] == 'reflection':
                trajectory_parts.append(f"--- Reviewer Feedback ---\n{record['content']}")

        return "\n\n".join(trajectory_parts)

    def get_last_execution(self) -> Optional[str]:
        """
        Get the most recent execution result (e.g., the latest generated code).
        Returns None if it doesn't exist.
        """
        for record in reversed(self.records):
            if record['type'] == 'execution':
                return record['content']
        return None
```

이 'Memory' 클래스의 디자인은 비교적 간결하며 주요 구조는 다음과 같습니다.

- `records` 목록을 사용하여 각 행동과 반성을 순서대로 저장합니다.
- `add_record` 메소드는 메모리에 새 항목을 추가하는 역할을 합니다.
- `get_trajectory` 메소드가 핵심입니다. 이는 메모리 궤적을 후속 프롬프트에 직접 삽입할 수 있는 텍스트 세그먼트로 "직렬화"하여 모델의 반영 및 최적화를 위한 완전한 컨텍스트를 제공합니다.
- `get_last_execution`을 사용하면 반영을 위한 최신 "첫 번째 초안"을 쉽게 얻을 수 있습니다.



### 4.4.3 반사 에이전트의 코딩 구현

'Memory' 모듈을 기반으로 이제 'ReflectionAgent'의 핵심 로직 구축을 진행할 수 있습니다. 전체 에이전트의 워크플로는 앞서 논의한 "실행-반영-수정" 루프를 중심으로 진행되며 신중하게 설계된 프롬프트를 통해 대규모 언어 모델이 다양한 역할을 수행하도록 안내합니다.

(1) 신속한 디자인

이전 패러다임과 달리 Reflection 메커니즘에는 서로 다른 역할이 함께 작동하려면 여러 프롬프트가 필요합니다.

1. **초기 실행 프롬프트**: 에이전트가 문제를 해결하기 위한 첫 번째 시도에 대한 프롬프트로, 비교적 간단한 내용으로 모델이 지정된 작업을 완료하기만 하면 됩니다.

```bash
INITIAL_PROMPT_TEMPLATE = """
You are a senior Python programmer. Please write a Python function according to the following requirements.
Your code must include a complete function signature, docstring, and follow PEP 8 coding standards.

Requirement: {task}

Please output the code directly without any additional explanations.
"""
```

2. **반사 프롬프트**: 이 프롬프트는 반사 메커니즘의 핵심입니다. 이는 모델에 "코드 검토자" 역할을 수행하고, 이전 라운드에서 생성된 코드를 비판적으로 분석하고, 구체적이고 실행 가능한 피드백을 제공하도록 지시합니다.

````bash
REFLECT_PROMPT_TEMPLATE = """
You are an extremely strict code review expert and senior algorithm engineer with ultimate requirements for code performance.
Your task is to review the following Python code and focus on finding its main bottlenecks in <strong>algorithm efficiency</strong>.

# Original Task:
{task}

# Code to Review:
```python
{암호}
```

Please analyze the time complexity of this code and consider whether there is an <strong>algorithmically superior</strong> solution to significantly improve performance.
If one exists, please clearly point out the deficiencies of the current algorithm and propose specific, feasible algorithm improvement suggestions (e.g., using sieve method instead of trial division).
Only if the code has reached optimality at the algorithm level can you answer "no improvement needed."

Please output your feedback directly without any additional explanations.
"""
````

3. **정제 프롬프트**: 피드백을 받은 후 이 프롬프트는 피드백 내용을 기반으로 원본 코드를 수정하고 최적화하도록 모델을 안내합니다.

````bash

REFINE_PROMPT_TEMPLATE = """
You are a senior Python programmer. You are optimizing your code based on feedback from a code review expert.

# Original Task:
{task}

# Your Previous Code Attempt:
{last_code_attempt}
Reviewer's Feedback:
{feedback}

Please generate an optimized new version of the code based on the reviewer's feedback.
Your code must include a complete function signature, docstring, and follow PEP 8 coding standards.
Please output the optimized code directly without any additional explanations.
"""
````

(2) 에이전트 캡슐화 및 구현

이제 이 프롬프트 로직 세트와 'Memory' 모듈을 'ReflectionAgent' 클래스에 통합하겠습니다.

```python
# Assume llm_client.py and memory.py are already defined
# from llm_client import HelloAgentsLLM
# from memory import Memory

class ReflectionAgent:
    def __init__(self, llm_client, max_iterations=3):
        self.llm_client = llm_client
        self.memory = Memory()
        self.max_iterations = max_iterations

    def run(self, task: str):
        print(f"\n--- Starting to Process Task ---\nTask: {task}")

        # --- 1. Initial Execution ---
        print("\n--- Performing Initial Attempt ---")
        initial_prompt = INITIAL_PROMPT_TEMPLATE.format(task=task)
        initial_code = self._get_llm_response(initial_prompt)
        self.memory.add_record("execution", initial_code)

        # --- 2. Iterative Loop: Reflection and Refinement ---
        for i in range(self.max_iterations):
            print(f"\n--- Iteration {i+1}/{self.max_iterations} ---")

            # a. Reflection
            print("\n-> Performing Reflection...")
            last_code = self.memory.get_last_execution()
            reflect_prompt = REFLECT_PROMPT_TEMPLATE.format(task=task, code=last_code)
            feedback = self._get_llm_response(reflect_prompt)
            self.memory.add_record("reflection", feedback)

            # b. Check if stopping is needed
            if "no improvement needed" in feedback.lower():
                print("\n✅ Reflection considers code needs no improvement, task completed.")
                break

            # c. Refinement
            print("\n-> Performing Refinement...")
            refine_prompt = REFINE_PROMPT_TEMPLATE.format(
                task=task,
                last_code_attempt=last_code,
                feedback=feedback
            )
            refined_code = self._get_llm_response(refine_prompt)
            self.memory.add_record("execution", refined_code)

        final_code = self.memory.get_last_execution()
        print(f"\n--- Task Completed ---\nFinal Generated Code:\n```python\n{final_code}\n```")
        return final_code

    def _get_llm_response(self, prompt: str) -> str:
        """A helper method for calling LLM and getting complete streaming response."""
        messages = [{"role": "user", "content": prompt}]
        response_text = self.llm_client.think(messages=messages) or ""
        return response_text

```

### 4.4.4 인스턴스 실행 및 분석

전체 코드는 이 책과 함께 제공되는 코드 저장소의 `code` 폴더에서도 찾을 수 있습니다. 여기서는 출력 인스턴스를 제공합니다.

````python
--- Starting to Process Task ---
Task: Write a Python function to find all prime numbers between 1 and n.

--- Performing Initial Attempt ---
🧠 Calling xxxxxx model...
✅ Large language model response successful:
```python
def find_primes(n):
    ...
    return primes
```
📝 Memory updated, added an 'execution' record.

--- Iteration 1/2 ---

-> Performing Reflection...
🧠 Calling xxxxxx model...
✅ Large language model response successful:
The current code has a time complexity of O(n * sqrt(n)). While this implementation is acceptable for smaller n values, performance will significantly degrade when n is very large. The main bottleneck is that each number needs trial division checking, leading to high time overhead.

It is recommended to use the Sieve of Eratosthenes algorithm, which has a time complexity of O(n log(log n)) and can significantly improve the efficiency of finding prime numbers.

Improved code as follows:
```python
def find_primes(n):
    ...
    return primes
```
📝 Memory updated, added a 'reflection' record.

-> Performing Refinement...
🧠 Calling xxxxxx model...
✅ Large language model response successful:
```python
def find_primes(n):
    ...
    return primes
```
📝 Memory updated, added an 'execution' record.

--- Iteration 2/2 ---

-> Performing Reflection...
🧠 Calling xxxxxx model...
✅ Large language model response successful:
The current code uses the Sieve of Eratosthenes with time complexity O(n log log n) and space complexity O(n). This algorithm is already very efficient for finding all prime numbers between 1 and n, and usually requires no further optimization. However, in some specific scenarios, the following improvements can be considered:

1. <strong>Segmented Sieve</strong>: Suitable for cases where n is very large but memory is limited. Divide the interval into multiple small segments, process each segment separately with the sieve method, reducing memory usage.
2. <strong>Odd Number Sieve</strong>: Except for 2, all prime numbers are odd. When initializing the `is_prime` array, only mark odd numbers, which can reduce space complexity by half while reducing some unnecessary calculations.

However, these improvements are not necessary for most application scenarios because the standard Sieve of Eratosthenes is already efficient enough. Therefore, in general cases, <strong>no improvement needed</strong>.
📝 Memory updated, added a 'reflection' record.

✅ Reflection considers code needs no improvement, task completed.

--- Task Completed ---
Final Generated Code:
```python
def find_primes(n):
    """
    Finds all prime numbers between 1 and n using the Sieve of Eratosthenes algorithm.

    :param n: The upper limit of the range to find prime numbers.
    :return: A list of all prime numbers between 1 and n.
    """
    if n < 2:
        return []

    is_prime = [True] * (n + 1)
    is_prime[0] = is_prime[1] = False

    p = 2
    while p * p <= n:
        if is_prime[p]:
            for i in range(p * p, n + 1, p):
                is_prime[i] = False
        p += 1

    primes = [num for num in range(2, n + 1) if is_prime[num]]
    return primes
```
````

이 실행 인스턴스는 리플렉션 메커니즘이 에이전트가 심층적인 최적화를 수행하도록 유도하는 방법을 보여줍니다.

1. **효과적인 "비판"은 최적화의 전제 조건입니다**: 첫 번째 반성 라운드에서 우리는 "매우 엄격하고" "알고리즘 효율성에 초점을 맞춘" 프롬프트를 사용했기 때문에 에이전트는 기능적으로 올바른 초기 코드에 만족하지 않았지만 'O(n * sqrt(n))' 시간 복잡도 병목 현상과 제안된 알고리즘 수준 개선 제안인 에라토스테네스의 체를 정확하게 지적했습니다.
2. **반복적 개선**: 에이전트는 명확한 피드백을 받은 후 개선 단계에서 보다 효율적인 체 방법을 성공적으로 구현하여 알고리즘 복잡도를 'O(n log log n)'으로 줄여 의미 있는 첫 번째 자체 반복을 완료했습니다.
3. **수렴 및 종료**: 두 번째 반성에서 이미 효율적인 체 방법을 마주하면서 에이전트는 더 깊은 지식을 보여주었습니다. 현재 알고리즘의 효율성을 확인했을 뿐만 아니라 분할체와 같은 좀 더 발전된 최적화 방향까지 언급했지만, 최종적으로는 "일반적인 경우에는 개선이 필요하지 않다"는 올바른 판단을 내렸습니다. 이 판단은 종료 조건을 촉발하여 최적화 프로세스가 수렴되도록 했습니다.

이 사례는 잘 설계된 반사 메커니즘의 가치가 오류를 수정하는 것뿐만 아니라 더 중요한 것은 **품질과 효율성의 단계적 개선을 달성하기 위한 솔루션 구동**에 있다는 것을 완전히 입증하여 복잡한 고품질 에이전트를 구축하기 위한 핵심 기술 중 하나입니다.

### 4.4.5 반사 메커니즘의 비용-편익 분석

Reflection 메커니즘은 작업 솔루션 품질을 향상시키는 데 탁월한 성능을 발휘하지만 이 기능에는 비용이 들지 않습니다. 실제 적용에서는 그에 따른 비용 대비 이점을 평가해야 합니다.

(1) 주요 비용

1. **모델 호출 오버헤드 증가**: 가장 직접적인 비용입니다. 각 반복에는 최소 두 개의 추가 대규모 언어 모델 호출이 필요합니다(반영용 하나, 개선용 하나). 여러 라운드를 반복하면 API 호출 비용과 계산 리소스 소비가 기하급수적으로 증가합니다.

2. **작업 지연 시간이 크게 증가함**: 성찰은 연속적인 프로세스입니다. 각 개선 라운드는 이전 라운드의 반영이 완료될 때까지 기다려야 합니다. 이로 인해 총 작업 시간이 크게 늘어나 실시간 요구 사항이 높은 시나리오에는 적합하지 않습니다.

3. **프롬프트 엔지니어링 복잡성 증가**: 우리의 사례에서 알 수 있듯이 Reflection의 성공은 주로 고품질의 타겟 프롬프트에 달려 있습니다. "실행", "반영", "개선"과 같은 다양한 단계에 대한 효과적인 프롬프트를 디자인하고 디버깅하려면 더 많은 개발 노력이 필요합니다.

(2) 핵심혜택

1. **솔루션 품질의 도약**: 가장 큰 이점은 "적격한" 초기 솔루션을 "우수한" 최종 솔루션으로 반복적으로 최적화할 수 있다는 것입니다. 기능적으로 올바른 것에서 성능 효율적인 것, 대략적인 논리에서 엄격한 논리로의 이러한 개선은 많은 중요한 작업에서 매우 중요합니다.

2. **강화된 견고성 및 신뢰성**: 에이전트는 내부 자체 수정 루프를 통해 초기 솔루션에서 잠재적인 논리적 결함, 사실적 오류 또는 부적절한 경계 사례 처리를 발견하고 수정할 수 있어 최종 결과의 신뢰성이 크게 향상됩니다.

요약하자면, 반사 메커니즘은 전형적인 "품질 비용" 전략입니다. **최종 결과의 품질, 정확성 및 신뢰성에 대한 요구 사항이 매우 높고 작업 완료 실시간 성능에 대한 요구 사항이 비교적 완화된** 시나리오에 매우 적합합니다. 예를 들어:

- 중요한 비즈니스 코드 또는 기술 보고서를 생성합니다.
- 과학 연구에서 복잡한 논리적 추론을 수행합니다.
- 심층적인 분석과 기획이 필요한 의사결정 지원 시스템.

반대로, 애플리케이션 시나리오에 빠른 응답이 필요하거나 "거의 정확한" 답변이 이미 충분하다면 더 가벼운 ReAct 또는 계획 및 해결 패러다임을 사용하는 것이 더 비용 효율적인 선택일 수 있습니다.

## 4.5 장 요약

이 장에서는 3장에서 습득한 대규모 언어 모델 지식을 바탕으로 ReAct, Plan-and-Solve 및 Reflection이라는 "직접 바퀴 만들기"를 통해 세 가지 고전적인 산업 에이전트 구성 패러다임을 처음부터 코딩하고 구현했습니다. 우리는 핵심 작업 원리를 탐색했을 뿐만 아니라 구체적인 실제 사례를 통해 각각의 장점, 한계 및 적용 가능한 시나리오를 깊이 이해했습니다.

**핵심 지식 검토:**

1. ReAct: 외부 세계와 상호 작용할 수 있는 ReAct 에이전트를 구축했습니다. "생각-행동-관찰"이라는 동적 루프를 통해 검색 엔진을 사용하여 자체 지식 기반이 다룰 수 없는 실시간 질문에 답하는 데 성공했습니다. 핵심 장점은 **환경 적응성** 및 **동적 오류 수정 기능**에 있으므로 외부 도구 입력이 필요한 탐색 작업을 처리하는 데 가장 먼저 선택됩니다.
2. 계획 및 해결: 먼저 계획한 다음 실행하는 계획 및 해결 에이전트를 구현하고 이를 다단계 추론이 필요한 수학 단어 문제를 해결하는 데 사용했습니다. 복잡한 작업을 명확한 단계로 분해한 후 하나씩 실행합니다. 핵심 장점은 **구조** 및 **안정성**에 있으며, 특히 결정된 논리적 경로와 집중적인 내부 추론을 통해 작업을 처리하는 데 적합합니다.
3. 반사(자체 반사 및 반복): 자체 최적화 기능을 갖춘 Reflection 에이전트를 구축했습니다. "실행-반영-수정" 반복 루프를 도입함으로써 초기에 비효율적인 코드 솔루션을 알고리즘적으로 우수한 고성능 버전으로 성공적으로 최적화했습니다. 핵심 가치는 결과 정확성과 신뢰성에 대한 요구 사항이 매우 높은 시나리오에 적합한 **솔루션 품질을 대폭 향상**하는 것입니다.

이 장에서 탐구된 세 가지 패러다임은 표 4.1에 표시된 것처럼 에이전트가 문제를 해결하기 위한 세 가지 다른 전략을 나타냅니다. 실제 응용 분야에서 어떤 것을 선택할지는 작업의 핵심 요구 사항에 따라 달라집니다.

<div align="center">
<p>표 4.1 다양한 에이전트 루프에 대한 선택 전략</p>
<img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/4-figures/4-4.png" alt="" width="70%"/>
</div>

이 시점에서 우리는 개별 에이전트를 구축하기 위한 핵심 기술을 마스터했습니다. 지식을 전환하고 실제 애플리케이션에 대한 더 깊은 통찰력을 얻기 위해 다음 섹션에서는 에이전트 구축을 위해 다양한 로우 코드 플랫폼과 경량 코드 솔루션을 사용하는 방법을 살펴보겠습니다.

## 연습

> **참고**: 일부 연습에는 표준 답변이 없습니다. 에이전트 패러다임 설계에 대한 학습자의 종합적인 이해와 실무 능력을 배양하는 데 중점을 두고 있습니다.

1. 이 장에서는 'ReAct', 'Plan-and-Solve', 'Reflection'이라는 세 가지 고전적인 에이전트 패러다임을 소개했습니다. 분석해 주십시오:

- 이 세 가지 패러다임이 '사고'와 '행동'을 구성하는 방식에 있어 본질적인 차이점은 무엇인가?
- '스마트 홈 제어 도우미'(조명, 에어컨, 커튼 등을 제어하고 사용자 습관에 따라 자동으로 조정해야 하는)를 설계한다면 어떤 패러다임을 기본 아키텍처로 선택하시겠습니까? 왜?
- 이 세 가지 패러다임이 결합될 수 있는가? 그렇다면 하이브리드 패러다임 에이전트 아키텍처를 설계하고 적용 가능한 시나리오를 설명하십시오.

2. 섹션 4.2의 'ReAct' 구현에서는 정규식을 사용하여 대규모 언어 모델의 출력(예: 'Thought' 및 'Action')을 구문 분석했습니다. 다음을 고려하십시오:

- 현재의 파싱 방식에는 어떤 잠재적인 취약점이 존재하는가? 어떤 상황에서 실패할 수 있나요?
- 정규식 외에 더욱 강력한 출력 구문 분석 솔루션은 무엇입니까?
- 보다 안정적인 출력 형식을 사용하도록 이 장의 코드를 수정하고 두 가지 접근 방식의 장단점을 비교해 보세요.

3. 도구 호출은 최신 에이전트의 핵심 기능 중 하나입니다. 섹션 4.2.2의 `ToolExecutor` 디자인을 기반으로 다음 확장 실습을 완료하세요.

> **참고**: 이것은 실습 연습 문제입니다. 실제로 코드를 작성하는 것이 좋습니다.

- `ReAct` 에이전트에 '계산기' 도구를 추가하면 복잡한 수학적 계산 문제(예: '(123 + 456) × 789 / 12 = ?`의 결과 계산)를 처리할 수 있습니다.
- "도구 선택 실패" 처리 메커니즘을 설계하고 구현합니다. 에이전트가 반복적으로 잘못된 도구를 호출하거나 잘못된 매개변수를 제공하는 경우 시스템이 이를 수정하도록 어떻게 안내해야 합니까?
- 고려 사항: 호출 가능한 도구 수가 50개 또는 심지어 100개로 늘어나더라도 현재 도구 설명 방법이 여전히 효과적으로 작동합니까? 엔지니어링 관점에서 비즈니스 요구에 따라 호출 가능한 도구 수가 크게 증가하는 경우 도구의 구성 및 검색 메커니즘을 어떻게 최적화할 수 있습니까?

4. '계획 및 해결' 패러다임은 작업을 '계획'과 '실행'이라는 두 단계로 분해합니다. 심층적으로 분석해 보세요.

- 섹션 4.3의 구현에서 계획 단계에서 생성된 계획은 "정적"입니다(한 번 생성되고 수정할 수 없음). 실행 중에 특정 단계를 완료할 수 없거나 결과가 기대에 미치지 못하는 것으로 확인되면 "동적 재계획" 메커니즘을 어떻게 설계해야 합니까?
- 'Plan-and-Solve'와 'ReAct' 비교: "베이징에서 상하이까지 출장 예약(항공편, 호텔, 렌터카 포함)"과 같은 작업을 처리할 때 어떤 패러다임이 더 적합합니까? 왜?
- "계층적 계획" 시스템을 설계해 보세요. 먼저 상위 수준의 추상 계획을 생성한 다음 각 상위 수준 단계에 대한 세부 하위 계획을 생성합니다. 이 디자인에는 어떤 장점이 있나요?

5. 'Reflection' 메커니즘은 "execute-reflect-refine" 루프를 통해 출력 품질을 향상시킵니다. 다음을 고려하십시오:

- 4.4절의 코드생성 사례에서는 동일한 모델이 서로 다른 단계에서 사용된다. 두 가지 다른 모델을 사용하는 경우(예: 반영을 위해 더 강력한 모델을 사용하고 실행을 위해 더 빠른 모델을 사용하는 경우) 어떤 영향을 미치나요?
- '반사' 메커니즘의 종료 조건은 "피드백에 **개선이 필요하지 않음**" 또는 "최대 반복 횟수에 도달함"입니다. 이 디자인이 합리적인가? 보다 지능적인 종료 조건을 설계할 수 있습니까?
- 초안을 생성하고 지속적으로 논문 내용을 최적화할 수 있는 "학술 논문 작성 도우미"를 구축하고 싶다고 가정해 보겠습니다. 문단 논리, 방법 혁신, 언어 표현, 인용 기준 등 다각적인 관점에서 반영하고 개선하는 다차원적인 반영 메커니즘을 설계해 주세요.

6. 신속한 엔지니어링은 에이전트의 최종 효과에 영향을 미치는 핵심 기술입니다. 이 장에서는 세심하게 디자인된 여러 프롬프트 템플릿을 보여주었습니다. 분석해 주십시오:

- 섹션 4.2.3의 'ReAct' 프롬프트와 섹션 4.3.2의 'Plan-and-Solve' 프롬프트를 비교하세요. 그들은 분명히 구조 설계에 있어서 상당한 차이를 가지고 있습니다. 이러한 차이점은 각각의 패러다임의 핵심 논리에 어떻게 도움이 됩니까?
- 섹션 4.4.3의 `Reflection` 프롬프트에서 "당신은 매우 엄격한 코드 리뷰 전문가입니다."와 같은 역할 설정을 사용했습니다. 이 역할 설정을 수정하고(예: "귀하는 코드 가독성을 중요시하는 오픈 소스 프로젝트 유지관리자입니다"로 변경) 출력 결과의 변경 사항을 관찰하고 역할 설정이 에이전트 동작에 미치는 영향을 요약합니다.
- 프롬프트에 'few-shot' 예시를 추가하면 특정 형식을 따르는 모델의 능력이 크게 향상될 수 있습니다. 이 장의 에이전트 중 하나에 'few-shot' 예제를 추가하고 효과를 비교해 보십시오.

7. 이제 한 전자상거래 스타트업은 비용 절감과 효율성 향상을 위해 인간 고객 서비스를 대체하기 위해 "고객 서비스 에이전트"를 사용하기를 희망합니다. 다음과 같은 기능이 있어야 합니다.

에이. 사용자의 환불 요청 이유 이해

비. 사용자의 주문정보 및 물류상태를 조회

기음. 회사 정책에 따라 환불 승인 여부를 지능적으로 판단

디. 적절한 회신 이메일을 생성하여 사용자의 이메일로 보냅니다.

이자형. 판단 결과가 다소 논란의 여지가 있는 경우(자신감이 기준치 미만), 스스로 반성하고 보다 신중한 제안을 할 수 있습니다.

이 제품의 제품 관리자로서:
- 이 장의 어떤 패러다임(또는 패러다임의 조합)을 시스템의 핵심 아키텍처로 선택하시겠습니까?
- 이 시스템에는 어떤 도구가 필요합니까? 최소 3개의 도구와 해당 기능에 대한 설명을 나열하세요.
- 상담원의 결정이 회사 이익에 부합하고 사용자에 대해 우호적인 태도를 유지할 수 있도록 프롬프트를 디자인하는 방법은 무엇입니까?
- 이 제품이 출시된 후 어떤 위험과 과제에 직면하게 됩니까? 기술적 수단을 통해 이러한 위험을 어떻게 줄일 수 있습니까?

## 참고자료

[1] 야오 S, 자오 J, 유 D, 외. 반응: 언어 모델에서 추론과 행동의 시너지 효과[C]//ICLR(International Conference on Learning Representation). 2023.

[2] 왕 L, Xu W, Lan Y, 외. 계획 및 해결 프롬프트: 대규모 언어 모델을 통한 제로샷 연쇄 추론 개선[J]. arXiv 사전 인쇄 arXiv:2305.04091, 2023.

[3] Shinn N, Cassano F, Gopinath A, 외. 반사: 언어 강화 학습을 갖춘 언어 에이전트[J]. 신경 정보 처리 시스템의 발전, 2023, 36: 8634-8652.

