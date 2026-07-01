# 10장: 에이전트 통신 프로토콜

이전 장에서는 추론, 도구 호출 및 메모리 기능을 갖춘 완전한 기능의 독립형 에이전트를 구축했습니다. 그러나 더 복잡한 AI 시스템을 구축하려고 시도할 때 다음과 같은 자연스러운 질문이 발생합니다. **에이전트가 어떻게 외부 세계와 효율적으로 상호 작용할 수 있습니까? 여러 상담원이 어떻게 서로 협력할 수 있나요?**

이것이 바로 에이전트 통신 프로토콜이 해결하려는 핵심 문제입니다. 이 장에서는 HelloAgents 프레임워크에 세 가지 통신 프로토콜을 소개합니다. 에이전트와 도구 간의 표준화된 통신을 위한 **MCP(Model Context Protocol)**, 에이전트 간 P2P 협업을 위한 **A2A(Agent-to-Agent Protocol)**, 대규모 에이전트 네트워크 구축을 위한 **ANP(Agent Network Protocol)**입니다. 이 세 가지 프로토콜은 함께 에이전트 통신을 위한 인프라 계층을 형성합니다.

이 장의 학습을 통해 에이전트 통신 프로토콜의 설계 철학과 실무 기술을 익히고, 세 가지 주류 프로토콜 간의 설계 차이점을 이해하고, 실제 문제를 해결하기 위해 적절한 프로토콜을 선택하는 방법을 배우게 됩니다.

## 10.1 에이전트 통신 프로토콜 기본 사항

10.1.1 통신 프로토콜이 필요한 이유

이미 강력한 추론 및 도구 호출 기능을 갖춘 7장에서 구축한 ReAct 에이전트를 떠올려보세요. 일반적인 사용 시나리오를 살펴보겠습니다.

```python
from hello_agents import ReActAgent, HelloAgentsLLM
from hello_agents.tools import CalculatorTool, SearchTool

llm = HelloAgentsLLM()
agent = ReActAgent(name="AI Assistant", llm=llm)
agent.add_tool(CalculatorTool())
agent.add_tool(SearchTool())

# Agent can complete tasks independently
response = agent.run("Search for the latest AI news and calculate the total market value of related companies")
```

이 에이전트는 잘 작동하지만 세 가지 근본적인 한계에 직면해 있습니다. 첫 번째는 **도구 통합 딜레마**입니다. 새로운 외부 서비스(예: GitHub API, 데이터베이스, 파일 시스템)에 액세스해야 할 때마다 특수 도구 클래스를 작성해야 합니다. 이는 노동 집약적일 뿐만 아니라, 서로 다른 개발자가 작성한 도구는 서로 호환될 수도 없습니다. 두 번째는 **기능 확장 병목 현상**입니다. 에이전트의 기능은 사전 정의된 도구 세트로 제한되며 새 서비스를 동적으로 검색하고 사용할 수 없습니다. 마지막으로 **협업 부족**입니다. 여러 전문 에이전트가 공동 작업해야 할 정도로 작업이 복잡한 경우(예: 연구원 + 작가 + 편집자) 수동 조정을 통해서만 작업을 조정할 수 있습니다.

보다 구체적인 예를 통해 이러한 제한 사항을 이해해 보겠습니다. 다음을 수행해야 하는 지능형 연구 보조자를 구축한다고 가정해 보겠습니다.

```python
# Traditional approach: Manually integrate each service
class GitHubTool(BaseTool):
    """Need to manually write GitHub API adapter"""
    def run(self, repo_url):
        # Lots of API calling code...
        pass

class DatabaseTool(BaseTool):
    """Need to manually write database adapter"""
    def run(self, query):
        # Database connection and query code...
        pass

class WeatherTool(BaseTool):
    """Need to manually write weather API adapter"""
    def run(self, location):
        # Weather API calling code...
        pass

# Each new service requires repeating this process
agent.add_tool(GitHubTool())
agent.add_tool(DatabaseTool())
agent.add_tool(WeatherTool())
```

이 접근 방식에는 명백한 문제가 있습니다. 코드 중복 (each tool must handle HTTP requests, error handling, authentication, etc.), 유지 관리가 어렵고(API를 변경하려면 관련 도구를 모두 수정해야 함) 재사용할 수 없으며(다른 개발자의 도구를 직접 사용할 수 없음), 확장성이 낮습니다(새로운 서비스를 추가하려면 광범위한 코딩 작업이 필요함).

**통신 프로토콜의 핵심 가치**는 바로 이러한 문제를 해결하는 것입니다. 에이전트가 각 서비스에 대한 특수 어댑터를 작성할 필요 없이 통합된 방식으로 다양한 외부 서비스에 액세스할 수 있도록 하는 표준화된 인터페이스 사양 세트를 제공합니다. 이는 각 장치 유형에 대해 특수한 통신 코드를 작성할 필요 없이 서로 다른 장치가 서로 통신할 수 있도록 하는 인터넷의 TCP/IP 프로토콜과 같습니다.

통신 프로토콜을 사용하면 위 코드를 다음과 같이 단순화할 수 있습니다.

```python
from hello_agents.tools import MCPTool

# Connect to MCP server, automatically obtain all tools
mcp_tool = MCPTool()  # Built-in server provides basic tools

# Or connect to professional MCP servers
github_mcp = MCPTool(server_command=["npx", "-y", "@modelcontextprotocol/server-github"])
database_mcp = MCPTool(server_command=["python", "database_mcp_server.py"])

# Agent automatically obtains all capabilities without manually writing adapters
agent.add_tool(mcp_tool)
agent.add_tool(github_mcp)
agent.add_tool(database_mcp)
```

통신 프로토콜이 가져오는 변화는 근본적입니다. **표준화된 인터페이스**를 통해 다양한 서비스에서 통합 액세스 방법을 제공할 수 있고, **상호 운용성**을 통해 다양한 개발자의 도구를 원활하게 통합할 수 있으며, **동적 검색**을 통해 에이전트가 런타임에 새로운 서비스와 기능을 검색할 수 있으며, **확장성**을 통해 시스템이 새로운 기능 모듈을 쉽게 추가할 수 있습니다.

### 10.1.2 세 가지 프로토콜 설계 철학의 비교

에이전트 통신 프로토콜은 단일 솔루션이 아니라 다양한 통신 시나리오를 위해 설계된 일련의 표준입니다. 이 장에서는 현재 주류 프로토콜인 MCP, A2A, ANP 세 가지를 실습 예로 사용합니다. 아래는 개요 비교입니다.

**(1) MCP: 에이전트와 도구 간의 브리지**

MCP(Model Context Protocol)는 Anthropic 팀<sup>[1]</sup>에서 제안한 것으로 **에이전트와 외부 도구/리소스 간의 통신 방법을 표준화**하는 것이 핵심 설계 철학입니다. 에이전트가 파일 시스템, 데이터베이스, GitHub, Slack 등과 같은 다양한 서비스에 액세스해야 한다고 상상해 보세요. 전통적인 접근 방식은 각 서비스에 대한 특수 어댑터를 작성하는 것인데, 이는 노동 집약적일 뿐만 아니라 유지 관리도 어렵습니다. MCP는 모든 서비스에 동일한 방식으로 액세스할 수 있도록 하는 통합 프로토콜 사양을 정의합니다.

MCP의 디자인 철학은 "컨텍스트 공유"입니다. 이는 단순한 RPC(원격 프로시저 호출) 프로토콜이 아니라 더 중요한 점은 에이전트와 도구가 풍부한 상황 정보를 공유할 수 있다는 것입니다. 그림 10.1에서 볼 수 있듯이 에이전트가 코드 저장소에 액세스하면 MCP 서버는 파일 콘텐츠를 제공할 수 있을 뿐만 아니라 코드 구조, 종속성 관계, 커밋 기록과 같은 상황별 정보도 제공하여 에이전트가 보다 지능적인 결정을 내릴 수 있도록 합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-1.png" alt="" width="85%"/>
<p>그림 10.1 MCP 설계 철학</p>
</div>

**(2) A2A: 에이전트 간 대화**

A2A(Agent-to-Agent Protocol) 프로토콜은 Google팀<sup>2</sup>에서 제안한 프로토콜로, 핵심 설계 철학은 **에이전트 간 P2P 통신을 구현**하는 것입니다. 에이전트와 도구 간의 통신에 중점을 두는 MCP와 달리 A2A는 에이전트가 서로 협력하는 방식에 중점을 둡니다. 이 디자인을 통해 에이전트는 인간 팀처럼 대화, 협상 및 협업에 참여할 수 있습니다.

A2A의 디자인 철학은 'Peer-to-Peer 커뮤니케이션'입니다. 그림 10.2에 표시된 것처럼 A2A 네트워크에서 각 에이전트는 서비스 제공자이자 서비스 소비자입니다. 에이전트는 요청을 적극적으로 시작할 수 있으며 다른 에이전트의 요청에도 응답할 수 있습니다. 이 P2P 설계는 중앙 집중식 코디네이터의 병목 현상을 방지하여 에이전트 네트워크를 더욱 유연하고 확장 가능하게 만듭니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-2.png" alt="" width="85%"/>
<p>그림 10.2 A2A 디자인 철학</p>
</div>

**(3) ANP: 에이전트 네트워크를 위한 인프라**

ANP(에이전트 네트워크 프로토콜)는 개념적 프로토콜 프레임워크<sup>3</sup>로, 현재 오픈 소스 커뮤니티에서 유지관리하고 있으며 아직 성숙한 생태계를 갖추고 있지 않습니다. 핵심 설계 철학은 **대규모 에이전트 네트워크를 위한 인프라 구축**입니다. MCP가 "도구에 액세스하는 방법"을 해결하고 A2A가 "다른 에이전트와 대화하는 방법"을 해결한다면 ANP는 "대규모 네트워크에서 에이전트를 검색하고 연결하는 방법"을 해결합니다.

ANP의 디자인 철학은 "분산형 서비스 발견"입니다. 수백 또는 수천 명의 에이전트가 포함된 네트워크에서 에이전트는 필요한 서비스를 어떻게 찾을 수 있습니까? 그림 10.3에 표시된 것처럼 ANP는 서비스 등록, 검색 및 라우팅 메커니즘을 제공하여 에이전트가 모든 연결 관계를 사전 구성할 필요 없이 네트워크에서 다른 서비스를 동적으로 검색할 수 있도록 합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-3.png" alt="" width="85%"/>
<p>그림 10.3 ANP 설계 철학</p>
</div>

마지막으로 표 10.1에서 비교표를 사용하여 이 세 가지 프로토콜 간의 차이점을 보다 명확하게 이해해 보겠습니다.

<div align="center">
<p>표 10.1 세 가지 프로토콜 비교</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-1.png" alt="" width="85%"/>
</div>

**(4) 올바른 프로토콜을 선택하는 방법은 무엇입니까?**

현재 프로토콜은 아직 초기 개발 단계에 있습니다. 다양한 도구의 적시성은 관리자에 따라 다르지만 MCP의 생태계는 상대적으로 성숙합니다. 대기업이 지원하는 MCP 도구를 선택하는 것이 더 좋습니다.

프로토콜 선택의 핵심은 요구 사항을 이해하는 데 있습니다.

- 에이전트가 외부 서비스(파일, 데이터베이스, API)에 액세스해야 하는 경우 **MCP**를 선택하세요.
- 작업 공동 작업을 위해 여러 상담원이 필요한 경우 **A2A**를 선택하세요.
- 대규모 에이전트 생태계를 구축하고 싶다면 **ANP**를 고려해 보세요

### 10.1.3 HelloAgents 통신 프로토콜 아키텍처 설계

세 가지 프로토콜의 설계 철학을 이해한 후, HelloAgents 프레임워크에서 이를 구현하고 사용하는 방법을 살펴보겠습니다. 우리의 설계 목표는 **학습자가 복잡한 시나리오를 처리할 수 있는 충분한 유연성을 유지하면서 가장 간단한 방법으로 이러한 프로토콜을 사용할 수 있도록 지원**하는 것입니다.

그림 10.4에서 볼 수 있듯이 HelloAgents 통신 프로토콜 아키텍처는 아래에서 위로 프로토콜 구현 계층, 도구 캡슐화 계층 및 에이전트 통합 계층의 3개 계층 설계를 채택합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-4.png" alt="" width="85%"/>
<p>그림 10.4 HelloAgents 통신 프로토콜 설계</p>
</div>

