# 6장 프레임워크 개발 실습

4장에서는 ReAct, Plan-and-Solve, Reflection 등 여러 에이전트의 핵심 워크플로를 네이티브 코드를 작성하여 구현했습니다. 이 과정을 통해 우리는 에이전트의 내부 실행 로직을 이해할 수 있었습니다. 이어 5장에서는 '사용자' 관점으로 전환해 로우코드 플랫폼이 가져다주는 편리성과 효율성을 경험했다.

이 장의 목표는 신뢰할 수 있는 에이전트 애플리케이션을 효율적이고 표준적으로 구축하기 위해 업계의 일부 주류 **에이전트 프레임워크**를 사용하는 방법을 탐색하는 것입니다. 먼저 현재 시장에 나와 있는 주류 에이전트 프레임워크를 개괄적으로 살펴본 후, 여러 대표적인 프레임워크에 대한 완전한 실제 사례를 통해 프레임워크 중심 개발 모델을 경험해 보겠습니다.

## 6.1 수동 구현에서 프레임워크 개발까지

일회성 스크립트 작성에서 성숙한 프레임워크 사용으로 전환하는 것은 소프트웨어 엔지니어링 분야에서 중요한 정신적 도약입니다. 4장에서 우리가 작성한 코드는 주로 교육과 이해를 위한 것이었습니다. 특정 작업을 잘 완료할 수 있지만 이를 사용하여 복잡한 논리를 가진 여러 가지 유형의 에이전트를 구축하려는 경우 곧 병목 현상이 발생하게 됩니다.

프레임워크의 본질은 검증된 "사양" 세트를 제공하는 것입니다. 이는 모든 에이전트에 공통된 모든 반복 작업을 추상화하고 캡슐화하므로((such as main loops, state management, tool invocation, logging, etc.)) 일반적인 기본 구현보다는 새로운 에이전트를 구축할 때 고유한 비즈니스 로직에 집중할 수 있습니다.

### 6.1.1 에이전트 프레임워크가 필요한 이유

실제 작업을 시작하기 전에 먼저 프레임워크를 사용해야 하는 이유를 명확히 해야 합니다. 독립적인 에이전트 스크립트를 직접 작성하는 것과 비교할 때 프레임워크 사용의 가치는 주로 다음 측면에 반영됩니다.

1. **코드 재사용 및 개발 효율성 향상**: 이것이 가장 직접적인 가치입니다. 좋은 프레임워크는 에이전트 작업의 핵심 루프(에이전트 루프)를 캡슐화하는 일반 `Agent` 기본 클래스 또는 실행기를 제공합니다. ReAct이든 Plan-and-Solve이든 프레임워크에서 제공하는 표준 구성 요소를 기반으로 신속하게 구축할 수 있으므로 반복 작업을 피할 수 있습니다.
2. **핵심 구성 요소의 분리 및 확장성 달성**: 강력한 에이전트 시스템은 느슨하게 연결된 여러 모듈로 구성되어야 합니다. 프레임워크의 설계로 인해 우리는 서로 다른 관심사를 분리해야 합니다.
   - **모델 레이어**: 대규모 언어 모델과의 상호 작용을 담당하며 다양한 모델(OpenAI, Anthropic, 로컬 모델)을 쉽게 대체할 수 있습니다.
   - **도구 레이어**: 표준화된 도구 정의, 등록 및 실행 인터페이스를 제공합니다. 새로운 도구를 추가해도 다른 코드에는 영향을 미치지 않습니다.
   - **메모리 계층**: 단기 및 장기 메모리를 처리하고 필요에 따라 다양한 메모리 전략(예: 슬라이딩 창, 요약 메모리)을 전환할 수 있습니다. 이러한 모듈형 설계는 전체 시스템의 확장성을 높여 모든 구성요소를 간단하게 교체하거나 업그레이드할 수 있게 해줍니다.
3. **복잡한 상태 관리 표준화**: `ReflectionAgent`에서 구현한 `Memory` 클래스는 단순한 시작일 뿐입니다. 실제 장기 실행 에이전트 애플리케이션에서 상태 관리는 컨텍스트 창 제한, 기록 정보 지속성, 다중 전환 대화 상태 추적 및 기타 문제를 처리해야 하는 큰 과제입니다. 프레임워크는 강력하고 일반적인 상태 관리 메커니즘을 제공할 수 있으므로 개발자는 매번 이러한 복잡한 문제를 처리할 필요가 없습니다.
4. **관찰 가능성 및 디버깅 프로세스 단순화**: 에이전트 동작이 복잡해지면 의사 결정 프로세스를 이해하는 것이 중요합니다. 잘 설계된 프레임워크에는 강력한 관찰 기능이 내장되어 있을 수 있습니다. 예를 들어 이벤트 콜백 메커니즘(콜백)을 도입하면 에이전트 수명 주기의 주요 노드(예: `on_llm_start`, `on_tool_end`, `on_agent_finish`)에서 로깅 또는 데이터 보고를 자동으로 트리거할 수 있으므로 에이전트의 전체 실행 궤적을 쉽게 추적하고 디버그할 수 있습니다. 이는 코드에 `print` 문을 수동으로 추가하는 것보다 훨씬 효율적이고 체계적입니다.

따라서 수동 구현에서 프레임워크 개발로 이동하는 것은 코드 구성의 변화일 뿐만 아니라 복잡하고 안정적이며 유지 관리가 가능한 에이전트 애플리케이션을 구축하는 데 필요한 경로이기도 합니다.

### 6.1.2 주류 프레임워크의 선택 및 비교

에이전트 프레임워크 생태계는 전례 없는 속도로 발전하고 있습니다. LangChain과 LlamaIndex가 1세대 일반 LLM 애플리케이션 프레임워크의 패러다임을 정의했다면, 차세대 프레임워크는 특히 **다중 에이전트 협업** 및 **복잡한 작업 흐름 제어**와 같은 특정 도메인의 심층적인 문제를 해결하는 데 더 중점을 둡니다.

이 장의 후속 실제 작업에서는 이러한 최첨단 분야에서 매우 대표적인 네 가지 프레임워크인 AutoGen, AgentScope, CAMEL 및 LangGraph에 중점을 둘 것입니다. 표 6.1에 표시된 것처럼 복잡한 에이전트 시스템을 구현하기 위한 다양한 기술 경로를 나타내는 설계 철학이 다릅니다.

<div align="center">
<p>표 6.1 4가지 에이전트 프레임워크 비교</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/6-figures/01.png" alt="" width="90%"/>
</div>


- **AutoGen**: AutoGen의 핵심 아이디어는 대화<sup>[1]</sup>를 통해 협업을 달성하는 것입니다. 이는 다중 에이전트 시스템을 여러 "대화 가능한" 에이전트로 구성된 그룹 채팅으로 추상화합니다. 개발자는 다양한 역할(예: `Coder`, `ProductManager`, `Tester`)을 정의하고 역할 간의 상호 작용 규칙을 설정할 수 있습니다(예를 들어 `Coder`가 코드 작성을 마친 후 `Tester`가 자동으로 이어받음). 작업 해결 프로세스는 이러한 에이전트가 최종 목표가 달성될 때까지 자동화된 메시지 전달을 통해 그룹 채팅에서 지속적으로 대화하고, 협업하고, 반복하는 프로세스입니다.
- **AgentScope**: AgentScope는 다중 에이전트 애플리케이션<sup>[2]</sup>을 위해 특별히 설계된 완전한 기능을 갖춘 개발 플랫폼입니다. 핵심 기능은 **사용 편의성**과 **엔지니어링**입니다. 개발자가 쉽게 에이전트를 정의하고, 통신 네트워크를 구축하고, 전체 애플리케이션 수명주기를 관리할 수 있는 매우 친숙한 프로그래밍 인터페이스를 제공합니다. 내장된 **메시지 전달 메커니즘**과 분산 배포 지원 덕분에 복잡한 대규모 다중 에이전트 시스템을 구축하고 운영하는 데 매우 적합합니다.
- **CAMEL**: CAMEL은 **롤플레잉**<sup>[3]</sup>이라는 새로운 협업 방식을 제공합니다. 핵심 개념은 두 에이전트(예: `AI Researcher` 및 `Python Programmer`)에 대해 각각의 역할과 공통 작업 목표만 설정하면 되며, 에이전트는 "**Inception Prompting**"의 안내에 따라 여러 라운드의 대화를 자율적으로 수행하여 서로 영감을 주고 협력하여 작업을 함께 완료할 수 있다는 것입니다. 이는 다중 에이전트 대화 프로세스 설계의 복잡성을 크게 줄여줍니다.
- **LangGraph**: LangChain 생태계의 확장인 LangGraph는 에이전트의 실행 프로세스를 **Graph**<sup>[4]</sup>로 모델링하여 다른 접근 방식을 취합니다. 전통적인 체인 구조에서는 정보가 한 방향으로만 흐를 수 있습니다. LangGraph는 각 작업(예: LLM 호출, 도구 실행)을 그래프의 **노드**로 정의하고 **에지**를 사용하여 노드 간의 점프 논리를 정의합니다. 이 디자인은 자연스럽게 **Cycles**을 지원하므로 반복, 수정, 자기 성찰이 포함된 Reflection과 같은 복잡한 워크플로를 구현하는 것이 매우 간단하고 직관적입니다.

다음 섹션에서는 이 네 가지 프레임워크 각각에 대한 완전한 실제 사례를 통해 프레임워크 중심 개발 모델을 심층적으로 경험해 보겠습니다. **참고** 시연된 모든 프로젝트 소스 파일은 `code` 폴더에 배치되며, 본문에서는 원칙적인 부분만 설명합니다.

## 6.2 프레임워크 1: AutoGen

앞서 언급했듯이 AutoGen의 디자인 철학은 "대화를 통한 협업 추진"에 뿌리를 두고 있습니다. 복잡한 작업 해결 프로세스를 다양한 역할을 가진 상담원 간의 일련의 자동화된 대화에 교묘하게 매핑합니다. 이 핵심 개념을 기반으로 AutoGen 프레임워크는 계속해서 발전하고 있습니다. 버전 `0.7.4`을 예로 사용하겠습니다. 버전은 최신 버전이고 클래스 상속 설계에서 보다 유연한 구성 아키텍처로 전환하는 중요한 아키텍처 리팩토링을 나타내기 때문입니다. 이 프레임워크를 깊이 이해하고 적용하려면 먼저 가장 핵심적인 구성 요소와 기본 대화 상호 작용 메커니즘을 설명해야 합니다.

### 6.2.1 AutoGen의 핵심 메커니즘

버전 `0.7.4`의 출시는 AutoGen 개발의 중요한 이정표이며 프레임워크의 기본 디자인에 근본적인 혁신을 가져왔습니다. 이번 업데이트는 단순한 기능 추가가 아니라 프레임워크의 모듈성, 동시성 성능 및 개발자 경험 개선을 목표로 전체 아키텍처를 다시 생각하는 것입니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/6-figures/02.png" alt="" width="90%"/>
<p>그림 6.1 AutoGen 아키텍처 다이어그램</p>
</div>

(1) 프레임워크 구조의 진화

그림 6.1에서 볼 수 있듯이 새로운 아키텍처의 가장 중요한 변화는 명확한 계층화와 비동기 우선 설계 철학의 도입입니다.

- **계층형 디자인:** 프레임워크는 두 개의 핵심 모듈로 분할됩니다.
  - `autogen-core`: 프레임워크의 기본 기반으로 언어 모델과의 상호 작용 및 메시지 전달과 같은 핵심 기능을 캡슐화합니다. 그 존재는 프레임워크의 안정성과 향후 확장성을 보장합니다.
  - `autogen-agentchat`: `core`을 기반으로 구축되어 대화형 에이전트 애플리케이션 개발을 위한 상위 수준 인터페이스를 제공하여 다중 에이전트 애플리케이션의 개발 프로세스를 단순화합니다. 이러한 계층화 전략은 각 구성 요소의 책임을 명확하게 하고 시스템 결합을 줄입니다.
- **비동기 우선:** 새 아키텍처는 비동기 프로그래밍으로 완전히 전환됩니다(`async/await`). 다중 에이전트 공동 작업 시나리오에서 네트워크 요청(예: LLM API 호출)은 시간이 많이 걸리는 작업입니다. 비동기 모드를 사용하면 시스템이 한 에이전트의 응답을 기다리는 동안 다른 작업을 처리할 수 있으므로 스레드 차단을 방지하고 동시 처리 기능과 시스템 리소스 활용 효율성을 크게 향상시킬 수 있습니다.

(2) 핵심 에이전트 구성 요소

에이전트는 작업을 실행하는 기본 단위입니다. 버전 `0.7.4`에서는 에이전트 설계가 더욱 집중적이고 모듈화되었습니다.

- **AssistantAgent(보조 에이전트):** 대규모 언어 모델(LLM)을 핵심으로 캡슐화하는 주요 작업 해결사입니다. 계획 제안, 기사 작성, 코드 작성 등 대화 기록을 기반으로 논리적이고 지식이 풍부한 답변을 생성하는 것이 임무입니다. 다양한 시스템 메시지(시스템 메시지)를 통해 다양한 "전문가" 역할을 할당할 수 있습니다.
- **UserProxyAgent(사용자 프록시 에이전트):** 이는 AutoGen의 기능적으로 고유한 구성 요소입니다. 이는 두 가지 역할을 수행합니다. 인간 사용자의 "대변인" 역할을 하며 작업 시작과 의도 전달을 담당합니다. 코드를 실행하거나 도구를 호출하고 결과를 다른 에이전트에 다시 공급하도록 구성할 수 있는 신뢰할 수 있는 "실행자"입니다. 이 디자인은 '생각'(`AssistantAgent`으로 완성)과 '행동'을 명확하게 구분합니다.

