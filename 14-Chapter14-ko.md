# 14장: 자동화된 심층 연구 에이전트

13장의 여행 도우미 프로젝트에서는 멀티 에이전트 제품에 HelloAgents를 적용하는 방법을 경험했습니다. 이 장에서는 계속해서 **지식 집약적 애플리케이션**: **심층 연구 작업을 자동으로 실행할 수 있는 Agent Assistant 구축**에 중점을 둡니다.

여행 계획에 비해 심층 연구의 어려움은 정보의 지속적인 발산, 사실의 빠른 업데이트, 인용 출처에 대한 사용자의 높은 요구 사항에 있습니다. 신뢰할 수 있는 연구 보고서를 제공하려면 상담원에게 다음 세 가지 핵심 기능을 갖추어야 합니다.

**(1) 문제 분석**: 사용자의 공개 주제를 검색 가능한 쿼리문으로 분해합니다.

**(2) 다단계 정보 수집**: 다양한 검색 API를 결합하여 자료를 지속적으로 채굴하고 중복을 제거하고 통합합니다.

**(3) 반영 및 요약**: 단계 결과를 기반으로 지식 격차를 식별하고 검색을 계속할지 여부를 결정하며 구조화된 요약을 생성합니다.

## 14.1 프로젝트 개요 및 아키텍처 설계

14.1.1 심층 연구 보조원이 필요한 이유

정보 폭발 시대에 우리는 매일 새로운 기술이나 개념, 사건을 빠르게 이해해야 합니다. 전통적인 연구 방법에는 몇 가지 문제점이 있습니다. 첫 번째는 **정보 과부하**입니다. 검색 엔진은 수천 개의 결과를 반환하므로 유용한 정보를 찾으려면 링크를 하나씩 클릭하고 많은 콘텐츠를 읽어야 합니다. 두 번째는 **구조의 부족**입니다. 관련 정보를 찾았더라도 이 정보는 단편화되어 있고 체계적인 구성이 부족한 경우가 많습니다. 마지막으로 **반복노동**입니다. 새로운 주제를 연구할 때마다 '검색 → 읽기 → 요약 → 정리'의 과정을 반복해야 합니다.

이것이 심층연구조교가 해결해야 할 문제이다. 단순한 검색 도구가 아닌, 자율적으로 계획하고 실행하고 요약할 수 있는 연구 조교입니다.

**Deep Research Assistant의 핵심 가치:**

1. **시간 절약**: 1~2시간의 연구 작업을 5~10분으로 압축
2. **품질 향상**: 중요한 정보의 누락을 방지하기 위한 체계적인 조사 프로세스
3. **추적 가능**: 쉽게 확인하고 인용할 수 있도록 모든 검색 결과와 출처를 기록합니다.
4. **확장 가능**: 새로운 검색 엔진, 데이터 소스 및 분석 도구를 쉽게 추가

### 14.1.2 기술 아키텍처 개요

이 시스템은 그림 14.1에 표시된 것처럼 여전히 고전적인 **프런트엔드 및 백엔드 분리 아키텍처**를 채택하고 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-1.png" alt="" width="85%"/>
<p>그림 14.1 심층 연구 보조 기술 아키텍처</p>
</div>

이 시스템은 4계층 아키텍처로 설계되었습니다.

**프런트 엔드 레이어(Vue3+TypeScript)**: 전체 화면 모달 대화 상자 UI, 마크다운 결과 시각화

**백엔드 계층(FastAPI)**: API 라우팅(`/research/stream`)

**에이전트 레이어(HelloAgents)**: 3개의 전문 에이전트(TODO Planner, Task Summarizer, Report Writer) + 2개의 핵심 도구(SearchTool, NoteTool)

**외부 서비스 계층**: 검색 엔진 + LLM 제공업체

그림 14.2에 표시된 것처럼 완전한 조사 요청이 시스템을 통해 어떻게 흐르는지 살펴보겠습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-2.png" alt="" width="85%"/>
<p>그림 14.2 심층 연구 보조 데이터 흐름 프로세스</p>
</div>

1. **사용자 입력**: 사용자가 프런트엔드에 연구 주제를 입력합니다.
2. **프런트엔드 전송**: 프런트엔드는 SSE를 통해 `/research/stream`에 연결됩니다.
3. **백엔드 수신**: FastAPI가 요청을 수신하고 연구 상태를 생성합니다.
4. **계획 단계**: 연구 기획 에이전트를 호출하고 3개의 하위 작업으로 분해
5. **실행 단계**: 각 하위 작업을 하나씩 실행합니다.
   - SearchTool을 사용하여 검색하세요.
   - 작업 요약 에이전트를 호출하여 요약합니다.
   - NoteTool을 사용하여 결과 기록
6. **보고 단계**: 보고서 생성 에이전트 호출, 모든 요약 통합
7. **스트림 반환**: SSE를 통해 진행 상황과 결과를 프런트엔드로 푸시합니다.
8. **프런트 엔드 디스플레이**: 프런트 엔드는 작업 상태, 진행률 표시줄, 로그 및 보고서를 실시간으로 업데이트합니다.

프로젝트 디렉토리 구조는 다음과 같습니다.

```
helloagents-deepresearch/
├── backend/                    # Back-end code
│   ├── src/
│   │   ├── agent.py           # Core coordinator
│   │   ├── main.py            # FastAPI entry
│   │   ├── models.py          # Data models
│   │   ├── prompts.py         # Prompt templates
│   │   ├── config.py          # Configuration management
│   │   └── services/          # Service layer
│   │       ├── planner.py     # Planning service
│   │       ├── summarizer.py  # Summarization service
│   │       ├── reporter.py    # Report service
│   │       └── search.py      # Search service
│   ├── .env                   # Environment variables
│   ├── pyproject.toml         # Dependency management
│   └── workspace/             # Research notes
│
└── frontend/                   # Front-end code
    ├── src/
    │   ├── App.vue            # Main component
    │   ├── components/        # UI components
    │   │   └── ResearchModal.vue
    │   └── composables/       # Composable functions
    │       └── useResearch.ts
    ├── package.json           # npm dependencies
    └── vite.config.ts         # Build configuration
```

### 14.1.3 빠른 경험: 5분 안에 프로젝트 실행

구현 세부 사항을 살펴보기 전에 먼저 프로젝트를 실행하여 최종 결과를 살펴보겠습니다. 이렇게 하면 전체 시스템을 직관적으로 이해할 수 있습니다.

다음 명령을 사용하여 버전을 확인할 수 있습니다.

```bash
python --version  # Should show Python 3.10.x or higher
node --version    # Should show v16.x.x or higher
npm --version     # Should show 8.x.x or higher
```

**(1) 백엔드 시작**

```bash
# 1. Enter back-end directory
cd helloagents-deepresearch/backend

# 2. Install dependencies
# Method 1: Using uv (recommended, faster Python package manager)
uv sync

# Method 2: Using pip
pip install -e .

# 3. Configure environment variables
cp .env.example .env

# 4. Edit .env file, fill in your API keys
# Open .env file with your favorite editor
# At minimum, configure:
# - LLM_PROVIDER (e.g., openai, deepseek, qwen)
# - LLM_API_KEY (your LLM API key)
# - SEARCH_API (e.g., duckduckgo, tavily)

# 5. Start back-end
python src/main.py
```

모든 것이 정상이면 다음과 유사한 출력이 표시됩니다.

```
INFO:     Started server process [12345]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

**(2) 프런트엔드 시작**

새 터미널 창을 엽니다.

```bash
# 1. Enter front-end directory
cd helloagents-deepresearch/frontend

# 2. Install dependencies
npm install

# 3. Start front-end
npm run dev
```

모든 것이 정상이면 다음과 유사한 출력이 표시됩니다.

```
  VITE v5.0.0  ready in 500 ms

  ➜  Local:   http://localhost:5174/
  ➜  Network: use --host to expose
  ➜  press h + enter to show help
```

**(3) 연구 시작**

브라우저를 열고 `http://localhost:5174`를 방문하세요. 그림 14.3과 같이 중앙에 위치한 입력 카드를 볼 수 있습니다. 연구 주제(예: `What kind of organization is Datawhale?`)를 입력하고 검색 엔진(여러 개가 구성된 경우)을 선택한 다음 "연구 시작" 버튼을 클릭하세요.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-3.png" alt="" width="85%"/>
<p>그림 14.3 심층 연구 보조 검색 페이지</p>
</div>

그림 14.4와 같이 시스템이 자동으로 전체 화면으로 확장되어 왼쪽에는 연구 정보가, 오른쪽에는 연구 진행 및 결과가 실시간으로 표시됩니다. 전체 조사 과정은 주제의 복잡성과 검색 엔진의 응답 속도에 따라 약 1~3분 정도 소요됩니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-4.png" alt="" width="85%"/>
<p>그림 14.4 심층 연구 보조 확장 연구</p>
</div>

연구가 완료되면 다음이 표시됩니다.

- **작업 목록**: 모든 하위 작업과 해당 상태를 표시합니다.
- **진행 로그**: 연구 과정 중 모든 작업을 표시합니다.
- **최종 보고서**: 모든 하위 작업과 소스 인용의 요약이 포함된 구조화된 마크다운 보고서

이제 심층 연구 보조원을 성공적으로 실행하고 시스템을 직관적으로 이해하게 되었습니다.

## 14.2 TODO 중심 연구 패러다임

14.2.1 TODO 중심 연구란?

기존 검색 엔진은 단일 질문에만 답할 수 있는 반면 심층 연구는 일련의 관련 질문에 답해야 합니다. TODO 중심 연구 패러다임은 복잡한 연구 주제를 여러 하위 작업(TODO)으로 분해하고 하나씩 실행하고 결과를 통합합니다.

이 패러다임의 핵심 아이디어는 **복잡한 '연구' 업무를 '기획 → 실행 → 통합' 과정으로 전환**하는 것입니다.

