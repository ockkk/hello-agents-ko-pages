# 8장 기억과 인출

이전 장에서는 다양한 에이전트 패러다임과 도구 시스템을 구현하여 HelloAgents 프레임워크의 기본 아키텍처를 구축했습니다. 그러나 우리 프레임워크에는 여전히 중요한 기능인 **메모리**가 부족합니다. 에이전트가 이전 상호 작용을 기억하지 못하거나 과거 경험을 통해 학습할 수 없으면 지속적인 대화나 복잡한 작업에서 성능이 크게 제한됩니다.

이 장에서는 7장에서 구축한 프레임워크를 기반으로 HelloAgents에 **메모리 시스템** 및 **RAG(Retrieval-Augmented Generation)**라는 두 가지 핵심 기능을 추가합니다. 우리는 "프레임워크 확장 + 지식 대중화" 접근 방식을 채택하여 구축 과정에서 메모리와 RAG의 이론적 기초를 깊이 이해하고 궁극적으로 완전한 메모리 및 지식 검색 기능을 갖춘 에이전트 시스템을 구현합니다.


## 8.1 인지과학에서 에이전트 메모리까지

### 8.1.1 인간 기억 시스템으로부터의 영감

에이전트의 메모리 시스템을 구축하기 전에 먼저 인간이 정보를 처리하고 저장하는 방법을 인지과학적 관점에서 이해해 보겠습니다. 인간의 기억은 정보를 저장할 뿐만 아니라 중요도, 시간, 맥락에 따라 정보를 분류하고 정리하는 다단계 인지 시스템입니다. 인지 심리학은 그림 8.1에서 볼 수 있듯이 기억<sup>[1]</sup>의 구조와 과정을 이해하기 위한 고전적인 이론적 틀을 제공합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-1.png" alt="Human Memory System Structure" width="85%"/>
<p>그림 8.1 인간 기억 시스템의 계층 구조</p>
</div>

인지 심리학 연구에 따르면 인간의 기억은 다음과 같은 수준으로 나눌 수 있습니다.

1. **감각 기억**: 매우 짧은 기간 (0.5-3 seconds), 대용량, 감각으로 받은 모든 정보를 일시적으로 저장하는 역할
2. **작업 기억**: 짧은 기간(15-30초), 제한된 용량(7±2개 항목), 현재 작업의 정보 처리를 담당합니다.
3. **장기 기억**: 장기 기억(평생 지속될 수 있음), 거의 무제한 용량, 다음으로 나누어집니다.
   - **절차적 기억**: 기술 및 습관(자전거 타기 등)
   - **선언적 기억**: 언어로 표현될 수 있는 지식으로, 다음과 같이 더 세분화됩니다.
     - **의미기억**: 일반 지식 및 개념(예: "파리는 프랑스의 수도입니다")
     - **일화기억**: 개인의 경험 및 사건('어제 모임 내용' 등)

### 8.1.2 에이전트에 메모리와 RAG가 필요한 이유

인간 기억 시스템의 설계를 통해 에이전트에도 유사한 메모리 기능이 필요한 이유를 이해할 수 있습니다. 인간 지능의 중요한 특징은 과거 경험을 기억하고, 그로부터 배우고, 이러한 경험을 새로운 상황에 적용하는 능력입니다. 마찬가지로, 진정한 지능형 에이전트에는 메모리 기능도 필요합니다. LLM 기반 에이전트의 경우 일반적으로 **대화 상태를 잊어버리는 것**과 **내장된 지식의 한계**라는 두 가지 근본적인 한계에 직면합니다.

(1) 제한사항 1: 무국적으로 인한 대화 망각

현재의 대규모 언어 모델은 강력하기는 하지만 **상태 비저장**하도록 설계되었습니다. 이는 각 사용자 요청(또는 API 호출)이 독립적이고 관련 없는 계산임을 의미합니다. 모델 자체는 이전 대화의 내용을 자동으로 "기억"하지 않습니다. 이로 인해 몇 가지 문제가 발생합니다.

1. **컨텍스트 손실**: 긴 대화에서는 컨텍스트 창 제한으로 인해 중요한 초기 정보가 손실될 수 있습니다.
2. **개인화 부족**: 상담원은 사용자 선호도, 습관 또는 특정 요구 사항을 기억할 수 없습니다.
3. **제한된 학습 능력**: 과거의 성공이나 실패로부터 배우고 발전할 수 없습니다.
4. **일관성 문제**: 여러 차례 대화에서 모순된 답변을 제공할 수 있습니다.

구체적인 예를 통해 이 문제를 이해해 보겠습니다.

```python
# How to use Agent from Chapter 7
from hello_agents import SimpleAgent, HelloAgentsLLM

agent = SimpleAgent(name="Learning Assistant", llm=HelloAgentsLLM())

# First conversation
response1 = agent.run("My name is Zhang San, I'm learning Python and have mastered basic syntax")
print(response1)  # "Great! Python basic syntax is an important foundation for programming..."
 
# Second conversation (new session, such as after restarting the program and creating a new Agent)
agent = SimpleAgent(name="Learning Assistant", llm=HelloAgentsLLM())
response2 = agent.run("Do you remember my learning progress?")
print(response2)  # "Sorry, I don't know your learning progress..."
```

7장의 `SimpleAgent`는 동일한 인스턴스 내에서 `_history`의 현재 대화를 일시적으로 저장하므로 동일한 프로세스 및 인스턴스의 연속 턴이 최근 컨텍스트를 전달할 수 있습니다. 그러나 이 기록은 임시 메시지 목록일 뿐입니다. 세션 전체에 걸쳐 지속되지 않으며 장기 검색, 망각 또는 통합을 지원하지 않습니다.

이 문제를 해결하려면 프레임워크에 메모리 시스템을 도입해야 합니다.

(2) 한계 2: 모델 내장 지식의 한계

대화 기록을 잊어버리는 것 외에도 LLM의 또 다른 핵심 제한은 지식이 **정적이고 제한적**이라는 것입니다. 이러한 지식은 전적으로 훈련 데이터에서 나오므로 일련의 문제가 발생합니다.

1. **지식 적시성**: 대형 모델에는 학습 데이터 마감일이 있어 최신 정보에 액세스할 수 없습니다.
2. **영역별 지식**: 일반 모델은 특정 영역에 대한 깊이가 부족할 수 있습니다.
3. **사실적 정확성**: 검색 검증을 통해 모델 환각을 줄입니다.
4. **설명성**: 답변의 신뢰성을 높이기 위한 정보 소스 제공

이러한 한계를 극복하기 위해 RAG 기술이 등장했습니다. 핵심 아이디어는 모델이 답변을 생성하기 전에 외부 지식 베이스(예: 문서, 데이터베이스, API)에서 가장 관련성이 높은 정보를 검색하고 이 정보를 모델에 대한 컨텍스트로 제공하는 것입니다.

### 8.1.3 메모리 및 RAG 시스템 아키텍처 설계

7장에서 확립된 프레임워크 기반과 인지 과학의 영감을 바탕으로 우리는 그림 8.2와 같이 계층화된 메모리와 RAG 시스템 아키텍처를 설계했습니다. 이 아키텍처는 인간 메모리 시스템의 계층 구조를 활용할 뿐만 아니라 엔지니어링 구현의 확장성을 완전히 고려합니다. 구현 시 메모리와 RAG를 두 개의 독립적인 도구로 설계합니다. `memory_tool`는 대화 중 상호 작용 정보를 저장하고 유지하는 역할을 담당하고, `rag_tool`은 사용자가 제공한 지식 기반에서 관련 정보를 컨텍스트로 검색하는 역할을 담당하며 중요한 검색 결과를 메모리 시스템에 자동으로 저장할 수 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-2.png" alt="HelloAgents Memory and RAG System Architecture" width="95%"/>
<p>그림 8.2 HelloAgents 메모리 및 RAG 시스템의 전체 아키텍처</p>
</div>

메모리 시스템은 4계층 아키텍처 설계를 채택합니다.

```
HelloAgents Memory System
├── Infrastructure Layer
│   ├── MemoryManager - Memory manager (unified scheduling and coordination)
│   ├── MemoryItem - Memory data structure (standardized memory items)
│   ├── MemoryConfig - Configuration management (system parameter settings)
│   └── BaseMemory - Memory base class (common interface definition)
├── Memory Types Layer
│   ├── WorkingMemory - Working memory (temporary information, TTL management)
│   ├── EpisodicMemory - Episodic memory (specific events, time series)
│   ├── SemanticMemory - Semantic memory (abstract knowledge, graph relationships)
│   └── PerceptualMemory - Perceptual memory (multimodal data)
├── Storage Backend Layer
│   ├── QdrantVectorStore - Vector storage (high-performance semantic retrieval)
│   ├── Neo4jGraphStore - Graph storage (knowledge graph management)
│   └── SQLiteDocumentStore - Document storage (structured persistence)
└── Embedding Service Layer
    ├── DashScopeEmbedding - Tongyi Qianwen embedding (cloud API)
    ├── LocalTransformerEmbedding - Local embedding (offline deployment)
    └── TFIDFEmbedding - TFIDF embedding (lightweight fallback)
```

RAG 시스템은 외부 지식을 획득하고 활용하는 데 중점을 둡니다.

```
HelloAgents RAG System
├── Document Processing Layer
│   ├── DocumentProcessor - Document processor (multi-format parsing)
│   ├── Document - Document object (metadata management)
│   └── Pipeline - RAG pipeline (end-to-end processing)
├── Embedding Layer
│   └── Unified Embedding Interface - Reuses memory system's embedding service
├── Vector Storage Layer
│   └── QdrantVectorStore - Vector database (namespace isolation)
└── Intelligent Q&A Layer
    ├── Multi-strategy Retrieval - Vector retrieval + MQE + HyDE
    ├── Context Construction - Intelligent fragment merging and truncation
    └── LLM-Enhanced Generation - Accurate Q&A based on context
```

### 8.1.4 학습 목표 및 빠른 경험

먼저 8장의 핵심 학습 내용을 살펴보겠습니다.

```
hello-agents/
├── hello_agents/
│   ├── memory/                   # Memory system module
│   │   ├── base.py               # Basic data structures (MemoryItem, MemoryConfig, BaseMemory)
│   │   ├── manager.py            # Memory manager (unified coordination and scheduling)
│   │   ├── embedding.py          # Unified embedding service (DashScope/Local/TFIDF)
│   │   ├── types/                # Memory type implementations
│   │   │   ├── working.py        # Working memory (TTL management, pure in-memory)
│   │   │   ├── episodic.py       # Episodic memory (event sequence, SQLite+Qdrant)
│   │   │   ├── semantic.py       # Semantic memory (knowledge graph, Qdrant+Neo4j)
│   │   │   └── perceptual.py     # Perceptual memory (multimodal, SQLite+Qdrant)
│   │   ├── storage/              # Storage backend implementations
│   │   │   ├── qdrant_store.py   # Qdrant vector storage (high-performance vector retrieval)
│   │   │   ├── neo4j_store.py    # Neo4j graph storage (knowledge graph management)
│   │   │   └── document_store.py # SQLite document storage (structured persistence)
│   │   └── rag/                  # RAG system
│   │       ├── pipeline.py       # RAG pipeline (end-to-end processing)
│   │       └── document.py       # Document processor (multi-format parsing)
│   └── tools/builtin/            # Extended built-in tools
│       ├── memory_tool.py        # Memory tool (Agent memory capability)
│       └── rag_tool.py           # RAG tool (intelligent Q&A capability)
└──
```

**빠른 시작: HelloAgents 프레임워크 설치**

독자가 이 장의 전체 기능을 빠르게 경험할 수 있도록 직접 설치할 수 있는 Python 패키지를 제공합니다. 다음 명령을 사용하여 이 장에 해당하는 버전을 설치할 수 있습니다.

```bash
# If you encounter model unavailability in version 0.2.0, please refer to issue#320 or switch to version 0.2.9 for testing.
pip install "hello-agents[all]==0.2.0"
python -m spacy download zh_core_web_sm
python -m spacy download en_core_web_sm
```

또한 `.env`에서 그래프 데이터베이스, 벡터 데이터베이스, LLM, Embedding 솔루션 API를 구성해야 합니다. 튜토리얼에서는 Qdrant가 벡터 데이터베이스로, Neo4J가 그래프 데이터베이스로, Bailian 플랫폼이 Embedding으로 선호됩니다. API를 사용할 수 없는 경우 로컬 배포 모델 솔루션으로 전환할 수 있습니다.

```bash
# ================================
# Qdrant Vector Database Configuration - Get API key: https://cloud.qdrant.io/
# ================================
# Use Qdrant cloud service (recommended)
QDRANT_URL=https://your-cluster.qdrant.tech:6333
QDRANT_API_KEY=your_qdrant_api_key_here

# Or use local Qdrant (requires Docker)
# QDRANT_URL=http://localhost:6333
# QDRANT_API_KEY=

# Qdrant collection configuration
QDRANT_COLLECTION=hello_agents_vectors
QDRANT_VECTOR_SIZE=384
QDRANT_DISTANCE=cosine
QDRANT_TIMEOUT=30

# ================================
# Neo4j Graph Database Configuration - Get API key: https://neo4j.com/cloud/aura/
# ================================
# Use Neo4j Aura cloud service (recommended)
NEO4J_URI=neo4j+s://your-instance.databases.neo4j.io
NEO4J_USERNAME=neo4j
NEO4J_PASSWORD=your_neo4j_password_here

# Or use local Neo4j (requires Docker)
# NEO4J_URI=bolt://localhost:7687
# NEO4J_USERNAME=neo4j
# NEO4J_PASSWORD=hello-agents-password

# Neo4j connection configuration
NEO4J_DATABASE=neo4j
NEO4J_MAX_CONNECTION_LIFETIME=3600
NEO4J_MAX_CONNECTION_POOL_SIZE=50
NEO4J_CONNECTION_TIMEOUT=60

# ==========================
# Embedding Configuration Example - Get from Alibaba Cloud Console: https://dashscope.aliyun.com/
# ==========================
# - If empty, dashscope defaults to text-embedding-v3; local defaults to sentence-transformers/all-MiniLM-L6-v2
EMBED_MODEL_TYPE=dashscope
EMBED_MODEL_NAME=
EMBED_API_KEY=
EMBED_BASE_URL=
```