(3) GroupChatManager에서 팀으로

작업에 여러 에이전트가 협력해야 하는 경우 대화 프로세스를 조정하는 메커니즘이 필요합니다. 이전 버전에서는 `GroupChatManager`가 이 책임을 맡았습니다. 새로운 아키텍처에서는 `RoundRobinGroupChat`와 같은 보다 유연한 `Team` 또는 그룹 채팅 개념이 도입되었습니다.

- **라운드 로빈 그룹 채팅(RoundRobinGroupChat):** 이는 명확하고 순차적인 대화 조정 메커니즘입니다. 미리 정의된 순서에 따라 참가 상담원이 차례로 발언하게 됩니다. 이 모드는 일반적인 소프트웨어 개발 프로세스와 같이 고정된 프로세스가 있는 작업에 매우 적합합니다. 제품 관리자가 먼저 요구 사항을 제안한 다음 엔지니어가 코드를 작성하고 마지막으로 코드 검토자가 확인합니다.
- **작업 흐름:**
  1. 먼저 `RoundRobinGroupChat` 인스턴스를 생성하고 협업에 참여하는 모든 에이전트 (such as product managers, engineers, etc.)를 여기에 추가합니다.
  2. 작업이 시작되면 그룹 채팅은 미리 설정된 순서에 따라 해당 에이전트를 차례로 활성화합니다.
  3. 선택된 상담원은 현재 대화 내용을 바탕으로 응답합니다.
  4. 그룹 채팅은 대화 내역에 새로운 답변을 추가하고 다음 상담원을 활성화합니다.
  5. 이 프로세스는 최대 대화 라운드 수에 도달하거나 미리 설정된 종료 조건이 충족될 때까지 계속됩니다.

이러한 방식으로 AutoGen은 관리하기 쉬운 명확한 프로세스를 통해 복잡한 협업 관계를 자동화된 "원탁 회의"로 단순화합니다. 개발자는 각 팀원의 역할과 발언 순서만 정의하면 되며, 나머지 협업 프로세스는 그룹 채팅 메커니즘을 통해 자율적으로 진행될 수 있습니다.

다음 섹션에서는 새로운 아키텍처에서 다양한 역할을 가진 에이전트를 정의하고 `RoundRobinGroupChat`가 조정하는 그룹 채팅에서 에이전트를 구성하여 시뮬레이션된 소프트웨어 개발팀의 인스턴스를 구축하여 실제 프로그래밍 작업을 공동으로 완료하는 방법을 직접 경험해 보겠습니다.

### 6.2.2 소프트웨어 개발팀

AutoGen의 핵심 구성요소와 대화 메커니즘을 이해한 후, 이 섹션에서는 완전한 실제 사례를 통해 이러한 새로운 기능을 적용하는 방법을 구체적으로 보여줍니다. 우리는 실제 소프트웨어 개발 작업을 완료하기 위해 협력할 다양한 전문 기술을 갖춘 여러 에이전트로 구성된 시뮬레이션 소프트웨어 개발 팀을 구축할 것입니다.

(1) 사업목적

우리의 목표는 **실시간으로 비트코인의 현재 가격을 표시**하는 명확한 기능을 갖춘 웹 애플리케이션을 개발하는 것입니다. 이 작업은 규모가 작지만 요구 사항 분석, 기술 선택, 코딩 구현부터 코드 검토 및 최종 테스트까지 소프트웨어 개발의 일반적인 단계를 완전히 다룹니다. 이는 AutoGen의 자동화된 협업 프로세스를 테스트하기 위한 이상적인 시나리오입니다.

(2) 에이전트 팀의 역할

실제 소프트웨어 개발 프로세스를 시뮬레이션하기 위해 우리는 서로 다른 책임을 맡은 4개의 에이전트를 설계했습니다.

- **ProductManager(제품 관리자):** 사용자의 모호한 요구 사항을 명확하고 실행 가능한 개발 계획으로 변환하는 역할을 담당합니다.
- **엔지니어:** 개발 계획에 따라 특정 애플리케이션 코드 작성을 담당합니다.
- **CodeReviewer(코드 검토자):** 엔지니어가 제출한 코드를 검토하여 품질, 가독성 및 견고성을 보장하는 역할을 담당합니다.
- **UserProxy(사용자 프록시):** 최종 사용자를 나타내고 초기 작업을 시작하며 최종 전달된 코드를 실행하고 확인하는 역할을 담당합니다.

이 역할 분할은 복잡한 작업을 도메인 "전문가"가 처리하는 여러 하위 작업으로 나누는 다중 에이전트 시스템 설계의 핵심 단계입니다.

### 6.2.3 핵심 코드 구현

아래에서는 이 자동화된 팀의 핵심 코드를 단계별로 분석해 보겠습니다.

(1) 모델 클라이언트 구성

모든 LLM 기반 에이전트에는 언어 모델과 상호 작용하려면 모델 클라이언트가 필요합니다. AutoGen `0.7.4`은 OpenAI API 사양 (including OpenAI official service, Azure OpenAI, and local model services such as Ollama, etc.)과 호환되는 모든 모델 서비스와 편리하게 인터페이스할 수 있는 표준화된 `OpenAIChatCompletionClient`을 제공합니다.

독립된 기능을 통해 모델 클라이언트를 생성 및 구성하고, 환경 변수를 통해 API Key 및 서비스 주소를 관리합니다. 이는 코드 유연성과 보안을 향상시키는 좋은 엔지니어링 방법입니다.

```python
from autogen_ext.models.openai import OpenAIChatCompletionClient

def create_openai_model_client():
    """Create and configure OpenAI model client"""
    return OpenAIChatCompletionClient(
        model=os.getenv("LLM_MODEL_ID", "gpt-4o"),
        api_key=os.getenv("LLM_API_KEY"),
        base_url=os.getenv("LLM_BASE_URL", "https://api.openai.com/v1")
    )
```

(2) 대리인 역할의 정의

에이전트 정의의 핵심은 고품질의 시스템 메시지(System Message)를 작성하는 것입니다. 시스템 메시지는 상담원을 위한 "행동 지침" 및 "전문 지식 기반"을 설정하는 것과 같으며 상담원의 역할, 책임, 작업 흐름은 물론 다른 상담원과 상호 작용하는 방식까지 정확하게 지정합니다. 잘 설계된 시스템 메시지는 다중 에이전트 시스템이 효율적이고 정확하게 협업할 수 있도록 보장하는 핵심입니다. 우리 소프트웨어 개발 팀에서는 각 역할에 대한 정의를 캡슐화하기 위해 독립적인 기능을 만들었습니다.

**제품 관리자(ProductManager)**

제품 관리자는 전체 프로세스를 시작하는 일을 담당합니다. 시스템 메시지는 책임을 정의할 뿐만 아니라 출력 구조를 표준화하고 대화를 다음 단계(엔지니어)로 안내하는 명확한 지침을 포함합니다.

```python
def create_product_manager(model_client):
    """Create product manager agent"""
    system_message = """You are an experienced product manager specializing in requirement analysis and project planning for software products.

Your core responsibilities include:
1. **Requirement Analysis**: Deeply understand user needs, identify core functions and boundary conditions
2. **Technical Planning**: Develop clear technical implementation paths based on requirements
3. **Risk Assessment**: Identify potential technical risks and user experience issues
4. **Coordination and Communication**: Communicate effectively with engineers and other team members

When receiving a development task, please analyze it according to the following structure:
1. Requirement understanding and analysis
2. Functional module division
3. Technology selection recommendations
4. Implementation priority sorting
5. Acceptance criteria definition

Please respond concisely and clearly, and say "Please engineer start implementation" after completing the analysis."""

    return AssistantAgent(
        name="ProductManager",
        model_client=model_client,
        system_message=system_message,
    )
```

**엔지니어**

엔지니어의 시스템 메시지는 기술 구현에 중점을 둡니다. 엔지니어의 기술 전문 지식을 나열하고 작업을 받은 후 특정 작업 단계를 지정하며 코드 검토자에게 프로세스를 안내하는 지침도 포함됩니다.

```python
def create_engineer(model_client):
    """Create software engineer agent"""
    system_message = """You are a senior software engineer skilled in Python development and web application construction.

Your technical expertise includes:
1. **Python Programming**: Proficient in Python syntax and best practices
2. **Web Development**: Expert in frameworks such as Streamlit, Flask, Django
3. **API Integration**: Rich experience in third-party API integration
4. **Error Handling**: Focus on code robustness and exception handling

When receiving a development task, please:
1. Carefully analyze technical requirements
2. Choose appropriate technical solutions
3. Write complete code implementation
4. Add necessary comments and explanations
5. Consider boundary cases and exception handling

Please provide complete runnable code and say "Please code reviewer check" after completion."""

    return AssistantAgent(
        name="Engineer",
        model_client=model_client,
        system_message=system_message,
    )
```

**코드 검토자(CodeReviewer)**

코드 검토자의 정의는 코드 품질, 보안 및 표준화에 중점을 둡니다. 시스템 메시지에는 검토 초점과 프로세스가 자세히 설명되어 있어 코드 전달 전 품질 체크포인트를 보장합니다.

```python
def create_code_reviewer(model_client):
    """Create code reviewer agent"""
    system_message = """You are an experienced code review expert focusing on code quality and best practices.

Your review focus includes:
1. **Code Quality**: Check code readability, maintainability, and performance
2. **Security**: Identify potential security vulnerabilities and risk points
3. **Best Practices**: Ensure code follows industry standards and best practices
4. **Error Handling**: Verify the completeness and rationality of exception handling

Review process:
1. Carefully read and understand code logic
2. Check code standards and best practices
3. Identify potential issues and improvement points
4. Provide specific modification suggestions
5. Evaluate overall code quality

Please provide specific review comments and say "Code review completed, please user proxy test" after completion."""

    return AssistantAgent(
        name="CodeReviewer",
        model_client=model_client,
        system_message=system_message,
    )
```

**사용자 프록시(UserProxy)**

`UserProxyAgent`은 응답을 위해 LLM에 의존하지 않고 시스템에서 사용자의 프록시 역할을 하는 특수 에이전트입니다. `description` 필드는 해당 책임을 명확하게 설명합니다. 특히 중요한 점은 작업이 최종 완료된 후 `TERMINATE` 명령을 발행하여 전체 협업 프로세스를 정상적으로 종료하는 역할을 담당한다는 것입니다.

```python
def create_user_proxy():
    """Create user proxy agent"""
    return UserProxyAgent(
        name="UserProxy",
        description="""User proxy, responsible for the following duties:
1. Propose development requirements on behalf of users
2. Execute final code implementation
3. Verify whether functions meet expectations
4. Provide user feedback and suggestions

Please reply TERMINATE after completing the test.""",
    )
```

이러한 네 가지 독립적인 정의 기능을 통해 우리는 완전한 기능을 갖춘 "가상 팀"을 구축했을 뿐만 아니라 시스템 메시지를 통한 "신속한 엔지니어링"이 효율적인 다중 에이전트 애플리케이션 설계의 핵심 부분임을 입증했습니다.

(3) 팀 협업 프로세스 정의

이 경우 소프트웨어 개발 프로세스는 상대적으로 고정되어 있으므로(요구 사항 -> 코딩 -> 검토 -> 테스트) `RoundRobinGroupChat`(라운드 로빈 그룹 채팅)이 이상적인 선택입니다. 비즈니스 로직 순서대로 참가자 목록에 4명의 에이전트를 추가합니다.

```python
from autogen_agentchat.teams import RoundRobinGroupChat
from autogen_agentchat.conditions import TextMentionTermination

# Define team chat and collaboration rules
team_chat = RoundRobinGroupChat(
    participants=[
        product_manager,
        engineer,
        code_reviewer,
        user_proxy
    ],
    termination_condition=TextMentionTermination("TERMINATE"),
    max_turns=20,
)
```

- **참가자 순서:** `participants` 목록의 순서에 따라 상담원이 말하는 순서가 결정됩니다.
- **종료 조건:** `termination_condition`는 협업 프로세스 종료 시기를 제어하는 데 핵심입니다. 여기서는 "TERMINATE"라는 키워드가 포함된 메시지가 있으면 대화가 종료되도록 설정했습니다. 우리 설계에서는 이 명령이 최종 테스트 완료 후 `UserProxy`에 의해 발행됩니다.
- **최대 회전 수:** `max_turns`는 대화가 무한 루프에 빠지는 것을 방지하고 불필요한 리소스 소비를 방지하는 데 사용되는 안전 밸브입니다.

(4) 기동 및 실행

AutoGen `0.7.4`은 비동기식 아키텍처를 채택하므로 전체 협업 프로세스의 시작과 실행이 비동기식 기능으로 완료되고 최종적으로 `asyncio.run()`을 통해 실행됩니다.