예를 통해 이러한 변환을 이해해 보겠습니다. "Datawhale은 어떤 조직인가요?"에 대해 조사하고 싶다고 가정해 보겠습니다. 전통적인 검색 방법은 다음과 같습니다.

```
User input: What kind of organization is Datawhale?
Search engine: Returns 10-20 links
User: Click on links one by one, read content, take notes
Result: Fragmented information, lacking systematization
```

이 접근 방식의 문제점은 각 링크가 주제의 한 측면만 다루고 체계적인 구조가 부족하며 수동 구성 및 요약이 필요하다는 것입니다.

**TODO 기반 접근 방식: 체계적인 연구**

```
User input: What kind of organization is Datawhale?

System planning:
  ├─ TODO 1: Basic information about Datawhale (organizational positioning)
  ├─ TODO 2: Main projects of Datawhale (core content)
  ├─ TODO 3: Community culture of Datawhale (values)
  └─ TODO 4: Influence of Datawhale (social contribution)

System execution:
  For each TODO:
    1. Search for relevant materials
    2. Summarize key information
    3. Record source citations

System integration:
  Generate structured report:
    ├─ Part 1: Organizational positioning (from TODO 1)
    ├─ Part 2: Core content (from TODO 2)
    ├─ Part 3: Values (from TODO 3)
    ├─ Part 4: Social contribution (from TODO 4)
    └─ References: All source citations
```

이 접근 방식의 장점은 복잡한 주제를 명확한 하위 질문으로 분해하고, 각 하위 작업에 대한 검색 결과와 요약을 기록하여 쉽게 추적할 수 있으며, 체계적인 연구 프로세스를 통해 중요한 정보의 누락을 방지할 수 있다는 것입니다. 새로운 하위 작업을 추가하거나 실행 순서를 조정하는 것도 쉽습니다.

완전한 TODO 중심 연구 시스템에는 세 가지 핵심 요소가 포함됩니다.

**(1) 지능형 플래너(TODO Planner)**: 연구 주제를 하위 작업으로 분해하는 역할을 담당합니다. 좋은 기획자는 주제의 주요 측면과 연구 목적을 이해하고, 주제를 3~5개의 하위 작업으로 분해하고(너무 적으면 모든 것을 다루지 못하고, 너무 많으면 중복됨) 각 하위 작업에 적합한 검색 쿼리를 설계해야 합니다.

**(2) 작업 실행자**: 각 하위 작업을 실행하는 역할을 담당합니다. 집행자는 검색 엔진을 사용하여 관련 자료를 얻고, 주요 정보를 추출하고, 중복된 내용을 제거하는 동시에, 쉬운 확인을 위해 모든 출처 인용을 저장해야 합니다.

**(3) 보고서 작성자**: 모든 하위 작업의 결과를 통합하는 역할을 담당합니다. 생성기는 콘텐츠를 논리적 순서로 구성하고, 중복된 정보를 병합하고, 각 관점에 대한 출처 인용을 추가해야 합니다.

우리의 경우 TODO 중심 연구 프로세스가 그림 14.5에 나와 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-5.png" alt="" width="85%"/>
<p>그림 14.5 TODO 중심 연구 프로세스</p>
</div>

전체 프로세스는 선형이지만 각 단계에는 명확한 입력과 출력이 있습니다. 이러한 설계를 통해 시스템을 쉽게 이해하고 디버깅할 수 있습니다.

14.2.2 3단계 연구 프로세스

TODO 중심 연구 프로세스는 계획, 실행, 보고의 세 단계로 구분됩니다. 각 단계마다 이를 담당하는 전담 에이전트가 있습니다.

**(1) 1단계: 계획**

기획 단계의 목표는 연구 주제를 3~5개의 하위 작업으로 분해하는 것입니다. 시스템은 연구 주제와 현재 날짜를 입력으로 받고 JSON 형식의 하위 작업 목록을 출력합니다. 각 하위 작업에는 제목(작업 제목), 의도(연구 의도) 및 쿼리(검색 쿼리)의 세 가지 필드가 포함됩니다.

연구 기획 Agent는 주제 특성에 따라 다양한 분해 전략을 채택하며, 일반적으로 기본 개념부터 시작하여 기술 현황, 실제 적용 및 개발 동향을 이해하고 필요한 경우 비교 분석을 수행합니다. 예를 들어 "Datawhale은 어떤 종류의 조직인가요?"에 대해 계획 에이전트는 다음 하위 작업을 생성할 수 있습니다.

```json
[
  {
    "title": "Basic information about Datawhale",
    "intent": "Understand Datawhale's organizational positioning, founding time, development history",
    "query": "Datawhale organization introduction history 2024"
  },
  {
    "title": "Main projects of Datawhale",
    "intent": "Understand Datawhale's core open source projects and tutorials",
    "query": "Datawhale projects tutorials open source 2024"
  },
  ...
]
```

좋은 계획은 포괄적이고, 논리적으로 명확해야 하며, 정확한 쿼리와 적절한 수의 항목이 있어야 합니다.

**(2) 2단계: 실행**

실행단계에서는 각 하위과제를 하나씩 실행하며 관련 자료를 검색하고 요약한다. 시스템은 하위 작업 목록과 검색 엔진 구성을 입력으로 받고, 각 하위 작업에 대한 요약(마크다운 형식)과 소스 인용 목록을 출력합니다. 실행 과정은 다음과 같습니다.

각 하위 작업에 대해 실행자는 다음을 수행합니다.

1. **재료 검색**: 구성된 검색 엔진을 사용하여 검색을 실행합니다.

   ```python
   search_results = search_tool.run({
       "input": task.query,
       "backend": "tavily",
       "mode": "structured",
       "max_results": 5
   })
   ```

2. **검색결과 가져오기**: 제목, URL, 내용 추출

   ```json
   {
     "results": [
       {
         "title": "What is a Multimodal Model?",
         "url": "https://example.com/multimodal-model",
         "snippet": "A multimodal model is an AI model that can process multiple types of data..."
       },
       ...
     ]
   }
   ```

3. **통화 요약 상담사** : 검색 결과를 요약

   ```python
   summary = summarizer_agent.run(
       task=task,
       search_results=search_results
   )
   ```

4. **기록 요약 및 출처**: NoteTool에 저장

   ```python
   note_tool.run({
       "action": "create",
       "title": task.title,
       "content": f"## {task.title}\n\n{summary}\n\n## Sources\n{sources}",
       "tags": ["research", "summary"]
   })
   ```

작업 요약 에이전트는 각 검색 결과에서 핵심 관점을 추출하고, 유사한 정보를 병합하고, 중요한 숫자, 날짜, 이름 및 기타 주요 데이터를 유지하고, 각 관점에 대한 소스 인용을 추가합니다. 예를 들어 "Datawhale에 대한 기본 정보"라는 검색 결과에 대해 요약 에이전트는 다음을 생성할 수 있습니다.

```markdown
## Basic Information about Datawhale

Datawhale is an open source organization focused on data science and AI, founded in 2018[1]. The organization's core mission is "for the learner, grow together with learners", committed to building a pure learning community[2].

**Core Positioning:**

1. **Open Source Education Platform**: Provides high-quality AI and data science learning resources[1]
2. **Learner Community**: Gathers tens of thousands of AI learners and practitioners[3]
3. **Knowledge Sharing**: Advocates open source spirit, all content is completely free and open[2]

**Development History:**

- **2018**: Datawhale was founded, released first open source tutorial[1]
- **2020**: Became one of the leading AI learning communities in China[3]
- **2024**: Released 50+ open source projects, impacting 100,000+ learners[4]

## Sources

[1] https://github.com/datawhalechina
[2] https://datawhale.club/about
[3] https://www.zhihu.com/org/datawhale
[4] https://datawhale.cn
```

실행 중에 시스템은 실시간으로 진행 정보를 프런트엔드에 푸시합니다.

```json
{
  "type": "status",
  "message": "Searching: Basic information about Datawhale"
}
```

```json
{
  "type": "status",
  "message": "Summarizing search results..."
}
```

```json
{
  "type": "task",
  "task": {
    "id": 1,
    "title": "Basic information about Datawhale",
    "status": "completed"
  }
}
```

**(3) 3단계: 보고**

보고 단계의 목표는 모든 하위 작업의 요약을 통합하여 최종 보고서를 생성하는 것입니다. 시스템은 모든 하위 작업의 요약과 연구 주제를 입력으로 받아 최종 보고서를 Markdown 형식으로 출력합니다. 보고서는 제목, 개요, 각 하위과제에 대한 세부 분석, 요약, 참고자료 등 5개 부분으로 구성됩니다. 예를 들어 "Datawhale은 어떤 조직인가요?"의 경우 최종 보고서는 다음과 같습니다.

```markdown
# What Kind of Organization is Datawhale?

## Overview

This report systematically researched the open source organization Datawhale, covering four aspects: basic information, main projects, community culture, and influence.

## 1. Basic Information about Datawhale

Datawhale is an open source organization focused on data science and AI, founded in 2018...

(Insert summary of subtask 1 here)

## 2. Main Projects of Datawhale

Datawhale has released multiple high-quality open source tutorials, including Hello-Agents, Joyful-Pandas, etc...

(Insert summary of subtask 2 here)
...
## Summary

Through this research, we learned about Datawhale's organizational positioning, core projects, community culture, and social contributions. Datawhale is a pure learning community that has made important contributions to AI education.

## References

[1] https://github.com/datawhalechina
[2] https://datawhale.club/about
...
```

보고서 생성 에이전트는 하위 작업의 논리적 순서로 콘텐츠를 구성하고, 시작 부분에 간략한 개요를 추가하고, 중복된 정보를 병합하고, 마크다운 형식을 통합하고, 모든 소스 인용을 참조 섹션에 구성합니다.

