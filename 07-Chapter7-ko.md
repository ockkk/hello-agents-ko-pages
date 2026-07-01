# 7장 에이전트 프레임워크 구축

이전 장에서는 에이전트의 기본 사항을 설명하고 주류 프레임워크가 제공하는 개발 편의성을 경험했습니다. 이 장부터 우리는 더욱 도전적이고 가치 있는 단계인 **처음부터 에이전트 프레임워크 구축—HelloAgents**에 들어갑니다.

학습 과정의 연속성과 재현성을 보장하기 위해 HelloAgents는 버전 반복을 통해 개발을 진행합니다. 각 장에서는 이전 장을 기반으로 새로운 기능 모듈을 추가하고 에이전트 관련 지식 포인트를 통합하고 구현합니다. 궁극적으로 우리는 이 자체 구축 프레임워크를 사용하여 이 책의 다음 장에서 고급 응용 사례를 효율적으로 구현할 것입니다.

## 7.1 전체 프레임워크 아키텍처 설계

### 7.1.1 자신만의 에이전트 프레임워크를 구축해야 하는 이유

오늘날 빠르게 발전하는 에이전트 기술 환경에서는 이미 시장에 성숙한 에이전트 프레임워크가 많이 나와 있습니다. 그렇다면 왜 우리는 처음부터 새로운 프레임워크를 구축해야 할까요?

(1) 시장 프레임워크의 빠른 반복과 한계

에이전트 분야는 새로운 개념이 끊임없이 등장하며 빠르게 발전하는 분야입니다. 각 프레임워크에는 에이전트 설계에 대한 자체 포지셔닝과 이해가 있지만 에이전트의 핵심 지식 포인트는 일관됩니다.

- **과도한 추상화의 복잡성**: 많은 프레임워크에서는 일반성을 추구하기 위해 수많은 추상화 계층과 구성 옵션을 도입합니다. LangChain을 예로 들면, 체인 호출 메커니즘은 유연하지만 초보자를 위한 학습 곡선이 가파르고 간단한 작업을 완료하려면 많은 개념을 이해해야 하는 경우가 많습니다.
- **빠른 반복으로 인한 불안정성**: 상용 프레임워크는 시장 점유율을 확보하기 위해 API 인터페이스를 자주 변경합니다. 개발자는 버전 업그레이드 후에도 코드가 실행되지 않아 유지 관리 비용이 많이 드는 문제에 직면하는 경우가 많습니다.
- **블랙박스 구현 로직**: 많은 프레임워크가 핵심 로직을 너무 촘촘하게 캡슐화하므로 개발자가 에이전트의 내부 작업 메커니즘을 이해하기 어렵고 심층적인 사용자 정의 기능이 부족합니다. 문제가 발생하면 문서와 커뮤니티 지원에만 의존할 수 있습니다. 특히 커뮤니티가 충분히 활성화되지 않은 경우 피드백을 진행하는 사람이 없으면 피드백에 매우 오랜 시간이 걸려 후속 개발 효율성에 영향을 미칠 수 있습니다.
- **종속성의 복잡성**: 성숙한 프레임워크는 종종 설치 패키지 크기가 큰 많은 수의 종속성 패키지를 전달하므로 다른 프로젝트 코드와 협력해야 할 때 종속성 충돌 문제가 발생할 수 있습니다.

(2) 사용자에서 빌더로의 역량 도약

자신만의 에이전트 프레임워크를 구축하는 것은 실제로 "사용자"에서 "빌더"로 전환하는 과정입니다. 이러한 변화가 가져오는 가치는 장기적입니다.

- **에이전트 작동 원리에 대한 심층적인 이해**: 개발자는 각 구성 요소를 직접 구현함으로써 에이전트의 사고 과정, 도구 호출 메커니즘, 다양한 디자인 패턴의 장단점을 진정으로 이해할 수 있습니다.
- **완벽한 제어권 확보**: 자체 구축된 프레임워크는 모든 코드 라인을 완벽하게 제어하여 타사 프레임워크 설계 철학의 제약을 받지 않고 특정 요구 사항에 따라 정밀한 조정이 가능함을 의미합니다.
- **시스템 설계 역량 육성**: 프레임워크 구축 프로세스에는 개발자의 장기적인 성장에 중요한 가치가 있는 모듈형 설계, 인터페이스 추상화, 오류 처리 등 핵심 소프트웨어 엔지니어링 기술이 포함됩니다.

(3) 커스터마이징 요구의 필요성과 깊은 숙달

실제 애플리케이션에서 에이전트에 대한 요구 사항은 다양한 시나리오에 따라 크게 다르며, 종종 일반 프레임워크를 기반으로 한 보조 개발이 필요합니다.

- **특정 도메인에 대한 최적화 요구**: 금융, 의료, 교육과 같은 수직 도메인에는 타겟 프롬프트 템플릿, 특수 도구 통합 및 맞춤형 보안 전략이 필요한 경우가 많습니다.
- **성능 및 리소스의 정확한 제어**: 프로덕션 환경에서는 응답 시간, 메모리 사용량 및 동시 처리 기능에 대한 엄격한 요구 사항이 있습니다. 일반 프레임워크의 "일률적인" 솔루션은 종종 세련된 요구 사항을 충족할 수 없습니다.
- **학습 및 교육을 위한 투명성 요구 사항**: 교육 시나리오에서 학습자는 에이전트 구성 프로세스의 모든 단계를 명확하게 확인하고 다양한 패러다임의 작동 메커니즘을 이해하기를 기대합니다. 이를 위해서는 프레임워크에 높은 관찰 가능성과 해석 가능성이 필요합니다.

### 7.1.2 HelloAgents 프레임워크의 설계 철학

새로운 에이전트 프레임워크를 구축하는 것은 기능의 수에 관한 것이 아니라 디자인 철학이 기존 프레임워크의 문제점을 진정으로 해결할 수 있는지 여부에 관한 것입니다. HelloAgents 프레임워크의 설계는 핵심 질문을 중심으로 진행됩니다. 학습자가 에이전트의 작동 원리를 어떻게 신속하게 시작하고 깊이 이해할 수 있습니까?

성숙한 프레임워크를 처음 접하면 풍부한 기능에 매료될 수 있지만 곧 문제를 발견하게 될 것입니다. 간단한 작업을 완료하려면 체인, 에이전트, 도구, 메모리, 검색기 등과 같은 12개 이상의 다양한 개념을 이해해야 하는 경우가 많습니다. 각 개념에는 고유한 추상화 계층이 있으므로 학습 곡선이 매우 가파르게 됩니다. 이러한 복잡성은 강력한 기능을 제공하지만 초보자에게는 장애물이 되기도 합니다. HelloAgents 프레임워크는 기능적 완성도와 학습 친화성 사이의 균형을 찾으려고 시도하여 4가지 핵심 디자인 철학을 형성합니다.

(1) 경량성과 교육 친화적인 균형

훌륭한 학습 프레임워크는 완전한 가독성을 갖추어야 합니다. HelloAgents는 간단한 원칙에 따라 핵심 코드를 장별로 구분합니다. 특정 프로그래밍 기반을 갖춘 모든 개발자는 합리적인 시간 내에 프레임워크의 작동 원리를 완전히 이해할 수 있어야 합니다. 종속성 관리에서 프레임워크는 최소한의 전략을 채택합니다. OpenAI의 공식 SDK와 몇 가지 필수 기본 라이브러리를 제외하고 무거운 종속성은 도입되지 않습니다. 문제가 발생하면 복잡한 종속 관계에서 답을 찾지 않고도 프레임워크 자체 코드를 직접 찾을 수 있습니다.

(2) 표준 API를 기반으로 한 실용적인 선택

OpenAI의 API는 업계 표준이 되었으며 거의 ​​모든 주류 LLM 제공업체는 이 인터페이스와 호환되도록 열심히 노력하고 있습니다. HelloAgents는 추상 인터페이스를 재창조하는 대신 이 표준을 기반으로 구축하기로 선택합니다. 이 결정은 주로 몇 가지 사항에 의해 결정됩니다. 첫째, 호환성 보장입니다. HelloAgents 사용을 마스터한 후 다른 프레임워크로 마이그레이션하거나 이를 기존 프로젝트에 통합할 때 기본 API 호출 논리는 완전히 일관됩니다. 둘째, 학습비용 절감이다. 모든 작업은 이미 익숙한 표준 인터페이스를 기반으로 하기 때문에 새로운 개념 모델을 배울 필요가 없습니다.

(3) 점진적인 학습 경로의 신중한 설계

HelloAgents는 명확한 학습 경로를 제공합니다. 각 장의 학습 코드를 pip를 통해 다운로드할 수 있는 기록 버전으로 저장하므로 모든 핵심 기능을 직접 작성하므로 코드 사용 비용에 대해 걱정할 필요가 없습니다. 이 디자인을 사용하면 자신의 필요와 속도에 따라 앞으로 나아갈 수 있습니다. 각 업그레이드는 개념적 점프나 이해의 차이 없이 자연스럽습니다. 이 장의 내용도 이전 6장의 내용을 기반으로 한다는 점은 언급할 가치가 있습니다. 마찬가지로, 이 장에서는 후속 고급 지식 학습을 위한 프레임워크 기반도 마련합니다.

(4) 통합된 "도구" 추상화: 모든 것이 도구입니다.

가볍고 교육 친화적인 철학을 철저히 구현하기 위해 HelloAgents는 아키텍처를 핵심적으로 단순화했습니다. 핵심 Agent 클래스를 제외하면 모든 것이 도구입니다. 메모리, RAG(Retrieval-Augmented Generation), RL(Reinforcement Learning), MCP(Protocol) 및 기타 여러 프레임워크에서 독립적으로 학습해야 하는 기타 모듈은 모두 HelloAgents에서 "도구"로 일률적으로 추상화됩니다. 이 디자인의 원래 의도는 불필요한 추상화 계층을 제거하여 학습자가 "에이전트 호출 도구"의 가장 직관적인 핵심 논리로 돌아갈 수 있도록 함으로써 빠른 시작과 깊은 이해의 통일성을 진정으로 달성하는 것입니다.

### 7.1.3 이 장의 학습 목표

먼저 7장의 핵심 학습 내용을 살펴보겠습니다.

```
hello-agents/
├── hello_agents/
│   │
│   ├── core/                     # Core framework layer
│   │   ├── agent.py              # Agent base class
│   │   ├── llm.py                # HelloAgentsLLM unified interface
│   │   ├── message.py            # Message system
│   │   ├── config.py             # Configuration management
│   │   └── exceptions.py         # Exception system
│   │
│   ├── agents/                   # Agent implementation layer
│   │   ├── simple_agent.py       # SimpleAgent implementation
│   │   ├── react_agent.py        # ReActAgent implementation
│   │   ├── reflection_agent.py   # ReflectionAgent implementation
│   │   └── plan_solve_agent.py   # PlanAndSolveAgent implementation
│   │
│   ├── tools/                    # Tool system layer
│   │   ├── base.py               # Tool base class
│   │   ├── registry.py           # Tool registration mechanism
│   │   ├── chain.py              # Tool chain management system
│   │   ├── async_executor.py     # Asynchronous tool executor
│   │   └── builtin/              # Built-in tool set
│   │       ├── calculator.py     # Calculator tool
│   │       └── search.py         # Search tool
└──
```

특정 코드 작성을 시작하기 전에 먼저 명확한 아키텍처 청사진을 설정해야 합니다. HelloAgents의 아키텍처 설계는 코드 구성을 유지하고 장별 콘텐츠 확장을 용이하게 하는 "계층 분리, 단일 책임, 통합 인터페이스"의 핵심 원칙을 따릅니다.

**빠른 시작: HelloAgents 프레임워크 설치**

독자가 이 장의 전체 기능을 빠르게 경험할 수 있도록 직접 설치할 수 있는 Python 패키지를 제공합니다. 다음 명령을 사용하여 이 장에 해당하는 버전을 설치할 수 있습니다.

```bash
# hello-agents framework code visible link: https://github.com/jjyaoao/HelloAgents
# Python version needs to be >= 3.10
pip install "hello-agents==0.1.1"
```

이 장을 배우는 방법은 두 가지가 있습니다.

1. **체험학습**: `pip`을 이용해 프레임워크를 직접 설치하고, 예제 코드를 실행하며, 다양한 기능을 빠르게 체험해 보세요.
2. **딥 러닝**: 이 장의 내용을 따라 각 구성 요소를 처음부터 구현하고 프레임워크의 설계 아이디어와 구현 세부 사항을 깊이 이해합니다.