이 장의 학습은 두 가지 방법으로 수행할 수 있습니다.

1. **체험학습**: `pip`을 이용해 프레임워크를 직접 설치하고, 예제 코드를 실행하며, 다양한 기능을 빠르게 체험해 보세요.
2. **딥 러닝**: 장의 내용을 따르고, 각 구성 요소를 처음부터 구현하고, 프레임워크의 설계 철학과 구현 세부 사항을 깊이 이해합니다.

'먼저 경험한 후 구현'하는 학습 경로를 채택하는 것이 좋습니다. 이 장에서는 완전한 테스트 파일을 제공합니다. 핵심 기능을 다시 작성하고 테스트를 실행하여 구현이 올바른지 확인할 수 있습니다.

7장에서 확립된 설계 원칙에 따라 새로운 에이전트 클래스를 생성하는 대신 메모리 및 RAG 기능을 표준 도구로 캡슐화합니다. 시작하기 전에 Hello-agents를 사용하여 메모리와 RAG 기능을 갖춘 에이전트 구축을 30초 동안 경험해 보겠습니다!

```python
# Configure the LLM API in .env in the same folder
from hello_agents import SimpleAgent, HelloAgentsLLM, ToolRegistry
from hello_agents.tools import MemoryTool, RAGTool

# Create LLM instance
llm = HelloAgentsLLM()

# Create Agent
agent = SimpleAgent(
    name="Intelligent Assistant",
    llm=llm,
    system_prompt="You are an AI assistant with memory and knowledge retrieval capabilities"
)

# Create tool registry
tool_registry = ToolRegistry()

# Add memory tool
memory_tool = MemoryTool(user_id="user123")
tool_registry.register_tool(memory_tool)

# Add RAG tool
rag_tool = RAGTool(knowledge_base_path="./knowledge_base")
tool_registry.register_tool(rag_tool)

# Configure tools for Agent
agent.tool_registry = tool_registry

# Start conversation
response = agent.run("Hello! Please remember my name is Zhang San, I am a Python developer")
print(response)
```

모든 것이 올바르게 구성되면 다음 내용을 볼 수 있습니다.

```bash
[OK] SQLite database tables and indexes created
[OK] SQLite document storage initialized: ./memory_data\memory.db
INFO:hello_agents.memory.storage.qdrant_store:✅ Successfully connected to Qdrant cloud service: https://0c517275-2ad0-4442-8309-11c36dc7e811.us-east-1-1.aws.cloud.qdrant.io:6333
INFO:hello_agents.memory.storage.qdrant_store:✅ Using existing Qdrant collection: hello_agents_vectors
INFO:hello_agents.memory.types.semantic:✅ Embedding model ready, dimension: 1024
INFO:hello_agents.memory.types.semantic:✅ Qdrant vector database initialization complete
INFO:hello_agents.memory.storage.neo4j_store:✅ Successfully connected to Neo4j cloud service: neo4j+s://851b3a28.databases.neo4j.io
INFO:hello_agents.memory.types.semantic:✅ Neo4j graph database initialization complete
INFO:hello_agents.memory.storage.neo4j_store:✅ Neo4j index creation complete
INFO:hello_agents.memory.types.semantic:✅ Neo4j graph database initialization complete
INFO:hello_agents.memory.types.semantic:🏥 Database health status: Qdrant=✅, Neo4j=✅
INFO:hello_agents.memory.types.semantic:✅ Loaded Chinese spaCy model: zh_core_web_sm
INFO:hello_agents.memory.types.semantic:✅ Loaded English spaCy model: en_core_web_sm
INFO:hello_agents.memory.types.semantic:📚 Available language models: Chinese, English
INFO:hello_agents.memory.types.semantic:Enhanced semantic memory initialization complete (using Qdrant+Neo4j professional databases)
INFO:hello_agents.memory.manager:MemoryManager initialization complete, enabled memory types: ['working', 'episodic', 'semantic']
✅ Tool 'memory' registered.
INFO:hello_agents.memory.storage.qdrant_store:✅ Successfully connected to Qdrant cloud service: https://0c517275-2ad0-4442-8309-11c36dc7e811.us-east-1-1.aws.cloud.qdrant.io:6333
INFO:hello_agents.memory.storage.qdrant_store:✅ Using existing Qdrant collection: rag_knowledge_base
✅ RAG tool initialization successful: namespace=default, collection=rag_knowledge_base
✅ Tool 'rag' registered.
Hello, Zhang San! Nice to meet you. As a Python developer, you must be passionate about programming. If you have any technical questions or need to discuss Python-related topics, feel free to reach out to me anytime. I'll do my best to help you. Is there anything I can help you with right now?
```

## 8.2 메모리 시스템: 에이전트에 메모리 제공

### 8.2.1 메모리 시스템 작업 흐름

코드 구현 단계에 들어가기 전에 먼저 메모리 시스템의 워크플로우를 정의해야 합니다. 이 워크플로는 인지 과학의 기억 모델을 참조하고 각 인지 단계를 특정 기술 구성 요소 및 작업에 매핑합니다. 이 매핑 관계를 이해하면 후속 코드 구현에 도움이 됩니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-3.png" alt="Memory Formation Process" width="90%"/>
<p>그림 8.3 기억 형성의 인지 과정</p>
</div>

그림 8.3에서 볼 수 있듯이, 인지 과학 연구에 따르면 인간의 기억 형성은 다음 단계를 거칩니다.

1. **인코딩**: 인지된 정보를 저장 가능한 형태로 변환
2. **저장**: 인코딩된 정보를 메모리 시스템에 저장
3. **검색**: 필요에 따라 메모리에서 관련 정보 추출
4. **통합**: 단기 기억을 장기 기억으로 전환
5. **망각**: 중요하지 않거나 오래된 정보를 삭제하는 것

이러한 영감을 바탕으로 우리는 HelloAgents를 위한 완전한 메모리 시스템을 설계했습니다. 핵심 아이디어는 인간 두뇌가 다양한 유형의 정보를 처리하는 방식을 모방하여 메모리를 여러 특수 모듈로 나누고 지능형 관리 메커니즘을 구축하는 것입니다. 그림 8.4는 메모리 추가, 검색, 통합 및 망각과 같은 주요 링크를 포함하여 이 시스템의 작업 흐름을 자세히 보여줍니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-4.png" alt="Memory System Workflow" width="95%"/>
<p>그림 8.4 HelloAgents 메모리 시스템의 전체 워크플로</p>
</div>

우리의 메모리 시스템은 네 가지 유형의 메모리 모듈로 구성되어 있으며 각각은 특정 애플리케이션 시나리오 및 수명주기에 최적화되어 있습니다.

먼저 **워킹 메모리**는 에이전트의 '단기 기억' 역할을 하며 주로 현재 대화의 맥락 정보를 저장하는 데 사용됩니다. 고속 액세스 및 응답을 보장하기 위해 용량을 의도적으로 제한하고(예: 기본적으로 50개 항목) 수명 주기를 단일 세션에 바인딩하여 세션이 끝나면 자동으로 지워집니다.

두 번째는 특정 상호작용 이벤트와 에이전트의 학습 경험을 장기간 저장하는 역할을 하는 **Episodic Memory**입니다. 작업 기억과 달리 일화 기억은 풍부한 맥락 정보를 포함하고 시계열이나 주제별 회고적 검색을 지원하여 에이전트가 과거 경험을 "검토"하고 학습할 수 있는 기반 역할을 합니다.

특정 사건에 대응하는 것은 보다 추상적인 지식, 개념, 규칙을 저장하는 **의미 기억**입니다. 예를 들어 대화를 통해 알게 된 사용자 선호도, 장기적으로 따라야 하는 지침, 도메인 지식 포인트 등 모두 여기에 저장하기에 적합합니다. 기억의 이 부분은 지속성과 중요도가 높으며 에이전트가 "지식 시스템"을 형성하고 연관 추론을 수행하는 핵심입니다.

마지막으로, 점점 더 풍부해지는 멀티미디어와 상호 작용하기 위해 **지각 기억**을 도입했습니다. 이 모듈은 특히 이미지 및 오디오와 같은 다중 모드 정보를 처리하고 교차 모드 검색을 지원합니다. 정보의 중요성과 사용 가능한 저장 공간에 따라 수명주기가 동적으로 관리됩니다.

### 8.2.2 빠른 경험: 30초 안에 메모리 기능 시작하기

구현 세부 사항을 살펴보기 전에 메모리 시스템의 기본 기능을 빠르게 경험해 보겠습니다.

```python
from hello_agents import SimpleAgent, HelloAgentsLLM, ToolRegistry
from hello_agents.tools import MemoryTool

# Create Agent with memory capability
llm = HelloAgentsLLM()
agent = SimpleAgent(name="Memory Assistant", llm=llm)

# Create memory tool
memory_tool = MemoryTool(user_id="user123")
tool_registry = ToolRegistry()
tool_registry.register_tool(memory_tool)
agent.tool_registry = tool_registry

# Experience memory features
print("=== Adding Multiple Memories ===")

# Add first memory
result1 = memory_tool.execute("add", content="User Zhang San is a Python developer focusing on machine learning and data analysis", memory_type="semantic", importance=0.8)
print(f"Memory 1: {result1}")

# Add second memory
result2 = memory_tool.execute("add", content="Li Si is a frontend engineer skilled in React and Vue.js development", memory_type="semantic", importance=0.7)
print(f"Memory 2: {result2}")

# Add third memory
result3 = memory_tool.execute("add", content="Wang Wu is a product manager responsible for user experience design and requirements analysis", memory_type="semantic", importance=0.6)
print(f"Memory 3: {result3}")

print("\n=== Searching Specific Memories ===")
# Search for frontend-related memories
print("🔍 Searching 'frontend engineer':")
result = memory_tool.execute("search", query="frontend engineer", limit=3)
print(result)

print("\n=== Memory Summary ===")
result = memory_tool.execute("summary")
print(result)
```

### 8.2.3 MemoryTool 상세 설명

이제 MemoryTool에서 지원하는 특정 작업부터 시작하여 점차적으로 기본 구현을 살펴보는 하향식 접근 방식을 채택해 보겠습니다. 메모리 시스템의 통합 인터페이스인 MemoryTool은 "통합 입력, 분산 처리"라는 아키텍처 패턴을 따릅니다.

````python
def execute(self, action: str, **kwargs) -> str:
    """Execute memory operation

    Supported operations:
    - add: Add memory (supports 4 types: working/episodic/semantic/perceptual)
    - search: Search memory
    - summary: Get memory summary
    - stats: Get statistics
    - update: Update memory
    - remove: Delete memory
    - forget: Forget memory (multiple strategies)
    - consolidate: Consolidate memory (short-term → long-term)
    - clear_all: Clear all memories
    """

    if action == "add":
        return self._add_memory(**kwargs)
    elif action == "search":
        return self._search_memory(**kwargs)
    elif action == "summary":
        return self._get_summary(**kwargs)
    # ... other operations
````

이 통합된 `execute` 인터페이스 디자인은 에이전트 호출 방법을 단순화합니다. 특정 작업은 `action` 매개변수를 통해 지정되며 `**kwargs`를 사용하면 각 작업에 서로 다른 매개변수 요구사항이 있을 수 있습니다. 여기에는 몇 가지 중요한 작업이 나열되어 있습니다.

(1) 작업 1: 추가

`add` 연산은 메모리 시스템의 기초입니다. 이는 인지된 정보를 기억으로 인코딩하는 인간 두뇌의 과정을 시뮬레이션합니다. 구현에서는 메모리 콘텐츠를 저장할 뿐만 아니라 각 메모리에 풍부한 상황 정보를 추가해야 합니다. 이 정보는 후속 검색 및 관리에 중요한 역할을 합니다.

````python
def _add_memory(
    self,
    content: str = "",
    memory_type: str = "working",
    importance: float = 0.5,
    file_path: str = None,
    modality: str = None,
    **metadata
) -> str:
    """Add memory"""
    try:
        # Ensure session ID exists
        if self.current_session_id is None:
            self.current_session_id = f"session_{datetime.now().strftime('%Y%m%d_%H%M%S')}"

        # Perceptual memory file support
        if memory_type == "perceptual" and file_path:
            inferred = modality or self._infer_modality(file_path)
            metadata.setdefault("modality", inferred)
            metadata.setdefault("raw_data", file_path)

        # Add session information to metadata
        metadata.update({
            "session_id": self.current_session_id,
            "timestamp": datetime.now().isoformat()
        })

        memory_id = self.memory_manager.add_memory(
            content=content,
            memory_type=memory_type,
            importance=importance,
            metadata=metadata,
            auto_classify=False
        )

        return f"✅ Memory added (ID: {memory_id[:8]}...)"

    except Exception as e:
        return f"❌ Failed to add memory: {str(e)}"
````

이는 주로 세 가지 주요 작업, 즉 세션 ID 자동 관리(각 메모리에 명확한 세션 속성 보장), 다중 모달 데이터의 지능적 처리(자동으로 파일 유형 추론 및 관련 메타데이터 저장), 상황 정보 자동 보완(각 메모리에 타임스탬프 및 세션 정보 추가)을 구현합니다. 그 중 `importance` 매개변수 (default 0.5)는 메모리의 중요도 수준을 표시하는 데 사용되며 값 범위는 0.0-1.0입니다. 이 메커니즘은 다양한 정보의 중요성에 대한 인간 두뇌의 평가를 시뮬레이션합니다. 이러한 설계를 통해 에이전트는 서로 다른 기간의 대화를 자동으로 구별하고 후속 검색 및 관리를 위해 풍부한 상황 정보를 제공할 수 있습니다.