## 14.3 에이전트 시스템 설계

### 14.3.1 상담원 책임 부문

심층 연구 보조원에서 우리는 각각 특정 작업을 담당하는 세 가지 전문 에이전트를 설계했습니다. 이를 통해 각 에이전트를 간단하고 이해하기 쉽게 만들고 유지 관리할 수 있습니다.

7장에서는 `SimpleAgent`을 사용하여 에이전트를 구축하는 방법을 배웠습니다. `SimpleAgent`의 디자인 철학은 간단하고 직접적입니다. `run()` 메소드가 호출될 때마다 에이전트는 사용자의 질문을 분석하고 도구 호출 여부를 결정한 후 결과를 반환합니다. 이 디자인은 간단한 작업을 처리할 때 매우 효과적이지만 심층 연구와 같은 복잡한 작업에 직면할 때는 다중 에이전트 협업 접근 방식을 계속 사용해야 합니다.

표 14.1에 표시된 대로 세 명의 에이전트는 각각 계획, 요약 및 보고서 생성을 담당합니다.

<div align="center">
<p>표 14.1 세 대리인의 책임 분배</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-table-1.png" alt="" width="85%"/>
</div>

각 Agent의 디자인을 자세히 소개하겠습니다.

**에이전트 1: 연구 기획 전문가(TODO 플래너)**

**책임**: 연구 주제를 3~5개의 하위 작업으로 분해

**디자인 철학**: 연구 기획 전문가의 핵심 업무는 사용자의 연구 주제를 이해하고 주제의 핵심 측면을 분석한 후 일련의 하위 작업을 생성하는 것입니다. 이 과정은 인간 연구자들이 연구를 시작하기 전 진행하는 '브레인스토밍' 단계와 유사합니다.

**프롬프트 디자인**:

```python
todo_planner_instructions = """
You are a research planning expert. Your task is to decompose the user's research topic into 3-5 subtasks.

Current date: {current_date}

Research topic: {research_topic}

Please analyze this research topic and decompose it into 3-5 subtasks. Each subtask should:
1. Cover an important aspect of the topic
2. Have a clear research objective
3. Be able to find relevant materials through search engines

Please return the subtask list in JSON format, each subtask containing:
- title: Task title (concise and clear)
- intent: Task intent (why research this)
- query: Search query (query string for search engines, can use English for better search results)

Example output:
[
  {{
    "title": "What is a multimodal model",
    "intent": "Understand the basic concepts of multimodal models to lay the foundation for subsequent research",
    "query": "multimodal model definition concept 2024"
  }},
  ...
]

Please ensure:
1. Number of subtasks is between 3-5
2. Subtasks have logical relationships (e.g., from basics to applications, from current status to trends)
3. Search queries can accurately find relevant materials
4. Only return JSON, do not include other text
"""
```

**핵심 디자인 포인트**: 프롬프트에는 최신 정보를 얻기 위한 현재 날짜가 포함되고, 쉬운 구문 분석을 위해 명시적으로 JSON 형식 출력이 필요하며, 에이전트가 예제를 통해 예상 출력을 이해하는 데 도움이 되고, 하위 작업 수 및 논리적 관계와 같은 제약 조건을 강조합니다.

**구현 코드**:

여기서 ToolAwareSimpleAgent는 SimpleAgent의 확장입니다. 이에 대해 섹션 14.3.2에서 배울 수 있으므로 여기서 자세히 알아볼 필요는 없습니다.

```python
class PlanningService:
    def __init__(self, llm: HelloAgentsLLM):
        self._agent = ToolAwareSimpleAgent(
            name="TODO Planner",
            system_prompt="You are a research planning expert",
            llm=llm,
            tool_call_listener=self._on_tool_call
        )

    def plan_todo_list(self, state: SummaryState) -> List[TodoItem]:
        prompt = todo_planner_instructions.format(
            current_date=get_current_date(),
            research_topic=state.research_topic,
        )

        response = self._agent.run(prompt)
        tasks_payload = self._extract_tasks(response)

        todo_items = []
        for idx, item in enumerate(tasks_payload, start=1):
            task = TodoItem(
                id=idx,
                title=item["title"],
                intent=item["intent"],
                query=item["query"],
            )
            todo_items.append(task)

        return todo_items

    def _extract_tasks(self, response: str) -> List[dict]:
        """Extract JSON from Agent response"""
        # Use regex to extract JSON part
        json_match = re.search(r'\[.*\]', response, re.DOTALL)
        if json_match:
            json_str = json_match.group(0)
            return json.loads(json_str)
        else:
            raise ValueError("Unable to extract JSON from response")
```

**에이전트 2: 작업 요약 전문가(Task Summarizer)**

**책임**: 검색결과 요약, 핵심정보 추출

**디자인 철학**: 작업 요약 전문가의 핵심 업무는 검색 결과를 읽고 핵심 정보를 추출하여 구조적으로 제시하는 것입니다. 이 과정은 인간 연구자들이 문헌을 읽은 후 메모를 하는 것과 유사합니다.

**프롬프트 디자인**:

```python
task_summarizer_instructions = """
You are a task summarization expert. Your task is to summarize search results and extract key information.

Task title: {task_title}
Task intent: {task_intent}
Search query: {task_query}

Search results:
{search_results}

Please carefully read the above search results, extract key information, and return a summary in Markdown format.

The summary should include:
1. **Core Viewpoints**: Core viewpoints and conclusions from search results
2. **Key Data**: Important numbers, dates, names, etc.
3. **Source Citations**: Add source citations for each viewpoint (using [1], [2], etc.)

Please ensure:
1. Summary is concise and clear, avoiding redundancy
2. Retain important details and data
3. Add source citations for each viewpoint
4. Use Markdown format (headings, lists, bold, etc.)

Example output:
## Core Viewpoints

Multimodal models are AI models that can process multiple types of data[1]. Unlike traditional unimodal models, multimodal models can simultaneously understand text, images, audio, etc.[2].

**Key Features:**
- Cross-modal understanding[1]
- Unified representation[3]
- End-to-end training[2]

## Sources

[1] https://example.com/source1
[2] https://example.com/source2
[3] https://example.com/source3
"""
```

**주요 설계 포인트**: 프롬프트에는 에이전트가 작업을 이해하는 데 도움이 되는 작업 제목, 의도, 쿼리 및 기타 컨텍스트가 포함되어 있으며, 핵심 관점, 주요 데이터 및 출처 인용을 포함하는 출력이 명시적으로 요구되고, 각 관점에 대한 출처 인용 추가를 강조하며, 예제를 통해 에이전트가 예상되는 출력 형식을 이해하는 데 도움이 됩니다.

**구현 코드**:

```python
class SummarizationService:
    def __init__(self, llm: HelloAgentsLLM):
        self._agent = ToolAwareSimpleAgent(
            name="Task Summarizer",
            system_prompt="You are a task summarization expert",
            llm=llm,
            tool_call_listener=self._on_tool_call
        )

    def summarize_task(
        self,
        task: TodoItem,
        search_results: List[dict]
    ) -> str:
        # Format search results
        formatted_sources = self._format_sources(search_results)

        prompt = task_summarizer_instructions.format(
            task_title=task.title,
            task_intent=task.intent,
            task_query=task.query,
            search_results=formatted_sources,
        )

        summary = self._agent.run(prompt)
        return summary

    def _format_sources(self, search_results: List[dict]) -> str:
        """Format search results"""
        formatted = []
        for idx, result in enumerate(search_results, start=1):
            formatted.append(
                f"[{idx}] {result['title']}\n"
                f"URL: {result['url']}\n"
                f"Snippet: {result['snippet']}\n"
            )
        return "\n".join(formatted)
```

**에이전트 3: 보고서 작성 전문가(보고서 작성자)**

**책임**: 모든 하위 작업의 요약을 통합하고 최종 보고서를 생성합니다.

**디자인 철학**: 보고서 작성 전문가의 핵심 임무는 모든 하위 작업의 요약을 구조화된 보고서로 통합하는 것입니다. 이 과정은 인간 연구자가 모든 조사를 마친 후 연구 보고서를 작성하는 것과 유사합니다.

**프롬프트 디자인**:

```python
report_writer_instructions = """
You are a report writing expert. Your task is to integrate the summaries of all subtasks and generate a structured research report.

Research topic: {research_topic}

Subtask summaries:
{task_summaries}

Please integrate all the above subtask summaries and generate a structured research report.

The report should include:
1. **Title**: Research topic
2. **Overview**: Briefly introduce the research topic and report structure (2-3 paragraphs)
3. **Detailed Analysis of Each Subtask**: Organize in logical order (using level-2 headings)
4. **Summary**: Summarize the main findings of the research (1-2 paragraphs)
5. **References**: All source citations (grouped by subtask)

Please ensure:
1. Report structure is clear and logically coherent
2. Eliminate duplicate information
3. Retain all source citations
4. Use Markdown format

Example output:
# Latest Advances in Multimodal Large Models

## Overview

This report systematically researched the latest advances in multimodal large models...

## 1. What is a Multimodal Model

(Insert summary of subtask 1 here)

## 2. What are the Latest Multimodal Models

(Insert summary of subtask 2 here)

...

## Summary

Through this research, we learned about...

## References

### Task 1: What is a Multimodal Model
[1] https://example.com/source1
...
"""
```

**주요 디자인 포인트**: 프롬프트는 보고서에 제목, 개요, 세부 분석, 요약, 참조 및 기타 구조를 포함하도록 명시적으로 요구하고, 콘텐츠를 논리적 순서로 구성하는 것을 강조하고, 중복 정보를 병합하여 중복성을 제거하고, 모든 출처 인용을 유지하도록 요구합니다.

**구현 코드**:

```python
class ReportingService:
    def __init__(self, llm: HelloAgentsLLM):
        self._agent = ToolAwareSimpleAgent(
            name="Report Writer",
            system_prompt="You are a report writing expert",
            llm=llm,
            tool_call_listener=self._on_tool_call
        )

    def generate_report(
        self,
        research_topic: str,
        task_summaries: List[Tuple[TodoItem, str]]
    ) -> str:
        # Format subtask summaries
        formatted_summaries = self._format_summaries(task_summaries)

        prompt = report_writer_instructions.format(
            research_topic=research_topic,
            task_summaries=formatted_summaries,
        )

        report = self._agent.run(prompt)
        return report

    def _format_summaries(
        self,
        task_summaries: List[Tuple[TodoItem, str]]
    ) -> str:
        """Format subtask summaries"""
        formatted = []
        for idx, (task, summary) in enumerate(task_summaries, start=1):
            formatted.append(
                f"## Task {idx}: {task.title}\n"
                f"Intent: {task.intent}\n\n"
                f"{summary}\n"
            )
        return "\n".join(formatted)
```

### 14.3.2 ToolAwareSimpleAgent 설계

7장에서는 HelloAgents 프레임워크의 기본 Agent인 `SimpleAgent`를 구현했습니다. 하지만 심층 연구 보조원에는 **도구 호출을 녹음**할 수 있는 에이전트가 필요합니다. 이것이 `ToolAwareSimpleAgent`의 유래입니다.

심층 연구 보조원에서는 다음에 대한 각 에이전트의 도구 호출 상태를 기록해야 합니다.

1. **디버깅**: 에이전트가 호출한 도구와 전달된 매개변수를 확인합니다.
2. **로깅**: 연구 과정 중 모든 작업을 기록합니다.
3. **분석**: 에이전트의 행동 패턴을 분석합니다.
4. **진행상황 표시**: 에이전트가 수행 중인 작업을 실시간으로 표시합니다.

`SimpleAgent` 자체는 도구 호출 듣기를 지원하지 않으므로 확장이 필요합니다.

`ToolAwareSimpleAgent`는 `SimpleAgent` 위에 `tool_call_listener` 매개변수를 추가합니다. 도구가 호출될 때마다 호출되는 콜백 함수입니다.

**사용 예:**

```python
from hello_agents import ToolAwareSimpleAgent

def tool_listener(call_info):
    print(f"Agent: {call_info['agent_name']}")
    print(f"Tool: {call_info['tool_name']}")
    print(f"Parameters: {call_info['parsed_parameters']}")
    print(f"Result: {call_info['result']}")

agent = ToolAwareSimpleAgent(
    name="Research Assistant",
    system_prompt="You are a research assistant",
    llm=llm,
    tool_call_listener=tool_listener
)
```

`ToolAwareSimpleAgent`은 `SimpleAgent`을 상속하고 `_execute_tool_call` 메서드를 재정의합니다.

```python
class ToolAwareSimpleAgent(SimpleAgent):
    def __init__(
        self,
        name: str,
        system_prompt: str,
        llm: HelloAgentsLLM,
        tool_registry: Optional[ToolRegistry] = None,
        tool_call_listener: Optional[Callable] = None,
    ):
        super().__init__(
            name=name,
            system_prompt=system_prompt,
            llm=llm,
            tool_registry=tool_registry,
        )
        self._tool_call_listener = tool_call_listener

    def _execute_tool_call(self, tool_name: str, parameters: str) -> str:
        """Execute tool call and notify listener"""
        # Parse parameters
        parsed_parameters = self._parse_parameters(parameters)

        # Call tool
        result = super()._execute_tool_call(tool_name, parameters)

        # Notify listener
        if self._tool_call_listener:
            self._tool_call_listener({
                "agent_name": self.name,
                "tool_name": tool_name,
                "parsed_parameters": parsed_parameters,
                "result": result,
            })

        return result
```

심층 연구 보조원에서는 `ToolAwareSimpleAgent`를 사용하여 모든 에이전트 도구 호출을 녹음합니다.

```python
class DeepResearchAgent:
    def __init__(self, config: Configuration):
        self.config = config
        self.llm = HelloAgentsLLM(...)

        # Create tool call listener
        def tool_listener(call_info):
            self._emit_event({
                "type": "tool_call",
                "agent": call_info["agent_name"],
                "tool": call_info["tool_name"],
                "parameters": call_info["parsed_parameters"],
            })

        # Create three Agents, all using the same listener
        self.planner = PlanningService(self.llm, tool_listener)
        self.summarizer = SummarizationService(self.llm, tool_listener)
        self.reporter = ReportingService(self.llm, tool_listener)
```

이렇게 하면 모든 Agent 도구 통화가 녹음되고 SSE를 통해 프런트엔드로 푸시되어 사용자에게 실시간으로 표시됩니다.

### 14.3.3 에이전트 협업 모드

그림 14.6에 표시된 것처럼 세 에이전트는 **순차적 협력** 관계를 갖습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-6.png" alt="" width="85%"/>
<p>그림 14.6 에이전트 협업 프로세스</p>
</div>

순차 협업 모드의 특징은 다음과 같습니다.

1. **선형 프로세스**: 에이전트는 고정된 순서로 실행됩니다.
2. **입력 및 출력 지우기**: 각 에이전트의 입력은 이전 에이전트의 출력에서 나옵니다.
3. **동시성 없음**: 동시에 하나의 에이전트만 작업합니다.

`DeepResearchAgent`은 전체 시스템의 핵심 조정자로서 세 에이전트의 일정을 관리하는 역할을 담당합니다.

```python
class DeepResearchAgent:
    def run(self, research_topic: str) -> str:
        # 1. Planning stage
        self._emit_event({"type": "status", "message": "Planning research tasks..."})
        todo_list = self.planner.plan_todo_list(research_topic)
        self._emit_event({"type": "tasks", "tasks": todo_list})

        # 2. Execution stage
        task_summaries = []
        for task in todo_list:
            self._emit_event({
                "type": "status",
                "message": f"Researching: {task.title}"
            })

            # Search
            search_results = self.search_service.search(task.query)

            # Summarize
            summary = self.summarizer.summarize_task(task, search_results)
            task_summaries.append((task, summary))

            self._emit_event({
                "type": "task_completed",
                "task_id": task.id
            })

        # 3. Reporting stage
        self._emit_event({"type": "status", "message": "Generating report..."})
        report = self.reporter.generate_report(research_topic, task_summaries)
        self._emit_event({"type": "report", "content": report})

        return report
```

## 14.4 도구 시스템 통합

### 14.4.1 검색 도구 확장

7장에서는 Tavily와 SerpApi 검색 엔진을 통합하여 다중 소스 검색의 설계 아이디어를 보여주는 `SearchTool`의 기본 버전을 구현했습니다. 이 장의 심층 연구 보조원에서는 DuckDuckGo, Perplexity, SearXNG 및 기타 검색 엔진을 추가하고 고급 모드(여러 검색 엔진 결합)를 구현하여 `SearchTool`의 기능을 더욱 확장했습니다. 검색은 심층 연구 보조원의 가장 핵심 기능이며, 이러한 확장을 통해 시스템은 다양한 사용 시나리오와 요구 사항에 적응할 수 있습니다.

표 14.2에서 볼 수 있듯이 이번에 추가된 검색엔진들은 서로 다른 특성과 적용 가능한 시나리오를 가지고 있다.

<div align="center">
<p>표 14.2 다중 검색 엔진 비교</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-table-2.png" alt="" width="85%"/>
</div>

별도로 연장하는 방법에 대해서는 더 이상 논의하지 않겠습니다. 구현 방법은 7장의 소스 코드와 확장 사례를 참조할 수 있습니다. `SearchTool`은 통합 검색 인터페이스를 제공합니다. 어떤 검색 엔진을 사용하든 호출 방법은 동일합니다.

심층 연구 보조원에서는 구성 파일을 통해 검색 엔진을 선택합니다.

```python
# config.py
class SearchAPI(str, Enum):
    TAVILY = "tavily"
    DUCKDUCKGO = "duckduckgo"
    PERPLEXITY = "perplexity"
    SEARXNG = "searxng"
    ADVANCED = "advanced"

class Configuration(BaseModel):
    search_api: SearchAPI = SearchAPI.DUCKDUCKGO
    # ...
```

```python
# .env
SEARCH_API=tavily
```

이렇게 하면 사용자는 코드 수정 없이 `.env` 파일만 수정하여 검색 엔진을 선택할 수 있습니다.

`SearchTool`에서 반환된 결과는 다음을 포함하는 사전입니다.

- `results`: 검색 결과 목록, 각 결과에는 제목, URL, 스니펫이 포함됩니다.
- `backend`: 사용된 검색 엔진
- `answer`: AI가 생성한 답변(Perplexity에만 해당)
- `notices`: 알림 정보 (such as API limits, errors, etc.)

다음은 몇 가지 특별한 경우를 처리하는 것입니다.

검색결과에 중복된 URL이 포함될 수 있으므로 중복을 제거해야 합니다.

```python
def deduplicate_sources(sources: List[dict]) -> List[dict]:
    """Remove duplicate URLs"""
    seen_urls = set()
    unique_sources = []

    for source in sources:
        if source["url"] not in seen_urls:
            seen_urls.add(source["url"])
            unique_sources.append(source)

    return unique_sources
```

검색 결과에는 많은 양의 텍스트가 포함될 수 있으므로 각 소스에 대한 토큰 수를 제한해야 합니다.

```python
def limit_source_tokens(source: dict, max_tokens: int = 2000) -> dict:
    """Limit the number of tokens for a source"""
    snippet = source["snippet"]

    # Simple token estimation: 1 token is approximately 4 characters
    max_chars = max_tokens * 4

    if len(snippet) > max_chars:
        snippet = snippet[:max_chars] + "..."

    return {
        **source,
        "snippet": snippet
    }
```

### 14.4.2 NoteTool 사용법