```python
async def run_software_development_team():
    # ... Initialize client and agents ...

    # Define task description
    task = """We need to develop a Bitcoin price display application with the following specific requirements:
            Core functions:
            - Display Bitcoin current price in real-time (USD)
            - Display 24-hour price change trend (percentage and amount of increase/decrease)
            - Provide price refresh function

            Technical requirements:
            - Use Streamlit framework to create web application
            - Simple and beautiful interface, user-friendly
            - Add appropriate error handling and loading status

            Please team collaborate to complete this task, from requirement analysis to final implementation."""

    # Asynchronously execute team collaboration and stream output conversation process
    result = await Console(team_chat.run_stream(task=task))
    return result

# Main program entry
if __name__ == "__main__":
    result = asyncio.run(run_software_development_team())
```

프로그램이 실행되면 `task`이 초기 메시지로 `team_chat`에 전달되고, 제품 관리자는 첫 번째 참가자로 메시지를 수신한 후 전체 자동화된 협업 프로세스가 시작됩니다.

(5) 기대되는 협업 효과

이 소프트웨어 개발 팀을 운영하면 완전한 협업 프로세스를 관찰할 수 있습니다.

```bash
🔧 Initializing model client...
👥 Creating agent team...
🚀 Starting AutoGen software development team collaboration...
============================================================
---------- TextMessage (user) ----------
We need to develop a Bitcoin price display application with the following specific requirements:
...
Please team collaborate to complete this task, from requirement analysis to final implementation.
---------- TextMessage (ProductManager) ----------
### 1. Requirement Understanding and Analysis
...
Please engineer start implementation.
---------- TextMessage (Engineer) ----------
### Technical Solution Implementation
...
Please code reviewer check.
---------- TextMessage (CodeReviewer) ----------
### Code Review
...
Code review completed, please user proxy test.
---------- TextMessage (UserProxy) ----------
Requirements completed
---------- TextMessage (ProductManager) ----------
Great, thank you for your feedback! If you have any questions during use, or have other functional requirements and improvement suggestions, please feel free to let us know. We will continue to provide support and improvements. Looking forward to you having a pleasant experience with our application!
---------- TextMessage (Engineer) ----------
Glad to hear the project was completed successfully. If you or users have any questions or need help, please feel free to contact us. Thank you for your support of our work, let's work together to ensure the application runs stably and continuously optimize user experience!
---------- TextMessage (CodeReviewer) ----------
Thank you very much for everyone's efforts and collaboration, which enabled the project to be completed successfully. In the future, if there are more technical support needs or areas that need improvement, we are willing to contribute to the continuous optimization of the project. Looking forward to users enjoying a smooth experience, and also welcome more feedback and suggestions. Thank you again for the team's cooperation!
---------- TextMessage (UserProxy) ----------
Enter your response: TERMINATE
============================================================
✅ Team collaboration completed!

📋 Collaboration result summary:
- Number of participating agents: 4
- Task completion status: Success
```

전체 협업 프로세스는 **자연스러운 대화 기반 협업**, **역할 전문화 부서**, **프로세스 자동화 관리**, **완전한 개발 폐쇄 루프** 등 AutoGen 프레임워크의 장점을 보여줍니다.

### 6.2.4 AutoGen의 장점과 한계 분석

모든 기술 프레임워크에는 적용 가능한 특정 시나리오와 설계 장단점이 있습니다. 이 섹션에서는 AutoGen의 핵심 장점과 실제 응용 분야에서 직면할 수 있는 한계 및 과제를 객관적으로 분석합니다.

(1) 장점

- 사례에서 볼 수 있듯이 에이전트 팀을 위한 복잡한 상태 머신이나 제어 흐름 논리를 설계할 필요가 없으며 전체 소프트웨어 개발 프로세스를 제품 관리자, 엔지니어 및 리뷰어 간의 대화에 자연스럽게 매핑합니다. 이 접근 방식은 인간 팀의 협업 모드에 더 가깝고 복잡한 작업을 모델링하기 위한 임계값을 크게 낮춥니다. 개발자는 '어떻게(프로세스 제어)'보다는 '누가(역할)', '무엇을(책임)'을 정의하는 데 더 많은 에너지를 집중할 수 있습니다.
- 프레임워크를 사용하면 시스템 메시지(시스템 메시지)를 통해 각 에이전트에 고도로 전문화된 역할을 할당할 수 있습니다. 이 경우 `ProductManager`는 요구사항에 중점을 두고, `CodeReviewer`은 품질에 중점을 둡니다. 잘 설계된 에이전트는 다양한 프로젝트에서 재사용할 수 있으며 유지 관리 및 확장이 쉽습니다.
- 프로세스 지향 작업의 경우 `RoundRobinGroupChat`와 같은 메커니즘은 명확하고 예측 가능한 협업 프로세스를 제공합니다. 동시에 `UserProxyAgent`의 디자인은 "Human-in-the-Loop"를 위한 자연스러운 인터페이스를 제공합니다. 이는 작업 개시자 역할과 프로세스의 감독자 및 최종 수락자 역할을 모두 수행할 수 있습니다. 이 설계는 자동화된 시스템이 항상 사람의 감독하에 있음을 보장합니다.

(2) 제한사항

- `RoundRobinGroupChat`은 순차적인 프로세스를 제공하지만 LLM을 기반으로 한 대화는 본질적으로 불확실합니다. 상담원은 기대에서 벗어난 응답을 생성하여 대화가 예상치 못한 방향으로 진행되거나 심지어 루프에 빠질 수도 있습니다.
- 에이전트 팀의 작업 결과가 기대에 미치지 못하는 경우 디버깅 프로세스가 매우 까다로울 수 있습니다. 기존 프로그램과 달리 명확한 오류 스택은 없지만 긴 대화 기록을 얻습니다. 이를 "대화형 디버깅" 딜레마라고 합니다.

(3) Non-OpenAI 모델에 대한 구성 보충

OpenAI 시리즈가 아닌 모델 (such as DeepSeek, Tongyi Qianwen, etc.)을 사용하려면 버전 0.7.4에서 `OpenAIChatCompletionClient` 매개변수에 모델 정보 사전을 전달해야 합니다. DeepSeek을 예로 들면 다음과 같습니다.

```python
from autogen_ext.models.openai import OpenAIChatCompletionClient

model_client = OpenAIChatCompletionClient(
    model="deepseek-chat",
    api_key=os.getenv("DEEPSEEK_API_KEY"),
    base_url="https://api.deepseek.com/v1",
    model_info={
        "function_calling": True,
        "max_tokens": 4096,
        "context_length": 32768,
        "vision": False,
        "json_output": True,
        "family": "deepseek",
        "structured_output": True,
    }
)
```

이 `model_info` 사전은 AutoGen이 모델의 기능 경계를 이해하는 데 도움이 되므로 다양한 모델 서비스에 더 잘 적응할 수 있습니다.



## 6.3 프레임워크 2: AgentScope

AutoGen의 설계 철학이 "대화를 통한 협업 추진"이라면 AgentScope는 **엔지니어링 우선 다중 에이전트 플랫폼**이라는 또 다른 기술 경로를 나타냅니다. Alibaba DAMO Academy에서 개발한 AgentScope는 안정성이 뛰어난 대규모 다중 에이전트 애플리케이션을 구축하기 위해 특별히 설계되었습니다. 직관적이고 사용하기 쉬운 프로그래밍 인터페이스를 제공할 뿐만 아니라, 더 중요한 것은 분산 배포, 오류 복구, 관찰 가능성과 같은 엔터프라이즈급 기능이 내장되어 있어 장기간 안정적으로 실행해야 하는 프로덕션 환경 애플리케이션을 구축하는 데 특히 적합합니다.

### 6.3.1 AgentScope 설계

AutoGen과 비교할 때 AgentScope의 핵심 차이점은 **메시지 기반 아키텍처 설계** 및 **산업급 엔지니어링 방식**에 있습니다. AutoGen이 유연한 "대화 스튜디오"에 가깝다면 AgentScope는 개발자에게 개발, 테스트, 배포까지 전체 수명주기 지원을 제공하는 완전한 "에이전트 운영 체제"입니다. 많은 프레임워크에서 채택하는 상속 기반 설계와 달리 AgentScope는 **구성 아키텍처** 및 **메시지 기반 모드**를 선택합니다. 이 설계는 시스템의 모듈성을 향상시킬 뿐만 아니라 탁월한 동시성 성능 및 분산 기능의 기반을 마련합니다.

(1) 계층화된 아키텍처 시스템

그림 6.2에서 볼 수 있듯이 AgentScope는 명확한 계층의 모듈식 설계를 채택하여 하위 수준의 기본 구성 요소부터 최상위 수준의 애플리케이션 오케스트레이션에 이르기까지 완전한 에이전트 개발 생태계를 형성합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/6-figures/03.png" alt="" width="90%"/>
<p>그림 6.2 AgentScope 아키텍처 다이어그램</p>
</div>

이 아키텍처에서 맨 아래 레이어는 전체 프레임워크에 대한 핵심 구성 요소를 제공하는 **기본 구성 요소** 레이어입니다. `Message` 구성 요소는 통합 메시지 형식을 정의하여 간단한 텍스트 상호 작용부터 복잡한 다중 모드 콘텐츠까지 모든 것을 지원합니다. `Memory` 구성요소는 단기 및 장기 메모리 관리를 제공합니다. `Model API` 레이어는 다양한 대규모 언어 모델에 대한 호출을 추상화합니다. `Tool` 구성요소는 외부 세계와 상호 작용하는 에이전트의 능력을 캡슐화합니다.

기본 구성 요소 위에 있는 **에이전트 수준 인프라** 계층은 더 높은 수준의 추상화를 제공합니다. 이 계층에는 사전 구축된 다양한 에이전트(예: 브라우저 사용 에이전트, 심층 연구 에이전트)가 포함될 뿐만 아니라 에이전트 후크, 병렬 도구 호출 및 상태 관리와 같은 고급 기능을 지원하는 클래식 ReAct 패러다임도 구현됩니다. 특히 주목할만한 점은 이 레이어가 기본적으로 **비동기 실행 및 실시간 제어**를 지원한다는 점입니다. 이는 다른 프레임워크에 비해 AgentScope의 중요한 장점입니다.

**다중 에이전트 협력** 계층은 AgentScope의 핵심 혁신이 있는 곳입니다. `MsgHub`은 에이전트 간의 메시지 라우팅 및 상태 관리를 담당하는 메시지 센터 역할을 합니다. `Pipeline` 시스템은 유연한 워크플로 조정 기능을 제공하여 순차 및 동시 실행과 같은 다양한 실행 모드를 지원합니다. 이 설계를 통해 개발자는 복잡한 다중 에이전트 공동 작업 시나리오를 쉽게 구축할 수 있습니다.

최상위 **배포 및 개발** 계층은 AgentScope의 엔지니어링 강조를 반영합니다. `AgentScope Runtime`은 프로덕션급 런타임 환경을 제공하고, `AgentScope Studio`은 개발자에게 완전한 시각적 개발 도구 체인을 제공합니다.

(2) 메시지 중심

AgentScope의 핵심 혁신은 **메시지 기반 아키텍처**에 있습니다. 이 아키텍처에서는 모든 에이전트 상호 작용이 기존 함수 호출이 아닌 **메시지** 전송 및 수신으로 추상화됩니다.

```python
from agentscope.message import Msg

# Standard structure of message
message = Msg(
    name="Alice",           # Sender name
    content="Hello, Bob!",  # Message content
    role="user",           # Role type
    metadata={             # Metadata information
        "timestamp": "2024-01-15T10:30:00Z",
        "message_type": "text",
        "priority": "normal"
    }
)
```

메시지를 상호 작용의 기본 단위로 사용하면 다음과 같은 몇 가지 주요 이점이 있습니다.

- **비동기식 디커플링**: 메시지의 발신자와 수신자가 서로를 기다릴 필요 없이 시간에 맞춰 분리되어 자연스럽게 높은 동시성 시나리오를 지원합니다.
- **위치 투명성**: 에이전트는 다른 에이전트가 로컬 프로세스에 있는지 아니면 원격 서버에 있는지 신경 쓸 필요가 없습니다. 메시지 시스템이 자동으로 라우팅을 처리합니다.
- **관찰 가능성**: 모든 메시지를 기록, 추적 및 분석할 수 있으므로 복잡한 시스템의 디버깅 및 모니터링이 크게 단순화됩니다.
- **신뢰성**: 메시지를 지속적으로 저장하고 재시도할 수 있습니다. 시스템에 장애가 발생하더라도 상호 작용의 최종 일관성을 보장하여 시스템의 내결함성을 향상시킬 수 있습니다.

(3) 에이전트 수명주기 관리

AgentScope에서 각 에이전트는 명확한 수명 주기 (initialization, running, pausing, destruction, etc.)를 가지며 통합된 기본 클래스 `AgentBase`를 기반으로 구현됩니다. 개발자는 일반적으로 핵심 `reply` 방법에만 집중하면 됩니다.

```python
from agentscope.agents import AgentBase

class CustomAgent(AgentBase):
    def __init__(self, name: str, **kwargs):
        super().__init__(name=name, **kwargs)
        # Agent initialization logic

    def reply(self, x: Msg) -> Msg:
        # Agent's core response logic
        response = self.model(x.content)
        return Msg(name=self.name, content=response, role="assistant")

    def observe(self, x: Msg) -> None:
        # Agent's observation logic (optional)
        self.memory.add(x)
```