각 메모리 유형에 대해 다양한 사용 예를 제공합니다.

```python
# 1. Working Memory - Temporary information, limited capacity
memory_tool.execute("add",
    content="User just asked a question about Python functions",
    memory_type="working",
    importance=0.6
)

# 2. Episodic Memory - Specific events and experiences
memory_tool.execute("add",
    content="On March 15, 2024, user Zhang San completed their first Python project",
    memory_type="episodic",
    importance=0.8,
    event_type="milestone",
    location="Online learning platform"
)

# 3. Semantic Memory - Abstract knowledge and concepts
memory_tool.execute("add",
    content="Python is an interpreted, object-oriented programming language",
    memory_type="semantic",
    importance=0.9,
    knowledge_type="factual"
)

# 4. Perceptual Memory - Multimodal information
memory_tool.execute("add",
    content="User uploaded a Python code screenshot containing function definitions",
    memory_type="perceptual",
    importance=0.7,
    modality="image",
    file_path="./uploads/code_screenshot.png"
)
```

(2) 동작 2: 검색

`search` 연산은 메모리 시스템의 핵심 기능이다. 수많은 추억 중에서 쿼리와 가장 관련성이 높은 콘텐츠를 빠르게 찾아야 합니다. 의미론적 이해, 관련성 계산, 결과 정렬 등 여러 단계가 포함됩니다.

````python
def _search_memory(
    self,
    query: str,
    limit: int = 5,
    memory_types: List[str] = None,
    memory_type: str = None,
    min_importance: float = 0.1
) -> str:
    """Search memory"""
    try:
        # Parameter standardization
        if memory_type and not memory_types:
            memory_types = [memory_type]

        results = self.memory_manager.retrieve_memories(
            query=query,
            limit=limit,
            memory_types=memory_types,
            min_importance=min_importance
        )

        if not results:
            return f"🔍 No memories found related to '{query}'"

        # Format results
        formatted_results = []
        formatted_results.append(f"🔍 Found {len(results)} related memories:")

        for i, memory in enumerate(results, 1):
            memory_type_label = {
                "working": "Working Memory",
                "episodic": "Episodic Memory",
                "semantic": "Semantic Memory",
                "perceptual": "Perceptual Memory"
            }.get(memory.memory_type, memory.memory_type)

            content_preview = memory.content[:80] + "..." if len(memory.content) > 80 else memory.content
            formatted_results.append(
                f"{i}. [{memory_type_label}] {content_preview} (Importance: {memory.importance:.2f})"
            )

        return "\n".join(formatted_results)

    except Exception as e:
        return f"❌ Failed to search memory: {str(e)}"
````

검색 작업은 단수 및 복수 매개변수 형식(`memory_type` 및 `memory_types`)을 모두 지원하도록 설계되어 사용자가 가장 자연스러운 방식으로 요구 사항을 표현할 수 있습니다. 그 중 `min_importance` 매개변수 (default 0.1)는 저품질 메모리를 필터링하는 데 사용됩니다. 검색 기능을 사용하려면 다음 예를 참조하세요.

```python
# Basic search
result = memory_tool.execute("search", query="Python programming", limit=5)

# Search by specifying memory type
result = memory_tool.execute("search",
    query="learning progress",
    memory_type="episodic",
    limit=3
)

# Multi-type search
result = memory_tool.execute("search",
    query="function definition",
    memory_types=["semantic", "episodic"],
    min_importance=0.5
)
```

(3) 동작 3: 잊어버리기

망각 메커니즘은 인지적으로 가장 과학적인 특징이다. 이는 인간 두뇌의 선택적 망각 과정을 시뮬레이션하고 중요도 기반(중요하지 않은 기억 삭제), 시간 기반(오래된 기억 삭제), 용량 기반(저장 용량이 한계에 도달할 때 가장 덜 중요한 기억 삭제)의 세 가지 전략을 지원합니다.

````python
def _forget(self, strategy: str = "importance_based", threshold: float = 0.1, max_age_days: int = 30) -> str:
    """Forget memories (supports multiple strategies)"""
    try:
        count = self.memory_manager.forget_memories(
            strategy=strategy,
            threshold=threshold,
            max_age_days=max_age_days
        )
        return f"🧹 Forgot {count} memories (strategy: {strategy})"
    except Exception as e:
        return f"❌ Failed to forget memories: {str(e)}"
````

**세 가지 망각 전략 사용:**

```python
# 1. Importance-based forgetting - Delete memories below importance threshold
memory_tool.execute("forget",
    strategy="importance_based",
    threshold=0.2
)

# 2. Time-based forgetting - Delete memories older than specified days
memory_tool.execute("forget",
    strategy="time_based",
    max_age_days=30
)

# 3. Capacity-based forgetting - Delete least important when memory count exceeds limit
memory_tool.execute("forget",
    strategy="capacity_based",
    threshold=0.3
)
```

(4) 작업 4: 통합

````python
def _consolidate(self, from_type: str = "working", to_type: str = "episodic", importance_threshold: float = 0.7) -> str:
    """Consolidate memories (promote important short-term memories to long-term memories)"""
    try:
        count = self.memory_manager.consolidate_memories(
            from_type=from_type,
            to_type=to_type,
            importance_threshold=importance_threshold,
        )
        return f"🔄 Consolidated {count} memories to long-term memory ({from_type} → {to_type}, threshold={importance_threshold})"
    except Exception as e:
        return f"❌ Failed to consolidate memories: {str(e)}"
````

통합 작업은 신경과학의 기억 통합 개념을 활용하여 인간 두뇌가 단기 기억을 장기 기억으로 전환하는 과정을 시뮬레이션합니다. 기본 설정은 중요도가 0.7을 초과하는 작업 메모리를 에피소드 메모리로 변환하는 것입니다. 이 임계값은 정말 중요한 정보만 장기적으로 보존되도록 보장합니다. 전체 프로세스가 자동화됩니다. 사용자는 특정 메모리를 수동으로 선택할 필요가 없습니다. 시스템은 기준에 맞는 메모리를 지능적으로 식별하고 유형 변환을 수행합니다.

**메모리 통합 사용 예:**

```python
# Convert important working memories to episodic memories
memory_tool.execute("consolidate",
    from_type="working",
    to_type="episodic",
    importance_threshold=0.7
)

# Convert important episodic memories to semantic memories
memory_tool.execute("consolidate",
    from_type="episodic",
    to_type="semantic",
    importance_threshold=0.8
)
```

이러한 핵심 작업의 협력을 통해 MemoryTool은 완전한 메모리 수명주기 관리 시스템을 구축합니다. 메모리 생성, 검색, 요약부터 망각, 통합 및 관리에 이르기까지 폐쇄 루프 지능형 메모리 관리 시스템을 형성하여 에이전트에 진정으로 인간과 유사한 메모리 기능을 제공합니다.

### 8.2.4 MemoryManager 상세 설명

MemoryTool의 인터페이스 디자인을 이해한 후 기본 구현을 자세히 살펴보고 MemoryTool이 MemoryManager와 어떻게 협력하는지 살펴보겠습니다. 이 계층형 디자인은 소프트웨어 엔지니어링의 우려 분리 원칙을 구현합니다. MemoryTool은 사용자 인터페이스와 매개변수 처리에 중점을 두고 있으며 MemoryManager는 핵심 메모리 관리 로직을 담당합니다.

MemoryTool은 초기화 중에 MemoryManager 인스턴스를 생성하고 구성에 따라 다양한 유형의 메모리 모듈을 활성화합니다. 이 설계를 통해 사용자는 특정 요구 사항에 따라 활성화할 메모리 유형을 선택할 수 있으므로 불필요한 리소스 소비를 피하면서 기능적 완전성을 보장할 수 있습니다.

````python
class MemoryTool(Tool):
    """Memory tool - Provides memory functionality for Agent"""

    def __init__(
        self,
        user_id: str = "default_user",
        memory_config: MemoryConfig = None,
        memory_types: List[str] = None
    ):
        super().__init__(
            name="memory",
            description="Memory tool - Can store and retrieve conversation history, knowledge, and experience"
        )

        # Initialize memory manager
        self.memory_config = memory_config or MemoryConfig()
        self.memory_types = memory_types or ["working", "episodic", "semantic"]

        self.memory_manager = MemoryManager(
            config=self.memory_config,
            user_id=user_id,
            enable_working="working" in self.memory_types,
            enable_episodic="episodic" in self.memory_types,
            enable_semantic="semantic" in self.memory_types,
            enable_perceptual="perceptual" in self.memory_types
        )
````

MemoryManager는 메모리 시스템의 핵심 코디네이터로서 다양한 유형의 메모리 모듈을 관리하고 통일된 운영 인터페이스를 제공하는 역할을 담당합니다.

````python
class MemoryManager:
    """Memory manager - Unified memory operation interface"""

    def __init__(
        self,
        config: Optional[MemoryConfig] = None,
        user_id: str = "default_user",
        enable_working: bool = True,
        enable_episodic: bool = True,
        enable_semantic: bool = True,
        enable_perceptual: bool = False
    ):
        self.config = config or MemoryConfig()
        self.user_id = user_id

        # Initialize storage and retrieval components
        self.store = MemoryStore(self.config)
        self.retriever = MemoryRetriever(self.store, self.config)

        # Initialize various types of memory
        self.memory_types = {}

        if enable_working:
            self.memory_types['working'] = WorkingMemory(self.config, self.store)

        if enable_episodic:
            self.memory_types['episodic'] = EpisodicMemory(self.config, self.store)

        if enable_semantic:
            self.memory_types['semantic'] = SemanticMemory(self.config, self.store)

        if enable_perceptual:
            self.memory_types['perceptual'] = PerceptualMemory(self.config, self.store)
````

### 8.2.5 네 가지 유형의 메모리

이제 네 가지 메모리 유형의 구체적인 구현을 살펴보겠습니다. 각 메모리 유형에는 고유한 특성과 적용 시나리오가 있습니다.

(1) 작업기억

작업기억은 기억체계에서 가장 활동적인 부분이다. 현재 대화 세션에서 임시 정보를 저장하는 역할을 담당합니다. 작업 메모리의 설계 초점은 시스템의 응답 속도와 리소스 효율성을 보장하는 빠른 액세스 및 자동 정리에 있습니다.

작업 메모리는 자동 정리를 위한 TTL(Time To Live) 메커니즘과 결합된 순수 메모리 내 스토리지 솔루션을 채택합니다. 이 설계의 장점은 액세스 속도가 매우 빠르다는 것이지만 시스템을 다시 시작한 후에는 작업 메모리의 내용이 손실된다는 의미이기도 합니다. 이 특성은 임시 정보와 휘발성 정보를 저장하는 작업 메모리의 위치에 완벽하게 들어맞습니다.

````python
class WorkingMemory:
    """Working memory implementation
    Features:
    - Limited capacity (default 50 items) + TTL automatic cleanup
    - Pure in-memory storage, extremely fast access
    - Hybrid retrieval: TF-IDF vectorization + keyword matching
    """

    def __init__(self, config: MemoryConfig):
        self.max_capacity = config.working_memory_capacity or 50
        self.max_age_minutes = config.working_memory_ttl or 60
        self.memories = []

    def add(self, memory_item: MemoryItem) -> str:
        """Add working memory"""
        self._expire_old_memories()  # Expiration cleanup

        if len(self.memories) >= self.max_capacity:
            self._remove_lowest_priority_memory()  # Capacity management

        self.memories.append(memory_item)
        return memory_item.id

    def retrieve(self, query: str, limit: int = 5, **kwargs) -> List[MemoryItem]:
        """Hybrid retrieval: TF-IDF vectorization + keyword matching"""
        self._expire_old_memories()

        # Try TF-IDF vector retrieval
        vector_scores = self._try_tfidf_search(query)

        # Calculate comprehensive score
        scored_memories = []
        for memory in self.memories:
            vector_score = vector_scores.get(memory.id, 0.0)
            keyword_score = self._calculate_keyword_score(query, memory.content)

            # Hybrid scoring
            base_relevance = vector_score * 0.7 + keyword_score * 0.3 if vector_score > 0 else keyword_score
            time_decay = self._calculate_time_decay(memory.timestamp)
            importance_weight = 0.8 + (memory.importance * 0.4)

            final_score = base_relevance * time_decay * importance_weight
            if final_score > 0:
                scored_memories.append((final_score, memory))

        scored_memories.sort(key=lambda x: x[0], reverse=True)
        return [memory for _, memory in scored_memories[:limit]]
````

작업 메모리 검색은 하이브리드 검색 전략을 채택합니다. 먼저 의미 검색을 위해 TF-IDF 벡터화를 사용하려고 시도하고, 실패하면 키워드 일치로 대체됩니다. 이러한 설계는 다양한 환경에서 안정적인 검색 서비스를 보장합니다. 채점 알고리즘은 의미론적 유사성, 시간 감소 및 중요도 가중치를 결합합니다. 최종 점수 공식은 `(similarity × time decay) × (0.8 + importance × 0.4)`입니다.

(2) 일화기억

에피소드 기억은 특정 사건과 경험을 저장하는 역할을 합니다. 그 설계 초점은 사건과 시간적 순서 관계의 무결성을 유지하는 데 있습니다. 에피소드 메모리는 SQLite + Qdrant의 하이브리드 스토리지 솔루션을 채택합니다. SQLite는 구조화된 데이터와 복잡한 쿼리를 저장하는 역할을 하고, Qdrant는 효율적인 벡터 검색을 담당합니다.