심층 연구 조교에서는 `NoteTool`를 사용하여 연구 진행을 지속합니다. `NoteTool`은 9장에 통합된 내장 도구로, 노트를 생성하고, 읽고, 업데이트하고, 삭제하는 데 사용됩니다.

연구 과정에서 각 하위 작업에 대한 검색 결과, 요약, 최종 연구 보고서를 기록해야 합니다. 이 정보는 연구가 중단되었을 때 마지막 진행부터 계속될 수 있도록 디스크에 저장되어야 하며, 연구 과정에서 모든 작업을 확인하고 연구의 품질과 효율성을 분석하는 것도 편리합니다.

`NoteTool`는 지정된 작업 공간 디렉터리에 메모를 저장하며 각 메모는 Markdown 파일입니다. 노트 파일명은 작업 ID이며 내용에는 작업 제목, 작업 의도, 검색어, 검색 결과, 요약이 포함됩니다.

최종 생성된 파일 스타일은 다음 트리 구조입니다.

```
workspace/
├── notes/
│   ├── 1.md  # Notes for task 1
│   ├── 2.md  # Notes for task 2
│   ├── 3.md  # Notes for task 3
│   └── ...
└── reports/
    └── final_report.md  # Final report
```

심층 연구 보조원에서는 `NoteTool`를 사용하여 각 하위 작업의 연구 진행 상황을 기록합니다.

```python
class NotesService:
    def __init__(self, workspace: str):
        self.note_tool = NoteTool(workspace=workspace)

    def save_task_summary(
        self,
        task: TodoItem,
        search_results: List[dict],
        summary: str
    ):
        """Save task summary"""
        # Format note content
        content = self._format_note_content(
            task=task,
            search_results=search_results,
            summary=summary
        )

        # Create note
        self.note_tool.run({
            "action": "create",
            "title": f"Task {task.id}: {task.title}",
            "content": content,
            "tags": ["research", "summary"]
        })

    def _format_note_content(
        self,
        task: TodoItem,
        search_results: List[dict],
        summary: str
    ) -> str:
        """Format note content"""
        content = f"# Task {task.id}: {task.title}\n\n"
        content += f"## Task Information\n\n"
        content += f"- **Intent**: {task.intent}\n"
        content += f"- **Query**: {task.query}\n\n"

        content += f"## Search Results\n\n"
        for idx, result in enumerate(search_results, start=1):
            content += f"[{idx}] {result['title']}\n"
            content += f"URL: {result['url']}\n"
            content += f"Snippet: {result['snippet']}\n\n"

        content += f"## Summary\n\n{summary}\n"

        return content
```

### 14.4.3 ToolRegistry 도구 관리

`ToolRegistry`은 HelloAgents 프레임워크의 도구 레지스트리로, 7장에서도 지원되며 모든 도구의 등록 및 호출을 관리하는 데 사용됩니다. 심층 연구 보조원에서는 `ToolRegistry`을 사용하여 `SearchTool` 및 `NoteTool`을 관리합니다.

에이전트를 생성하기 전에 먼저 도구를 등록해야 합니다.

```python
from hello_agents import ToolAwareSimpleAgent
from hello_agents.tools import ToolRegistry
from hello_agents.tools import SearchTool
from hello_agents.tools import NoteTool

# Create tools
search_tool = SearchTool(backend="hybrid")
note_tool = NoteTool(workspace="./workspace/notes")

# Create registry
registry = ToolRegistry()

# Register tools
registry.register_tool(search_tool)
registry.register_tool(note_tool)

# Create Agent
agent = ToolAwareSimpleAgent(
    name="Research Assistant",
    system_prompt="You are a research assistant",
    llm=llm,
    tool_registry=registry
)
```

에이전트가 도구를 호출해야 하는 경우 그림 14.7에 표시된 대로 도구 호출 명령을 생성합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-7.png" alt="" width="85%"/>
<p>그림 14.7 도구 호출 프로세스</p>
</div>

**도구 호출 프로세스**:

1. **에이전트 생성 명령**: 에이전트가 `[TOOL_CALL:search_tool:{"input": "Datawhale organization", "backend": "tavily"}]`과 같은 도구 호출 명령을 생성합니다.
2. **명령 구문 분석**: `ToolRegistry` 명령을 구문 분석하고 도구 이름과 매개변수를 추출합니다.
3. **도구 찾기**: `ToolRegistry` 도구 이름을 기준으로 해당 도구를 찾습니다.
4. **도구 호출**: 도구의 `run` 메소드를 호출하여 매개변수를 전달합니다.
5. **결과 반환**: 도구가 실행 결과를 반환합니다.
6. **결과 형식화**: 결과를 문자열로 형식화하여 에이전트에 반환합니다.

## 14.5 서비스 계층 구현

이 섹션에서는 PlanningService, SummarizationService, ReportingService 및 SearchService를 포함한 핵심 서비스 구현을 자세히 소개합니다. 이러한 서비스는 특정 비즈니스 로직을 담당하는 에이전트와 도구를 연결하는 브리지입니다.

### 14.5.1 작업 계획 서비스

`PlanningService`은 연구 기획 에이전트를 호출하여 연구 주제를 하위 작업으로 분해하는 역할을 담당합니다. 이는 전체 연구 과정의 첫 번째이자 가장 중요한 단계입니다.

**(1) 구현 접근 방식**

핵심 책임은 다음과 같습니다.

1. **계획 프롬프트 작성**: 연구 주제 및 현재 날짜를 기반으로 프롬프트 작성
2. **통화 계획 에이전트**: TODO Planner 에이전트를 호출하여 하위 작업 목록을 생성합니다.
3. **JSON 응답 구문 분석**: 에이전트 응답에서 JSON 형식의 하위 작업 목록을 추출합니다.
4. **하위 작업 형식 확인**: 각 하위 작업에 필수 필드(제목, 의도, 쿼리)가 포함되어 있는지 확인하세요.

```python
import re
import json
from typing import List, Callable, Optional
from datetime import datetime

from hello_agents import HelloAgentsLLM
from hello_agents import ToolAwareSimpleAgent
from models import TodoItem, SummaryState
from prompts import todo_planner_instructions

class PlanningService:
    """Task planning service"""

    def __init__(
        self,
        llm: HelloAgentsLLM,
        tool_call_listener: Optional[Callable] = None
    ):
        self._llm = llm
        self._tool_call_listener = tool_call_listener

        # Create planning Agent
        self._agent = ToolAwareSimpleAgent(
            name="TODO Planner",
            system_prompt="You are a research planning expert, skilled at decomposing complex research topics into clear subtasks.",
            llm=llm,
            tool_call_listener=tool_call_listener
        )

    def plan_todo_list(self, state: SummaryState) -> List[TodoItem]:
        """Plan TODO list

        Args:
            state: Research state, containing research topic

        Returns:
            Subtask list
        """
        # Build Prompt
        prompt = todo_planner_instructions.format(
            current_date=self._get_current_date(),
            research_topic=state.research_topic,
        )

        # Call Agent
        response = self._agent.run(prompt)

        # Parse JSON
        tasks_payload = self._extract_tasks(response)

        # Validate and create TodoItem
        todo_items = []
        for idx, item in enumerate(tasks_payload, start=1):
            # Validate required fields
            if not all(key in item for key in ["title", "intent", "query"]):
                raise ValueError(f"Task {idx} is missing required fields")

            task = TodoItem(
                id=idx,
                title=item["title"],
                intent=item["intent"],
                query=item["query"],
            )
            todo_items.append(task)

        return todo_items

    def _get_current_date(self) -> str:
        """Get current date"""
        return datetime.now().strftime("%Y-%m-%d")

    def _extract_tasks(self, response: str) -> List[dict]:
        """Extract JSON from Agent response

        The Agent's response may contain extra text, such as:
        "Okay, I will plan the following tasks for you:\n[{...}, {...}]\nThese tasks cover..."

        We need to extract the JSON part.
        """
        # Method 1: Use regex to extract JSON array
        json_match = re.search(r'\[.*\]', response, re.DOTALL)
        if json_match:
            json_str = json_match.group(0)
            try:
                return json.loads(json_str)
            except json.JSONDecodeError as e:
                raise ValueError(f"JSON parsing failed: {e}")

        # Method 2: If no JSON array is found, try to parse the entire response directly
        try:
            return json.loads(response)
        except json.JSONDecodeError:
            raise ValueError("Unable to extract JSON from response")
```

**(2) JSON 구문 분석 및 검증**

에이전트가 반환한 JSON에는 추가 텍스트 또는 형식 오류가 포함될 수 있으므로 강력한 구문 분석 논리가 필요합니다.

**일반적인 문제**:

1. **추가 텍스트 포함**: 에이전트는 JSON 앞뒤에 설명 텍스트를 추가할 수 있습니다.
2. **형식 오류**: JSON에 따옴표, 쉼표 등이 누락될 수 있습니다.
3. **누락된 필드**: 일부 하위 작업에 필수 필드가 누락되었을 수 있습니다.

**해결책**:

1. **정규식 사용**: JSON 부분 추출
2. **다중 구문 분석 전략**: 먼저 JSON 배열을 추출한 다음 직접 구문 분석을 시도합니다.
3. **필드 유효성 검사**: 각 하위 작업에 필수 필드가 포함되어 있는지 확인하세요.

**예**:

```python
# Agent response example 1: Contains extra text
response1 = """
Okay, I will plan the following tasks for you:

[
  {
    "title": "What is a multimodal model",
    "intent": "Understand basic concepts",
    "query": "multimodal model definition"
  },
  {
    "title": "Latest multimodal models",
    "intent": "Understand technical status",
    "query": "latest multimodal models 2024"
  }
]

These tasks cover the basic information and core projects of the Datawhale organization.
"""

# Extract JSON
tasks1 = service._extract_tasks(response1)
# Result: [{"title": "Basic information about Datawhale", ...}, ...]

# Agent response example 2: Pure JSON
response2 = """
[
  {"title": "Basic information about Datawhale", "intent": "Understand organizational positioning", "query": "Datawhale organization introduction"},
  {"title": "Main projects of Datawhale", "intent": "Understand core content", "query": "Datawhale projects tutorials 2024"}
]
"""

# Extract JSON
tasks2 = service._extract_tasks(response2)
# Result: [{"title": "What is a multimodal model", ...}, ...]
```

