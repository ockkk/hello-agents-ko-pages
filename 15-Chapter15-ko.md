# 15장: 사이버 타운 구축

이번 장에서는 **에이전트 기술과 게임 엔진을 결합하여 활력이 넘치는 AI 타운을 구축**하는 새로운 방향을 모색하겠습니다.

"심즈"나 "동물의 숲"에 나오는 실물 같은 NPC를 기억하시나요? 그들은 각자의 성격, 기억, 사회적 관계를 가지고 있습니다. 이 장의 사이버 타운은 유사한 프로젝트이지만 기존 게임과 달리 NPC는 진정한 "지능"을 가지고 있습니다. NPC는 플레이어 대화를 이해하고 과거 상호 작용을 기억하며 애정 수준에 따라 다르게 반응할 수 있습니다. 본 장의 사이버 타운에는 다음과 같은 핵심 기능이 포함되어 있습니다.

**(1) 지능형 NPC 대화 시스템**: 플레이어는 NPC와 자연어 대화를 나눌 수 있으며 NPC는 역할 설정 및 기억에 따라 응답합니다.

**(2) 기억 시스템**: NPC는 단기 및 장기 기억을 가지고 있어 플레이어와의 상호 작용 기록을 기억할 수 있습니다.

**(3) 애정 시스템**: 플레이어에 대한 NPC 태도는 상호 작용에 따라 낯선 사람에서 친숙한 사람으로, 친근한 사람에서 친밀한 사람으로 변합니다.

**(4) 게임화된 상호 작용**: 플레이어는 2D 픽셀 스타일의 사무실 장면에서 자유롭게 이동하고 다양한 NPC와 상호 작용할 수 있습니다.

**(5) 실시간 로깅 시스템**: 쉬운 디버깅 및 분석을 위해 모든 대화와 상호 작용이 기록됩니다.

## 15.1 프로젝트 개요 및 아키텍처 설계

### 15.1.1 AI 타운을 건설해야 하는 이유

기존 게임의 NPC는 일반적으로 고정된 대사만 말하거나 미리 설정된 대화 트리를 통해 제한된 상호 작용을 할 수 있습니다. 가장 복잡한 RPG 게임에서도 NPC 대화는 각본가가 미리 작성합니다. 이 접근 방식은 제어 가능하지만 실제 "지능"과 "활력"이 부족합니다.

게임 속 NPC가 더 이상 미리 설정된 옵션에 국한되지 않고 사용자가 말하는 모든 내용을 이해할 수 있다고 상상해 보세요. NPC와 자연어로 대화할 수 있습니다. NPC는 지난번에 말한 내용, 관계, 심지어 선호도까지 기억합니다. NPC마다 고유한 직업, 성격, 말하는 스타일이 있습니다. 당신에 대한 NPC의 태도는 낯선 사람부터 친구, 심지어 가까운 친구까지 상호 작용에 따라 달라집니다.

이것이 AI 기술이 게임에 가져다주는 새로운 가능성이다. 대규모 언어 모델을 게임 엔진과 결합함으로써 우리는 진정으로 "살아있는" NPC를 만들 수 있습니다. 이는 단순한 기술 시연이 아니라 미래의 게임 형태에 대한 탐구입니다. 교육용 게임에서 NPC는 역사적 인물과 과학자 역할을 맡아 학생들과 대화형 교육을 수행할 수 있습니다. 가상 사무실에서 NPC는 동료와 멘토 역할을 하여 도움과 조언을 제공할 수 있습니다. NPC는 또한 정신 건강 분야에 적용되는 사용자와 정서적 의사소통을 수행하는 동반자 역할을 할 수도 있습니다. 물론 가장 직접적인 적용은 기존 게임에 AI NPC를 추가하여 플레이어 경험을 향상시키는 것입니다.

### 15.1.2 기술 아키텍처 개요

Cyber ​​Town은 그림 15.1과 같이 4개 계층으로 구분된 **게임 엔진 + 백엔드 서비스** 분리 아키텍처를 채택합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-1.png" alt="" width="85%"/>
<p>그림 15.1 사이버 타운 기술 아키텍처</p>
</div>

프런트엔드 레이어는 게임 렌더링, 플레이어 제어, NPC 표시 및 대화 UI를 담당하는 Godot 4.5 게임 엔진을 사용합니다. Godot는 오픈 소스 2D/3D 게임 엔진으로, 픽셀 스타일 게임을 빠르게 개발하는 데 매우 적합합니다. 백엔드 계층은 API 라우팅, NPC 상태 관리, 대화 처리 및 로깅을 담당하는 FastAPI 프레임워크를 사용합니다. FastAPI는 뛰어난 성능과 쉬운 개발 기능을 갖춘 최신 Python 웹 프레임워크입니다. 에이전트 계층은 NPC 지능, 메모리 관리 및 애정 계산을 담당하는 자체 HelloAgents 프레임워크를 사용합니다. 각 NPC는 독립적인 메모리와 상태를 가진 SimpleAgent 인스턴스입니다. 외부 서비스 계층은 LLM API, Qdrant 벡터 데이터베이스 및 SQLite 관계형 데이터베이스를 포함하여 LLM 기능, 벡터 저장소 및 데이터 지속성을 제공합니다.

데이터 흐름 프로세스는 그림 15.2에 나와 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-2.png" alt="" width="85%"/>
<p>그림 15.2 데이터 흐름 프로세스</p>
</div>

플레이어는 NPC와 상호작용하기 위해 Godot에서 E 키를 누르고, Godot는 HTTP API를 통해 FastAPI 백엔드에 대화 요청을 보냅니다. 백엔드는 HelloAgents의 SimpleAgent를 호출하여 대화를 처리하고, Agent는 메모리 시스템에서 관련 내역을 검색한 후 LLM을 호출하여 응답을 생성합니다. 백엔드는 NPC 상태와 애정을 업데이트하고, 콘솔과 파일에 로그를 기록하고, 마지막으로 Godot 프런트엔드에 응답을 반환합니다. Godot는 NPC 응답을 표시하고 UI를 업데이트하여 완전한 상호 작용 루프를 완료합니다.

프로젝트 구조는 다음과 같으므로 소스 코드를 쉽게 찾을 수 있습니다.

```
Helloagents-AI-Town/
├── helloagents-ai-town/           # Godot game project
│   ├── project.godot              # Godot project configuration
│   ├── scenes/                    # Game scenes
│   │   ├── main.tscn              # Main scene (office)
│   │   ├── player.tscn            # Player character
│   │   ├── npc.tscn               # NPC character
│   │   └── dialogue_ui.tscn       # Dialogue UI
│   ├── scripts/                   # GDScript scripts
│   │   ├── main.gd                # Main scene logic
│   │   ├── player.gd              # Player control
│   │   ├── npc.gd                 # NPC behavior
│   │   ├── dialogue_ui.gd         # Dialogue UI logic
│   │   ├── api_client.gd          # API client
│   │   └── config.gd              # Configuration management
│   └── assets/                    # Game assets
│       ├── characters/            # Character sprites
│       ├── interiors/             # Interior scenes
│       ├── ui/                    # UI materials
│       └── audio/                 # Sound effects and music
│
└── backend/                       # Python back-end
    ├── main.py                    # FastAPI main program
    ├── agents.py                  # NPC Agent system
    ├── relationship_manager.py    # Affection management
    ├── state_manager.py           # State management
    ├── logger.py                  # Logging system
    ├── config.py                  # Configuration management
    ├── models.py                  # Data models
    ├── requirements.txt           # Python dependencies
    └── .env.example               # Environment variable example
```

자세한 아키텍처 설계 및 데이터 흐름은 후속 섹션에서 소개됩니다.

### 15.1.3 빠른 경험: 5분 안에 프로젝트 실행

구현 세부 사항을 살펴보기 전에 먼저 프로젝트를 실행하여 최종 결과를 살펴보겠습니다. 이렇게 하면 전체 시스템을 직관적으로 이해할 수 있습니다.

**환경 요구 사항:**

- Godot 4.2 이상
- 파이썬 3.10 이상
- LLM API 키 (OpenAI, DeepSeek, Zhipu, etc.)

**프로젝트 받기:**

`code/chapter15/Helloagents-AI-Town`을 확인하거나 GitHub에서 전체 hello-agents 저장소를 복제할 수 있습니다.

**백엔드 시작:**

```bash
# 1. Enter backend directory
cd Helloagents-AI-Town/backend

# 2. Install dependencies
pip install -r requirements.txt

# 3. Configure environment variables
cp .env.example .env
# Edit .env file, fill in your API key

# 4. Start back-end service
python main.py
```

성공적으로 시작되면 다음 출력이 표시됩니다.

```
============================================================
🎮 Cyber Town back-end service starting...
============================================================
✅ All services started!
📡 API address: http://0.0.0.0:8000
📚 API documentation: http://0.0.0.0:8000/docs
============================================================
```

**고도를 시작하세요:**