````python
class EpisodicMemory:
    """Episodic memory implementation
    Features:
    - SQLite+Qdrant hybrid storage architecture
    - Supports time series and session-level retrieval
    - Structured filtering + semantic vector retrieval
    """

    def __init__(self, config: MemoryConfig):
        self.doc_store = SQLiteDocumentStore(config.database_path)
        self.vector_store = QdrantVectorStore(config.qdrant_url, config.qdrant_api_key)
        self.embedder = create_embedding_model_with_fallback()
        self.sessions = {}  # Session index

    def add(self, memory_item: MemoryItem) -> str:
        """Add episodic memory"""
        # Create episode object
        episode = Episode(
            episode_id=memory_item.id,
            session_id=memory_item.metadata.get("session_id", "default"),
            timestamp=memory_item.timestamp,
            content=memory_item.content,
            context=memory_item.metadata
        )

        # Update session index
        session_id = episode.session_id
        if session_id not in self.sessions:
            self.sessions[session_id] = []
        self.sessions[session_id].append(episode.episode_id)

        # Persistent storage (SQLite + Qdrant)
        self._persist_episode(episode)
        return memory_item.id

    def retrieve(self, query: str, limit: int = 5, **kwargs) -> List[MemoryItem]:
        """Hybrid retrieval: structured filtering + semantic vector retrieval"""
        # 1. Structured pre-filtering (time range, importance, etc.)
        candidate_ids = self._structured_filter(**kwargs)

        # 2. Vector semantic retrieval
        hits = self._vector_search(query, limit * 5, kwargs.get("user_id"))

        # 3. Comprehensive scoring and sorting
        results = []
        for hit in hits:
            if self._should_include(hit, candidate_ids, kwargs):
                score = self._calculate_episode_score(hit)
                memory_item = self._create_memory_item(hit)
                results.append((score, memory_item))

        results.sort(key=lambda x: x[0], reverse=True)
        return [item for _, item in results[:limit]]

    def _calculate_episode_score(self, hit) -> float:
        """Episodic memory scoring algorithm"""
        vec_score = float(hit.get("score", 0.0))
        recency_score = self._calculate_recency(hit["metadata"]["timestamp"])
        importance = hit["metadata"].get("importance", 0.5)

        # Scoring formula: (vector similarity × 0.8 + temporal recency × 0.2) × importance weight
        base_relevance = vec_score * 0.8 + recency_score * 0.2
        importance_weight = 0.8 + (importance * 0.4)

        return base_relevance * importance_weight
````

일화 기억의 검색 구현은 복잡한 다중 요인 채점 메커니즘을 보여줍니다. 이는 의미적 유사성을 고려할 뿐만 아니라 궁극적으로 중요도 가중치에 따라 조정되는 시간적 최근성 고려 사항도 통합합니다. 채점 공식은 `(vector similarity × 0.8 + temporal recency × 0.2) × (0.8 + importance × 0.4)`이며, 검색 결과가 의미상으로나 시간적으로 모두 관련되도록 보장합니다.

(3) 의미기억

의미기억은 기억체계에서 가장 복잡한 부분이다. 추상적인 개념, 규칙 및 지식을 저장하는 역할을 담당합니다. 의미기억의 설계 초점은 지식과 지능적 추론 능력의 구조화된 표현에 있습니다. 시맨틱 메모리는 Neo4j 그래프 데이터베이스와 Qdrant 벡터 데이터베이스의 하이브리드 아키텍처를 채택합니다. 이 설계를 통해 시스템은 지식 그래프를 사용하여 빠른 의미 검색과 복잡한 관계 추론을 모두 수행할 수 있습니다.

````python
class SemanticMemory(BaseMemory):
    """Semantic memory implementation

    Features:
    - Uses HuggingFace Chinese pre-trained models for text embedding
    - Vector retrieval for fast similarity matching
    - Knowledge graph storage for entities and relationships
    - Hybrid retrieval strategy: vector + graph + semantic reasoning
    """

    def __init__(self, config: MemoryConfig, storage_backend=None):
        super().__init__(config, storage_backend)

        # Embedding model (unified provision)
        self.embedding_model = get_text_embedder()

        # Professional database storage
        self.vector_store = QdrantConnectionManager.get_instance(**qdrant_config)
        self.graph_store = Neo4jGraphStore(**neo4j_config)

        # Entity and relation cache
        self.entities: Dict[str, Entity] = {}
        self.relations: List[Relation] = []

        # NLP processor (supports Chinese and English)
        self.nlp = self._init_nlp()
````

의미기억의 추가 과정은 지식 그래프 구성의 전체 흐름을 구현합니다. 시스템은 메모리 콘텐츠를 저장할 뿐만 아니라 엔터티와 관계를 자동으로 추출하여 구조화된 지식 표현을 구축합니다.

```python
def add(self, memory_item: MemoryItem) -> str:
    """Add semantic memory"""
    # 1. Generate text embedding
    embedding = self.embedding_model.encode(memory_item.content)

    # 2. Extract entities and relations
    entities = self._extract_entities(memory_item.content)
    relations = self._extract_relations(memory_item.content, entities)

    # 3. Store to Neo4j graph database
    for entity in entities:
        self._add_entity_to_graph(entity, memory_item)

    for relation in relations:
        self._add_relation_to_graph(relation, memory_item)

    # 4. Store to Qdrant vector database
    metadata = {
        "memory_id": memory_item.id,
        "entities": [e.entity_id for e in entities],
        "entity_count": len(entities),
        "relation_count": len(relations)
    }

    self.vector_store.add_vectors(
        vectors=[embedding.tolist()],
        metadata=[metadata],
        ids=[memory_item.id]
    )
```

의미 기억의 검색은 벡터 검색의 의미 이해 기능과 그래프 검색의 관계 추론 기능을 결합한 하이브리드 검색 전략을 구현합니다.

```python
def retrieve(self, query: str, limit: int = 5, **kwargs) -> List[MemoryItem]:
    """Retrieve semantic memory"""
    # 1. Vector retrieval
    vector_results = self._vector_search(query, limit * 2, user_id)

    # 2. Graph retrieval
    graph_results = self._graph_search(query, limit * 2, user_id)

    # 3. Hybrid ranking
    combined_results = self._combine_and_rank_results(
        vector_results, graph_results, query, limit
    )

    return combined_results[:limit]
```

하이브리드 순위 알고리즘은 다단계 채점 메커니즘을 채택합니다.

```python
def _combine_and_rank_results(self, vector_results, graph_results, query, limit):
    """Hybrid ranking of results"""
    combined = {}

    # Merge vector and graph retrieval results
    for result in vector_results:
        combined[result["memory_id"]] = {
            **result,
            "vector_score": result.get("score", 0.0),
            "graph_score": 0.0
        }

    for result in graph_results:
        memory_id = result["memory_id"]
        if memory_id in combined:
            combined[memory_id]["graph_score"] = result.get("similarity", 0.0)
        else:
            combined[memory_id] = {
                **result,
                "vector_score": 0.0,
                "graph_score": result.get("similarity", 0.0)
            }

    # Calculate hybrid score
    for memory_id, result in combined.items():
        vector_score = result["vector_score"]
        graph_score = result["graph_score"]
        importance = result.get("importance", 0.5)

        # Base relevance score
        base_relevance = vector_score * 0.7 + graph_score * 0.3

        # Importance weight [0.8, 1.2]
        importance_weight = 0.8 + (importance * 0.4)

        # Final score: similarity * importance weight
        combined_score = base_relevance * importance_weight
        result["combined_score"] = combined_score

    # Sort and return
    sorted_results = sorted(
        combined.values(),
        key=lambda x: x["combined_score"],
        reverse=True
    )

    return sorted_results[:limit]
```

의미기억의 점수 공식은 `(vector similarity × 0.7 + graph similarity × 0.3) × (0.8 + importance × 0.4)`입니다. 이 디자인의 핵심 아이디어는 다음과 같습니다.

- **벡터 검색 가중치 (0.7)**: 의미론적 유사성이 주요 요소로, 검색 결과가 쿼리와 의미론적으로 관련되어 있음을 보장합니다.
- **그래프 검색 가중치 (0.3)**: 보충으로서의 관계 추론, 개념 간의 암묵적 연관성 발견
- **중요도 가중치 범위 [0.8, 1.2]**: 중요도가 유사성 순위에 과도한 영향을 주지 않도록 하여 검색 정확도를 유지합니다.

(4) 지각기억

지각 기억은 텍스트, 이미지, 오디오와 같은 다양한 형식의 데이터 저장 및 검색을 지원합니다. 이는 양식 분리 저장 전략을 채택하여 다양한 양식의 데이터에 대한 독립적인 벡터 컬렉션을 생성합니다. 이 디자인은 검색 정확성을 보장하면서 차원 불일치 문제를 방지합니다.

````python
class PerceptualMemory(BaseMemory):
    """Perceptual memory implementation

    Features:
    - Supports multimodal data (text, images, audio, etc.)
    - Cross-modal similarity search
    - Semantic understanding of perceptual data
    - Supports content generation and retrieval
    """

    def __init__(self, config: MemoryConfig, storage_backend=None):
        super().__init__(config, storage_backend)

        # Multimodal encoders
        self.text_embedder = get_text_embedder()
        self._clip_model = self._init_clip_model()  # Image encoding
        self._clap_model = self._init_clap_model()  # Audio encoding

        # Modality-separated vector storage
        self.vector_stores = {
            "text": QdrantConnectionManager.get_instance(
                collection_name="perceptual_text",
                vector_size=self.vector_dim
            ),
            "image": QdrantConnectionManager.get_instance(
                collection_name="perceptual_image",
                vector_size=self._image_dim
            ),
            "audio": QdrantConnectionManager.get_instance(
                collection_name="perceptual_audio",
                vector_size=self._audio_dim
            )
        }
````

지각 기억 검색은 동일 양식과 교차 양식 모드를 모두 지원합니다. 동일 양식 검색은 정확한 일치를 위해 특수 인코더를 사용하는 반면, 교차 양식 검색에는 보다 복잡한 의미 체계 정렬 메커니즘이 필요합니다.

```python
def retrieve(self, query: str, limit: int = 5, **kwargs) -> List[MemoryItem]:
    """Retrieve perceptual memory (can filter modality; same-modality vector retrieval + time/importance fusion)"""
    user_id = kwargs.get("user_id")
    target_modality = kwargs.get("target_modality")
    query_modality = kwargs.get("query_modality", target_modality or "text")

    # Same-modality vector retrieval
    try:
        query_vector = self._encode_data(query, query_modality)
        store = self._get_vector_store_for_modality(target_modality or query_modality)

        where = {"memory_type": "perceptual"}
        if user_id:
            where["user_id"] = user_id
        if target_modality:
            where["modality"] = target_modality

        hits = store.search_similar(
            query_vector=query_vector,
            limit=max(limit * 5, 20),
            where=where
        )
    except Exception:
        hits = []

    # Fusion ranking (vector similarity + temporal recency + importance weight)
    results = []
    for hit in hits:
        vector_score = float(hit.get("score", 0.0))
        recency_score = self._calculate_recency_score(hit["metadata"]["timestamp"])
        importance = hit["metadata"].get("importance", 0.5)

        # Scoring algorithm
        base_relevance = vector_score * 0.8 + recency_score * 0.2
        importance_weight = 0.8 + (importance * 0.4)
        combined_score = base_relevance * importance_weight

        results.append((combined_score, self._create_memory_item(hit)))

    results.sort(key=lambda x: x[0], reverse=True)
    return [item for _, item in results[:limit]]
```

지각 기억의 점수 공식은 `(vector similarity × 0.8 + temporal recency × 0.2) × (0.8 + importance × 0.4)`입니다. 지각 메모리의 점수 매기기 메커니즘은 통합된 벡터 공간을 통해 텍스트, 이미지, 오디오와 같은 다양한 양식 데이터의 의미론적 정렬을 달성하는 교차 모드 검색도 지원합니다. 교차 모드 검색을 수행할 때 시스템은 검색 결과의 다양성과 정확성을 보장하기 위해 점수 가중치를 자동으로 조정합니다. 또한 지각 기억의 시간적 최근성 계산은 지수 붕괴 모델을 채택합니다.

```python
def _calculate_recency_score(self, timestamp: str) -> float:
    """Calculate temporal recency score"""
    try:
        memory_time = datetime.fromisoformat(timestamp)
        current_time = datetime.now()
        age_hours = (current_time - memory_time).total_seconds() / 3600

        # Exponential decay: maintain high score within 24 hours, then gradually decay
        decay_factor = 0.1  # Decay coefficient
        recency_score = math.exp(-decay_factor * age_hours / 24)

        return max(0.1, recency_score)  # Maintain minimum base score of 0.1
    except Exception:
        return 0.5  # Default medium score
```

이 시간 붕괴 모델은 인간 기억의 망각 곡선을 시뮬레이션하여 지각 기억 시스템이 시간적으로 더 관련성이 높은 기억 내용의 검색 우선순위를 지정할 수 있도록 보장합니다.

## 8.3 RAG 시스템: 지식 검색 강화

### 8.3.1 RAG 기초

HelloAgents의 RAG 시스템 구현에 대해 알아보기 전에 먼저 RAG 기술의 기본 개념과 개발 역사, 핵심 원리를 이해해 보겠습니다. 이 텍스트는 RAG를 기초로 작성되지 않았으므로 시스템 설계의 기술적 선택과 혁신을 더 잘 이해하기 위해 여기서 관련 개념을 빠르게 검토할 것입니다.

(1) RAG란 무엇입니까?