**(3) 계획 품질 평가**

좋은 계획은 다음 기준을 충족해야 합니다.

1. **포괄적인 범위**: 주제의 모든 중요한 측면을 다룹니다.
2. **명확한 논리**: 하위 작업 간의 명확한 논리적 관계
3. **정확한 검색어**: 검색어로 관련 자료를 정확하게 찾을 수 있습니다.
4. **적절한 수량**: 3~5개의 하위 작업

평가 방법을 추가할 수 있습니다:

```python
def evaluate_plan(self, todo_items: List[TodoItem]) -> dict:
    """Evaluate planning quality

    Returns:
        Evaluation results, including score and suggestions
    """
    score = 100
    suggestions = []

    # Check quantity
    if len(todo_items) < 3:
        score -= 20
        suggestions.append("Too few subtasks, may miss important information")
    elif len(todo_items) > 5:
        score -= 10
        suggestions.append("Too many subtasks, may have redundancy")

    # Check query quality
    for task in todo_items:
        if len(task.query.split()) < 2:
            score -= 10
            suggestions.append(f"Query for task '{task.title}' is too simple")

    # Check logical relationships
    # (More complex logic checks can be added here)

    return {
        "score": score,
        "suggestions": suggestions
    }
```

### 14.5.2 요약 서비스

`SummarizationService`은 작업 요약 에이전트를 호출하여 검색 결과를 요약하는 역할을 담당합니다. 이는 연구 과정의 핵심 연결고리이며 연구의 질을 결정합니다.

그 책임은 다음과 같습니다:

1. **검색 결과 형식 지정**: 검색 결과 형식을 읽을 수 있는 텍스트로 지정
2. **요약 프롬프트 작성**: 작업 정보 및 검색 결과를 기반으로 프롬프트 작성
3. **요약 에이전트 호출**: 작업 요약 에이전트를 호출하여 요약을 생성합니다.
4. **소스 인용 추출**: 요약에서 소스 인용을 추출합니다.

핵심 코드:

```python
from typing import List, Callable, Optional, Tuple

from hello_agents import HelloAgentsLLM
from hello_agents import ToolAwareSimpleAgent
from models import TodoItem
from prompts import task_summarizer_instructions

class SummarizationService:
    """Summarization service"""

    def __init__(
        self,
        llm: HelloAgentsLLM,
        tool_call_listener: Optional[Callable] = None
    ):
        self._llm = llm
        self._tool_call_listener = tool_call_listener

        # Create summarization Agent
        self._agent = ToolAwareSimpleAgent(
            name="Task Summarizer",
            system_prompt="You are a task summarization expert, skilled at extracting key information from search results.",
            llm=llm,
            tool_call_listener=tool_call_listener
        )

    def summarize_task(
        self,
        task: TodoItem,
        search_results: List[dict]
    ) -> Tuple[str, List[str]]:
        """Summarize task

        Args:
            task: Task information
            search_results: Search results list

        Returns:
            (Summary text, source URL list)
        """
        # Format search results
        formatted_sources = self._format_sources(search_results)

        # Build Prompt
        prompt = task_summarizer_instructions.format(
            task_title=task.title,
            task_intent=task.intent,
            task_query=task.query,
            search_results=formatted_sources,
        )

        # Call Agent
        summary = self._agent.run(prompt)

        # Extract source URLs
        source_urls = [result["url"] for result in search_results]

        return summary, source_urls

    def _format_sources(self, search_results: List[dict]) -> str:
        """Format search results

        Format search results into readable text, including:
        - Serial number
        - Title
        - URL
        - Snippet
        """
        formatted = []
        for idx, result in enumerate(search_results, start=1):
            formatted.append(
                f"[{idx}] {result['title']}\n"
                f"URL: {result['url']}\n"
                f"Snippet: {result['snippet']}\n"
            )
        return "\n".join(formatted)
```

### 보고서 구조 디자인

최종 보고서에는 다음 부분이 포함되어야 합니다.

## 참고자료

### 작업 1: 다중 모드 모델이란 무엇입니까?
- https://example.com/multimodal-model-definition
...

### 작업 2: 최신 다중 모드 모델이란 무엇입니까?
- https://example.com/gpt4v
...
...

### 14.5.3 보고서 생성 서비스

`ReportingService`은 보고서 생성 에이전트를 호출하여 모든 하위 작업의 요약을 통합하는 역할을 담당합니다. 이는 최종 연구 보고서를 생성하는 연구 프로세스의 마지막 단계입니다.

그 책임은 다음과 같습니다:

1. **하위 작업 요약 형식 지정**: 모든 하위 작업 요약의 형식을 통일된 형식으로 지정합니다.
2. **보고서 프롬프트 작성**: 연구 주제 및 하위 작업 요약을 기반으로 프롬프트 작성
3. **보고서 에이전트 호출**: 보고서 작성자 에이전트를 호출하여 최종 보고서를 생성합니다.
4. **인용 정리**: 모든 출처 인용을 참고 문헌 섹션으로 정리합니다.

**핵심 코드 구현**:

```python
from typing import List, Callable, Optional, Tuple

from hello_agents import HelloAgentsLLM
from hello_agents import ToolAwareSimpleAgent
from models import TodoItem
from prompts import report_writer_instructions

class ReportingService:
    """Report generation service"""

    def __init__(
        self,
        llm: HelloAgentsLLM,
        tool_call_listener: Optional[Callable] = None
    ):
        self._llm = llm
        self._tool_call_listener = tool_call_listener

        # Create report Agent
        self._agent = ToolAwareSimpleAgent(
            name="Report Writer",
            system_prompt="You are a report writing expert, skilled at integrating information and generating structured reports.",
            llm=llm,
            tool_call_listener=tool_call_listener
        )

    def generate_report(
        self,
        research_topic: str,
        task_summaries: List[Tuple[TodoItem, str, List[str]]]
    ) -> str:
        """Generate final report

        Args:
            research_topic: Research topic
            task_summaries: Subtask summary list, each element is (task, summary, source URL list)

        Returns:
            Final report (Markdown format)
        """
        # Format subtask summaries
        formatted_summaries = self._format_summaries(task_summaries)

        # Build Prompt
        prompt = report_writer_instructions.format(
            research_topic=research_topic,
            task_summaries=formatted_summaries,
        )

        # Call Agent
        report = self._agent.run(prompt)

        return report

    def _format_summaries(
        self,
        task_summaries: List[Tuple[TodoItem, str, List[str]]]
    ) -> str:
        """Format subtask summaries

        Format all subtask summaries into a unified format, including:
        - Task serial number
        - Task title
        - Task intent
        - Summary content
        - Source URLs
        """
        formatted = []
        for idx, (task, summary, source_urls) in enumerate(task_summaries, start=1):
            formatted.append(
                f"## Task {idx}: {task.title}\n\n"
                f"**Intent**: {task.intent}\n\n"
                f"{summary}\n\n"
                f"**Sources**:\n"
            )
            for url in source_urls:
                formatted.append(f"- {url}\n")
            formatted.append("\n")

        return "".join(formatted)
```

### 14.5.4 검색 스케줄링 서비스

`SearchService`은 검색 엔진 예약, 검색 실행 및 결과 반환을 담당합니다. Agent와 SearchTool을 연결하는 브릿지입니다. 여기서는 SimpleAgent가 직접 도구를 호출하는 일반적인 형태를 채택하지 않고 대신 SearchTool의 실행 결과를 중간 계층을 통해 Agent에 반환하므로 Agent가 획득한 정보를 처리하는 데 더 집중할 수 있습니다.

그 책임은 다음과 같습니다:

1. **검색 엔진 예약**: 구성에 따라 검색 엔진을 선택합니다.
2. **검색 실행**: SearchTool을 호출하여 검색을 실행합니다.
3. **처리 결과**: 중복 제거, 토큰 제한, 포맷
4. **오류 처리**: 검색 실패 상황 처리

핵심 코드:

```python
from typing import List, Optional
import logging

from hello_agents.tools import SearchTool
from config import Configuration

logger = logging.getLogger(__name__)

class SearchService:
    """Search scheduling service"""

    def __init__(self, config: Configuration):
        self.config = config

        # Create SearchTool
        self.search_tool = SearchTool(backend="hybrid")

    def search(
        self,
        query: str,
        max_results: int = 5
    ) -> List[dict]:
        """Execute search

        Args:
            query: Search query
            max_results: Maximum number of results

        Returns:
            Search results list
        """
        try:
            # Call SearchTool
            raw_response = self.search_tool.run({
                "input": query,
                "backend": self.config.search_api.value,
                "mode": "structured",
                "max_results": max_results
            })

            # Extract results
            results = raw_response.get("results", [])

            # Process results
            results = self._deduplicate_sources(results)
            results = self._limit_source_tokens(results)

            logger.info(f"Search successful: {query}, returned {len(results)} results")

            return results

        except Exception as e:
            logger.error(f"Search failed: {query}, error: {e}")
            return []

    def _deduplicate_sources(self, sources: List[dict]) -> List[dict]:
        """Remove duplicate URLs"""
        seen_urls = set()
        unique_sources = []

        for source in sources:
            url = source.get("url", "")
            if url and url not in seen_urls:
                seen_urls.add(url)
                unique_sources.append(source)

        return unique_sources

    def _limit_source_tokens(
        self,
        sources: List[dict],
        max_tokens_per_source: int = 2000
    ) -> List[dict]:
        """Limit the number of tokens per source"""
        limited_sources = []

        for source in sources:
            snippet = source.get("snippet", "")

            # Simple token estimation: 1 token is approximately 4 characters
            max_chars = max_tokens_per_source * 4

            if len(snippet) > max_chars:
                snippet = snippet[:max_chars] + "..."

            limited_sources.append({
                **source,
                "snippet": snippet
            })

        return limited_sources
```