Godot 설치는 매우 간단합니다. Windows에서는 직접 `.exe` 파일을 제공하고 Mac에서는 `.dmg` 파일도 제공합니다. 공식 홈페이지([Windows](https://godotengine.org/download/windows/) / [Mac](https://godotengine.org/download/macos/)))에서 직접 다운로드할 수 있습니다.

Godot 엔진을 열고 "가져오기" 버튼을 클릭한 후 `Helloagents-AI-Town/helloagents-ai-town/scenes/main.tscn`를 찾아 "가져오기 및 편집"을 클릭하세요. Godot가 리소스를 가져온 후 `F5`을 누르거나 "실행" 버튼을 클릭하여 게임을 시작하세요.

**핵심 기능을 경험해 보세요:**

게임이 시작되면 그림 15.3과 같이 픽셀 스타일의 Datawhale 사무실 장면이 표시됩니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-3.png" alt="" width="85%"/>
<p>그림 15.3 사이버 타운 게임 장면</p>
</div>

WASD 키를 사용하여 플레이어 캐릭터를 이동하세요. NPC 근처로 걸어가면 화면에 "상호작용하려면 E를 누르세요"라는 메시지가 표시됩니다. E 키를 누르면 대화 상자가 나타나며, 그림 15.4와 같이 말하고 싶은 내용을 입력할 수 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-4.png" alt="" width="85%"/>
<p>그림 15.4 NPC</p>과의 대화 인터페이스
</div>

NPC는 역할 설정(Python 엔지니어, 제품 관리자, UI 디자이너) 및 상호 작용 기록을 기반으로 응답합니다. 대화가 진행됨에 따라 NPC의 호감도는 '낯선 사람'에서 '친숙한 사람'으로, 다시 '친근한 사람', '친밀한 사람', 심지어 '친한 친구'로 점차 높아집니다.

**애정 시스템은 백엔드에서 구현됩니다**. 각 대화는 플레이어의 메시지 내용과 감정 분석을 기반으로 애정 값을 조정합니다. 애정 값은 프런트엔드 게임 인터페이스에 직접 표시되지 않지만, 모든 애정 변화는 백엔드 로그에 자세히 기록됩니다. `backend/logs/dialogue_YYYY-MM-DD.log` 파일에서 각 대화의 호감도 변화를 확인할 수 있습니다. 로그 파일에는 현재 애정 수치, 검색된 관련 기억, NPC의 답변, 애정 변화량 (+2.0, +3.0, etc.), 변경 이유 (friendly greeting, normal communication, etc.), 감성 분석 결과 (positive, neutral, etc.) 등 각 대화에 대한 자세한 정보가 기록됩니다. 이 디자인을 통해 개발자는 NPC와 플레이어 간의 관계 발전을 명확하게 추적할 수 있으며 나중에 프런트엔드에 애정 UI를 추가하기 위한 데이터 기반을 제공합니다.

모든 대화는 백엔드 로그 파일에 기록됩니다. 다음 명령을 사용하면 실시간으로 볼 수 있습니다.

```bash
# In the backend directory
python view_logs.py
```

이 간단한 경험은 AI Town의 핵심 기능을 보여줍니다. 다음으로 이러한 기능을 구현하는 방법을 살펴보겠습니다.

## 15.2 NPC 에이전트 시스템

### 15.2.1 HelloAgent 기반의 SimpleAgent

사이버 타운에서는 각 NPC가 독립된 에이전트입니다. 우리는 NPC 지능을 구현하기 위해 HelloAgents 프레임워크의 SimpleAgent를 사용합니다. SimpleAgent는 LLM 호출, 메시지 관리, 도구 호출과 같은 핵심 기능을 캡슐화하는 경량 에이전트 구현입니다.

7장에서 배운 SimpleAgent를 떠올려 보세요. 그 핵심은 간단한 대화 루프입니다. 즉, 사용자 메시지를 받고, LLM을 호출하여 응답을 생성하고, 결과를 반환합니다. Cyber ​​Town에서는 각 NPC에 대해 SimpleAgent 인스턴스를 생성하고 고유한 시스템 프롬프트를 구성하여 각 NPC에게 서로 다른 성격과 역할 설정을 제공해야 합니다.

NPC 에이전트를 생성하는 방법을 살펴보겠습니다. 먼저, ID, 이름, 직업, 성격 등 NPC의 기본 정보를 정의해야 합니다. 그런 다음 이 정보를 기반으로 시스템 프롬프트를 구축하여 LLM이 이 NPC의 역할을 수행하도록 합니다. 마지막으로 SimpleAgent 인스턴스를 생성하고 메모리 시스템을 구성합니다.

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.memory import MemoryManager, WorkingMemory, EpisodicMemory

def create_npc_agent(npc_id: str, name: str, role: str, personality: str):
    """Create NPC Agent"""
    # Build system prompt
    system_prompt = f"""You are {name}, a {role}.
Your personality traits: {personality}

You work in the Datawhale office, working with colleagues to promote the development of the open source community.
Please have natural conversations with players based on your role and personality.
Remember your previous conversations to maintain dialogue coherence.
"""

    # Create LLM instance
    llm = HelloAgentsLLM()

    # Create memory manager
    memory_manager = MemoryManager(
        working_memory=WorkingMemory(capacity=10, ttl_minutes=120),
        episodic_memory=EpisodicMemory(
            db_path=f"memory_data/{npc_id}_episodic.db",
            collection_name=f"{npc_id}_memories"
        )
    )

    # Create Agent
    agent = SimpleAgent(
        name=name,
        llm=llm,
        system_prompt=system_prompt,
        memory_manager=memory_manager
    )

    return agent
```

이 코드는 NPC 에이전트를 생성하는 방법을 보여줍니다. 시스템 프롬프트는 NPC의 신원과 성격을 정의하며, 메모리 관리자를 통해 NPC가 플레이어와의 대화 내역을 기억할 수 있습니다. WorkingMemory는 10개의 메시지를 저장할 수 있고 보관 시간은 120분인 단기 기억 장치입니다. EpisodicMemory는 SQLite 데이터베이스와 Qdrant 벡터 데이터베이스를 저장용으로 사용하는 장기 메모리이며, 관련 과거 대화를 검색할 수 있습니다.

NPC 에이전트의 작업 흐름은 그림 15.5에 나와 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-5.png" alt="" width="85%"/>
<p>그림 15.5 NPC 에이전트 작업 흐름</p>
</div>

### 15.2.2 NPC 역할 설정 및 프롬프트 디자인

좋은 NPC는 뚜렷한 성격과 역할 설정이 필요합니다. 사이버 타운에서는 서로 다른 직업과 성격을 대표하는 세 명의 NPC를 디자인했습니다.

**장산(Zhang San) - Python 엔지니어**

Zhang San은 HelloAgents 프레임워크의 핵심 개발을 담당하는 수석 Python 엔지니어입니다. 엄격한 성격을 갖고 있고, 직설적으로 말하고, 전문적인 용어를 사용하는 것을 좋아한다. 그는 코드 품질에 대한 높은 요구 사항을 갖고 있으며 프로그래밍 팁과 모범 사례를 자주 공유합니다.

```python
npc_zhang = {
    "npc_id": "zhang_san",
    "name": "Zhang San",
    "role": "Python Engineer",
    "personality": "Rigorous, professional, likes to share technical knowledge. Speaks directly, focuses on code quality."
}
```

**Li Si - 제품 관리자**

Li Si는 HelloAgents 프레임워크의 제품 기획 및 사용자 경험 디자인을 담당하는 숙련된 제품 관리자입니다. 활달한 성격에 커뮤니케이션에 능숙하며, 항상 유저의 입장에서 생각하는 능력을 갖고 있습니다. 그는 제품 디자인과 사용자 요구 사항에 대해 논의하는 것을 좋아하며 종종 "왜"라고 묻습니다.

```python
npc_li = {
    "npc_id": "li_si",
    "name": "Li Si",
    "role": "Product Manager",
    "personality": "Outgoing, good at communication, focuses on user experience. Likes to think from the user's perspective."
}
```

**왕 우 - UI 디자이너**

Wang Wu는 HelloAgents 프레임워크의 인터페이스 디자인과 시각적 표현을 담당하는 창의적인 UI 디자이너입니다. 그는 온화한 성격, 독특한 미학, 색상과 레이아웃에 대한 예리한 인식을 가지고 있습니다. 그는 디자인 컨셉과 미학에 대해 토론하는 것을 좋아하며 종종 디자인 영감을 공유합니다.

```python
npc_wang = {
    "npc_id": "wang_wu",
    "name": "Wang Wu",
    "role": "UI Designer",
    "personality": "Gentle, creative, unique aesthetics. Focuses on visual presentation and user experience."
}
```

이 세 NPC는 서로 다른 특성을 가지고 있습니다. 플레이어는 자신의 관심사에 따라 다양한 NPC와 상호 작용하도록 선택할 수 있습니다. Zhang San은 프로그래밍 기술을 가르쳐주고, Li Si는 제품 디자인에 대해 토론할 수 있으며, Wang Wu는 디자인 영감을 공유할 수 있습니다.

### 15.2.3 메모리 시스템 통합

기억 시스템은 NPC 지능의 핵심입니다. 과거 대화를 기억할 수 있는 NPC는 플레이어에게 더욱 현실감 있고 흥미로운 느낌을 선사할 것입니다. HelloAgents의 `WorkingMemory` 및 `EpisodicMemory`을 사용하여 단기 및 장기 기억을 구성합니다.

단기 기억은 제한된 용량으로 최근 대화 내용을 저장하고 시간이 지남에 따라 자동으로 정리됩니다. 그 역할은 NPC가 맥락을 이해할 수 있도록 대화 일관성을 유지하는 것입니다. 예를 들어 플레이어가 "이게 무슨 색이에요?"라고 말하면 NPC는 "그것"이 무엇을 가리키는지 단기 기억에서 찾아야 합니다.

장기 기억은 의미 검색을 위한 벡터 데이터베이스를 사용하여 모든 대화 기록을 저장합니다. 플레이어가 주제를 언급하면 ​​NPC는 이전에 논의된 내용을 회상하면서 장기 기억에서 관련 역사적 대화를 검색할 수 있습니다. 예를 들어, 플레이어가 "저번에 논의한 프로젝트를 기억하시나요?"라고 말하면 NPC는 장기 기억에서 관련 대화 기록을 찾을 수 있습니다.

메모리 시스템의 아키텍처는 그림 15.6에 나와 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-6.png" alt="" width="85%"/>
<p>그림 15.6 메모리 시스템 아키텍처</p>
</div>

실제 사용 시 에이전트는 먼저 단기 기억에서 최근 대화를 얻은 다음 장기 기억에서 관련 과거 대화를 검색하고 이 정보를 함께 LLM에 전송하여 보다 정확하고 개인화된 답변을 생성합니다.

```python
# Agent's dialogue processing flow
def process_dialogue(agent, player_message):
    # 1. Get recent conversations from short-term memory
    recent_messages = agent.memory_manager.working_memory.get_recent_messages(5)

    # 2. Retrieve relevant history from long-term memory
    relevant_memories = agent.memory_manager.episodic_memory.search(
        query=player_message,
        top_k=3
    )

    # 3. Build context
    context = {
        "recent": recent_messages,
        "relevant": relevant_memories
    }

    # 4. Call Agent to generate reply
    reply = agent.run(player_message, context=context)

    # 5. Save to memory system
    agent.memory_manager.add_interaction(player_message, reply)

    return reply
```

이 프로세스를 통해 NPC는 플레이어와의 상호 작용 기록을 기억하고 이를 대화에 반영할 수 있습니다.

### 15.2.4 일괄 대화 생성: 경부하 모드

실제 작업에서는 문제가 빠르게 발견되었습니다. 여러 플레이어가 서로 다른 NPC와 동시에 대화할 때 백엔드는 여러 LLM 요청을 동시에 처리해야 합니다. 각 요청은 API를 호출해야 하며, 이로 인해 비용이 증가할 뿐만 아니라 동시성 제한으로 인해 요청 실패 또는 지연이 발생할 수도 있습니다.

이 문제를 해결하기 위해 **일괄 대화 생성 시스템**을 설계했습니다. 핵심 아이디어는 여러 NPC 대화 요청을 하나의 LLM 호출로 병합하여 LLM이 모든 NPC 응답을 한 번에 생성하도록 하는 것입니다. 이는 레스토랑의 "미리 만들어진 요리"와 같습니다. 미리 일괄 준비하고 필요할 때 직접 사용하여 비용과 대기 시간을 크게 줄입니다.

배치 생성의 작업 흐름은 그림 15.7에 나와 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-7.png" alt="" width="85%"/>
<p>그림 15.7 배치 생성과 기존 모드</p>
</div>

배치 생성기의 구현은 매우 영리합니다. 우리는 LLM이 모든 NPC 대화를 한 번에 생성하고 이를 JSON 형식으로 반환하도록 요구하는 특수 프롬프트를 구축합니다. 이렇게 하면 한 번의 API 호출로 모든 NPC 응답을 얻을 수 있어 비용이 원래 비용의 1/3로 줄어들고 대기 시간이 크게 줄어듭니다.

```python
class NPCBatchGenerator:
    """Generator for batch generating NPC dialogues"""

    def __init__(self):
        self.llm = HelloAgentsLLM()
        self.npc_configs = NPC_ROLES  # All NPC configurations

    def generate_batch_dialogues(self, context: Optional[str] = None) -> Dict[str, str]:
        """Batch generate dialogues for all NPCs

        Args:
            context: Scene context (such as "morning work time", "lunch time", etc.)

        Returns:
            Dict[str, str]: Mapping from NPC names to dialogue content
        """
        # Build batch generation prompt
        prompt = self._build_batch_prompt(context)

        # One LLM call generates all dialogues
        response = self.llm.invoke([
            {"role": "system", "content": "You are a game NPC dialogue generator, skilled at creating natural and realistic office dialogues."},
            {"role": "user", "content": prompt}
        ])

        # Parse JSON response
        dialogues = json.loads(response)
        # Return format: {"Zhang San": "...", "Li Si": "...", "Wang Wu": "..."}

        return dialogues

    def _build_batch_prompt(self, context: Optional[str] = None) -> str:
        """Build batch generation prompt"""
        # Automatically infer scene based on time
        if context is None:
            context = self._get_current_context()

        # Build NPC descriptions
        npc_descriptions = []
        for name, cfg in self.npc_configs.items():
            desc = f"- {name}({cfg['title']}): {cfg['activity']} at {cfg['location']}, personality {cfg['personality']}"
            npc_descriptions.append(desc)

        npc_desc_text = "\n".join(npc_descriptions)

        prompt = f"""Please generate current dialogues or behavior descriptions for 3 NPCs in the Datawhale office.

【Scene】{context}

【NPC Information】
{npc_desc_text}

【Generation Requirements】
1. Generate 1 sentence for each NPC (20-40 characters)
2. Content should match role settings, current activities, and scene atmosphere
3. Can be self-talk, work status description, or simple thoughts
4. Should be natural and realistic, like real office colleagues
5. **Must strictly return in JSON format**

【Output Format】(strictly follow)
{{"Zhang San": "...", "Li Si": "...", "Wang Wu": "..."}}

【Example Output】
{{"Zhang San": "This bug is really annoying, been debugging for two hours...", "Li Si": "Hmm, the priority of this feature needs to be re-evaluated.", "Wang Wu": "The latte art on this coffee is really nice, inspiration is coming!"}}

Please generate (only return JSON, no other content):
"""
        return prompt
```

이 디자인의 핵심은 프롬프트 구성입니다. 우리는 LLM이 JSON 형식을 반환하고 예제 출력을 제공하도록 명시적으로 요구합니다. LLM은 이 형식에 따라 엄격하게 응답을 생성하므로 모든 NPC 대화를 얻으려면 JSON만 구문 분석하면 됩니다.

일괄 생성에는 추가적인 이점이 있습니다. 모든 NPC 대화는 동일한 컨텍스트에서 생성되므로 어느 정도의 상관 관계가 있습니다. 예를 들어, Zhang San이 버그를 디버깅하는 경우 Li Si는 살펴보는 데 도움이 된다고 언급할 수 있습니다. Wang Wu가 인터페이스를 디자인하고 있다면 Zhang San은 나중에 디자인 초안을 확인하겠다고 말할 수도 있습니다. 이는 전체 사무실의 분위기를 더욱 현실적이고 일관되게 만듭니다.

물론 일괄 생성에는 몇 가지 제한 사항이 있습니다. 플레이어와의 직접적인 상호작용보다는 NPC '배경 대화'나 '혼자 대화'를 생성하는 데 더 적합합니다. 플레이어가 시작한 대화의 경우 개인화되고 정확한 답변을 보장하기 위해 여전히 개별 에이전트를 사용하여 처리합니다. 일괄 생성은 주로 다음 시나리오에서 사용됩니다.

1. **NPC 배경 대화**: 플레이어가 장면에 들어올 때 NPC가 무엇을 하고 말하고 있는지
2. **시간 제한 업데이트**: NPC 상태 및 대화를 정기적으로 업데이트합니다.
3. **장면 분위기**: 시간(아침, 점심, 저녁)에 따라 다양한 대화 생성
4. **비용 절감**: 동시성 시나리오에서 일괄 생성을 사용하여 API 호출 빈도를 줄입니다.

**하이브리드 모드: 일괄 생성 + 즉각적인 응답**

실제 구현에서는 배치 생성과 즉각적인 응답을 결합한 하이브리드 모드를 채택했습니다. 이 디자인은 매우 영리하여 효율성과 상호 작용 품질을 모두 보장합니다.

특히 시스템은 백그라운드에서 주기적으로 일괄 생성을 실행하여 현재 장면의 모든 NPC에 대한 "백그라운드 대화"를 생성합니다. 이러한 대화는 캐시되며 플레이어가 NPC에 접근했지만 아직 상호 작용을 시작하지 않은 경우 NPC는 "코드 디버깅 중...", "제품 문서를 읽는 중..." 등과 같은 배경 대화 상자를 표시합니다. 이로 인해 NPC는 정적 모델이 아닌 "살아 있는" 것처럼 보입니다.

그러나 플레이어가 상호 작용을 시작하기 위해 E 키를 누르면 시스템은 즉시 즉시 응답 모드로 전환됩니다. 이 시점에서 백엔드는 NPC의 전담 에이전트를 호출하여 플레이어의 특정 메시지, 역사적 기억 및 애정 수준을 기반으로 개인화된 응답을 생성합니다. 이 프로세스는 실시간으로 진행되므로 NPC 응답이 플레이어 입력과 관련성이 높습니다.

```python
# Hybrid mode implementation in main.py
@app.post("/dialogue")
async def dialogue(request: DialogueRequest):
    """Handle player-NPC dialogue (instant response mode)"""
    npc_id = request.npc_id
    player_message = request.player_message
    player_name = request.player_name

    # Get NPC Agent (each NPC has an independent Agent)
    agent = npc_agents.get(npc_id)
    if not agent:
        raise HTTPException(status_code=404, detail="NPC not found")

    # Instantly generate personalized reply
    # Here we don't use batch generation, but call Agent's run method
    reply = agent.run(player_message)

    # Update affection
    affinity_change = relationship_manager.update_affinity(
        npc_id, player_name, player_message, reply
    )

    return {
        "npc_reply": reply,
        "affinity_score": affinity_change["score"],
        "affinity_level": affinity_change["level"]
    }

# Background task: periodically batch generate background dialogues
async def background_dialogue_update():
    """Background task: update NPC background dialogues every 5 minutes"""
    while True:
        try:
            # Use batch generator to generate background dialogues for all NPCs
            batch_generator = get_batch_generator()
            dialogues = batch_generator.generate_batch_dialogues()

            # Update to state manager
            for npc_name, dialogue in dialogues.items():
                state_manager.update_npc_background_dialogue(npc_name, dialogue)

            print(f"✅ Background dialogue update complete: {len(dialogues)} NPCs")
        except Exception as e:
            print(f"❌ Background dialogue update failed: {e}")

        # Wait 5 minutes
        await asyncio.sleep(300)
```

이 하이브리드 모드의 장점은 매우 분명합니다.

1. **비용 절감**: 백그라운드 대화는 일괄 생성을 사용하고, 한 번의 호출로 모든 NPC 대화가 생성되며, 비용이 저렴합니다.
2. **품질 보증**: 플레이어 상호 작용은 즉각적인 응답을 사용하며 각 응답은 개인화되고 고품질입니다.
3. **향상된 경험**: NPC에는 항상 "배경 대화"가 있어 매우 생생하게 나타납니다. 플레이어 상호 작용은 정확한 답변과 좋은 경험을 제공합니다.
4. **유연한 조정**: 서버 로드에 따라 일괄 생성 빈도를 동적으로 조정할 수 있습니다.

일괄 생성과 즉각적인 응답의 결합을 통해 효율적이면서도 지능적인 NPC 시스템을 구현했습니다. 일반적인 상황에서는 플레이어가 아무런 차이를 느끼지 못하지만 백엔드 비용과 성능이 크게 최적화됩니다. 이 설계 접근 방식은 많은 수의 AI 호출이 필요한 다른 시나리오에도 적용될 수 있습니다.

## 15.3 애정 시스템 설계

### 15.3.1 애정도 분류

사이버타운에서는 플레이어를 대하는 NPC의 태도가 상호작용에 따라 변화합니다. 우리는 낯선 사람부터 가까운 친구까지 5단계 애정 시스템을 설계했습니다. 각 수준은 서로 다른 점수 범위와 그에 따른 행동 성과를 가집니다.

애정 시스템의 핵심 아이디어는 NPC와 플레이어 간의 관계를 정량화하여 NPC 응답을 보다 현실적이고 계층적으로 만드는 것입니다. 플레이어가 처음 게임에 입장하면 모든 NPC는 플레이어에 대해 낯선 태도를 취하며 대답은 정중하지만 냉담합니다. 대화가 진행될수록 플레이어들이 친근하게 행동하면 NPC의 호감도가 점차 높아지며, 답변도 더욱 친절하고 자세해집니다.

우리는 그림 15.8과 같이 애정을 5가지 수준으로 나누며, 각 수준은 점수 범위에 해당합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-8.png" alt="" width="85%"/>
<p>그림 15.8 애정도 분류</p>
</div>

- **낯선 사람(0~20점)**: NPC가 플레이어를 방금 만났습니다. 태도는 예의바르지만 거리를 유지합니다. 답변은 간단하며 개인 정보를 적극적으로 공유하지 않습니다.

- **익숙함(21-40점)**: NPC가 플레이어를 기억하기 시작하고 간단한 교환을 하려고 합니다. 답변이 더욱 자연스러워지고 때로는 업무 관련 정보를 공유하기도 합니다.

- **우호적(41-60점)**: NPC는 플레이어를 친구로 대하고 더 많은 정보를 공유할 의향이 있습니다. 답변은 좀 더 자세하게, 플레이어의 상황에 대해 적극적으로 질문하겠습니다.

- **친밀함(61-80점)**: NPC는 플레이어를 매우 신뢰하며 사적인 주제를 기꺼이 공유합니다. 답변은 열정으로 가득 차 있으며 플레이어에게 도움과 조언을 제공할 것입니다.

- **친한 친구(81~100점)**: NPC는 플레이어를 가장 친한 친구로 대하고 모든 것에 대해 이야기합니다. 답변은 매우 친절하며 내면의 생각과 감정을 공유합니다.

이 디자인을 통해 플레이어는 NPC와의 관계 변화를 명확하게 느낄 수 있으며, 이후 게임 플레이의 기반도 제공합니다. 예를 들어, 특정 호감도에 도달한 후에만 NPC는 특정 정보를 공유하거나 특별한 임무를 제공합니다.

### 15.3.2 호감도 계산 로직

애정 계산은 여러 요소를 고려해야 합니다. 단순히 각 대화에 대해 고정된 점수를 추가할 수는 없으며, 이로 인해 시스템이 기계적이고 비현실적으로 보일 수 있습니다. 좋은 애정 시스템은 플레이어의 태도를 파악하고 대화 내용에 따라 점수를 동적으로 조정할 수 있어야 합니다.

Cyber ​​Town에서는 LLM을 사용하여 대화 내용을 분석하여 플레이어의 태도가 우호적인지, 중립적인지, 비우호적인지 판단합니다. 그런 다음 판단 결과에 따라 애정 점수를 조정합니다. 이 과정은 자동으로 진행되므로 플레이어가 의도적으로 옵션을 선택할 필요가 없으므로 상호 작용이 더욱 자연스러워집니다.

애정 계산 과정은 그림 15.9에 나와 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-9.png" alt="" width="85%"/>
<p>그림 15.9 호감도 계산 과정</p>
</div>

```python
class RelationshipManager:
    """Affection manager"""

    def __init__(self):
        self.affinity_data = {}  # Store affection data
        self.llm = HelloAgentsLLM()  # For analyzing conversations

    def analyze_sentiment(self, player_message: str, npc_reply: str) -> int:
        """Analyze conversation sentiment, return affection change value"""
        prompt = f"""Analyze the player's attitude in the following conversation:
Player: {player_message}
NPC: {npc_reply}

Please judge if the player's attitude is:
1. Friendly (+5 points): Polite, enthusiastic, expressing thanks or agreement
2. Neutral (+2 points): Normal inquiry or statement
3. Unfriendly (-3 points): Rude, indifferent, critical or negative

Only return the number, no other content."""

        response = self.llm.think([{"role": "user", "content": prompt}])
        try:
            score_change = int(response.strip())
            return max(-3, min(5, score_change))  # Limit between -3 and 5
        except:
            return 2  # Default neutral

    def update_affinity(self, npc_id: str, player_name: str,
                       player_message: str, npc_reply: str) -> dict:
        """Update affection"""
        key = f"{npc_id}_{player_name}"

        # Get current affection
        if key not in self.affinity_data:
            self.affinity_data[key] = {
                "score": 0,
                "level": "Stranger",
                "interaction_count": 0
            }

        # Analyze conversation sentiment
        score_change = self.analyze_sentiment(player_message, npc_reply)

        # Update score
        current_score = self.affinity_data[key]["score"]
        new_score = max(0, min(100, current_score + score_change))

        # Update level
        level = self.get_affinity_level(new_score)

        # Update data
        self.affinity_data[key].update({
            "score": new_score,
            "level": level,
            "interaction_count": self.affinity_data[key]["interaction_count"] + 1
        })

        return self.affinity_data[key]

    def get_affinity_level(self, score: int) -> str:
        """Get affection level based on score"""
        if score <= 20:
            return "Stranger"
        elif score <= 40:
            return "Familiar"
        elif score <= 60:
            return "Friendly"
        elif score <= 80:
            return "Intimate"
        else:
            return "Close Friend"
```

이 구현에서는 LLM을 사용하여 대화 내용을 분석하고 플레이어의 태도를 자동으로 판단하고 애정을 조정합니다. 이 디자인은 애정 시스템을 더욱 지능적이고 자연스럽게 만들어줍니다. 플레이어는 고의로 NPC를 기쁘게 할 필요 없이 정상적으로 의사소통만 하면 됩니다.

15.3.3 애정은 대화에 영향을 미칩니다

애정은 단순한 숫자가 아니라 실제로 NPC 행동에 영향을 미칩니다. 사이버 타운에서는 NPC가 현재 애정 수준에 따라 응답 스타일을 조정할 수 있도록 NPC 시스템 프롬프트를 수정했습니다.

애정도가 낮을 ​​때 NPC들은 예의바르지만 거리를 두는 태도를 유지합니다. 애정도가 높아지면 NPC는 더욱 열정적이고 말이 많아집니다. 이러한 변경은 시스템 프롬프트를 동적으로 조정하여 이루어집니다.

```python
def create_npc_agent_with_affinity(npc_id: str, name: str, role: str,
                                   personality: str, affinity_level: str):
    """Create NPC Agent with affection"""

    # Adjust prompts based on affection level
    affinity_prompts = {
        "Stranger": "You just met this player, be polite but not overly enthusiastic. Keep replies brief and professional.",
        "Familiar": "You already know this player, can have normal exchanges. Replies should be natural and friendly.",
        "Friendly": "You treat this player as a friend, willing to share more information. Replies should be detailed and enthusiastic.",
        "Intimate": "You trust this player very much, can share private topics. Replies should be full of care.",
        "Close Friend": "You treat this player as your best friend, talk about everything. Replies should be cordial and sincere."
    }

    system_prompt = f"""You are {name}, a {role}.
Your personality traits: {personality}

Current relationship with player: {affinity_level}
{affinity_prompts.get(affinity_level, affinity_prompts["Stranger"])}

You work in the Datawhale office, working with colleagues to promote the development of the open source community.
Please reply naturally based on your role, personality, and relationship with the player.
"""

    # Create Agent
    llm = HelloAgentsLLM()
    agent = SimpleAgent(
        name=name,
        llm=llm,
        system_prompt=system_prompt
    )

    return agent
```

이 디자인은 애정을 가지고 NPC 행동을 동적으로 변화시킵니다. 플레이어는 상호작용이 증가함에 따라 자신을 대하는 NPC의 태도가 점차 변화하여 게임의 몰입도와 재미가 크게 향상되는 것을 확실히 느낄 수 있습니다.

## 15.4 백엔드 서비스 구현

### 15.4.1 FastAPI 애플리케이션 구조

Cyber ​​Town의 백엔드는 Godot 프런트 엔드의 요청 처리, HelloAgents의 NPC 에이전트 호출, NPC 상태 및 애정 관리, 로그 기록을 담당하는 FastAPI 프레임워크를 사용하여 구축되었습니다. 명확한 애플리케이션 구조로 인해 코드를 더 쉽게 유지 관리하고 확장할 수 있습니다.

FastAPI 애플리케이션은 그림 15.10과 같이 다양한 기능을 다양한 파일로 분리하는 모듈식 설계를 채택합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-10.png" alt="" width="85%"/>
<p>그림 15.10 백엔드 애플리케이션 구조</p>
</div>

FastAPI 애플리케이션의 항목 파일인 `main.py`부터 시작해 보겠습니다.

```python
from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel, Field
from typing import Optional
import uvicorn

from agents import NPCAgentManager
from relationship_manager import RelationshipManager
from state_manager import StateManager
from logger import DialogueLogger
from config import settings

# Create FastAPI application
app = FastAPI(
    title="Cyber Town Back-End Service",
    description="AI NPC dialogue system based on HelloAgents",
    version="1.0.0"
)

# Configure CORS, allow Godot front-end access
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Production environment should limit specific domains
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Initialize各个managers
agent_manager = NPCAgentManager()
relationship_manager = RelationshipManager()
state_manager = StateManager()
dialogue_logger = DialogueLogger()

@app.on_event("startup")
async def startup_event():
    """Initialization on application startup"""
    print("=" * 60)
    print("🎮 Cyber Town back-end service starting...")
    print("=" * 60)

    # Initialize NPC Agents
    agent_manager.initialize_npcs()
    print("✅ NPC Agents initialized")

    # Initialize state manager
    state_manager.initialize_npcs()
    print("✅ State manager initialized")

@app.get("/")
async def root():
    """Health check"""
    return {
        "status": "running",
        "message": "Cyber Town back-end service is running",
        "version": "1.0.0",
        "npcs": state_manager.get_npc_count()
    }

if __name__ == "__main__":
    uvicorn.run(
        app,
        host=settings.HOST,
        port=settings.PORT,
        log_level="info"
    )
```

이 기본 프로그램 파일은 FastAPI 애플리케이션의 기본 구조를 정의하고, 원본 간 요청을 허용하도록 CORS 미들웨어를 구성하고, 시작 시 관리자를 초기화합니다. 다음으로 특정 API 경로를 구현하겠습니다.

### 15.4.2 API 경로 설계

Cyber ​​Town의 백엔드는 Godot 프론트엔드의 요청을 처리하기 위해 여러 핵심 API 엔드포인트를 제공해야 합니다. 이 경로를 `main.py`에 추가합니다.

**NPC 상태 가져오기**

이 API는 위치, 사용 여부 등을 포함하여 모든 NPC의 현재 상태를 반환합니다.

```python
from models import NPCStatusResponse

@app.get("/npcs/status", response_model=NPCStatusResponse)
async def get_npc_status():
    """Get status of all NPCs"""
    npcs = state_manager.get_all_npc_states()
    return {"npcs": npcs}

@app.get("/npcs/{npc_id}/status")
async def get_single_npc_status(npc_id: str):
    """Get status of a single NPC"""
    npc = state_manager.get_npc_state(npc_id)
    if not npc:
        raise HTTPException(status_code=404, detail=f"NPC {npc_id} does not exist")
    return npc
```

**대화 인터페이스**

이는 플레이어-NPC 대화를 처리하는 가장 핵심적인 API입니다.

```python
from models import DialogueRequest, DialogueResponse

@app.post("/dialogue", response_model=DialogueResponse)
async def dialogue(request: DialogueRequest):
    """Handle player-NPC dialogue"""
    # 1. Verify NPC exists
    if not agent_manager.has_npc(request.npc_id):
        raise HTTPException(status_code=404, detail=f"NPC {request.npc_id} does not exist")

    # 2. Check if NPC is busy
    if state_manager.is_npc_busy(request.npc_id):
        raise HTTPException(status_code=409, detail=f"NPC {request.npc_id} is talking with another player")

    # 3. Mark NPC as busy
    state_manager.set_npc_busy(request.npc_id, True)

    try:
        # 4. Get current affection
        affinity_info = relationship_manager.get_affinity(
            request.npc_id,
            request.player_name
        )

        # 5. Call Agent to generate reply
        agent = agent_manager.get_agent(request.npc_id, affinity_info["level"])
        reply = agent.run(request.player_message)

        # 6. Update affection
        new_affinity = relationship_manager.update_affinity(
            request.npc_id,
            request.player_name,
            request.player_message,
            reply
        )

        # 7. Record log
        dialogue_logger.log_dialogue(
            npc_id=request.npc_id,
            player_name=request.player_name,
            player_message=request.player_message,
            npc_reply=reply,
            affinity_info=new_affinity
        )

        # 8. Return reply
        return DialogueResponse(
            npc_reply=reply,
            affinity_level=new_affinity["level"],
            affinity_score=new_affinity["score"]
        )

    except Exception as e:
        dialogue_logger.log_error(f"Dialogue processing failed: {str(e)}")
        raise HTTPException(status_code=500, detail=f"Dialogue processing failed: {str(e)}")

    finally:
        # 9. Release NPC status
        state_manager.set_npc_busy(request.npc_id, False)
```

**애정 질문**

이 API를 사용하면 플레이어-NPC 애정을 쿼리할 수 있습니다.

```python
from models import AffinityInfo

@app.get("/affinity/{npc_id}/{player_name}", response_model=AffinityInfo)
async def get_affinity(npc_id: str, player_name: str):
    """Get player-NPC affection"""
    if not agent_manager.has_npc(npc_id):
        raise HTTPException(status_code=404, detail=f"NPC {npc_id} does not exist")

    affinity = relationship_manager.get_affinity(npc_id, player_name)
    return affinity
```

API 경로 호출 흐름은 그림 15.11에 나와 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-11.png" alt="" width="85%"/>
<p>그림 15.11 API 호출 흐름</p>
</div>

15.4.3 상태 관리 및 로깅 시스템

**상태 관리자**

상태 관리자는 위치, 바쁜지 여부, 현재 작업 등을 포함하여 각 NPC의 현재 상태를 추적하는 일을 담당합니다. 이는 NPC가 여러 플레이어와 동시에 대화하는 것을 방지하는 등 동시성 문제를 방지하는 데 중요합니다.

```python
# state_manager.py
from typing import Dict, List, Optional
from datetime import datetime

class StateManager:
    """NPC state manager"""

    def __init__(self):
        self.npc_states: Dict[str, dict] = {}

    def initialize_npcs(self):
        """Initialize NPC states"""
        npcs = [
            {
                "npc_id": "zhang_san",
                "name": "Zhang San",
                "role": "Python Engineer",
                "position": {"x": 300, "y": 200}
            },
            {
                "npc_id": "li_si",
                "name": "Li Si",
                "role": "Product Manager",
                "position": {"x": 500, "y": 200}
            },
            {
                "npc_id": "wang_wu",
                "name": "Wang Wu",
                "role": "UI Designer",
                "position": {"x": 700, "y": 200}
            }
        ]

        for npc in npcs:
            self.npc_states[npc["npc_id"]] = {
                **npc,
                "is_busy": False,
                "current_action": "idle",
                "last_interaction": None
            }

    def get_npc_state(self, npc_id: str) -> Optional[dict]:
        """Get NPC state"""
        return self.npc_states.get(npc_id)

    def get_all_npc_states(self) -> List[dict]:
        """Get all NPC states"""
        return list(self.npc_states.values())

    def is_npc_busy(self, npc_id: str) -> bool:
        """Check if NPC is busy"""
        npc = self.npc_states.get(npc_id)
        return npc["is_busy"] if npc else False

    def set_npc_busy(self, npc_id: str, busy: bool):
        """Set NPC busy status"""
        if npc_id in self.npc_states:
            self.npc_states[npc_id]["is_busy"] = busy
            if busy:
                self.npc_states[npc_id]["last_interaction"] = datetime.now().isoformat()

    def get_npc_count(self) -> int:
        """Get NPC count"""
        return len(self.npc_states)
```

**로깅 시스템**

로깅 시스템은 콘솔과 파일이라는 이중 출력을 구현합니다. 이를 통해 실시간으로 확인하고 이력 기록을 저장하는 것이 편리합니다.

```python
# logger.py
import logging
from datetime import datetime
from pathlib import Path

class DialogueLogger:
    """Dialogue logger"""

    def __init__(self, log_dir: str = "logs"):
        self.log_dir = Path(log_dir)
        self.log_dir.mkdir(exist_ok=True)

        # Create log file name (by date)
        today = datetime.now().strftime("%Y-%m-%d")
        log_file = self.log_dir / f"dialogue_{today}.log"

        # Configure logging
        self.logger = logging.getLogger("DialogueLogger")
        self.logger.setLevel(logging.INFO)

        # Console handler
        console_handler = logging.StreamHandler()
        console_handler.setLevel(logging.INFO)
        console_formatter = logging.Formatter(
            '%(asctime)s - %(levelname)s - %(message)s',
            datefmt='%H:%M:%S'
        )
        console_handler.setFormatter(console_formatter)

        # File handler
        file_handler = logging.FileHandler(log_file, encoding='utf-8')
        file_handler.setLevel(logging.INFO)
        file_formatter = logging.Formatter(
            '%(asctime)s - %(levelname)s - %(message)s',
            datefmt='%Y-%m-%d %H:%M:%S'
        )
        file_handler.setFormatter(file_formatter)

        # Add handlers
        self.logger.addHandler(console_handler)
        self.logger.addHandler(file_handler)

    def log_dialogue(self, npc_id: str, player_name: str,
                    player_message: str, npc_reply: str,
                    affinity_info: dict):
        """Log dialogue"""
        log_message = f"""
{'='*60}
NPC: {npc_id}
Player: {player_name}
Player message: {player_message}
NPC reply: {npc_reply}
Affection: {affinity_info['level']} ({affinity_info['score']}/100)
Interaction count: {affinity_info['interaction_count']}
{'='*60}
"""
        self.logger.info(log_message)

    def log_error(self, error_message: str):
        """Log error"""
        self.logger.error(error_message)
```

이 로깅 시스템은 대화 내용을 파일에 저장하면서 콘솔에 실시간으로 표시합니다. 매일의 로그는 후속 분석을 쉽게 하기 위해 별도의 파일에 저장됩니다.

### 15.4.4 Godot의 장면 시스템 이해하기

게임 장면 구축을 시작하기 전에 먼저 Godot의 핵심 개념인 장면과 노드를 이해해야 합니다. 이것이 Godot와 다른 게임 엔진의 가장 큰 차이점이자 가장 강력한 기능 중 하나입니다.

**노드란 무엇입니까?**

노드는 Godot의 가장 기본적인 빌딩 블록입니다. 노드를 레고 벽돌로 생각할 수 있으며, 각 노드에는 특정 기능이 있습니다. 예를 들어 Sprite2D 노드는 이미지를 표시하는 데 사용되고, AudioStreamPlayer 노드는 오디오를 재생하는 데 사용되며, CharacterBody2D 노드는 캐릭터 물리 이동을 처리하는 데 사용됩니다. Godot는 수백 가지 유형의 노드를 제공하며, 각 노드는 한 가지 일을 잘 수행하는 데 중점을 둡니다.

노드는 부모-자식 관계를 형성하여 트리 구조를 형성할 수 있습니다. 상위 노드는 하위 노드에 영향을 미칠 수 있습니다. 예를 들어 상위 노드를 이동하면 모든 하위 노드가 동시에 이동하고, 상위 노드를 숨기면 모든 하위 노드가 동시에 숨겨집니다. 이러한 계층적 관계를 통해 복잡한 게임 개체를 쉽게 구성하고 관리할 수 있습니다.

**장면이란 무엇입니까?**

장면은 .tscn 파일에 저장된 노드 모음입니다. 장면을 "프리팹"으로 생각할 수 있습니다. 예를 들어, 캐릭터 스프라이트, 충돌체, 음향 효과 등과 같은 모든 관련 노드를 포함하는 "플레이어" 장면을 만들 수 있습니다. 그런 다음 게임에서 이 장면을 여러 번 사용하면 각 사용이 독립적인 인스턴스를 생성합니다.

장면의 힘은 재사용성과 모듈성에 있습니다. 중첩된 구조를 형성하여 다른 장면 내에서 한 장면을 인스턴스화할 수 있습니다. 예를 들어 기본 장면에는 플레이어 장면, 여러 NPC 장면 및 UI 장면이 포함될 수 있습니다. NPC 장면을 수정하면 모든 NPC 인스턴스에 자동으로 영향을 미치므로 개발 및 유지 관리가 크게 단순화됩니다.

**간단한 예**

장면과 노드를 이해하기 위해 간단한 예를 들어보겠습니다. "플레이어" 장면을 만들고 싶다고 가정해 보겠습니다.

```
Player (CharacterBody2D)  ← Root node, responsible for physics movement
├─ AnimatedSprite2D       ← Child node, displays character animation
├─ CollisionShape2D       ← Child node, defines collision shape
└─ Camera2D               ← Child node, camera follows player
```

이 장면에는 트리 구조를 형성하는 4개의 노드가 포함되어 있습니다. CharacterBody2D는 루트 노드이고 나머지 세 개는 하위 노드입니다. 각 노드에 스크립트를 추가하여 동작을 제어하거나 루트 노드에 스크립트를 추가하여 모든 하위 노드를 조정할 수 있습니다.

우리가 메인 씬에서 이 플레이어 씬을 인스턴스화할 때 Godot는 이 전체 노드 트리의 복사본을 생성합니다. 여러 플레이어 인스턴스를 만들 수 있으며 각 인스턴스는 자체 위치, 상태 및 동작과 독립적입니다.

**장면 인스턴스화의 장점**

Cyber ​​Town에는 장산(Zhang San), 리시(Li Si), 왕우(Wang Wu)라는 세 명의 NPC가 있습니다. 씬 시스템을 사용하지 않으면 각 NPC에 대해 별도로 노드를 생성하고 속성을 설정하고 스크립트를 작성해야 하므로 반복 작업이 많이 발생합니다. 장면 시스템을 사용하면 일반 NPC 장면을 만든 다음 이를 세 번 인스턴스화하고 스크립트 매개변수를 통해 다른 이름과 역할 정보를 설정하기만 하면 됩니다.

이 디자인의 이점은 모든 NPC에 새 기능을 추가하려는 경우(예: 머리 위에 대화 거품 표시) NPC 장면만 수정하면 되며 모든 인스턴스에 자동으로 이 기능이 제공된다는 것입니다.

## 15.5 Godot 게임 장면 구성

**왜 Godot를 게임 엔진으로 선택하나요?**

많은 게임 엔진 중에서 우리는 주로 다음 사항을 고려하여 Godot 4.5를 프론트엔드 엔진으로 선택했습니다:

(1) **Godot는 2D 게임 개발에서 자연스러운 이점을 가지고 있습니다**. Cyber ​​Town은 하향식 2D 픽셀 스타일 게임입니다. Godot의 2D 엔진은 매우 성숙하여 TileMap, AnimatedSprite2D, CharacterBody2D 등과 같은 2D 게임용으로 특별히 설계된 노드 유형을 제공합니다. 개발 효율성은 Unity와 같은 엔진보다 훨씬 높습니다. Godot의 장면 시스템을 사용하면 플레이어, NPC, UI와 같은 요소를 독립적인 장면으로 캡슐화한 다음 이를 메인 장면에서 인스턴스화할 수 있습니다. 이 구성 요소 기반 디자인은 우리 요구 사항에 매우 적합합니다.

(2) **Godot는 완전한 오픈 소스이며 무료입니다**. Godot는 로열티 수수료나 수익 공유가 없는 MIT 라이센스를 사용하며 이는 교육 프로젝트 및 오픈 소스 프로젝트에 매우 적합합니다. 라이선스 문제에 대한 걱정 없이 엔진 소스 코드를 자유롭게 수정하고 게임을 상용화할 수 있습니다. 반면 유니티는 강력하지만 2024년 런타임 수수료 정책을 도입해 개발자 커뮤니티에서 광범위한 논란을 불러일으켰다.

(3) **Godot은 학습 비용이 매우 낮습니다**. Godot는 간결하고 이해하기 쉬운 구문과 매우 부드러운 학습 곡선을 갖춘 Python과 유사한 동적으로 유형이 지정된 언어인 GDScript를 주요 스크립팅 언어로 사용합니다. Python에 이미 익숙한 독자의 경우 GDScript를 배우는 데 장벽이 거의 없습니다. 변수 선언, 함수 정의, 제어 흐름 및 기타 구문은 Python과 매우 유사합니다. 몇 시간 내에 게임 스크립트 작성을 시작할 수도 있습니다. Godot의 노드 트리 구조도 매우 직관적입니다. 편집기에서 장면의 계층적 관계를 시각적으로 볼 수 있어 초보자에게 매우 친숙합니다.

(4) **Godot는 Python 백엔드와 매우 간단하게 통합됩니다**. Godot에는 HTTP를 통해 FastAPI 백엔드와 쉽게 통신할 수 있는 내장 HTTPRequest 노드가 있습니다. 게임에서 백엔드 AI 기능을 호출하려면 모든 API 호출을 캡슐화하는 API 클라이언트 스크립트만 생성하면 됩니다. 이러한 프런트엔드와 백엔드 분리 아키텍처를 통해 게임 로직과 AI 로직을 독립적으로 개발하고 테스트할 수 있어 개발 효율성이 크게 향상됩니다.

물론 Godot에는 몇 가지 제한 사항이 있습니다. 예를 들어, Godot의 3D 기능은 여전히 ​​Unreal Engine 및 Unity에 비해 뒤떨어져 있습니다. 대규모 3D 게임을 개발하려면 다른 엔진을 고려해야 할 수도 있습니다. 하지만 2D 게임, 인디 게임, 교육 프로젝트의 경우 Godot가 탁월한 선택입니다.

15.5.1 장면 디자인 및 리소스 구성

Godot의 장면 시스템을 이해한 후 Cyber ​​Town의 장면 디자인을 살펴보겠습니다. 전체 게임은 Main(메인 장면), Player(플레이어), NPC(비플레이어 캐릭터), DialogueUI(대화 인터페이스)의 네 가지 핵심 장면으로 구성됩니다. 각 장면은 별도로 편집하고 테스트한 다음 결합하여 완전한 게임을 구성할 수 있는 독립적인 모듈입니다.

Cyber ​​Town의 장면 구성은 모듈식 디자인을 채택합니다. 먼저 플레이어(플레이어), NPC(비플레이어 캐릭터), DialogueUI(대화 인터페이스)의 세 가지 기본 장면을 만듭니다. 그런 다음 Main(메인 장면)에서 이러한 장면을 인스턴스화하고 결합합니다. 세 NPC(Zhang San, Li Si, Wang Wu)는 모두 동일한 NPC 장면의 인스턴스이며 스크립트 매개변수를 통해 서로 다른 역할 정보가 설정되어 있다는 점은 특히 주목할 가치가 있습니다.

먼저 그림 15.12에 표시된 대로 네 가지 핵심 장면의 구조를 살펴보겠습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-12.png" alt="" width="85%"/>
<p>그림 15.12 사이버 타운의 4가지 핵심 장면</p>
</div>

이 다이어그램은 4개의 독립적인 장면과 내부 구조를 보여줍니다. **장면 1(메인)**은 배경 이미지(Sprite2D), 플레이어 인스턴스, NPC 구성 노드(아래 NPC 인스턴스 3개 포함), 대화 인터페이스 인스턴스, 벽 구성 노드 및 배경 음악을 포함하는 기본 장면입니다. 여기서 Player, NPC_Zhang, NPC_Li, NPC_Wang 및 DialogueUI는 일반 노드가 아닌 장면 인스턴스입니다. **장면 2(플레이어)**는 애니메이션, 충돌, 카메라 및 두 개의 사운드 효과 노드를 포함한 플레이어 캐릭터 구조를 정의합니다. **장면 3(NPC)**은 일반 템플릿입니다. Zhang San, Li Si 및 Wang Wu는 모두 충돌, 애니메이션, 상호 작용 영역 및 두 개의 레이블을 포함하는 이 장면의 인스턴스입니다. **장면 4(DialogueUI)**는 Panel과 다양한 UI 요소가 포함된 CanvasLayer 노드입니다.

장면 인스턴스화 프로세스는 다음과 같이 이해할 수 있습니다: 우리는 NPC의 노드 구조를 정의하는 Godot 편집기에서 NPC.tscn 장면 파일을 만들었습니다. 그런 다음 메인 장면에서 이 NPC 장면을 세 번 "인스턴스화"하여 각각 NPC_Zhang, NPC_Li 및 NPC_Wang이라는 이름의 세 개의 독립 복사본을 만들었습니다. 각 복사본에는 고유한 위치와 상태가 있지만 동일한 노드 구조를 공유합니다. NPC에 새 사운드 효과 노드를 추가하는 등 NPC.tscn을 수정하면 세 인스턴스 모두 자동으로 이 사운드 효과를 얻게 됩니다.

Godot에서 이러한 장면을 만드는 단계는 다음과 같습니다:

1. **플레이어 장면 만들기**: 새 장면을 만들고, 루트 노드로 CharacterBody2D를 선택하고, AnimatedSprite2D, CollisionShape2D, Camera2D, InteractSound 및 RunningSound 하위 노드를 추가하고 Player.tscn으로 저장합니다.

2. **NPC 장면 만들기**: 새 장면을 만들고 루트 노드로 CharacterBody2D를 선택하고 CollisionShape2D, AnimatedSprite2D, InteractionArea(아래 CollisionShape2D가 있는 Area2D), NameLabel 및 DialogueLabel 하위 노드를 추가하고 NPC.tscn으로 저장합니다.

3. **DialogueUI 장면 만들기**: 새 장면을 만들고 CanvasLayer를 루트 노드로 선택하고 Panel 하위 노드를 추가하고 패널 아래에 NPCName, NPCTitle, DialogueText(RichTextLabel), PlayerInput(LineEdit), SendButton 및 CloseButton을 추가하고 DialogueUI.tscn으로 저장합니다.

4. **메인 장면 생성**: 새 장면을 생성하고, 루트 노드로 Node2D를 선택하고, 배경 이미지로 배경(Sprite2D)을 추가하고, 배경 아래에 고래 장식을 추가한 다음 플레이어 장면을 인스턴스화하고, NPC 노드를 생성하고 그 아래에 NPC 장면을 세 번 인스턴스화하고, DialogueUI 장면을 인스턴스화하고, 벽 충돌 구성을 위한 벽 노드를 생성하고, 마지막으로 배경 음악을 재생하기 위해 AudioStreamPlayer를 추가합니다.

이 장면 구성 방법의 장점은 다음과 같습니다. 각 장면은 독립적이며 별도로 테스트할 수 있습니다. NPC는 동일한 장면의 인스턴스를 사용하며 한 번 수정하면 모든 NPC에 영향을 미칩니다. 장면은 결합도가 낮은 신호를 통해 통신하며 유지 관리 및 확장이 쉽습니다.

### 15.5.2 플레이어 제어 구현

플레이어 캐릭터는 게임에서 가장 중요한 요소 중 하나입니다. WASD 이동 제어, 애니메이션 전환, 충돌 감지, NPC와의 상호 작용 및 음향 효과 시스템을 구현해야 합니다.

플레이어 장면 구조에는 다음이 포함됩니다. 물리 이동 및 충돌을 담당하는 루트 노드인 CharacterBody2D; 캐릭터 애니메이션을 표시하는 AnimatedSprite2D; 충돌 모양을 정의하는 CollisionShape2D; 플레이어를 따라가는 Camera2D; 상호 작용 음향 효과와 걷는 음향 효과를 각각 재생하는 두 개의 AudioStreamPlayer.

플레이어 제어 스크립트 `player.gd`는 움직임, 상호 ​​작용 및 음향 효과 논리를 구현합니다.

```python
extends CharacterBody2D

# Movement speed
@export var speed: float = 200.0

# Currently interactable NPC
var nearby_npc: Node = null

# Interaction state (disable movement during interaction)
var is_interacting: bool = false

# Node references
@onready var animated_sprite: AnimatedSprite2D = $AnimatedSprite2D
@onready var camera: Camera2D = $Camera2D

# Sound effect references
@onready var interact_sound: AudioStreamPlayer = null
@onready var running_sound: AudioStreamPlayer = null

# Walking sound effect state
var is_playing_running_sound: bool = false

func _ready():
    # Add to player group (important! NPCs need this group to identify player)
    add_to_group("player")

    # Get sound effect nodes (optional, won't error if doesn't exist)
    interact_sound = get_node_or_null("InteractSound")
    running_sound = get_node_or_null("RunningSound")

    # Enable camera
    camera.enabled = true

    # Play default animation
    if animated_sprite.sprite_frames != null and animated_sprite.sprite_frames.has_animation("idle"):
        animated_sprite.play("idle")

func _physics_process(_delta: float):
    # If interacting, disable movement
    if is_interacting:
        velocity = Vector2.ZERO
        move_and_slide()
        # Play idle animation
        if animated_sprite.sprite_frames != null and animated_sprite.sprite_frames.has_animation("idle"):
            animated_sprite.play("idle")
        # Stop walking sound effect
        stop_running_sound()
        return

    # Get input direction
    var input_direction = Input.get_vector("ui_left", "ui_right", "ui_up", "ui_down")

    # Set velocity
    velocity = input_direction * speed

    # Move
    move_and_slide()

    # Update animation and direction
    update_animation(input_direction)

    # Update walking sound effect
    update_running_sound(input_direction)

func update_animation(direction: Vector2):
    """Update character animation (supports 4 directions)"""
    if animated_sprite.sprite_frames == null:
        return

    # Play animation based on movement direction
    if direction.length() > 0:
        # Moving - determine main direction
        if abs(direction.x) > abs(direction.y):
            # Left-right movement
            if direction.x > 0:
                # Right
                if animated_sprite.sprite_frames.has_animation("walk_right"):
                    animated_sprite.play("walk_right")
                    animated_sprite.flip_h = false
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
                    animated_sprite.flip_h = false
            else:
                # Left
                if animated_sprite.sprite_frames.has_animation("walk_left"):
                    animated_sprite.play("walk_left")
                    animated_sprite.flip_h = false
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
                    animated_sprite.flip_h = true
        else:
            # Up-down movement
            if direction.y > 0:
                # Down
                if animated_sprite.sprite_frames.has_animation("walk_down"):
                    animated_sprite.play("walk_down")
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
            else:
                # Up
                if animated_sprite.sprite_frames.has_animation("walk_up"):
                    animated_sprite.play("walk_up")
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
    else:
        # Idle
        if animated_sprite.sprite_frames.has_animation("idle"):
            animated_sprite.play("idle")

func _input(event: InputEvent):
    # Press E key to interact with NPC
    if event is InputEventKey:
        if event.pressed and not event.echo:
            if event.keycode == KEY_E or event.keycode == KEY_ENTER:
                if nearby_npc != null:
                    interact_with_npc()

func interact_with_npc():
    """Interact with nearby NPC"""
    if nearby_npc != null:
        # Play interaction sound effect
        if interact_sound:
            interact_sound.play()

        # Send signal to dialogue system
        get_tree().call_group("dialogue_system", "start_dialogue", nearby_npc.npc_name)

func set_nearby_npc(npc: Node):
    """Set nearby NPC"""
    nearby_npc = npc

func set_interacting(interacting: bool):
    """Set interaction state"""
    is_interacting = interacting
    if interacting:
        # Stop walking sound effect
        stop_running_sound()

func update_running_sound(direction: Vector2):
    """Update walking sound effect"""
    if running_sound == null:
        return

    # If moving
    if direction.length() > 0:
        # If sound effect not playing yet, start playing
        if not is_playing_running_sound:
            running_sound.play()
            is_playing_running_sound = true
    else:
        # If stopped moving, stop sound effect
        stop_running_sound()

func stop_running_sound():
    """Stop walking sound effect"""
    if running_sound and is_playing_running_sound:
        running_sound.stop()
        is_playing_running_sound = false
```

이 스크립트는 완전한 플레이어 제어를 구현합니다. 플레이어는 WASD 키(또는 화살표 키)를 사용하여 이동하고, 캐릭터는 이동 방향에 따라 해당 4방향 애니메이션 (walk_up/down/left/right)을 재생합니다. 플레이어가 NPC에 접근하면 NPC는 `set_nearby_npc()`를 호출하여 자신을 상호 작용 가능한 개체로 설정하고 플레이어는 E 키를 눌러 상호 작용을 시작할 수 있습니다. 상호 작용 중에 음향 효과가 재생되고 `call_group()`은 대화 시스템에 대화를 시작하라고 알립니다. 대화 중에 `set_interacting(true)`는 플레이어의 움직임을 비활성화하며, 이는 대화가 끝나면 복원됩니다. 걷는 소리 효과는 플레이어가 움직일 때 자동으로 재생되고 멈추면 자동으로 멈춥니다.

15.5.3 NPC 행동과 상호작용

NPC는 장면을 무작위로 순찰하고 돌아다니고, 플레이어 상호 작용에 응답하고, 대화 거품을 표시하는 세 가지 핵심 기능을 구현해야 합니다. Area2D를 사용하여 플레이어가 NPC 근처에 있는지 감지합니다. 플레이어가 상호작용 범위에 들어가면 플레이어에게 알림이 표시되며, E 키를 누르면 대화가 시작됩니다.

NPC 장면 구조에는 다음이 포함됩니다. 루트 노드인 CharacterBody2D; CollisionShape2D는 NPC 충돌 모양을 정의합니다. AnimatedSprite2D는 NPC 애니메이션을 표시합니다. InteractionArea(Area2D)는 상호 작용 범위를 정의하는 아래 CollisionShape2D를 사용하여 플레이어가 상호 작용 범위에 들어가는 것을 감지합니다. NameLabel은 NPC 이름을 표시합니다. DialogueLabel은 대화 풍선을 표시합니다.

NPC 스크립트 `npc.gd`는 순찰, 상호 작용 및 대화 풍선 논리를 구현합니다.

```python
extends CharacterBody2D

# NPC information
@export var npc_name: String = "Zhang San"
@export var npc_title: String = "Python Engineer"

# NPC appearance configuration
@export var sprite_frames: SpriteFrames = null  # Custom sprite frame resource

# NPC movement configuration
@export var move_speed: float = 50.0  # Movement speed
@export var wander_enabled: bool = true  # Whether to enable patrol
@export var wander_range: float = 200.0  # Patrol range
@export var wander_interval_min: float = 3.0  # Minimum patrol interval (seconds)
@export var wander_interval_max: float = 8.0  # Maximum patrol interval (seconds)

# Current dialogue content (obtained from back-end)
var current_dialogue: String = ""

# Node references
@onready var animated_sprite: AnimatedSprite2D = $AnimatedSprite2D
@onready var interaction_area: Area2D = $InteractionArea
@onready var name_label: Label = $NameLabel
@onready var dialogue_label: Label = $DialogueLabel

# Player reference
var player: Node = null

# Patrol-related variables
var wander_target: Vector2 = Vector2.ZERO  # Patrol target position
var wander_timer: float = 0.0  # Patrol timer
var is_wandering: bool = false  # Whether currently patrolling
var is_interacting: bool = false  # Whether currently interacting with player
var spawn_position: Vector2 = Vector2.ZERO  # Spawn position

func _ready():
    # Add to npcs group
    add_to_group("npcs")

    # Set NPC name
    name_label.text = npc_name

    # Connect interaction area signals
    interaction_area.body_entered.connect(_on_body_entered)
    interaction_area.body_exited.connect(_on_body_exited)

    # Initialize dialogue label
    dialogue_label.text = ""
    dialogue_label.visible = false

    # Set custom sprite frames (if any)
    if sprite_frames != null:
        animated_sprite.sprite_frames = sprite_frames

    # Play default animation
    if animated_sprite.sprite_frames != null and animated_sprite.sprite_frames.has_animation("idle"):
        animated_sprite.play("idle")

    # Record spawn position
    spawn_position = global_position

    # Initialize patrol timer
    if wander_enabled:
        wander_timer = randf_range(wander_interval_min, wander_interval_max)
        choose_new_wander_target()

func _on_body_entered(body: Node2D):
    """Player enters interaction range"""
    if body.is_in_group("player"):
        player = body

        if player.has_method("set_nearby_npc"):
            player.set_nearby_npc(self)

func _on_body_exited(body: Node2D):
    """Player leaves interaction range"""
    if body.is_in_group("player"):
        if player != null and player.has_method("set_nearby_npc"):
            player.set_nearby_npc(null)
        player = null

func update_dialogue(dialogue: String):
    """Update NPC dialogue content"""
    current_dialogue = dialogue
    dialogue_label.text = dialogue
    dialogue_label.visible = true

    # Hide dialogue after 10 seconds
    await get_tree().create_timer(10.0).timeout
    dialogue_label.visible = false

func _physics_process(delta: float):
    """Physics update - handle movement"""
    # If interacting with player, stop movement
    if is_interacting:
        velocity = Vector2.ZERO
        move_and_slide()
        # Play idle animation
        if animated_sprite.sprite_frames != null and animated_sprite.sprite_frames.has_animation("idle"):
            animated_sprite.play("idle")
        return

    # If patrol not enabled, don't move
    if not wander_enabled:
        return

    # Update patrol timer
    wander_timer -= delta

    # If timer ends, choose new target and start moving
    if wander_timer <= 0:
        choose_new_wander_target()
        wander_timer = randf_range(wander_interval_min, wander_interval_max)

    # If patrolling, move to target
    if is_wandering:
        # Check if reached target
        if global_position.distance_to(wander_target) < 10:
            # Reached target, stop movement
            is_wandering = false
            velocity = Vector2.ZERO
            move_and_slide()
            # Play idle animation
            if animated_sprite.sprite_frames != null and animated_sprite.sprite_frames.has_animation("idle"):
                animated_sprite.play("idle")
        else:
            # Continue moving to target
            var direction = (wander_target - global_position).normalized()
            velocity = direction * move_speed
            move_and_slide()
            # Update animation
            update_animation(direction)
    else:
        # Stop movement
        velocity = Vector2.ZERO
        move_and_slide()
        # Play idle animation
        if animated_sprite.sprite_frames != null and animated_sprite.sprite_frames.has_animation("idle"):
            animated_sprite.play("idle")

func choose_new_wander_target():
    """Choose new patrol target"""
    # Randomly choose a point near spawn position
    var offset = Vector2(
        randf_range(-wander_range, wander_range),
        randf_range(-wander_range, wander_range)
    )
    wander_target = spawn_position + offset
    is_wandering = true

func update_animation(direction: Vector2):
    """Update animation"""
    if animated_sprite.sprite_frames == null:
        return

    if direction.length() > 0:
        # Movement animation
        if abs(direction.x) > abs(direction.y):
            # Left-right movement
            if direction.x > 0:
                if animated_sprite.sprite_frames.has_animation("walk_right"):
                    animated_sprite.play("walk_right")
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
                    animated_sprite.flip_h = false
            else:
                if animated_sprite.sprite_frames.has_animation("walk_left"):
                    animated_sprite.play("walk_left")
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
                    animated_sprite.flip_h = true
        else:
            # Up-down movement
            if direction.y > 0:
                if animated_sprite.sprite_frames.has_animation("walk_down"):
                    animated_sprite.play("walk_down")
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
            else:
                if animated_sprite.sprite_frames.has_animation("walk_up"):
                    animated_sprite.play("walk_up")
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
    else:
        # Idle animation
        if animated_sprite.sprite_frames.has_animation("idle"):
            animated_sprite.play("idle")

func set_interacting(interacting: bool):
    """Set interaction state"""
    is_interacting = interacting
```

이 스크립트는 완전한 NPC 동작을 구현합니다. NPC는 스폰 위치 주변의 `wander_range` 내를 무작위로 순찰하며 새로운 목표 지점을 선택하고 `wander_interval_min`~`wander_interval_max`초마다 그곳으로 이동합니다. 이동 중에는 4방향 애니메이션 (walk_up/down/left/right)이 재생되며, 목표물에 도달하면 정지 후 대기 애니메이션이 재생됩니다. 플레이어가 InteractionArea에 들어가면 NPC는 플레이어의 `set_nearby_npc(self)` 메서드를 호출하여 자신을 상호 작용 가능한 개체로 설정합니다. 플레이어가 E 키를 누르면 대화 시스템이 NPC의 `set_interacting(true)` 메서드를 호출하고 NPC는 이동을 멈춥니다. 대화가 끝나면 `set_interacting(false)`가 호출되고 NPC가 순찰을 재개합니다. 메인 장면에서는 주기적으로 `update_dialogue()` 메서드를 호출하여 NPC의 대화 풍선을 업데이트하고 NPC 간의 자율적인 대화 내용을 표시합니다.

## 15.6 프런트엔드 및 백엔드 통신 구현

### 15.6.1 API 클라이언트 캡슐화

Godot 프론트엔드는 HTTP를 통해 FastAPI 백엔드와 통신해야 합니다. 모든 API 호출을 캡슐화하는 API 클라이언트 스크립트 `api_client.gd`를 생성하고 다른 스크립트에서 편리하게 사용할 수 있도록 AutoLoad(자동 로드) 싱글톤으로 설정합니다.

API 클라이언트는 Godot의 HTTPRequest 노드를 사용하여 HTTP 요청을 보냅니다. HTTPRequest는 요청을 보낸 후 게임을 차단하지 않고 신호를 통해 요청 완료를 알리는 비동기 노드입니다. 이를 통해 게임 유동성이 보장됩니다. 네트워크 대기 시간이 길어도 끊김 현상이 발생하지 않습니다. 우리는 신호 메커니즘을 사용하여 Wait를 사용하는 대신 다른 스크립트에 API 응답을 알리므로 여러 스크립트가 동일한 API 응답을 동시에 수신할 수 있습니다.

```python
# api_client.gd
extends Node

# Signal definitions
signal chat_response_received(npc_name: String, message: String)
signal chat_error(error_message: String)
signal npc_status_received(dialogues: Dictionary)
signal npc_list_received(npcs: Array)

# HTTP request nodes
var http_chat: HTTPRequest
var http_status: HTTPRequest
var http_npcs: HTTPRequest

func _ready():
    # Create HTTP request nodes
    http_chat = HTTPRequest.new()
    http_status = HTTPRequest.new()
    http_npcs = HTTPRequest.new()

    add_child(http_chat)
    add_child(http_status)
    add_child(http_npcs)

    # Connect signals
    http_chat.request_completed.connect(_on_chat_request_completed)
    http_status.request_completed.connect(_on_status_request_completed)
    http_npcs.request_completed.connect(_on_npcs_request_completed)

# ==================== Chat API ====================
func send_chat(npc_name: String, message: String) -> void:
    """Send chat request"""
    var data = {
        "npc_name": npc_name,
        "message": message
    }

    var json_string = JSON.stringify(data)
    var headers = ["Content-Type: application/json"]

    var error = http_chat.request(
        Config.API_CHAT,
        headers,
        HTTPClient.METHOD_POST,
        json_string
    )

    if error != OK:
        print("[ERROR] Failed to send chat request: ", error)
        chat_error.emit("Network request failed")

func _on_chat_request_completed(_result: int, response_code: int, _headers: PackedStringArray, body: PackedByteArray) -> void:
    """Handle chat response"""
    if response_code != 200:
        print("[ERROR] Chat request failed: HTTP ", response_code)
        chat_error.emit("Server error: " + str(response_code))
        return

    var json = JSON.new()
    var parse_result = json.parse(body.get_string_from_utf8())

    if parse_result != OK:
        print("[ERROR] Failed to parse response")
        chat_error.emit("Response parsing failed")
        return

    var response = json.data

    if response.has("success") and response["success"]:
        var npc_name = response["npc_name"]
        var msg = response["message"]
        print("[INFO] Received NPC reply: ", npc_name, " -> ", msg)
        chat_response_received.emit(npc_name, msg)
    else:
        chat_error.emit("Chat failed")

# ==================== NPC Status API ====================
func get_npc_status() -> void:
    """Get NPC status"""
    # Check if request is being processed
    if http_status.get_http_client_status() != HTTPClient.STATUS_DISCONNECTED:
        print("[WARN] NPC status request is being processed, skipping this request")
        return

    var error = http_status.request(Config.API_NPC_STATUS)

    if error != OK:
        print("[ERROR] Failed to get NPC status: ", error)

func _on_status_request_completed(_result: int, response_code: int, _headers: PackedStringArray, body: PackedByteArray) -> void:
    """Handle NPC status response"""
    if response_code != 200:
        print("[ERROR] NPC status request failed: HTTP ", response_code)
        return

    var json = JSON.new()
    var parse_result = json.parse(body.get_string_from_utf8())

    if parse_result != OK:
        print("[ERROR] Failed to parse NPC status")
        return

    var response = json.data

    if response.has("dialogues"):
        var dialogues = response["dialogues"]
        print("[INFO] Received NPC status update: ", dialogues.size(), " NPCs")
        npc_status_received.emit(dialogues)

# ==================== NPC List API ====================
func get_npc_list() -> void:
    """Get NPC list"""
    var error = http_npcs.request(Config.API_NPCS)

    if error != OK:
        print("[ERROR] Failed to get NPC list: ", error)

func _on_npcs_request_completed(_result: int, response_code: int, _headers: PackedStringArray, body: PackedByteArray) -> void:
    """Handle NPC list response"""
    if response_code != 200:
        print("[ERROR] NPC list request failed: HTTP ", response_code)
        return

    var json = JSON.new()
    var parse_result = json.parse(body.get_string_from_utf8())

    if parse_result != OK:
        print("[ERROR] Failed to parse NPC list")
        return

    var response = json.data

    if response.has("npcs"):
        var npcs = response["npcs"]
        print("[INFO] Received NPC list: ", npcs.size(), " NPCs")
        npc_list_received.emit(npcs)
```

이 API 클라이언트는 채팅 요청 보내기(`send_chat`), NPC 상태 가져오기(`get_npc_status`), NPC 목록 가져오기(`get_npc_list`)의 세 가지 핵심 기능을 캡슐화합니다. 모든 HTTP 요청은 비동기식이며 신호를 통해 응답 결과를 알려줍니다. 우리는 각 API에 대해 독립적인 HTTPRequest 노드를 생성하여 서로 간섭하지 않고 여러 요청을 동시에 보낼 수 있도록 했습니다. API URL은 편리한 통합 관리를 위해 Config 싱글톤에서 가져옵니다. 대화 시스템은 `chat_response_received` 신호를 수신하여 NPC 응답을 수신하고, 메인 장면은 `npc_status_received` 신호를 수신하여 NPC 대화 말풍선을 업데이트합니다.

### 15.6.2 대화 UI 구현

대화 UI는 플레이어-NPC 상호작용을 위한 인터페이스입니다. NPC 이름, 제목, 대화 내용 표시, 입력 상자, 버튼 등을 포함하는 간단하고 아름다운 대화 상자를 디자인해야 합니다.

대화 상자 UI 구조는 그림 15.13에 나와 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-13.png" alt="" width="85%"/>
<p>그림 15.13 대화 UI 구조</p>
</div>

대화 UI 디자인은 매우 간단합니다. DialogueUI는 CanvasLayer 노드입니다. 즉, 항상 게임 화면 위에 표시되고 다른 게임 개체에 의해 가려지지 않습니다. 패널은 화면 하단에 고정된 대화 상자 배경입니다. 패널 아래에는 직접 배치된 6개의 UI 요소가 있습니다. NPCName은 NPC 이름을 표시하고, NPCTitle은 제목을 표시하고, DialogueText는 RichTextLabel을 사용하여 대화 내용을 표시하고(서식 있는 텍스트 형식 지원), PlayerInput은 플레이어 입력을 위한 LineEdit이고, SendButton 및 CloseButton은 각각 메시지를 보내고 대화 상자를 닫는 데 사용됩니다.

대화 UI 스크립트 `dialogue_ui.gd`는 대화 인터페이스 로직을 구현합니다.

```python
# dialogue_ui.gd
extends CanvasLayer

# UI node references
@onready var panel = $Panel
@onready var npc_name_label = $Panel/NPCName
@onready var npc_title_label = $Panel/NPCTitle
@onready var dialogue_text = $Panel/DialogueText
@onready var input_field = $Panel/PlayerInput
@onready var send_button = $Panel/SendButton
@onready var close_button = $Panel/CloseButton

# API client
var api_client: Node = null

# Current NPC in dialogue
var current_npc_name: String = ""

func _ready():
    # Hide dialogue box on initialization
    visible = false

    # Connect button signals
    send_button.pressed.connect(_on_send_button_pressed)
    close_button.pressed.connect(_on_close_button_pressed)
    input_field.text_submitted.connect(_on_text_submitted)

    # Get API client
    api_client = get_node_or_null("/root/APIClient")

func start_dialogue(npc_name: String):
    """Start dialogue with NPC"""
    current_npc_name = npc_name

    # Set NPC information
    npc_name_label.text = npc_name
    npc_title_label.text = get_npc_title(npc_name)

    # Clear dialogue content
    dialogue_text.clear()
    dialogue_text.append_text("[color=gray]Conversation with " + npc_name + " started...[/color]\n")

    # Clear input field
    input_field.text = ""

    # Show dialogue box
    show_dialogue()

    # Focus input field
    input_field.grab_focus()

func show_dialogue():
    """Show dialogue box"""
    visible = true

    # Notify player to enter interaction state (disable movement)
    var player = get_tree().get_first_node_in_group("player")
    if player and player.has_method("set_interacting"):
        player.set_interacting(true)

func hide_dialogue():
    """Hide dialogue box"""
    visible = false
    current_npc_name = ""

    # Notify player to exit interaction state (enable movement)
    var player = get_tree().get_first_node_in_group("player")
    if player and player.has_method("set_interacting"):
        player.set_interacting(false)

func _on_send_button_pressed():
    """Send button clicked"""
    send_message()

func _on_close_button_pressed():
    """Close button clicked"""
    hide_dialogue()

func _on_text_submitted(_text: String):
    """Input field enter pressed"""
    send_message()

func send_message():
    """Send message"""
    var message = input_field.text.strip_edges()

    if message.is_empty():
        return

    if current_npc_name.is_empty():
        return

    # Display player message
    dialogue_text.append_text("\n[color=cyan]Player:[/color] " + message + "\n")

    # Clear input field
    input_field.text = ""

    # Disable input
    input_field.editable = false
    send_button.disabled = true

    # Send API request
    if api_client:
        api_client.send_chat_request(current_npc_name, message)

func on_chat_response_received(npc_name: String, response: String):
    """Received NPC reply"""
    if npc_name == current_npc_name:
        # Display NPC reply
        dialogue_text.append_text("[color=yellow]" + npc_name + ":[/color] " + response + "\n")

        # Enable input
        input_field.editable = true
        send_button.disabled = false
        input_field.grab_focus()

func get_npc_title(npc_name: String) -> String:
    """Get NPC title"""
    var titles = {
        "Zhang San": "Python Engineer",
        "Li Si": "Product Manager",
        "Wang Wu": "UI Designer"
    }
    return titles.get(npc_name, "")
```

이 대화 UI는 완전한 대화 기능을 구현합니다. 플레이어는 메시지를 입력하고 보낼 수 있으며 UI는 RichTextLabel의 Append_text 메서드를 사용하여 대화 내용을 표시하고 서식 있는 텍스트 형식((colors, bold, etc.))을 지원합니다. 모든 API 호출은 비동기식이므로 중복 전송을 방지하기 위해 응답을 기다리는 동안 입력 상자를 비활성화합니다. 대화 상자가 표시되면 플레이어에게 상호 작용 상태로 들어가도록 알리고 이동을 비활성화하며 닫히면 이동을 복원합니다.

### 15.6.3 메인 장면 통합

마지막으로 플레이어 제어, NPC 상호 작용, 대화 UI, NPC 상태 업데이트 등 모든 기능을 메인 장면에 통합해야 합니다. 메인 장면 스크립트 `main.gd`는 이러한 구성 요소를 조정하고 주기적으로 백엔드에서 NPC 상태를 가져와 NPC 대화 풍선을 업데이트합니다.

```python
# main.gd
extends Node2D

# NPC node references
@onready var npc_zhang: Node2D = $NPCs/NPC_Zhang
@onready var npc_li: Node2D = $NPCs/NPC_Li
@onready var npc_wang: Node2D = $NPCs/NPC_Wang

# API client
var api_client: Node = null

# NPC status update timer
var status_update_timer: float = 0.0

func _ready():
    print("[INFO] Main scene initialization")

    # Get API client
    api_client = get_node_or_null("/root/APIClient")
    if api_client:
        api_client.npc_status_received.connect(_on_npc_status_received)

        # Immediately get NPC status once
        api_client.get_npc_status()
    else:
        print("[ERROR] API client not found")

func _process(delta: float):
    # Periodically update NPC status
    status_update_timer += delta
    if status_update_timer >= Config.NPC_STATUS_UPDATE_INTERVAL:
        status_update_timer = 0.0
        if api_client:
            api_client.get_npc_status()

func _on_npc_status_received(dialogues: Dictionary):
    """Received NPC status update"""
    print("[INFO] Update NPC status: ", dialogues)

    # Update each NPC's dialogue
    for npc_name in dialogues:
        var dialogue = dialogues[npc_name]
        update_npc_dialogue(npc_name, dialogue)

func update_npc_dialogue(npc_name: String, dialogue: String):
    """Update specified NPC's dialogue"""
    var npc_node = get_npc_node(npc_name)
    if npc_node and npc_node.has_method("update_dialogue"):
        npc_node.update_dialogue(dialogue)

func get_npc_node(npc_name: String) -> Node2D:
    """Get NPC node by name"""
    match npc_name:
        "Zhang San":
            return npc_zhang
        "Li Si":
            return npc_li
        "Wang Wu":
            return npc_wang
        _:
            return null
```

메인 씬 스크립트의 핵심 기능은 백엔드에서 NPC 상태를 주기적으로 얻는 것입니다. `_ready()`에서는 APIClient 싱글톤에 대한 참조를 얻고 `npc_status_received` 신호를 연결합니다. 그런 다음 즉시 `get_npc_status()`를 호출하여 NPC 상태를 한 번 가져옵니다. `_process()`에서는 타이머를 사용하여 `Config.NPC_STATUS_UPDATE_INTERVAL`초(기본값 30초)마다 `get_npc_status()`를 호출합니다. NPC 상태 업데이트가 수신되면 `_on_npc_status_received()` 콜백 함수는 모든 NPC를 순회하고 해당 `update_dialogue()` 메서드를 호출하여 대화 풍선을 업데이트합니다. 이렇게 하면 플레이어가 NPC와 상호 작용하지 않더라도 NPC 간의 자율적인 대화를 계속 볼 수 있습니다.

전체 프런트엔드 및 백엔드 통신 프로세스는 그림 15.14에 나와 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-14.png" alt="" width="85%"/>
<p>그림 15.14 완전한 프론트엔드 및 백엔드 통신 프로세스</p>
</div>

이 시점에서 모든 프런트엔드 및 백엔드 통신 기능이 구현되었습니다. 플레이어는 게임 내에서 자유롭게 이동하고, NPC와 상호 작용하며, 자연스러운 언어 대화를 나눌 수 있습니다. 한편, 메인 씬은 주기적으로 백엔드에서 NPC 상태를 획득하고 NPC 대화 풍선을 업데이트하며 NPC 간의 자율적인 대화를 표시합니다. 전체 시스템은 구성 요소 간의 느슨한 결합을 통해 통신용 신호 메커니즘을 사용하므로 유지 관리 및 확장이 쉽습니다.

## 15.7 요약 및 전망

### 15.7.1 장 검토

이번 장에서는 전체 AI 타운 프로젝트인 Cyber ​​Town을 완성했습니다. 이 프로젝트는 HelloAgents 프레임워크와 Godot 게임 엔진을 결합하여 생생한 가상 세계를 만듭니다. 우리가 배운 핵심 내용을 복습해 봅시다.

**기술적 아키텍처 설계**

우리는 게임 엔진 + 백엔드 서비스의 분리된 아키텍처를 채택하여 프런트엔드 렌더링, 백엔드 로직, AI 인텔리전스를 서로 다른 레이어로 분리했습니다. Godot는 게임 그래픽과 플레이어 상호 작용을 처리하고, FastAPI는 API 서비스와 상태 관리를 처리하며, HelloAgents는 NPC 지능과 메모리 시스템을 처리합니다. 이러한 계층형 설계를 통해 각 부품을 독립적으로 개발하고 테스트할 수 있으며 향후 확장을 위한 좋은 기반도 제공됩니다.

**NPC 에이전트 시스템**

HelloAgents의 SimpleAgent를 사용하여 각 NPC에 대한 독립적인 에이전트를 만들었습니다. 각 NPC에는 고유한 역할 설정, 성격 특성 및 기억 시스템이 있습니다. 세심하게 설계된 시스템 프롬프트를 통해 우리는 Zhang San을 엄격한 Python 엔지니어로, Li Si를 커뮤니케이션에 능숙한 제품 관리자로, Wang Wu를 창의적인 UI 디자이너로 만들었습니다. 이러한 NPC는 플레이어의 대화를 이해할 수 있을 뿐만 아니라 역할 특성에 따라 반응할 수도 있습니다.

**기억과 애정 시스템**

우리는 2계층 메모리 시스템을 구현했습니다. 단기 기억은 대화 일관성을 유지하고 장기 기억은 모든 상호 작용 기록을 저장합니다. NPC는 벡터 데이터베이스의 의미 검색을 통해 이전에 논의된 주제를 기억할 수 있습니다. 애정 시스템을 통해 플레이어에 대한 NPC의 태도는 낯선 사람부터 가까운 친구까지, 각 레벨마다 다른 행동 표현을 통해 상호 작용에 따라 바뀔 수 있습니다. 이러한 디자인은 NPC를 더욱 현실적이고 흥미롭게 보이게 만듭니다.

**게임 장면 구성**

우리는 Godot를 사용하여 픽셀 스타일의 사무실 장면을 만들고 플레이어 제어, NPC 방황, 상호 작용 감지 및 대화 UI를 구현했습니다. 장면 시스템의 모듈식 설계를 통해 새로운 NPC, 새로운 장면, 새로운 기능을 쉽게 추가할 수 있습니다. GDScript의 간결한 구문은 게임 로직 구현을 직관적이고 효율적으로 만듭니다.

**프런트엔드 및 백엔드 통신**

우리는 HTTP REST API를 사용하여 Godot 프런트엔드와 FastAPI 백엔드 간의 통신을 구현했습니다. 비동기식 요청 및 신호 시스템을 통해 게임 유동성을 보장했습니다. 네트워크 지연 시간이 길어도 플레이어 경험에는 영향을 미치지 않습니다. API 클라이언트 캡슐화를 통해 다른 스크립트에서 백엔드 서비스를 편리하게 호출할 수 있으며, 다이얼로그 UI 구현을 통해 플레이어가 NPC와 자연스럽게 소통할 수 있습니다.

프로젝트의 기술 스택은 그림 15.15에 나와 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-15.png" alt="" width="85%"/>
<p>그림 15.15 사이버 타운 기술 스택</p>
</div>

### 15.7.2 확장 방향

사이버 타운은 단지 시작점일 뿐이며 확장 방향은 다양합니다. 이러한 확장 기능은 게임의 재미를 향상시킬 뿐만 아니라 게임에서 AI 기술의 더 많은 가능성을 탐색할 수 있습니다.

**(1) 멀티플레이어 온라인 지원**

현재 Cyber ​​Town은 싱글 플레이어 게임이지만 멀티 플레이어 온라인 게임으로 확장할 수 있습니다. 여러 플레이어가 동시에 동일한 사무실에 입장하여 NPC 및 다른 플레이어와 상호 작용할 수 있습니다. 이를 위해서는 실시간 통신을 위한 WebSocket과 플레이어 데이터 및 NPC 상태를 유지하는 데이터베이스를 도입해야 합니다. NPC는 다양한 플레이어와의 상호 작용을 기억하고 각 플레이어에 대한 독립적인 애정 수준을 유지할 수 있습니다.

**(2) 퀘스트 시스템**

NPC를 위한 퀘스트 시스템을 설계할 수 있습니다. 플레이어의 NPC에 대한 호감도가 일정 수준에 도달하면 NPC는 특별한 퀘스트를 제공합니다. 예를 들어, Zhang San은 플레이어에게 코드 디버깅을 도와달라고 요청할 수 있고, Li Si는 플레이어에게 사용자 피드백 수집을 요청하고, Wang Wu는 플레이어에게 디자인 제안을 평가하도록 요청할 수 있습니다. 퀘스트를 완료하면 보상을 받을 수 있고, 애정도도 더욱 높아질 수 있습니다.

**(3) NPC 간 상호작용**

현재 NPC는 플레이어와만 상호 작용하지만 NPC가 서로 상호 작용할 수 있도록 설정할 수 있습니다. Zhang San은 Li Si와 제품 요구 사항을 논의할 수 있고, Li Si는 Wang Wu와 인터페이스 디자인을 논의할 수 있으며, Wang Wu는 Zhang San과 기술 구현을 논의할 수 있습니다. 이러한 상호 작용은 백그라운드에서 자동으로 발생할 수 있으며, 플레이어는 NPC 간의 대화를 관찰할 수 있어 전체 세계가 더욱 생생하게 보입니다.

**(4) 감정 시스템**

애정 외에도 NPC에 대한 보다 복잡한 감정 시스템을 추가할 수 있습니다. NPC는 행복, 슬픔, 분노, 흥분 등 다양한 감정 상태를 가질 수 있으며 이는 NPC 응답 스타일과 행동에 영향을 미칩니다. 예를 들어, NPC가 기분이 좋으면 정보를 더 기꺼이 공유할 것입니다. 기분이 좋지 않을 때는 다소 추울 수도 있습니다.

**(5) 동적 이벤트 시스템**

우리는 게임 세계를 더욱 풍요롭게 만들기 위해 역동적인 이벤트를 디자인할 수 있습니다. 예를 들어, 모든 NPC와 플레이어가 모여 프로젝트 진행 상황을 논의하는 팀 회의를 정기적으로 개최합니다. 또는 NPC의 생일을 축하하는 생일 파티를 열 수도 있습니다. 또는 모두의 협력이 필요한 긴급 작업. 이러한 이벤트는 게임의 다양성과 재미를 높일 수 있습니다.

**(6) 더 큰 세계**

현재 Cyber ​​Town에는 사무실 장면이 하나뿐이지만 더 큰 세계로 확장할 수 있습니다. 카페, 도서관, 공원 등 각각 다른 NPC와 상호 작용 방법을 사용하는 다양한 장면을 추가할 수 있습니다. 플레이어는 다양한 장면 사이를 이동하고 더 넓은 가상 세계를 탐색할 수 있습니다.

**(7) 맞춤형 학습**

NPC는 각 플레이어의 선호도와 습관을 배울 수 있습니다. 예를 들어, 플레이어가 Zhang San과 Python에 대해 자주 토론한다면 NPC는 플레이어가 프로그래밍에 관심이 있다는 것을 기억하고 향후 관련 콘텐츠를 적극적으로 공유할 것입니다. 플레이어가 밤에 게임하는 것을 좋아한다면 NPC는 이 시간 습관을 기억하고 밤에 더 활동적일 것입니다.

15.7.3 성찰과 전망

사이버 타운은 게임에서 AI 기술의 엄청난 잠재력을 보여줍니다. 기존 게임의 NPC는 사전 설정된 대화 트리와 스크립트에 의해 제한되는 반면, AI NPC는 자연어를 이해하고 생성하여 플레이어와 실제 대화를 나눌 수 있습니다. 이는 게임 몰입도를 높일 뿐만 아니라 게임 디자인에 새로운 가능성을 열어줍니다.

그러나 AI NPC도 몇 가지 어려움에 직면해 있습니다. 첫 번째는 비용 문제입니다. 각 대화에는 LLM API 호출이 필요하며, 이로 인해 특정 비용이 발생합니다. 대규모 멀티플레이어 온라인 게임의 경우 이 비용이 매우 높을 수 있습니다. 두 번째는 대기 시간 문제입니다. LLM 추론에는 시간이 걸리며, 네트워크 대기 시간이 길면 플레이어가 NPC 응답을 보기 위해 몇 초 정도 기다려야 할 수도 있습니다. 마지막으로 콘텐츠 제어 문제가 있습니다. LLM에서 생성된 콘텐츠는 완전히 제어할 수 없으므로 잘 설계된 프롬프트와 콘텐츠 필터링 메커니즘이 필요합니다.

이러한 어려움에도 불구하고 AI NPC의 미래는 여전히 가능성으로 가득 차 있습니다. LLM 기술이 발전할수록 추론 속도는 빨라지고 비용은 낮아질 것입니다. 현지화된 소규모 LLM도 빠르게 발전하고 있습니다. 미래에는 네트워크 요청이 전혀 필요 없이 플레이어의 장치에서 직접 실행될 수 있습니다. AI 기술과 게임의 결합은 플레이어에게 전례 없는 경험을 선사할 것입니다.

Part 5의 졸업 프로젝트 장에서는 단일 에이전트와 다중 에이전트를 사용하여 일반 에이전트를 구성하는 방법을 알아봅니다. 이는 여러분의 창의적인 시간이 될 것이므로 계속 지켜봐 주시기 바랍니다!