이 디자인 패턴은 에이전트의 내부 논리를 외부 통신과 분리합니다. 개발자는 `reply` 메서드에서 에이전트가 "생각하고 응답하는" 방법만 정의하면 됩니다.

(4) 메시지 전달 메커니즘

AgentScope에는 전체 메시지 기반 아키텍처의 허브인 **메시지 센터(MsgHub)**가 내장되어 있습니다. MsgHub는 메시지 라우팅 및 배포를 담당할 뿐만 아니라 지속성 및 분산 통신과 같은 고급 기능도 통합합니다. 다음과 같은 특징이 있습니다.

- **유연한 메시지 라우팅**: 지점 간, 브로드캐스트, 멀티캐스트 등 다양한 통신 모드를 지원하며 유연하고 복잡한 상호 작용 네트워크를 구축할 수 있습니다.
- **메시지 지속성**: 모든 메시지를 데이터베이스(예: SQLite, MongoDB)에 자동으로 저장하여 장기 실행 작업 상태를 복구할 수 있습니다.
- **기본 분산 지원**: 이는 AgentScope의 대표적인 기능입니다. 에이전트는 다양한 프로세스나 서버에 배포될 수 있으며 `MsgHub`는 개발자에게 완전히 투명하게 RPC(원격 프로시저 호출)를 통해 노드 간 통신을 자동으로 처리합니다.

기본 아키텍처에서 제공되는 이러한 엔지니어링 기능은 높은 동시성과 높은 안정성이 필요한 복잡한 애플리케이션 시나리오를 처리할 때 AgentScope를 기존 대화 기반 프레임워크보다 더 유리하게 만듭니다. 물론 이를 위해서는 개발자가 메시지 중심의 비동기 프로그래밍 패러다임을 이해하고 이에 적응해야 합니다.

다음 섹션에서는 구체적인 실제 사례인 삼국지 늑대인간 게임을 통해 AgentScope 프레임워크의 기능, 특히 동시 상호 작용 처리 시 이점을 깊이 경험해 보겠습니다.

### 6.3.2 삼국지 늑대인간 게임

AgentScope의 메시지 기반 아키텍처와 다중 에이전트 협업 기능을 깊이 이해하기 위해 중국 고전 문화 요소를 통합한 "삼국지 늑대인간" 게임을 구축할 것입니다. 이 사례는 복잡한 다중 에이전트 상호 작용을 처리하는 데 있어 AgentScope의 장점을 보여줄 뿐만 아니라 더 중요한 것은 **실시간 협업**, **롤플레잉** 및 **전략적 게임**이 필요한 시나리오에서 메시지 기반 아키텍처의 기능을 최대한 활용하는 방법을 보여줍니다. 전통적인 늑대인간과 달리 "삼국지 늑대인간"은 유비, 관우, 제갈량과 같은 고전 캐릭터를 게임에 도입합니다. 각 에이전트는 늑대인간의 기본 작업(예: 늑대인간 살해, 선견자 확인, 마을 주민 추론)을 완료해야 할 뿐만 아니라 해당 삼국지 캐릭터의 성격 특성과 행동 패턴을 구현해야 합니다. 이 설계를 통해 **다단계 역할 모델링** 처리 시 AgentScope의 성능을 관찰할 수 있습니다.

(1) 아키텍처 설계 및 핵심 구성요소

이 사례의 시스템 설계는 게임 로직을 3개의 독립적인 수준으로 나누는 계층적 분리 원칙을 따르며, 각 수준은 AgentScope의 하나 이상의 핵심 구성 요소에 매핑됩니다.

- **게임 제어 레이어**: `ThreeKingdomsWerewolfGame` 클래스는 전역 상태(플레이어 생존 목록, 현재 게임 단계 등) 유지, 게임 프로세스 진행(야간 단계, 주간 단계 호출) 및 승패 판단을 담당하는 게임의 메인 컨트롤러 역할을 합니다.
- **에이전트 상호 작용 계층**: `MsgHub`에 의해 완전히 구동됩니다. 늑대인간 간의 비밀 협상이든 낮 동안의 공개 토론이든 에이전트 간의 모든 통신은 메시지 센터를 통해 라우팅되고 배포됩니다.
- **역할 모델링 레이어**: 각 플레이어는 `DialogAgent`을 기반으로 하는 인스턴스입니다. 세심하게 설계된 시스템 프롬프트를 통해 각 에이전트에 '게임 역할'과 '삼국지 성격'이라는 이중 정체성을 주입합니다.

(2) 메시지 중심의 게임 흐름

이 사례의 핵심 디자인은 게임 흐름을 관리하기 위해 **상태 머신** 대신 **메시지 중심**을 사용하는 것입니다. 기존 구현에서 게임 단계 전환은 일반적으로 중앙 집중식 상태 시스템에 의해 제어됩니다. AgentScope 패러다임에서 게임 흐름은 잘 정의된 일련의 메시지 상호 작용 패턴으로 자연스럽게 모델링됩니다.

예를 들어, 늑대인간 단계의 구현은 단순한 함수 호출이 아니라 `MsgHub`을 통해 늑대인간 플레이어만 포함하는 임시 개인 통신 채널을 동적으로 생성합니다.

```python
async def werewolf_phase(self, round_num: int):
    """Werewolf phase - demonstrating message-driven collaboration mode"""
    if not self.werewolves:
        return None

    # Establish werewolf-exclusive communication channel through message center
    async with MsgHub(
        self.werewolves,
        enable_auto_broadcast=True,
        announcement=await self.moderator.announce(
            f"Werewolves, please discuss tonight's kill target. Surviving players: {format_player_list(self.alive_players)}"
        ),
    ) as werewolves_hub:
        # Discussion phase: werewolves exchange strategies through messages
        for _ in range(MAX_DISCUSSION_ROUND):
            for wolf in self.werewolves:
                await wolf(structured_model=DiscussionModelCN)

        # Voting phase: collect and count werewolves' kill decisions
        werewolves_hub.set_auto_broadcast(False)
        kill_votes = await fanout_pipeline(
            self.werewolves,
            msg=await self.moderator.announce("Please choose kill target"),
            structured_model=WerewolfKillModelCN,
            enable_gather=False,
        )
```

이 디자인의 장점은 게임 로직이 일련의 엄격한 상태 전환이 아니라 "특정 상황에서 어떤 메시지 교환 모드를 수행할지"로 명확하게 표현된다는 것입니다. 일일 토론(전체 방송), 선견자 확인(점대점 요청) 및 기타 단계는 모두 동일한 설계 패러다임을 따릅니다.

(3) 구조화된 출력으로 게임 규칙을 제한

늑대인간 게임의 주요 과제는 에이전트 행동이 게임 규칙을 준수하는지 확인하는 방법입니다. AgentScope의 **구조화된 출력 메커니즘**은 이 문제에 대한 솔루션을 제공합니다. 우리는 다양한 게임 동작에 대해 엄격한 데이터 모델을 정의합니다.

```python
class DiscussionModelCN(BaseModel):
    """Output format for discussion phase"""
    reach_agreement: bool = Field(
        description="Whether consensus has been reached",
        default=False
    )
    confidence_level: int = Field(
        description="Confidence level in current reasoning (1-10)",
        ge=1, le=10,
        default=5
    )
    key_evidence: Optional[str] = Field(
        description="Key evidence supporting your viewpoint",
        default=None
    )

class WitchActionModelCN(BaseModel):
    """Output format for witch action"""
    use_antidote: bool = Field(description="Whether to use antidote")
    use_poison: bool = Field(description="Whether to use poison")
    target_name: Optional[str] = Field(description="Poison target player name")
```

이러한 방식으로 우리는 에이전트 출력의 **형식 일관성**을 보장할 뿐만 아니라 더 중요한 것은 **게임 규칙의 자동화된 제약**을 달성합니다. 예를 들어, 마녀 요원은 같은 대상에게 동시에 해독제와 독을 사용할 수 없으며, 예언자는 하루에 한 명의 플레이어만 확인할 수 있습니다. 이러한 제약 조건은 데이터 모델의 필드 정의 및 유효성 검사 논리를 통해 자동으로 실행됩니다.

(4) 역할모델링의 이중적 도전

이 경우 가장 흥미로운 기술적 과제는 에이전트가 **게임 기능적 역할** (werewolf, seer, etc.) 및 **문화적 성격 역할** (Liu Bei, Cao Cao, etc.)이라는 두 가지 수준의 역할을 동시에 잘 수행하도록 만드는 방법입니다. 우리는 신속한 엔지니어링을 통해 이 문제를 해결합니다.

```python
def get_role_prompt(role: str, character: str) -> str:
    """Get role prompt - integrating game rules and character personality"""
    base_prompt = f"""You are {character}, playing {role} in this Three Kingdoms Werewolf game.

Important rules:
1. You can only participate in the game through dialogue and reasoning
2. Do not attempt to call any external tools or functions
3. Strictly reply in the required JSON format

Role characteristics:
"""

    if role == "Werewolf":
        return base_prompt + f"""
- You are in the werewolf camp, with the goal of eliminating all good people
- At night, you can negotiate with other werewolves on kill targets
- During the day, you must hide your identity and mislead good people
- Speak and act with {character}'s personality
"""
```

이 디자인을 통해 우리는 흥미로운 현상을 관찰할 수 있습니다. 동일한 게임 역할을 수행할 때 서로 다른 삼국지 캐릭터가 완전히 다른 전략과 음성 스타일을 나타냅니다. 예를 들어, 늑대인간 역을 맡은 '조조'는 더 교활하고 변장에 능해 보일 수 있는 반면, 늑대인간 역을 맡은 '장페이'는 더 직접적이고 충동적으로 보일 수 있습니다.

(5) 동시 처리 및 결함 허용 메커니즘

AgentScope의 비동기 아키텍처는 이 다중 에이전트 게임에서 중요한 역할을 합니다. 게임에는 투표 단계와 같이 **여러 에이전트로부터 동시에 결정을 수집**해야 하는 시나리오가 있는 경우가 많습니다.

```python
# Collect voting decisions from all players in parallel
vote_msgs = await fanout_pipeline(
    self.alive_players,
    await self.moderator.announce("Please vote to choose the player to eliminate"),
    structured_model=get_vote_model_cn(self.alive_players),
    enable_gather=False,
)
```

`fanout_pipeline`을 사용하면 동일한 메시지를 모든 에이전트에게 병렬로 보내고 응답을 비동기적으로 수집할 수 있습니다. 이는 게임의 실행 효율성을 향상시킬 뿐만 아니라, 더 중요하게는 실제 늑대인간 게임에서 "동시 투표" 시나리오를 시뮬레이션합니다. 동시에 주요 지점에 내결함성 처리를 추가합니다.

```python
try:
    response = await wolf(
        "Please analyze the current situation and express your viewpoint.",
        structured_model=DiscussionModelCN
    )
except Exception as e:
    print(f"⚠️ {wolf.name} error during discussion: {e}")
    # Create default response to ensure game continues
    default_response = DiscussionModelCN(
        reach_agreement=False,
        confidence_level=5,
        key_evidence="Unable to analyze temporarily"
    )
```

이 설계를 통해 에이전트에 예외가 발생하더라도 전체 게임 프로세스가 계속될 수 있습니다.

(6) 사례 출력 및 요약

AgentScope의 작동 메커니즘을 보다 직관적으로 경험하기 위해 다음은 게임의 야간 단계에서 발췌한 실제 실행 로그로, "손권"과 "주유"를 플레이하는 두 늑대인간 에이전트가 비밀 협상을 진행하고 킬을 실행하는 과정을 보여줍니다.