그림 14.8과 같이 구성에 따라 검색 엔진을 선택합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-8.png" alt="" width="85%"/>
<p>그림 14.8 검색 엔진 예약 프로세스</p>
</div>

**일정 논리**:

1. **구성 읽기**: `.env` 파일에서 `SEARCH_API` 구성을 읽습니다.
2. **엔진 선택**: 구성에 따라 검색 엔진을 선택합니다 (tavily, duckduckgo, perplexity, etc.)
3. **검색 실행**: SearchTool을 호출하여 검색을 실행합니다.
4. **처리 결과**: 중복 제거, 토큰 제한, 포맷
5. **결과 반환**: 처리된 검색 결과를 반환합니다.

효율성을 높이고 비용을 절감하기 위해 검색 결과 캐싱을 추가할 수 있습니다.

```python
import hashlib
import json
from pathlib import Path

class SearchService:
    def __init__(self, config: Configuration):
        self.config = config
        self.search_tool = SearchTool(backend="hybrid")

        # Cache directory
        self.cache_dir = Path("./cache/search")
        self.cache_dir.mkdir(parents=True, exist_ok=True)

    def search(
        self,
        query: str,
        max_results: int = 5,
        use_cache: bool = True
    ) -> List[dict]:
        """Execute search (with cache)"""
        # Generate cache key
        cache_key = self._generate_cache_key(query, max_results)
        cache_file = self.cache_dir / f"{cache_key}.json"

        # Try to read from cache
        if use_cache and cache_file.exists():
            logger.info(f"Reading search results from cache: {query}")
            with open(cache_file, "r", encoding="utf-8") as f:
                return json.load(f)

        # Execute search
        results = self._execute_search(query, max_results)

        # Save to cache
        if use_cache and results:
            with open(cache_file, "w", encoding="utf-8") as f:
                json.dump(results, f, ensure_ascii=False, indent=2)

        return results

    def _generate_cache_key(self, query: str, max_results: int) -> str:
        """Generate cache key"""
        # Generate MD5 hash using query and max results
        content = f"{query}_{max_results}_{self.config.search_api.value}"
        return hashlib.md5(content.encode()).hexdigest()
```

4가지 핵심 서비스(PlanningService, SummarizationService, ReportingService, SearchService)를 통해 완전한 연구 프로세스를 구축했습니다. 이러한 서비스는 각각 자신의 임무를 수행하고 명확한 인터페이스를 통해 협업하여 연구 주제부터 최종 보고서까지 자동화된 프로세스를 달성합니다.

## 14.6 프론트엔드 상호작용 디자인

이전 섹션에서는 완전한 백엔드 시스템을 구현했습니다. 이 섹션에서는 전체 화면 모달 대화 상자 UI, 실시간 진행률 표시 및 연구 결과 시각화를 포함한 프런트 엔드 상호 작용 디자인을 자세히 소개합니다.

### 14.6.1 전체 화면 모달 대화 상자 UI 디자인

심층 연구 보조원은 다음과 같은 장점이 있는 전체 화면 모달 대화 상자 UI 디자인을 채택합니다.

1. **몰입형 경험**: 방해 요소를 피하고 연구에 집중하는 전체 화면 디스플레이
2. **명확한 계층 구조**: 메인 페이지와 검색 페이지가 분리되어 있으며 명확한 계층 구조가 있습니다.
3. **쉬운 닫기**: 닫기 버튼을 클릭하거나 ESC 키를 눌러 메인 페이지로 돌아갑니다.
4. **반응형 디자인**: 다양한 화면 크기에 적응

그림 14.9에 표시된 것처럼 전체 화면 모달 대화 상자에는 다음 부분이 포함되어 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-9.png" alt="" width="85%"/>
<p>그림 14.9 전체 화면 모달 대화 상자 UI</p>
</div>

**UI 구성요소**:

1. **상단 표시줄**: 연구 주제와 닫기 버튼이 포함되어 있습니다.
2. **진행 영역**: 현재 연구 진행 상황(기획, 실행, 보고)을 표시합니다.
3. **콘텐츠 영역**: 연구 결과를 표시합니다(Markdown 형식).
4. **하단바**: 상태 정보 표시 (such as "Researching...", "Completed")

해당 Vue 구현은 다음과 같습니다 (ResearchModal.vue):

```vue
<template>
  <div v-if="isOpen" class="modal-overlay" @click.self="close">
    <div class="modal-container">
      <!-- Top bar -->
      <div class="modal-header">
        <h2>{{ researchTopic }}</h2>
        <button @click="close" class="close-button">
          <svg><!-- Close icon --></svg>
        </button>
      </div>

      <!-- Progress area -->
      <div class="progress-section">
        <div class="progress-bar">
          <div
            class="progress-fill"
            :style="{ width: progressPercentage + '%' }"
          ></div>
        </div>
        <div class="progress-text">{{ progressText }}</div>
      </div>

      <!-- Content area -->
      <div class="content-section">
        <div v-if="isLoading" class="loading-spinner">
          <div class="spinner"></div>
          <p>Researching, please wait...</p>
        </div>

        <div v-else class="markdown-content" v-html="renderedMarkdown"></div>
      </div>

      <!-- Bottom bar -->
      <div class="modal-footer">
        <span class="status-text">{{ statusText }}</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue'
import { marked } from 'marked'

interface Props {
  isOpen: boolean
  researchTopic: string
}

const props = defineProps<Props>()
const emit = defineEmits<{
  close: []
}>()

// State
const isLoading = ref(true)
const progressPercentage = ref(0)
const progressText = ref('Preparing...')
const statusText = ref('Researching...')
const markdownContent = ref('')

// Render Markdown
const renderedMarkdown = computed(() => {
  return marked(markdownContent.value)
})

// Close modal
const close = () => {
  emit('close')
}

// Listen for ESC key
const handleKeydown = (e: KeyboardEvent) => {
  if (e.key === 'Escape') {
    close()
  }
}

// Add keyboard listener on mount
watch(() => props.isOpen, (isOpen) => {
  if (isOpen) {
    document.addEventListener('keydown', handleKeydown)
  } else {
    document.removeEventListener('keydown', handleKeydown)
  }
})
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}
...
</style>
```

다양한 화면 크기에 적응하기 위해 미디어 쿼리를 추가합니다.

```css
/* Tablet devices */
@media (max-width: 768px) {
  .modal-container {
    width: 95vw;
    height: 95vh;
  }

  .modal-header,
  .progress-section,
  .content-section,
  .modal-footer {
    padding: 15px 20px;
  }
}

/* Mobile devices */
@media (max-width: 480px) {
  .modal-container {
    width: 100vw;
    height: 100vh;
    border-radius: 0;
  }

  .modal-header h2 {
    font-size: 18px;
  }
}
```

### 14.6.2 실시간 진행 표시

심층 연구 보조원은 SSE를 사용하여 실시간 진행 상황 표시를 구현합니다. SSE는 서버가 클라이언트에 데이터를 적극적으로 보낼 수 있도록 하는 서버 푸시 기술이며, 이에 대해서는 프로토콜 장에서도 설명합니다.

그림 14.10에 표시된 것처럼 SSE 프로세스에는 다음 단계가 포함됩니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-10.png" alt="" width="85%"/>
<p>그림 14.10 SSE 프로세스</p>
</div>

**프로세스 설명**:

1. **클라이언트가 요청 시작**: 연구 주제가 포함된 POST 요청을 `/api/research`로 보냅니다.
2. **서버가 SSE 연결을 설정**: `text/event-stream` 응답 반환
3. **서버 푸시 진행** : 연구 진행(기획, 실행, 보고)을 주기적으로 푸시합니다.
4. **클라이언트가 진행 상황 수신**: SSE 이벤트 수신, UI 업데이트
5. **연구 완료**: 서버가 최종 보고서를 푸시하고 연결을 종료합니다.

프런트엔드 및 백엔드 프로젝트에서 SSE를 사용하려면 다음 구성도 수행해야 합니다.

**백엔드 FastAPI SSE 엔드포인트**:

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
from typing import AsyncGenerator
import asyncio
import json

app = FastAPI()

async def research_stream(topic: str) -> AsyncGenerator[str, None]:
    """Research streaming generator

    Generate SSE format data:
    data: {"type": "progress", "data": {...}}

    """
    try:
        # 1. Planning stage
        yield f"data: {json.dumps({'type': 'progress', 'stage': 'planning', 'percentage': 10, 'text': 'Planning research tasks...'})}\n\n"

        # Call PlanningService
        todo_items = await planning_service.plan_todo_list(topic)

        yield f"data: {json.dumps({'type': 'plan', 'data': [item.dict() for item in todo_items]})}\n\n"

        # 2. Execution stage
        task_summaries = []
        for idx, task in enumerate(todo_items, start=1):
            # Update progress
            percentage = 10 + (idx / len(todo_items)) * 70
            yield f"data: {json.dumps({'type': 'progress', 'stage': 'executing', 'percentage': percentage, 'text': f'Researching task {idx}/{len(todo_items)}: {task.title}'})}\n\n"

            # Search
            search_results = await search_service.search(task.query)

            # Summarize
            summary, source_urls = await summarization_service.summarize_task(task, search_results)

            task_summaries.append((task, summary, source_urls))

            # Push task summary
            yield f"data: {json.dumps({'type': 'task_summary', 'task_id': task.id, 'summary': summary})}\n\n"

        # 3. Reporting stage
        yield f"data: {json.dumps({'type': 'progress', 'stage': 'reporting', 'percentage': 90, 'text': 'Generating final report...'})}\n\n"

        # Generate report
        report = await reporting_service.generate_report(topic, task_summaries)

        # Push final report
        yield f"data: {json.dumps({'type': 'report', 'data': report})}\n\n"

        # Complete
        yield f"data: {json.dumps({'type': 'progress', 'stage': 'completed', 'percentage': 100, 'text': 'Research complete!'})}\n\n"

    except Exception as e:
        # Error handling
        yield f"data: {json.dumps({'type': 'error', 'message': str(e)})}\n\n"