**(1) 프로토콜 구현 계층**: 이 계층에는 세 가지 프로토콜의 특정 구현이 포함됩니다. MCP는 FastMCP 라이브러리를 기반으로 구현되어 클라이언트 및 서버 기능을 제공합니다. A2A는 Google의 공식 a2a-sdk를 기반으로 구현됩니다. ANP는 자체 개발한 경량 구현으로 서비스 검색 및 네트워크 관리 기능을 제공합니다. 물론 현재 공식적인 [구현](https://github.com/agent-network-protocol/AgentConnect),)도 있지만 향후 반복을 고려하여 여기서는 개념만 시뮬레이션합니다.

**(2) 도구 캡슐화 계층**: 이 계층은 프로토콜 구현을 통합 도구 인터페이스로 캡슐화합니다. MCPTool, A2ATool 및 ANPTool은 모두 BaseTool에서 상속되어 일관된 `run()` 메소드를 제공합니다. 이 설계를 통해 에이전트는 동일한 방식으로 다양한 프로토콜을 사용할 수 있습니다.

**(3) 에이전트 통합 계층**: 이 계층은 에이전트와 프로토콜 간의 통합 지점입니다. 모든 에이전트 (ReActAgent, SimpleAgent, etc.)는 기본 프로토콜 세부정보에 신경 쓸 필요 없이 도구 시스템을 통해 프로토콜 도구를 사용합니다.

### 10.1.4 이 장의 학습 목표 및 빠른 경험

먼저 10장의 학습 내용을 살펴보겠습니다.

```
hello_agents/
├── protocols/                          # Communication protocol module
│   ├── mcp/                            # MCP protocol implementation (Model Context Protocol)
│   │   ├── client.py                   # MCP client (supports 5 transport methods)
│   │   ├── server.py                   # MCP server (FastMCP wrapper)
│   │   └── utils.py                    # Utility functions (create_context/parse_context)
│   ├── a2a/                            # A2A protocol implementation (Agent-to-Agent Protocol)
│   │   └── implementation.py           # A2A server/client (based on a2a-sdk, optional dependency)
│   └── anp/                            # ANP protocol implementation (Agent Network Protocol)
│       └── implementation.py           # ANP service discovery/registration (conceptual implementation)
└── tools/builtin/                      # Built-in tools module
    └── protocol_tools.py               # Protocol tool wrappers (MCPTool/A2ATool/ANPTool)
```

이 장의 내용에서는 주로 적용에 중점을 두고 있으며, 학습 목표는 자신의 프로젝트에 프로토콜을 적용할 수 있는 능력을 갖추는 것입니다. 또한 프로토콜은 현재 초기 개발 단계에 있으므로 바퀴를 재발명하는 데 너무 많은 노력을 기울일 필요가 없습니다. 실제 작업을 시작하기 전에 개발 환경을 준비합시다.

```bash
# Install HelloAgents framework (Chapter 10 version)
pip install "hello-agents[protocol]==0.2.2"

# Install NodeJS, refer to documentation in Additional-Chapter
```

가장 간단한 코드로 세 가지 프로토콜의 기본 기능을 경험해 보겠습니다.

```python
from hello_agents.tools import MCPTool, A2ATool, ANPTool

# 1. MCP: Access tools
mcp_tool = MCPTool()
result = mcp_tool.run({
    "action": "call_tool",
    "tool_name": "add",
    "arguments": {"a": 10, "b": 20}
})
print(f"MCP calculation result: {result}")  # Output: 30.0

# 2. ANP: Service discovery
anp_tool = ANPTool()
anp_tool.run({
    "action": "register_service",
    "service_id": "calculator",
    "service_type": "math",
    "endpoint": "http://localhost:8080"
})
services = anp_tool.run({"action": "discover_services"})
print(f"Discovered services: {services}")

# 3. A2A: Agent communication
a2a_tool = A2ATool("http://localhost:5000")
print("A2A tool created successfully")
```

이 간단한 예는 세 가지 프로토콜의 핵심 기능을 보여줍니다. 다음 섹션에서는 각 프로토콜의 자세한 사용법과 모범 사례를 자세히 알아봅니다.


## 10.2 실제 MCP 프로토콜

이제 MCP에 대해 자세히 알아보고 에이전트가 외부 도구 및 리소스에 액세스할 수 있도록 하는 방법을 마스터해 보겠습니다.

### 10.2.1 MCP 프로토콜 개념 소개

**(1) MCP: 에이전트용 "USB-C"**

에이전트가 다음과 같은 여러 작업을 동시에 수행해야 할 수도 있다고 상상해 보세요.
- 로컬 파일 시스템에서 문서 읽기
- PostgreSQL 데이터베이스 쿼리
- GitHub에서 코드 검색
- Slack 메시지 보내기
- Google 드라이브에 액세스

전통적으로는 다양한 API, 인증 방법, 오류 처리 등을 처리하면서 각 서비스에 대한 어댑터 코드를 작성해야 했습니다. 이는 노동 집약적일 뿐만 아니라 유지 관리도 어렵습니다. 더 중요한 것은 다양한 LLM 플랫폼마다 함수 호출 구현이 크게 다르기 때문에 모델을 전환할 때 광범위한 코드 재작성이 필요하다는 것입니다.

MCP의 등장은 이 모든 것을 변화시켰습니다. USB-C가 다양한 장치의 연결 방법을 통합한 것처럼 **MCP는 에이전트와 외부 도구 간의 상호 작용 방법**을 통합했습니다. Claude, GPT 또는 기타 모델을 사용하든 MCP 프로토콜을 지원하는 한 동일한 도구와 리소스에 원활하게 액세스할 수 있습니다.

**(2) MCP 아키텍처**

MCP 프로토콜은 호스트, 클라이언트, 서버의 3계층 아키텍처 설계를 채택합니다. 그림 10.5의 시나리오를 통해 이러한 구성 요소가 어떻게 함께 작동하는지 이해해 보겠습니다.

Claude Desktop을 사용하면서 "내 데스크탑에 어떤 문서가 있나요?"라고 묻는다고 가정해 보겠습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-5.png" alt="" width="85%"/>
<p>그림 10.5 MCP 사례 시연</p>
</div>

**3계층 아키텍처의 책임:**

1. **호스트(호스트 계층)**: Claude Desktop은 사용자 질문을 받고 Claude 모델과 상호 작용하는 호스트 역할을 합니다. 호스트는 사용자가 직접 상호 작용하고 전체 대화 흐름을 관리하는 인터페이스입니다.

2. **클라이언트(클라이언트 계층)**: Claude 모델이 파일 시스템에 액세스해야 한다고 결정하면 호스트에 내장된 MCP 클라이언트가 활성화됩니다. 클라이언트는 적절한 MCP 서버와의 연결 설정, 요청 전송 및 응답 수신을 담당합니다.

3. **Server(Server Layer)**: 파일 시스템 MCP Server가 호출되어 실제 파일 검색 작업을 실행하고 데스크톱 디렉터리에 액세스하여 발견된 문서 목록을 반환합니다.

**완전한 상호작용 흐름:** 사용자 질문 → Claude Desktop(호스트) → Claude 모델 분석 → 파일 정보 필요 → MCP 클라이언트 연결 → 파일 시스템 MCP Server → 작업 실행 → 결과 반환 → Claude가 답변 생성 → Claude Desktop에 표시

이 아키텍처 설계의 장점은 **관심사 분리**에 있습니다. 호스트는 사용자 경험에 초점을 맞추고, 클라이언트는 프로토콜 통신에 초점을 맞추고, 서버는 특정 기능 구현에 초점을 맞춥니다. 개발자는 호스트와 클라이언트의 구현 세부 사항에 신경 쓰지 않고 해당 MCP 서버 개발에만 집중하면 됩니다.

**(3) MCP의 핵심 기능**

표 10.2에 표시된 것처럼 MCP 프로토콜은 세 가지 핵심 기능을 제공하여 완전한 도구 액세스 프레임워크를 구성합니다.

<div align="center">
<p>표 10.2 MCP 핵심 기능</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-2.png" alt="" width="85%"/>
</div>

이 세 가지 기능의 차이점은 **도구가 활성**(작업 실행), **리소스가 수동**(데이터 제공), **프롬프트가 유익함**(템플릿 제공)입니다.

**(4) MCP 워크플로**

그림 10.6에 표시된 대로 특정 예를 통해 MCP의 전체 작업 흐름을 이해해 보겠습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-6.png" alt="" width="85%"/>
<p>그림 10.6 MCP 사례 시연</p>
</div>

핵심 질문은 다음과 같습니다. **Claude(또는 다른 LLM)는 사용할 도구를 어떻게 결정합니까?**

사용자가 질문하면 전체 도구 선택 프로세스는 다음과 같습니다.

1. **도구 검색 단계**: MCP 클라이언트가 서버에 연결한 후 먼저 `list_tools()`를 호출하여 사용 가능한 모든 도구(도구 이름, 기능 설명, 매개변수 정의 포함)에 대한 설명 정보를 얻습니다.

2. **컨텍스트 구축**: 클라이언트는 도구 목록을 LLM이 이해할 수 있는 형식으로 변환하고 이를 시스템 프롬프트에 추가합니다. 예를 들어:
   ```
   You can use the following tools:
   - read_file(path: str): Read the content of the file at the specified path
   - search_code(query: str, language: str): Search in the codebase
   ```

3. **모델 추론**: LLM은 사용자의 질문과 사용 가능한 도구를 분석하여 도구 호출 여부와 호출할 도구를 결정합니다. 이 결정은 도구 설명과 현재 대화 컨텍스트를 기반으로 합니다.

4. **도구 실행**: LLM이 도구를 사용하기로 결정하면 클라이언트는 MCP 서버를 통해 선택한 도구를 실행하고 결과를 얻습니다.

5. **결과 통합**: 도구 실행 결과는 LLM으로 다시 전송되며, LLM은 결과를 결합하여 최종 답변을 생성합니다.

이 프로세스는 **완전히 자동화**되어 있으며, LLM은 도구 설명의 품질에 따라 도구 사용 여부와 사용 방법을 결정합니다. 따라서 명확하고 정확한 도구 설명을 작성하는 것이 중요합니다.

**(5) MCP와 함수 호출의 차이점**

많은 개발자들이 묻습니다. **이미 함수 호출을 사용하고 있는데 왜 MCP가 필요한가요?** 표 10.3을 통해 차이점을 이해해 보겠습니다.

<div align="center">
<p>표 10.3 함수 호출과 MCP 비교</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-3.png" alt="" width="85%"/>
</div>

여기서는 동일한 작업의 두 가지 구현을 자세히 비교하기 위해 GitHub 리포지토리와 로컬 파일 시스템에 액세스해야 하는 에이전트의 예를 사용합니다.

**방법 1: 함수 호출 사용**

```python
# Step 1: Define functions for each LLM provider
# OpenAI format
openai_tools = [
    {
        "type": "function",
        "function": {
            "name": "search_github",
            "description": "Search GitHub repositories",
            "parameters": {
                "type": "object",
                "properties": {
                    "query": {"type": "string", "description": "Search keywords"}
                },
                "required": ["query"]
            }
        }
    }
]

# Claude format
claude_tools = [
    {
        "name": "search_github",
        "description": "Search GitHub repositories",
        "input_schema": {  # Note: not parameters
            "type": "object",
            "properties": {
                "query": {"type": "string", "description": "Search keywords"}
            },
            "required": ["query"]
        }
    }
]

# Step 2: Implement tool functions yourself
def search_github(query):
    import requests
    response = requests.get(
        "https://api.github.com/search/repositories",
        params={"q": query}
    )
    return response.json()

# Step 3: Handle different model response formats
# OpenAI response
if response.choices[0].message.tool_calls:
    tool_call = response.choices[0].message.tool_calls[0]
    result = search_github(**json.loads(tool_call.function.arguments))

# Claude response
if response.content[0].type == "tool_use":
    tool_use = response.content[0]
    result = search_github(**tool_use.input)
```

**방법 2: MCP 사용**

```python
from hello_agents.protocols import MCPClient

# Step 1: Connect to community-provided MCP server (no need to implement yourself)
github_client = MCPClient([
    "npx", "-y", "@modelcontextprotocol/server-github"
])

fs_client = MCPClient([
    "npx", "-y", "@modelcontextprotocol/server-filesystem", "."
])

# Step 2: Unified calling method (model-independent)
async with github_client:
    # Automatically discover tools
    tools = await github_client.list_tools()

    # Call tool (standardized interface)
    result = await github_client.call_tool(
        "search_repositories",
        {"query": "AI agents"}
    )

# Step 3: Any model supporting MCP can use it
# OpenAI, Claude, Llama, etc. all use the same MCP client
```

첫째, 함수 호출과 MCP는 경쟁 관계가 아니라 상호 보완 관계라는 점을 명확히 할 필요가 있습니다. 함수 호출은 모델의 고유한 지능을 반영하는 대규모 언어 모델의 핵심 기능으로, 모델이 언제 함수를 호출해야 하는지 이해하고 해당 호출 매개변수를 정확하게 생성할 수 있도록 해줍니다. 반면 MCP는 인프라 프로토콜의 역할을 수행하여 도구가 엔지니어링 수준에서 모델과 연결되는 방식에 대한 엔지니어링 문제를 해결하고 표준화된 방식으로 도구를 설명하고 호출합니다.

이해하기 위해 간단한 비유를 사용할 수 있습니다. 함수 호출은 전화를 걸 때, 상대방과 통신하는 방법, 전화를 끊을 때를 포함하여 "전화 거는 방법" 기술을 배우는 것과 같습니다. 반면 MCP는 모든 전화기가 다른 전화기에 성공적으로 전화를 걸 수 있도록 보장하는 전 세계적으로 통합된 "전화 통신 표준"입니다.

상호 보완적인 관계를 이해한 후, 다음으로 HelloAgents에서 MCP 프로토콜을 사용하는 방법을 살펴보겠습니다.

### 10.2.2 MCP 클라이언트 사용

HelloAgents는 FastMCP 2.0을 기반으로 완전한 MCP 클라이언트 기능을 구현합니다. 우리는 다양한 사용 시나리오에 맞게 비동기식 및 동기식 API를 모두 제공합니다. 대부분의 애플리케이션에서는 동시 요청과 장기 실행 작업을 더 잘 처리하는 비동기 API를 사용하는 것이 좋습니다. 아래에서는 단계별 작동 데모를 제공합니다.

**(1) MCP 서버에 연결**

MCP 클라이언트는 다양한 연결 방법을 지원하며 가장 일반적인 방법은 Stdio 모드 (communicating with local processes through standard input/output)입니다.

```python
import asyncio
from hello_agents.protocols import MCPClient

async def connect_to_server():
    # Method 1: Connect to community-provided file system server
    # npx will automatically download and run the @modelcontextprotocol/server-filesystem package
    client = MCPClient([
        "npx", "-y",
        "@modelcontextprotocol/server-filesystem",
        "."  # Specify root directory
    ])

    # Use async with to ensure connection is properly closed
    async with client:
        # Use client here
        tools = await client.list_tools()
        print(f"Available tools: {[t['name'] for t in tools]}")

    # Method 2: Connect to custom Python MCP server
    client = MCPClient(["python", "my_mcp_server.py"])
    async with client:
        # Use client...
        pass

# Run async function
asyncio.run(connect_to_server())
```

**(2) 사용 가능한 도구 검색**

성공적으로 연결한 후 첫 번째 단계는 일반적으로 서버가 제공하는 도구를 쿼리하는 것입니다.

```python
async def discover_tools():
    client = MCPClient(["npx", "-y", "@modelcontextprotocol/server-filesystem", "."])

    async with client:
        # Get all available tools
        tools = await client.list_tools()

        print(f"Server provides {len(tools)} tools:")
        for tool in tools:
            print(f"\nTool name: {tool['name']}")
            print(f"Description: {tool.get('description', 'No description')}")

            # Print parameter information
            if 'inputSchema' in tool:
                schema = tool['inputSchema']
                if 'properties' in schema:
                    print("Parameters:")
                    for param_name, param_info in schema['properties'].items():
                        param_type = param_info.get('type', 'any')
                        param_desc = param_info.get('description', '')
                        print(f"  - {param_name} ({param_type}): {param_desc}")

asyncio.run(discover_tools())

# Output example:
# Server provides 5 tools:
#
# Tool name: read_file
# Description: Read file content
# Parameters:
#   - path (string): File path
#
# Tool name: write_file
# Description: Write file content
# Parameters:
#   - path (string): File path
#   - content (string): File content
```

**(3) 통화 도구**

도구를 호출할 때 JSON 스키마를 준수하는 도구 이름과 매개변수를 제공하기만 하면 됩니다.

```python
async def use_tools():
    client = MCPClient(["npx", "-y", "@modelcontextprotocol/server-filesystem", "."])

    async with client:
        # Read file
        result = await client.call_tool("read_file", {"path": "my_README.md"})
        print(f"File content:\n{result}")

        # List directory
        result = await client.call_tool("list_directory", {"path": "."})
        print(f"Current directory files: {result}")

        # Write file
        result = await client.call_tool("write_file", {
            "path": "output.txt",
            "content": "Hello from MCP!"
        })
        print(f"Write result: {result}")

asyncio.run(use_tools())
```

참조를 위해 MCP 서비스에 전화하는 더 안전한 방법은 다음과 같습니다.

```python
async def safe_tool_call():
    client = MCPClient(["npx", "-y", "@modelcontextprotocol/server-filesystem", "."])

    async with client:
        try:
            # Try to read a potentially non-existent file
            result = await client.call_tool("read_file", {"path": "nonexistent.txt"})
            print(result)
        except Exception as e:
            print(f"Tool call failed: {e}")
            # Can choose to retry, use default value, or report error to user

asyncio.run(safe_tool_call())
```

**(4) 리소스 액세스**

도구 외에도 MCP 서버는 다음과 같은 리소스도 제공할 수 있습니다.

```python
# List available resources
resources = client.list_resources()
print(f"Available resources: {[r['uri'] for r in resources]}")

# Read resource
resource_content = client.read_resource("file:///path/to/resource")
print(f"Resource content: {resource_content}")
```

**(5) 프롬프트 템플릿 사용**

MCP 서버는 사전 정의된 프롬프트 템플릿을 제공할 수 있습니다.

```python
# List available prompts
prompts = client.list_prompts()
print(f"Available prompts: {[p['name'] for p in prompts]}")

# Get prompt content
prompt = client.get_prompt("code_review", {"language": "python"})
print(f"Prompt content: {prompt}")
```

**(6) 전체 예: GitHub MCP 서비스 사용**

캡슐화된 MCP 도구를 사용하여 전체 예제를 통해 커뮤니티에서 제공하는 GitHub MCP 서비스를 사용하는 방법을 살펴보겠습니다.

```python
"""
GitHub MCP Service Example

Note: Need to set environment variable
    Windows: $env:GITHUB_PERSONAL_ACCESS_TOKEN="your_token_here"
    Linux/macOS: export GITHUB_PERSONAL_ACCESS_TOKEN="your_token_here"
"""

from hello_agents.tools import MCPTool

# Create GitHub MCP tool
github_tool = MCPTool(
    server_command=["npx", "-y", "@modelcontextprotocol/server-github"]
)

# 1. List available tools
print("📋 Available tools:")
result = github_tool.run({"action": "list_tools"})
print(result)

# 2. Search repositories
print("\n🔍 Search repositories:")
result = github_tool.run({
    "action": "call_tool",
    "tool_name": "search_repositories",
    "arguments": {
        "query": "AI agents language:python",
        "page": 1,
        "perPage": 3
    }
})
print(result)

```

### 10.2.3 MCP 전송 방법 설명

MCP 프로토콜의 중요한 특징은 **전송 불가지론**입니다. 이는 MCP 프로토콜 자체가 특정 전송 방법에 의존하지 않으며 다른 통신 채널에서 실행될 수 있음을 의미합니다. FastMCP 2.0을 기반으로 하는 HelloAgents는 완벽한 전송 방법 지원을 제공하므로 실제 시나리오에 따라 가장 적합한 전송 모드를 선택할 수 있습니다.

**(1) 운송 방법 개요**

HelloAgents' `MCPClient`는 표 10.4에 표시된 것처럼 각각 서로 다른 사용 사례를 갖는 5가지 전송 방법을 지원합니다.

<div align="center">
<p>표 10.4 MCP 전송 방법 비교</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-4.png" alt="" width="85%"/>
</div>

**(2) 운송 방법 사용 예**

```python
from hello_agents.tools import MCPTool

# 1. Memory Transport - Memory transport (for testing)
# No parameters specified, uses built-in demo server
mcp_tool = MCPTool()

# 2. Stdio Transport - Standard input/output transport (local development)
# Use command list to start local server
mcp_tool = MCPTool(server_command=["python", "examples/mcp_example_server.py"])

# 3. Stdio Transport with Args - Command transport with parameters
# Can pass additional parameters
mcp_tool = MCPTool(server_command=["python", "examples/mcp_example_server.py", "--debug"])

# 4. Stdio Transport - Community server (npx method)
# Use npx to start community MCP server
mcp_tool = MCPTool(server_command=["npx", "-y", "@modelcontextprotocol/server-filesystem", "."])

# 5. HTTP/SSE/StreamableHTTP Transport
# Note: MCPTool is mainly for Stdio and Memory transport
# For HTTP/SSE and other remote transports, recommend using MCPClient directly
```

**(3) 메모리 전송**

사용 사례: 단위 테스트, 신속한 프로토타이핑

```python
from hello_agents.tools import MCPTool

# Use built-in demo server (Memory transport)
mcp_tool = MCPTool()

# List available tools
result = mcp_tool.run({"action": "list_tools"})
print(result)

# Call tool
result = mcp_tool.run({
    "action": "call_tool",
    "tool_name": "add",
    "arguments": {"a": 10, "b": 20}
})
print(result)
```

**(4) Stdio 전송 - 표준 입력/출력 전송**

사용 사례: 로컬 개발, 디버깅, Python 스크립트 서버

```python
from hello_agents.tools import MCPTool

# Method 1: Use custom Python server
mcp_tool = MCPTool(server_command=["python", "my_mcp_server.py"])

# Method 2: Use community server (file system)
mcp_tool = MCPTool(server_command=["npx", "-y", "@modelcontextprotocol/server-filesystem", "."])

# List tools
result = mcp_tool.run({"action": "list_tools"})
print(result)

# Call tool
result = mcp_tool.run({
    "action": "call_tool",
    "tool_name": "read_file",
    "arguments": {"path": "README.md"}
})
print(result)
```

**(5) HTTP 전송**

사용 사례: 프로덕션 환경, 원격 서비스, 마이크로서비스 아키텍처

```python
# Note: MCPTool is mainly for Stdio and Memory transport
# For HTTP/SSE and other remote transports, recommend using underlying MCPClient

import asyncio
from hello_agents.protocols import MCPClient

async def test_http_transport():
    # Connect to remote HTTP MCP server
    client = MCPClient("http://api.example.com/mcp")

    async with client:
        # Get server information
        tools = await client.list_tools()
        print(f"Remote server tools: {len(tools)} tools")

        # Call remote tool
        result = await client.call_tool("process_data", {
            "data": "Hello, World!",
            "operation": "uppercase"
        })
        print(f"Remote processing result: {result}")

# Note: Requires actual HTTP MCP server
# asyncio.run(test_http_transport())
```

**(6) SSE 전송 - 서버에서 보낸 이벤트 전송**

사용 사례: 실시간 통신, 스트리밍 처리, 긴 연결

```python
# Note: MCPTool is mainly for Stdio and Memory transport
# For SSE transport, recommend using underlying MCPClient

import asyncio
from hello_agents.protocols import MCPClient

async def test_sse_transport():
    # Connect to SSE MCP server
    client = MCPClient(
        "http://localhost:8080/sse",
        transport_type="sse"
    )

    async with client:
        # SSE is especially suitable for streaming processing
        result = await client.call_tool("stream_process", {
            "input": "Large data processing request",
            "stream": True
        })
        print(f"Streaming processing result: {result}")

# Note: Requires MCP server supporting SSE
# asyncio.run(test_sse_transport())
```

**(7) StreamableHTTP 전송 - 스트리밍 HTTP 전송**

사용 사례: 양방향 스트리밍 통신이 필요한 HTTP 시나리오

```python
# Note: MCPTool is mainly for Stdio and Memory transport
# For StreamableHTTP transport, recommend using underlying MCPClient

import asyncio
from hello_agents.protocols import MCPClient

async def test_streamable_http_transport():
    # Connect to StreamableHTTP MCP server
    client = MCPClient(
        "http://localhost:8080/mcp",
        transport_type="streamable_http"
    )

    async with client:
        # Supports bidirectional streaming communication
        tools = await client.list_tools()
        print(f"StreamableHTTP server tools: {len(tools)} tools")

# Note: Requires MCP server supporting StreamableHTTP
# asyncio.run(test_streamable_http_transport())
```

### 10.2.4 에이전트에서 MCP 도구 사용

이전에는 MCP 클라이언트를 직접 사용하는 방법을 배웠습니다. 그러나 실제 애플리케이션에서는 수동으로 호출 코드를 작성하는 것보다 에이전트가 **자동으로** MCP 도구를 호출하도록 하는 것을 선호합니다. HelloAgents는 `MCPTool` 래퍼를 제공하여 MCP 서버가 에이전트의 도구 체인에 원활하게 통합될 수 있도록 합니다.

**(1) MCP 도구의 자동 확장 메커니즘**

HelloAgents' `MCPTool`에는 **자동 확장** 기능이 있습니다. MCP 도구를 에이전트에 추가하면 MCP 서버에서 제공하는 모든 도구가 자동으로 독립된 도구로 확장되어 에이전트가 이를 일반 도구처럼 호출할 수 있습니다.

**방법 1: 내장된 데모 서버 사용**

이전에 계산기 도구 기능을 구현했는데 여기서는 이를 MCP 서비스로 변환합니다. 이것이 가장 간단한 사용 방법입니다.

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import MCPTool

agent = SimpleAgent(name="Assistant", llm=HelloAgentsLLM())

# No configuration needed, automatically uses built-in demo server
mcp_tool = MCPTool(name="calculator")
agent.add_tool(mcp_tool)
# ✅ MCP tool 'calculator' expanded into 6 independent tools

# Agent can directly use expanded tools
response = agent.run("Calculate 25 times 16")
print(response)  # Output: The result of 25 times 16 is 400
```

**자동 확장 후 도구**:

- `calculator_add` - 덧셈 계산기
- `calculator_subtract` - 뺄셈 계산기
- `calculator_multiply` - 곱셈 계산기
- `calculator_divide` - 나눗셈 계산기
- `calculator_greet` - 친근한 인사
- `calculator_get_system_info` - 시스템 정보 가져오기

에이전트가 호출할 때 매개변수(예: `[TOOL_CALL:calculator_multiply:a=25,b=16]`)만 제공하면 시스템이 자동으로 유형 변환 및 MCP 호출을 처리합니다.

**방법 2: 외부 MCP 서버에 연결**

실제 프로젝트에서는 더욱 강력한 MCP 서버에 연결해야 합니다. 이러한 서버는 다음과 같습니다.
- **커뮤니티에서 제공하는 공식 서버** (such as file system, GitHub, database, etc.)
- **직접 작성하는 사용자 정의 서버**(비즈니스 로직 캡슐화)

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import MCPTool

agent = SimpleAgent(name="File Assistant", llm=HelloAgentsLLM())

# Example 1: Connect to community-provided file system server
fs_tool = MCPTool(
    name="filesystem",  # Specify unique name
    description="Access local file system",
    server_command=["npx", "-y", "@modelcontextprotocol/server-filesystem", "."]
)
agent.add_tool(fs_tool)

# Example 2: Connect to custom Python MCP server
# For how to write custom MCP servers, refer to Section 10.5
custom_tool = MCPTool(
    name="custom_server",  # Use different name
    description="Custom business logic server",
    server_command=["python", "my_mcp_server.py"]
)
agent.add_tool(custom_tool)

# Agent can now automatically use these tools!
response = agent.run("Please read the my_README.md file and summarize its main content")
print(response)
```

여러 MCP 서버를 사용하는 경우 각 MCPTool에 대해 다른 이름을 지정해야 합니다. 이 이름은 충돌을 피하기 위해 확장된 도구 이름에 접두사로 추가됩니다. 예: `name="fs"`는 `fs_read_file`, `fs_write_file` 등으로 확장됩니다. 특정 비즈니스 로직을 캡슐화하기 위해 자체 MCP 서버를 작성해야 하는 경우 섹션 10.5를 참조하세요.

**(2) MCP 도구 자동 확장 작동 방식**

자동 확장 메커니즘을 이해하면 MCP 도구를 더 잘 사용하는 데 도움이 됩니다. 작동 방식을 살펴보겠습니다.

```python
# User code
fs_tool = MCPTool(name="fs", server_command=[...])
agent.add_tool(fs_tool)

# What happens internally:
# 1. MCPTool connects to server, discovers 14 tools
# 2. Creates wrapper for each tool:
#    - fs_read_text_file (parameters: path, tail, head)
#    - fs_write_file (parameters: path, content)
#    - ...
# 3. Registers to Agent's tool registry

# Agent call
response = agent.run("Read README.md")

# Inside Agent:
# 1. Identifies need to call fs_read_text_file
# 2. Generates parameters: path=README.md
# 3. Wrapper converts to MCP format:
#    {"action": "call_tool", "tool_name": "read_text_file", "arguments": {"path": "README.md"}}
# 4. Calls MCP server
# 5. Returns file content
```

시스템은 도구 매개변수 정의에 따라 유형을 자동으로 변환합니다.

```python
# Agent calls calculator
agent.run("Calculate 25 times 16")

# Agent generates: a=25,b=16 (string)
# System automatically converts to: {"a": 25.0, "b": 16.0} (number)
# MCP server receives correct number type
```

**(3) 실제 사례: 지능형 문서 도우미**

완전한 지능형 문서 도우미를 만들어 보겠습니다. 여기서는 간단한 다중 에이전트 오케스트레이션을 보여줍니다.

```python
"""
Multi-Agent Collaborative Intelligent Document Assistant

Uses two SimpleAgents for division of labor:
- Agent1: GitHub search expert
- Agent2: Document generation expert
"""
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import MCPTool
from dotenv import load_dotenv

# Load environment variables from .env file
load_dotenv(dotenv_path="../HelloAgents/.env")

print("="*70)
print("Multi-Agent Collaborative Intelligent Document Assistant")
print("="*70)

# ============================================================
# Agent 1: GitHub Search Expert
# ============================================================
print("\n[Step 1] Creating GitHub search expert...")

github_searcher = SimpleAgent(
    name="GitHub Search Expert",
    llm=HelloAgentsLLM(),
    system_prompt="""You are a GitHub search expert.
Your task is to search GitHub repositories and return results.
Please return clear, structured search results, including:
- Repository name
- Brief description

Keep it concise, don't add extra explanations."""
)

# Add GitHub tool
github_tool = MCPTool(
    name="gh",
    server_command=["npx", "-y", "@modelcontextprotocol/server-github"]
)
github_searcher.add_tool(github_tool)

# ============================================================
# Agent 2: Document Generation Expert
# ============================================================
print("\n[Step 2] Creating document generation expert...")

document_writer = SimpleAgent(
    name="Document Generation Expert",
    llm=HelloAgentsLLM(),
    system_prompt="""You are a document generation expert.
Your task is to generate structured Markdown reports based on provided information.

The report should include:
- Title
- Introduction
- Main content (listed in points, including project names, descriptions, etc.)
- Summary

Please output the complete Markdown format report content directly, do not use tools to save."""
)

# Add file system tool
fs_tool = MCPTool(
    name="fs",
    server_command=["npx", "-y", "@modelcontextprotocol/server-filesystem", "."]
)
document_writer.add_tool(fs_tool)

# ============================================================
# Execute Task
# ============================================================
print("\n" + "="*70)
print("Starting task execution...")
print("="*70)

try:
    # Step 1: GitHub search
    print("\n[Step 3] Agent1 searching GitHub...")
    search_task = "Search for GitHub repositories about 'AI agent', return the top 5 most relevant results"

    search_results = github_searcher.run(search_task)

    print("\nSearch results:")
    print("-" * 70)
    print(search_results)
    print("-" * 70)

    # Step 2: Generate report
    print("\n[Step 4] Agent2 generating report...")
    report_task = f"""
Based on the following GitHub search results, generate a Markdown format research report:

{search_results}

Report requirements:
1. Title: # AI Agent Framework Research Report
2. Introduction: Explain this is a GitHub project survey about AI Agents
3. Main findings: List found projects and their features (including names, descriptions, etc.)
4. Summary: Summarize common characteristics of these projects

Please output the complete Markdown format report directly.
"""

    report_content = document_writer.run(report_task)

    print("\nReport content:")
    print("=" * 70)
    print(report_content)
    print("=" * 70)

    # Step 3: Save report
    print("\n[Step 5] Saving report to file...")
    import os
    try:
        with open("report.md", "w", encoding="utf-8") as f:
            f.write(report_content)
        print("✅ Report saved to report.md")

        # Verify file
        file_size = os.path.getsize("report.md")
        print(f"✅ File size: {file_size} bytes")
    except Exception as e:
        print(f"❌ Save failed: {e}")

    print("\n" + "="*70)
    print("Task completed!")
    print("="*70)

except Exception as e:
    print(f"\n❌ Error: {e}")
    import traceback
    traceback.print_exc()

```

`github_searcher`은 이 과정에서 `gh_search_repositories`을 호출하여 GitHub 프로젝트를 검색합니다. 획득된 결과는 입력으로 `document_writer`에 반환되어 보고서 생성을 추가로 안내하고 최종적으로 보고서를 report.md에 저장합니다.

### 10.2.5 MCP 커뮤니티 생태계

MCP 프로토콜의 가장 큰 장점은 **풍부한 커뮤니티 생태계**입니다. Anthropic 및 커뮤니티 개발자는 파일 시스템, 데이터베이스, API 서비스 등과 같은 다양한 시나리오를 다루는 다수의 기성 MCP 서버를 만들었습니다. 이는 도구 어댑터를 처음부터 작성할 필요가 없으며 이러한 검증된 서버를 직접 사용할 수 있음을 의미합니다.

MCP 커뮤니티를 위한 세 가지 리소스 저장소는 다음과 같습니다.

1. **멋진 MCP 서버** (https://github.com/punkpeye/awesome-mcp-servers)
   - 커뮤니티에서 관리하는 선별된 MCP 서버 목록
   - 다양한 타사 서버 포함
   - 기능별로 분류되어 있어 찾기 쉽습니다.

2. **MCP 서버 웹사이트** (https://mcpservers.org/)
   - 공식 MCP 서버 디렉토리 웹사이트
   - 검색 및 필터링 기능 제공
   - 사용 지침 및 예제가 포함되어 있습니다.

3. **공식 MCP 서버** (https://github.com/modelcontextprotocol/servers)
   - Anthropic이 공식적으로 관리하는 서버
   - 최고 품질, 가장 완벽한 문서
   - 일반적으로 사용되는 서비스의 구현을 포함합니다.

표 10.5 및 10.6은 일반적으로 사용되는 공식 MCP 서버와 인기 있는 커뮤니티 MCP 서버를 보여줍니다.

<div align="center">
<p>표 10.5 일반적으로 사용되는 공식 MCP 서버</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-5.png" alt="" width="85%"/>
</div>

<div align="center">
<p>표 10.6 인기 있는 커뮤니티 MCP 서버</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-6.png" alt="" width="85%"/>
</div>

다음은 참고할 수 있는 특히 흥미로운 TODO 사례입니다.

1. **자동화된 웹 테스팅(극작가)**

   ```python
   # Agent can automatically:
   # - Open browser to visit website
   # - Fill forms and submit
   # - Screenshot to verify results
   # - Generate test reports
   playwright_tool = MCPTool(
       name="playwright",
       server_command=["npx", "-y", "@playwright/mcp"]
   )
   ```

2. **지능형 노트 길잡이(흑요석 + Perplexity)**
   ```python
   # Agent can:
   # - Search latest tech news (Perplexity)
   # - Organize into structured notes
   # - Save to Obsidian knowledge base
   # - Automatically establish links between notes
   ```

3. **프로젝트 관리 자동화(Jira + GitHub)**
   ```python
   # Agent can:
   # - Create Jira tasks from GitHub Issues
   # - Sync code commits to Jira
   # - Automatically update Sprint progress
   # - Generate project reports
   ```

5. **콘텐츠 제작 워크플로(YouTube + Notion + Spotify)**

   ```python
   # Agent can:
   # - Get YouTube video subtitles
   # - Generate content summaries
   # - Save to Notion database
   # - Play background music (Spotify)
   ```

이 섹션의 설명을 통해 더 많은 MCP 구현 사례를 탐색할 수 있기를 바라며 HelloAgents에 대한 기여를 환영합니다! 다음으로 A2A 프로토콜에 대해 알아보겠습니다.

## 10.3 실제 A2A 프로토콜

A2A(Agent-to-Agent)는 에이전트 간 직접 통신 및 협업을 지원하는 프로토콜입니다.

10.3.1 프로토콜 설계 동기

MCP 프로토콜은 에이전트와 도구 간의 상호 작용을 해결하고 A2A 프로토콜은 에이전트 간의 협업 문제를 해결합니다. 연구자, 작가, 편집자 등 다중 에이전트 협업이 필요한 작업에서는 의사소통, 작업 위임, 기능 협상 및 상태 동기화가 필요합니다.

기존 중앙 조정자(스타 토폴로지) 솔루션에는 세 가지 주요 문제가 있습니다.

- **단일 장애 지점**: 코디네이터 장애로 인해 전체 시스템이 마비됩니다.
- **성능 병목 현상**: 모든 통신은 중앙 노드를 통과하므로 동시성이 제한됩니다.
- **확장 어려움**: 에이전트를 추가하거나 수정하려면 중앙 논리를 변경해야 합니다.

A2A 프로토콜은 P2P(Peer-to-Peer) 아키텍처(메시 토폴로지)를 채택하여 에이전트가 직접 통신할 수 있도록 하여 위의 문제를 근본적으로 해결합니다. 그 핵심은 **Task**와 **Artifact**라는 두 가지 추상적 개념인데, 이는 표 10.7에서 볼 수 있듯이 MCP와의 가장 큰 차이점입니다.

<div align="center">
<p>표 10.7 A2A 핵심 개념</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-7.png" alt="" width="85%"/>
</div>

협업 프로세스 관리를 구현하기 위해 A2A는 그림 10.7과 같이 생성, 협상, 위임, 진행 중, 완료 및 실패와 같은 상태를 포함하는 작업에 대한 표준화된 수명 주기를 정의합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-7.png" alt="" width="85%"/>
<p>그림 10.7 A2A 작업 수명주기</p>
</div>


이 메커니즘을 통해 에이전트는 작업 협상, 진행 상황 추적 및 예외 처리를 수행할 수 있습니다.

A2A 요청 수명 주기는 요청이 따르는 네 가지 주요 단계(에이전트 검색, 인증, 메시지 보내기 API, 메시지 스트림 API 보내기)를 자세히 설명하는 시퀀스입니다. 공식 웹 사이트의 흐름도에서 가져온 아래 그림 10.8은 클라이언트, A2A 서버 및 인증 서버 간의 상호 작용을 보여주는 작업 흐름을 보여줍니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-8.png" alt="" width="85%"/>
<p>그림 10.8 A2A 요청 수명 주기</p>
</div>

### 10.3.2 실제 A2A 프로토콜

대부분의 기존 A2A 구현은 `Sample Code`이며 Python 구현조차도 상당히 번거롭습니다. 따라서 여기서는 A2A-SDK를 통해 부분적인 기능을 구현하면서 프로토콜의 아이디어를 시뮬레이션하는 방법만 채택합니다.

**(2) 간단한 A2A 에이전트 생성**

다시 계산기 사례를 데모로 사용하여 A2A 에이전트를 만들어 보겠습니다.

```python
from hello_agents.protocols.a2a.implementation import A2AServer, A2A_AVAILABLE

def create_calculator_agent():
    """Create a calculator agent"""
    if not A2A_AVAILABLE:
        print("❌ A2A SDK not installed, please run: pip install a2a-sdk")
        return None

    print("🧮 Creating calculator agent")

    # Create A2A server
    calculator = A2AServer(
        name="calculator-agent",
        description="Professional mathematical calculation agent",
        version="1.0.0",
        capabilities={
            "math": ["addition", "subtraction", "multiplication", "division"],
            "advanced": ["power", "sqrt", "factorial"]
        }
    )

    # Add basic calculation skills
    @calculator.skill("add")
    def add_numbers(query: str) -> str:
        """Addition calculation"""
        try:
            # Simple parsing of "calculate 5 + 3" format
            parts = query.replace("calculate", "").replace("plus", "+").replace("add", "+")
            if "+" in parts:
                numbers = [float(x.strip()) for x in parts.split("+")]
                result = sum(numbers)
                return f"Calculation result: {' + '.join(map(str, numbers))} = {result}"
            else:
                return "Please use format: calculate 5 + 3"
        except Exception as e:
            return f"Calculation error: {e}"

    @calculator.skill("multiply")
    def multiply_numbers(query: str) -> str:
        """Multiplication calculation"""
        try:
            parts = query.replace("calculate", "").replace("times", "*").replace("×", "*")
            if "*" in parts:
                numbers = [float(x.strip()) for x in parts.split("*")]
                result = 1
                for num in numbers:
                    result *= num
                return f"Calculation result: {' × '.join(map(str, numbers))} = {result}"
            else:
                return "Please use format: calculate 5 * 3"
        except Exception as e:
            return f"Calculation error: {e}"

    @calculator.skill("info")
    def get_info(query: str) -> str:
        """Get agent information"""
        return f"I am {calculator.name}, can perform basic mathematical calculations. Supported skills: {list(calculator.skills.keys())}"

    print(f"✅ Calculator agent created successfully, supported skills: {list(calculator.skills.keys())}")
    return calculator

# Create agent
calc_agent = create_calculator_agent()
if calc_agent:
    # Test skills
    print("\n🧪 Testing agent skills:")
    test_queries = [
        "Get information",
        "Calculate 10 + 5",
        "Calculate 6 * 7"
    ]

    for query in test_queries:
        if "information" in query.lower():
            result = calc_agent.skills["info"](query)
        elif "+" in query:
            result = calc_agent.skills["add"](query)
        elif "*" in query or "×" in query:
            result = calc_agent.skills["multiply"](query)
        else:
            result = "Unknown query type"

        print(f"  📝 Query: {query}")
        print(f"  🤖 Reply: {result}")
        print()
```

**(2) 사용자 정의 A2A 에이전트**

자신만의 A2A 에이전트를 생성할 수도 있습니다. 간단한 데모는 다음과 같습니다.

```python
from hello_agents.protocols.a2a.implementation import A2AServer, A2A_AVAILABLE

def create_custom_agent():
    """Create custom agent"""
    if not A2A_AVAILABLE:
        print("Please install A2A SDK first: pip install a2a-sdk")
        return None

    # Create agent
    agent = A2AServer(
        name="my-custom-agent",
        description="My custom agent",
        capabilities={"custom": ["skill1", "skill2"]}
    )

    # Add skills
    @agent.skill("greet")
    def greet_user(name: str) -> str:
        """Greet user"""
        return f"Hello, {name}! I am a custom agent."

    @agent.skill("calculate")
    def simple_calculate(expression: str) -> str:
        """Simple calculation"""
        try:
            # Safe calculation (only supports basic operations)
            allowed_chars = set('0123456789+-*/(). ')
            if all(c in allowed_chars for c in expression):
                result = eval(expression)
                return f"Calculation result: {expression} = {result}"
            else:
                return "Error: Only basic mathematical operations supported"
        except Exception as e:
            return f"Calculation error: {e}"

    return agent

# Create and test custom agent
custom_agent = create_custom_agent()
if custom_agent:
    # Test skills
    print("Testing greeting skill:")
    result1 = custom_agent.skills["greet"]("Zhang San")
    print(result1)

    print("\nTesting calculation skill:")
    result2 = custom_agent.skills["calculate"]("10 + 5 * 2")
    print(result2)
```

### 10.3.3 HelloAgents A2A 도구 사용

HelloAgents는 통합된 A2A 도구 인터페이스를 제공합니다.

**(1) A2A 에이전트 서버 생성**

먼저 에이전트 서버를 생성해 보겠습니다.

```python
from hello_agents.protocols import A2AServer
import threading
import time

# Create researcher Agent service
researcher = A2AServer(
    name="researcher",
    description="Agent responsible for searching and analyzing materials",
    version="1.0.0"
)

# Define skills
@researcher.skill("research")
def handle_research(text: str) -> str:
    """Handle research requests"""
    import re
    match = re.search(r'research\s+(.+)', text, re.IGNORECASE)
    topic = match.group(1).strip() if match else text

    # Actual research logic (simplified here)
    result = {
        "topic": topic,
        "findings": f"Research results about {topic}...",
        "sources": ["Source 1", "Source 2", "Source 3"]
    }
    return str(result)

# Start service in background
def start_server():
    researcher.run(host="localhost", port=5000)

if __name__ == "__main__":
    server_thread = threading.Thread(target=start_server, daemon=True)
    server_thread.start()

    print("✅ Researcher Agent service started at http://localhost:5000")

    # Keep program running
    try:
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        print("\nService stopped")
```

**(2) A2A 에이전트 클라이언트 생성**

이제 서버와 통신할 클라이언트를 만들어 보겠습니다.

```python
from hello_agents.protocols import A2AClient

# Create client to connect to researcher Agent
client = A2AClient("http://localhost:5000")

# Send research request
response = client.execute_skill("research", "research AI applications in healthcare")
print(f"Received response: {response.get('result')}")

# Output:
# Received response: {'topic': 'AI applications in healthcare', 'findings': 'Research results about AI applications in healthcare...', 'sources': ['Source 1', 'Source 2', 'Source 3']}
```

**(3) 에이전트 네트워크 생성**

여러 에이전트 간의 협업을 위해 여러 에이전트를 서로 연결할 수 있습니다.

```python
from hello_agents.protocols import A2AServer, A2AClient
import threading
import time

# 1. Create multiple Agent services
researcher = A2AServer(
    name="researcher",
    description="Researcher"
)

@researcher.skill("research")
def do_research(text: str) -> str:
    import re
    match = re.search(r'research\s+(.+)', text, re.IGNORECASE)
    topic = match.group(1).strip() if match else text
    return str({"topic": topic, "findings": f"Research results for {topic}"})

writer = A2AServer(
    name="writer",
    description="Writer"
)

@writer.skill("write")
def write_article(text: str) -> str:
    import re
    match = re.search(r'write\s+(.+)', text, re.IGNORECASE)
    content = match.group(1).strip() if match else text

    # Try to parse research data
    try:
        data = eval(content)
        topic = data.get("topic", "Unknown topic")
        findings = data.get("findings", "No research results")
    except:
        topic = "Unknown topic"
        findings = content

    return f"# {topic}\n\nBased on research: {findings}\n\nArticle content..."

editor = A2AServer(
    name="editor",
    description="Editor"
)

@editor.skill("edit")
def edit_article(text: str) -> str:
    import re
    match = re.search(r'edit\s+(.+)', text, re.IGNORECASE)
    article = match.group(1).strip() if match else text

    result = {
        "article": article + "\n\n[Edited and optimized]",
        "feedback": "Article quality is good",
        "approved": True
    }
    return str(result)

# 2. Start all services
threading.Thread(target=lambda: researcher.run(port=5000), daemon=True).start()
threading.Thread(target=lambda: writer.run(port=5001), daemon=True).start()
threading.Thread(target=lambda: editor.run(port=5002), daemon=True).start()
time.sleep(2)  # Wait for services to start

# 3. Create clients to connect to each Agent
researcher_client = A2AClient("http://localhost:5000")
writer_client = A2AClient("http://localhost:5001")
editor_client = A2AClient("http://localhost:5002")

# 4. Collaboration workflow
def create_content(topic):
    # Step 1: Research
    research = researcher_client.execute_skill("research", f"research {topic}")
    research_data = research.get('result', '')

    # Step 2: Write
    article = writer_client.execute_skill("write", f"write {research_data}")
    article_content = article.get('result', '')

    # Step 3: Edit
    final = editor_client.execute_skill("edit", f"edit {article_content}")
    return final.get('result', '')

# Usage
result = create_content("AI applications in healthcare")
print(f"\nFinal result:\n{result}")
```

### 10.3.4 에이전트에서 A2A 도구 사용

이제 A2A를 HelloAgents 에이전트에 통합하는 방법을 살펴보겠습니다.

**(1) A2ATool 래퍼 사용**

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import A2ATool
from dotenv import load_dotenv

load_dotenv()
llm = HelloAgentsLLM()

# Assume a researcher Agent service is already running at http://localhost:5000

# Create coordinator Agent
coordinator = SimpleAgent(name="Coordinator", llm=llm)

# Add A2A tool, connect to researcher Agent
researcher_tool = A2ATool(
    name="researcher",
    description="Researcher Agent, can search and analyze materials",
    agent_url="http://localhost:5000"
)
coordinator.add_tool(researcher_tool)

# Coordinator can call researcher Agent
response = coordinator.run("Please have the researcher help me research AI applications in education")
print(response)
```

**(2) 실제 사례: 지능형 고객 서비스 시스템**

세 명의 에이전트를 사용하여 완전한 지능형 고객 서비스 시스템을 구축해 보겠습니다.
- **접수원**: 고객의 질문 유형을 분석합니다.
- **기술 전문가**: 기술적인 질문에 답변합니다.
- **영업 컨설턴트**: 영업 관련 질문에 답변해 드립니다.

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import A2ATool
from hello_agents.protocols import A2AServer
import threading
import time
from dotenv import load_dotenv

load_dotenv()
llm = HelloAgentsLLM()

# 1. Create technical expert Agent service
tech_expert = A2AServer(
    name="tech_expert",
    description="Technical expert, answers technical questions"
)

@tech_expert.skill("answer")
def answer_tech_question(text: str) -> str:
    import re
    match = re.search(r'answer\s+(.+)', text, re.IGNORECASE)
    question = match.group(1).strip() if match else text
    # In actual applications, this would call LLM or knowledge base
    return f"Technical answer: Regarding '{question}', I suggest you check our technical documentation..."

# 2. Create sales consultant Agent service
sales_advisor = A2AServer(
    name="sales_advisor",
    description="Sales consultant, answers sales questions"
)

@sales_advisor.skill("answer")
def answer_sales_question(text: str) -> str:
    import re
    match = re.search(r'answer\s+(.+)', text, re.IGNORECASE)
    question = match.group(1).strip() if match else text
    return f"Sales answer: Regarding '{question}', we have special offers..."

# 3. Start services
threading.Thread(target=lambda: tech_expert.run(port=6000), daemon=True).start()
threading.Thread(target=lambda: sales_advisor.run(port=6001), daemon=True).start()
time.sleep(2)

# 4. Create receptionist Agent (using HelloAgents' SimpleAgent)
receptionist = SimpleAgent(
    name="Receptionist",
    llm=llm,
    system_prompt="""You are a customer service receptionist, responsible for:
1. Analyzing customer question types (technical questions or sales questions)
2. Forwarding questions to appropriate experts
3. Organizing expert answers and returning them to customers

Please remain polite and professional."""
)

# Add technical expert tool
tech_tool = A2ATool(
    agent_url="http://localhost:6000",
    name="tech_expert",
    description="Technical expert, answers technical-related questions"
)
receptionist.add_tool(tech_tool)

# Add sales consultant tool
sales_tool = A2ATool(
    agent_url="http://localhost:6001",
    name="sales_advisor",
    description="Sales consultant, answers price and purchase-related questions"
)
receptionist.add_tool(sales_tool)

# 5. Handle customer inquiries
def handle_customer_query(query):
    print(f"\nCustomer inquiry: {query}")
    print("=" * 50)
    response = receptionist.run(query)
    print(f"\nCustomer service reply: {response}")
    print("=" * 50)

# Test different types of questions
if __name__ == "__main__":
    handle_customer_query("How do I call your API?")
    handle_customer_query("What is the price of the enterprise version?")
    handle_customer_query("How do I integrate it into my Python project?")
```

**(3) 고급 사용법: 에이전트 협상**

A2A 프로토콜은 에이전트 간의 협상 메커니즘도 지원합니다.

```python
from hello_agents.protocols import A2AServer, A2AClient
import threading
import time

# Create two Agents that need to negotiate
agent1 = A2AServer(
    name="agent1",
    description="Agent 1"
)

@agent1.skill("propose")
def handle_proposal(text: str) -> str:
    """Handle negotiation proposals"""
    import re

    # Parse proposal
    match = re.search(r'propose\s+(.+)', text, re.IGNORECASE)
    proposal_str = match.group(1).strip() if match else text

    try:
        proposal = eval(proposal_str)
        task = proposal.get("task")
        deadline = proposal.get("deadline")

        # Evaluate proposal
        if deadline >= 7:  # Need at least 7 days
            result = {"accepted": True, "message": "Proposal accepted"}
        else:
            result = {
                "accepted": False,
                "message": "Timeline too tight",
                "counter_proposal": {"deadline": 7}
            }
        return str(result)
    except:
        return str({"accepted": False, "message": "Invalid proposal format"})

agent2 = A2AServer(
    name="agent2",
    description="Agent 2"
)

@agent2.skill("negotiate")
def negotiate_task(text: str) -> str:
    """Initiate negotiation"""
    import re

    # Parse task and deadline
    match = re.search(r'negotiate\s+task:(.+?)\s+deadline:(\d+)', text, re.IGNORECASE)
    if match:
        task = match.group(1).strip()
        deadline = int(match.group(2))

        # Send proposal to agent1
        proposal = {"task": task, "deadline": deadline}
        return str({"status": "negotiating", "proposal": proposal})
    else:
        return str({"status": "error", "message": "Invalid negotiation request"})

# Start services
threading.Thread(target=lambda: agent1.run(port=7000), daemon=True).start()
threading.Thread(target=lambda: agent2.run(port=7001), daemon=True).start()
```

## 10.4 실제 ANP 프로토콜

MCP 프로토콜이 도구 호출을 해결하고 A2A 프로토콜이 P2P 에이전트 협업을 해결한 후 ANP 프로토콜은 대규모 개방형 네트워크 환경에서 에이전트 관리 문제를 해결하는 데 중점을 둡니다.

섹션 10.2 및 10.3에서는 MCP(도구 액세스) 및 A2A(에이전트 협업)에 대해 배웠습니다. 이제 **대규모 개방형 에이전트 네트워크** 구축에 중점을 둔 ANP(Agent Network Protocol) 프로토콜에 대해 알아 보겠습니다.

### 10.4.1 프로토콜 목표

네트워크에 다양한 기능((e.g., natural language processing, image recognition, data analysis, etc.))을 가진 다수의 에이전트가 포함되어 있는 경우 시스템은 다음과 같은 일련의 문제에 직면합니다.

- **서비스 검색**: 새 작업이 도착하면 해당 작업을 처리할 수 있는 에이전트를 빠르게 찾는 방법은 무엇입니까?
- **지능형 라우팅**: 여러 에이전트가 동일한 작업을 처리할 수 있는 경우 가장 적합한 에이전트 (e.g., based on load, cost, etc.)를 선택하고 해당 에이전트에 작업을 파견하는 방법은 무엇입니까?
- **동적 확장**: 새로 합류한 에이전트를 다른 구성원이 검색하고 호출할 수 있게 만드는 방법은 무엇입니까?

ANP의 설계 목표는 위의 서비스 검색, 라우팅 선택 및 네트워크 확장성 문제를 해결하기 위한 표준화된 메커니즘을 제공하는 것입니다.

설계 목표를 달성하기 위해 ANP는 표 10.8에 표시된 대로 다음 핵심 개념을 정의합니다.

<div align="center">
<p>표 10.8 ANP 핵심 개념</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-8.png" alt="" width="85%"/>
</div>

또한 그림 10.9와 같이 ANP의 아키텍처 설계를 소개하기 위해 공식 [시작 가이드](https://github.com/agent-network-protocol/AgentNetworkProtocol/blob/main/docs/chinese/ANP入门指南.md))를 빌려왔습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-9.png" alt="" width="85%"/>
<p>그림 10.9 ANP 전체 프로세스</p>
</div>


이 순서도의 주요 단계는 다음과 같습니다.

**1. 서비스 검색 및 일치:** 먼저 에이전트 A는 공개 검색 서비스를 사용하여 의미론적 또는 기능적 설명을 기반으로 쿼리하여 작업 요구 사항을 충족하는 에이전트 B를 찾습니다. 디스커버리 서비스는 각 에이전트가 노출한 표준 엔드포인트(`.well-known/agent-descriptions`)를 사전 크롤링하여 인덱스를 구축함으로써 서비스 수요자와 제공자 간의 동적 매칭을 실현합니다.

**2. DID 기반 신원 확인:** 상호 작용이 시작될 때 에이전트 A는 개인 키를 사용하여 자신의 DID가 포함된 요청에 서명합니다. 에이전트 B는 이를 수신한 후 DID를 구문 분석하여 해당 공개 키를 획득하고 이를 사용하여 서명의 진위와 요청의 무결성을 확인함으로써 양 당사자 간에 신뢰할 수 있는 통신을 설정합니다.

**3. 표준화된 서비스 실행:** 신원 확인이 통과된 후 에이전트 B는 요청에 응답하고 양 당사자는 사전 정의된 표준 인터페이스 및 데이터 형식에 따라 데이터를 교환하거나 서비스 (such as booking, querying, etc.)를 호출합니다. 표준화된 상호 작용 프로세스는 플랫폼 간 및 시스템 간 상호 운용성을 달성하기 위한 기반입니다.

요약하면, 이 메커니즘의 핵심은 DID를 사용하여 분산형 신뢰 기반을 구축하고 표준화된 설명 프로토콜을 활용하여 동적 서비스 검색을 달성하는 것입니다. 이 접근 방식을 통해 에이전트는 중앙 조정 없이 인터넷에서 안전하고 효율적으로 협업 네트워크를 형성할 수 있습니다.

### 10.4.2 ANP 서비스 검색 사용

**(1) 서비스 디스커버리 센터 생성**

```python
from hello_agents.protocols import ANPDiscovery, register_service

# Create service discovery center
discovery = ANPDiscovery()

# Register Agent services
register_service(
    discovery=discovery,
    service_id="nlp_agent_1",
    service_name="NLP Processing Expert A",
    service_type="nlp",
    capabilities=["text_analysis", "sentiment_analysis", "ner"],
    endpoint="http://localhost:8001",
    metadata={"load": 0.3, "price": 0.01, "version": "1.0.0"}
)

register_service(
    discovery=discovery,
    service_id="nlp_agent_2",
    service_name="NLP Processing Expert B",
    service_type="nlp",
    capabilities=["text_analysis", "translation"],
    endpoint="http://localhost:8002",
    metadata={"load": 0.7, "price": 0.02, "version": "1.1.0"}
)

print("✅ Service registration completed")
```

**(2) 서비스 검색**

```python
from hello_agents.protocols import discover_service

# Find by type
nlp_services = discover_service(discovery, service_type="nlp")
print(f"Found {len(nlp_services)} NLP services")

# Select service with lowest load
best_service = min(nlp_services, key=lambda s: s.metadata.get("load", 1.0))
print(f"Best service: {best_service.service_name} (load: {best_service.metadata['load']})")
```

**(3) 에이전트 네트워크 구축**

```python
from hello_agents.protocols import ANPNetwork

# Create network
network = ANPNetwork(network_id="ai_cluster")

# Add nodes
for service in discovery.list_all_services():
    network.add_node(service.service_id, service.endpoint)

# Establish connections (based on capability matching)
network.connect_nodes("nlp_agent_1", "nlp_agent_2")

stats = network.get_network_stats()
print(f"✅ Network construction completed, total {stats['total_nodes']} nodes")
```

### 10.4.3 실제 사례

완전한 분산 작업 스케줄링 시스템을 구축해 보겠습니다.

```python
from hello_agents.protocols import ANPDiscovery, register_service
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools.builtin import ANPTool
import random
from dotenv import load_dotenv

load_dotenv()
llm = HelloAgentsLLM()

# 1. Create service discovery center
discovery = ANPDiscovery()

# 2. Register multiple compute nodes
for i in range(10):
    register_service(
        discovery=discovery,
        service_id=f"compute_node_{i}",
        service_name=f"Compute Node {i}",
        service_type="compute",
        capabilities=["data_processing", "ml_training"],
        endpoint=f"http://node{i}:8000",
        metadata={
            "load": random.uniform(0.1, 0.9),
            "cpu_cores": random.choice([4, 8, 16]),
            "memory_gb": random.choice([16, 32, 64]),
            "gpu": random.choice([True, False])
        }
    )

print(f"✅ Registered {len(discovery.list_all_services())} compute nodes")

# 3. Create task scheduler Agent
scheduler = SimpleAgent(
    name="Task Scheduler",
    llm=llm,
    system_prompt="""You are an intelligent task scheduler, responsible for:
1. Analyzing task requirements
2. Selecting the most suitable compute node
3. Assigning tasks

When selecting nodes, consider: load, CPU cores, memory, GPU, and other factors."""
)

# Add ANP tool
anp_tool = ANPTool(
    name="service_discovery",
    description="Service discovery tool, can find and select compute nodes",
    discovery=discovery
)
scheduler.add_tool(anp_tool)

# 4. Intelligent task assignment
def assign_task(task_description):
    print(f"\nTask: {task_description}")
    print("=" * 50)

    # Let Agent intelligently select node
    response = scheduler.run(f"""
    Please select the most suitable compute node for the following task:
    {task_description}

    Requirements:
    1. List all available nodes
    2. Analyze characteristics of each node
    3. Select the most suitable node
    4. Explain selection reasoning
    """)

    print(response)
    print("=" * 50)

# Test different types of tasks
assign_task("Train a large deep learning model, requires GPU support")
assign_task("Process large amounts of text data, requires high memory")
assign_task("Run lightweight data analysis task")
```

로드 밸런싱 예시입니다

```python
from hello_agents.protocols import ANPDiscovery, register_service
import random

# Create service discovery center
discovery = ANPDiscovery()

# Register multiple services of the same type
for i in range(5):
    register_service(
        discovery=discovery,
        service_id=f"api_server_{i}",
        service_name=f"API Server {i}",
        service_type="api",
        capabilities=["rest_api"],
        endpoint=f"http://api{i}:8000",
        metadata={"load": random.uniform(0.1, 0.9)}
    )

# Load balancing function
def get_best_server():
    """Select server with lowest load"""
    servers = discovery.discover_services(service_type="api")
    if not servers:
        return None

    best = min(servers, key=lambda s: s.metadata.get("load", 1.0))
    return best

# Simulate request allocation
for i in range(10):
    server = get_best_server()
    print(f"Request {i+1} -> {server.service_name} (load: {server.metadata['load']:.2f})")

    # Update load (simulated)
    server.metadata["load"] += 0.1
```

## 10.5 사용자 정의 MCP 서버 구축

이전 섹션에서는 기존 MCP 서비스를 사용하는 방법을 배웠습니다. 또한 다양한 프로토콜의 특성에 대해서도 배웠습니다. 이제 자체 MCP 서버를 구축하는 방법을 알아 보겠습니다.

### 10.5.1 첫 번째 MCP 서버 만들기

**(1) 사용자 정의 MCP 서버를 구축하는 이유는 무엇입니까?**

공용 MCP 서비스를 직접 사용할 수 있지만 많은 실제 애플리케이션 시나리오에서는 특정 요구 사항을 충족하기 위해 사용자 지정 MCP 서버를 구축해야 합니다.

주요 동기는 다음과 같습니다.

- **비즈니스 로직 캡슐화**: 에이전트의 통합 호출을 위한 표준화된 MCP 도구로 기업별 비즈니스 프로세스 또는 복잡한 작업을 캡슐화합니다.
- **개인 데이터 액세스**: 공용 네트워크에 노출될 수 없는 내부 데이터베이스, API 또는 기타 개인 데이터 소스에 액세스하기 위한 안전하고 제어 가능한 인터페이스 또는 프록시를 만듭니다.
- **성능 최적화**: 엄격한 응답 대기 시간 요구 사항이 있는 고주파 호출 또는 애플리케이션 시나리오에 대한 심층적인 최적화를 수행합니다.
- **사용자 정의 기능 확장**: 독점 알고리즘 모델 통합, 특정 하드웨어 장치 연결 등 표준 MCP 서비스에서 제공하지 않는 특정 기능을 구현합니다.

**(2) 교육 사례: 날씨 쿼리 MCP 서버**

간단한 날씨 쿼리 서버부터 시작하여 점차적으로 MCP 서버 개발을 배워보겠습니다.

```python
#!/usr/bin/env python3
"""Weather Query MCP Server"""

import json
import requests
import os
from datetime import datetime
from typing import Dict, Any
from hello_agents.protocols import MCPServer

# Create MCP server
weather_server = MCPServer(name="weather-server", description="Real weather query service")

CITY_MAP = {
    "Beijing": "Beijing", "Shanghai": "Shanghai", "Guangzhou": "Guangzhou",
    "Shenzhen": "Shenzhen", "Hangzhou": "Hangzhou", "Chengdu": "Chengdu",
    "Chongqing": "Chongqing", "Wuhan": "Wuhan", "Xi'an": "Xi'an",
    "Nanjing": "Nanjing", "Tianjin": "Tianjin", "Suzhou": "Suzhou"
}


def get_weather_data(city: str) -> Dict[str, Any]:
    """Get weather data from wttr.in"""
    city_en = CITY_MAP.get(city, city)
    url = f"https://wttr.in/{city_en}?format=j1"
    response = requests.get(url, timeout=10)
    response.raise_for_status()
    data = response.json()
    current = data["current_condition"][0]

    return {
        "city": city,
        "temperature": float(current["temp_C"]),
        "feels_like": float(current["FeelsLikeC"]),
        "humidity": int(current["humidity"]),
        "condition": current["weatherDesc"][0]["value"],
        "wind_speed": round(float(current["windspeedKmph"]) / 3.6, 1),
        "visibility": float(current["visibility"]),
        "timestamp": datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    }


# Define tool function
def get_weather(city: str) -> str:
    """Get current weather for specified city"""
    try:
        weather_data = get_weather_data(city)
        return json.dumps(weather_data, ensure_ascii=False, indent=2)
    except Exception as e:
        return json.dumps({"error": str(e), "city": city}, ensure_ascii=False)


def list_supported_cities() -> str:
    """List all supported Chinese cities"""
    result = {"cities": list(CITY_MAP.keys()), "count": len(CITY_MAP)}
    return json.dumps(result, ensure_ascii=False, indent=2)


def get_server_info() -> str:
    """Get server information"""
    info = {
        "name": "Weather MCP Server",
        "version": "1.0.0",
        "tools": ["get_weather", "list_supported_cities", "get_server_info"]
    }
    return json.dumps(info, ensure_ascii=False, indent=2)


# Register tools to server
weather_server.add_tool(get_weather)
weather_server.add_tool(list_supported_cities)
weather_server.add_tool(get_server_info)


if __name__ == "__main__":
    weather_server.run()
```

**(3) 사용자 정의 MCP 서버 테스트**

그런 다음 테스트 스크립트를 만듭니다.

```python
#!/usr/bin/env python3
"""Test Weather Query MCP Server"""

import asyncio
import json
import sys
import os

sys.path.insert(0, os.path.join(os.path.dirname(__file__), '..', 'HelloAgents'))
from hello_agents.protocols.mcp.client import MCPClient


async def test_weather_server():
    server_script = os.path.join(os.path.dirname(__file__), "14_weather_mcp_server.py")
    client = MCPClient(["python", server_script])

    try:
        async with client:
            # Test 1: Get server information
            info = json.loads(await client.call_tool("get_server_info", {}))
            print(f"Server: {info['name']} v{info['version']}")

            # Test 2: List supported cities
            cities = json.loads(await client.call_tool("list_supported_cities", {}))
            print(f"Supported cities: {cities['count']} cities")

            # Test 3: Query Beijing weather
            weather = json.loads(await client.call_tool("get_weather", {"city": "Beijing"}))
            if "error" not in weather:
                print(f"\nBeijing weather: {weather['temperature']}°C, {weather['condition']}")

            # Test 4: Query Shenzhen weather
            weather = json.loads(await client.call_tool("get_weather", {"city": "Shenzhen"}))
            if "error" not in weather:
                print(f"Shenzhen weather: {weather['temperature']}°C, {weather['condition']}")

            print("\n✅ All tests completed!")

    except Exception as e:
        print(f"❌ Test failed: {e}")


if __name__ == "__main__":
    asyncio.run(test_weather_server())
```

**(4) 에이전트에서 사용자 지정 MCP 서버 사용**

```python
"""Using Weather MCP Server in Agent"""

import os
from dotenv import load_dotenv
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import MCPTool

load_dotenv()


def create_weather_assistant():
    """Create weather assistant"""
    llm = HelloAgentsLLM()

    assistant = SimpleAgent(
        name="Weather Assistant",
        llm=llm,
        system_prompt="""You are a weather assistant that can query city weather.
Use the get_weather tool to query weather, supports Chinese city names.
"""
    )

    # Add weather MCP tool
    server_script = os.path.join(os.path.dirname(__file__), "14_weather_mcp_server.py")
    weather_tool = MCPTool(server_command=["python", server_script])
    assistant.add_tool(weather_tool)

    return assistant


def demo():
    """Demo"""
    assistant = create_weather_assistant()

    print("\nQuery Beijing weather:")
    response = assistant.run("How's the weather in Beijing today?")
    print(f"Answer: {response}\n")


def interactive():
    """Interactive mode"""
    assistant = create_weather_assistant()

    while True:
        user_input = input("\nYou: ").strip()
        if user_input.lower() in ['quit', 'exit']:
            break
        response = assistant.run(user_input)
        print(f"Assistant: {response}")


if __name__ == "__main__":
    import sys
    if len(sys.argv) > 1 and sys.argv[1] == "demo":
        demo()
    else:
        interactive()
```

```
🔗 Connecting to MCP server...
✅ Connection successful!
🔌 Connection disconnected
✅ Tool 'mcp_get_weather' registered.
✅ Tool 'mcp_list_supported_cities' registered.
✅ Tool 'mcp_get_server_info' registered.
✅ MCP tool 'mcp' expanded into 3 independent tools

You: I want to query Beijing's weather
🔗 Connecting to MCP server...
✅ Connection successful!
🔌 Connection disconnected
Assistant: The current weather in Beijing is as follows:

- Temperature: 10.0°C
- Feels like: 9.0°C
- Humidity: 94%
- Weather condition: Light rain
- Wind speed: 1.7 m/s
- Visibility: 10.0 km
- Timestamp: October 9, 2025 13:46:40

Please bring rain gear and adjust your clothing according to weather changes.
```

### 10.5.2 MCP 서버 업로드

실제 날씨 쿼리 MCP 서버를 만들었습니다. 이제 전 세계 개발자들이 우리 서비스를 사용할 수 있도록 Smithery 플랫폼에 게시해 보겠습니다.

(1) 대장간이란 무엇입니까?

[Smithery](https://smithery.ai/)는 Python의 PyPI 또는 Node.js의 npm과 유사한 MCP 서버의 공식 게시 플랫폼입니다. Smithery를 통해 사용자는 다음을 수행할 수 있습니다.

- 🔍 MCP 서버 검색 및 검색
- 📦 한 번의 클릭으로 MCP 서버 설치
- 📊 서버 사용 통계 및 등급 보기
- 🔄 자동으로 서버 업데이트 받기

(2) 출판 준비
먼저 프로젝트를 표준 출판 형식으로 구성해야 합니다. 이 폴더는 참조용으로 `code` 디렉토리에 구성되어 있습니다.

```
weather-mcp-server/
├── README.md           # Project documentation
├── LICENSE            # Open source license
├── Dockerfile         # Docker build configuration (recommended)
├── pyproject.toml     # Python project configuration (required)
├── requirements.txt   # Python dependencies
├── smithery.yaml      # Smithery configuration file (required)
└── server.py          # MCP server main file
```

`smithery.yaml`은 Smithery 플랫폼의 구성 파일입니다.
```yaml
name: weather-mcp-server
displayName: Weather MCP Server
description: Real-time weather query MCP server based on HelloAgents framework
version: 1.0.0
author: HelloAgents Team
homepage: https://github.com/yourusername/weather-mcp-server
license: MIT
categories:
  - weather
  - data
tags:
  - weather
  - real-time
  - helloagents
  - wttr
runtime: container
build:
  dockerfile: Dockerfile
  dockerBuildPath: .
startCommand:
  type: http
tools:
  - name: get_weather
    description: Get current weather for a city
  - name: list_supported_cities
    description: List all supported cities
  - name: get_server_info
    description: Get server information
```

구성 설명:

- `name`: 서버의 고유 식별자(소문자, 하이픈으로 구분)
- `displayName`: 표시 이름
- `description`: 간략한 설명
- `version`: 버전 번호(의미론적 버전을 따름)
- `runtime`: 런타임 환경 (python/node)
- `entrypoint`: 항목 파일
- `tools`: 도구 목록

`pyproject.toml`은 Python 프로젝트의 표준 구성 파일입니다. Smithery에는 나중에 서버에 패키지될 예정이므로 이 파일이 필요합니다.

```toml
[build-system]
requires = ["setuptools>=61.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "weather-mcp-server"
version = "1.0.0"
description = "Real-time weather query MCP server based on HelloAgents framework"
readme = "README.md"
license = {text = "MIT"}
authors = [
    {name = "HelloAgents Team", email = "xxx"}
]
requires-python = ">=3.10"
dependencies = [
    "hello-agents>=0.2.1",
    "requests>=2.31.0",
]

[project.urls]
Homepage = "https://github.com/yourusername/weather-mcp-server"
Repository = "https://github.com/yourusername/weather-mcp-server"
"Bug Tracker" = "https://github.com/yourusername/weather-mcp-server/issues"

[tool.setuptools]
py-modules = ["server"]
```


구성 설명:

- `[build-system]`: 빌드 도구 지정(setuptools)
- `[project]`: 프로젝트 메타데이터
  - `name`: 프로젝트 이름
  - `version`: 버전 번호(의미론적 버전을 따름)
  - `dependencies`: 프로젝트 종속성 목록
  - `requires-python`: Python 버전 요구 사항
- `[project.urls]`: 프로젝트 관련 링크
- `[tool.setuptools]`: setuptools 구성

Smithery가 Dockerfile을 자동으로 생성하지만 사용자 정의 Dockerfile을 제공하면 성공적인 배포가 보장됩니다.

```dockerfile
# Multi-stage build for weather-mcp-server
FROM python:3.12-slim-bookworm as base

# Set working directory
WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    --no-install-recommends \
    && rm -rf /var/lib/apt/lists/*

# Copy project files
COPY pyproject.toml requirements.txt ./
COPY server.py ./

# Install Python dependencies
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt

# Set environment variables
ENV PYTHONUNBUFFERED=1
ENV PORT=8081

# Expose port (Smithery uses 8081)
EXPOSE 8081

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD python -c "import sys; sys.exit(0)"

# Run the MCP server
CMD ["python", "server.py"]
```

Dockerfile 구성 설명:

- **기본 이미지**: `python:3.12-slim-bookworm` - 경량 Python 이미지
- **작업 디렉터리**: `/app` - 애플리케이션 루트 디렉터리
- **포트**: `8081` - Smithery 플랫폼 표준 포트
- **시작 명령**: `python server.py` - MCP 서버 실행

여기서는 `hello-agents` 저장소를 포크하고, `code`에서 소스 코드를 가져온 다음, 자신의 GitHub를 사용하여 `weather-mcp-server`라는 저장소를 만들고, `yourusername`을 GitHub 사용자 이름으로 변경해야 합니다.

(3) 대장간 제출

브라우저를 열고 [https://smithery.ai/](https://smithery.ai/). GitHub 계정을 사용하여 Smithery에 로그인하세요. 페이지에서 "게시 서버" 버튼을 클릭하고 GitHub 저장소 URL: `https://github.com/yourusername/weather-mcp-server`을 입력한 후 게시를 기다립니다.

게시가 완료되면 그림 10.10과 같이 다음과 유사한 페이지를 볼 수 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-10.png" alt="" width="85%"/>
<p>그림 10.10 Smithery 출판 성공 페이지</p>
</div>



서버가 성공적으로 게시되면 사용자는 다음과 같은 방법으로 서버를 사용할 수 있습니다.

방법 1: Smithery CLI를 통해

```bash
# Install Smithery CLI
npm install -g @smithery/cli

# Install your server
smithery install weather-mcp-server
```

방법 2: Claude Desktop에서 구성

```json
{
  "mcpServers": {
    "weather": {
      "command": "smithery",
      "args": ["run", "weather-mcp-server"]
    }
  }
}
```

방법 3: HelloAgents에서 사용

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools.builtin.protocol_tools import MCPTool

agent = SimpleAgent(name="Weather Assistant", llm=HelloAgentsLLM())

# Use Smithery-installed server
weather_tool = MCPTool(
    server_command=["smithery", "run", "weather-mcp-server"]
)
agent.add_tool(weather_tool)

response = agent.run("How's the weather in Beijing today?")
```

물론 이는 단지 예일 뿐이며 직접 탐색해 볼 수 있는 사용법이 더 많이 있습니다. 아래 그림 10.11은 MCP 도구가 성공적으로 게시될 때 포함된 정보를 보여 주며 서비스 이름 "Weather", 고유 식별자 `@jjyaoao/weather-mcp-server` 및 상태 정보를 표시합니다. 도구 영역에는 방금 구현한 방법이 표시되고, 연결 영역에는 여러 언어/환경에서 서비스의 **액세스 URL 주소** 및 **구성 코드 스니펫**을 포함하여 이 서비스를 연결하고 사용하는 데 필요한 기술 정보가 제공됩니다. 더 자세히 알아보려면 이 [링크](https://smithery.ai/server/@jjyaoao/weather-mcp-server).)를 클릭하세요.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-11.png" alt="" width="85%"/>
<p>그림 10.11 Smithery에 성공적으로 게시된 MCP 도구</p>
</div>

이제 자신만의 MCP 서버를 만들 차례입니다!



## 10.6 장 요약

이 장에서는 에이전트 통신을 위한 세 가지 핵심 프로토콜인 MCP, A2A, ANP를 체계적으로 소개하고 이들의 설계 철학, 응용 시나리오 및 실제 방법을 살펴보았습니다.

**프로토콜 포지셔닝:**

- **MCP(Model Context Protocol)**: 에이전트와 도구 간의 브리지로서 개별 에이전트의 기능을 향상시키는 데 적합한 통합 도구 액세스 인터페이스를 제공합니다.
- **A2A(Agent-to-Agent Protocol)**: 에이전트 간 대화 시스템으로 직접 커뮤니케이션 및 작업 협상을 지원하므로 소규모 팀의 긴밀한 협업에 적합합니다.
- **ANP(에이전트 네트워크 프로토콜)**: 에이전트를 위한 "인터넷"으로서 대규모 개방형 에이전트 네트워크 구축에 적합한 서비스 검색, 라우팅 및 로드 밸런싱 메커니즘을 제공합니다.

**HelloAgents 통합 솔루션**

`HelloAgents` 프레임워크에서 이 세 가지 프로토콜은 도구(Tool)로 균일하게 추상화되어 원활한 통합을 달성하므로 개발자는 에이전트에 다양한 수준의 통신 기능을 유연하게 추가할 수 있습니다.

```python
# Unified Tool interface
from hello_agents.tools import MCPTool, A2ATool, ANPTool

# All protocols can be added to Agent as Tools
agent.add_tool(MCPTool(...))
agent.add_tool(A2ATool(...))
agent.add_tool(ANPTool(...))
```

**실제 경험 요약**

- 불필요한 중복 개발을 줄이기 위해 성숙한 커뮤니티 MCP 서비스를 우선적으로 사용합니다.
- 시스템 규모에 따라 적절한 프로토콜 선택: 소규모 협업 시나리오에는 A2A를 권장하고, 대규모 네트워크 시나리오에는 ANP를 사용해야 합니다.

이 장을 완료한 후 다음을 수행하는 것이 좋습니다.

1. **실습 연습**:
   - 나만의 MCP 서버 구축
   - 프로토콜을 사용하여 다중 에이전트 협업 시스템 구축
   - MCP, A2A, ANP의 조합 적용 전략
2. **심층 학습**:
   - MCP 공식 문서 읽기: https://modelcontextprotocol.io
   - A2A 공식 문서 읽기: https://a2a-protocol.org/latest/
   - ANP 공식 문서 읽기: https://agent-network-protocol.com/guide/
3. **커뮤니티에 참여**:
   - 커뮤니티에 새로운 MCP 서비스 기여
   - 자신이 개발한 Agent 구현 사례 공유
   - 관련 프로토콜에 대한 기술 표준 토론에 참여하거나 이슈에 질문하거나 HelloAgents가 새로운 사례 사례를 지원하도록 직접 지원합니다.

**10장을 완료한 것을 축하합니다!**

이제 에이전트 통신 프로토콜의 핵심 지식을 마스터했습니다. 계속해서 좋은 일을 하세요! 🚀

## 연습

> **참고**: 일부 연습에는 표준 답변이 없습니다. 에이전트 통신 프로토콜에 대한 학습자의 포괄적인 이해와 실무 능력을 배양하는 데 중점을 두고 있습니다.

1. 이 장에서는 MCP, A2A, ANP의 세 가지 에이전트 통신 프로토콜을 소개했습니다. 분석해 주십시오:

- 10.1.2절에서는 세 가지 프로토콜의 설계 철학을 비교했습니다. 심층 분석해 보세요. MCP는 왜 "컨텍스트 공유"를 강조하고, A2A는 "대화 협업"을 강조하며, ANP는 "네트워크 토폴로지"를 강조합니까? 이러한 디자인 철학은 각각 어떤 핵심 문제를 해결합니까?
   - 다음 기능이 필요한 "지능형 고객 서비스 시스템"을 구축한다고 가정해 보겠습니다. (1) 고객 데이터베이스에 액세스하고 주문 시스템; (2) 여러 전문 고객 서비스 에이전트가 협력하여 복잡한 문제를 처리합니다. (3) 대규모 동시 사용자 요청을 지원합니다. 각 기능에 가장 적합한 프로토콜을 선택하고 그 이유를 설명해주세요.
   - 세 가지 프로토콜을 조합해서 사용할 수 있나요? MCP, A2A, ANP를 동시에 사용하여 완전한 에이전트 시스템을 구축하는 방법을 보여주는 실제 적용 시나리오를 설계하십시오. 시스템 아키텍처 다이어그램을 그리고 각 프로토콜의 책임을 설명합니다.

2. MCP(Model Context Protocol)는 에이전트-도구 통신을 위한 표준 프로토콜입니다. 10.2절의 내용을 바탕으로 깊이 생각해 보십시오.

> **참고**: 이것은 실습 연습 문제이므로 실제 작동을 권장합니다.

- 10.2.3절의 MCP 서버 구현에서는 `list_tools` 및 `call_tool`과 같은 핵심 메소드를 정의했습니다. 다음 도구를 제공하는 새로운 MCP 서버를 추가하여 이 구현을 확장하십시오: (1) 데이터베이스 쿼리 도구; (2) 데이터 시각화 도구; (3) 보고서 생성 도구. 복잡한 데이터 분석 작업을 완료하려면 도구가 협업할 수 있어야 합니다.
   - MCP 프로토콜은 "리소스"와 "프롬프트"라는 두 가지 중요한 개념을 지원하지만 이 장에서는 주로 "도구"에 중점을 둡니다. 리소스 및 프롬프트의 디자인 목적을 이해하려면 MCP 공식 문서를 참조하고 이 세 가지 핵심 개념을 사용하여 보다 강력한 에이전트 시스템을 구축하는 방법을 보여주는 애플리케이션 시나리오를 디자인하십시오.
   - MCP는 JSON-RPC 2.0을 기본 통신 프로토콜로 사용하고 stdio를 통해 프로세스 간 통신합니다. 분석해 보십시오. 이 디자인의 장점과 한계는 무엇입니까? 원격 MCP 서버 (accessed via HTTP/WebSocket)를 지원해야 하는 경우 현재 구현을 어떻게 확장해야 합니까?

3. A2A(Agent-to-Agent Protocol)는 에이전트 간 대화 협업을 지원합니다. 섹션 10.3의 내용을 바탕으로 다음 확장 연습을 완료하세요.

> **참고**: 이것은 실습 연습 문제이므로 실제 작동을 권장합니다.

- 10.3.4항의 '연구팀' 사례에서는 연구자와 작가가 A2A 프로토콜을 통해 협력하여 논문 작성을 완료합니다. 논문 품질을 검토하고 개정 제안을 제공할 수 있는 제3의 대리인 "검토자"를 추가하여 이 사례를 연장해 주세요. 세 에이전트 간의 협업 프로세스를 설계하고 완전한 코드를 구현합니다.
   - A2A 프로토콜은 `task` 및 `task_result`과 같은 메시지 유형을 정의합니다. 분석해 보십시오. 협업 중에 충돌이 발생하는 경우(예: 동일한 문제에 대해 두 에이전트가 서로 다른 의견을 갖는 경우) 충돌 해결 메커니즘을 어떻게 설계해야 합니까? "협상", "투표" 등의 메시지 유형을 추가하여 A2A 프로토콜을 확장하세요.
   - A2A 프로토콜을 6장에서 소개한 AutoGen, CAMEL 등의 다중 에이전트 프레임워크와 비교해보세요. 표준 프로토콜인 A2A와 이러한 프레임워크의 관계는 무엇인가요? 서로 교체할 수 있나요? A2A 프로토콜 기반 에이전트가 AutoGen 프레임워크의 에이전트와 통신할 수 있는 솔루션을 설계하십시오.

4. ANP(Agent Network Protocol)는 대규모 에이전트 네트워크를 지원합니다. 섹션 10.4의 내용을 바탕으로 심층 분석해 보십시오.

- 섹션 10.4.2에서는 스타, 메시, 계층 및 기타 구조를 포함한 ANP의 네트워크 토폴로지 설계를 소개했습니다. 분석해 보십시오. 어떤 시나리오에서 어떤 토폴로지 구조를 선택해야 합니까? 네트워크 규모가 에이전트 10개에서 에이전트 1000개로 확장된다면 토폴로지 구조는 어떻게 발전해야 할까요?
   - ANP 프로토콜은 "라우팅" 및 "검색" 메커니즘을 지원하므로 에이전트가 적합한 협업 파트너를 동적으로 찾을 수 있습니다. "지능형 라우팅 알고리즘"을 설계하십시오. 작업 유형, 에이전트 기능, 네트워크 부하 및 기타 요소를 기반으로 최적의 메시지 라우팅 경로를 자동으로 선택하십시오.
   - 섹션 10.4.4의 "스마트 시티" 사례에서는 여러 에이전트가 협력하여 도시 시스템을 관리합니다. 생각해 보십시오: 중요한 에이전트(예: 트래픽 관리 에이전트)가 실패하는 경우 전체 시스템은 어떻게 대응해야 합니까? 오류 감지, 백업 전환, 상태 복구 및 기타 기능을 포함하는 "내결함성 메커니즘"을 설계하십시오.

5. 에이전트 통신 프로토콜의 보안 및 개인 정보 보호는 실제 응용 분야의 주요 문제입니다. 생각해 보십시오:

- 섹션 10.2.4의 MCP 클라이언트 구현에서 에이전트는 MCP 서버가 제공하는 모든 도구를 호출할 수 있습니다. 분석해 보십시오. 이 디자인에는 어떤 보안 위험이 있습니까? MCP 서버가 위험한 작업(예: 파일 삭제, 시스템 명령 실행)을 제공하는 경우 권한 제어 메커니즘을 어떻게 설계해야 합니까?
   - A2A 및 ANP 프로토콜에는 민감한 정보(예: 사용자 개인 정보 데이터, 비즈니스 비밀)가 포함될 수 있는 여러 에이전트 간의 통신이 포함됩니다. "종단 간 암호화" 솔루션을 설계하십시오. 에이전트 ID 인증 및 액세스 제어를 지원하는 동시에 전송 중에 메시지가 도청되거나 변조되지 않도록 하십시오.
   - 대규모 에이전트 네트워크에서는 악의적인 에이전트가 허위 정보를 보내거나, 서비스 거부 공격을 시작하거나, 다른 에이전트의 데이터를 훔칠 수 있습니다. "신뢰 평가 시스템"을 설계하십시오. 과거 행동, 협업 품질, 커뮤니티 평가 및 기타 요소를 기반으로 각 에이전트의 신뢰성을 동적으로 평가하고 그에 따라 커뮤니케이션 전략을 조정하십시오.

## 참고자료

[1] 인류. (2024). *모델 컨텍스트 프로토콜*. 2025년 10월 7일, https://modelcontextprotocol.io/에서 검색함

[2] A2A 프로젝트. (2025). *A2A 프로토콜: 에이전트 간 통신을 위한 개방형 프로토콜*. 2025년 10월 7일, https://a2a-protocol.org/에서 검색함

[3] Chang, G., Lin, E., Yuan, C., Cai, R., Chen, B., Xie, X., & Zhang, Y. (2025). *에이전트 네트워크 프로토콜 기술 백서*. arXiv. https://doi.org/10.48550/arXiv.2508.00007