'먼저 경험한 후 구현'하는 학습 경로를 채택하는 것이 좋습니다. 이 장에서는 완전한 테스트 파일을 제공합니다. 핵심 기능을 다시 작성하고 테스트를 실행하여 구현이 올바른지 확인할 수 있습니다. 이 학습 방법은 실용성과 학습 효과를 모두 보장합니다. 프레임워크의 구현 세부 사항을 깊이 이해하고 싶거나 프레임워크 개발에 참여하고 싶다면 이 [GitHub 저장소](https://github.com/jjyaoao/helloagents).)를 방문하세요.

시작하기 전, Hello-agent를 활용한 간단한 에이전트 구축을 30초 만에 경험해보세요!

```python
# Configure the LLM API in the .env file in the same-level folder. You can refer to the .env.example in the code folder, or reuse the .env file from previous chapter cases.
from hello_agents import SimpleAgent, HelloAgentsLLM
from dotenv import load_dotenv

# Load environment variables
load_dotenv()

# Create LLM instance - framework automatically detects provider
llm = HelloAgentsLLM()

# Or manually specify provider (optional)
# llm = HelloAgentsLLM(provider="modelscope")

# Create SimpleAgent
agent = SimpleAgent(
    name="AI Assistant",
    llm=llm,
    system_prompt="You are a helpful AI assistant"
)

# Basic conversation
response = agent.run("Hello! Please introduce yourself")
print(response)

# Add tool functionality (optional)
from hello_agents.tools import CalculatorTool
calculator = CalculatorTool()
# Need to implement MySimpleAgent in 7.4.1 for invocation, subsequent chapters will support this invocation method
# agent.add_tool(calculator)

# Now you can use tools
response = agent.run("Please help me calculate 2 + 3 * 4")
print(response)

# View conversation history
print(f"Number of historical messages: {len(agent.get_history())}")
```



## 7.2 HelloAgentsLLM 확장

이 섹션의 내용은 섹션 4.1.3에서 생성된 `HelloAgentsLLM`을 기반으로 하는 반복적인 업그레이드입니다. 우리는 이 기본 클라이언트를 보다 적응력이 뛰어난 모델 호출 허브로 변환할 것입니다. 이 업그레이드는 주로 다음 세 가지 목표를 중심으로 진행됩니다.

1. **다중 제공업체 지원**: OpenAI, ModelScope, Zhipu AI 등과 같은 다양한 주류 LLM 서비스 제공업체 간에 원활한 전환을 달성하여 특정 공급업체에 대한 프레임워크 바인딩을 방지합니다.
2. **로컬 모델 통합**: 섹션 3.2.3의 Hugging Face Transformers 솔루션에 대한 프로덕션급 보완책으로 두 가지 고성능 로컬 배포 솔루션인 VLLM 및 Ollama를 도입하여 데이터 개인 정보 보호 및 비용 제어 요구 사항을 충족합니다.
3. **자동 감지 메커니즘**: 프레임워크가 환경 정보를 기반으로 사용되는 LLM 서비스 유형을 지능적으로 추론할 수 있는 자동 인식 메커니즘을 구축하여 사용자 구성 프로세스를 단순화합니다.

### 7.2.1 여러 공급자 지원

이전에 정의한 `HelloAgentsLLM` 클래스는 두 가지 핵심 매개변수 `api_key` 및 `base_url`를 통해 OpenAI 인터페이스와 호환되는 모든 서비스에 이미 연결할 수 있습니다. 이는 이론적으로는 보편성을 보장하지만 실제 응용에서는 서비스 제공자마다 환경 변수 이름, 기본 API 주소 및 권장 모델에 차이가 있습니다. 사용자가 서비스 공급자를 변경할 때마다 코드를 수동으로 쿼리하고 수정해야 한다면 개발 효율성에 큰 영향을 미치게 됩니다. 이 문제를 해결하기 위해 `provider`을 소개합니다. 개선 아이디어는 `HelloAgentsLLM`가 다양한 서비스 제공자의 구성 세부사항을 내부적으로 처리하여 사용자에게 통일되고 간결한 호출 경험을 제공하도록 하는 것입니다. 구체적인 구현 세부 사항은 섹션 7.2.3 "자동 탐지 메커니즘"에서 자세히 설명하겠습니다. 여기서는 먼저 이 메커니즘을 사용하여 프레임워크를 확장하는 방법에 중점을 둡니다.

아래에서는 `HelloAgentsLLM`을 상속하여 ModelScope 플랫폼에 대한 지원을 추가하는 방법을 보여줍니다. 우리는 독자들이 프레임워크를 "사용"하는 방법을 배울 뿐만 아니라 프레임워크를 "확장"하는 방법도 숙달할 수 있기를 바랍니다. 설치된 라이브러리의 소스 코드를 직접 수정하는 것은 후속 라이브러리 업그레이드를 어렵게 만들기 때문에 권장되지 않습니다.

(1) 맞춤형 LLM 클래스 생성 및 상속

프로젝트 디렉토리에 `my_llm.py` 파일이 있다고 가정해 보겠습니다. 먼저 `hello_agents` 라이브러리에서 `HelloAgentsLLM` 기본 클래스를 가져온 다음 이를 상속받는 `MyLLM`이라는 새 클래스를 만듭니다.

```python
# my_llm.py
import os
from typing import Optional
from openai import OpenAI
from hello_agents import HelloAgentsLLM

class MyLLM(HelloAgentsLLM):
    """
    A custom LLM client that adds support for ModelScope through inheritance.
    """
    pass # Leave empty for now
```

(2) 새로운 공급자를 지원하기 위해 `__init__` 방법 재정의

다음으로 `MyLLM` 클래스의 `__init__` 메서드를 재정의합니다. 우리의 목표는 사용자가 `provider="modelscope"`를 통과하면 사용자 정의 로직을 실행하는 것입니다. 그렇지 않으면 상위 클래스 `HelloAgentsLLM`의 원래 로직을 호출하여 OpenAI와 같은 다른 내장 제공자를 계속 지원할 수 있습니다.

```python
class MyLLM(HelloAgentsLLM):
    def __init__(
        self,
        model: Optional[str] = None,
        api_key: Optional[str] = None,
        base_url: Optional[str] = None,
        provider: Optional[str] = "auto",
        **kwargs
    ):
        # Check if provider is 'modelscope' that we want to handle
        if provider == "modelscope":
            print("Using custom ModelScope Provider")
            self.provider = "modelscope"

            # Parse ModelScope credentials
            self.api_key = api_key or os.getenv("MODELSCOPE_API_KEY")
            self.base_url = base_url or "https://api-inference.modelscope.cn/v1/"

            # Validate credentials exist
            if not self.api_key:
                raise ValueError("ModelScope API key not found. Please set MODELSCOPE_API_KEY environment variable.")

            # Set default model and other parameters
            self.model = model or os.getenv("LLM_MODEL_ID") or "Qwen/Qwen2.5-VL-72B-Instruct"
            self.temperature = kwargs.get('temperature', 0.7)
            self.max_tokens = kwargs.get('max_tokens')
            self.timeout = kwargs.get('timeout', 60)

            # Create OpenAI client instance with obtained parameters
            self._client = OpenAI(api_key=self.api_key, base_url=self.base_url, timeout=self.timeout)

        else:
            # If not modelscope, use parent class's original logic to handle
            super().__init__(model=model, api_key=api_key, base_url=base_url, provider=provider, **kwargs)

```

이 코드는 "재정의" 개념을 보여줍니다. `provider="modelscope"`의 경우를 가로채서 특별히 처리합니다. 다른 모든 경우에는 `super().__init__(...)`을 통해 상위 클래스에 다시 전달하여 원래 프레임워크 기능을 모두 유지합니다.

(3) 사용자 정의 `MyLLM` 클래스 사용

이제 네이티브 `HelloAgentsLLM`을 사용하는 것처럼 프로젝트의 비즈니스 로직에서 자체 `MyLLM` 클래스를 사용할 수 있습니다.

먼저 `.env` 파일에서 ModelScope API 키를 구성합니다.

```bash
# .env file
MODELSCOPE_API_KEY="your-modelscope-api-key"
```

그런 다음 기본 프로그램에서 `MyLLM`을 가져와 사용합니다.

```python
# my_main.py
from dotenv import load_dotenv
from my_llm import MyLLM # Note: Import our own class here

# Load environment variables
load_dotenv()

# Instantiate our overridden client and specify provider
llm = MyLLM(provider="modelscope")

# Prepare messages
messages = [{"role": "user", "content": "Hello, please introduce yourself."}]

# Make the call, think and other methods are inherited from parent class, no need to override
response_stream = llm.think(messages)

# Print response
print("ModelScope Response:")
for chunk in response_stream:
    # Chunk already printed in my_llm, just pass here
    # print(chunk, end="", flush=True)
    pass
```

위의 단계를 통해 소스 코드를 수정하지 않고도 `hello-agents` 라이브러리에 새로운 기능을 성공적으로 확장했습니다. 이 방법은 코드의 청결성과 유지 관리성을 보장할 뿐만 아니라 향후 `hello-agents` 라이브러리를 업그레이드할 때 사용자 정의된 기능이 손실되지 않도록 보장합니다.

### 7.2.2 로컬 모델 호출

섹션 3.2.3에서는 Hugging Face Transformers 라이브러리를 사용하여 오픈 소스 모델을 로컬에서 실행하는 방법을 배웠습니다. 이 방법은 입문 학습 및 기능 검증에 매우 적합하지만 기본 구현은 높은 동시성 요청을 처리할 때 성능이 제한되며 일반적으로 프로덕션 환경에서 첫 번째 선택이 아닙니다.

고성능, 프로덕션급 모델 추론 서비스를 로컬에서 달성하기 위해 커뮤니티에서는 VLLM 및 Ollama와 같은 우수한 도구를 제작했습니다. 연속 일괄 처리 및 PagedAttention과 같은 기술을 통해 모델 처리량 및 운영 효율성을 크게 향상시키고 모델을 OpenAI 표준과 호환되는 API 서비스로 캡슐화합니다. 이는 이를 `HelloAgentsLLM`에 원활하게 통합할 수 있음을 의미합니다.

**VLLM**

VLLM은 LLM 추론을 위해 설계된 고성능 Python 라이브러리입니다. PagedAttention과 같은 고급 기술을 통해 표준 Transformers 구현보다 몇 배 더 높은 처리량을 달성할 수 있습니다. 다음은 VLLM 서비스를 로컬로 배포하는 전체 단계입니다.

먼저, 하드웨어 환경(특히 CUDA 버전)에 따라 VLLM을 설치해야 합니다. 버전 불일치 문제를 방지하려면 [공식 문서](https://docs.vllm.ai/en/latest/getting_started/installation.html)에 따라 설치하는 것이 좋습니다.

```python
pip install vllm
```

설치 후 다음 명령을 사용하여 OpenAI 호환 API 서비스를 시작합니다. VLLM은 Hugging Face Hub에서 지정된 모델 가중치를 자동으로 다운로드합니다(로컬에 존재하지 않는 경우). 우리는 여전히 Qwen1.5-0.5B-Chat 모델을 예로 사용합니다.

```
# Start VLLM service and load Qwen1.5-0.5B-Chat model
python -m vllm.entrypoints.openai.api_server \
    --model Qwen/Qwen1.5-0.5B-Chat \
    --host 0.0.0.0 \
    --port 8000
```

서비스 시작 후 `http://localhost:8000/v1` 주소에서 OpenAI 호환 API를 제공합니다.

**올라마**

Ollama는 모델 다운로드, 구성 및 서비스 시작을 단일 명령으로 캡슐화하여 로컬 모델 관리 및 배포를 더욱 단순화하므로 빠른 시작에 매우 적합합니다. Ollama [공식 웹사이트](https://ollama.com))를 방문하여 운영 체제에 맞는 클라이언트를 다운로드하고 설치하세요.

설치 후 터미널을 열고 다음 명령을 실행하여 모델을 다운로드하고 실행합니다(예: Llama 3 사용). Ollama는 모델 다운로드, 서비스 캡슐화 및 하드웨어 가속 구성을 자동으로 처리합니다.

```
# First run will automatically download the model, subsequent runs will directly start the service
ollama run llama3
```

터미널에 모델의 대화형 프롬프트가 표시되면 서비스가 백그라운드에서 성공적으로 시작되었음을 나타냅니다. Ollama는 기본적으로 `http://localhost:11434/v1` 주소에 OpenAI 호환 API 인터페이스를 노출합니다.

**`HelloAgentsLLM`과 통합**

VLLM과 Ollama는 모두 업계 표준 API를 따르므로 이를 `HelloAgentsLLM`에 통합하는 것은 매우 간단합니다. 클라이언트를 인스턴스화할 때 이를 새로운 `provider`로 처리하기만 하면 됩니다.

예를 들어 로컬로 실행되는 **VLLM** 서비스에 연결하는 경우:

```python
llm_client = HelloAgentsLLM(
    provider="vllm",
    model="Qwen/Qwen1.5-0.5B-Chat", # Must match the model specified when starting the service
    base_url="http://localhost:8000/v1",
    api_key="vllm" # Local services usually don't need a real API Key, can fill in any non-empty string
)
```

또는 환경 변수를 설정하고 클라이언트가 자동 감지하도록 하여 코드 수정이 전혀 발생하지 않도록 합니다.

```bash
# Set in .env file
LLM_BASE_URL="http://localhost:8000/v1"
LLM_API_KEY="vllm"

# Directly instantiate in Python code
llm_client = HelloAgentsLLM() # Will automatically detect as vllm
```

마찬가지로 로컬 **Ollama** 서비스에 연결하는 것도 마찬가지로 간단합니다.

```python
llm_client = HelloAgentsLLM(
    provider="ollama",
    model="llama3", # Must match the model specified in `ollama run`
    base_url="http://localhost:11434/v1",
    api_key="ollama" # Local services also don't need a real Key
)
```

이 통합 설계를 통해 에이전트 핵심 코드는 클라우드 API와 로컬 모델 간에 자유롭게 전환하기 위해 수정이 필요하지 않습니다. 이는 후속 애플리케이션 개발, 배포, 비용 제어 및 데이터 개인 정보 보호를 위한 뛰어난 유연성을 제공합니다.

### 7.2.3 자동 감지 메커니즘

사용자의 구성 부담을 최대한 최소화하고 "구성보다 관례" 원칙을 따르기 위해 `HelloAgentsLLM`는 두 가지 핵심 보조 방법인 `_auto_detect_provider` 및 `_resolve_credentials`를 내부적으로 설계합니다. `_auto_detect_provider`은 환경 정보를 기반으로 서비스 제공자를 추론하는 역할을 담당하고, `_resolve_credentials`는 추론 결과를 기반으로 특정 매개변수 구성을 완료하는 역할을 합니다.

`_auto_detect_provider` 메소드는 다음 우선순위에 따라 환경 정보를 기반으로 서비스 제공자를 자동으로 추론하는 역할을 합니다.

1. **최우선순위: 특정 서비스 제공자에 대한 환경변수 확인** 이는 가장 직접적이고 신뢰할 수 있는 판단 기준입니다. 프레임워크에서는 `MODELSCOPE_API_KEY`, `OPENAI_API_KEY`, `ZHIPU_API_KEY` 등의 환경 변수가 있는지 순차적으로 확인합니다. 하나라도 발견되면 해당 서비스 제공자를 즉시 ​​결정합니다.

2. **두 번째로 높은 우선순위: `base_url`을 기준으로 결정** 사용자가 특정 서비스 공급자의 키를 설정하지 않았지만 일반 `LLM_BASE_URL`을 설정한 경우 프레임워크는 대신 이 URL을 구문 분석합니다.

- **도메인 매칭**: URL에 `"api-inference.modelscope.cn"`, `"api.openai.com"` 등의 특성 문자열이 포함되어 있는지 확인하여 클라우드 서비스 제공업체를 식별합니다.

- **포트 매칭**: URL에 `:11434`(Ollama), `:8000`(VLLM) 등 로컬 서비스에 대한 표준 포트가 포함되어 있는지 확인하여 로컬 배포 솔루션을 식별합니다.

3. **보조 판단: API 키 형식 분석** 경우에 따라 위의 두 가지 방법 중 어느 것도 확인할 수 없는 경우 프레임워크는 일반 환경 변수 `LLM_API_KEY`의 형식을 분석하려고 시도합니다. 예를 들어 일부 서비스 제공업체의 API 키에는 고정된 접두어 또는 고유한 인코딩 형식이 있습니다. 그러나 이 방법은 모호함((e.g., multiple service providers have similar key formats))을 가질 수 있으므로 우선순위가 낮아서 보조적인 수단으로만 사용된다.

일부 키 코드는 다음과 같습니다.

```python
def _auto_detect_provider(self, api_key: Optional[str], base_url: Optional[str]) -> str:
    """
    Automatically detect LLM provider
    """
    # 1. Check environment variables for specific providers (highest priority)
    if os.getenv("MODELSCOPE_API_KEY"): return "modelscope"
    if os.getenv("OPENAI_API_KEY"): return "openai"
    if os.getenv("ZHIPU_API_KEY"): return "zhipu"
    # ... Other service provider environment variable checks

    # Get generic environment variables
    actual_api_key = api_key or os.getenv("LLM_API_KEY")
    actual_base_url = base_url or os.getenv("LLM_BASE_URL")

    # 2. Determine based on base_url
    if actual_base_url:
        base_url_lower = actual_base_url.lower()
        if "api-inference.modelscope.cn" in base_url_lower: return "modelscope"
        if "open.bigmodel.cn" in base_url_lower: return "zhipu"
        if "localhost" in base_url_lower or "127.0.0.1" in base_url_lower:
            if ":11434" in base_url_lower: return "ollama"
            if ":8000" in base_url_lower: return "vllm"
            return "local" # Other local ports

    # 3. Auxiliary judgment based on API key format
    if actual_api_key:
        if actual_api_key.startswith("ms-"): return "modelscope"
        # ... Other key format judgments

    # 4. Default return 'auto', use generic configuration
    return "auto"
```

`provider`가 결정되면(사용자 지정 또는 자동 감지 여부) `_resolve_credentials` 메소드가 서비스 제공자의 차별화된 구성을 처리합니다. `provider` 값을 기반으로 해당 환경 변수를 적극적으로 검색하고 이에 대한 기본값 `base_url`을 설정합니다. 일부 주요 구현은 다음과 같습니다.

```python
def _resolve_credentials(self, api_key: Optional[str], base_url: Optional[str]) -> tuple[str, str]:
    """Resolve API key and base_url based on provider"""
    if self.provider == "openai":
        resolved_api_key = api_key or os.getenv("OPENAI_API_KEY") or os.getenv("LLM_API_KEY")
        resolved_base_url = base_url or os.getenv("LLM_BASE_URL") or "https://api.openai.com/v1"
        return resolved_api_key, resolved_base_url

    elif self.provider == "modelscope":
        resolved_api_key = api_key or os.getenv("MODELSCOPE_API_KEY") or os.getenv("LLM_API_KEY")
        resolved_base_url = base_url or os.getenv("LLM_BASE_URL") or "https://api-inference.modelscope.cn/v1/"
        return resolved_api_key, resolved_base_url

    # ... Logic for other service providers
```

간단한 예를 통해 자동 감지가 가져다주는 편리함을 경험해 보겠습니다. 사용자가 로컬 Ollama 서비스를 사용하고 싶다면 다음과 같이 `.env` 파일만 구성하면 됩니다.

```bash
LLM_BASE_URL="http://localhost:11434/v1"
LLM_MODEL_ID="llama3"
```

코드에서 `LLM_API_KEY`을 구성하거나 `provider`을 지정할 필요가 없습니다. 그런 다음 Python 코드에서는 간단히 `HelloAgentsLLM`를 인스턴스화합니다.

```python
from dotenv import load_dotenv
from hello_agents import HelloAgentsLLM

load_dotenv()

# No need to pass provider, framework will auto-detect
llm = HelloAgentsLLM()
# Framework internal logs will show provider detected as 'ollama'

# Subsequent invocation methods remain completely unchanged
messages = [{"role": "user", "content": "Hello!"}]
for chunk in llm.think(messages):
    print(chunk, end="")

```

이 과정에서 `_auto_detect_provider` 메소드는 `LLM_BASE_URL`의 `"localhost"` 및 `:11434`를 구문 분석하여 `provider`을 `"ollama"`로 추론합니다. 이어서 `_resolve_credentials` 메소드는 Ollama에 대한 올바른 기본 매개변수를 설정합니다.

섹션 4.1.3의 기본 구현과 비교하여 현재 HelloAgentsLLM은 다음과 같은 중요한 이점을 갖습니다.

<div align="center">
<p>표 7.1 HelloAgentLLM 버전별 기능 비교</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/7-figures/table-01.png" alt="" width="90%"/>
</div>

위의 표 7.1에서 볼 수 있듯이 이러한 진화는 프레임워크 디자인의 중요한 원칙, 즉 **간단하게 시작하고 점진적으로 개선**을 구현합니다. 인터페이스 단순성을 유지하면서 기능적 완성도를 높였습니다.



## 7.3 프레임워크 인터페이스 구현

이전 섹션에서는 대규모 언어 모델과의 통신이라는 핵심 문제를 해결하는 핵심 구성 요소인 `HelloAgentsLLM`를 구축했습니다. 그러나 데이터 흐름을 처리하고, 구성을 관리하고, 예외를 처리하고, 상위 계층 애플리케이션 구성을 위한 명확하고 통합된 구조를 제공하려면 일련의 지원 인터페이스와 구성 요소가 여전히 필요합니다. 이 섹션에서는 다음 세 가지 핵심 파일을 다룹니다.

- `message.py`: 프레임워크 내에서 통합된 메시지 형식을 정의하여 에이전트와 모델 간의 정보 전송 표준화를 보장합니다.
- `config.py`: 중앙 집중식 구성 관리 솔루션을 제공하여 프레임워크 동작을 쉽게 조정하고 확장할 수 있습니다.
- `agent.py`: 모든 에이전트에 대한 추상 기본 클래스(`Agent`)를 정의하여 향후 다양한 유형의 에이전트 구현을 위한 통합 인터페이스 및 사양을 제공합니다.

### 7.3.1 메시지 클래스

에이전트와 대규모 언어 모델 간의 상호 작용에서 대화 기록은 중요한 컨텍스트입니다. 이 정보를 표준화된 방식으로 관리하기 위해 간단한 `Message` 클래스를 설계했습니다. 이는 이후의 컨텍스트 엔지니어링 장에서 확장될 것입니다.

```python
"""Message system"""
from typing import Optional, Dict, Any, Literal
from datetime import datetime
from pydantic import BaseModel

# Define message role type, restricting its values
MessageRole = Literal["user", "assistant", "system", "tool"]

class Message(BaseModel):
    """Message class"""

    content: str
    role: MessageRole
    timestamp: datetime = None
    metadata: Optional[Dict[str, Any]] = None

    def __init__(self, content: str, role: MessageRole, **kwargs):
        super().__init__(
            content=content,
            role=role,
            timestamp=kwargs.get('timestamp', datetime.now()),
            metadata=kwargs.get('metadata', {})
        )

    def to_dict(self) -> Dict[str, Any]:
        """Convert to dictionary format (OpenAI API format)"""
        return {
            "role": self.role,
            "content": self.content
        }

    def __str__(self) -> str:
        return f"[{self.role}] {self.content}"
```

이 클래스의 디자인에는 몇 가지 핵심 사항이 있습니다. 먼저, `role` 필드의 값을 `"user"`, `"assistant"`, `"system"`, `"tool"`부터 `typing.Literal`까지 4가지 유형으로 엄격하게 제한합니다. 이는 OpenAI API 사양에 직접적으로 대응하고 유형 안전성을 보장합니다. 두 개의 핵심 필드인 `content` 및 `role` 외에도 `timestamp` 및 `metadata`도 추가하여 로깅 및 향후 기능 확장을 위한 공간을 확보했습니다. 마지막으로 `to_dict()` 메소드는 내부적으로 사용되는 `Message` 객체를 OpenAI API와 호환되는 사전 형식으로 변환하는 핵심 기능 중 하나로, "내부적으로 풍부하고 외부적으로 호환 가능"이라는 설계 원칙을 구현합니다.

### 7.3.2 구성 클래스

`Config` 클래스의 역할은 하드 코딩된 구성 매개변수를 코드에 중앙집중화하고 환경 변수 읽기를 지원하는 것입니다.

```python
"""Configuration management"""
import os
from typing import Optional, Dict, Any
from pydantic import BaseModel

class Config(BaseModel):
    """HelloAgents configuration class"""

    # LLM configuration
    default_model: str = "gpt-3.5-turbo"
    default_provider: str = "openai"
    temperature: float = 0.7
    max_tokens: Optional[int] = None

    # System configuration
    debug: bool = False
    log_level: str = "INFO"

    # Other configuration
    max_history_length: int = 100

    @classmethod
    def from_env(cls) -> "Config":
        """Create configuration from environment variables"""
        return cls(
            debug=os.getenv("DEBUG", "false").lower() == "true",
            log_level=os.getenv("LOG_LEVEL", "INFO"),
            temperature=float(os.getenv("TEMPERATURE", "0.7")),
            max_tokens=int(os.getenv("MAX_TOKENS")) if os.getenv("MAX_TOKENS") else None,
        )

    def to_dict(self) -> Dict[str, Any]:
        """Convert to dictionary"""
        return self.dict()
```

먼저 구성 항목을 논리적으로 `LLM configuration`, `System configuration` 등으로 나누어 구조를 한눈에 알 수 있도록 합니다. 둘째, 각 구성 항목에는 합리적인 기본값이 있어 프레임워크가 구성이 없는 상태에서도 작동할 수 있습니다. 가장 핵심은 `from_env()` 클래스 메서드로, 이를 통해 사용자는 코드를 수정하지 않고 환경 변수를 설정하여 기본 구성을 재정의할 수 있으며, 이는 특히 다른 환경에 배포할 때 유용합니다.

### 7.3.3 에이전트 추상 기본 클래스

`Agent` 클래스는 전체 프레임워크의 최상위 수준 추상화입니다. 에이전트가 가져야 하는 일반적인 동작과 속성을 정의하지만 특정 구현 방법에 대해서는 신경 쓰지 않습니다. 우리는 모든 구체적인 에이전트 구현 (such as ⟪P2⟫, ⟪P3⟫, etc. in subsequent chapters)이 동일한 "인터페이스"를 따르도록 하는 Python의 `abc`(추상 기본 클래스) 모듈을 통해 이를 구현합니다.

```python
"""Agent base class"""
from abc import ABC, abstractmethod
from typing import Optional, Any
from .message import Message
from .llm import HelloAgentsLLM
from .config import Config

class Agent(ABC):
    """Agent base class"""

    def __init__(
        self,
        name: str,
        llm: HelloAgentsLLM,
        system_prompt: Optional[str] = None,
        config: Optional[Config] = None
    ):
        self.name = name
        self.llm = llm
        self.system_prompt = system_prompt
        self.config = config or Config()
        self._history: list[Message] = []

    @abstractmethod
    def run(self, input_text: str, **kwargs) -> str:
        """Run Agent"""
        pass

    def add_message(self, message: Message):
        """Add message to history"""
        self._history.append(message)

    def clear_history(self):
        """Clear history"""
        self._history.clear()

    def get_history(self) -> list[Message]:
        """Get history"""
        return self._history.copy()

    def __str__(self) -> str:
        return f"Agent(name={self.name}, provider={self.llm.provider})"
```

이 클래스의 디자인은 객체 지향 프로그래밍의 추상화 원칙을 구현합니다. 첫째, `ABC`을 상속받아 직접 인스턴스화할 수 없는 추상 클래스로 정의된다. 생성자 `__init__`는 이름, LLM 인스턴스, 시스템 프롬프트 및 구성과 같은 에이전트의 핵심 종속성을 명확하게 정의합니다. 가장 중요한 부분은 `@abstractmethod`로 장식된 `run` 메서드로, 모든 하위 클래스가 이 메서드를 구현하도록 하여 모든 에이전트가 통합된 실행 진입점을 갖도록 보장합니다. 또한 기본 클래스는 `Message` 클래스와 협력하여 구성 요소 간의 연결을 반영하는 공통 이력 관리 방법도 제공합니다.

이 시점에서 우리는 `HelloAgents` 프레임워크의 핵심 기본 구성 요소의 설계 및 구현을 완료했습니다.

## 7.4 에이전트 패러다임의 프레임워크 구현

이 섹션의 내용에서는 4장에서 구축한 세 가지 클래식 Agent 패러다임(ReAct, Plan-and-Solve, Reflection)을 기반으로 프레임워크 리팩토링을 수행하고 SimpleAgent를 기본 대화 패러다임으로 추가합니다. 우리는 이러한 독립적인 에이전트 구현을 통합 아키텍처를 기반으로 하는 프레임워크 구성 요소로 변환할 것입니다. 이 리팩토링은 주로 다음 세 가지 핵심 목표를 중심으로 진행됩니다.

1. **프롬프트 엔지니어링의 체계적인 개선**: 특정 작업 중심에서 일반화된 디자인으로 전환하면서 4장의 프롬프트를 심층적으로 최적화하는 동시에 형식 제약 조건과 역할 정의를 강화합니다.
2. **인터페이스와 형식의 표준화 및 통일**: 모든 에이전트가 동일한 초기화 매개변수, 메서드 서명 및 기록 관리 메커니즘을 따르는 통합 에이전트 기본 클래스와 표준화된 실행 인터페이스를 설정합니다.
3. **고도로 구성 가능한 사용자 정의 기능**: 사용자 정의 프롬프트 템플릿, 구성 매개변수 및 실행 전략을 지원합니다.

### 7.4.1 SimpleAgent

SimpleAgent는 프레임워크 기반에서 완전한 대화형 에이전트를 구축하는 방법을 보여주는 가장 기본적인 에이전트 구현입니다. 기존 `SimpleAgent` 클래스를 확장하고 핵심 메서드를 재정의하여 보다 확장 가능한 버전을 구축하겠습니다. 먼저 프로젝트 디렉터리에 `my_simple_agent.py` 파일을 만듭니다.

```python
# my_simple_agent.py
from typing import Optional, Iterator
from hello_agents import SimpleAgent, HelloAgentsLLM, Config, Message

class MySimpleAgent(SimpleAgent):
    """
    Rewritten simple conversation Agent
    Demonstrates how to build a custom Agent by extending SimpleAgent
    """

    def __init__(
        self,
        name: str,
        llm: HelloAgentsLLM,
        system_prompt: Optional[str] = None,
        config: Optional[Config] = None,
        tool_registry: Optional['ToolRegistry'] = None,
        enable_tool_calling: bool = True
    ):
        super().__init__(name, llm, system_prompt, config)
        self.tool_registry = tool_registry
        self.enable_tool_calling = enable_tool_calling and tool_registry is not None
        print(f"✅ {name} initialization complete, tool calling: {'enabled' if self.enable_tool_calling else 'disabled'}")
```

다음으로 `run` 메서드를 재정의해야 합니다. SimpleAgent는 선택적 도구 호출 기능을 지원하며, 이는 후속 장에서 확장도 용이하게 합니다.

```python
# Continue adding in my_simple_agent.py
import re

class MySimpleAgent(SimpleAgent):
    # ... previous __init__ method

    def run(self, input_text: str, max_tool_iterations: int = 3, **kwargs) -> str:
        """
        Rewritten run method - implements simple conversation logic, supports optional tool calling
        """
        print(f"🤖 {self.name} is processing: {input_text}")

        # Build message list
        messages = []

        # Add system message (may include tool information)
        enhanced_system_prompt = self._get_enhanced_system_prompt()
        messages.append({"role": "system", "content": enhanced_system_prompt})

        # Add history messages
        for msg in self._history:
            messages.append({"role": msg.role, "content": msg.content})

        # Add current user message
        messages.append({"role": "user", "content": input_text})

        # If tool calling is not enabled, use simple conversation logic
        if not self.enable_tool_calling:
            response = self.llm.invoke(messages, **kwargs)
            self.add_message(Message(input_text, "user"))
            self.add_message(Message(response, "assistant"))
            print(f"✅ {self.name} response complete")
            return response

        # Logic supporting multiple rounds of tool calling
        return self._run_with_tools(messages, input_text, max_tool_iterations, **kwargs)

    def _get_enhanced_system_prompt(self) -> str:
        """Build enhanced system prompt, including tool information"""
        base_prompt = self.system_prompt or "You are a helpful AI assistant."

        if not self.enable_tool_calling or not self.tool_registry:
            return base_prompt

        # Get tool description
        tools_description = self.tool_registry.get_tools_description()
        if not tools_description or tools_description == "No tools available":
            return base_prompt

        tools_section = "\n\n## Available Tools\n"
        tools_section += "You can use the following tools to help answer questions:\n"
        tools_section += tools_description + "\n"

        tools_section += "\n## Tool Calling Format\n"
        tools_section += "When you need to use a tool, please use the following format:\n"
        tools_section += "`[TOOL_CALL:{tool_name}:{parameters}]`\n"
        tools_section += "For example: `[TOOL_CALL:search:Python programming]` or `[TOOL_CALL:memory:recall=user information]`\n\n"
        tools_section += "Tool calling results will be automatically inserted into the conversation, and then you can continue answering based on the results.\n"

        return base_prompt + tools_section
```

이제 도구 호출의 핵심 논리를 구현합니다.

```python
# Continue adding in my_simple_agent.py
class MySimpleAgent(SimpleAgent):
    # ... previous methods

    def _run_with_tools(self, messages: list, input_text: str, max_tool_iterations: int, **kwargs) -> str:
        """Running logic supporting tool calling"""
        current_iteration = 0
        final_response = ""

        while current_iteration < max_tool_iterations:
            # Call LLM
            response = self.llm.invoke(messages, **kwargs)

            # Check if there are tool calls
            tool_calls = self._parse_tool_calls(response)

            if tool_calls:
                print(f"🔧 Detected {len(tool_calls)} tool calls")
                # Execute all tool calls and collect results
                tool_results = []
                clean_response = response

                for call in tool_calls:
                    result = self._execute_tool_call(call['tool_name'], call['parameters'])
                    tool_results.append(result)
                    # Remove tool call markers from response
                    clean_response = clean_response.replace(call['original'], "")

                # Build message containing tool results
                messages.append({"role": "assistant", "content": clean_response})

                # Add tool results
                tool_results_text = "\n\n".join(tool_results)
                messages.append({"role": "user", "content": f"Tool execution results:\n{tool_results_text}\n\nPlease provide a complete answer based on these results."})

                current_iteration += 1
                continue

            # No tool calls, this is the final answer
            final_response = response
            break

        # If maximum iterations exceeded, get last response
        if current_iteration >= max_tool_iterations and not final_response:
            final_response = self.llm.invoke(messages, **kwargs)

        # Save to history
        self.add_message(Message(input_text, "user"))
        self.add_message(Message(final_response, "assistant"))
        print(f"✅ {self.name} response complete")

        return final_response

    def _parse_tool_calls(self, text: str) -> list:
        """Parse tool calls in text"""
        pattern = r'\[TOOL_CALL:([^:]+):([^\]]+)\]'
        matches = re.findall(pattern, text)

        tool_calls = []
        for tool_name, parameters in matches:
            tool_calls.append({
                'tool_name': tool_name.strip(),
                'parameters': parameters.strip(),
                'original': f'[TOOL_CALL:{tool_name}:{parameters}]'
            })

        return tool_calls

    def _execute_tool_call(self, tool_name: str, parameters: str) -> str:
        """Execute tool call"""
        if not self.tool_registry:
            return f"❌ Error: Tool registry not configured"

        try:
            # Intelligent parameter parsing
            if tool_name == 'calculator':
                # Calculator tool directly passes expression
                result = self.tool_registry.execute_tool(tool_name, parameters)
            else:
                # Other tools use intelligent parameter parsing
                param_dict = self._parse_tool_parameters(tool_name, parameters)
                tool = self.tool_registry.get_tool(tool_name)
                if not tool:
                    return f"❌ Error: Tool '{tool_name}' not found"
                result = tool.run(param_dict)

            return f"🔧 Tool {tool_name} execution result:\n{result}"

        except Exception as e:
            return f"❌ Tool call failed: {str(e)}"

    def _parse_tool_parameters(self, tool_name: str, parameters: str) -> dict:
        """Intelligently parse tool parameters"""
        param_dict = {}

        if '=' in parameters:
            # Format: key=value or action=search,query=Python
            if ',' in parameters:
                # Multiple parameters: action=search,query=Python,limit=3
                pairs = parameters.split(',')
                for pair in pairs:
                    if '=' in pair:
                        key, value = pair.split('=', 1)
                        param_dict[key.strip()] = value.strip()
            else:
                # Single parameter: key=value
                key, value = parameters.split('=', 1)
                param_dict[key.strip()] = value.strip()
        else:
            # Directly pass parameters, intelligently infer based on tool type
            if tool_name == 'search':
                param_dict = {'query': parameters}
            elif tool_name == 'memory':
                param_dict = {'action': 'search', 'query': parameters}
            else:
                param_dict = {'input': parameters}

        return param_dict
```

또한 사용자 정의 에이전트에 스트리밍 응답 기능과 편의 메서드를 추가할 수도 있습니다.

```python
# Continue adding in my_simple_agent.py
class MySimpleAgent(SimpleAgent):
    # ... previous methods

    def stream_run(self, input_text: str, **kwargs) -> Iterator[str]:
        """
        Custom streaming run method
        """
        print(f"🌊 {self.name} starting streaming processing: {input_text}")

        messages = []

        if self.system_prompt:
            messages.append({"role": "system", "content": self.system_prompt})

        for msg in self._history:
            messages.append({"role": msg.role, "content": msg.content})

        messages.append({"role": "user", "content": input_text})

        # Stream call LLM
        full_response = ""
        print("📝 Real-time response: ", end="")
        for chunk in self.llm.stream_invoke(messages, **kwargs):
            full_response += chunk
            print(chunk, end="", flush=True)
            yield chunk

        print()  # New line

        # Save complete conversation to history
        self.add_message(Message(input_text, "user"))
        self.add_message(Message(full_response, "assistant"))
        print(f"✅ {self.name} streaming response complete")

    def add_tool(self, tool) -> None:
        """Add tool to Agent (convenience method)"""
        if not self.tool_registry:
            from hello_agents import ToolRegistry
            self.tool_registry = ToolRegistry()
            self.enable_tool_calling = True

        self.tool_registry.register_tool(tool)
        print(f"🔧 Tool '{tool.name}' added")

    def has_tools(self) -> bool:
        """Check if tools are available"""
        return self.enable_tool_calling and self.tool_registry is not None

    def remove_tool(self, tool_name: str) -> bool:
        """Remove tool (convenience method)"""
        if self.tool_registry:
            self.tool_registry.unregister(tool_name)
            return True
        return False

    def list_tools(self) -> list:
        """List all available tools"""
        if self.tool_registry:
            return self.tool_registry.list_tools()
        return []
```

테스트 파일 `test_simple_agent.py` 만들기:

```python
# test_simple_agent.py
from dotenv import load_dotenv
from hello_agents import HelloAgentsLLM, ToolRegistry
from hello_agents.tools import CalculatorTool
from my_simple_agent import MySimpleAgent

# Load environment variables
load_dotenv()

# Create LLM instance
llm = HelloAgentsLLM()

# Test 1: Basic conversation Agent (no tools)
print("=== Test 1: Basic Conversation ===")
basic_agent = MySimpleAgent(
    name="Basic Assistant",
    llm=llm,
    system_prompt="You are a friendly AI assistant, please answer questions in a concise and clear manner."
)

response1 = basic_agent.run("Hello, please introduce yourself")
print(f"Basic conversation response: {response1}\n")

# Test 2: Agent with tools
print("=== Test 2: Tool-Enhanced Conversation ===")
tool_registry = ToolRegistry()
calculator = CalculatorTool()
tool_registry.register_tool(calculator)

enhanced_agent = MySimpleAgent(
    name="Enhanced Assistant",
    llm=llm,
    system_prompt="You are an intelligent assistant that can use tools to help users.",
    tool_registry=tool_registry,
    enable_tool_calling=True
)

response2 = enhanced_agent.run("Please help me calculate 15 * 8 + 32")
print(f"Tool-enhanced response: {response2}\n")

# Test 3: Streaming response
print("=== Test 3: Streaming Response ===")
print("Streaming response: ", end="")
for chunk in basic_agent.stream_run("Please explain what artificial intelligence is"):
    pass  # Content already printed in real-time in stream_run

# Test 4: Dynamic tool addition
print("\n=== Test 4: Dynamic Tool Management ===")
print(f"Before adding tool: {basic_agent.has_tools()}")
basic_agent.add_tool(calculator)
print(f"After adding tool: {basic_agent.has_tools()}")
print(f"Available tools: {basic_agent.list_tools()}")

# View conversation history
print(f"\nConversation history: {len(basic_agent.get_history())} messages")
```

이 섹션에서는 `Agent` 기본 클래스를 상속하여 프레임워크 사양을 따르는 완전한 기능을 갖춘 기본 대화형 에이전트 `MySimpleAgent`를 성공적으로 구축했습니다. 기본적인 대화 지원은 물론, 선택적 도구 호출 기능, 스트리밍 응답, 편리한 도구 관리 방법까지 갖췄습니다.

### 7.4.2 리액트에이전트

프레임워크 기반 ReActAgent는 핵심 로직을 그대로 유지하면서 주로 프레임워크의 도구 시스템과의 신속한 최적화 및 통합을 통해 코드 구성 및 유지 관리성을 개선합니다.

(1) 프롬프트 템플릿 개선

혼란을 피하기 위해 "한 번에 한 단계만 실행할 수 있음"을 강조하면서 원래 형식 요구 사항을 유지하고 두 가지 유형의 작업에 대한 사용 시나리오를 명확하게 합니다.

```python
MY_REACT_PROMPT = """You are an AI assistant with reasoning and action capabilities. You can analyze problems through thinking, then call appropriate tools to obtain information, and finally provide accurate answers.

## Available Tools
{tools}

## Workflow
Please respond strictly in the following format, executing only one step at a time:

Thought: Analyze the current problem and think about what information is needed or what action to take.
Action: Choose an action, the format must be one of the following:
- `{{tool_name}}[{{tool_input}}]` - Call specified tool
- `Finish[final answer]` - When you have enough information to give a final answer

## Important Reminders
1. Each response must include both Thought and Action parts
2. Tool call format must strictly follow: tool_name[parameters]
3. Only use Finish when you are confident you have enough information to answer the question
4. If the information returned by the tool is insufficient, continue using other tools or different parameters of the same tool

## Current Task
**Question:** {question}

## Execution History
{history}

Now begin your reasoning and action:
"""
```

(2) 재작성된 ReActAgent 구현 완료

ReActAgent를 다시 작성하려면 `my_react_agent.py` 파일을 만듭니다.

```python
# my_react_agent.py
import re
from typing import Optional, List, Tuple
from hello_agents import ReActAgent, HelloAgentsLLM, Config, Message, ToolRegistry

class MyReActAgent(ReActAgent):
    """
    Rewritten ReAct Agent - Agent combining reasoning and action
    """

    def __init__(
        self,
        name: str,
        llm: HelloAgentsLLM,
        tool_registry: ToolRegistry,
        system_prompt: Optional[str] = None,
        config: Optional[Config] = None,
        max_steps: int = 5,
        custom_prompt: Optional[str] = None
    ):
        super().__init__(name, llm, system_prompt, config)
        self.tool_registry = tool_registry
        self.max_steps = max_steps
        self.current_history: List[str] = []
        self.prompt_template = custom_prompt if custom_prompt else MY_REACT_PROMPT
        print(f"✅ {name} initialization complete, max steps: {max_steps}")
```

초기화 매개변수의 의미는 다음과 같습니다.

- `name`: 에이전트 이름.
- `llm`: 대규모 언어 모델과의 통신을 담당하는 `HelloAgentsLLM` 인스턴스입니다.
- `tool_registry`: 에이전트에서 사용할 수 있는 도구를 관리하고 실행하는 데 사용되는 `ToolRegistry`의 인스턴스입니다.
- `system_prompt`: 에이전트의 역할과 행동 지침을 설정하는 데 사용되는 시스템 프롬프트입니다.
- `config`: 프레임워크 수준 설정을 전달하는 데 사용되는 구성 개체입니다.
- `max_steps`: ReAct 루프의 최대 실행 단계로 무한 루프를 방지합니다.
- `custom_prompt`: 기본 ReAct 프롬프트를 대체하는 데 사용되는 사용자 정의 프롬프트 템플릿입니다.

프레임워크 기반 ReActAgent는 실행 프로세스를 명확한 단계로 분해합니다.

```python
def run(self, input_text: str, **kwargs) -> str:
    """Run ReAct Agent"""
    self.current_history = []
    current_step = 0

    print(f"\n🤖 {self.name} starting to process question: {input_text}")

    while current_step < self.max_steps:
        current_step += 1
        print(f"\n--- Step {current_step} ---")

        # 1. Build prompt
        tools_desc = self.tool_registry.get_tools_description()
        history_str = "\n".join(self.current_history)
        prompt = self.prompt_template.format(
            tools=tools_desc,
            question=input_text,
            history=history_str
        )

        # 2. Call LLM
        messages = [{"role": "user", "content": prompt}]
        response_text = self.llm.invoke(messages, **kwargs)

        # 3. Parse output
        thought, action = self._parse_output(response_text)

        # 4. Check completion condition
        if action and action.startswith("Finish"):
            final_answer = self._parse_action_input(action)
            self.add_message(Message(input_text, "user"))
            self.add_message(Message(final_answer, "assistant"))
            return final_answer

        # 5. Execute tool call
        if action:
            tool_name, tool_input = self._parse_action(action)
            observation = self.tool_registry.execute_tool(tool_name, tool_input)
            self.current_history.append(f"Action: {action}")
            self.current_history.append(f"Observation: {observation}")

    # Reached maximum steps
    final_answer = "Sorry, I cannot complete this task within the limited number of steps."
    self.add_message(Message(input_text, "user"))
    self.add_message(Message(final_answer, "assistant"))
    return final_answer
```

위의 리팩토링을 통해 ReAct 패러다임을 프레임워크에 성공적으로 통합했습니다. 핵심 개선 사항은 통합된 `ToolRegistry` 인터페이스를 활용하고 구성 가능하고 보다 엄격한 프롬프트 템플릿을 통해 에이전트의 사고-행동 루프 실행의 안정성을 향상시키는 것입니다. ReAct 테스트 케이스의 경우 툴 호출이 필요하므로 문서 말미에 테스트 코드를 제공합니다.

### 7.4.3 반사 에이전트

이러한 유형의 에이전트는 이미 4장에서 핵심 로직을 구현했으므로 여기서는 해당 프롬프트만 제공합니다. 4장의 코드 생성을 위한 구체적인 프롬프트와 달리 프레임워크 버전은 일반화된 디자인을 채택하여 텍스트 생성, 분석, 생성과 같은 다양한 시나리오에 적합하고 `custom_prompts` 매개변수를 통해 사용자의 심층적인 사용자 정의를 지원합니다.

```python
DEFAULT_PROMPTS = {
    "initial": """
Please complete the task according to the following requirements:

Task: {task}

Please provide a complete and accurate answer.
""",
    "reflect": """
Please carefully review the following answer and identify possible problems or areas for improvement:

# Original Task:
{task}

# Current Answer:
{content}

Please analyze the quality of this answer, point out deficiencies, and provide specific improvement suggestions.
If the answer is already good, please respond "No improvement needed".
""",
    "refine": """
Please improve your answer based on the feedback:

# Original Task:
{task}

# Previous Answer:
{last_attempt}

# Feedback:
{feedback}

Please provide an improved answer.
"""
}
```

4장의 코드와 위의 ReAct 구현을 기반으로 자신만의 MyReflectionAgent를 구축해 볼 수 있습니다. 아래는 아이디어 검증을 위한 테스트 코드입니다.

```python
# test_reflection_agent.py
from dotenv import load_dotenv
from hello_agents import HelloAgentsLLM
from my_reflection_agent import MyReflectionAgent

load_dotenv()
llm = HelloAgentsLLM()

# Use default general prompts
general_agent = MyReflectionAgent(name="My Reflection Assistant", llm=llm)

# Use custom code generation prompts (similar to Chapter 4)
code_prompts = {
    "initial": "You are a Python expert, please write a function: {task}",
    "reflect": "Please review the algorithm efficiency of the code:\nTask: {task}\nCode: {content}",
    "refine": "Please optimize the code based on feedback:\nTask: {task}\nFeedback: {feedback}"
}
code_agent = MyReflectionAgent(
    name="My Code Generation Assistant",
    llm=llm,
    custom_prompts=code_prompts
)

# Test usage
result = general_agent.run("Write a short article about the development history of artificial intelligence")
print(f"Final result: {result}")
```

### 7.4.4 PlanAndSolveAgent

4장의 자유 텍스트 계획 출력과 달리 프레임워크 버전에서는 Planner가 계획을 Python 목록 형식으로 출력하도록 요구하고 후속 단계의 안정적인 실행을 보장하기 위한 완전한 예외 처리 메커니즘을 제공합니다. 프레임워크 기반 계획 및 해결 프롬프트:

````bash
# Default planner prompt template
DEFAULT_PLANNER_PROMPT = """
You are a top AI planning expert. Your task is to decompose complex problems raised by users into an action plan consisting of multiple simple steps.
Please ensure that each step in the plan is an independent, executable subtask and is strictly arranged in logical order.
Your output must be a Python list, where each element is a string describing a subtask.

Question: {question}

Please output your plan strictly in the following format:
```python
["1단계", "2단계", "3단계", ...]
```
"""

# Default executor prompt template
DEFAULT_EXECUTOR_PROMPT = """
You are a top AI execution expert. Your task is to solve problems step by step strictly according to the given plan.
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
````

이 섹션에서는 사용자가 직접 설계하고 구현할 수 있는 포괄적인 테스트 파일 `test_plan_solve_agent.py`을 제공합니다.

```python
# test_plan_solve_agent.py
from dotenv import load_dotenv
from hello_agents.core.llm import HelloAgentsLLM
from my_plan_solve_agent import MyPlanAndSolveAgent

# Load environment variables
load_dotenv()

# Create LLM instance
llm = HelloAgentsLLM()

# Create custom PlanAndSolveAgent
agent = MyPlanAndSolveAgent(
    name="My Planning Execution Assistant",
    llm=llm
)

# Test complex problem
question = "A fruit store sold 15 apples on Monday. The number of apples sold on Tuesday was twice that of Monday. The number sold on Wednesday was 5 less than Tuesday. How many apples were sold in total over these three days?"

result = agent.run(question)
print(f"\nFinal result: {result}")

# View conversation history
print(f"Conversation history: {len(agent.get_history())} messages")
```

마지막으로 새 프롬프트를 추가하고 `custom_prompt`를 구현하여 맞춤 프롬프트를 로드할 수 있습니다.

```python
# Create custom prompts specifically for math problems
math_prompts = {
    "planner": """
You are a math problem planning expert. Please decompose the math problem into calculation steps:

Question: {question}

Output format:
python
["Calculation step 1", "Calculation step 2", "Sum total"]

""",
    "executor": """
You are a math calculation expert. Please calculate the current step:

Question: {question}
Plan: {plan}
History: {history}
Current step: {current_step}

Please only output the numerical result:
"""
}

# Create math-specific Agent using custom prompts
math_agent = MyPlanAndSolveAgent(
    name="Math Calculation Assistant",
    llm=llm,
    custom_prompts=math_prompts
)

# Test math problem
math_result = math_agent.run(question)
print(f"Math-specific Agent result: {math_result}")
```

표 7.2에서 볼 수 있듯이 이 프레임워크 리팩토링을 통해 우리는 4장의 다양한 에이전트 패러다임의 핵심 기능을 유지했을 뿐만 아니라 코드 구성, 유지 관리성 및 확장성을 크게 향상시켰습니다. 이제 모든 에이전트는 각자의 특성과 장점을 유지하면서 통합 인프라를 공유합니다.

<div align="center">
<p>표 7.2 여러 장에 걸친 에이전트 구현 비교</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/7-figures/table-02.png" alt="" width="90%"/>
</div>

### 7.4.5 FunctionCallAgent

FunctionCallAgent는 OpenAI의 기본 함수 호출 메커니즘을 기반으로 hello-agents 버전 0.2.8 이후에 도입된 에이전트입니다. OpenAI의 함수 호출 기능을 사용하여 에이전트를 구축하는 방법을 보여줍니다.
다음 기능을 지원합니다.

- _build_tool_schemas: 도구 설명을 통해 OpenAI 함수 호출 스키마를 구성합니다.
- _extract_message_content: OpenAI 응답에서 텍스트 콘텐츠를 추출합니다.
- _parse_function_call_arguments: 모델에서 반환된 JSON 문자열 매개변수를 구문 분석합니다.
- _convert_parameter_types: 매개변수 유형을 변환합니다.

이러한 기능은 기본 OpenAI 함수 호출 기능을 활성화하여 프롬프트가 제한된 접근 방식에 비해 더 강력한 견고성을 제공합니다.

```python
def _invoke_with_tools(self, messages: list[dict[str, Any]], tools: list[dict[str, Any]], tool_choice: Union[str, dict], **kwargs):
        """Invoke underlying OpenAI client to execute function calls"""
        client = getattr(self.llm, "_client", None)
        if client is None:
            raise RuntimeError("HelloAgentsLLM client not properly initialized, cannot execute function calls.")

        client_kwargs = dict(kwargs)
        client_kwargs.setdefault("temperature", self.llm.temperature)
        if self.llm.max_tokens is not None:
            client_kwargs.setdefault("max_tokens", self.llm.max_tokens)

        return client.chat.completions.create(
            model=self.llm.model,
            messages=messages,
            tools=tools,
            tool_choice=tool_choice,
            **client_kwargs,
        )

# Internal logic wraps OpenAI native function calling
# OpenAI native function calling example
from openai import OpenAI
client = OpenAI()

tools = [
  {
    "type": "function",
    "function": {
      "name": "get_current_weather",
      "description": "Get the current weather in a given location",
      "parameters": {
        "type": "object",
        "properties": {
          "location": {
            "type": "string",
            "description": "The city and state, e.g. San Francisco, CA",
          },
          "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]},
        },
        "required": ["location"],
      },
    }
  }
]
messages = [{"role": "user", "content": "What's the weather like in Boston today?"}]
completion = client.chat.completions.create(
  model="gpt-5",
  messages=messages,
  tools=tools,
  tool_choice="auto"
)

print(completion)
```

## 7.5 도구 시스템

이 섹션의 내용에서는 이전에 구축된 에이전트 인프라를 기반으로 하는 도구 시스템의 설계 및 구현을 심층적으로 탐색합니다. 인프라 구축부터 시작하여 점차 맞춤형 개발 설계까지 진행해 나가겠습니다. 이 섹션의 학습 목표는 다음 세 가지 핵심 측면을 중심으로 이루어집니다.

1. **통합 도구 추상화 및 관리**: 도구 개발, 등록, 검색 및 실행을 위한 통합 인프라를 제공하기 위해 표준화된 도구 기본 클래스 및 ToolRegistry 등록 메커니즘을 설정합니다.

2. **실습 기반 도구 개발**: 사례 연구로 수학적 계산 도구를 사용하여 사용자 정의 도구를 설계하고 구현하는 방법을 보여줌으로써 독자가 도구 개발의 전체 프로세스를 마스터할 수 있도록 합니다.

3. **고급 통합 및 최적화 전략**: 다중 소스 검색 도구 설계를 통해 여러 외부 서비스를 통합하고 지능형 백엔드 선택, 결과 병합 및 내결함성을 구현하는 방법을 시연하고 복잡한 시나리오에서 도구 시스템의 설계 사고를 반영합니다.

### 7.5.1 도구 기본 클래스 및 등록 메커니즘 설계

확장 가능한 도구 시스템을 구축하려면 먼저 표준화된 인프라 세트를 구축해야 합니다. 이 인프라에는 도구 기본 클래스, ToolRegistry 레지스트리 및 도구 관리 메커니즘이 포함됩니다.

(1) 도구 기본 클래스의 추상 설계

Tool 기본 클래스는 전체 도구 시스템의 핵심 추상화로, 모든 도구가 따라야 하는 인터페이스 사양을 정의합니다.

````python
class Tool(ABC):
    """Tool base class"""

    def __init__(self, name: str, description: str):
        self.name = name
        self.description = description

    @abstractmethod
    def run(self, parameters: Dict[str, Any]) -> str:
        """Execute tool"""
        pass

    @abstractmethod
    def get_parameters(self) -> List[ToolParameter]:
        """Get tool parameter definitions"""
        pass
````
이 디자인은 객체 지향 디자인의 핵심 아이디어를 구현합니다. 통합된 `run` 메소드 인터페이스를 통해 모든 도구를 일관된 방식으로 실행하고 사전 매개변수를 허용하고 문자열 결과를 반환하여 프레임워크 일관성을 보장할 수 있습니다. 동시에 도구에는 자체 설명 기능이 있습니다. `get_parameters` 메소드를 통해 호출자에게 필요한 매개변수가 무엇인지 명확하게 알릴 수 있습니다. 이 자체 검사 메커니즘은 자동화된 문서 생성 및 매개변수 검증을 위한 기반을 제공합니다. 이름 및 설명과 같은 메타데이터 디자인은 도구 시스템에 우수한 검색 가능성과 이해 가능성을 제공합니다.

(2) ToolParameter 매개변수 정의 시스템

복잡한 매개변수 검증 및 문서 생성을 지원하기 위해 ToolParameter 클래스를 설계했습니다.

````python
class ToolParameter(BaseModel):
    """Tool parameter definition"""
    name: str
    type: str
    description: str
    required: bool = True
    default: Any = None
````
이 설계를 통해 도구는 매개변수 요구 사항을 정확하게 설명하고 유형 확인, 기본값 설정 및 자동 문서 생성을 지원합니다.

(3) ToolRegistry 구현

ToolRegistry는 도구 등록, 검색, 실행 등의 핵심 기능을 제공하는 도구 시스템의 관리 허브입니다. 이 섹션에서는 주로 다음 기능을 사용합니다.

````python
class ToolRegistry:
    """HelloAgents tool registry"""

    def __init__(self):
        self._tools: dict[str, Tool] = {}
        self._functions: dict[str, dict[str, Any]] = {}

    def register_tool(self, tool: Tool):
        """Register Tool object"""
        if tool.name in self._tools:
            print(f"⚠️ Warning: Tool '{tool.name}' already exists and will be overwritten.")
        self._tools[tool.name] = tool
        print(f"✅ Tool '{tool.name}' registered.")

    def register_function(self, name: str, description: str, func: Callable[[str], str]):
        """
        Directly register a function as a tool (convenient method)

        Args:
            name: Tool name
            description: Tool description
            func: Tool function, accepts string parameter, returns string result
        """
        if name in self._functions:
            print(f"⚠️ Warning: Tool '{name}' already exists and will be overwritten.")

        self._functions[name] = {
            "description": description,
            "func": func
        }
        print(f"✅ Tool '{name}' registered.")
````
ToolRegistry는 두 가지 등록 방법을 지원합니다.

1. **도구 개체 등록**: 복잡한 도구에 적합하며 완전한 매개변수 정의 및 검증을 지원합니다.
2. **직접 기능 등록**: 간단한 도구에 적합하며 기존 기능을 빠르게 통합합니다.

(4) 도구 발견 및 관리 메커니즘

레지스트리는 풍부한 도구 관리 기능을 제공합니다.

````python
def get_tools_description(self) -> str:
    """Get formatted description string of all available tools"""
    descriptions = []

    # Tool object descriptions
    for tool in self._tools.values():
        descriptions.append(f"- {tool.name}: {tool.description}")

    # Function tool descriptions
    for name, info in self._functions.items():
        descriptions.append(f"- {name}: {info['description']}")

    return "\n".join(descriptions) if descriptions else "No tools available"
````
이 방법으로 생성된 설명 문자열은 에이전트의 프롬프트를 작성하는 데 직접 사용될 수 있으며 에이전트에게 사용 가능한 도구가 무엇인지 알려줍니다.

### 7.5.2 맞춤형 도구 개발

인프라를 갖춘 상태에서 완전한 사용자 정의 도구를 개발하는 방법을 살펴보겠습니다. 수학적 계산 도구는 간단하고 직관적이기 때문에 좋은 예입니다. 가장 직접적인 방법은 ToolRegistry의 기능 등록 기능을 이용하는 것입니다.

사용자 정의 수학 계산 도구를 만들어 보겠습니다. 먼저 프로젝트 디렉터리에 `my_calculator_tool.py`을 만듭니다.

```python
# my_calculator_tool.py
import ast
import operator
import math
from hello_agents import ToolRegistry

def my_calculate(expression: str) -> str:
    """Simple mathematical calculation function"""
    if not expression.strip():
        return "Calculation expression cannot be empty"

    # Supported basic operations
    operators = {
        ast.Add: operator.add,      # +
        ast.Sub: operator.sub,      # -
        ast.Mult: operator.mul,     # *
        ast.Div: operator.truediv,  # /
    }

    # Supported basic functions
    functions = {
        'sqrt': math.sqrt,
        'pi': math.pi,
    }

    try:
        node = ast.parse(expression, mode='eval')
        result = _eval_node(node.body, operators, functions)
        return str(result)
    except:
        return "Calculation failed, please check expression format"

def _eval_node(node, operators, functions):
    """Simplified expression evaluation"""
    if isinstance(node, ast.Constant):
        return node.value
    elif isinstance(node, ast.BinOp):
        left = _eval_node(node.left, operators, functions)
        right = _eval_node(node.right, operators, functions)
        op = operators.get(type(node.op))
        return op(left, right)
    elif isinstance(node, ast.Call):
        func_name = node.func.id
        if func_name in functions:
            args = [_eval_node(arg, operators, functions) for arg in node.args]
            return functions[func_name](*args)
    elif isinstance(node, ast.Name):
        if node.id in functions:
            return functions[node.id]

def create_calculator_registry():
    """Create tool registry containing calculator"""
    registry = ToolRegistry()

    # Register calculator function
    registry.register_function(
        name="my_calculator",
        description="Simple mathematical calculation tool, supports basic operations (+,-,*,/) and sqrt function",
        func=my_calculate
    )

    return registry
```

이 도구는 기본 산술 연산을 지원할 뿐만 아니라 일반적으로 사용되는 수학 함수 및 상수도 다루므로 대부분의 계산 시나리오의 요구 사항을 충족합니다. 또한 이 파일을 직접 확장하여 보다 완전한 계산 기능을 만들 수도 있습니다. 기능을 확인하는 데 도움이 되는 테스트 파일 `test_my_calculator.py`을 제공합니다.

```python
# test_my_calculator.py
from dotenv import load_dotenv
from my_calculator_tool import create_calculator_registry

# Load environment variables
load_dotenv()

def test_calculator_tool():
    """Test custom calculator tool"""

    # Create registry containing calculator
    registry = create_calculator_registry()

    print("🧪 Testing Custom Calculator Tool\n")

    # Simple test cases
    test_cases = [
        "2 + 3",           # Basic addition
        "10 - 4",          # Basic subtraction
        "5 * 6",           # Basic multiplication
        "15 / 3",          # Basic division
        "sqrt(16)",        # Square root
    ]

    for i, expression in enumerate(test_cases, 1):
        print(f"Test {i}: {expression}")
        result = registry.execute_tool("my_calculator", expression)
        print(f"Result: {result}\n")

def test_with_simple_agent():
    """Test integration with SimpleAgent"""
    from hello_agents import HelloAgentsLLM

    # Create LLM client
    llm = HelloAgentsLLM()

    # Create registry containing calculator
    registry = create_calculator_registry()

    print("🤖 Integration Test with SimpleAgent:")

    # Simulate scenario where SimpleAgent uses tool
    user_question = "Please help me calculate sqrt(16) + 2 * 3"

    print(f"User question: {user_question}")

    # Use tool to calculate
    calc_result = registry.execute_tool("my_calculator", "sqrt(16) + 2 * 3")
    print(f"Calculation result: {calc_result}")

    # Build final answer
    final_messages = [
        {"role": "user", "content": f"The calculation result is {calc_result}, please answer the user's question in natural language: {user_question}"}
    ]

    print("\n🎯 SimpleAgent's answer:")
    response = llm.think(final_messages)
    for chunk in response:
        print(chunk, end="", flush=True)
    print("\n")

if __name__ == "__main__":
    test_calculator_tool()
    test_with_simple_agent()
```

이 단순화된 수학적 계산 도구 사례를 통해 우리는 간단한 계산 함수를 작성하고 ToolRegistry를 통해 등록한 다음 SimpleAgent와 통합하는 사용자 정의 도구를 빠르게 개발하는 방법을 배웠습니다. 보다 직관적인 관찰을 위해 코드의 실행 논리를 명확하게 이해하기 위해 그림 7.1이 여기에 제공됩니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/7-figures/01.png" alt="" width="90%"/>
<p>그림 7.1 HelloAgents 기반 SimpleAgent 작업 흐름</p>
</div>

### 7.5.3 다중 소스 검색 도구

실제 애플리케이션에서는 보다 강력한 기능을 제공하기 위해 여러 외부 서비스를 통합해야 하는 경우가 많습니다. 검색 도구는 보다 완전한 실제 정보를 제공하기 위해 여러 검색 엔진을 통합하는 일반적인 예입니다. 1장에서는 Tavily의 검색 API를 사용했고, 4장에서는 SerpApi의 검색 API를 사용했습니다. 따라서 이번에는 이 두 API를 사용하여 다중 소스 검색 기능을 구현합니다. 해당 Python 종속성을 설치하지 않은 경우 다음 스크립트를 실행할 수 있습니다.

```bash
pip install "hello-agents[search]==0.1.1"
```

(1) 검색 도구를 위한 통합 인터페이스 디자인

HelloAgents 프레임워크에 내장된 SearchTool은 고급 다중 소스 검색 도구를 설계하는 방법을 보여줍니다.

````python
class SearchTool(Tool):
    """
    Intelligent hybrid search tool

    Supports multiple search engine backends, intelligently selects the best search source:
    1. Hybrid mode (hybrid) - Intelligently selects TAVILY or SERPAPI
    2. Tavily API (tavily) - Professional AI search
    3. SerpApi (serpapi) - Traditional Google search
    """

    def __init__(self, backend: str = "hybrid", tavily_key: Optional[str] = None, serpapi_key: Optional[str] = None):
        super().__init__(
            name="search",
            description="An intelligent web search engine. Supports hybrid search mode, automatically selects the best search source."
        )
        self.backend = backend
        self.tavily_key = tavily_key or os.getenv("TAVILY_API_KEY")
        self.serpapi_key = serpapi_key or os.getenv("SERPAPI_API_KEY")
        self.available_backends = []
        self._setup_backends()
````
이 디자인의 핵심 아이디어는 사용 가능한 API 키 및 종속성 라이브러리를 기반으로 최상의 검색 백엔드를 자동으로 선택하는 것입니다.

(2) TAVILY 및 SERPAPI 검색 소스 통합 전략

프레임워크는 지능형 백엔드 선택 논리를 구현합니다.

````python
def _search_hybrid(self, query: str) -> str:
    """Hybrid search - intelligently select the best search source"""
    # Prioritize Tavily (AI-optimized search)
    if "tavily" in self.available_backends:
        try:
            return self._search_tavily(query)
        except Exception as e:
            print(f"⚠️ Tavily search failed: {e}")
            # If Tavily fails, try SerpApi
            if "serpapi" in self.available_backends:
                print("🔄 Switching to SerpApi search")
                return self._search_serpapi(query)

    # If Tavily is unavailable, use SerpApi
    elif "serpapi" in self.available_backends:
        try:
            return self._search_serpapi(query)
        except Exception as e:
            print(f"⚠️ SerpApi search failed: {e}")

    # If both are unavailable, prompt user to configure API
    return "❌ No available search sources, please configure TAVILY_API_KEY or SERPAPI_API_KEY environment variables"
````
이 설계는 고가용성 시스템의 핵심 개념을 구현합니다. 성능 저하 메커니즘을 통해 시스템은 최적의 검색 소스에서 사용 가능한 대안으로 점진적으로 성능이 저하될 수 있습니다. 모든 검색 소스를 사용할 수 없는 경우 사용자에게 올바른 API 키를 구성하라는 메시지가 명확하게 표시됩니다.

(3) 검색결과의 통일된 형식

다양한 검색 엔진은 다양한 형식으로 결과를 반환합니다. 프레임워크는 통합된 형식 지정 방법을 통해 이를 처리합니다.

````python
def _search_tavily(self, query: str) -> str:
    """Search using Tavily"""
    response = self.tavily_client.search(
        query=query,
        search_depth="basic",
        include_answer=True,
        max_results=3
    )

    result = f"🎯 Tavily AI search results: {response.get('answer', 'No direct answer found')}\n\n"

    for i, item in enumerate(response.get('results', [])[:3], 1):
        result += f"[{i}] {item.get('title', '')}\n"
        result += f"    {item.get('content', '')[:200]}...\n"
        result += f"    Source: {item.get('url', '')}\n\n"

    return result
````

프레임워크의 디자인 철학을 기반으로 우리는 자체적인 고급 검색 도구를 만들 수 있습니다. 이번에는 클래스 기반 접근 방식을 사용하여 다양한 구현 방법을 보여줍니다. `my_advanced_search.py` 생성:

```python
# my_advanced_search.py
import os
from typing import Optional, List, Dict, Any
from hello_agents import ToolRegistry

class MyAdvancedSearchTool:
    """
    Custom advanced search tool class
    Demonstrates design patterns for multi-source integration and intelligent selection
    """

    def __init__(self):
        self.name = "my_advanced_search"
        self.description = "Intelligent search tool, supports multiple search sources, automatically selects best results"
        self.search_sources = []
        self._setup_search_sources()

    def _setup_search_sources(self):
        """Set up available search sources"""
        # Check Tavily availability
        if os.getenv("TAVILY_API_KEY"):
            try:
                from tavily import TavilyClient
                self.tavily_client = TavilyClient(api_key=os.getenv("TAVILY_API_KEY"))
                self.search_sources.append("tavily")
                print("✅ Tavily search source enabled")
            except ImportError:
                print("⚠️ Tavily library not installed")

        # Check SerpApi availability
        if os.getenv("SERPAPI_API_KEY"):
            try:
                import serpapi
                self.search_sources.append("serpapi")
                print("✅ SerpApi search source enabled")
            except ImportError:
                print("⚠️ SerpApi library not installed")

        if self.search_sources:
            print(f"🔧 Available search sources: {', '.join(self.search_sources)}")
        else:
            print("⚠️ No available search sources, please configure API keys")

    def search(self, query: str) -> str:
        """Execute intelligent search"""
        if not query.strip():
            return "❌ Error: Search query cannot be empty"

        # Check if there are available search sources
        if not self.search_sources:
            return """❌ No available search sources, please configure one of the following API keys:

1. Tavily API: Set environment variable TAVILY_API_KEY
   Get it at: https://tavily.com/

2. SerpAPI: Set environment variable SERPAPI_API_KEY
   Get it at: https://serpapi.com/

Restart the program after configuration."""

        print(f"🔍 Starting intelligent search: {query}")

        # Try multiple search sources, return best result
        for source in self.search_sources:
            try:
                if source == "tavily":
                    result = self._search_with_tavily(query)
                    if result and "not found" not in result.lower():
                        return f"📊 Tavily AI search results:\n\n{result}"

                elif source == "serpapi":
                    result = self._search_with_serpapi(query)
                    if result and "not found" not in result.lower():
                        return f"🌐 SerpApi Google search results:\n\n{result}"

            except Exception as e:
                print(f"⚠️ {source} search failed: {e}")
                continue

        return "❌ All search sources failed, please check network connection and API key configuration"

    def _search_with_tavily(self, query: str) -> str:
        """Search using Tavily"""
        response = self.tavily_client.search(query=query, max_results=3)

        if response.get('answer'):
            result = f"💡 AI direct answer: {response['answer']}\n\n"
        else:
            result = ""

        result += "🔗 Related results:\n"
        for i, item in enumerate(response.get('results', [])[:3], 1):
            result += f"[{i}] {item.get('title', '')}\n"
            result += f"    {item.get('content', '')[:150]}...\n\n"

        return result

    def _search_with_serpapi(self, query: str) -> str:
        """Search using SerpApi"""
        import serpapi

        search = serpapi.GoogleSearch({
            "q": query,
            "api_key": os.getenv("SERPAPI_API_KEY"),
            "num": 3
        })

        results = search.get_dict()

        result = "🔗 Google search results:\n"
        if "organic_results" in results:
            for i, res in enumerate(results["organic_results"][:3], 1):
                result += f"[{i}] {res.get('title', '')}\n"
                result += f"    {res.get('snippet', '')}\n\n"

        return result

def create_advanced_search_registry():
    """Create registry containing advanced search tool"""
    registry = ToolRegistry()

    # Create search tool instance
    search_tool = MyAdvancedSearchTool()

    # Register search tool's method as function
    registry.register_function(
        name="advanced_search",
        description="Advanced search tool, integrates Tavily and SerpAPI multiple search sources, provides more comprehensive search results",
        func=search_tool.search
    )

    return registry
```

다음으로 우리가 직접 작성한 도구를 테스트할 수 있습니다. `test_advanced_search.py` 생성:

```python
# test_advanced_search.py
from dotenv import load_dotenv
from my_advanced_search import create_advanced_search_registry, MyAdvancedSearchTool

# Load environment variables
load_dotenv()

def test_advanced_search():
    """Test advanced search tool"""

    # Create registry containing advanced search tool
    registry = create_advanced_search_registry()

    print("🔍 Testing Advanced Search Tool\n")

    # Test queries
    test_queries = [
        "History of Python programming language",
        "Latest developments in artificial intelligence",
        "2024 technology trends"
    ]

    for i, query in enumerate(test_queries, 1):
        print(f"Test {i}: {query}")
        result = registry.execute_tool("advanced_search", query)
        print(f"Result: {result}\n")
        print("-" * 60 + "\n")

def test_api_configuration():
    """Test API configuration check"""
    print("🔧 Testing API Configuration Check:")

    # Directly create search tool instance
    search_tool = MyAdvancedSearchTool()

    # If API is not configured, configuration prompt will be displayed
    result = search_tool.search("machine learning algorithms")
    print(f"Search result: {result}")

def test_with_agent():
    """Test integration with Agent"""
    print("\n🤖 Integration Test with Agent:")
    print("Advanced search tool is ready and can be integrated with Agent")

    # Display tool description
    registry = create_advanced_search_registry()
    tools_desc = registry.get_tools_description()
    print(f"Tool description:\n{tools_desc}")

if __name__ == "__main__":
    test_advanced_search()
    test_api_configuration()
    test_with_agent()
```

이 고급 검색 도구 설계 실습을 통해 클래스를 사용하여 복잡한 도구 시스템을 구축하는 방법을 배웠습니다. 함수 접근 방식에 비해 클래스 접근 방식은 상태를 유지해야 하는 도구(예: API 클라이언트, 구성 정보)에 더 적합합니다.

### 7.5.4 도구 시스템의 고급 기능

기본 도구 개발과 다중 소스 통합을 마스터한 후 도구 시스템의 고급 기능을 살펴보겠습니다. 이러한 기능을 통해 도구 시스템은 복잡한 프로덕션 환경에서 안정적으로 실행되고 에이전트에 더욱 강력한 기능을 제공할 수 있습니다.

(1) 툴체인 호출 메커니즘

실제 응용 프로그램에서 에이전트는 복잡한 작업을 완료하기 위해 여러 도구를 결합해야 하는 경우가 많습니다. 6장에서 언급한 그래프 개념을 빌려 이 시나리오를 지원하는 도구 체인 관리자를 설계할 수 있습니다.

```python
# tool_chain_manager.py
from typing import List, Dict, Any, Optional
from hello_agents import ToolRegistry

class ToolChain:
    """Tool chain - supports sequential execution of multiple tools"""

    def __init__(self, name: str, description: str):
        self.name = name
        self.description = description
        self.steps: List[Dict[str, Any]] = []

    def add_step(self, tool_name: str, input_template: str, output_key: str = None):
        """
        Add tool execution step

        Args:
            tool_name: Tool name
            input_template: Input template, supports variable substitution
            output_key: Key name for output result, used for reference in subsequent steps
        """
        self.steps.append({
            "tool_name": tool_name,
            "input_template": input_template,
            "output_key": output_key or f"step_{len(self.steps)}_result"
        })

    def execute(self, registry: ToolRegistry, initial_input: str, context: Dict[str, Any] = None) -> str:
        """Execute tool chain"""
        context = context or {}
        context["input"] = initial_input

        print(f"🔗 Starting tool chain execution: {self.name}")

        for i, step in enumerate(self.steps, 1):
            tool_name = step["tool_name"]
            input_template = step["input_template"]
            output_key = step["output_key"]

            # Replace variables in template
            try:
                tool_input = input_template.format(**context)
            except KeyError as e:
                return f"❌ Tool chain execution failed: Template variable {e} not found"

            print(f"  Step {i}: Using {tool_name} to process '{tool_input[:50]}...'")

            # Execute tool
            result = registry.execute_tool(tool_name, tool_input)
            context[output_key] = result

            print(f"  ✅ Step {i} completed, result length: {len(result)} characters")

        # Return result of last step
        final_result = context[self.steps[-1]["output_key"]]
        print(f"🎉 Tool chain '{self.name}' execution completed")
        return final_result

class ToolChainManager:
    """Tool chain manager"""

    def __init__(self, registry: ToolRegistry):
        self.registry = registry
        self.chains: Dict[str, ToolChain] = {}

    def register_chain(self, chain: ToolChain):
        """Register tool chain"""
        self.chains[chain.name] = chain
        print(f"✅ Tool chain '{chain.name}' registered")

    def execute_chain(self, chain_name: str, input_data: str, context: Dict[str, Any] = None) -> str:
        """Execute specified tool chain"""
        if chain_name not in self.chains:
            return f"❌ Tool chain '{chain_name}' does not exist"

        chain = self.chains[chain_name]
        return chain.execute(self.registry, input_data, context)

    def list_chains(self) -> List[str]:
        """List all tool chains"""
        return list(self.chains.keys())

# Usage example
def create_research_chain() -> ToolChain:
    """Create a research tool chain: search -> calculate -> summarize"""
    chain = ToolChain(
        name="research_and_calculate",
        description="Search for information and perform related calculations"
    )

    # Step 1: Search for information
    chain.add_step(
        tool_name="search",
        input_template="{input}",
        output_key="search_result"
    )

    # Step 2: Perform calculations based on search results (if needed)
    chain.add_step(
        tool_name="my_calculator",
        input_template="Calculate relevant values based on the following information: {search_result}",
        output_key="calculation_result"
    )

    return chain
```

(2) 비동기 도구 실행 지원

시간이 많이 걸리는 도구 작업의 경우 비동기 실행 지원을 제공할 수 있습니다.

```python
# async_tool_executor.py
import asyncio
import concurrent.futures
from typing import Dict, Any, List, Callable
from hello_agents import ToolRegistry

class AsyncToolExecutor:
    """Asynchronous tool executor"""

    def __init__(self, registry: ToolRegistry, max_workers: int = 4):
        self.registry = registry
        self.executor = concurrent.futures.ThreadPoolExecutor(max_workers=max_workers)

    async def execute_tool_async(self, tool_name: str, input_data: str) -> str:
        """Asynchronously execute a single tool"""
        loop = asyncio.get_event_loop()

        def _execute():
            return self.registry.execute_tool(tool_name, input_data)

        result = await loop.run_in_executor(self.executor, _execute)
        return result

    async def execute_tools_parallel(self, tasks: List[Dict[str, str]]) -> List[str]:
        """Execute multiple tools in parallel"""
        print(f"🚀 Starting parallel execution of {len(tasks)} tool tasks")

        # Create async tasks
        async_tasks = []
        for task in tasks:
            tool_name = task["tool_name"]
            input_data = task["input_data"]
            async_task = self.execute_tool_async(tool_name, input_data)
            async_tasks.append(async_task)

        # Wait for all tasks to complete
        results = await asyncio.gather(*async_tasks)

        print(f"✅ All tool tasks completed")
        return results

    def __del__(self):
        """Clean up resources"""
        if hasattr(self, 'executor'):
            self.executor.shutdown(wait=True)

# Usage example
async def test_parallel_execution():
    """Test parallel tool execution"""
    from hello_agents import ToolRegistry

    registry = ToolRegistry()
    # Assume search and calculator tools are already registered

    executor = AsyncToolExecutor(registry)

    # Define parallel tasks
    tasks = [
        {"tool_name": "search", "input_data": "Python programming"},
        {"tool_name": "search", "input_data": "machine learning"},
        {"tool_name": "my_calculator", "input_data": "2 + 2"},
        {"tool_name": "my_calculator", "input_data": "sqrt(16)"},
    ]

    # Execute in parallel
    results = await executor.execute_tools_parallel(tasks)

    for i, result in enumerate(results):
        print(f"Task {i+1} result: {result[:100]}...")
```

위의 설계 및 구현 경험을 바탕으로 도구 시스템 개발의 핵심 개념을 요약할 수 있습니다. 설계 수준에서 각 도구는 단일 책임 원칙을 따라야 하며 인터페이스 균일성을 유지하면서 특정 기능에 중점을 두고 포괄적인 예외 처리 및 보안 우선 입력 유효성 검사를 기본 요구 사항으로 처리해야 합니다. 성능 최적화 측면에서는 비동기 실행을 사용하여 동시 처리 기능을 향상시키는 동시에 외부 연결 및 시스템 리소스를 합리적으로 관리합니다.



## 7.6 장 요약

공식적으로 요약하기 전에 좋은 소식을 모든 사람과 공유하고 싶습니다. 이 장에서 구현된 모든 방법과 기능에 대해 완전한 테스트 사례가 GitHub 저장소에 제공됩니다. [이 링크](https://github.com/jjyaoao/HelloAgents/blob/main/examples/chapter07_basic_setup.py))를 방문하여 테스트 코드를 보고 실행할 수 있습니다. 이 파일에는 4가지 에이전트 패러다임의 데모, 도구 시스템의 통합 테스트, 고급 기능의 사용 예 및 대화형 에이전트 경험이 포함되어 있습니다. 구현이 올바른지 확인하고 싶거나 프레임워크의 실제 사용법을 깊이 이해하고 싶다면 이 테스트 사례가 귀중한 참고 자료가 될 것입니다.

이 장을 되돌아보면 우리는 어려운 작업을 완료했습니다. 단계별로 기본 에이전트 프레임워크인 HelloAgents를 구축했습니다. 이 프로세스는 "계층화된 분리, 단일 책임 및 통합 인터페이스"라는 핵심 원칙을 일관되게 따랐습니다.

프레임워크의 특정 구현에서 우리는 네 가지 기본 에이전트 패러다임을 다시 구현했습니다. SimpleAgent의 기본 대화 모드부터 ReActAgent의 추론과 액션의 조합까지 ReflectionAgent의 자기반성 및 반복적 최적화부터 PlanAndSolveAgent의 분해 계획 및 단계별 실행까지. 에이전트 기능 확장의 핵심인 도구 시스템은 완전한 엔지니어링 관행이었습니다.

더 중요한 것은 7장의 구성이 종점이 아니라 후속 장의 심층 학습에 필요한 기술적 기반을 제공한다는 것입니다. 우리는 초기 디자인에서 후속 콘텐츠의 확장성을 충분히 고려하여 고급 기능 구현에 필요한 인터페이스와 확장 지점을 확보했습니다. 우리가 함께 구축한 통합 LLM 인터페이스, 표준화된 메시지 시스템 및 도구 등록 메커니즘은 완전한 기술 기반을 구성합니다. 이를 통해 우리는 후속 장에서 보다 고급 주제를 보다 차분하게 학습할 수 있습니다. 8장의 메모리 및 RAG 시스템은 이를 기반으로 에이전트의 기능 경계를 확장합니다. 9장의 컨텍스트 엔지니어링에서는 우리가 확립한 메시지 처리 메커니즘을 자세히 살펴볼 것입니다. 10장의 에이전트 프로토콜에는 새로운 도구 확장이 필요합니다.

다음으로 RAG 시스템과 메모리 메커니즘을 프레임워크에 추가하는 방법을 함께 살펴보겠습니다. 8장을 기대해주세요!


## 연습

1. 이 장에서는 `HelloAgents` 프레임워크를 구축하고 "자체 Agent 프레임워크를 구축해야 하는 이유"를 설명했습니다. 분석해 주십시오:

- 섹션 7.1.1에서는 현재 주류 프레임워크의 네 가지 주요 제한 사항을 언급했습니다. [6장 연습](../chapter6/第六章%20框架开发实践.md#习题)이나 실제 프로젝트의 프레임워크를 사용한 실제 경험과 결합하여 이러한 문제가 개발 효율성에 어떤 영향을 미치는지 설명하세요.
   - `HelloAgents`은 `Memory`, `RAG`, `MCP` 등의 모듈을 도구로 추상화하여 "모든 것이 도구이다"라는 디자인 철학을 제안합니다. 이 디자인의 장점은 무엇입니까? 제한사항이 있나요? 예시를 제공해 주세요.
   - 4장의 처음부터 구현한 에이전트 코드와 이번 장의 프레임워크 구현을 비교하면 프레임워크가 구체적으로 어떤 개선을 가져오는가? 프레임워크를 디자인한다면 어떤 디자인 원칙을 우선시하시겠습니까?

2. 섹션 7.2에서는 여러 모델 제공자와 로컬 모델 호출을 지원하기 위해 `HelloAgentsLLM`을 확장했습니다.

> <strong>힌트</strong>: 이것은 실용적인 연습이므로 직접 조작하는 것이 좋습니다.

- 섹션 7.2.1의 예를 참조하여 `HelloAgentsLLM`(예: `Gemini`, `Anthropic`, `Kim`)에 새 모델 공급자에 대한 지원을 추가해 보세요. 상속을 통해 구현하고 해당 공급자의 환경 변수를 자동으로 감지할 수 있습니다.
   - 7.2.3절에서는 자동 탐지 메커니즘의 세 가지 우선 순위를 소개했습니다. 분석해 보십시오. `OPENAI_API_KEY` 및 `LLM_BASE_URL="http://localhost:11434/v1"`이 모두 설정된 경우 프레임워크는 최종적으로 어떤 공급자를 선택합니까? 이 우선순위 설계가 합리적인가요?
   - 이 장에서 소개된 `VLLM` 및 `Ollama` 외에도 `SGLang`와 같은 다른 로컬 모델 배포 솔루션이 있습니다. `SGLang`의 기본 정보와 특징을 먼저 검색하고 이해한 후, 사용 용이성, 리소스 소모, 추론 속도, 추론 정확도 측면에서 `VLLM`, `SGLang`, `Ollama`를 비교해 보세요.

3. 섹션 7.3에서는 `Message` 클래스, `Config` 클래스, `Agent` 기본 클래스를 구현했습니다. 분석해 주십시오:

- `Message` 클래스는 데이터 검증을 위해 `Pydantic`의 `BaseModel`를 사용합니다. 실제 응용 분야에서 이 디자인의 장점은 무엇입니까?
   - `Agent` 기본 클래스는 `run` 및 `_execute`라는 두 가지 메서드를 정의합니다. 여기서 `run`은 공용 인터페이스이고 `_execute`는 추상 메서드입니다. 이 디자인 패턴을 뭐라고 부르나요? 그 이점은 무엇입니까?
   - `Config` 클래스에서는 싱글턴 패턴을 사용했습니다. 싱글턴 패턴이 무엇인지, 구성 관리에서 싱글턴 패턴을 사용해야 하는 이유, 싱글턴 패턴을 사용하지 않을 경우 어떤 문제가 발생하는지 설명해주세요.

4. 섹션 7.4에서는 프레임워크 방식으로 4개의 `Agent` 패러다임을 구현했습니다.

> <strong>힌트</strong>: 이것은 실용적인 연습이므로 직접 조작하는 것이 좋습니다.

- 4장의 처음부터 구현된 `ReActAgent`과 이 장의 프레임워크 기반 `ReActAgent`을 비교하여 3가지 구체적인 개선 사항을 나열하고 이러한 개선 사항이 코드 유지 관리성과 확장성을 어떻게 향상시키는지 설명합니다.
   - `ReflectionAgent`는 "실행-반영-최적화" 루프를 구현합니다. "품질 채점" 메커니즘을 추가하여 이 구현을 확장하십시오. 각 반영 후 `LLM`가 현재 버전의 출력에 점수를 매기고 점수가 임계값 미만인 경우에만 최적화를 계속합니다. 그렇지 않으면 조기 종료됩니다.
   - `Agent` 기본 클래스에서 상속되고 각 단계에서 가능한 여러 사고 경로를 생성할 수 있는 `Tree-of-Thought Agent`이라는 새로운 `Agent` 패러다임을 설계하고 구현한 다음 계속하려면 최적의 경로를 선택하세요.

5. 섹션 7.5에서는 도구 시스템을 구축했습니다. 다음 질문을 고려해 보십시오.

- `BaseTool` 클래스는 모든 도구가 구현해야 하는 `execute` 추상 메서드를 정의합니다. 모든 도구가 통합 인터페이스를 구현해야 하는 이유를 설명해주세요. 도구가 여러 값(예: 제목, 요약, 링크를 반환하는 검색 도구)을 반환해야 하는 경우 어떻게 설계해야 합니까?
   - 섹션 7.5.3에서는 도구 체인(`ToolChain`)을 구현했습니다. 최소 3개 이상의 도구를 연결해야 하는 실제 응용 시나리오를 설계하고 도구 체인의 실행 흐름도를 그려주세요.
   - 비동기 도구 실행기(`AsyncToolExecutor`)는 스레드 풀을 사용하여 도구를 병렬로 실행합니다. 분석해 보십시오. 어떤 상황에서 병렬 도구 실행이 성능 향상을 가져올 수 있습니까?

6. 프레임워크 확장성은 설계 시 중요한 고려 사항 중 하나입니다. 이제 몇 가지 흥미로운 새 기능과 특성을 구현하기 위해 `HelloAgents` 프레임워크를 확장해야 합니다.

- 먼저 `HelloAgents`에 "스트리밍 출력" 기능을 추가하여 `Agent`이 응답 생성 시 실시간으로 중간 결과를 반환할 수 있도록 합니다(`ChatGPT` 사용자 인터페이스의 입력 효과와 유사). 이 기능에 대한 구현 계획을 설계하고 어떤 클래스와 메서드를 수정해야 하는지 설명해주세요.
   - 그런 다음 대화 기록을 자동으로 관리하고 대화 분기 및 역추적을 지원할 수 있는 "다단계 대화 관리" 기능을 프레임워크에 추가합니다. 이것을 어떻게 디자인하시겠습니까? 어떤 새로운 수업이 필요합니까? 기존 `Message` 시스템과 어떻게 통합하나요?
   - 마지막으로, 타사 개발자가 프레임워크의 핵심 코드를 수정하지 않고도 플러그인 (such as adding new ⟪P1⟫ types, new tool types, etc.)을 통해 프레임워크 기능을 확장할 수 있도록 하는 `HelloAgents`용 "플러그인 시스템"을 설계해 주세요. 플러그인 시스템의 아키텍처 다이어그램을 그리고 주요 인터페이스를 설명합니다.