```
🎮 Welcome to Three Kingdoms Werewolf!

=== Game Initialization ===
Game Moderator: 📢 【Sun Quan】You are playing a werewolf in this Three Kingdoms Werewolf game, your character is Sun Quan. You can kill a player at night
Game Moderator: 📢 【Zhou Yu】You are playing a werewolf in this Three Kingdoms Werewolf game, your character is Zhou Yu. You can kill a player at night
...

Game Moderator: 📢 Three Kingdoms Werewolf game begins! Participants: Sun Quan, Zhou Yu, Cao Cao, Zhang Fei, Sima Yi, Zhao Yun
✅ Game setup complete, 6 players in total

=== Round 1 ===
🌙 Night 1 falls, everyone close your eyes...

【Werewolf Phase】
Game Moderator: 📢 🐺 Werewolves please open your eyes, choose tonight's kill target...
Game Moderator: 📢 Werewolves, please discuss tonight's kill target. Surviving players: Sun Quan, Zhou Yu, Cao Cao, Zhang Fei, Sima Yi, Zhao Yun

Sun Quan: Tonight we should eliminate Zhou Yu, this person is extremely intelligent and poses a great threat to us.
Zhou Yu: Sun Quan, you make a good point. But although Zhou Yu is intelligent, he may not be the biggest threat tonight. Cao Cao has great power, if we don't eliminate him early, it may be unfavorable to us.
Sun Quan: Cao Cao is indeed a threat, but if Zhou Yu lives, he can see through our schemes. Better to solve the immediate hidden danger first.
Zhou Yu: Sun Quan, your concern is not without reason. But if Cao Cao becomes our enemy, he can unite other forces to pose a greater threat to us.
Sun Quan: What you say also makes sense, Cao Cao's alliance is indeed troublesome. Then let's deal with Cao Cao first.
Zhou Yu: Very good, Sun Quan. Cao Cao is our primary target tonight.

Game Moderator: 📢 Please choose kill target
Sun Quan: I agree, Cao Cao must be eliminated.
Zhou Yu: I agree, Cao Cao is the target we need to solve tonight.

【Seer Phase】
Game Moderator: 📢 🔮 Seer please open your eyes, choose the player to verify...
Cao Cao: I want to verify Sun Quan.
Game Moderator: 📢 Verification result: Sun Quan is a werewolf

【Witch Phase】
Game Moderator: 📢 🧙‍♀️ Witch please open your eyes...
Game Moderator: 📢 Tonight Cao Cao was killed by werewolves
Zhang Fei: I used the antidote to save Cao Cao last night, now the antidote has been used up.
Game Moderator: 📢 You used the antidote to save Cao Cao

Game Moderator: 📢 Last night was peaceful, no one died.

【Day Discussion Phase】
Game Moderator: 📢 ☀️ Day 1 dawns, everyone please open your eyes...
Game Moderator: 📢 Now begin free discussion. Surviving players: Sun Quan, Zhou Yu, Cao Cao, Zhang Fei, Sima Yi, Zhao Yun

Sun Quan: Everyone, Cao Cao has great power and is a potential threat to us all. Tonight I suggest we concentrate our forces against him.
Zhou Yu: Sun Quan is absolutely right, Cao Cao is not only powerful himself but may also ally with other players, posing a greater threat to us.
Cao Cao: I verified Sun Quan last night, originally thought he was a good person, but the game moderator's result was werewolf. This means there's a werewolf lying.
Zhang Fei: I did save Cao Cao last night, indicating he was attacked by werewolves. But Cao Cao's verification result of Sun Quan is suspicious.
Sima Yi: There seems to be a contradiction between Cao Cao's verification result and Zhang Fei's saving action, we need more information to determine who is the werewolf.
Zhao Yun: The situation is indeed complex, we need to carefully analyze everyone's statements.

【Voting Phase】
Game Moderator: 📢 Please vote to choose the player to eliminate

Sun Quan: Cao Cao's threat is still great, I insist we should vote for him.
Zhou Yu: Based on last night's situation, I think we should vote for Cao Cao first, his threat is the greatest.
Cao Cao: I choose to vote for Sun Quan. According to the game moderator's feedback, Sun Quan is indeed a werewolf.
Zhang Fei: I insist on the fact that I saved Cao Cao last night, but the fact that Sun Quan was found to be a werewolf confuses me.
Sima Yi: We need more information to determine who is the werewolf.

[Game continues...]
```

### 6.3.3 AgentScope의 장점과 한계 분석

이번 "삼국지 늑대인간" 사례를 통해 우리는 AgentScope 프레임워크의 핵심 장점을 깊이 경험했습니다. 메시지 중심 아키텍처를 핵심으로 하는 프레임워크는 복잡한 게임 프로세스를 일련의 동시 비동기 메시지 전달 이벤트로 우아하게 매핑함으로써 기존 상태 시스템의 경직성과 복잡성을 방지합니다. 강력한 구조화된 출력 기능과 결합하여 게임 규칙을 코드 수준 제약 조건으로 직접 변환하여 시스템 안정성과 예측 가능성을 크게 향상시킵니다. 이 디자인 패러다임은 성능 측면에서 기본 동시성 이점을 보여줄 뿐만 아니라 단일 에이전트에 예외가 발생하더라도 전체 프로세스가 내결함성 처리에서 강력하게 실행될 수 있도록 보장합니다.

그러나 AgentScope의 엔지니어링 이점으로 인해 특정 복잡성 비용도 발생합니다. 메시지 기반 아키텍처는 강력하지만 개발자에게 비동기 프로그래밍, 분산 통신 및 기타 개념에 대한 이해가 필요한 높은 기술 요구 사항이 있습니다. 간단한 다중 에이전트 대화 시나리오의 경우 이 아키텍처는 "과잉 엔지니어링"의 위험이 있어 지나치게 복잡해 보일 수 있습니다. 또한 비교적 새로운 프레임워크로서 생태계와 커뮤니티 리소스는 여전히 추가 개선이 필요합니다. 따라서 AgentScope는 대규모의 신뢰성이 높은 프로덕션 수준 다중 에이전트 시스템을 구축하는 데 더 적합하며, 신속한 프로토타입 개발이나 간단한 애플리케이션 시나리오에는 보다 가벼운 프레임워크를 선택하는 것이 더 적합할 수 있습니다.



## 6.4 프레임워크 3: CAMEL

AutoGen 및 AgentScope와 같은 포괄적인 프레임워크와 달리 CAMEL의 원래 핵심 목표는 두 에이전트가 자율적으로 협력하여 최소한의 인간 개입으로 "역할극"을 통해 복잡한 작업을 해결할 수 있도록 하는 방법을 탐색하는 것입니다.

### 6.4.1 CAMEL의 자율적 협업

CAMEL의 자율적 협업의 초석은 **롤플레잉**과 **인셉션 프롬프트**라는 두 가지 핵심 개념입니다.

(1) 롤플레잉

CAMEL의 원래 디자인에서는 일반적으로 두 명의 에이전트가 협력하여 작업을 완료합니다. 이 두 에이전트에는 상호보완적이고 명확하게 정의된 "역할"이 할당됩니다. 하나는 요구 사항 제안, 지침 발행 및 작업 단계 구상을 담당하는 **"AI 사용자"** 역할을 합니다. 다른 하나는 특정 작업을 실행하고 지침에 따라 솔루션을 제공하는 **"AI 도우미"** 역할을 합니다.

예를 들어, "주식 거래 전략 분석 도구 개발" 작업에서:

- **AI 사용자** 역할은 "선임 주식 거래자"일 수 있습니다. 시장과 전략은 이해하지만 프로그래밍은 이해하지 못합니다.
- **AI Assistant** 역할은 "뛰어난 Python 프로그래머"입니다. 프로그래밍에는 능숙하지만 주식 거래에 대해서는 아무것도 모릅니다.

이러한 설정을 통해 작업 해결 과정은 자연스럽게 두 명의 '교차 도메인 전문가' 간의 대화로 전환됩니다. 거래자는 전문적인 요구 사항을 제안하고, 프로그래머는 이를 코드 구현으로 변환하며, 두 사람은 서로 협력하여 독립적으로 수행할 수 없는 복잡한 작업을 완료합니다.

(2) 시작 프롬프트

단순히 역할을 설정하는 것만으로는 충분하지 않습니다. 인간의 지속적인 감독 없이 두 AI가 항상 "자신의 역할을 유지"하고 공동 목표를 향해 효율적으로 이동할 수 있도록 어떻게 보장할 수 있습니까? CAMEL의 핵심 기술인 Inception Prompting이 발휘되는 곳입니다. "인셉션 프롬프트"는 대화가 시작되기 전에 두 에이전트에 주입되는 신중하게 설계되고 구조화된 초기 지침(시스템 프롬프트)입니다. 이 지침은 에이전트에 이식된 "작업 프로그램"과 같으며 일반적으로 다음과 같은 주요 부분을 포함합니다.

- **자신의 역할을 명확히 합니다**: 예를 들어 "당신은 선임 주식 거래자입니다..."
- **협력자의 역할 알리기**: 예를 들어 "당신은 훌륭한 Python 프로그래머와 함께 일하고 있습니다..."
- **공통 목표 정의**: 예를 들어 "귀하의 공통 목표는 주식 거래 전략 분석 도구를 개발하는 것입니다."
- **행동 제약 및 통신 프로토콜 설정**: 가장 중요한 부분입니다. 예를 들어, 지침은 AI 사용자에게 "한 번에 하나의 명확하고 구체적인 단계만 제안"하고 AI 보조자에게 "이전 단계를 완료하기 전에 더 자세한 내용을 묻지 않도록" 요구하는 동시에 작업 완료를 식별하기 위해 응답이 끝날 때 두 당사자가 특정 마커(예: `<SOLUTION>`)를 사용해야 함을 지정합니다.

이러한 제약은 대화가 주제에서 벗어나거나 비효율적인 루프에 빠지지 않고 그림 6.3에 표시된 것처럼 고도로 구조화된 작업 중심 방식으로 진행되도록 보장합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/6-figures/04.png" alt="" width="90%"/>
<p>그림 6.3 주식 거래 로봇을 만드는 CAMEL</p>
</div>

다음 섹션에서는 구체적인 예를 통해 이 프로세스를 경험해 보겠습니다.

### 6.4.2 AI 대중과학 전자책

CAMEL 프레임워크의 롤플레잉 기능을 이해하기 위해 AI 심리학자가 AI 저자와 협력하여 "미루는 심리학"에 대한 짧은 전자책을 공동 제작하는 등 실용적인 협력 사례를 구축할 것입니다. 이 사례는 두 명의 에이전트가 각자의 전문 영역을 활용하여 단일 에이전트가 어려움을 겪는 복잡한 창의적 작업을 공동으로 완료할 수 있도록 하는 CAMEL의 핵심 이점을 구현합니다.

(1) 태스크 설정

**시나리오 설정**: 과학적인 엄격함과 우수한 가독성이 모두 요구되는 일반 독자를 위한 미루기 심리학에 관한 인기 과학 전자책을 만듭니다.

**상담원 역할**:

- **심리학자**: 심리학에 대한 깊은 이론적 기초를 보유하고 인지행동과학, 신경과학 및 기타 관련 분야에 익숙하며 전문적인 학문적 통찰력과 실증적 연구 지원을 제공할 수 있습니다.
- **작가**: 뛰어난 글쓰기 능력과 서사 능력을 보유하고 있으며, 복잡한 학문적 개념을 생생하고 이해하기 쉬운 텍스트로 변환하는 데 능하며, 독자 경험과 내용 가독성에 중점을 둡니다.

(2) 협업 업무 정의

먼저 두 AI 전문가의 공통 목표를 명확히 할 필요가 있다. 우리는 상세한 문자열 `task_prompt`을 통해 이 작업을 정의합니다.

```python
from colorama import Fore
from camel.societies import RolePlaying
from camel.utils import print_text_animated
from camel.models import ModelFactory
from camel.types import ModelPlatformType
from dotenv import load_dotenv
import os

load_dotenv()
LLM_API_KEY = os.getenv("LLM_API_KEY")
LLM_BASE_URL = os.getenv("LLM_BASE_URL")
LLM_MODEL = os.getenv("LLM_MODEL")

# Create model, using Qwen as an example, calling Alibaba Cloud Bailian platform API
model = ModelFactory.create(
    model_platform=ModelPlatformType.QWEN,
    model_type=LLM_MODEL,
    url=LLM_BASE_URL,
    api_key=LLM_API_KEY
)

# Define collaboration task
task_prompt = """
Create a short e-book on "The Psychology of Procrastination" for general readers interested in psychology.
Requirements:
1. Content should be scientifically rigorous, based on empirical research
2. Language should be easy to understand, avoiding excessive professional terminology
3. Include practical improvement suggestions and case analysis
4. Length controlled at 8000-10000 words
5. Clear structure, including introduction, core chapters, and summary
"""

print(Fore.YELLOW + f"Collaboration task:\n{task_prompt}\n")
```

`task_prompt`은 전체 협업에 대한 "작업 사양"입니다. 이는 우리가 달성하려는 목표일 뿐만 아니라 CAMEL이 배후에서 "인셉션 프롬프트"를 생성하는 데 사용되어 두 에이전트 간의 대화가 항상 이 핵심 목표를 중심으로 진행되도록 합니다.

(3) 롤플레잉 '소사이어티' 초기화

다음으로 `RolePlaying` 세션 인스턴스를 만듭니다. 이것이 우리가 제공하는 역할과 업무를 기반으로 두 에이전트 협업 "사회"를 빠르게 구축하는 CAMEL의 핵심 작업입니다.

```python
# Initialize role-playing session
# AI writer as "user", responsible for proposing writing structure and requirements
# AI psychologist as "assistant", responsible for providing professional knowledge and content
role_play_session = RolePlaying(
    assistant_role_name="Psychologist",
    user_role_name="Writer",
    task_prompt=task_prompt,
    model=model,
    with_task_specify=False, # In this example, we directly use the given task_prompt
)

print(Fore.CYAN + f"Specific task description:\n{role_play_session.task_prompt}\n")
```

`RolePlaying`은 복잡한 프롬프트 엔지니어링을 캡슐화하는 CAMEL에서 제공하는 고급 API입니다. 두 역할과 작업의 이름만 전달하면 됩니다. CAMEL의 설계에서 `user` 역할은 대화의 "운전자"이자 "요구자"이고, `assistant` 역할은 "실행자" 및 "솔루션 제공자"입니다. 따라서 `user_role_name`에는 기획구조를 담당하는 '작가'를, `assistant_role_name`에는 전문지식 제공을 담당하는 '심리학자'를 배정합니다.

(4) 자동 대화 시작 및 실행

마지막으로 두 AI 전문가가 자동화된 협업을 시작할 수 있도록 전체 대화 프로세스를 구동하는 루프를 작성합니다.