RAG(Retrieval-Augmented Generation)는 정보 검색과 텍스트 생성을 결합한 기술입니다. 핵심 아이디어는 답변을 생성하기 전에 먼저 외부 지식 기반에서 관련 정보를 검색한 다음 검색된 정보를 대규모 언어 모델에 대한 컨텍스트로 제공하여 보다 정확하고 신뢰할 수 있는 답변을 생성하는 것입니다.

따라서 검색-증강 생성(Retrieval-Augmented Generation)은 세 단어로 나눌 수 있습니다. **검색**은 지식 베이스에서 관련 콘텐츠를 쿼리하는 것을 의미합니다. **증강**은 모델 생성을 지원하기 위해 검색 결과를 프롬프트에 통합하는 것을 의미합니다. **세대**는 정확성과 투명성을 결합한 답변을 출력합니다.

(2) 기본 작업 흐름

전체 RAG 애플리케이션 워크플로는 주로 두 가지 핵심 단계로 나뉩니다. **데이터 준비 단계**에서 시스템은 **데이터 추출**, **텍스트 분할** 및 **벡터화**를 통해 검색 가능한 데이터베이스에 외부 지식을 구축합니다. 그 후 **애플리케이션 단계**에서 시스템은 사용자 **쿼리**에 응답하고 데이터베이스에서 관련 정보를 **검색**하고 **프롬프트에 삽입**한 후 마지막으로 대규모 언어 모델을 구동하여 **답변을 생성**합니다.

(3) 개발 연혁

첫 번째 단계: Naive RAG(2020-2021). 이것은 일반적으로 "검색-읽기" 모드라고 하는 직접적이고 간단한 프로세스를 갖춘 RAG 기술의 초기 단계입니다. **검색 방법**: 주로 `TF-IDF` 또는 `BM25`과 같은 전통적인 키워드 일치 알고리즘에 의존합니다. 이러한 방법은 용어 빈도와 문서 빈도를 계산하여 관련성을 평가하며 문자 일치 효과는 좋지만 의미적 유사성을 이해하는 데 어려움이 있습니다. **생성 모드**: 검색된 문서 콘텐츠를 처리하지 않고 프롬프트 컨텍스트에 직접 연결한 다음 생성 모델로 보냅니다.

두 번째 단계: 고급 RAG(2022-2023). 벡터 데이터베이스와 텍스트 임베딩 기술의 성숙으로 RAG는 급속한 개발 단계에 들어섰습니다. 연구원과 개발자는 "검색"과 "생성"의 다양한 단계에서 수많은 최적화 기술을 도입했습니다. **검색 방법**: **밀도 임베딩** 기반의 의미 검색으로 전환되었습니다. 모델은 텍스트를 고차원 벡터로 변환함으로써 키워드뿐만 아니라 의미론적 유사성을 이해하고 일치시킬 수 있습니다. **생성 모드**: 쿼리 재작성, 문서 청크, 순위 재지정 등과 같은 다양한 최적화 기술이 도입되었습니다.

3단계: 모듈형 RAG(2023년~현재). 고급 RAG를 기반으로 하는 최신 RAG 시스템은 모듈화, 자동화 및 지능을 향해 더욱 발전합니다. 시스템의 다양한 부분은 보다 다양하고 복잡한 애플리케이션 시나리오에 적응할 수 있도록 플러그 가능하고 구성 가능한 독립 모듈로 설계되었습니다. **검색 방법**: 하이브리드 검색, 다중 쿼리 확장, 가상 문서 임베딩 등 **생성 모드**: 사고 연쇄 추론, 자기 성찰 및 수정 등

### 8.3.2 RAG 시스템 작동 원리

구현 세부 사항을 살펴보기 전에 순서도를 사용하여 HelloAgents RAG 시스템의 전체 워크플로를 개략적으로 설명할 수 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-5.png" alt="RAG System Core Principle" width="85%"/>
<p>그림 8.5 RAG 시스템의 핵심 작동 원리</p>
</div>

그림 8.5에 표시된 대로 RAG 시스템의 두 가지 주요 작동 모드를 보여줍니다.
1. **데이터 처리 워크플로**: 지식 문서를 처리하고 저장합니다. 여기에서는 들어오는 모든 외부 지식 소스를 처리를 위해 마크다운 형식으로 일관되게 변환한다는 설계 아이디어로 도구 `Markitdown`를 채택했습니다.
2. **쿼리 및 생성 워크플로**: 쿼리를 기반으로 관련 정보를 검색하고 답변을 생성합니다.

### 8.3.3 빠른 경험: 30초 안에 RAG 기능 시작하기

RAG 시스템의 기본 기능을 빠르게 경험해 보겠습니다.

```python
from hello_agents import SimpleAgent, HelloAgentsLLM, ToolRegistry
from hello_agents.tools import RAGTool

# Create Agent with RAG capability
llm = HelloAgentsLLM()
agent = SimpleAgent(name="Knowledge Assistant", llm=llm)

# Create RAG tool
rag_tool = RAGTool(
    knowledge_base_path="./knowledge_base",
    collection_name="test_collection",
    rag_namespace="test"
)

tool_registry = ToolRegistry()
tool_registry.register_tool(rag_tool)
agent.tool_registry = tool_registry

# Experience RAG features
# Add first knowledge
result1 = rag_tool.execute("add_text",
    text="Python is a high-level programming language first released by Guido van Rossum in 1991. Python's design philosophy emphasizes code readability and concise syntax.",
    document_id="python_intro")
print(f"Knowledge 1: {result1}")

# Add second knowledge
result2 = rag_tool.execute("add_text",
    text="Machine learning is a branch of artificial intelligence that uses algorithms to enable computers to learn patterns from data. It mainly includes three types: supervised learning, unsupervised learning, and reinforcement learning.",
    document_id="ml_basics")
print(f"Knowledge 2: {result2}")

# Add third knowledge
result3 = rag_tool.execute("add_text",
    text="RAG (Retrieval-Augmented Generation) is an AI technology that combines information retrieval and text generation. It enhances the generation capability of large language models by retrieving relevant knowledge.",
    document_id="rag_concept")
print(f"Knowledge 3: {result3}")


print("\n=== Search Knowledge ===")
result = rag_tool.execute("search",
    query="History of Python programming language",
    limit=3,
    min_score=0.1
)
print(result)

print("\n=== Knowledge Base Statistics ===")
result = rag_tool.execute("stats")
print(result)
```

다음으로 HelloAgents RAG 시스템의 구체적인 구현을 살펴보겠습니다.

### 8.3.4 RAG 시스템 아키텍처 설계

이 섹션에서는 메모리 시스템 설명과 다른 접근 방식을 채택합니다. `Memory_tool`은 체계적인 구현인 반면, 우리 설계의 RAG는 파이프라인으로 구성할 수 있는 도구로 정의되기 때문입니다. RAG 시스템의 핵심 아키텍처는 "5계층 7단계" 설계 패턴으로 요약될 수 있습니다.

```
User Layer: RAGTool unified interface
  ↓
Application Layer: Intelligent Q&A, search, management
  ↓
Processing Layer: Document parsing, chunking, vectorization
  ↓
Storage Layer: Vector database, document storage
  ↓
Foundation Layer: Embedding model, LLM, database
```

이 계층형 설계의 장점은 전체 시스템의 안정성을 유지하면서 각 계층을 독립적으로 최적화하고 교체할 수 있다는 것입니다. 예를 들어, 상위 수준 비즈니스 로직에 영향을 주지 않고 임베딩 모델을 문장 변환기에서 Bailian API로 쉽게 전환할 수 있습니다. 마찬가지로 처리 워크플로 코드는 완전히 재사용 가능하며 필요한 부분을 선택하여 자신의 프로젝트에 넣을 수도 있습니다. RAGTool은 RAG 시스템의 통합 진입점 역할을 하며 간결한 API 인터페이스를 제공합니다.

````python
class RAGTool(Tool):
    """RAG tool

    Provides complete RAG capabilities:
    - Add multi-format documents (PDF, Office, images, audio, etc.)
    - Intelligent retrieval and recall
    - LLM-enhanced Q&A
    - Knowledge base management
    """

    def __init__(
        self,
        knowledge_base_path: str = "./knowledge_base",
        qdrant_url: str = None,
        qdrant_api_key: str = None,
        collection_name: str = "rag_knowledge_base",
        rag_namespace: str = "default"
    ):
        # Initialize RAG pipeline
        self._pipelines: Dict[str, Dict[str, Any]] = {}
        self.llm = HelloAgentsLLM()

        # Create default pipeline
        default_pipeline = create_rag_pipeline(
            qdrant_url=self.qdrant_url,
            qdrant_api_key=self.qdrant_api_key,
            collection_name=self.collection_name,
            rag_namespace=self.rag_namespace
        )
        self._pipelines[self.rag_namespace] = default_pipeline
````

전체 처리 작업 흐름은 다음과 같습니다.
```
Any format document → MarkItDown conversion → Markdown text → Intelligent chunking → Vectorization → Storage and retrieval
```

(1) 다중 모드 문서 로딩

RAG 시스템의 핵심 장점 중 하나는 강력한 다중 모드 문서 처리 기능입니다. 시스템은 MarkItDown을 통합 문서 변환 엔진으로 사용하여 거의 모든 일반적인 문서 형식을 지원합니다. MarkItDown은 Microsoft의 오픈 소스 범용 문서 변환 도구입니다. 이는 모든 형식의 문서를 구조화된 마크다운 텍스트로 균일하게 변환하는 역할을 하는 HelloAgents RAG 시스템의 핵심 구성 요소입니다. 입력이 PDF, Word, Excel, 이미지 또는 오디오인지 여부는 궁극적으로 표준 Markdown 형식으로 변환된 다음 통합 청크, 벡터화 및 저장 워크플로로 들어갑니다.

```python
def _convert_to_markdown(path: str) -> str:
    """
    Universal document reader using MarkItDown with enhanced PDF processing.
    Core function: Convert documents of any format to Markdown text

    Supported formats:
    - Documents: PDF, Word, Excel, PowerPoint
    - Images: JPG, PNG, GIF (via OCR)
    - Audio: MP3, WAV, M4A (via transcription)
    - Text: TXT, CSV, JSON, XML, HTML
    - Code: Python, JavaScript, Java, etc.
    """
    if not os.path.exists(path):
        return ""

    # Use enhanced processing for PDF files
    ext = (os.path.splitext(path)[1] or '').lower()
    if ext == '.pdf':
        return _enhanced_pdf_processing(path)

    # Use MarkItDown unified conversion for other formats
    md_instance = _get_markitdown_instance()
    if md_instance is None:
        return _fallback_text_reader(path)

    try:
        result = md_instance.convert(path)
        markdown_text = getattr(result, "text_content", None)
        if isinstance(markdown_text, str) and markdown_text.strip():
            print(f"[RAG] MarkItDown conversion successful: {path} -> {len(markdown_text)} chars Markdown")
            return markdown_text
        return ""
    except Exception as e:
        print(f"[WARNING] MarkItDown conversion failed {path}: {e}")
        return _fallback_text_reader(path)
```

(2) 지능형 청킹 전략

MarkItDown 변환 후 모든 문서는 표준 Markdown 형식으로 통합됩니다. 이는 후속 지능형 청킹을 위한 구조화된 기반을 제공합니다. HelloAgents는 정확한 분할을 위해 Markdown의 구조적 특성을 최대한 활용하여 Markdown 형식에 맞게 특별히 지능적인 청킹 전략을 구현합니다.

마크다운 구조 인식 청크 작업 흐름:

```
Standard Markdown text → Heading hierarchy parsing → Paragraph semantic segmentation → Token calculation chunking → Overlap strategy optimization → Vectorization preparation
       ↓                ↓              ↓            ↓           ↓            ↓
   Unified format      #/##/###      Semantic boundary  Size control  Information continuity  Embedding vector
   Clear structure     Hierarchy recognition  Integrity guarantee  Retrieval optimization  Context preservation  Similarity matching
```