@app.post("/api/research")
async def research(request: ResearchRequest):
    """Research endpoint (SSE)"""
    return StreamingResponse(
        research_stream(request.topic),
        media_type="text/event-stream",
        headers={
            "Cache-Control": "no-cache",
            "Connection": "keep-alive",
        }
    )
```

**SSE 수신을 위해 EventSource를 사용하는 프런트엔드**:

```typescript
// composables/useResearch.ts
import { ref } from 'vue'

export function useResearch() {
  const isLoading = ref(false)
  const progressPercentage = ref(0)
  const progressText = ref('')
  const markdownContent = ref('')
  const error = ref<string | null>(null)

  const startResearch = (topic: string) => {
    isLoading.value = true
    error.value = null

    // Create EventSource
    const eventSource = new EventSource(`/api/research?topic=${encodeURIComponent(topic)}`)

    // Listen for messages
    eventSource.onmessage = (event) => {
      const data = JSON.parse(event.data)

      switch (data.type) {
        case 'progress':
          progressPercentage.value = data.percentage
          progressText.value = data.text
          break

        case 'plan':
          // Display planning results
          console.log('Planning results:', data.data)
          break

        case 'task_summary':
          // Append task summary to Markdown
          markdownContent.value += `\n\n## Task ${data.task_id}\n\n${data.summary}`
          break

        case 'report':
          // Display final report
          markdownContent.value = data.data
          break

        case 'error':
          error.value = data.message
          eventSource.close()
          isLoading.value = false
          break

        case 'completed':
          eventSource.close()
          isLoading.value = false
          break
      }
    }

    // Error handling
    eventSource.onerror = (err) => {
      console.error('SSE error:', err)
      error.value = 'Connection failed, please retry'
      eventSource.close()
      isLoading.value = false
    }
  }

  return {
    isLoading,
    progressPercentage,
    progressText,
    markdownContent,
    error,
    startResearch,
  }
}
```

**구성요소에서 사용**:

```vue
<script setup lang="ts">
import { useResearch } from '@/composables/useResearch'

const {
  isLoading,
  progressPercentage,
  progressText,
  markdownContent,
  error,
  startResearch
} = useResearch()

const handleStartResearch = (topic: string) => {
  startResearch(topic)
}
</script>
```

14.6.3 연구 결과 시각화

연구 결과는 제목, 단락, 목록, 인용문 및 기타 요소를 포함하여 Markdown 형식으로 표시됩니다. 우리는 `marked` 라이브러리를 사용하여 Markdown을 HTML로 변환하고 사용자 정의 스타일을 추가합니다.

**렌더링 마크다운**:

```typescript
import { marked } from 'marked'

// Configure marked
marked.setOptions({
  breaks: true,  // Support line breaks
  gfm: true,     // Support GitHub Flavored Markdown
})

// Render
const renderedHtml = marked(markdownContent.value)
```

연구 보고서에는 다수의 출처 인용이 포함되어 있으며, 이를 특별히 처리해야 합니다.

```markdown
## References

### Task 1: Basic Information about Datawhale
- [Datawhale GitHub](https://github.com/datawhalechina)
- [Datawhale Official Website](https://datawhale.club)

### Task 2: Main Projects of Datawhale
- [Hello-Agents Tutorial](https://github.com/datawhalechina/Hello-Agents)
...
```

전체 화면 모달 대화 상자 UI, SSE 실시간 진행률 표시 및 Markdown 결과 시각화를 통해 사용자 친화적인 프런트 엔드 인터페이스를 구축했습니다. 사용자는 연구 진행 상황을 명확하게 확인하고 연구 결과를 아름다운 형식으로 볼 수 있습니다.

## 14.7 장 요약

이 장에서는 완전히 자동화된 심층 연구 에이전트 시스템을 처음부터 구축했습니다. 핵심 사항을 검토해 보겠습니다.

**(1) TODO 중심 연구 패러다임**

우리는 TODO 중심 연구라는 새로운 연구 패러다임을 제안했습니다. 이 패러다임은 복잡한 연구 주제를 실행 가능한 하위 작업으로 분해하고 다음 세 단계를 통해 연구를 완료합니다.

- **기획 단계**: 연구 주제를 3~5개의 하위 작업으로 분해하고 각 하위 작업에는 제목, 의도, 검색어가 포함됩니다.
- **실행 단계**: 하위 작업별 검색 및 요약을 수행하여 구조화된 지식 생성
- **보고 단계**: 모든 하위 업무 요약 통합, 최종 연구 보고서 생성

이 패러다임의 장점은 다음과 같습니다.

1. **강력한 제어 가능성**: 각 하위 작업에는 명확한 목표와 범위가 있습니다.
2. **신뢰할 수 있는 품질**: 전담 에이전트가 각 단계에서 품질을 보장합니다.
3. **디버그 용이**: 각 하위 작업을 개별적으로 디버깅할 수 있습니다.
4. **좋은 확장성**: 쉽게 새 하위 작업을 추가하거나 기존 하위 작업을 수정할 수 있습니다.

**(2) 3개 에이전트 협업 시스템**

우리는 각각 자신의 임무를 수행하는 세 명의 전문 에이전트를 설계했습니다.

- **TODO Planner(연구 기획 전문가)**: 연구 주제를 하위 작업으로 분해하는 역할을 담당합니다.
- **Task Summarizer(Task Summarization Expert)** : 하위 Task별 검색 결과를 요약하는 역할을 담당
- **보고서 작성자(보고서 작성 전문가)** : 모든 하위 업무의 요약을 통합하고 최종 보고서를 생성하는 역할을 담당합니다.

이 디자인의 장점은 다음과 같습니다.

1. **명확한 책임**: 각 에이전트는 특정 작업에 집중합니다.
2. **프롬프트 최적화**: 각 상담원에 대한 특화된 프롬프트를 사용자 정의할 수 있습니다.
3. **유지보수 용이**: 하나의 에이전트를 수정해도 다른 에이전트에는 영향을 주지 않습니다.
4. **품질 보증**: 각 에이전트는 해당 분야의 "전문가"입니다.

**(3) ToolAwareSimpleAgent 설계**

HelloAgents 프레임워크의 `SimpleAgent`를 확장하고 `ToolAwareSimpleAgent`을 구현했습니다. 이 에이전트에는 도구 호출 수신 기능이 있으며 다음을 수행할 수 있습니다.

- **도구 호출 듣기**: 콜백 기능을 통해 각 도구 호출을 듣습니다.
- **실시간 피드백**: 공구 호출 정보를 실시간으로 프론트엔드에 푸시
- **디버깅 지원**: 간편한 디버깅을 위해 모든 도구 호출을 녹음합니다.

이 에이전트는 HelloAgents 프레임워크에 통합되었으며 다른 프로젝트에서 재사용할 수 있습니다.

**(4) 도구 시스템 통합**

우리는 HelloAgents 프레임워크의 도구 시스템을 완전히 활용했습니다.

- **SearchTool**: 더 많은 검색 엔진을 지원하도록 확장됨 (Tavily, DuckDuckGo, Perplexity, etc.)
- **NoteTool**: 지속적인 연구 진행, 지원 복구 및 감사
- **ToolRegistry**: 모든 도구의 통합 관리, 맞춤형 확장 지원

구성 기반 설계를 통해 사용자는 코드 수정 없이 쉽게 검색 엔진을 전환할 수 있습니다.

**(5) 핵심 서비스 구현**

에이전트와 도구를 연결하는 네 가지 핵심 서비스를 구현했습니다.

- **PlanningService**: 계획 에이전트 호출, JSON 구문 분석, 형식 검증
- **SummarizationService**: 요약 Agent 호출, 검색 결과 처리, 소스 추출
- **ReportingService**: 보고서 에이전트 호출, 요약 통합, 보고서 생성
- **SearchService**: 검색 엔진 예약, 결과 처리, 오류 저하, 결과 캐싱

이러한 서비스는 각각 자신의 임무를 수행하고 명확한 인터페이스를 통해 협업하여 연구 주제부터 최종 보고서까지 자동화된 프로세스를 달성합니다.

**(6) 프런트엔드 상호 작용 디자인**

우리는 사용자 친화적인 프런트엔드 인터페이스를 설계했습니다.

- **전체 화면 모달 대화 상자**: 몰입형 경험, 명확한 계층 구조
- **SSE 실시간 진행**: 연구 진행 상황 실시간 표시, 우수한 사용자 경험
- **마크다운 시각화**: 아름다운 형식, 명확한 구조

Vue 3 + TypeScript + SSE 기술 스택을 통해 최신 웹 애플리케이션을 구현했습니다.

이 지식은 심층 연구 보조원에게 적용될 수 있을 뿐만 아니라 다른 AI 애플리케이션에도 적용될 수 있습니다. 독자들이 이 장을 기반으로 더 많은 가능성을 탐색하고 더 강력한 AI 시스템을 구축할 수 있기를 바랍니다.

다음 장에서는 게임 엔진인 Cyber ​​Town과 결합된 다중 에이전트 시스템을 구축하여 에이전트 간의 복잡한 상호 작용 및 협업 패턴을 살펴보겠습니다. 계속 지켜봐 주시기 바랍니다!