```python
# Start collaboration conversation
chat_turn_limit, n = 30, 0
# Call init_chat() to get the initial conversation message generated by AI
input_msg = role_play_session.init_chat()

while n < chat_turn_limit:
    n += 1
    # step() method drives a complete round of conversation, AI user and AI assistant each speak once
    assistant_response, user_response = role_play_session.step(input_msg)

    # Check if messages are returned to prevent premature conversation termination
    if assistant_response.msg is None or user_response.msg is None:
        break

    print_text_animated(Fore.BLUE + f"Writer (AI User):\n\n{user_response.msg.content}\n")
    print_text_animated(Fore.GREEN + f"Psychologist (AI Assistant):\n\n{assistant_response.msg.content}\n")

    # Check task completion flag
    if "<CAMEL_TASK_DONE>" in user_response.msg.content or "<CAMEL_TASK_DONE>" in assistant_response.msg.content:
        print(Fore.MAGENTA + "✅ E-book creation completed!")
        break

    # Use assistant's reply as input for next round of conversation
    input_msg = assistant_response.msg

print(Fore.YELLOW + f"Total of {n} rounds of collaborative conversation")
```

이 `while` 루프는 자동화된 협업의 핵심입니다. 수동으로 서두를 작성할 필요 없이 작업과 역할에 따라 `init_chat()` 방식으로 대화가 자동으로 시작됩니다. 루프의 각 단계는 `step()`(작가가 요구 사항을 제안하고 심리학자가 콘텐츠를 제공함)를 호출하여 완전한 상호 작용 라운드를 구동하고 이전 라운드의 심리학자의 출력을 다음 라운드의 입력으로 사용하여 생성 체인을 형성합니다. 전체 프로세스는 미리 설정된 대화 차례 제한에 도달할 때까지 계속되거나 에이전트 중 하나가 작업 완료 플래그 `<CAMEL_TASK_DONE>`를 출력한 후 자동으로 종료됩니다.

(5) 협업 프로세스 시연

위의 코드를 실행하면 단조로운 Q&A의 긴 문자열만 나오는 것이 아니라 마치 인간 전문가 팀처럼 고도로 구조화된 협업 프로세스가 자동으로 진행되는 모습을 관찰할 수 있습니다. 전체 생성 과정은 자연스럽게 여러 단계로 나뉩니다.

**1단계(약 1~5라운드): 프레임워크 구축 및 목표 정렬** 대화 초기 단계에서는 먼저 "작가" 에이전트가 주도적인 역할을 맡아 전자책의 전체 구조와 장 배열에 대한 초기 아이디어를 제안합니다. 이후 '심리학자'는 이 프레임워크를 전문적인 관점에서 검토하고 보완하여 핵심 학문 모듈 (such as theoretical foundations, key concepts, etc.)이 누락되지 않도록 함으로써 협업 초기에 최종 결과물에 대한 합의에 도달합니다.

**2단계(약 6~20라운드): 핵심 콘텐츠 생성 및 지식 번역** 가장 효율적인 콘텐츠 제작 단계입니다. 협업 모드는 안정적인 "요청-응답" 루프가 됩니다.

- **심리학자**: "시간 할인 이론" 및 "실행 기능 결함"과 같은 핵심 개념에 대한 과학적 설명과 같은 "하드코어" 전문 지식을 제공하고 관점을 뒷받침하기 위해 관련 실험 연구를 인용하는 일을 담당합니다.
- **작가**: 엄밀하지만 모호할 수 있는 학문적 개념을 생생하고 비유적인 은유와 생활 사례로 풀어내는 '번역가'의 역할을 담당합니다. 예를 들어, "뇌의 현재 편견"이라는 개념을 "장기적인 건강이 아닌 즉각적인 사탕에만 관심이 있는 고의적인 아이"와 비교할 수 있습니다.

**3단계(약 21~25라운드): 반복 최적화 및 품질 보증** 책의 주요 내용이 완료되면 대화의 초점은 기존 텍스트를 다듬고 개선하는 데로 이동합니다. 이때 두 에이전트의 역할은 미묘한 변화를 겪게 됩니다.

- **작가**: 기사의 전체적인 유창성, 논리적 일관성, 언어 스타일을 검토하는 데 더 중점을 두고 '독자 경험' 관점에서 수정 제안을 제안합니다.
- **심리학자**: 번역 및 다듬기 과정에서 핵심 지식의 과학적 정확성이 손실되지 않도록 보장하고 보다 강력한 실증적 연구 지원으로 특정 관점을 보완하는 '사실 확인자' 역할을 다시 수행합니다.

**4단계(결론): 요약 및 고양** 마지막 몇 라운드의 대화에서는 양 당사자가 협력하여 실용적인 제안 요약과 전체 책에 대한 검토를 완료하여 전자책이 독자에게 깊은 인상을 남기고 실용적인 가치를 제공하는 명확하고 강력한 결말을 갖도록 합니다.

```
Collaboration task:
Create a short e-book on "The Psychology of Procrastination" for general readers interested in psychology.
Requirements:
1. Content should be scientifically rigorous, based on empirical research
2. Language should be easy to understand, avoiding excessive professional terminology
3. Include practical improvement suggestions and case analysis
4. Length controlled at 8000-10000 words
5. Clear structure, including introduction, core chapters, and summary

Specific task description:
Write an 8000–10000 word short e-book "The Psychology of Procrastination" for general readers: empirically based, easy to understand. Structure: introduction, causes (cognitive/emotional/reward), motivation and decision-making, habit formation and intervention, practical strategies and exercises, three case analyses, summary and resources. Each chapter contains research citations and actionable steps.

Writer:
Instruction: Please write a 400–600 word Chinese draft for the "Introduction" chapter of the e-book...
Input: None

Psychologist:
Solution:
Draft: Procrastination refers to the behavior and internal tendency of repeatedly postponing or avoiding a task despite knowing it should be completed. It can be an occasional time management problem...

Next request.

Writer:
Instruction: Please revise the following introduction draft into a 450–550 word Chinese text...
Input: Draft: Procrastination refers to the behavior and internal tendency of repeatedly postponing or avoiding a task...
.....
```

### 6.4.3 CAMEL의 장점과 한계 분석

이전 전자책 제작 사례를 통해 CAMEL 프레임워크 특유의 롤플레잉 패러다임을 깊이 경험했습니다. 이제 실제 프로젝트에서 현명한 기술적 선택을 하기 위해 이러한 디자인 철학의 장점과 한계를 객관적으로 분석해 보겠습니다.

(1) 장점

CAMEL의 가장 큰 장점은 "가벼운 구조, 무거운 프롬프트" 디자인 철학에 있습니다. AutoGen의 복잡한 대화 관리 및 AgentScope의 분산 아키텍처와 비교하여 CAMEL은 신중하게 설계된 초기 프롬프트를 통해 고품질 에이전트 협업을 달성할 수 있습니다. 자연스럽게 나타나는 이러한 공동 작업 동작은 하드 코딩된 워크플로보다 더 유연하고 효율적인 경우가 많습니다.