모든 문서가 Markdown 형식으로 변환되었으므로 시스템은 정확한 의미론적 분할을 위해 Markdown의 제목 구조 (#, ##, ###, etc.)를 사용할 수 있습니다.

```python
def _split_paragraphs_with_headings(text: str) -> List[Dict]:
    """Split paragraphs based on heading hierarchy, maintaining semantic integrity"""
    lines = text.splitlines()
    heading_stack: List[str] = []
    paragraphs: List[Dict] = []
    buf: List[str] = []
    char_pos = 0

    def flush_buf(end_pos: int):
        if not buf:
            return
        content = "\n".join(buf).strip()
        if not content:
            return
        paragraphs.append({
            "content": content,
            "heading_path": " > ".join(heading_stack) if heading_stack else None,
            "start": max(0, end_pos - len(content)),
            "end": end_pos,
        })

    for ln in lines:
        raw = ln
        if raw.strip().startswith("#"):
            # Process heading line
            flush_buf(char_pos)
            level = len(raw) - len(raw.lstrip('#'))
            title = raw.lstrip('#').strip()

            if level <= 0:
                level = 1
            if level <= len(heading_stack):
                heading_stack = heading_stack[:level-1]
            heading_stack.append(title)

            char_pos += len(raw) + 1
            continue

        # Accumulate paragraph content
        if raw.strip() == "":
            flush_buf(char_pos)
            buf = []
        else:
            buf.append(raw)
        char_pos += len(raw) + 1

    flush_buf(char_pos)

    if not paragraphs:
        paragraphs = [{"content": text, "heading_path": None, "start": 0, "end": len(text)}]

    return paragraphs
```

Markdown 단락 분할을 기반으로 시스템은 토큰 수에 따라 지능형 청크를 추가로 수행합니다. 입력이 이미 구조화된 Markdown 텍스트이므로 시스템은 청크 경계를 보다 정확하게 제어하여 각 청크가 벡터화 처리에 적합하고 Markdown 구조의 무결성을 유지하도록 보장할 수 있습니다.

```python
def _chunk_paragraphs(paragraphs: List[Dict], chunk_tokens: int, overlap_tokens: int) -> List[Dict]:
    """Intelligent chunking based on token count"""
    chunks: List[Dict] = []
    cur: List[Dict] = []
    cur_tokens = 0
    i = 0

    while i < len(paragraphs):
        p = paragraphs[i]
        p_tokens = _approx_token_len(p["content"]) or 1

        if cur_tokens + p_tokens <= chunk_tokens or not cur:
            cur.append(p)
            cur_tokens += p_tokens
            i += 1
        else:
            # Generate current chunk
            content = "\n\n".join(x["content"] for x in cur)
            start = cur[0]["start"]
            end = cur[-1]["end"]
            heading_path = next((x["heading_path"] for x in reversed(cur) if x.get("heading_path")), None)

            chunks.append({
                "content": content,
                "start": start,
                "end": end,
                "heading_path": heading_path,
            })

            # Build overlap section
            if overlap_tokens > 0 and cur:
                kept: List[Dict] = []
                kept_tokens = 0
                for x in reversed(cur):
                    t = _approx_token_len(x["content"]) or 1
                    if kept_tokens + t > overlap_tokens:
                        break
                    kept.append(x)
                    kept_tokens += t
                cur = list(reversed(kept))
                cur_tokens = kept_tokens
            else:
                cur = []
                cur_tokens = 0

    # Process last chunk
    if cur:
        content = "\n\n".join(x["content"] for x in cur)
        start = cur[0]["start"]
        end = cur[-1]["end"]
        heading_path = next((x["heading_path"] for x in reversed(cur) if x.get("heading_path")), None)

        chunks.append({
            "content": content,
            "start": start,
            "end": end,
            "heading_path": heading_path,
        })

    return chunks
```

동시에, 다른 언어와 호환되도록 시스템은 청크 크기를 정확하게 제어하는 ​​데 중요한 중국어-영어 혼합 텍스트에 대한 토큰 추정 알고리즘을 구현합니다.

```python
def _approx_token_len(text: str) -> int:
    """Approximate token length estimation, supports Chinese-English mixed text"""
    # CJK characters counted as 1 token each
    cjk = sum(1 for ch in text if _is_cjk(ch))
    # Other characters counted by whitespace tokenization
    non_cjk_tokens = len([t for t in text.split() if t])
    return cjk + non_cjk_tokens

def _is_cjk(ch: str) -> bool:
    """Determine if character is CJK"""
    code = ord(ch)
    return (
        0x4E00 <= code <= 0x9FFF or  # CJK Unified Ideographs
        0x3400 <= code <= 0x4DBF or  # CJK Extension A
        0x20000 <= code <= 0x2A6DF or # CJK Extension B
        0x2A700 <= code <= 0x2B73F or # CJK Extension C
        0x2B740 <= code <= 0x2B81F or # CJK Extension D
        0x2B820 <= code <= 0x2CEAF or # CJK Extension E
        0xF900 <= code <= 0xFAFF      # CJK Compatibility Ideographs
    )
```

(3) 통합 임베딩 및 벡터 저장

임베딩 모델은 RAG 시스템의 핵심입니다. 텍스트를 고차원 벡터로 변환하여 컴퓨터가 텍스트의 의미론적 유사성을 이해하고 비교할 수 있도록 합니다. RAG 시스템의 검색 기능은 임베딩 모델의 품질과 벡터 저장 효율성에 크게 좌우됩니다. HelloAgents는 통합 임베딩 인터페이스를 구현합니다. 데모 목적으로 여기에서는 Bailian API를 사용합니다. 아직 구성되지 않은 경우 로컬 `all-MiniLM-L6-v2` 모델로 전환할 수 있습니다. 두 솔루션이 모두 지원되지 않는 경우 TF-IDF 알고리즘도 대체 솔루션으로 구성됩니다. 실제 사용시에는 원하는 모델이나 API로 대체하거나, 프레임워크 내용을 확장해보시면 됩니다~

```python
def index_chunks(
    store = None,
    chunks: List[Dict] = None,
    cache_db: Optional[str] = None,
    batch_size: int = 64,
    rag_namespace: str = "default"
) -> None:
    """
    Index markdown chunks with unified embedding and Qdrant storage.
    Uses Bailian API with fallback to sentence-transformers.
    """
    if not chunks:
        print("[RAG] No chunks to index")
        return

    # Use unified embedding model
    embedder = get_text_embedder()
    dimension = get_dimension(384)

    # Create default Qdrant storage
    if store is None:
        store = _create_default_vector_store(dimension)
        print(f"[RAG] Created default Qdrant store with dimension {dimension}")

    # Preprocess Markdown text for better embedding quality
    processed_texts = []
    for c in chunks:
        raw_content = c["content"]
        processed_content = _preprocess_markdown_for_embedding(raw_content)
        processed_texts.append(processed_content)

    print(f"[RAG] Embedding start: total_texts={len(processed_texts)} batch_size={batch_size}")

    # Batch encoding
    vecs: List[List[float]] = []
    for i in range(0, len(processed_texts), batch_size):
        part = processed_texts[i:i+batch_size]
        try:
            # Use unified embedder (handles caching internally)
            part_vecs = embedder.encode(part)

            # Standardize to List[List[float]] format
            if not isinstance(part_vecs, list):
                if hasattr(part_vecs, "tolist"):
                    part_vecs = [part_vecs.tolist()]
                else:
                    part_vecs = [list(part_vecs)]

            # Process vector format and dimension
            for v in part_vecs:
                try:
                    if hasattr(v, "tolist"):
                        v = v.tolist()
                    v_norm = [float(x) for x in v]

                    # Dimension check and adjustment
                    if len(v_norm) != dimension:
                        print(f"[WARNING] Vector dimension anomaly: expected {dimension}, actual {len(v_norm)}")
                        if len(v_norm) < dimension:
                            v_norm.extend([0.0] * (dimension - len(v_norm)))
                        else:
                            v_norm = v_norm[:dimension]

                    vecs.append(v_norm)
                except Exception as e:
                    print(f"[WARNING] Vector conversion failed: {e}, using zero vector")
                    vecs.append([0.0] * dimension)

        except Exception as e:
            print(f"[WARNING] Batch {i} encoding failed: {e}")
            # Implement retry mechanism
            # ... retry logic ...

        print(f"[RAG] Embedding progress: {min(i+batch_size, len(processed_texts))}/{len(processed_texts)}")
```

8.3.5 고급 검색 전략

RAG 시스템의 검색 능력은 핵심 경쟁력이다. 실제 응용 프로그램에서는 사용자 쿼리와 문서의 실제 내용 사이에 표현 차이가 있어 관련 문서가 검색되지 않을 수 있습니다. 이 문제를 해결하기 위해 HelloAgents는 MQE(Multi-Query Expansion), HyDE(Hypothetical Document Embedding) 및 통합 확장 검색 프레임워크라는 세 가지 보완적인 고급 검색 전략을 구현합니다.

(1) 다중 쿼리 확장(MQE)

MQE(Multi-Query Expansion)는 의미상 동일한 다양한 쿼리를 생성하여 검색 재현율을 향상시키는 기술입니다. 이 방법의 핵심 통찰력은 동일한 질문이 여러 가지 다른 표현을 가질 수 있고, 다른 표현이 다른 관련 문서와 일치할 수 있다는 것입니다. 예를 들어, "파이썬을 배우는 방법"은 "파이썬 초보자 튜토리얼", "파이썬 학습 방법", "파이썬 프로그래밍 가이드", 기타 쿼리로 확장될 수 있습니다. 이러한 확장된 쿼리를 병렬로 실행하고 결과를 병합함으로써 시스템은 단어 차이로 인해 중요한 정보가 누락되는 것을 방지하면서 더 넓은 범위의 관련 문서를 다룰 수 있습니다.

MKE의 장점은 사용자 쿼리의 가능한 여러 의미를 자동으로 이해할 수 있다는 점이며, 특히 모호한 쿼리나 전문 용어 쿼리에 효과적입니다. 시스템은 LLM을 사용하여 확장된 쿼리를 생성하여 확장의 다양성과 의미적 관련성을 보장합니다.

```python
def _prompt_mqe(query: str, n: int) -> List[str]:
    """Use LLM to generate diverse query expansions"""
    try:
        from ...core.llm import HelloAgentsLLM
        llm = HelloAgentsLLM()
        prompt = [
            {"role": "system", "content": "You are a retrieval query expansion assistant. Generate semantically equivalent or complementary diverse queries. Use Chinese, keep it short, avoid punctuation."},
            {"role": "user", "content": f"Original query: {query}\nPlease provide {n} differently phrased queries, one per line."}
        ]
        text = llm.invoke(prompt)
        lines = [ln.strip("- \t") for ln in (text or "").splitlines()]
        outs = [ln for ln in lines if ln]
        return outs[:n] or [query]
    except Exception:
        return [query]
```

(2) 가설 문서 임베딩(HyDE)

HyDE(Hypothetical Document Embedding)는 혁신적인 검색 기술입니다. 핵심 아이디어는 "답변을 사용하여 답을 찾는 것"입니다. 전통적인 검색 방법은 질문을 사용하여 문서를 일치시키지만 의미 공간에서 질문과 답변의 분포에 차이가 있는 경우가 많습니다. 즉, 질문은 일반적으로 의문문이고 문서 내용은 서술문입니다. HyDE는 LLM이 먼저 가상 답변 단락을 생성한 다음 이 답변 단락을 사용하여 실제 문서를 검색함으로써 쿼리와 문서 간의 의미적 격차를 줄입니다.

이 방법의 장점은 의미 공간에서 가상 답변이 실제 답변에 더 가깝기 때문에 관련 문서와 보다 정확한 매칭이 가능하다는 것입니다. 가설 답변의 내용이 완전히 정확하지 않더라도 여기에 포함된 핵심 용어, 개념 및 표현 스타일은 검색 시스템이 올바른 문서를 찾을 수 있도록 효과적으로 안내할 수 있습니다. 특히 전문 도메인 쿼리의 경우 HyDE는 도메인 용어가 포함된 가상 문서를 생성하여 검색 정확도를 크게 향상시킬 수 있습니다.

```python
def _prompt_hyde(query: str) -> Optional[str]:
    """Generate hypothetical document to improve retrieval"""
    try:
        from ...core.llm import HelloAgentsLLM
        llm = HelloAgentsLLM()
        prompt = [
            {"role": "system", "content": "Based on the user's question, first write a possible answer paragraph for use as a query document in vector retrieval (no analysis process)."},
            {"role": "user", "content": f"Question: {query}\nPlease directly write a medium-length, objective paragraph containing key terminology."}
        ]
        return llm.invoke(prompt)
    except Exception:
        return None
```

(3) 확장된 검색 프레임워크

HelloAgents는 MQE와 HyDE의 두 가지 전략을 통합 확장 검색 프레임워크로 통합합니다. 시스템을 통해 사용자는 `enable_mqe` 및 `enable_hyde` 매개변수를 통해 특정 시나리오를 기반으로 활성화할 전략을 선택할 수 있습니다. 높은 재현율이 필요한 시나리오의 경우 두 전략을 동시에 활성화할 수 있습니다. 성능에 민감한 시나리오의 경우 기본 검색만 사용할 수 있습니다.

확장 검색의 핵심 메커니즘은 3단계 "확장-검색-병합" 작업 흐름입니다. 첫째, 시스템은 원본 쿼리(MKE에서 생성된 다양한 쿼리 및 HyDE에서 생성된 가상 문서 포함)를 기반으로 여러 확장된 쿼리를 생성합니다. 그런 다음 확장된 각 쿼리에 대해 병렬로 벡터 검색을 실행하여 후보 문서 풀을 얻습니다. 마지막으로 중복 제거 및 점수 정렬을 통해 모든 결과를 병합하여 가장 관련성이 높은 상위 k 문서를 반환합니다. 이 디자인의 독창성은 `candidate_pool_multiplier` 매개변수(기본값은 4)를 통해 후보 풀을 확장하여 심사를 위한 충분한 후보 문서를 보장하는 동시에 지능형 중복 제거를 통해 중복 콘텐츠 반환을 방지한다는 것입니다.

```python
def search_vectors_expanded(
    store = None,
    query: str = "",
    top_k: int = 8,
    rag_namespace: Optional[str] = None,
    only_rag_data: bool = True,
    score_threshold: Optional[float] = None,
    enable_mqe: bool = False,
    mqe_expansions: int = 2,
    enable_hyde: bool = False,
    candidate_pool_multiplier: int = 4,
) -> List[Dict]:
    """
    Search with query expansion using unified embedding and Qdrant.
    """
    if not query:
        return []

    # Create default storage
    if store is None:
        store = _create_default_vector_store()

    # Query expansion
    expansions: List[str] = [query]

    if enable_mqe and mqe_expansions > 0:
        expansions.extend(_prompt_mqe(query, mqe_expansions))
    if enable_hyde:
        hyde_text = _prompt_hyde(query)
        if hyde_text:
            expansions.append(hyde_text)

    # Deduplication and trimming
    uniq: List[str] = []
    for e in expansions:
        if e and e not in uniq:
            uniq.append(e)
    expansions = uniq[: max(1, len(uniq))]

    # Allocate candidate pool
    pool = max(top_k * candidate_pool_multiplier, 20)
    per = max(1, pool // max(1, len(expansions)))

    # Build RAG data filter
    where = {"memory_type": "rag_chunk"}
    if only_rag_data:
        where["is_rag_data"] = True
        where["data_source"] = "rag_pipeline"
    if rag_namespace:
        where["rag_namespace"] = rag_namespace

    # Collect results from all expanded queries
    agg: Dict[str, Dict] = {}
    for q in expansions:
        qv = embed_query(q)
        hits = store.search_similar(
            query_vector=qv,
            limit=per,
            score_threshold=score_threshold,
            where=where
        )
        for h in hits:
            mid = h.get("metadata", {}).get("memory_id", h.get("id"))
            s = float(h.get("score", 0.0))
            if mid not in agg or s > float(agg[mid].get("score", 0.0)):
                agg[mid] = h

    # Sort by score and return
    merged = list(agg.values())
    merged.sort(key=lambda x: float(x.get("score", 0.0)), reverse=True)
    return merged[:top_k]
```

실제 적용에서는 이 세 가지 전략을 결합하여 사용하는 것이 가장 효과적입니다. MQE는 표현 다양성 문제를 처리하는 데 탁월하고 HyDE는 의미 격차 문제를 처리하는 데 탁월하며 통합 프레임워크는 결과 품질과 다양성을 보장합니다. 일반 쿼리의 경우 MAE를 활성화하는 것이 좋습니다. 전문 도메인 쿼리의 경우 MQE와 HyDE를 동시에 활성화하는 것이 좋습니다. 성능에 민감한 시나리오의 경우 기본 검색 또는 MAE만 사용할 수 있습니다.

물론 다른 흥미로운 방법도 많이 있습니다. 이것은 모든 사람에게 적합한 확장 기능 소개입니다. 실제 사용 시나리오에서는 문제에 적합한 솔루션을 찾으려고 노력해야 합니다.

## 8.4 지능형 문서 Q&A 도우미 구축

이전 섹션에서는 HelloAgents의 메모리 시스템과 RAG 시스템의 설계 및 구현에 대해 자세히 설명했습니다. 이제 이 두 시스템을 유기적으로 결합하여 지능형 문서 Q&A 도우미를 구축하는 방법을 완전한 실제 사례를 통해 시연해 보겠습니다.

### 8.4.1 사례 배경 및 목적

실제 업무에서는 수많은 기술 문서, 연구 논문, 제품 매뉴얼, 기타 PDF 파일을 처리해야 하는 경우가 많습니다. 기존의 문서 읽기 방법은 비효율적이어서 지식 간의 연관성을 설정하는 것은 물론 핵심 정보를 빠르게 찾는 것도 어렵습니다.

이 사례에서는 Datawhale의 또 다른 대규모 실습 튜토리얼 Happy-LLM의 공개 베타 PDF 문서 `Happy-LLM-0727.pdf`를 **Gradio 기반 웹 애플리케이션** 구축을 위한 예로 사용하고, RAGTool 및 MemoryTool을 사용하여 완전한 대화형 학습 도우미를 구축하는 방법을 보여줍니다. PDF는 이 [링크](https://github.com/datawhalechina/happy-llm/releases/download/v1.0.1/Happy-LLM-0727.pdf).)에서 얻을 수 있습니다.

우리는 다음 기능을 구현하기를 희망합니다:

1. **지능형 문서 처리**: MarkItDown을 사용하여 PDF에서 Markdown으로의 통합 변환, Markdown 구조를 기반으로 한 지능형 청킹 전략, 효율적인 벡터화 및 인덱스 구성을 달성합니다.

2. **고급 검색 Q&A**: 재현율을 향상시키는 MQE(Multi-Query Expansion), 검색 정확성을 향상시키는 HyDE(Hypothetical Document Embedding), 상황 인식 지능형 Q&A

3. **다단계 메모리 관리**: 작업 메모리는 현재 학습 작업과 맥락을 관리하고, 에피소드 메모리는 학습 이벤트와 쿼리 기록을 기록하고, 의미 메모리는 개념적 지식과 이해를 저장하고, 지각 메모리는 문서 기능과 다중 모드 정보를 처리합니다.

4. **맞춤형 학습 지원**: 학습 이력, 기억 통합 및 선택적 망각, 학습 보고서 생성 및 진행 상황 추적을 기반으로 한 맞춤형 추천

전체 시스템의 작업 흐름을 보다 명확하게 보여주기 위해 그림 8.6에서는 5단계 간의 관계와 데이터 흐름을 보여줍니다. 5단계는 완전한 폐쇄 루프를 형성합니다. 1단계에서는 처리된 PDF 문서의 정보를 메모리 시스템에 기록하고, 2단계의 검색 결과도 메모리 시스템에 기록합니다. 3단계에서는 메모리 시스템의 전체 기능(추가, 검색, 통합, 삭제)을 보여주고, 4단계에서는 RAG와 메모리를 통합하여 지능형 라우팅을 제공하며, 5단계에서는 모든 통계 정보를 수집하여 학습 보고서를 생성합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-6.png" alt="" width="85%"/>
<p>그림 8.6 지능형 Q&A 도우미의 5단계 실행 워크플로</p>
</div>

다음으로 이 웹 애플리케이션을 구현하는 방법을 보여드리겠습니다. 전체 애플리케이션은 세 가지 핵심 부분으로 나뉩니다.

1. **핵심 보조 클래스(PDFLearningAssistant)**: RAGTool 및 MemoryTool의 호출 논리를 캡슐화합니다.
2. **Gradio Web Interface**: 친숙한 사용자 상호작용 인터페이스를 제공합니다. 이 부분은 예제 코드를 참조하여 학습할 수 있습니다.
3. **기타 핵심 기능**: 노트 기록, 학습 복습, 통계 보기, 보고서 생성

### 8.4.2 핵심 보조 클래스 구현

먼저 RAGTool 및 MemoryTool의 호출 논리를 캡슐화하는 핵심 보조 클래스 `PDFLearningAssistant`를 구현합니다.

(1) 클래스 초기화

```python
class PDFLearningAssistant:
    """Intelligent document Q&A assistant"""

    def __init__(self, user_id: str = "default_user"):
        """Initialize learning assistant

        Args:
            user_id: User ID, used to isolate data for different users
        """
        self.user_id = user_id
        self.session_id = f"session_{datetime.now().strftime('%Y%m%d_%H%M%S')}"

        # Initialize tools
        self.memory_tool = MemoryTool(user_id=user_id)
        self.rag_tool = RAGTool(rag_namespace=f"pdf_{user_id}")

        # Learning statistics
        self.stats = {
            "session_start": datetime.now(),
            "documents_loaded": 0,
            "questions_asked": 0,
            "concepts_learned": 0
        }

        # Currently loaded document
        self.current_document = None
```

이 초기화 프로세스에서 우리는 몇 가지 주요 설계 결정을 내렸습니다.

**MemoryTool 초기화**: `user_id` 매개변수를 통해 사용자 수준 메모리 격리를 구현합니다. 다양한 사용자의 학습 기억은 완전히 독립적이며, 각 사용자는 자신만의 작업 기억, 일화 기억, 의미 기억, 지각 기억 공간을 가지고 있습니다.

**RAGTool 초기화**: `rag_namespace` 매개변수를 통해 기술 자료 네임스페이스 격리를 구현합니다. `f"pdf_{user_id}"`을 네임스페이스로 사용하면 각 사용자는 자신만의 독립적인 PDF 지식 기반을 갖게 됩니다.

**세션 관리**: `session_id`는 단일 학습 세션의 전체 프로세스를 추적하는 데 사용되어 후속 학습 여정 검토 및 분석을 용이하게 합니다.

**통계 정보**: `stats` 사전은 학습 보고서 생성을 위한 주요 학습 측정항목을 기록합니다.

(2) PDF 문서 로드

```python
def load_document(self, pdf_path: str) -> Dict[str, Any]:
    """Load PDF document into knowledge base

    Args:
        pdf_path: PDF file path

    Returns:
        Dict: Result containing success and message
    """
    if not os.path.exists(pdf_path):
        return {"success": False, "message": f"File does not exist: {pdf_path}"}

    start_time = time.time()

    # [RAGTool] Process PDF: MarkItDown conversion → Intelligent chunking → Vectorization
    result = self.rag_tool.execute(
        "add_document",
        file_path=pdf_path,
        chunk_size=1000,
        chunk_overlap=200
    )

    process_time = time.time() - start_time

    if result.get("success", False):
        self.current_document = os.path.basename(pdf_path)
        self.stats["documents_loaded"] += 1

        # [MemoryTool] Record to learning memory
        self.memory_tool.execute(
            "add",
            content=f"Loaded document 《{self.current_document}》",
            memory_type="episodic",
            importance=0.9,
            event_type="document_loaded",
            session_id=self.session_id
        )

        return {
            "success": True,
            "message": f"Loading successful! (Time: {process_time:.1f}s)",
            "document": self.current_document
        }
    else:
        return {
            "success": False,
            "message": f"Loading failed: {result.get('error', 'Unknown error')}"
        }
```

단 한 줄의 코드로 PDF 처리를 완료할 수 있습니다.

```python
result = self.rag_tool.execute(
    "add_document",
    file_path=pdf_path,
    chunk_size=1000,
    chunk_overlap=200
)
```

이 호출은 RAGTool의 전체 처리 워크플로(MarkItDown 변환, 향상된 처리, 지능형 청크, 벡터화 저장)를 트리거합니다. 이러한 내부 세부 사항은 섹션 8.3에서 자세히 소개되었습니다. 우리는 다음 사항에만 집중하면 됩니다:

- **작업 유형**: `"add_document"` - 기술 자료에 문서 추가
- **파일 경로**: `file_path` - PDF 파일 경로
- **청킹 매개변수**: `chunk_size=1000, chunk_overlap=200` - 텍스트 청킹 제어
- **반환결과** : 처리현황 및 통계정보를 담은 사전

문서가 성공적으로 로드된 후 MemoryTool을 사용하여 이를 에피소드 메모리에 기록합니다.

```python
self.memory_tool.execute(
    "add",
    content=f"Loaded document 《{self.current_document}》",
    memory_type="episodic",
    importance=0.9,
    event_type="document_loaded",
    session_id=self.session_id
)
```

**일화 메모리를 사용하는 이유는 무엇입니까?** 이것은 특정 타임스탬프가 있는 이벤트이기 때문에 일화 메모리를 사용하여 기록하는 데 적합합니다. `session_id` 매개변수는 이 이벤트를 현재 학습 세션과 연결하여 학습 여정에 대한 후속 검토를 용이하게 합니다.

이 메모리 기록은 후속 맞춤형 서비스의 기반을 마련합니다.

- 사용자가 "이전에 어떤 문서를 로드했습니까?"라고 묻습니다. → 일화 기억에서 검색
- 시스템은 사용자의 학습 여정과 문서 사용을 추적할 수 있습니다.

### 8.4.3 지능형 Q&A 기능

문서가 로드된 후 사용자는 문서에 대해 질문할 수 있습니다. 사용자 질문을 처리하기 위해 `ask` 메소드를 구현합니다.

```python
def ask(self, question: str, use_advanced_search: bool = True) -> str:
    """Ask questions about the document

    Args:
        question: User question
        use_advanced_search: Whether to use advanced retrieval (MQE + HyDE)

    Returns:
        str: Answer
    """
    if not self.current_document:
        return "⚠️ Please load a document first!"

    # [MemoryTool] Record question to working memory
    self.memory_tool.execute(
        "add",
        content=f"Question: {question}",
        memory_type="working",
        importance=0.6,
        session_id=self.session_id
    )

    # [RAGTool] Use advanced retrieval to get answer
    answer = self.rag_tool.execute(
        "ask",
        question=question,
        limit=5,
        enable_advanced_search=use_advanced_search,
        enable_mqe=use_advanced_search,
        enable_hyde=use_advanced_search
    )

    # [MemoryTool] Record to episodic memory
    self.memory_tool.execute(
        "add",
        content=f"Learning about '{question}'",
        memory_type="episodic",
        importance=0.7,
        event_type="qa_interaction",
        session_id=self.session_id
    )

    self.stats["questions_asked"] += 1

    return answer
```

`self.rag_tool.execute("ask", ...)`을 호출하면 RAGTool은 내부적으로 다음과 같은 고급 검색 워크플로를 실행합니다.

1. **다중 쿼리 확장(MQE)**:

   ```python
   # Generate diverse queries
   expanded_queries = self._generate_multi_queries(question)
   # For example, for "What is a large language model?", it might generate:
   # - "What is the definition of a large language model?"
   # - "Please explain large language models"
   # - "What does LLM mean?"
   ```

MQE는 LLM을 통해 의미상 동일하지만 다르게 표현된 쿼리를 생성하여 여러 각도에서 사용자 의도를 이해하고 재현율을 30%-50% 향상시킵니다.

2. **가설 문서 삽입(HyDE)**:

- 쿼리와 문서 사이의 의미적 격차를 해소하여 가상의 답변 문서를 생성합니다.
   - 검색을 위해 가상 답변 벡터 사용

이러한 고급 검색 기술의 내부 구현은 섹션 8.3.5에서 자세히 소개되었습니다.

### 8.4.4 기타 핵심 기능

문서 및 지능형 Q&A를 로드하는 것 외에도 메모 기록, 학습 검토, 통계 보기 및 보고서 생성과 같은 기능도 구현해야 합니다.

```python
def add_note(self, content: str, concept: Optional[str] = None):
    """Add learning note"""
    self.memory_tool.execute(
        "add",
        content=content,
        memory_type="semantic",
        importance=0.8,
        concept=concept or "general",
        session_id=self.session_id
    )
    self.stats["concepts_learned"] += 1

def recall(self, query: str, limit: int = 5) -> str:
    """Review learning journey"""
    result = self.memory_tool.execute(
        "search",
        query=query,
        limit=limit
    )
    return result

def get_stats(self) -> Dict[str, Any]:
    """Get learning statistics"""
    duration = (datetime.now() - self.stats["session_start"]).total_seconds()
    return {
        "Session Duration": f"{duration:.0f}s",
        "Documents Loaded": self.stats["documents_loaded"],
        "Questions Asked": self.stats["questions_asked"],
        "Learning Notes": self.stats["concepts_learned"],
        "Current Document": self.current_document or "Not loaded"
    }

def generate_report(self, save_to_file: bool = True) -> Dict[str, Any]:
    """Generate learning report"""
    memory_summary = self.memory_tool.execute("summary", limit=10)
    rag_stats = self.rag_tool.execute("stats")

    duration = (datetime.now() - self.stats["session_start"]).total_seconds()
    report = {
        "session_info": {
            "session_id": self.session_id,
            "user_id": self.user_id,
            "start_time": self.stats["session_start"].isoformat(),
            "duration_seconds": duration
        },
        "learning_metrics": {
            "documents_loaded": self.stats["documents_loaded"],
            "questions_asked": self.stats["questions_asked"],
            "concepts_learned": self.stats["concepts_learned"]
        },
        "memory_summary": memory_summary,
        "rag_status": rag_stats
    }

    if save_to_file:
        report_file = f"learning_report_{self.session_id}.json"
        with open(report_file, 'w', encoding='utf-8') as f:
            json.dump(report, f, ensure_ascii=False, indent=2, default=str)
        report["report_file"] = report_file

    return report
```

이러한 메서드는 각각 다음을 구현합니다.

- **add_note**: 학습 노트를 의미 기억에 저장합니다.
- **recall**: 기억 시스템에서 학습 여정을 검색합니다.
- **get_stats**: 현재 세션의 통계 정보를 가져옵니다.
- **generate_report**: 상세 학습 보고서를 생성하고 JSON 파일로 저장합니다.

### 8.4.5 실행 효과 시연

다음은 런닝 효과 시연입니다. 그림 8.7에서 볼 수 있듯이 메인 페이지에 들어간 후 먼저 데이터베이스, 모델, API 및 기타 로딩 작업을 로드하는 어시스턴트를 초기화해야 합니다. 그런 다음 PDF 문서를 전달하고 클릭하여 문서를 로드합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-7.png" alt="" width="85%"/>
<p>그림 8.7 Q&A 도우미 메인 페이지</p>
</div>

첫 번째 기능은 업로드된 문서를 기반으로 검색하고 관련 자료의 참조 소스와 유사성 계산을 반환할 수 있는 지능형 Q&A입니다. 이는 그림 8.8에 표시된 대로 RAG 도구 기능을 보여줍니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-8.png" alt="" width="85%"/>
<p>그림 8.8 Q&A 도우미 메인 페이지</p>
</div>

두 번째 기능은 학습 노트입니다. 그림 8.9와 같이 관련 개념을 선택하여 메모 내용을 작성할 수 있다. 이 부분은 메모리 도구를 사용하며, 쉬운 통계 및 전체 학습 보고서의 후속 반환을 위해 데이터베이스에 개인 메모를 저장합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-9.png" alt="" width="85%"/>
<p>그림 8.9 Q&A 도우미 메인 페이지</p>
</div>

마지막으로 학습 진행 상황 및 보고서 생성에 대한 통계가 있습니다. 그림 8.10과 같이 어시스턴트 사용 중 로드된 문서 수, 질문 수, 메모 수를 확인할 수 있습니다. 마지막으로 Q&A 결과와 메모가 JSON 문서로 구성되어 반환됩니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-10.png" alt="" width="85%"/>
<p>그림 8.10 Q&A 도우미 메인 페이지</p>
</div>

이 Q&A 지원 사례를 통해 RAGTool 및 MemoryTool을 사용하여 완전한 **웹 기반 지능형 문서 Q&A 시스템**을 구축하는 방법을 시연했습니다. 전체 코드는 `code/chapter8/11_Q&A_Assistant.py`에서 찾을 수 있습니다. 시작한 후 `http://localhost:7860`을 방문하여 지능형 학습 도우미를 사용해 보세요.

독자들은 이 사례를 개인적으로 실행하고, RAG 및 메모리의 기능을 경험하고, 이를 기반으로 확장 및 사용자 정의하여 자신의 요구 사항을 충족하는 지능형 응용 프로그램을 구축하는 것이 좋습니다!

## 8.5 장 요약 및 전망

이 장에서는 HelloAgents 프레임워크에 메모리 시스템과 RAG 시스템이라는 두 가지 핵심 기능을 성공적으로 추가했습니다.

이 장의 내용을 깊이 배우고 적용하기를 원하는 독자들을 위해 우리는 다음과 같은 제안을 제공합니다.

1. 0에서 1까지 기본 메모리 모듈을 직접 설계하고 점진적으로 반복하여 더 복잡한 기능을 추가합니다.

2. 특정 작업에 대한 최적의 솔루션을 찾기 위해 프로젝트의 다양한 임베딩 모델과 검색 전략을 시도하고 평가합니다.

3. 학습된 메모리와 RAG 시스템을 실제 개인 프로젝트에 적용하여 실제로 능력을 테스트하고 개선합니다.

고급 탐사

1. 최첨단 메모리 및 RAG 리포지토리를 추적하고 연구하여 뛰어난 구현을 학습합니다.
2. RAG 아키텍처를 다중 모드(텍스트 + 이미지) 또는 교차 모드 시나리오에 적용할 수 있는 가능성을 탐색합니다.
3. HelloAgents 오픈소스 프로젝트에 참여하여 아이디어와 코드를 기여해 보세요.

이 장의 연구를 통해 귀하는 메모리 및 RAG 시스템의 구현 기술을 숙달했을 뿐만 아니라, 더 중요한 것은 인지 과학 이론을 실용적인 엔지니어링 솔루션으로 변환하는 방법을 이해하게 되었습니다. 이러한 학제간 사고 방식은 AI 분야에서 귀하의 추가 발전을 위한 견고한 기반을 마련할 것입니다.

마지막으로 그림 8.11과 같이 마인드맵을 통해 이 장의 전체 지식 시스템을 요약해 보겠습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-11.png" alt="" width="85%"/>
<p>그림 8.11 Hello-agent 8장 지식 요약</p>
</div>

이 장에서는 HelloAgents 프레임워크의 메모리 시스템과 RAG 기술의 기능을 보여주었습니다. 우리는 진정한 "지능형" 학습 도우미를 성공적으로 구축했습니다. 이 아키텍처는 고객 서비스, 기술 지원, 개인 비서 및 기타 분야와 같은 다른 애플리케이션 시나리오로 쉽게 확장될 수 있습니다.

다음 장에서는 컨텍스트 엔지니어링을 통해 에이전트의 대화 품질과 사용자 경험을 더욱 향상시키는 방법을 계속해서 살펴보겠습니다. 계속 지켜봐 주시기 바랍니다!

## 연습

> **참고**: 일부 연습에는 표준 답변이 없습니다. 메모리 시스템과 RAG 기술에 대한 학습자의 포괄적인 이해와 실무 능력을 배양하는 데 중점을 두고 있습니다.

1. 이 장에서는 작업 기억, 일화 기억, 의미 기억, 지각 기억이라는 네 가지 기억 유형을 소개했습니다. 분석해 주십시오:

- 섹션 8.2.5에서는 각 메모리 유형마다 고유한 점수 공식이 있습니다. 일화기억과 의미기억의 채점 메커니즘을 비교하고 왜 일화기억은 "시간적 최근성"을 더 강조하는 반면(P0⟫), 의미기억은 "그래프 검색"을 더 강조하는 이유를 설명해주세요((weight 0.3))?
   - "개인 건강 관리 도우미"(사용자의 식단, 운동, 수면 데이터를 기록하고 건강 조언을 제공해야 함)를 디자인한다면 이 네 가지 메모리 유형을 어떻게 결합하시겠습니까? 각 메모리 유형에 대한 특정 애플리케이션 시나리오를 설계하십시오.
   - 작업 메모리는 TTL(Time To Live) 메커니즘을 사용하여 만료된 데이터를 자동으로 정리합니다. 생각해 보십시오: 어떤 상황에서 중요한 작업 기억이 장기 기억으로 "통합"되어야 합니까? 자동 통합 트리거 조건을 설계하는 방법은 무엇입니까?

2. 섹션 8.3의 RAG 시스템에서는 MarkItDown을 사용하여 다양한 형식의 문서를 Markdown으로 균일하게 변환합니다. 깊이 생각해주세요:

> **참고**: 이것은 실습 연습 문제이므로 실제 작동을 권장합니다.

- 현재 지능형 청킹 전략은 분할을 위한 Markdown 제목 계층 구조 (#, ##, ###)를 기반으로 합니다. 명확한 제목 구조가 없는 문서(예: 소설, 법률 조항)를 처리하는 경우 청킹 전략을 어떻게 최적화해야 합니까? "의미론적 경계"를 기반으로 청킹 알고리즘을 구현해 보세요.
   - 섹션 8.3.5에서는 MQE(Multi-Query Expansion) 및 HyDE(Hypothetical Document Embeddings)라는 두 가지 고급 검색 전략을 도입했습니다. 실제 시나리오 (such as technical document Q&A, medical knowledge retrieval)를 선택하고 기본 검색, MQE, HyDE의 효과 차이를 비교하고 각각의 적용 가능한 시나리오를 분석하십시오.
   - RAG 시스템의 검색 품질은 임베딩 모델 선택에 따라 크게 달라집니다. 이 장에서 언급된 세 가지 임베딩 솔루션(Bailian API, 로컬 Transformer, TF-IDF)을 정확성, 속도, 비용, 오프라인 배포 등의 측면에서 비교하고 선택 권장 사항을 제공하십시오.

3. 기억 시스템의 "망각" 메커니즘은 인간의 인지를 시뮬레이션하는 중요한 설계입니다. 섹션 8.2.3의 MemoryTool을 기반으로 다음 확장 연습을 완료하세요.

> **참고**: 이것은 실습 연습 문제이므로 실제 작동을 권장합니다.

- 현재 중요도 기반, 시간 기반, 용량 기반의 세 가지 망각 전략이 제공됩니다. 중요성, 접근 빈도, 시간 쇠퇴 및 기타 요소를 종합적으로 고려하는 "지능형 망각" 전략을 설계하고 구현하십시오. 가중치 점수를 사용하여 어떤 기억을 잊어야 하는지 결정하십시오.
   - 장기 실행 에이전트 시스템에서는 메모리 데이터베이스에 많은 양의 데이터가 축적될 수 있습니다. "메모리 보관" 메커니즘을 설계하십시오. 오랫동안 사용하지 않았지만 잠재적으로 귀중한 메모리를 콜드 스토리지로 옮기고 필요할 때 복원하십시오. 이 메커니즘을 기존의 네 가지 메모리 유형과 어떻게 통합해야 합니까?
   - 생각해보세요: 에이전트가 특정 민감한 정보(예: 사용자 개인 정보 데이터)를 "잊어야" 하는 경우 데이터베이스에서 해당 정보를 삭제하는 것으로 충분합니까? 벡터 데이터베이스와 그래프 데이터베이스를 사용하는 경우 데이터가 완전히 지워졌는지 확인하는 방법은 무엇입니까?

4. 섹션 8.4의 "지능형 학습 도우미" 사례에서는 MemoryTool과 RAGTool을 결합했습니다. 심층적으로 분석해 보세요.

- 이 경우의 `ask_question()` 방법은 RAG 검색과 메모리 검색을 모두 사용합니다. 분석해 보십시오. 어떤 상황에서 RAG가 우선시되어야 합니까? 어떤 상황에서 메모리를 우선시해야 합니까? 가장 적절한 검색 방법을 자동으로 선택하는 "지능형 라우팅" 메커니즘을 설계하는 방법은 무엇입니까?
   - 현재 학습 보고서(`generate_report()`)에는 통계 정보만 포함되어 있습니다. 이 기능을 확장하여 사용자의 학습 궤적을 분석하고, 지식 사각지대를 식별하고, 다음 학습 콘텐츠를 추천할 수 있는 보다 지능적인 학습 보고서 생성기를 설계하십시오. 이를 위해 어떤 메모리 유형과 검색 전략이 필요합니까?
   - 각 사용자가 독립적인 메모리와 지식 기반을 갖는 다중 사용자 웹 서비스로 이 학습 도우미를 배포한다고 가정합니다. 데이터 격리 솔루션을 설계해 주세요. Qdrant 및 Neo4j에서 사용자 수준 데이터 격리를 구현하는 방법은 무엇입니까? 다중 사용자 시나리오에서 검색 성능을 최적화하는 방법은 무엇입니까?

5. 시맨틱 메모리는 Neo4j 그래프 데이터베이스를 사용하여 지식 그래프를 저장합니다. 생각해 보십시오:

- 8.2.5절의 의미기억 구현에서는 시스템이 자동으로 엔터티와 관계를 추출하여 지식 그래프를 구축합니다. 분석해 보세요. 이 자동 추출은 얼마나 정확합니까? 어떤 상황에서 잘못된 엔터티나 관계가 추출될 수 있나요? "지식 그래프 품질 평가" 메커니즘을 어떻게 설계하나요?
   - 지식 그래프의 중요한 장점은 복잡한 관계 추론을 지원한다는 것입니다. 순수 벡터 검색으로는 완료할 수 없는 작업을 수행하기 위해 Neo4j의 그래프 쿼리 기능(예: 멀티 홉 관계, 경로 찾기)을 완전히 활용하는 쿼리 시나리오를 설계하십시오.
   - 의미기억의 "벡터 검색 + 그래프 검색" 하이브리드 전략을 순수 벡터 검색과 비교합니다. 어떤 유형의 쿼리에서 그래프 검색이 상당한 성능 향상을 가져올 수 있습니까? 구체적인 예를 들어 설명해주세요.

## 참고자료

[1] Atkinson, R.C., & Shiffrin, R.M. (1968). 인간 기억: 제안된 시스템과 그 제어 프로세스. *학습 및 동기 부여 심리학* (Vol. 2, pp. 89-195). 학술 언론.