CAMEL 프레임워크가 급속한 개발과 진화를 겪고 있다는 점은 주목할 가치가 있습니다. [GitHub 저장소](https://github.com/camel-ai/camel),에서 CAMEL은 단순한 두 에이전트 협업 프레임워크 그 이상이며 현재 다음 기능을 갖추고 있음을 알 수 있습니다.

- **다중 모드 기능**: 텍스트, 이미지, 오디오 등 다양한 형식으로 상담원 협업을 지원합니다.
- **도구 통합**: 검색, 계산, 코드 실행 등을 포함한 풍부한 도구 라이브러리가 내장되어 있습니다.
- **모델 적응**: OpenAI, Anthropic, Google 및 오픈 소스 모델과 같은 여러 LLM 백엔드 지원
- **생태계 연계**: LangChain, CrewAI, AutoGen 등 주류 프레임워크와의 상호 운용성 달성

(2) 주요 제한사항

1. 신속한 엔지니어링에 대한 높은 의존도

CAMEL의 성공은 주로 초기 프롬프트의 품질에 달려 있습니다. 이로 인해 다음과 같은 몇 가지 과제가 발생합니다.

- **Prompt Design Threshold**: 대상 도메인 및 LLM 행동 특성에 대한 깊은 이해가 필요합니다.
- **디버깅 복잡성**: 협업이 효과적이지 않으면 문제가 역할 정의, 작업 설명 또는 상호 작용 규칙에 있는지 정확히 파악하기 어렵습니다.
- **일관성 문제**: 서로 다른 LLM은 동일한 프롬프트에 대해 서로 다른 이해를 가질 수 있습니다.

2. 협업 규모의 한계

CAMEL은 두 에이전트 협업에서 뛰어난 성능을 발휘하지만 대규모 다중 에이전트 시나리오를 처리할 때 문제에 직면합니다.

- **대화 관리**: AutoGen과 같은 복잡한 대화 라우팅 메커니즘이 부족합니다.
- **상태 동기화**: AgentScope와 같은 분산 상태 관리 기능이 없습니다.
- **갈등 해결**: 여러 에이전트가 동의하지 않는 경우 효과적인 중재 메커니즘이 부족합니다.

3. 업무 적용 범위

CAMEL은 심층적인 협업과 창의적 사고가 필요한 작업에 특히 적합하지만 다음과 같은 특정 시나리오에서는 최적의 선택이 아닐 수 있습니다.

- **엄격한 프로세스 제어**: 정밀한 단계 제어가 필요한 작업에는 LangGraph의 그래프 구조가 더 적합합니다.
- **대규모 동시성**: AgentScope의 메시지 기반 아키텍처는 동시성이 높은 시나리오에서 더 많은 이점을 제공합니다.
- **복잡한 의사결정 트리**: AutoGen의 그룹 채팅 모드는 다자간 의사결정 시나리오에서 더 유연합니다.

전반적으로 CAMEL은 독특하고 우아한 다중 에이전트 협업 패러다임을 나타냅니다. "인간 중심" 롤플레잉 설계를 통해 복잡한 시스템 엔지니어링 문제를 직관적인 대인 협력 패턴으로 변환합니다. 생태계가 지속적으로 개선되고 기능이 계속 확장됨에 따라 CAMEL은 지능형 협업 시스템 구축을 위한 중요한 선택 중 하나가 되고 있습니다.

## 6.5 프레임워크 4: LangGraph

### 6.5.1 LangGraph 구조 개요

LangChain 생태계의 중요한 확장인 LangGraph는 에이전트 프레임워크 설계에서 완전히 새로운 방향을 제시합니다. 이전에 소개된 "대화" 기반 프레임워크(예: AutoGen 및 CAMEL)와 달리 LangGraph는 에이전트의 실행 흐름을 **상태 머신**으로 모델링하고 이를 **방향 그래프**로 표현합니다. 이 패러다임에서 그래프의 **노드**는 특정 계산 단계(예: LLM 호출, 도구 실행)를 나타내고 **가장자리**는 한 노드에서 다른 노드로의 전환 논리를 정의합니다. 이 디자인의 혁명적인 측면은 기본적으로 루프를 지원하므로 반복, 반영 및 자체 수정이 가능한 복잡한 에이전트 워크플로를 전례 없이 직관적이고 간단하게 구축할 수 있다는 것입니다.

LangGraph를 이해하려면 먼저 LangGraph의 세 가지 기본 구성 요소를 파악해야 합니다.

**먼저 글로벌 상태(State)**입니다. 전체 그래프의 실행 프로세스는 공유 상태 개체를 중심으로 진행됩니다. 이 상태는 일반적으로 대화 기록, 중간 결과, 반복 횟수 등과 같이 추적해야 하는 모든 정보를 포함할 수 있는 Python `TypedDict`로 정의됩니다. 모든 노드는 이 중앙 상태를 읽고 업데이트할 수 있습니다.

```python
from typing import TypedDict, List

# Define global state data structure
class AgentState(TypedDict):
    messages: List[str]      # Conversation history
    current_task: str        # Current task
    final_answer: str        # Final answer
    # ... any other state to track
```

**둘째, 노드(Nodes)**입니다. 각 노드는 현재 상태를 입력으로 받고 업데이트된 상태를 출력으로 반환하는 Python 함수입니다. 노드는 특정 작업을 수행하는 단위입니다.

```python
# Define a "planner" node function
def planner_node(state: AgentState) -> AgentState:
    """Formulate a plan based on current task and update state."""
    current_task = state["current_task"]
    # ... call LLM to generate plan ...
    plan = f"Plan generated for task '{current_task}'..."

    # Append new message to state
    state["messages"].append(plan)
    return state

# Define an "executor" node function
def executor_node(state: AgentState) -> AgentState:
    """Execute latest plan and update state."""
    latest_plan = state["messages"][-1]
    # ... execute plan and get result ...
    result = f"Result of executing plan '{latest_plan}'..."

    state["messages"].append(result)
    return state
```

**마지막으로 엣지(Edges)**입니다. 엣지는 노드를 연결하고 워크플로 방향을 정의하는 역할을 담당합니다. 가장 간단한 에지는 한 노드의 출력이 항상 다른 고정 노드로 흐르도록 지정하는 일반 에지입니다. LangGraph의 가장 강력한 기능은 **조건부 가장자리**에 있습니다. 이는 함수를 사용하여 현재 상태를 판단한 후 다음으로 이동할 노드를 동적으로 결정합니다. 이것이 루프와 복잡한 논리적 분기를 구현하는 핵심입니다.

```python
def should_continue(state: AgentState) -> str:
    """Condition function: decide next route based on state."""
    # Assume if messages are less than 3, need to continue planning
    if len(state["messages"]) < 3:
        # Returned string needs to match the key defined when adding conditional edge
        return "continue_to_planner"
    else:
        state["final_answer"] = state["messages"][-1]
        return "end_workflow"
```

상태, 노드 및 에지를 정의한 후 이를 빌딩 블록과 같은 실행 가능한 워크플로로 조합할 수 있습니다.

```python
from langgraph.graph import StateGraph, END

# Initialize a state graph and bind our defined state structure
workflow = StateGraph(AgentState)

# Add node functions to the graph
workflow.add_node("planner", planner_node)
workflow.add_node("executor", executor_node)

# Set graph entry point
workflow.set_entry_point("planner")

# Add regular edge, connecting planner and executor
workflow.add_edge("planner", "executor")

# Add conditional edge, implementing dynamic routing
workflow.add_conditional_edges(
    # Starting node
    "executor",
    # Judgment function
    should_continue,
    # Route mapping: map judgment function's return value to target node
    {
        "continue_to_planner": "planner", # If returns "continue_to_planner", jump back to planner node
        "end_workflow": END               # If returns "end_workflow", end process
    }
)

# Compile graph, generate executable application
app = workflow.compile()

# Run graph
inputs = {"current_task": "Analyze recent AI industry news", "messages": []}
for event in app.stream(inputs):
    print(event)
```

### 6.5.2 3단계 Q&A 도우미
LangGraph의 핵심 개념을 이해한 후, 실제 사례를 통해 배운 내용을 통합해 보겠습니다. 우리는 사용자 질문에 답하기 위해 명확하고 고정된 3단계 프로세스를 따르는 단순화된 Q&A 대화 도우미를 구축할 것입니다.

1. **이해**: 먼저 사용자의 쿼리 의도를 분석합니다.
2. **검색**: 그런 다음 의도와 관련된 정보 검색을 시뮬레이션합니다.
3. **답변**: 마지막으로 의도와 검색된 정보를 바탕으로 최종 답변을 생성합니다.

이 사례에서는 상태를 정의하고, 노드를 생성하고, 이를 완전한 워크플로우에 선형적으로 연결하는 방법을 명확하게 보여줍니다. 코드를 상태 정의, 노드 생성, 그래프 구축, 애플리케이션 실행의 네 가지 핵심 단계로 분류하겠습니다.

(1) 전역 상태 정의

먼저 전체 워크플로를 통해 실행되는 전역 상태를 정의해야 합니다. **이것은 그래프의 각 노드 간에 전달되는 공유 데이터 구조로, 워크플로의 지속적인 컨텍스트 역할을 합니다.** 각 노드는 이 구조에서 데이터를 읽고 업데이트할 수 있습니다.

```python
from typing import TypedDict, Annotated
from langgraph.graph.message import add_messages

class SearchState(TypedDict):
    messages: Annotated[list, add_messages]
    user_query: str      # User requirement summary after LLM understanding
    search_query: str    # Optimized search query for Tavily API
    search_results: str  # Results returned by Tavily search
    final_answer: str    # Final generated answer
    step: str            # Mark current step
```

우리는 상태 객체에 대한 명확한 데이터 스키마를 정의하는 `SearchState` `TypedDict`을 만들었습니다. 핵심 설계는 `user_query` 및 `search_query` 필드를 모두 포함하는 것입니다. 이를 통해 에이전트는 먼저 사용자의 자연어 질문을 검색 엔진에 더 적합한 정제된 키워드로 최적화하여 검색 결과의 품질을 크게 향상시킬 수 있습니다.

(2) 워크플로우 노드 정의

상태 구조를 정의한 후 다음 단계는 워크플로를 구성하는 다양한 노드를 만드는 것입니다. LangGraph에서 각 노드는 특정 작업을 수행하는 Python 함수입니다. 이러한 함수는 현재 상태 객체를 입력으로 받고 업데이트된 필드가 포함된 사전을 반환합니다.

노드를 정의하기 전에 먼저 환경 변수 로드 및 대규모 언어 모델 인스턴스화를 포함하여 프로젝트 초기화 설정을 완료합니다.

```python
import os
from dotenv import load_dotenv
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage, AIMessage, SystemMessage
from tavily import TavilyClient

# Load environment variables from .env file
load_dotenv()

# Initialize model
# We will use this llm instance to drive the intelligence of all nodes
llm = ChatOpenAI(
    model=os.getenv("LLM_MODEL_ID", "gpt-4o-mini"),
    api_key=os.getenv("LLM_API_KEY"),
    base_url=os.getenv("LLM_BASE_URL", "https://api.openai.com/v1"),
    temperature=0.7
)
# Initialize Tavily client
tavily_client = TavilyClient(api_key=os.getenv("TAVILY_API_KEY"))
```

이제 3개의 핵심 노드를 하나씩 생성해 보겠습니다.

(1) 노드 이해 및 쿼리

이 노드는 워크플로의 첫 번째 단계입니다. 그 책임은 사용자 의도를 이해하고 이에 대한 최적화된 검색어를 생성하는 것입니다.

```python
def understand_query_node(state: SearchState) -> dict:
    """Step 1: Understand user query and generate search keywords"""
    user_message = state["messages"][-1].content

    understand_prompt = f"""Analyze the user's query: "{user_message}"
Please complete two tasks:
1. Concisely summarize what the user wants to know
2. Generate keywords most suitable for search engines (Chinese or English, must be precise)

Format:
Understanding: [User requirement summary]
Search terms: [Best search keywords]"""

    response = llm.invoke([SystemMessage(content=understand_prompt)])
    response_text = response.content

    # Parse LLM's output, extract search keywords
    search_query = user_message # Default to using original query
    if "Search terms:" in response_text or "搜索词：" in response_text:
        if "Search terms:" in response_text:
            search_query = response_text.split("Search terms:")[1].strip()
        else:
            search_query = response_text.split("搜索词：")[1].strip()

    return {
        "user_query": response_text,
        "search_query": search_query,
        "step": "understood",
        "messages": [AIMessage(content=f"I will search for you: {search_query}")]
    }
```

이 노드는 구조화된 프롬프트를 사용하여 LLM이 "의도 이해"와 "키워드 생성"이라는 두 가지 작업을 동시에 완료하도록 요구하고 구문 분석된 전용 검색 키워드를 상태의 `search_query` 필드에 업데이트하여 정확한 검색의 다음 단계를 준비합니다.

(2) 검색 노드

이 노드는 에이전트의 "도구 사용" 기능을 실행하는 역할을 담당합니다. 실제 인터넷 검색을 위해 Tavily API를 호출하고 기본적인 오류 처리 기능을 갖추고 있습니다.

```python
def tavily_search_node(state: SearchState) -> dict:
    """Step 2: Use Tavily API for real search"""
    search_query = state["search_query"]
    try:
        print(f"🔍 Searching: {search_query}")
        response = tavily_client.search(
            query=search_query, search_depth="basic", max_results=5, include_answer=True
        )
        # ... (process and format search results) ...
        search_results = ... # Formatted result string

        return {
            "search_results": search_results,
            "step": "searched",
            "messages": [AIMessage(content="✅ Search completed! Organizing answer...")]
        }
    except Exception as e:
        # ... (handle error) ...
        return {
            "search_results": f"Search failed: {e}",
            "step": "search_failed",
            "messages": [AIMessage(content="❌ Search encountered a problem...")]
        }
```

이 노드는 `tavily_client.search`을 통해 실제 API 호출을 시작합니다. 가능한 예외를 포착하기 위해 `try...except` 블록으로 래핑됩니다. 검색이 실패하면 `step` 상태를 `"search_failed"`로 업데이트합니다. 이는 다음 노드에서 대체 계획을 실행하는 데 사용됩니다.

(3) 응답 노드

최종 답변 노드는 이전 검색의 성공 여부에 따라 어느 정도 유연성을 갖고 다양한 답변 전략을 선택할 수 있습니다.

```python
def generate_answer_node(state: SearchState) -> dict:
    """Step 3: Generate final answer based on search results"""
    if state["step"] == "search_failed":
        # If search failed, execute fallback strategy, answer based on LLM's own knowledge
        fallback_prompt = f"Search API is temporarily unavailable, please answer the user's question based on your knowledge:\nUser question: {state['user_query']}"
        response = llm.invoke([SystemMessage(content=fallback_prompt)])
    else:
        # Search successful, generate answer based on search results
        answer_prompt = f"""Provide a complete and accurate answer to the user based on the following search results:
User question: {state['user_query']}
Search results:\n{state['search_results']}
Please synthesize the search results and provide an accurate, useful answer..."""
        response = llm.invoke([SystemMessage(content=answer_prompt)])

    return {
        "final_answer": response.content,
        "step": "completed",
        "messages": [AIMessage(content=response.content)]
    }
```

이 노드는 `state["step"]` 값을 확인하여 조건부 논리를 실행합니다. 검색에 실패하면 LLM의 내부 지식을 활용하여 답변하고 사용자에게 상황을 알립니다. 검색이 성공하면 실시간 검색 결과가 포함된 프롬프트를 사용하여 적시에 증거 기반 답변을 생성합니다.

(4) 그래프 작성

우리는 모든 노드를 함께 연결합니다.

```python
from langgraph.graph import StateGraph, START, END
from langgraph.checkpoint.memory import InMemorySaver

def create_search_assistant():
    workflow = StateGraph(SearchState)

    # Add nodes
    workflow.add_node("understand", understand_query_node)
    workflow.add_node("search", tavily_search_node)
    workflow.add_node("answer", generate_answer_node)

    # Set linear process
    workflow.add_edge(START, "understand")
    workflow.add_edge("understand", "search")
    workflow.add_edge("search", "answer")
    workflow.add_edge("answer", END)

    # Compile graph
    memory = InMemorySaver()
    app = workflow.compile(checkpointer=memory)
    return app
```

(5) 실행 케이스 시연

이 스크립트를 실행한 후 첫 번째 장의 사례와 같이 실시간 정보가 필요한 몇 가지 질문을 할 수 있습니다. `I'm going to Beijing tomorrow, what's the weather like? Are there suitable attractions?`

에이전트의 "사고" 프로세스가 터미널에 명확하게 표시되는 것을 볼 수 있습니다.

```
🔍 Intelligent Search Assistant Started!
I will use Tavily API to search for the latest and most accurate information for you
Supports various questions: news, technology, knowledge Q&A, etc.
(Enter 'quit' to exit)

🤔 What would you like to know: I'm going to Beijing tomorrow, what's the weather like? Are there suitable attractions?

============================================================
🧠 Understanding phase: I understand your needs: Understanding: The user wants to know about tomorrow's weather in Beijing and suitable attraction recommendations.
Search terms: Beijing tomorrow weather attraction recommendations Beijing weather tomorrow attractions
🔍 Searching: Beijing tomorrow weather attraction recommendations Beijing weather tomorrow attractions
🔍 Search phase: ✅ Search completed! Found relevant information, organizing answer for you...

💡 Final Answer:
Tomorrow (September 17, 2025) Beijing's weather forecast shows it is expected to be cloudy, with temperatures ranging from 17°C (62°F) to 25°C (77°F). This mild weather is very suitable for outdoor activities.

### Suitable Attraction Recommendations:
1. **Great Wall**: As one of China's most famous historical sites, the Great Wall is a must-visit. You can choose popular sections like Badaling or Mutianyu for your tour.

2. **Forbidden City**: The Forbidden City was the imperial palace of the Ming and Qing dynasties, with rich history and culture, suitable for tourists interested in Chinese history.

3. **Tiananmen Square**: This is one of China's symbols, with many important buildings and monuments on the square, suitable for taking photos.

4. **Summer Palace**: A very beautiful royal garden, suitable for strolling and enjoying natural scenery, especially the lakes and ancient buildings.

5. **798 Art District**: If you're interested in modern art, the 798 Art District is a place that integrates art, culture, and creativity, suitable for exploration and photography.

### Tips:
- Since tomorrow's weather is good, it's recommended to plan your travel route in advance and prepare some water and snacks to maintain sufficient energy during the tour.
- Since weather changes may affect the tour experience, it's recommended to check real-time weather updates.

Hope this information helps you arrange a pleasant Beijing trip! If you need more information about attractions or travel advice, feel free to ask anytime.

============================================================

🤔 What would you like to know:
```

그리고 지속적인 대화형 도우미이므로 계속해서 질문할 수 있습니다.

### 6.5.3 LangGraph의 장점과 한계 분석

모든 기술 프레임워크에는 적용 가능한 특정 시나리오와 설계 장단점이 있습니다. 본 섹션에서는 LangGraph의 핵심 장점과 실제 적용 시 직면할 수 있는 한계를 객관적으로 분석해 보겠습니다.

(1) 장점

- 지능형 검색 도우미 사례에서 볼 수 있듯이 LangGraph는 완전한 실시간 Q&A 프로세스를 상태, 노드 및 에지로 구성된 "흐름도"로 명시적으로 정의합니다. 이 디자인의 가장 큰 장점은 **높은 제어 가능성과 예측 가능성**입니다. 개발자는 에이전트 동작의 모든 단계를 정확하게 계획할 수 있으며, 이는 높은 안정성과 감사 가능성이 필요한 프로덕션 수준 애플리케이션을 구축하는 데 중요합니다. 가장 강력한 기능은 **사이클에 대한 기본 지원**에 있습니다. 조건부 가장자리를 통해 "반사 수정" 루프를 쉽게 구축할 수 있습니다. 예를 들어, 우리의 경우 검색이 실패하면 백업 계획으로 대체할 경로를 설계할 수 있습니다. 이는 자체 최적화 및 내결함성을 갖춘 에이전트를 구축하는 데 핵심입니다.

- 또한 각 노드가 독립적인 Python 함수이기 때문에 **높은 모듈성**을 제공합니다. 동시에 사람의 검토를 기다리는 노드를 프로세스에 삽입하는 것은 매우 간단해 신뢰할 수 있는 "Human-in-the-Loop" 협업을 구현하기 위한 견고한 기반을 제공합니다.

(2) 제한사항

- 대화 기반 프레임워크에 비해 LangGraph에서는 개발자가 더 많은 **상용구 코드**를 작성해야 합니다. 상태, 노드, 에지 및 일련의 작업을 정의하면 간단한 작업의 경우 개발 프로세스가 더욱 번거로워집니다. 개발자는 단순히 "무엇을 할지(what)"보다는 "프로세스를 어떻게 제어할지(how)"를 더 고민해야 합니다. 워크플로가 미리 정의되어 있기 때문에 LangGraph의 동작은 제어 가능하지만 대화형 에이전트의 동적 **"긴급" 상호 작용**이 부족합니다. 그 강점은 개방적이고 예측할 수 없는 사회적 협력을 시뮬레이션하는 것이 아니라 단호하고 신뢰할 수 있는 프로세스를 실행하는 데 있습니다.

- 디버깅 프로세스에도 문제가 있습니다. 대화 내역보다 프로세스가 명확하지만 노드 내의 논리적 오류, 노드 간에 전달되는 상태 데이터의 변형 또는 에지 전환 조건 판단의 실수 등 여러 지점에서 문제가 발생할 수 있습니다. 이를 위해서는 개발자가 전체 그래프의 작동 메커니즘을 전반적으로 이해해야 합니다.

## 6.6 장 요약

이번 장에서는 케이스 형태의 실습을 통해 최첨단 에이전트 프레임워크 중 일부를 경험했습니다.

우리는 각 프레임워크에 에이전트 구성 구현에 대한 고유한 접근 방식이 있다는 것을 확인했습니다.

- **AutoGen**은 복잡한 협업을 다중 역할로 추상화하고 자동으로 수행되는 '그룹 채팅'을 핵심으로 '대화를 통한 협업 추진'을 핵심으로 합니다.
- **AgentScope**는 산업 등급 애플리케이션의 견고성과 확장성에 중점을 두고 동시성 분산 다중 에이전트 시스템을 구축하기 위한 견고한 엔지니어링 기반을 제공합니다.
- **CAMEL**은 가벼운 "롤플레잉" 및 "인셉션 프롬프트" 패러다임을 통해 최소한의 코드로 두 전문 에이전트 간의 심층적이고 자율적인 협업을 촉진하는 방법을 보여줍니다.
- **LangGraph**는 보다 근본적인 "상태 머신" 모델로 돌아와 개발자가 명시적 그래프 구조, 특히 루프 기능을 통해 워크플로를 정밀하게 제어할 수 있게 하여 반사적이고 수정 가능한 에이전트를 구축할 수 있는 기반을 마련합니다.

이러한 프레임워크에 대한 심층 분석을 통해 **"긴급 협업"과 "명시적 제어" 사이의 선택**이라는 설계 균형을 추출할 수 있습니다. AutoGen과 CAMEL은 에이전트의 "역할"과 "목표"를 정의하는 데 더 많이 의존하므로 단순한 대화 규칙에서 복잡한 협업 동작이 "출현"될 수 있습니다. 이 접근 방식은 인간 상호 작용 패턴에 더 가깝지만 때로는 예측하고 디버깅하기가 어렵습니다. LangGraph에서는 개발자가 높은 신뢰성, 제어 가능성 및 관찰 가능성을 대가로 일부 "긴급" 놀라움을 희생하면서 모든 단계와 전환 조건을 명시적으로 정의해야 합니다. 동시에 AgentScope는 똑같이 중요한 두 번째 차원인 **엔지니어링**을 공개합니다. 어떤 협업 패러다임을 선택하든 실험적 프로토타입에서 프로덕션 애플리케이션으로 진행하려면 동시성, 내결함성, 분산 배포와 같은 엔지니어링 문제에 직면해야 합니다. AgentScope는 이러한 문제를 해결하기 위해 탄생했으며, "실행 가능"에서 "안정적으로 서비스 가능"으로의 중요한 도약을 나타냅니다.

요약하자면, 에이전트를 구축하는 방법은 한 가지만 있는 것이 아닙니다. 이 장에서 살펴본 프레임워크 디자인 철학을 깊이 이해하면 더 나은 "도구 사용자"가 될 뿐만 아니라 프레임워크 디자인의 다양한 장단점과 장단점도 이해할 수 있습니다.

다음 장에서는 이 튜토리얼의 핵심 내용을 입력하고 모든 이론과 실습을 통합하여 처음부터 자체 에이전트 프레임워크를 구축할 것입니다.


## 연습

1. 이 장에서는 `AutoGen`, `AgentScope`, `CAMEL` 및 `LangGraph`이라는 네 가지 고유한 에이전트 프레임워크를 소개했습니다. 분석해 주십시오:

- 6.1.2절의 표 6.1에서는 이 네 가지 프레임워크의 다양한 차원을 비교했습니다. 가장 익숙한 두 가지 프레임워크를 선택하고 '협업 모드', '제어 방법', '적용 시나리오'의 3가지 차원에서 심층적으로 비교해 보세요.
   - 이 장에서는 '긴급 협력'과 '명시적 통제' 사이의 균형에 대해 언급했습니다. 이 두 가지 디자인 철학의 의미를 어떻게 이해하시나요?

2. 6.2절의 `AutoGen` 사례에서는 "소프트웨어 개발팀"을 구성했습니다. 이 사례를 바탕으로 생각을 확장해 보십시오.

> **힌트**: 실습 문제이므로 실제 작동을 권장합니다.

- 현재 팀에서는 상담원이 정해진 순서로 말하는 `RoundRobinGroupChat`(라운드 로빈 그룹 채팅) 모드를 사용합니다. 요구 사항이 변경되어 재검토를 위해 엔지니어의 코드를 제품 관리자에게 반환해야 하는 경우 협업 프로세스를 어떻게 수정해야 합니까? "동적 롤백"을 지원하는 메커니즘을 설계하십시오.
   - 이 경우에는 `System Message`을 통해 각 Agent의 역할과 책임을 정의하였습니다. 이 팀에 "품질 보증"이라는 새로운 역할을 추가하고 코드 검토 후 자동화된 테스트를 수행할 수 있도록 시스템 메시지를 설계해 보세요.
   - `AutoGen`의 대화 협업은 잠재적으로 불안정하여 대화가 주제에서 벗어나거나 루프에 빠질 수 있습니다. 생각해 보십시오: 이상 징후가 감지될 때 개입할 "대화 품질 모니터링" 메커니즘을 어떻게 설계할 것인가?

3. 6.3절의 `AgentScope` 사례에서는 '삼국늑대인간' 게임을 구현했습니다. 심층적으로 분석해 보세요.

- 상담원간 커뮤니케이션을 관리하기 위해 `MsgHub`(메시지센터)를 사용한 사례입니다. 기존 함수 호출에 비해 메시지 기반 아키텍처가 어떤 이점이 있는지 설명해 주세요. 어떤 시나리오에서 이 아키텍처가 특히 유용합니까?
   - 게임은 에이전트 동작을 제한하기 위해 구조화된 출력(예: `DiscussionModelCN`, `WitchActionModelCN`)을 사용했습니다. 새로운 게임 역할 "Hunter"를 디자인하고 필드 정의 및 유효성 검사 규칙을 포함하여 해당 구조화된 출력 모델을 정의하십시오.
   - `AgentScope`은 분산 배포를 지원합니다. 즉, 다양한 에이전트가 다양한 서버에서 실행될 수 있습니다. 생각해 보십시오: "삼국지 늑대인간"과 같은 실시간 게임 시나리오에서 분산 배포는 어떤 기술적 과제를 가져오나요? 메시지 순서와 일관성을 보장하는 방법은 무엇입니까?

4. 6.4절의 `CAMEL` 사례에서는 심리학자와 작가가 협력하여 전자책을 만들었습니다.

- 이 경우 `<CAMEL_TASK_DONE>` 플래그가 감지되면 협업이 강제 종료됩니다. 그러나 두 대리인이 동의하지 않아(한 사람은 종료될 수 있다고 생각하고, 다른 사람은 종료되어서는 안 된다고 생각함) 합의에 도달할 수 없다면 어떻게 될까요? "충돌 해결" 호환성 메커니즘을 설계해 주세요.
   - `CAMEL`은 원래 두 에이전트 협업을 위해 설계되었지만 이제 다중 에이전트를 지원하도록 확장되었습니다. 다중 에이전트 공동 작업 모듈을 이해하려면 `CAMEL`의 최신 문서를 참조하세요.[`workforce`](https://docs.camel-ai.org/key_modules/workforce),) 아키텍처 다이어그램과 결합하여 `AutoGen`의 그룹 채팅 모드와 어떻게 다른지 설명하세요.

5. 섹션 6.5의 `LangGraph` 사례에서는 "3단계 Q&A 도우미"를 구축했습니다. 분석해 주십시오:

- `LangGraph`는 에이전트 프로세스를 상태 머신 및 방향성 그래프로 모델링합니다. 케이스의 "이해-검색-답변" 과정의 그래프 구조를 그려 노드, 에지, 상태 전이 조건을 표시해 주세요.
   - 현재 어시스턴트는 선형 프로세스입니다. "반사" 노드를 추가하여 이 사례를 확장하십시오. 생성된 답변 품질이 낮은 경우((e.g., too brief or lacking details)) 시스템은 답변을 다시 검색하거나 재생성해야 합니다. 이 루프 메커니즘에 대한 조건부 에지 논리를 설계하십시오.
   - `LangGraph`의 장점은 루프에 대한 기본 지원에 있습니다. 이 기능을 완전히 활용하는 보다 복잡한 애플리케이션 시나리오를 설계하십시오. 예를 들어 "코드 생성-테스트-수정" 루프, "논문 작성-검토-수정" 루프 등입니다. 전체 그래프 구조를 그리고 핵심 노드의 기능을 설명하십시오.

6. 프레임워크 선택은 에이전트 제품 개발의 주요 결정 중 하나입니다. 당신이 `AI` 회사의 기술 설계자이고 회사가 다음 세 가지 에이전트 제품 애플리케이션을 개발할 계획이라고 가정해 보겠습니다. 각 애플리케이션에 가장 적합한 프레임워크를 선택하고(`AutoGen`, `AgentScope`, `CAMEL`, `LangGraph` 또는 프레임워크 없이 처음부터 개발) 자세히 설명해주세요.

**애플리케이션 A**: 지능형 고객 서비스 시스템은 많은 수의 동시 사용자 요청(초당 1000명 이상)을 처리해야 하며, 2초 미만의 응답 시간이 필요하고, 시스템은 7×24시간 안정적으로 실행되어야 하며 수평적 확장을 지원해야 합니다.

**애플리케이션 B**: 과학 연구 논문 작성 지원 플랫폼에는 "연구원 에이전트"와 "작가 에이전트"가 필요하며 깊이 협력하여 문헌 검토, 실험 설계, 데이터 분석 및 논문 작성을 공동으로 완료합니다. 상담원은 여러 차례의 심층 토론을 수행하고 자율적으로 작업을 진행해야 합니다.

**신청 C**: 금융 위험 통제 승인 시스템, 엄격한 절차에 따라 대출 신청을 처리해야 합니다: 문서 검토 → 위험 평가 → 할당량 계산 → 준수 확인 → 수동 검토 → 최종 결정. 각 링크에는 명확한 판단 기준과 분기 논리가 있어 추적 및 감사 가능한 프로세스가 필요합니다.


## 참고자료

[1] 우 Q, Bansal G, Zhang J, 외. Autogen: 다중 에이전트 대화를 통해 차세대 LLM 애플리케이션 활성화[C]//언어 모델링에 관한 첫 번째 컨퍼런스. 2024.

[2] Gao D, Li Z, Pan X 등. Agentscope: 유연하면서도 강력한 다중 에이전트 플랫폼[J]. arXiv 사전 인쇄 arXiv:2402.14034, 2024.

[3] Li G, Hammoud H, Itani H 등. 낙타: 대규모 언어 모델 사회의 "마음" 탐색을 위한 의사소통 에이전트[J]. 신경 정보 처리 시스템의 발전, 2023, 36: 51991-52008.

[4] 랭체인. 랭그래프 [EB/OL]. (2024). https://github.com/langchain-ai/langgraph.

[5] 마이크로소프트. AutoGen - UserProxyAgent [EB/OL]. (2024). https://microsoft.github.io/autogen/stable/reference/python/autogen_agentchat.agents.html#autogen_agentchat.agents.UserProxyAgent.

