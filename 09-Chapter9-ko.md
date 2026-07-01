# 9장 컨텍스트 엔지니어링

이전 장에서는 에이전트용 메모리 시스템과 RAG를 소개했습니다. 그러나 실제 복잡한 시나리오에서 에이전트가 안정적으로 "생각"하고 "행동"할 수 있도록 하려면 기억과 검색만으로는 충분하지 않습니다. 모델에 적합한 "컨텍스트"를 지속적이고 체계적으로 구성하는 엔지니어링 방법론이 필요합니다. 이것이 이번 장의 주제입니다: 컨텍스트 엔지니어링. "각 모델 호출 전에 재사용 가능하고 측정 가능하며 발전 가능한 방식으로 입력 컨텍스트를 조립하고 최적화하는 방법"에 중점을 두어 정확성, 견고성 및 효율성<sup>[1][2]</sup>을 향상시킵니다.

독자가 이 장의 전체 기능을 빠르게 경험할 수 있도록 직접 설치할 수 있는 Python 패키지를 제공합니다. 다음 명령을 사용하여 이 장에 해당하는 버전을 설치할 수 있습니다.

```bash
pip install "hello-agents[all]==0.2.8"
```

이 장에서는 주로 컨텍스트 엔지니어링의 핵심 개념과 사례를 소개하고 HelloAgents 프레임워크에 컨텍스트 빌더와 두 가지 지원 도구를 추가합니다.

- **ContextBuilder**(`hello_agents/context/builder.py`): 통일된 컨텍스트 관리 인터페이스를 제공하는 GSSC(Gather-Select-Structure-Compress) 파이프라인을 구현하는 컨텍스트 빌더
- **NoteTool**(`hello_agents/tools/builtin/note_tool.py`): 에이전트의 지속적인 메모리 관리를 지원하는 구조화된 메모 도구
- **TerminalTool**(`hello_agents/tools/builtin/terminal_tool.py`): 에이전트에 대한 파일 시스템 작업 및 적시 컨텍스트 검색을 지원하는 터미널 도구

이러한 구성 요소는 함께 장기적인 작업 관리 및 에이전트 검색을 구현하는 데 핵심이 되는 완전한 컨텍스트 엔지니어링 솔루션을 구성하며 후속 섹션에서 자세히 소개됩니다.

프레임워크 설치 외에도 `.env`에서 LLM API를 구성해야 합니다. 이 장의 예제에서는 주로 컨텍스트 관리 및 지능적인 의사 결정을 위해 대규모 언어 모델을 사용합니다.

구성이 완료되면 이 장의 학습 여정을 시작할 수 있습니다!

## 9.1 컨텍스트 엔지니어링이란?

수년간 프롬프트 엔지니어링이 응용 AI의 초점이 된 후 **컨텍스트 엔지니어링**이라는 새로운 용어가 대두되었습니다. 오늘날 언어 모델을 사용하여 시스템을 구축하는 것은 더 이상 프롬프트에서 올바른 문구와 단어를 찾는 것이 아니라 보다 거시적인 질문에 답하는 것입니다. **어떤 종류의 컨텍스트 구성이 모델이 우리가 기대하는 동작을 생성하게 만들 가능성이 가장 높습니까?**

소위 "컨텍스트"는 LLM(대형 언어 모델)을 샘플링할 때 포함되는 토큰 세트를 나타냅니다. 당면한 엔지니어링 문제는 예상 결과를 안정적으로 얻기 위해 LLM의 고유한 제약 조건 하에서 **이러한 토큰의 유용성을 최적화**하는 것입니다. LLM을 효과적으로 활용하려면 "컨텍스트에 따라 생각"하는 것이 필요한 경우가 많습니다. 즉, 모든 호출에서 LLM에 표시되는 전체 상태를 검사하고 이 상태가 유도할 수 있는 동작을 예측하는 것입니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/9-figures/9-1.webp" alt="" width="85%"/>
<p>그림 9.1 프롬프트 엔지니어링과 컨텍스트 엔지니어링</p>
</div>

이 섹션에서는 새로운 컨텍스트 엔지니어링을 살펴보고 **제어 가능하고 효과적인** 에이전트를 구축하기 위한 세련된 정신 모델을 제공합니다.

**컨텍스트 엔지니어링과 프롬프트 엔지니어링**

그림 9.1에서 볼 수 있듯이, 선도적인 모델 공급업체의 관점에서 볼 때 컨텍스트 엔지니어링은 프롬프트 엔지니어링의 자연스러운 진화입니다. 프롬프트 엔지니어링은 더 나은 결과를 얻기 위해 LLM 지침을 작성하고 구성하는 방법(예: 시스템 프롬프트 작성 및 구조화된 전략)에 중점을 둡니다. 컨텍스트 엔지니어링은 **추론 단계에서 "최적의 정보 세트(토큰)"를 계획하고 유지하는 방법**이며, 여기에는 프롬프트 자체뿐만 아니라 컨텍스트 창에 입력되는 기타 모든 정보도 포함됩니다.

LLM 엔지니어링의 초기 단계에서는 대부분의 사용 사례(일일 채팅 제외)에서 단일 회전 분류 또는 텍스트 생성을 위해 미세 조정된 프롬프트 최적화가 필요했기 때문에 프롬프트가 주요 작업인 경우가 많았습니다. 이름에서 알 수 있듯이 프롬프트 엔지니어링의 핵심은 "효과적인 프롬프트를 작성하는 방법", 특히 시스템 프롬프트를 작성하는 것입니다. 그러나 더 오랜 시간 동안 여러 추론 라운드에 걸쳐 작동하는 더 강력한 에이전트를 엔지니어링하기 시작하면 시스템 지침, 도구, MCP(모델 컨텍스트 프로토콜), 외부 데이터, 메시지 기록 등을 포함하여 **전체 컨텍스트 상태**를 관리할 수 있는 전략이 필요합니다.

루프에서 실행되는 에이전트는 다음 추론 라운드와 관련될 수 있는 데이터를 지속적으로 생성합니다. 이 정보는 **주기적으로 개선**되어야 합니다. 따라서 컨텍스트 엔지니어링의 "예술과 기술"은 지속적으로 확장되는 "후보 정보 세계"에서 **제한된 컨텍스트 창에 어떤 콘텐츠가 들어가야 하는지 식별**하는 데 있습니다.

## 9.2 컨텍스트 엔지니어링이 중요한 이유

모델이 점점 더 빨라지고 더 큰 데이터 규모를 처리할 수 있지만 인간과 마찬가지로 LLM도 특정 지점에서 "방황"하거나 "혼란"할 것입니다. 건초 더미 벤치마크에서는 **컨텍스트 부패**라는 현상이 드러났습니다. 컨텍스트 창의 토큰 수가 증가함에 따라 컨텍스트에서 정보를 정확하게 기억하는 모델의 능력이 실제로 감소합니다.

모델마다 성능 저하 곡선이 더 완만할 수 있지만 이러한 특성은 거의 모든 모델에서 나타납니다. 따라서 **컨텍스트는 한계 수익이 감소하는 제한된 자원으로 보아야 합니다**. 인간의 작업 기억 용량이 제한되어 있는 것처럼 LLM에도 "주의 예산"이 있습니다. 각각의 새로운 토큰은 이 예산의 일부를 소비하므로 LLM에 어떤 토큰을 제공해야 하는지에 대해 더 주의해야 합니다.

이러한 희소성은 우연이 아니라 LLM의 구조적 제약에서 비롯됩니다. 변환기를 사용하면 각 토큰이 컨텍스트에서 **모든** 토큰과 연관을 설정하여 이론적으로 \(n^2\) 쌍의 주의 관계를 형성할 수 있습니다. 컨텍스트 길이가 늘어남에 따라 이러한 쌍별 관계를 모델링하는 모델의 능력은 "얇아지게 늘어나" 자연스럽게 "컨텍스트 규모"와 "주의 집중" 사이에 긴장감을 조성합니다. 또한 모델의 주의 패턴은 훈련 데이터 분포에서 비롯됩니다. 짧은 시퀀스는 일반적으로 긴 시퀀스보다 더 일반적이므로 모델은 "전체 컨텍스트 종속성"에 대한 경험이 적고 특수 매개변수도 적습니다.

위치 인코딩 보간과 같은 기술을 사용하면 모델이 추론 시간에 훈련하는 동안보다 더 긴 시퀀스에 "적응"할 수 있지만 토큰 위치를 이해하는 데 어느 정도 정밀도가 희생됩니다. 전반적으로 이러한 요소는 "절벽과 같은" 붕괴가 아닌 **성능 변화**를 형성합니다. 모델은 긴 컨텍스트에서 여전히 강력하지만 짧은 컨텍스트에 비해 정보 검색 및 장거리 추론의 정확성이 떨어집니다.

위의 현실을 바탕으로 강력한 에이전트를 구축하려면 **의식적인 컨텍스트 엔지니어링**이 필요합니다.

9.2.1 효과적인 맥락의 "해부학"

"제한된 주의 예산"이라는 제약 하에서 우수한 컨텍스트 엔지니어링의 목표는 **가능한 적지만 높은 신호 밀도 토큰을 사용하여 예상 결과를 얻을 확률을 최대화**하는 것입니다. 실제로는 다음 구성 요소를 중심으로 엔지니어링하는 것이 좋습니다.

- **시스템 프롬프트**: "딱 맞는" 높이의 정보 계층 구조를 갖춘 명확하고 간단한 언어입니다. 두 가지 극단적인 경우에 흔히 발생하는 함정:
  - 오버 하드코딩: 장기간 유지 관리 비용이 높고 취약성이 있는 복잡하고 취약한 if-else 논리를 프롬프트에 작성합니다.
  - 너무 모호함: 거시적 목표와 일반화된 지침만 제공하고 예상 결과에 대한 **구체적인 신호**가 부족하거나 잘못된 '공유 컨텍스트'를 가정합니다.
  프롬프트를 XML/마크다운으로 구분된 섹션 (such as ⟪P0⟫, ⟪P1⟫, tool guidance, output description, etc.)로 구성하는 것이 좋습니다. 형식에 관계없이 추구하는 것은 예상되는 동작을 완전히 설명할 수 있는 **"최소 필수 정보 세트"입니다**("최소"는 "최단"과 동일하지 않음). 먼저 최소 프롬프트에서 최상의 모델을 실행한 다음 실패 모드를 기반으로 명확한 지침과 예제를 추가합니다.

- **도구**: 도구는 에이전트와 정보/행동 공간 간의 계약을 정의하고 효율성을 촉진해야 합니다. 효율적인 에이전트 행동을 장려하는 동시에 **토큰 친화적인** 정보를 반환해야 합니다. 도구는 다음을 수행해야 합니다.
  - 중복이 적고 인터페이스 의미가 명확한 단일 책임을 갖습니다.
  - 오류에 강인해야 합니다.
  - 표현과 추론에서 모델의 강점을 최대한 활용하여 명확하고 모호하지 않은 매개변수 설명을 갖습니다.
  일반적인 실패 모드는 "비대해진 도구 세트"입니다. 기능 경계가 모호하여 "사용할 도구"에 대한 결정 자체가 모호해집니다. **엔지니어가 어떤 도구를 사용해야 할지 알 수 없다면 상담원이 더 잘할 것이라고 기대하지 마세요**. "MVTS(최소 실행 가능 도구 세트)"를 주의 깊게 식별하면 장기적인 상호 작용에서 안정성과 유지 관리 가능성이 크게 향상될 수 있습니다.

- **몇 가지 예시**: 항상 예시를 제공하는 것이 좋지만 프롬프트에 "모든 경계 조건"을 입력하는 것은 권장하지 않습니다. "예상되는 동작"을 직접적으로 묘사하는 **다양하고 일반적인** 예시 세트를 신중하게 선택하세요. LLM의 경우 **좋은 예는 수천 단어의 가치가 있습니다**.

전반적인 지침 원칙은 **충분하지만 간결한 정보**입니다. 그림 9.2에서 볼 수 있듯이 이는 런타임에 진입하는 동적 검색입니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/9-figures/9-2.webp" alt="" width="85%"/>
<p>그림 9.2 시스템 프롬프트 보정</p>
</div>

### 9.2.2 컨텍스트 검색 및 에이전트 검색

간결한 정의: **에이전트 = LLM이 루프에서 자동으로 도구를 호출**. 기본 모델의 기능이 향상됨에 따라 에이전트의 자율성 수준이 향상될 수 있습니다. 에이전트는 복잡한 문제 공간을 보다 독립적으로 탐색하고 오류에서 복구할 수 있습니다.

엔지니어링 실무는 "추론 전 일회성 검색(임베딩 검색)"에서 "**JIT(Just-in-time) 컨텍스트**"로 점차 전환되고 있습니다. 후자는 더 이상 모든 관련 데이터를 미리 로드하지 않지만 **경량 참조** (file paths, storage queries, URLs, etc.)를 유지하여 런타임 시 도구를 통해 필요한 데이터를 동적으로 로드합니다. 이를 통해 모델은 전체 데이터 블록을 컨텍스트에 한꺼번에 채우지 않고도 대상 쿼리를 작성하고, 필요한 결과를 캐시하고, <code>head</code>/<code>tail</code>와 같은 명령을 사용하여 대량의 데이터를 분석할 수 있습니다. 인지 패턴은 인간에 더 가깝습니다. 모든 정보를 기억하지는 않지만 파일 시스템, 받은 편지함, 북마크와 같은 외부 색인을 사용하여 요청 시 추출합니다.

저장 효율성 외에도 **참조의 메타데이터** 자체는 동작(디렉터리 계층 구조, 명명 규칙, 타임스탬프 등)을 개선하는 데 도움이 될 수 있으며 모두 암시적으로 "목적과 적시성"을 전달합니다. 예를 들어, <code>tests/test_utils.py</code>과 <code>src/core/test_utils.py</code>는 서로 다른 의미를 갖습니다.

에이전트가 자율적으로 탐색하고 검색할 수 있도록 하면 **점진적 공개**도 가능해집니다. 각 상호 작용 단계는 새로운 컨텍스트를 생성하고 이에 따라 다음 결정을 안내합니다. 즉, 파일 크기는 복잡성을 암시하고, 목적에 따른 이름 힌트는 타임스탬프는 관련성에 대한 힌트를 제공합니다. 에이전트는 작업 메모리에 "현재 필요한 하위 집합"만 유지하고 보충 지속성을 위해 "메모 작성"을 사용하여 계층별로 이해를 구축할 수 있으므로 "포괄성에 의해 끌려가지 않고" 초점을 유지할 수 있습니다.

단점은 런타임 탐색이 사전 계산된 검색보다 느린 경우가 많으며 모델에 올바른 도구와 경험적 방법이 있는지 확인하기 위해 "의견이 있는" 엔지니어링 설계가 필요하다는 것입니다. 안내가 없으면 상담원은 도구를 오용하거나, 막다른 골목을 쫓거나, 주요 정보를 놓치게 되어 상황 낭비를 초래할 수 있습니다.

많은 시나리오에서는 **하이브리드 전략**이 더 효과적입니다. 즉, 속도를 보장하기 위해 소량의 "고가치" 컨텍스트를 미리 로드한 다음 에이전트가 필요에 따라 자동 탐색을 계속할 수 있도록 허용합니다. 경계 선택은 작업 역학 및 적시성 요구 사항에 따라 달라집니다. 엔지니어링에서는 "프로젝트 규칙 설명 (such as README/guides)"과 같은 파일을 미리 로드하는 동시에 <code>glob</code>, <code>grep</code>와 같은 기본 형식을 제공하여 에이전트가 특정 파일을 적시에 검색할 수 있도록 함으로써 오래된 인덱스와 복잡한 구문 트리로 인한 매몰 비용을 피할 수 있습니다.

9.2.3 장거리 작업을 위한 컨텍스트 엔지니어링

장거리 작업에서는 에이전트가 컨텍스트 창을 초과하는 작업 시퀀스에서 일관성, 컨텍스트 일관성 및 목표 방향을 유지해야 합니다. 예를 들어 대규모 코드베이스 마이그레이션, 몇 시간에 걸친 체계적인 연구 등이 있습니다. 컨텍스트 창을 무한히 늘릴 것으로 기대하는 것은 "컨텍스트 오염" 및 관련성 저하 문제를 해결할 수 없으므로 이러한 제약에 직접 직면하는 엔지니어링 방법이 필요합니다: **압축**, **구조화된 메모 작성** 및 **하위 에이전트 아키텍처**.

- **다짐**
  - 정의: 대화가 컨텍스트 제한에 접근하면 충실도 높은 요약을 수행하고 요약으로 새 컨텍스트 창을 다시 시작하여 장거리 일관성을 유지합니다.
  - 연습: 모델이 아키텍처 결정, 해결되지 않은 결함, 구현 세부 사항, 반복적인 도구 출력 및 노이즈 폐기를 압축하고 유지하도록 합니다. 새 창에는 압축된 요약과 최근에 관련성이 높은 몇 가지 아티팩트(예: "최근에 액세스한 파일")가 포함됩니다.
  - 조정 제안: 먼저 **재현율**을 최적화하고(주요 정보가 누락되지 않았는지 확인), 그런 다음 **정밀도**를 최적화합니다(중복 콘텐츠 제거). 안전한 "가벼운 터치" 압축은 "도구 호출 및 깊은 기록 결과"를 정리하는 것입니다.

- **구조화된 메모 작성**
  - 정의: "에이전트 메모리"라고도 합니다. 에이전트는 고정된 빈도로 **컨텍스트 외부의 영구 저장소**에 주요 정보를 기록하고 후속 단계에서 필요할 때 다시 가져옵니다.
  - 가치: 매우 낮은 컨텍스트 오버헤드로 지속적인 상태 및 종속성을 유지합니다. 예를 들어 TODO 목록, 프로젝트 NOTES.md, 주요 결론/의존성/차단 항목 색인, 수십 개의 도구 호출 및 다중 컨텍스트 재설정 전반에 걸쳐 진행 상황과 일관성을 유지합니다.
  - 참고: 비코딩 시나리오 (such as long-term strategic tasks, goal management and statistical counting in games/simulations)에서도 동일하게 효과적입니다. 8장의 <code>MemoryTool</code>과 결합하면 파일 기반/벡터 기반 외부 메모리를 쉽게 구현하고 런타임에 검색할 수 있습니다.

- **하위 에이전트 아키텍처**
  - 아이디어: 기본 에이전트는 높은 수준의 계획 및 종합을 담당하고, 여러 전문 하위 에이전트는 각각 깊이 파고들어 도구를 호출하고 "깨끗한 컨텍스트 창"에서 탐색한 후 **축약된 요약**(일반적으로 1,000~2,000개 토큰)만 반환합니다.
  - 이점: 우려사항의 분리를 달성합니다. 복잡한 검색 컨텍스트는 하위 에이전트 내부에 남아 있는 반면, 주 에이전트는 통합과 추론에 중점을 둡니다. 병렬 탐색이 필요한 복잡한 연구/분석 작업에 적합합니다.
  - 경험: 공공 다중 에이전트 연구 시스템은 이 패턴이 복잡한 연구 작업에서 단일 에이전트 기준에 비해 상당한 이점을 가지고 있음을 보여줍니다.

방법 절충은 다음과 같은 경험 법칙을 따를 수 있습니다.

- **압축**: 긴 대화 연속성이 필요한 작업에 적합하며 컨텍스트 "릴레이"를 강조합니다.
- **구조화된 메모 작성**: 마일스톤/단계적 결과가 있는 반복적인 개발 및 연구에 적합합니다.
- **하위 에이전트 아키텍처**: 병렬 탐색의 이점을 누릴 수 있는 복잡한 연구 및 분석에 적합합니다.

모델 기능이 계속해서 향상되더라도 "긴 상호 작용에서 일관성과 초점을 유지하는 것"은 강력한 에이전트를 구축하는 데 있어서 여전히 핵심 과제로 남아 있습니다. 신중하고 체계적인 컨텍스트 엔지니어링은 장기적으로 핵심 가치를 유지할 것입니다.

## 9.3 Hello-Agents 연습: ContextBuilder

이 섹션에서는 HelloAgents 프레임워크의 컨텍스트 엔지니어링 사례를 자세히 설명합니다. 우리는 디자인 동기, 핵심 데이터 구조, 구현 세부 사항에서 사례 완료에 이르기까지 프로덕션 수준의 컨텍스트 관리 시스템을 구축하는 방법을 점진적으로 시연할 것입니다. ContextBuilder의 디자인 철학은 "간단하고 효율적"입니다. 불필요한 복잡성을 제거하고 "관련성 + 최신성" 점수를 기준으로 균일하게 선택하며 에이전트 모듈성 및 유지 관리 용이성의 엔지니어링 방향을 따릅니다.

9.3.1 디자인 동기와 목표

ContextBuilder를 구축하기 전에 먼저 디자인 목표와 핵심 가치를 명확히 해야 합니다. 뛰어난 컨텍스트 관리 시스템은 다음과 같은 주요 문제를 해결해야 합니다.

1. **통합 항목**: "수집-선택-구조-압축"을 재사용 가능한 파이프라인으로 추상화하여 에이전트 구현에서 반복적인 템플릿 코드를 줄입니다. 이 통합 인터페이스 설계를 통해 개발자는 각 에이전트에서 컨텍스트 관리 논리를 반복적으로 작성하지 않아도 됩니다.

2. **안정적인 형식**: 디버깅, A/B 테스트 및 평가를 용이하게 하는 고정된 뼈대를 사용하여 컨텍스트 템플릿을 출력합니다. 우리는 섹션화된 템플릿 구조를 채택했습니다.
   - `[Role & Policies]`: 상담원의 역할 포지셔닝 및 행동 지침을 명확히 합니다.
   - `[Task]` : 현재 완료해야 할 특정 작업
   - `[State]`: Agent의 현재 상태 및 컨텍스트 정보
   - `[Evidence]` : 외부 지식베이스에서 검색된 증거정보
   - `[Context]`: 역사적 대화와 관련된 추억
   - `[Output]`: 예상되는 출력 형식 및 요구 사항

3. **예산 수호자**: 토큰 예산 내에서 가능한 한 높은 가치의 정보를 유지하여 초과 한도 상황에 대한 대체 압축 전략을 제공합니다. 이를 통해 엄청난 양의 정보가 포함된 시나리오에서도 시스템이 안정적으로 실행될 수 있습니다.

4. **최소 규칙**: 복잡성 증가를 피하기 위해 소스/우선순위와 같은 분류 차원을 도입하지 마십시오. 실습에 따르면 관련성과 최근성을 기반으로 한 간단한 점수 매기기 메커니즘이 대부분의 시나리오에서 충분히 효과적이라는 것을 알 수 있습니다.

### 9.3.2 핵심 데이터 구조

ContextBuilder의 구현은 시스템의 구성과 정보 단위를 정의하는 두 가지 핵심 데이터 구조에 의존합니다.

(1) ContextPacket: 후보자 정보 패키지

```python
from dataclasses import dataclass
from typing import Optional, Dict, Any
from datetime import datetime

@dataclass
class ContextPacket:
    """Candidate information package

    Attributes:
        content: Information content
        timestamp: Timestamp
        token_count: Token count
        relevance_score: Relevance score (0.0-1.0)
        metadata: Optional metadata
    """
    content: str
    timestamp: datetime
    token_count: int
    relevance_score: float = 0.5
    metadata: Optional[Dict[str, Any]] = None

    def __post_init__(self):
        """Post-initialization processing"""
        if self.metadata is None:
            self.metadata = {}
        # Ensure relevance score is within valid range
        self.relevance_score = max(0.0, min(1.0, self.relevance_score))
```

`ContextPacket`은 시스템 정보의 기본 단위입니다. 각 후보 정보는 콘텐츠, 타임스탬프, 토큰 수, 관련성 점수와 같은 핵심 속성을 포함하는 ContextPacket으로 캡슐화됩니다. 이 통합된 데이터 구조는 후속 선택 및 정렬 논리를 단순화합니다.

(2) ContextConfig: 구성 관리

```python
@dataclass
class ContextConfig:
    """Context building configuration

    Attributes:
        max_tokens: Maximum token count
        reserve_ratio: Ratio reserved for system instructions (0.0-1.0)
        min_relevance: Minimum relevance threshold
        enable_compression: Whether to enable compression
        recency_weight: Recency weight (0.0-1.0)
        relevance_weight: Relevance weight (0.0-1.0)
    """
    max_tokens: int = 3000
    reserve_ratio: float = 0.2
    min_relevance: float = 0.1
    enable_compression: bool = True
    recency_weight: float = 0.3
    relevance_weight: float = 0.7

    def __post_init__(self):
        """Validate configuration parameters"""
        assert 0.0 <= self.reserve_ratio <= 1.0, "reserve_ratio must be in [0, 1] range"
        assert 0.0 <= self.min_relevance <= 1.0, "min_relevance must be in [0, 1] range"
        assert abs(self.recency_weight + self.relevance_weight - 1.0) < 1e-6, \
            "recency_weight + relevance_weight must equal 1.0"
```

`ContextConfig`는 구성 가능한 모든 매개변수를 캡슐화하여 시스템 동작을 유연하게 조정할 수 있도록 합니다. 특히 주목할만한 것은 `reserve_ratio` 매개변수입니다. 이는 시스템 명령과 같은 주요 정보가 항상 충분한 공간을 가지며 다른 정보에 의해 압착되지 않도록 보장합니다.

### 9.3.3 GSSC 파이프라인 상세 설명

ContextBuilder의 핵심은 컨텍스트 구축 프로세스를 4개의 명확한 단계로 분해하는 GSSC(Gather-Select-Structure-Compress) 파이프라인입니다. 각 단계의 구현 세부정보를 살펴보겠습니다.

(1) 수집: 다중 소스 정보 수집

첫 번째 단계는 여러 소스로부터 후보자 정보를 수집하는 것입니다. 이 단계의 핵심은 내결함성과 유연성입니다.

```python
def _gather(
    self,
    user_query: str,
    conversation_history: Optional[List[Message]] = None,
    system_instructions: Optional[str] = None,
    custom_packets: Optional[List[ContextPacket]] = None
) -> List[ContextPacket]:
    """Collect all candidate information

    Args:
        user_query: User query
        conversation_history: Conversation history
        system_instructions: System instructions
        custom_packets: Custom information packages

    Returns:
        List[ContextPacket]: Candidate information list
    """
    packets = []

    # 1. Add system instructions (highest priority, not scored)
    if system_instructions:
        packets.append(ContextPacket(
            content=system_instructions,
            timestamp=datetime.now(),
            token_count=self._count_tokens(system_instructions),
            relevance_score=1.0,  # System instructions always retained
            metadata={"type": "system_instruction", "priority": "high"}
        ))

    # 2. Retrieve relevant memories from memory system
    if self.memory_tool:
        try:
            memory_results = self.memory_tool.run({
                "action": "search",
                "query": user_query,
                "limit": 10,
                "min_importance": 0.3
            })
            # Parse memory results and convert to ContextPacket
            memory_packets = self._parse_memory_results(memory_results, user_query)
            packets.extend(memory_packets)
        except Exception as e:
            print(f"[WARNING] Memory retrieval failed: {e}")

    # 3. Retrieve relevant knowledge from RAG system
    if self.rag_tool:
        try:
            rag_results = self.rag_tool.run({
                "action": "search",
                "query": user_query,
                "limit": 5,
                "min_score": 0.3
            })
            # Parse RAG results and convert to ContextPacket
            rag_packets = self._parse_rag_results(rag_results, user_query)
            packets.extend(rag_packets)
        except Exception as e:
            print(f"[WARNING] RAG retrieval failed: {e}")

    # 4. Add conversation history (only keep recent N entries)
    if conversation_history:
        recent_history = conversation_history[-5:]  # Default keep recent 5 entries
        for msg in recent_history:
            packets.append(ContextPacket(
                content=f"{msg.role}: {msg.content}",
                timestamp=msg.timestamp if hasattr(msg, 'timestamp') else datetime.now(),
                token_count=self._count_tokens(msg.content),
                relevance_score=0.6,  # Base relevance of historical messages
                metadata={"type": "conversation_history", "role": msg.role}
            ))

    # 5. Add custom information packages
    if custom_packets:
        packets.extend(custom_packets)

    print(f"[ContextBuilder] Collected {len(packets)} candidate information packages")
    return packets
```

이 구현에서는 몇 가지 중요한 설계 고려 사항을 보여줍니다.

- **내결함성 메커니즘**: 각 외부 데이터 소스 호출은 try-Exception으로 래핑되어 단일 소스의 오류가 전체 프로세스에 영향을 미치지 않도록 보장합니다.
- **우선순위 처리**: 시스템 지침은 높은 우선순위로 표시되어 항상 유지됩니다.
- **기록 제한**: 대화 기록은 가장 최근 항목만 유지하므로 기록 정보가 컨텍스트 창을 차지하지 않습니다.

(2) 선택: 지능형 정보 선택

두 번째 단계는 관련성과 최신성을 기준으로 후보자 정보를 점수화하고 선택하는 것입니다. 이는 전체 파이프라인의 핵심이며 최종 컨텍스트의 품질을 직접적으로 결정합니다.

```python
def _select(
    self,
    packets: List[ContextPacket],
    user_query: str,
    available_tokens: int
) -> List[ContextPacket]:
    """Select the most relevant information packages

    Args:
        packets: Candidate information package list
        user_query: User query (for calculating relevance)
        available_tokens: Available token count

    Returns:
        List[ContextPacket]: Selected information package list
    """
    # 1. Separate system instructions and other information
    system_packets = [p for p in packets if p.metadata.get("type") == "system_instruction"]
    other_packets = [p for p in packets if p.metadata.get("type") != "system_instruction"]

    # 2. Calculate tokens occupied by system instructions
    system_tokens = sum(p.token_count for p in system_packets)
    remaining_tokens = available_tokens - system_tokens

    if remaining_tokens <= 0:
        print("[WARNING] System instructions have occupied all token budget")
        return system_packets

    # 3. Calculate comprehensive scores for other information
    scored_packets = []
    for packet in other_packets:
        # Calculate relevance score (if not yet calculated)
        if packet.relevance_score == 0.5:  # Default value, needs recalculation
            relevance = self._calculate_relevance(packet.content, user_query)
            packet.relevance_score = relevance

        # Calculate recency score
        recency = self._calculate_recency(packet.timestamp)

        # Combined score = relevance weight × relevance + recency weight × recency
        combined_score = (
            self.config.relevance_weight * packet.relevance_score +
            self.config.recency_weight * recency
        )

        # Filter information below minimum relevance threshold
        if packet.relevance_score >= self.config.min_relevance:
            scored_packets.append((combined_score, packet))

    # 4. Sort by score in descending order
    scored_packets.sort(key=lambda x: x[0], reverse=True)

    # 5. Greedy selection: fill from high to low score until token limit is reached
    selected = system_packets.copy()
    current_tokens = system_tokens

    for score, packet in scored_packets:
        if current_tokens + packet.token_count <= available_tokens:
            selected.append(packet)
            current_tokens += packet.token_count
        else:
            # Token budget is full, stop selection
            break

    print(f"[ContextBuilder] Selected {len(selected)} information packages, total {current_tokens} tokens")
    return selected

def _calculate_relevance(self, content: str, query: str) -> float:
    """Calculate relevance between content and query

    Uses simple keyword overlap algorithm. In production, can be replaced with vector similarity calculation.

    Args:
        content: Content text
        query: Query text

    Returns:
        float: Relevance score (0.0-1.0)
    """
    # Tokenization (simple implementation, can use more complex tokenizers)
    content_words = set(content.lower().split())
    query_words = set(query.lower().split())

    if not query_words:
        return 0.0

    # Jaccard similarity
    intersection = content_words & query_words
    union = content_words | query_words

    return len(intersection) / len(union) if union else 0.0

def _calculate_recency(self, timestamp: datetime) -> float:
    """Calculate temporal recency score

    Uses exponential decay model, maintains high score within 24 hours, then gradually decays.

    Args:
        timestamp: Information timestamp

    Returns:
        float: Recency score (0.0-1.0)
    """
    import math

    age_hours = (datetime.now() - timestamp).total_seconds() / 3600

    # Exponential decay: maintain high score within 24 hours, then gradually decay
    decay_factor = 0.1  # Decay coefficient
    recency_score = math.exp(-decay_factor * age_hours / 24)

    return max(0.1, min(1.0, recency_score))  # Limit to [0.1, 1.0] range
```

선택 단계의 핵심 알고리즘에는 몇 가지 중요한 엔지니어링 고려 사항이 포함되어 있습니다.

- **점수 매기기 메커니즘**: 구성 가능한 가중치와 함께 관련성과 최근성의 가중치 조합을 사용합니다.
- **그리디 알고리즘**: 높은 점수부터 낮은 점수까지 채워 제한된 예산 내에서 가장 가치 있는 정보를 선택할 수 있도록 보장
- **필터링 메커니즘**: `min_relevance` 매개변수를 통해 품질이 낮은 정보를 필터링합니다.

(3) 구조: 구조화된 출력

세 번째 단계는 선택한 정보를 구조화된 컨텍스트 템플릿으로 구성하는 것입니다.

```python
def _structure(self, selected_packets: List[ContextPacket], user_query: str) -> str:
    """Organize selected information packages into structured context template

    Args:
        selected_packets: Selected information package list
        user_query: User query

    Returns:
        str: Structured context string
    """
    # Group by type
    system_instructions = []
    evidence = []
    context = []

    for packet in selected_packets:
        packet_type = packet.metadata.get("type", "general")

        if packet_type == "system_instruction":
            system_instructions.append(packet.content)
        elif packet_type in ["rag_result", "knowledge"]:
            evidence.append(packet.content)
        else:
            context.append(packet.content)

    # Build structured template
    sections = []

    # [Role & Policies]
    if system_instructions:
        sections.append("[Role & Policies]\n" + "\n".join(system_instructions))

    # [Task]
    sections.append(f"[Task]\n{user_query}")

    # [Evidence]
    if evidence:
        sections.append("[Evidence]\n" + "\n---\n".join(evidence))

    # [Context]
    if context:
        sections.append("[Context]\n" + "\n".join(context))

    # [Output]
    sections.append("[Output]\nPlease provide accurate, evidence-based answers based on the above information.")

    return "\n\n".join(sections)
```

구조화 단계에서는 흩어져 있는 정보 패키지를 명확한 섹션으로 구성합니다. 이 디자인에는 다음과 같은 몇 가지 장점이 있습니다.

- **가독성**: 명확한 섹션을 통해 인간과 모델 모두가 컨텍스트 구조를 더 쉽게 이해할 수 있습니다.
- **디버깅 가능성**: 문제 위치 파악이 더 쉽고, 문제가 있는 정보가 있는 영역을 빠르게 식별할 수 있습니다.
- **확장성**: 새로운 정보 소스를 추가하려면 새 섹션을 만들기만 하면 됩니다.

(4) 압축: 대체 압축

네 번째 단계는 초과 제한 컨텍스트를 압축하는 것입니다.

```python
def _compress(self, context: str, max_tokens: int) -> str:
    """Compress over-limit context

    Args:
        context: Original context
        max_tokens: Maximum token limit

    Returns:
        str: Compressed context
    """
    current_tokens = self._count_tokens(context)

    if current_tokens <= max_tokens:
        return context  # No compression needed

    print(f"[ContextBuilder] Context over limit ({current_tokens} > {max_tokens}), executing compression")

    # Section compression: maintain structural integrity
    sections = context.split("\n\n")
    compressed_sections = []
    current_total = 0

    for section in sections:
        section_tokens = self._count_tokens(section)

        if current_total + section_tokens <= max_tokens:
            # Fully retain
            compressed_sections.append(section)
            current_total += section_tokens
        else:
            # Partially retain
            remaining_tokens = max_tokens - current_total
            if remaining_tokens > 50:  # Retain at least 50 tokens
                # Simple truncation (can use LLM summarization in production)
                truncated = self._truncate_text(section, remaining_tokens)
                compressed_sections.append(truncated + "\n[... Content compressed ...]")
            break

    compressed_context = "\n\n".join(compressed_sections)
    final_tokens = self._count_tokens(compressed_context)
    print(f"[ContextBuilder] Compression complete: {current_tokens} -> {final_tokens} tokens")

    return compressed_context

def _truncate_text(self, text: str, max_tokens: int) -> str:
    """Truncate text to specified token count

    Args:
        text: Original text
        max_tokens: Maximum token count

    Returns:
        str: Truncated text
    """
    # Simple implementation: estimate by character ratio
    # Should use precise tokenizer in production
    char_per_token = len(text) / self._count_tokens(text) if self._count_tokens(text) > 0 else 4
    max_chars = int(max_tokens * char_per_token)

    return text[:max_chars]

def _count_tokens(self, text: str) -> int:
    """Estimate token count of text

    Args:
        text: Text content

    Returns:
        int: Token count
    """
    # Simple estimation: Chinese 1 char ≈ 1 token, English 1 word ≈ 1.3 tokens
    # Should use actual tokenizer in production
    chinese_chars = sum(1 for ch in text if '\u4e00' <= ch <= '\u9fff')
    english_words = len([w for w in text.split() if w])

    return int(chinese_chars + english_words * 1.3)
```

압축 단계의 설계는 "구조적 무결성 유지" 원칙을 구현합니다. 토큰 예산이 빠듯한 경우에도 각 섹션의 주요 정보를 유지하려고 노력합니다.

### 9.3.4 전체 사용 예

이제 전체 예제를 통해 실제 프로젝트에서 ContextBuilder를 사용하는 방법을 살펴보겠습니다.

(1) 기본 사용법

```python
from hello_agents.context import ContextBuilder, ContextConfig
from hello_agents.tools import MemoryTool, RAGTool
from hello_agents.core.message import Message
from datetime import datetime

# 1. Initialize tools
memory_tool = MemoryTool(user_id="user123")
rag_tool = RAGTool(knowledge_base_path="./knowledge_base")

# 2. Create ContextBuilder
config = ContextConfig(
    max_tokens=3000,
    reserve_ratio=0.2,
    min_relevance=0.2,
    enable_compression=True
)

builder = ContextBuilder(
    memory_tool=memory_tool,
    rag_tool=rag_tool,
    config=config
)

# 3. Prepare conversation history
conversation_history = [
    Message(content="I'm developing a data analysis tool", role="user", timestamp=datetime.now()),
    Message(content="Great! Data analysis tools usually need to handle large amounts of data. What tech stack do you plan to use?", role="assistant", timestamp=datetime.now()),
    Message(content="I plan to use Python and Pandas, and have completed the CSV reading module", role="user", timestamp=datetime.now()),
    Message(content="Good choice! Pandas is very powerful for data processing. Next you may need to consider data cleaning and transformation.", role="assistant", timestamp=datetime.now()),
]

# 4. Add some memories
memory_tool.run({
    "action": "add",
    "content": "User is developing a data analysis tool using Python and Pandas",
    "memory_type": "semantic",
    "importance": 0.8
})

memory_tool.run({
    "action": "add",
    "content": "Completed development of CSV reading module",
    "memory_type": "episodic",
    "importance": 0.7
})

# 5. Build context
context = builder.build(
    user_query="How to optimize Pandas memory usage?",
    conversation_history=conversation_history,
    system_instructions="You are a senior Python data engineering consultant. Your answers need to: 1) Provide specific actionable advice 2) Explain technical principles 3) Provide code examples"
)

print("=" * 80)
print("Built context:")
print("=" * 80)
print(context)
print("=" * 80)
```

(2) 러닝 효과 시연

위의 코드를 실행하면 다음과 같은 구조화된 컨텍스트 출력이 표시됩니다.

```
================================================================================
Built context:
================================================================================
[Role & Policies]
You are a senior Python data engineering consultant. Your answers need to: 1) Provide specific actionable advice 2) Explain technical principles 3) Provide code examples

[Task]
How to optimize Pandas memory usage?

[Evidence]
Core strategies for Pandas memory optimization include:
1. Use appropriate data types (such as category instead of object)
2. Read large files in chunks
3. Use chunksize parameter
---
Data type optimization can significantly reduce memory usage. For example, downgrading int64 to int32 can save 50% memory.

[Context]
user: I'm developing a data analysis tool
assistant: Great! Data analysis tools usually need to handle large amounts of data. What tech stack do you plan to use?
user: I plan to use Python and Pandas, and have completed the CSV reading module
assistant: Good choice! Pandas is very powerful for data processing. Next you may need to consider data cleaning and transformation.
Memory: User is developing a data analysis tool using Python and Pandas
Memory: Completed development of CSV reading module

[Output]
Please provide accurate, evidence-based answers based on the above information.
================================================================================
```

이 구조화된 컨텍스트에는 필요한 모든 정보가 포함되어 있습니다.

- **[역할 및 정책]**: AI의 역할과 답변 요구 사항을 명확히 합니다.
- **[태스크]** : 사용자의 질문을 명확하게 표현합니다.
- **[증거]**: RAG 시스템에서 검색된 관련 지식
- **[컨텍스트]**: 대화 이력 및 관련 추억, 충분한 배경 정보 제공
- **[출력]**: LLM에게 답변 구성 방법을 안내합니다.

(3) Agent와의 통합

마지막으로 ContextBuilder를 에이전트에 통합하는 방법을 살펴보겠습니다.

```python
from hello_agents import SimpleAgent, HelloAgentsLLM, ToolRegistry
from hello_agents.context import ContextBuilder, ContextConfig
from hello_agents.tools import MemoryTool, RAGTool

class ContextAwareAgent(SimpleAgent):
    """Agent with context awareness capability"""

    def __init__(self, name: str, llm: HelloAgentsLLM, **kwargs):
        super().__init__(name=name, llm=llm, system_prompt=kwargs.get("system_prompt", ""))

        # Initialize context builder
        self.memory_tool = MemoryTool(user_id=kwargs.get("user_id", "default"))
        self.rag_tool = RAGTool(knowledge_base_path=kwargs.get("knowledge_base_path", "./kb"))

        self.context_builder = ContextBuilder(
            memory_tool=self.memory_tool,
            rag_tool=self.rag_tool,
            config=ContextConfig(max_tokens=4000)
        )

        self.conversation_history = []

    def run(self, user_input: str) -> str:
        """Run Agent, automatically build optimized context"""

        # 1. Use ContextBuilder to build optimized context
        optimized_context = self.context_builder.build(
            user_query=user_input,
            conversation_history=self.conversation_history,
            system_instructions=self.system_prompt
        )

        # 2. Call LLM with optimized context
        messages = [
            {"role": "system", "content": optimized_context},
            {"role": "user", "content": user_input}
        ]
        response = self.llm.invoke(messages)

        # 3. Update conversation history
        from hello_agents.core.message import Message
        from datetime import datetime

        self.conversation_history.append(
            Message(content=user_input, role="user", timestamp=datetime.now())
        )
        self.conversation_history.append(
            Message(content=response, role="assistant", timestamp=datetime.now())
        )

        # 4. Record important interactions to memory system
        self.memory_tool.run({
            "action": "add",
            "content": f"Q: {user_input}\nA: {response[:200]}...",  # Summary
            "memory_type": "episodic",
            "importance": 0.6
        })

        return response

# Usage example
agent = ContextAwareAgent(
    name="Data Analysis Consultant",
    llm=HelloAgentsLLM(),
    system_prompt="You are a senior Python data engineering consultant.",
    user_id="user123",
    knowledge_base_path="./data_science_kb"
)

response = agent.run("How to optimize Pandas memory usage?")
print(response)
```

이 접근 방식을 통해 ContextBuilder는 에이전트의 "컨텍스트 관리 두뇌"가 되어 정보 수집, 필터링 및 구성을 자동으로 처리하여 에이전트가 항상 최적의 컨텍스트에서 추론하고 생성할 수 있도록 합니다.

### 9.3.5 모범 사례 및 최적화 권장 사항

실제로 ContextBuilder를 적용할 때 다음 모범 사례에 주목할 가치가 있습니다.

1. **동적으로 토큰 예산 조정**: 작업 복잡성에 따라 `max_tokens`를 동적으로 조정하고, 간단한 작업에 더 작은 예산을 사용하고, 복잡한 작업에 예산을 늘립니다.

2. **관련성 계산 최적화**: 프로덕션 환경에서는 단순한 키워드 중복을 벡터 유사성 계산으로 대체하여 검색 품질을 향상시킵니다.

3. **캐싱 메커니즘**: 변경되지 않는 시스템 지침 및 지식 기반 콘텐츠의 경우 반복 계산을 피하기 위해 캐싱 메커니즘을 구현합니다.

4. **모니터링 및 로깅**: 후속 최적화를 위해 각 컨텍스트 빌드 (number of selected information, token usage rate, etc.)에 대한 통계 정보를 기록합니다.

5. **A/B 테스트**: 핵심 매개변수(예: 관련성 가중치, 최신성 가중치)에 대해 A/B 테스트를 통해 최적의 구성을 찾습니다.

## 9.4 NoteTool: 구조화된 메모

NoteTool은 "장거리 작업"을 위해 제공되는 구조화된 외부 메모리 구성 요소입니다. Markdown 파일을 전달자로 사용하며 헤더에 YAML 머리말을 사용하여 주요 정보를 기록하고 본문에 상태, 결론, 차단 항목 및 작업 항목을 기록합니다. 이 디자인은 사람의 가독성, 버전 제어 편의성, 컨텍스트에 대한 재주입 용이성을 결합하여 장거리 에이전트를 구축하는 데 중요한 도구가 됩니다.

9.4.1 디자인 철학과 응용 시나리오

구현 세부 사항을 살펴보기 전에 먼저 NoteTool의 설계 철학과 일반적인 응용 프로그램 시나리오를 이해해 보겠습니다.

(1) NoteTool이 필요한 이유는 무엇입니까?

8장에서는 강력한 메모리 관리 기능을 제공하는 MemoryTool을 소개했습니다. 그러나 MemoryTool은 주로 **대화 기억**, 즉 단기 작업 기억, 일화 기억, 의미 기억에 중점을 둡니다. 장기적인 추적과 체계적인 관리가 필요한 **프로젝트 기반 작업**에는 보다 가볍고 인간 친화적인 녹음 방법이 필요합니다.

NoteTool은 다음을 제공하여 이러한 격차를 해소합니다.

- **구조적 기록**: Markdown + YAML 형식을 사용하며 기계 구문 분석과 사람이 읽고 편집하는 데 모두 적합합니다.
- **버전 친화적**: 일반 텍스트 형식으로 Git과 같은 버전 제어 시스템을 자연스럽게 지원합니다.
- **낮은 오버헤드**: 복잡한 데이터베이스 작업이 필요하지 않으며 경량 상태 추적에 적합합니다.
- **유연한 분류**: `type` 및 `tags`을 통해 메모를 유연하게 정리하여 다차원 검색을 지원합니다.

(2) 일반적인 적용 시나리오

NoteTool은 특히 다음 시나리오에 적합합니다.

**시나리오 1: 장기 프로젝트 추적**

에이전트가 며칠 또는 몇 주가 걸릴 수 있는 대규모 코드베이스 리팩토링 작업을 지원한다고 상상해 보세요. NoteTool은 다음을 기록할 수 있습니다.

- `task_state` : 현재 단계의 작업 상태 및 진행 상황
- `conclusion`: 각 단계 종료 후 주요 결론
- `blocker`: 발생한 문제 및 차단 지점
- `action`: 다음 실행 계획

```python
# Record task status
notes.run({
    "action": "create",
    "title": "Refactoring Project - Phase 1",
    "content": "Completed refactoring of data model layer, test coverage reached 85%. Next will refactor business logic layer.",
    "note_type": "task_state",
    "tags": ["refactoring", "phase1"]
})

# Record blocker
notes.run({
    "action": "create",
    "title": "Dependency Conflict Issue",
    "content": "Found some third-party library versions incompatible, need to resolve. Impact scope: 3 modules in business logic layer.",
    "note_type": "blocker",
    "tags": ["dependency", "urgent"]
})
```

**시나리오 2: 연구 과제 관리**

문헌 검토를 수행하는 지능형 연구 조교는 NoteTool을 사용하여 다음을 기록할 수 있습니다.

- 각 논문의 핵심 관점 (`conclusion`)
- 심층적으로 조사할 주제 (`action`)
- 중요 참고자료 (`reference`)

**시나리오 3: ContextBuilder와의 협력**

각 대화 라운드 전에 에이전트는 `search` 또는 `list` 작업을 통해 관련 메모를 검색하고 이를 컨텍스트에 삽입할 수 있습니다.

```python
# In Agent's run method
def run(self, user_input: str) -> str:
    # 1. Retrieve relevant notes
    relevant_notes = self.note_tool.run({
        "action": "search",
        "query": user_input,
        "limit": 3
    })

    # 2. Convert note content to ContextPacket
    note_packets = []
    for note in relevant_notes:
        note_packets.append(ContextPacket(
            content=note['content'],
            timestamp=note['updated_at'],
            token_count=self._count_tokens(note['content']),
            relevance_score=0.7,
            metadata={"type": "note", "note_type": note['type']}
        ))

    # 3. Pass notes when building context
    context = self.context_builder.build(
        user_query=user_input,
        custom_packets=note_packets,
        ...
    )
```

### 9.4.2 저장 형식 상세 설명

NoteTool은 구조와 가독성의 균형을 맞춘 Markdown + YAML의 하이브리드 형식을 채택합니다.

(1) 노트 파일 형식

각 메모는 다음 형식의 독립적인 `.md` 파일입니다.

```markdown
---
id: note_20250119_153000_0
title: Project Progress - Phase 1
type: task_state
tags: [refactoring, phase1, backend]
created_at: 2025-01-19T15:30:00
updated_at: 2025-01-19T15:30:00
---

# Project Progress - Phase 1

## Completion Status

Completed refactoring of data model layer, main changes include:

1. Unified entity class naming conventions
2. Introduced type hints to improve code maintainability
3. Optimized database query performance

## Test Coverage

- Unit test coverage: 85%
- Integration test coverage: 70%

## Next Steps

1. Refactor business logic layer
2. Resolve dependency conflict issues
3. Increase integration test coverage to 85%
```

이 형식의 장점:

- **YAML 메타데이터**: 기계 구문 분석이 가능하며 정확한 필드 추출 및 검색을 지원합니다.
- **마크다운 본문**: 사람이 읽을 수 있고 다양한 형식을 지원합니다. (headings, lists, code blocks, etc.)
- **파일 이름을 ID로**: 관리를 단순화합니다. 각 메모의 파일 이름은 고유 식별자입니다.

(2) 인덱스 파일

NoteTool은 메모를 빠르게 검색하고 관리할 수 있도록 `notes_index.json` 파일을 유지합니다.

```json
{
  "note_20250119_153000_0": {
    "id": "note_20250119_153000_0",
    "title": "Project Progress - Phase 1",
    "type": "task_state",
    "tags": ["refactoring", "phase1", "backend"],
    "created_at": "2025-01-19T15:30:00",
    "updated_at": "2025-01-19T15:30:00",
    "file_path": "./notes/note_20250119_153000_0.md"
  }
}
```

이 인덱스 파일의 역할:

- **빠른 검색**: 각 파일을 열 필요 없이 색인에서 직접 검색합니다.
- **메타데이터 관리**: 모든 노트의 메타데이터를 중앙에서 관리
- **무결성 검사**: 누락되거나 손상된 파일을 감지할 수 있습니다.

### 9.4.3 핵심 동작 상세 설명

NoteTool은 노트의 전체 수명주기 관리를 다루는 7가지 핵심 작업을 제공합니다.

(1) 생성: 노트 생성

```python
def _create_note(
    self,
    title: str,
    content: str,
    note_type: str = "general",
    tags: Optional[List[str]] = None
) -> str:
    """Create note

    Args:
        title: Note title
        content: Note content (Markdown format)
        note_type: Note type (task_state/conclusion/blocker/action/reference/general)
        tags: Tag list

    Returns:
        str: Note ID
    """
    from datetime import datetime

    # 1. Generate unique ID
    timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
    note_id = f"note_{timestamp}_{len(self.index)}"

    # 2. Build metadata
    metadata = {
        "id": note_id,
        "title": title,
        "type": note_type,
        "tags": tags or [],
        "created_at": datetime.now().isoformat(),
        "updated_at": datetime.now().isoformat()
    }

    # 3. Build complete Markdown file content
    md_content = self._build_markdown(metadata, content)

    # 4. Save to file
    file_path = os.path.join(self.workspace, f"{note_id}.md")
    with open(file_path, 'w', encoding='utf-8') as f:
        f.write(md_content)

    # 5. Update index
    metadata["file_path"] = file_path
    self.index[note_id] = metadata
    self._save_index()

    return note_id

def _build_markdown(self, metadata: Dict, content: str) -> str:
    """Build Markdown file content (YAML + body)"""
    import yaml

    # YAML front matter
    yaml_header = yaml.dump(metadata, allow_unicode=True, sort_keys=False)

    # Combined format
    return f"---\n{yaml_header}---\n\n{content}"
```

사용 예:

```python
from hello_agents.tools import NoteTool

notes = NoteTool(workspace="./project_notes")

note_id = notes.run({
    "action": "create",
    "title": "Refactoring Project - Phase 1",
    "content": """## Completion Status
Completed refactoring of data model layer, test coverage reached 85%.

## Next Steps
Refactor business logic layer""",
    "note_type": "task_state",
    "tags": ["refactoring", "phase1"]
})

print(f"✅ Note created successfully, ID: {note_id}")
```

(2) 읽기: 메모 읽기

```python
def _read_note(self, note_id: str) -> Dict:
    """Read note content

    Args:
        note_id: Note ID

    Returns:
        Dict: Dictionary containing metadata and content
    """
    if note_id not in self.index:
        raise ValueError(f"Note does not exist: {note_id}")

    file_path = self.index[note_id]["file_path"]

    # Read file
    with open(file_path, 'r', encoding='utf-8') as f:
        raw_content = f.read()

    # Parse YAML metadata and Markdown body
    metadata, content = self._parse_markdown(raw_content)

    return {
        "metadata": metadata,
        "content": content
    }

def _parse_markdown(self, raw_content: str) -> Tuple[Dict, str]:
    """Parse Markdown file (separate YAML and body)"""
    import yaml

    # Find YAML delimiters
    parts = raw_content.split('---\n', 2)

    if len(parts) >= 3:
        # Has YAML front matter
        yaml_str = parts[1]
        content = parts[2].strip()
        metadata = yaml.safe_load(yaml_str)
    else:
        # No metadata, all as body
        metadata = {}
        content = raw_content.strip()

    return metadata, content
```

(3) 업데이트: 업데이트 노트

```python
def _update_note(
    self,
    note_id: str,
    title: Optional[str] = None,
    content: Optional[str] = None,
    note_type: Optional[str] = None,
    tags: Optional[List[str]] = None
) -> str:
    """Update note

    Args:
        note_id: Note ID
        title: New title (optional)
        content: New content (optional)
        note_type: New type (optional)
        tags: New tags (optional)

    Returns:
        str: Operation result message
    """
    if note_id not in self.index:
        raise ValueError(f"Note does not exist: {note_id}")

    # 1. Read existing note
    note = self._read_note(note_id)
    metadata = note["metadata"]
    old_content = note["content"]

    # 2. Update fields
    if title:
        metadata["title"] = title
    if note_type:
        metadata["type"] = note_type
    if tags is not None:
        metadata["tags"] = tags
    if content is not None:
        old_content = content

    # Update timestamp
    from datetime import datetime
    metadata["updated_at"] = datetime.now().isoformat()

    # 3. Rebuild and save
    md_content = self._build_markdown(metadata, old_content)
    file_path = metadata["file_path"]

    with open(file_path, 'w', encoding='utf-8') as f:
        f.write(md_content)

    # 4. Update index
    self.index[note_id] = metadata
    self._save_index()

    return f"✅ Note updated: {metadata['title']}"
```

(4) 검색 : 노트 검색

```python
def _search_notes(
    self,
    query: str,
    limit: int = 10,
    note_type: Optional[str] = None,
    tags: Optional[List[str]] = None
) -> List[Dict]:
    """Search notes

    Args:
        query: Search keyword
        limit: Return quantity limit
        note_type: Filter by type (optional)
        tags: Filter by tags (optional)

    Returns:
        List[Dict]: List of matching notes
    """
    results = []
    query_lower = query.lower()

    for note_id, metadata in self.index.items():
        # Type filter
        if note_type and metadata.get("type") != note_type:
            continue

        # Tag filter
        if tags:
            note_tags = set(metadata.get("tags", []))
            if not note_tags.intersection(tags):
                continue

        # Read note content
        try:
            note = self._read_note(note_id)
            content = note["content"]
            title = metadata.get("title", "")

            # Search in title and content
            if query_lower in title.lower() or query_lower in content.lower():
                results.append({
                    "note_id": note_id,
                    "title": title,
                    "type": metadata.get("type"),
                    "tags": metadata.get("tags", []),
                    "content": content,
                    "updated_at": metadata.get("updated_at")
                })
        except Exception as e:
            print(f"[WARNING] Failed to read note {note_id}: {e}")
            continue

    # Sort by update time
    results.sort(key=lambda x: x["updated_at"], reverse=True)

    return results[:limit]
```

(5) 목록: 목록 메모

```python
def _list_notes(
    self,
    note_type: Optional[str] = None,
    tags: Optional[List[str]] = None,
    limit: int = 20
) -> List[Dict]:
    """List notes (in reverse chronological order by update time)

    Args:
        note_type: Filter by type (optional)
        tags: Filter by tags (optional)
        limit: Return quantity limit

    Returns:
        List[Dict]: List of note metadata
    """
    results = []

    for note_id, metadata in self.index.items():
        # Type filter
        if note_type and metadata.get("type") != note_type:
            continue

        # Tag filter
        if tags:
            note_tags = set(metadata.get("tags", []))
            if not note_tags.intersection(tags):
                continue

        results.append(metadata)

    # Sort by update time
    results.sort(key=lambda x: x.get("updated_at", ""), reverse=True)

    return results[:limit]
```

(6) 요약: 참고 요약

```python
def _summary(self) -> Dict[str, Any]:
    """Generate note summary statistics

    Returns:
        Dict: Statistical information
    """
    total_count = len(self.index)

    # Count by type
    type_counts = {}
    for metadata in self.index.values():
        note_type = metadata.get("type", "general")
        type_counts[note_type] = type_counts.get(note_type, 0) + 1

    # Recently updated notes
    recent_notes = sorted(
        self.index.values(),
        key=lambda x: x.get("updated_at", ""),
        reverse=True
    )[:5]

    return {
        "total_notes": total_count,
        "type_distribution": type_counts,
        "recent_notes": [
            {
                "id": note["id"],
                "title": note.get("title", ""),
                "type": note.get("type"),
                "updated_at": note.get("updated_at")
            }
            for note in recent_notes
        ]
    }
```

(7) 삭제 : 메모 삭제

```python
def _delete_note(self, note_id: str) -> str:
    """Delete note

    Args:
        note_id: Note ID

    Returns:
        str: Operation result message
    """
    if note_id not in self.index:
        raise ValueError(f"Note does not exist: {note_id}")

    # 1. Delete file
    file_path = self.index[note_id]["file_path"]
    if os.path.exists(file_path):
        os.remove(file_path)

    # 2. Remove from index
    title = self.index[note_id].get("title", note_id)
    del self.index[note_id]
    self._save_index()

    return f"✅ Note deleted: {title}"
```

9.4.4 ContextBuilder와의 심층 통합

NoteTool의 진정한 힘은 ContextBuilder와의 결합 사용에 있습니다. 전체 사례 연구를 통해 이러한 통합을 보여드리겠습니다.

(1) 시나리오 설정

다음을 수행해야 하는 장기 프로젝트 도우미를 구축한다고 가정해 보겠습니다.

1. 프로젝트의 단계별 진행 상황을 기록합니다.
2. 보류 중인 문제 추적
3. 각 대화 중에 관련 메모를 자동으로 검토합니다.
4. 과거 기록을 바탕으로 일관된 권장 사항 제공

(2) 구현예

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.context import ContextBuilder, ContextConfig, ContextPacket
from hello_agents.tools import MemoryTool, RAGTool, NoteTool
from datetime import datetime

class ProjectAssistant(SimpleAgent):
    """Long-term project assistant, integrating NoteTool and ContextBuilder"""

    def __init__(self, name: str, project_name: str, **kwargs):
        super().__init__(name=name, llm=HelloAgentsLLM(), **kwargs)

        self.project_name = project_name

        # Initialize tools
        self.memory_tool = MemoryTool(user_id=project_name)
        self.rag_tool = RAGTool(knowledge_base_path=f"./{project_name}_kb")
        self.note_tool = NoteTool(workspace=f"./{project_name}_notes")

        # Initialize context builder
        self.context_builder = ContextBuilder(
            memory_tool=self.memory_tool,
            rag_tool=self.rag_tool,
            config=ContextConfig(max_tokens=4000)
        )

        self.conversation_history = []

    def run(self, user_input: str, note_as_action: bool = False) -> str:
        """Run assistant, automatically integrate notes"""

        # 1. Retrieve relevant notes from NoteTool
        relevant_notes = self._retrieve_relevant_notes(user_input)

        # 2. Convert notes to ContextPacket
        note_packets = self._notes_to_packets(relevant_notes)

        # 3. Build optimized context
        context = self.context_builder.build(
            user_query=user_input,
            conversation_history=self.conversation_history,
            system_instructions=self._build_system_instructions(),
            custom_packets=note_packets
        )

        # 4. Call LLM
        response = self.llm.invoke(context)

        # 5. If needed, record interaction as note
        if note_as_action:
            self._save_as_note(user_input, response)

        # 6. Update conversation history
        self._update_history(user_input, response)

        return response

    def _retrieve_relevant_notes(self, query: str, limit: int = 3) -> List[Dict]:
        """Retrieve relevant notes"""
        try:
            # Prioritize retrieving blocker and action type notes
            blockers = self.note_tool.run({
                "action": "list",
                "note_type": "blocker",
                "limit": 2
            })

            # General search
            search_results = self.note_tool.run({
                "action": "search",
                "query": query,
                "limit": limit
            })

            # Merge and deduplicate
            all_notes = {note['note_id']: note for note in blockers + search_results}
            return list(all_notes.values())[:limit]

        except Exception as e:
            print(f"[WARNING] Note retrieval failed: {e}")
            return []

    def _notes_to_packets(self, notes: List[Dict]) -> List[ContextPacket]:
        """Convert notes to context packets"""
        packets = []

        for note in notes:
            content = f"[Note: {note['title']}]\n{note['content']}"

            packets.append(ContextPacket(
                content=content,
                timestamp=datetime.fromisoformat(note['updated_at']),
                token_count=len(content) // 4,  # Simple estimation
                relevance_score=0.75,  # Notes have high relevance
                metadata={
                    "type": "note",
                    "note_type": note['type'],
                    "note_id": note['note_id']
                }
            ))

        return packets

    def _save_as_note(self, user_input: str, response: str):
        """Save interaction as note"""
        try:
            # Determine what type of note to save
            if "problem" in user_input.lower() or "blocker" in user_input.lower():
                note_type = "blocker"
            elif "plan" in user_input.lower() or "next" in user_input.lower():
                note_type = "action"
            else:
                note_type = "conclusion"

            self.note_tool.run({
                "action": "create",
                "title": f"{user_input[:30]}...",
                "content": f"## Question\n{user_input}\n\n## Analysis\n{response}",
                "note_type": note_type,
                "tags": [self.project_name, "auto_generated"]
            })

        except Exception as e:
            print(f"[WARNING] Failed to save note: {e}")

    def _build_system_instructions(self) -> str:
        """Build system instructions"""
        return f"""You are a long-term assistant for the {self.project_name} project.

Your responsibilities:
1. Provide coherent recommendations based on historical notes
2. Track project progress and pending issues
3. Reference relevant historical notes when answering
4. Provide specific, actionable next-step recommendations

Notes:
- Prioritize issues marked as blockers
- Indicate source of basis in recommendations (notes, memory, or knowledge base)
- Maintain awareness of overall project progress"""

    def _update_history(self, user_input: str, response: str):
        """Update conversation history"""
        from hello_agents.core.message import Message

        self.conversation_history.append(
            Message(content=user_input, role="user", timestamp=datetime.now())
        )
        self.conversation_history.append(
            Message(content=response, role="assistant", timestamp=datetime.now())
        )

        # Limit history length
        if len(self.conversation_history) > 10:
            self.conversation_history = self.conversation_history[-10:]

# Usage example
assistant = ProjectAssistant(
    name="Project Assistant",
    project_name="data_pipeline_refactoring"
)

# First interaction: Record project status
response = assistant.run(
    "We have completed refactoring of the data model layer, test coverage reached 85%. Next plan is to refactor the business logic layer.",
    note_as_action=True
)

# Second interaction: Raise issue
response = assistant.run(
    "When refactoring the business logic layer, I encountered dependency version conflict issues. How should I resolve this?"
)

# View note summary
summary = assistant.note_tool.run({"action": "summary"})
print(summary)
```

(3) 러닝 효과 시연

```bash
[ContextBuilder] Collected 8 candidate information packages
[ContextBuilder] Selected 7 information packages, total 3500 tokens

✅ Assistant answer:

I noticed this issue was mentioned in your previously recorded notes. According to the note [Refactoring Project - Phase 1], your current test coverage has reached 85%, which is a good foundation.

Regarding the dependency version conflict issue, I recommend:

1. **Use virtual environment isolation**: Create an independent virtual environment for the business logic layer to avoid dependency conflicts with other modules
2. **Lock versions**: Explicitly specify exact versions of all dependencies in requirements.txt
3. **Use pipdeptree**: Analyze the dependency tree to find the root cause of conflicts

I will mark this issue as a blocker and recommend prioritizing its resolution.

[Source: Note note_20250119_153000_0, Project knowledge base]

---

📋 Note summary:
{
  "total_notes": 2,
  "type_distribution": {
    "action": 1,
    "blocker": 1
  },
  "recent_notes": [
    {
      "id": "note_20250119_154500_1",
      "title": "When refactoring the business logic layer, I encountered dependency version conflict issues...",
      "type": "blocker",
      "updated_at": "2025-01-19T15:45:00"
    },
    {
      "id": "note_20250119_153000_0",
      "title": "We have completed refactoring of the data model layer...",
      "type": "action",
      "updated_at": "2025-01-19T15:30:00"
    }
  ]
}
```

9.4.5 모범 사례

실제로 NoteTool을 사용할 때 다음 모범 사례를 따르면 더욱 강력한 장거리 에이전트를 구축하는 데 도움이 될 수 있습니다.

1. **합리적인 지폐 분류**:
   - `task_state`: 단계별 진행상황 및 현황을 기록
   - `conclusion`: 중요한 결론과 결과를 기록합니다.
   - `blocker`: 녹화 차단 문제, 최우선 순위
   - `action`: 다음 실행 계획 기록
   - `reference` : 중요 참고자료 기록

2. **정기적인 정리 및 보관**:
   - 해결된 방해 요소의 경우 결론을 업데이트하세요.
   - 오래된 작업의 경우 즉시 삭제하거나 업데이트하세요.
   - 버전 관리를 위해 `["v1.0", "completed"]` 등의 태그를 사용하세요.

3. **ContextBuilder와의 협력**:
   - 각 대화 라운드 전에 관련 메모를 검색합니다.
   - 노트 유형에 따라 다른 관련성 점수 설정(차단기 > 조치 > 결론)
   - 컨텍스트 과부하를 피하기 위해 메모 수를 제한하십시오.

4. **인간-기계 협업**:
   - 노트는 사람이 읽을 수 있는 마크다운 형식으로 되어 있어 수동 편집이 가능합니다.
   - 버전 관리에 Git을 사용하여 노트 진화 추적
   - 주요 단계에서 상담사가 생성한 메모를 수동으로 검토합니다.

5. **자동화된 작업 흐름**:
   - 정기적으로 노트 요약 보고서 생성
   - 메모를 기반으로 프로젝트 진행 문서를 자동으로 생성
   - 노트 내용을 다른 시스템(예: Notion, Confluence)과 동기화

## 9.5 TerminalTool: 즉각적인 파일 시스템 액세스

이전 장에서는 각각 대화형 메모리와 지식 검색 기능을 제공하는 MemoryTool과 RAGTool을 소개했습니다. 그러나 많은 실제 시나리오에서 에이전트는 로그 파일 보기, 코드베이스 구조 분석, 구성 파일 검색 등 **파일 시스템에 대한 즉각적인 액세스 및 탐색**이 필요합니다. 이것이 바로 TerminalTool이 필요한 곳입니다.

TerminalTool은 에이전트에 **보안 명령줄 실행 기능**을 제공하여 공통 파일 시스템 및 텍스트 처리 명령을 지원하는 동시에 다계층 보안 메커니즘을 통해 시스템 보안을 보장합니다. 이 설계는 섹션 9.2.2에 언급된 "JIT(Just-in-time) 컨텍스트" 개념을 구현합니다. 즉, 에이전트는 모든 파일을 미리 로드할 필요가 없지만 요청 시 탐색하고 검색할 수 있습니다.

9.5.1 디자인 철학과 보안 메커니즘

(1) TerminalTool이 필요한 이유는 무엇입니까?

장거리 에이전트를 구축할 때 종종 다음과 같은 시나리오가 발생합니다.

**시나리오 1: 코드베이스 탐색**

개발 보조자는 사용자가 대규모 코드베이스의 구조를 이해하도록 도와야 합니다.

```python
# Traditional approach: Pre-index all files (high cost, may be outdated)
rag_tool.add_document("./project/**/*.py")  # Time-consuming, occupies large storage

# TerminalTool approach: Instant exploration
terminal.run({"command": "find . -name '*.py' -type f"})  # Fast, real-time
terminal.run({"command": "grep -r 'class UserService' ."})  # Precise location
terminal.run({"command": "head -n 50 src/services/user.py"})  # View on demand
```

**시나리오 2: 로그 파일 분석**

운영 보조자는 애플리케이션 로그를 분석해야 합니다.

```python
# Check log file size
terminal.run({"command": "ls -lh /var/log/app.log"})

# View latest error logs
terminal.run({"command": "tail -n 100 /var/log/app.log | grep ERROR"})

# Count error type distribution
terminal.run({"command": "grep ERROR /var/log/app.log | cut -d':' -f3 | sort | uniq -c"})
```

**시나리오 3: 데이터 파일 미리보기**

데이터 분석 도우미는 데이터 파일의 구조를 신속하게 이해해야 합니다.

```python
# View first few lines of CSV file
terminal.run({"command": "head -n 5 data/sales.csv"})

# Count lines
terminal.run({"command": "wc -l data/*.csv"})

# View column names
terminal.run({"command": "head -n 1 data/sales.csv | tr ',' '\n'"})
```

이러한 시나리오의 일반적인 특징은 **사전 인덱싱 및 벡터화 대신 실시간 경량 파일 시스템 액세스가 필요**하다는 것입니다. TerminalTool은 이러한 "탐색적" 작업 흐름을 위해 정확하게 설계되었습니다.

(2) 보안 메커니즘 상세 설명

에이전트가 명령을 실행하도록 허용하는 것은 강력하지만 위험한 기능입니다. TerminalTool은 다층 보안 메커니즘을 통해 시스템 보안을 보장합니다.

**첫 번째 레이어: 명령 화이트리스트**

안전한 읽기 전용 명령만 허용하고 시스템을 수정할 수 있는 모든 작업을 완전히 금지합니다.

```python
ALLOWED_COMMANDS = {
    # File listing and information
    'ls', 'dir', 'tree',
    # File content viewing
    'cat', 'head', 'tail', 'less', 'more',
    # File search
    'find', 'grep', 'egrep', 'fgrep',
    # Text processing
    'wc', 'sort', 'uniq', 'cut', 'awk', 'sed',
    # Directory operations
    'pwd', 'cd',
    # File information
    'file', 'stat', 'du', 'df',
    # Others
    'echo', 'which', 'whereis',
}
```

에이전트가 화이트리스트 외부의 명령을 실행하려고 시도하면 즉시 거부됩니다.

```python
terminal.run({"command": "rm -rf /"})
# ❌ Command not allowed: rm
# Allowed commands: cat, cd, cut, dir, du, ...
```

**두 번째 계층: 작업 디렉터리 제한(샌드박스)**

TerminalTool은 지정된 작업 디렉터리와 해당 하위 디렉터리에만 액세스할 수 있으며 시스템의 다른 부분에는 액세스할 수 없습니다.

```python
# Specify working directory during initialization
terminal = TerminalTool(workspace="./project")

# Allowed: Access files within working directory
terminal.run({"command": "cat ./src/main.py"})  # ✅

# Prohibited: Access files outside working directory
terminal.run({"command": "cat /etc/passwd"})  # ❌ Not allowed to access paths outside working directory

# Prohibited: Escape through ..
terminal.run({"command": "cd ../../../etc"})  # ❌ Not allowed to access paths outside working directory
```

이 샌드박스 메커니즘은 에이전트의 동작이 비정상이더라도 시스템의 다른 부분에 영향을 미칠 수 없도록 보장합니다.

**세 번째 레이어: 시간 초과 제어**

각 명령에는 무한 루프 또는 리소스 고갈을 방지하기 위해 실행 시간 제한이 있습니다.

```python
terminal = TerminalTool(
    workspace="./project",
    timeout=30  # 30 second timeout
)

# If command execution exceeds 30 seconds
terminal.run({"command": "find / -name '*.log'"})
# ❌ Command execution timeout (exceeded 30 seconds)
```

**네 번째 레이어: 출력 크기 제한**

메모리 오버플로를 방지하려면 명령 출력의 크기를 제한하십시오.

```python
terminal = TerminalTool(
    workspace="./project",
    max_output_size=10 * 1024 * 1024  # 10MB
)

# If output exceeds 10MB
terminal.run({"command": "cat huge_file.log"})
# ... (first 10MB of content) ...
# ⚠️ Output truncated (exceeded 10485760 bytes)
```

이러한 4개 계층의 보안 메커니즘을 통해 TerminalTool은 시스템 보안을 극대화하는 동시에 강력한 기능을 제공합니다.

### 9.5.2 핵심 기능 상세 설명

TerminalTool의 구현은 명령 실행과 디렉터리 탐색이라는 두 가지 핵심 기능에 중점을 둡니다.

(1) 명령 실행

핵심 `_execute_command` 메소드는 실제로 명령을 실행하는 역할을 담당합니다.

```python
def _execute_command(self, command: str) -> str:
    """Execute command"""
    try:
        # Execute command in current directory
        result = subprocess.run(
            command,
            shell=True,
            cwd=str(self.current_dir),  # Execute in current working directory
            capture_output=True,
            text=True,
            timeout=self.timeout,
            env=os.environ.copy()
        )

        # Merge standard output and standard error
        output = result.stdout
        if result.stderr:
            output += f"\n[stderr]\n{result.stderr}"

        # Check output size
        if len(output) > self.max_output_size:
            output = output[:self.max_output_size]
            output += f"\n\n⚠️ Output truncated (exceeded {self.max_output_size} bytes)"

        # Add return code information
        if result.returncode != 0:
            output = f"⚠️ Command return code: {result.returncode}\n\n{output}"

        return output if output else "✅ Command executed successfully (no output)"

    except subprocess.TimeoutExpired:
        return f"❌ Command execution timeout (exceeded {self.timeout} seconds)"
    except Exception as e:
        return f"❌ Command execution failed: {e}"
```

이 구현의 핵심 사항은 다음과 같습니다.

- **현재 디렉터리 인식**: `cwd` 매개변수를 사용하여 올바른 디렉터리에서 명령을 실행합니다.
- **오류 처리**: 표준 오류 캡처 및 병합, 완전한 진단 정보 제공
- **반환 코드 확인**: 0이 아닌 반환 코드는 경고로 표시됩니다.
- **내결함성 설계**: 시간 초과 및 예외가 올바르게 처리되어 에이전트 충돌이 발생하지 않습니다.

(2) 디렉토리 탐색

`cd` 명령의 특수 처리는 파일 시스템에서 에이전트 탐색을 지원합니다.

```python
def _handle_cd(self, parts: List[str]) -> str:
    """Handle cd command"""
    if not self.allow_cd:
        return "❌ cd command is disabled"

    if len(parts) < 2:
        # cd without parameters, return current directory
        return f"Current directory: {self.current_dir}"

    target_dir = parts[1]

    # Handle relative path
    if target_dir == "..":
        new_dir = self.current_dir.parent
    elif target_dir == ".":
        new_dir = self.current_dir
    elif target_dir == "~":
        new_dir = self.workspace
    else:
        new_dir = (self.current_dir / target_dir).resolve()

    # Check if within working directory
    try:
        new_dir.relative_to(self.workspace)
    except ValueError:
        return f"❌ Not allowed to access paths outside working directory: {new_dir}"

    # Check if directory exists
    if not new_dir.exists():
        return f"❌ Directory does not exist: {new_dir}"

    if not new_dir.is_dir():
        return f"❌ Not a directory: {new_dir}"

    # Update current directory
    self.current_dir = new_dir
    return f"✅ Switched to directory: {self.current_dir}"
```

이 설계는 다단계 파일 시스템 탐색에서 에이전트를 지원합니다.

```python
# Step 1: View project structure
terminal.run({"command": "ls -la"})

# Step 2: Enter source code directory
terminal.run({"command": "cd src"})

# Step 3: Find specific files
terminal.run({"command": "find . -name '*service*.py'"})

# Step 4: View file content
terminal.run({"command": "cat user_service.py"})
```

### 9.5.3 일반적인 사용 패턴

TerminalTool은 다양한 공통 파일 시스템 작동 패턴을 지원합니다.

(1) 탐색항법

에이전트는 인간 개발자처럼 코드베이스를 단계별로 탐색할 수 있습니다.

```python
from hello_agents.tools import TerminalTool

terminal = TerminalTool(workspace="./my_project")

# Step 1: View project root directory
print(terminal.run({"command": "ls -la"}))
"""
total 24
drwxr-xr-x  6 user  staff   192 Jan 19 16:00 .
drwxr-xr-x  5 user  staff   160 Jan 19 15:30 ..
-rw-r--r--  1 user  staff  1234 Jan 19 15:30 README.md
drwxr-xr-x  4 user  staff   128 Jan 19 15:30 src
drwxr-xr-x  3 user  staff    96 Jan 19 15:30 tests
-rw-r--r--  1 user  staff   456 Jan 19 15:30 requirements.txt
"""

# Step 2: View source code directory structure
terminal.run({"command": "cd src"})
print(terminal.run({"command": "tree"}))

# Step 3: Search for specific patterns
print(terminal.run({"command": "grep -r 'def process' ."}))
```

(2) 데이터 파일 분석

데이터 파일의 구조와 내용을 빠르게 이해합니다.

```python
terminal = TerminalTool(workspace="./data")

# View first few lines of CSV file
print(terminal.run({"command": "head -n 5 sales_2024.csv"}))
"""
date,product,quantity,revenue
2024-01-01,Widget A,150,4500.00
2024-01-01,Widget B,200,8000.00
2024-01-02,Widget A,180,5400.00
2024-01-02,Widget C,120,3600.00
"""

# Count total lines
print(terminal.run({"command": "wc -l *.csv"}))
"""
  10234 sales_2024.csv
   8567 sales_2023.csv
  18801 total
"""

# Extract and count product categories
print(terminal.run({"command": "tail -n +2 sales_2024.csv | cut -d',' -f2 | sort | uniq -c"}))
"""
  3456 Widget A
  4123 Widget B
  2655 Widget C
"""
```

(3) 로그 파일 분석

애플리케이션 로그를 실시간으로 분석하여 문제를 신속하게 찾습니다.

```python
terminal = TerminalTool(workspace="/var/log")

# View latest error logs
print(terminal.run({"command": "tail -n 50 app.log | grep ERROR"}))

# Count error type distribution
print(terminal.run({"command": "grep ERROR app.log | awk '{print $4}' | sort | uniq -c | sort -rn"}))
"""
  245 DatabaseConnectionError
  123 TimeoutException
   67 ValidationError
   34 AuthenticationError
"""

# Find logs for specific time period
print(terminal.run({"command": "grep '2024-01-19 15:' app.log | tail -n 20"}))
```

(4) 코드베이스 분석

코드 검토 및 이해 지원:

```python
terminal = TerminalTool(workspace="./codebase")

# Count lines of code
print(terminal.run({"command": "find . -name '*.py' -exec wc -l {} + | tail -n 1"}))

# Find all TODO comments
print(terminal.run({"command": "grep -rn 'TODO' --include='*.py'"}))

# Find definition of specific function
print(terminal.run({"command": "grep -rn 'def process_data' --include='*.py'"}))

# View function implementation
print(terminal.run({"command": "sed -n '/def process_data/,/^def /p' src/processor.py | head -n -1"}))
```

9.5.4 다른 도구와의 협업

TerminalTool의 진정한 힘은 MemoryTool, NoteTool 및 ContextBuilder와의 공동 사용에 있습니다.

(1) MemoryTool과의 협업

TerminalTool에서 발견한 정보는 메모리 시스템에 저장될 수 있습니다.

```python
# Use TerminalTool to discover project structure
structure = terminal.run({"command": "tree -L 2 src"})

# Store in semantic memory
memory_tool.run({
    "action": "add",
    "content": f"Project structure:\n{structure}",
    "memory_type": "semantic",
    "importance": 0.8,
    "metadata": {"type": "project_structure"}
})
```

(2) NoteTool과의 협업

중요한 발견은 구조화된 메모로 기록될 수 있습니다.

```python
# Discover a performance bottleneck
log_analysis = terminal.run({"command": "grep 'slow query' app.log | tail -n 10"})

# Record as blocker note
note_tool.run({
    "action": "create",
    "title": "Database Slow Query Issue",
    "content": f"## Problem Description\nFound multiple slow queries affecting system performance\n\n## Log Analysis\n```\n{log_analysis}\n```\n\n## Next Steps\n1. Analyze slow query SQL\n2. Add indexes\n3. Optimize query logic",
    "note_type": "blocker",
    "tags": ["performance", "database"]
})
```

(3) ContextBuilder와의 협업

TerminalTool 출력은 컨텍스트의 일부일 수 있습니다.

```python
# Explore codebase
code_structure = terminal.run({"command": "ls -R src"})
recent_changes = terminal.run({"command": "git log --oneline -10"})

# Convert to ContextPacket
from hello_agents.context import ContextPacket
from datetime import datetime

packets = [
    ContextPacket(
        content=f"Codebase structure:\n{code_structure}",
        timestamp=datetime.now(),
        token_count=len(code_structure) // 4,
        relevance_score=0.7,
        metadata={"type": "code_structure", "source": "terminal"}
    ),
    ContextPacket(
        content=f"Recent commits:\n{recent_changes}",
        timestamp=datetime.now(),
        token_count=len(recent_changes) // 4,
        relevance_score=0.8,
        metadata={"type": "git_history", "source": "terminal"}
    )
]

# Include this information when building context
context = context_builder.build(
    user_query="How to refactor the user service module?",
    custom_packets=packets
)
```

## 9.6 실제 Long-Horizon 에이전트: 코드베이스 유지 관리 도우미

이제 ContextBuilder, NoteTool 및 TerminalTool을 통합하여 완전한 장거리 에이전트인 **Codebase Maintenance Assistant**를 구축해 보겠습니다. 이 도우미는 다음을 수행할 수 있습니다.

1. 코드베이스 구조 탐색 및 이해
2. 발견된 문제점 및 개선점 기록
3. 장기 리팩터링 작업 추적
4. 컨텍스트 창 제한 하에서 일관성 유지

9.6.1 시나리오 설정 및 요구사항 분석

**비즈니스 시나리오**

중간 크기의 Python 웹 애플리케이션을 유지관리하고 있다고 가정해 보겠습니다. 이 코드베이스에는 Flask 프레임워크로 구축된 약 50개의 Python 파일이 포함되어 있으며 데이터 모델, 비즈니스 로직, API 인터페이스 및 기타 모듈을 다루며 점진적으로 정리해야 할 기술적 부채도 있습니다. 이 시나리오에서는 코드베이스를 탐색하고 프로젝트 구조, 종속성 및 코드 스타일을 이해하는 데 도움을 주는 지능형 보조자가 필요합니다. 코드 중복, 과도한 복잡성, 테스트 부족 등과 같은 코드 문제를 식별합니다. 작업 진행 상황을 추적하고, 해야 할 일 항목, 완료된 작업 및 발생한 방해 요소를 기록합니다. 역사적 맥락을 기반으로 일관된 리팩토링 권장 사항을 제공합니다.

**도전과제 및 솔루션**

이 시나리오는 몇 가지 일반적인 장거리 작업 문제에 직면해 있습니다. 첫 번째는 컨텍스트 창을 초과하는 정보의 문제입니다. 전체 코드베이스에는 수만 줄의 코드가 포함될 수 있으므로 컨텍스트 창에 한꺼번에 배치할 수 없습니다. 즉각적인 주문형 코드 탐색을 위해 TerminalTool을 사용하고 필요할 때만 특정 파일을 확인함으로써 이 문제를 해결합니다. 두 번째는 교차 세션 상태 관리 과제입니다. 리팩토링 작업은 며칠 동안 지속될 수 있으며 여러 세션에서 진행 상황을 유지해야 합니다. 우리는 NoteTool을 사용하여 단계별 진행 상황, 할 일 항목 및 주요 결정을 기록함으로써 이 문제를 해결합니다. 마지막으로, 맥락의 질과 관련성의 문제가 있습니다. 각 대화는 관련된 과거 정보를 검토해야 하지만 관련 없는 정보로 인해 압도되어서는 안 됩니다. 우리는 ContextBuilder를 사용하여 컨텍스트를 지능적으로 필터링하고 구성하여 높은 신호 밀도를 보장합니다.

### 9.6.2 시스템 아키텍처 설계

우리의 코드베이스 유지 관리 도우미는 그림 9.3과 같이 3계층 아키텍처를 채택합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/9-figures/9-3.png" alt="" width="85%"/>
<p>그림 9.3 코드베이스 유지 관리 도우미의 3계층 아키텍처</p>
</div>

9.6.3 핵심 구현

이제 이 시스템의 핵심 클래스를 구현해 보겠습니다.

```python
from typing import Dict, Any, List, Optional
from datetime import datetime
import json

from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.context import ContextBuilder, ContextConfig, ContextPacket
from hello_agents.tools import MemoryTool, NoteTool, TerminalTool
from hello_agents.core.message import Message


class CodebaseMaintainer:
    """Codebase Maintenance Assistant - Long-horizon agent example

    Integrates ContextBuilder + NoteTool + TerminalTool + MemoryTool
    Implements cross-session codebase maintenance task management
    """

    def __init__(
        self,
        project_name: str,
        codebase_path: str,
        llm: Optional[HelloAgentsLLM] = None
    ):
        self.project_name = project_name
        self.codebase_path = codebase_path
        self.session_id = f"session_{datetime.now().strftime('%Y%m%d_%H%M%S')}"

        # Initialize LLM
        self.llm = llm or HelloAgentsLLM()

        # Initialize tools
        self.memory_tool = MemoryTool(user_id=project_name)
        self.note_tool = NoteTool(workspace=f"./{project_name}_notes")
        self.terminal_tool = TerminalTool(workspace=codebase_path, timeout=60)

        # Initialize context builder
        self.context_builder = ContextBuilder(
            memory_tool=self.memory_tool,
            rag_tool=None,  # This case does not use RAG
            config=ContextConfig(
                max_tokens=4000,
                reserve_ratio=0.15,
                min_relevance=0.2,
                enable_compression=True
            )
        )

        # Conversation history
        self.conversation_history: List[Message] = []

        # Statistics
        self.stats = {
            "session_start": datetime.now(),
            "commands_executed": 0,
            "notes_created": 0,
            "issues_found": 0
        }

        print(f"✅ Codebase maintenance assistant initialized: {project_name}")
        print(f"📁 Working directory: {codebase_path}")
        print(f"🆔 Session ID: {self.session_id}")

    def run(self, user_input: str, mode: str = "auto") -> str:
        """Run assistant

        Args:
            user_input: User input
            mode: Running mode
                - "auto": Automatically decide whether to use tools
                - "explore": Focus on code exploration
                - "analyze": Focus on problem analysis
                - "plan": Focus on task planning

        Returns:
            str: Assistant's answer
        """
        print(f"\n{'='*80}")
        print(f"👤 User: {user_input}")
        print(f"{'='*80}\n")

        # Step 1: Execute preprocessing based on mode
        pre_context = self._preprocess_by_mode(user_input, mode)

        # Step 2: Retrieve relevant notes
        relevant_notes = self._retrieve_relevant_notes(user_input)
        note_packets = self._notes_to_packets(relevant_notes)

        # Step 3: Build optimized context
        context = self.context_builder.build(
            user_query=user_input,
            conversation_history=self.conversation_history,
            system_instructions=self._build_system_instructions(mode),
            custom_packets=note_packets + pre_context
        )

        # Step 4: Call LLM
        print("🤖 Thinking...")
        response = self.llm.invoke(context)

        # Step 5: Post-processing
        self._postprocess_response(user_input, response)

        # Step 6: Update conversation history
        self._update_history(user_input, response)

        print(f"\n🤖 Assistant: {response}\n")
        print(f"{'='*80}\n")

        return response

    def _preprocess_by_mode(
        self,
        user_input: str,
        mode: str
    ) -> List[ContextPacket]:
        """Execute preprocessing based on mode, collect relevant information"""
        packets = []

        if mode == "explore" or mode == "auto":
            # Explore mode: Automatically view project structure
            print("🔍 Exploring codebase structure...")

            structure = self.terminal_tool.run({"command": "find . -type f -name '*.py' | head -n 20"})
            self.stats["commands_executed"] += 1

            packets.append(ContextPacket(
                content=f"[Codebase Structure]\n{structure}",
                timestamp=datetime.now(),
                token_count=len(structure) // 4,
                relevance_score=0.6,
                metadata={"type": "code_structure", "source": "terminal"}
            ))

        if mode == "analyze":
            # Analyze mode: Check code complexity and issues
            print("📊 Analyzing code quality...")

            # Count lines of code
            loc = self.terminal_tool.run({"command": "find . -name '*.py' -exec wc -l {} + | tail -n 1"})

            # Find TODO and FIXME
            todos = self.terminal_tool.run({"command": "grep -rn 'TODO\\|FIXME' --include='*.py' | head -n 10"})

            self.stats["commands_executed"] += 2

            packets.append(ContextPacket(
                content=f"[Code Statistics]\n{loc}\n\n[To-Do Items]\n{todos}",
                timestamp=datetime.now(),
                token_count=(len(loc) + len(todos)) // 4,
                relevance_score=0.7,
                metadata={"type": "code_analysis", "source": "terminal"}
            ))

        if mode == "plan":
            # Planning mode: Load recent notes
            print("📋 Loading task planning...")

            task_notes = self.note_tool.run({
                "action": "list",
                "note_type": "task_state",
                "limit": 3
            })

            if task_notes:
                content = "\n".join([f"- {note['title']}" for note in task_notes])
                packets.append(ContextPacket(
                    content=f"[Current Tasks]\n{content}",
                    timestamp=datetime.now(),
                    token_count=len(content) // 4,
                    relevance_score=0.8,
                    metadata={"type": "task_plan", "source": "notes"}
                ))

        return packets

    def _retrieve_relevant_notes(self, query: str, limit: int = 3) -> List[Dict]:
        """Retrieve relevant notes"""
        try:
            # Prioritize retrieving blockers
            blockers = self.note_tool.run({
                "action": "list",
                "note_type": "blocker",
                "limit": 2
            })

            # Search relevant notes
            search_results = self.note_tool.run({
                "action": "search",
                "query": query,
                "limit": limit
            })

            # Merge and deduplicate
            all_notes = {note.get('note_id') or note.get('id'): note for note in (blockers or []) + (search_results or [])}
            return list(all_notes.values())[:limit]

        except Exception as e:
            print(f"[WARNING] Note retrieval failed: {e}")
            return []

    def _notes_to_packets(self, notes: List[Dict]) -> List[ContextPacket]:
        """Convert notes to context packets"""
        packets = []

        for note in notes:
            # Set different relevance scores based on note type
            relevance_map = {
                "blocker": 0.9,
                "action": 0.8,
                "task_state": 0.75,
                "conclusion": 0.7
            }

            note_type = note.get('type', 'general')
            relevance = relevance_map.get(note_type, 0.6)

            content = f"[Note: {note.get('title', 'Untitled')}]\nType: {note_type}\n\n{note.get('content', '')}"

            packets.append(ContextPacket(
                content=content,
                timestamp=datetime.fromisoformat(note.get('updated_at', datetime.now().isoformat())),
                token_count=len(content) // 4,
                relevance_score=relevance,
                metadata={
                    "type": "note",
                    "note_type": note_type,
                    "note_id": note.get('note_id') or note.get('id')
                }
            ))

        return packets

    def _build_system_instructions(self, mode: str) -> str:
        """Build system instructions"""
        base_instructions = f"""You are the codebase maintenance assistant for the {self.project_name} project.

Your core capabilities:
1. Use TerminalTool to explore codebase (ls, cat, grep, find, etc.)
2. Use NoteTool to record discoveries and tasks
3. Provide coherent recommendations based on historical notes

Current session ID: {self.session_id}
"""

        mode_specific = {
            "explore": """
Current mode: Explore codebase

You should:
- Actively use terminal commands to understand code structure
- Identify key modules and files
- Record project architecture in notes
""",
            "analyze": """
Current mode: Analyze code quality

You should:
- Find code issues (duplication, complexity, TODOs, etc.)
- Evaluate code quality
- Record discovered issues as blocker or action notes
""",
            "plan": """
Current mode: Task planning

You should:
- Review historical notes and tasks
- Formulate next action plan
- Update task status notes
""",
            "auto": """
Current mode: Auto decision

You should:
- Flexibly choose strategies based on user needs
- Use tools when needed
- Maintain professionalism and practicality in responses
"""
        }

        return base_instructions + mode_specific.get(mode, mode_specific["auto"])

    def _postprocess_response(self, user_input: str, response: str):
        """Post-processing: Analyze response, automatically record important information"""

        # If issues found, automatically create blocker note
        if any(keyword in response.lower() for keyword in ["issue", "bug", "error", "blocker", "problem"]):
            try:
                self.note_tool.run({
                    "action": "create",
                    "title": f"Issue found: {user_input[:30]}...",
                    "content": f"## User Input\n{user_input}\n\n## Issue Analysis\n{response[:500]}...",
                    "note_type": "blocker",
                    "tags": [self.project_name, "auto_detected", self.session_id]
                })
                self.stats["notes_created"] += 1
                self.stats["issues_found"] += 1
                print("📝 Automatically created issue note")
            except Exception as e:
                print(f"[WARNING] Failed to create note: {e}")

        # If task planning, automatically create action note
        elif any(keyword in user_input.lower() for keyword in ["plan", "next", "task", "todo"]):
            try:
                self.note_tool.run({
                    "action": "create",
                    "title": f"Task planning: {user_input[:30]}...",
                    "content": f"## Discussion\n{user_input}\n\n## Action Plan\n{response[:500]}...",
                    "note_type": "action",
                    "tags": [self.project_name, "planning", self.session_id]
                })
                self.stats["notes_created"] += 1
                print("📝 Automatically created action plan note")
            except Exception as e:
                print(f"[WARNING] Failed to create note: {e}")

    def _update_history(self, user_input: str, response: str):
        """Update conversation history"""
        self.conversation_history.append(
            Message(content=user_input, role="user", timestamp=datetime.now())
        )
        self.conversation_history.append(
            Message(content=response, role="assistant", timestamp=datetime.now())
        )

        # Limit history length (keep recent 10 rounds of conversation)
        if len(self.conversation_history) > 20:
            self.conversation_history = self.conversation_history[-20:]

    # === Convenience methods ===

    def explore(self, target: str = ".") -> str:
        """Explore codebase"""
        return self.run(f"Please explore the code structure of {target}", mode="explore")

    def analyze(self, focus: str = "") -> str:
        """Analyze code quality"""
        query = f"Please analyze code quality" + (f", focusing on {focus}" if focus else "")
        return self.run(query, mode="analyze")

    def plan_next_steps(self) -> str:
        """Plan next steps"""
        return self.run("Based on current progress, plan next steps", mode="plan")

    def execute_command(self, command: str) -> str:
        """Execute terminal command"""
        result = self.terminal_tool.run({"command": command})
        self.stats["commands_executed"] += 1
        return result

    def create_note(
        self,
        title: str,
        content: str,
        note_type: str = "general",
        tags: List[str] = None
    ) -> str:
        """Create note"""
        result = self.note_tool.run({
            "action": "create",
            "title": title,
            "content": content,
            "note_type": note_type,
            "tags": tags or [self.project_name]
        })
        self.stats["notes_created"] += 1
        return result

    def get_stats(self) -> Dict[str, Any]:
        """Get statistics"""
        duration = (datetime.now() - self.stats["session_start"]).total_seconds()

        # Get note summary
        try:
            note_summary = self.note_tool.run({"action": "summary"})
        except:
            note_summary = {}

        return {
            "session_info": {
                "session_id": self.session_id,
                "project": self.project_name,
                "duration_seconds": duration
            },
            "activity": {
                "commands_executed": self.stats["commands_executed"],
                "notes_created": self.stats["notes_created"],
                "issues_found": self.stats["issues_found"]
            },
            "notes": note_summary
        }

    def generate_report(self, save_to_file: bool = True) -> Dict[str, Any]:
        """Generate session report"""
        report = self.get_stats()

        if save_to_file:
            report_file = f"maintainer_report_{self.session_id}.json"
            with open(report_file, 'w', encoding='utf-8') as f:
                json.dump(report, f, ensure_ascii=False, indent=2, default=str)
            report["report_file"] = report_file
            print(f"📄 Report saved: {report_file}")

        return report
```

### 9.6.4 전체 사용 예

이제 전체 사용 시나리오를 통해 이 장거리 에이전트의 워크플로를 보여드리겠습니다.

```python
# ========== Initialize Assistant ==========

from hello_agents import HelloAgentsLLM

maintainer = CodebaseMaintainer(
    project_name="my_flask_app",
    codebase_path="./my_flask_app",
    llm=HelloAgentsLLM()
)


# ========== Day 1: Explore Codebase ==========
# 1. Initial exploration
response = maintainer.explore()
"""
🔍 Exploring codebase structure...
🤖 Thinking...

🤖 Assistant: I have scanned the codebase, this is a Flask web application, mainly containing the following modules:

📁 Project structure:
- app/
  ├── models/      # Data models (User, Product, Order, etc.)
  ├── routes/      # API routes
  ├── services/    # Business logic
  └── utils/       # Utility functions
- tests/           # Test files
- migrations/      # Database migrations
- config.py        # Configuration file

Total approximately 3,500 lines of Python code.

Recommended next steps:
1. Review core model definitions
2. Check routes and API design
3. Evaluate test coverage
"""

# 2. Deep dive into a specific module
response = maintainer.run("Please analyze the data model design in the app/models/ directory")
"""
🤖 Assistant: I reviewed the app/models/ directory and found the following data models:

1. User (user.py)
   - Fields: id, username, email, password_hash, created_at
   - Relationships: One-to-many Order
   - Issue: ⚠️ Missing index, email field not set with unique constraint

2. Product (product.py)
   - Fields: id, name, description, price, stock
   - Issue: ✅ Well designed

3. Order (order.py)
   - Fields: id, user_id, product_id, quantity, status
   - Issue: ⚠️ Missing creation time field, not conducive to data analysis

Recommendations:
1. Add unique constraint to User.email
2. Add created_at and updated_at fields to Order
3. Consider adding soft delete support

I have recorded these issues as blocker notes.
"""
# [Automatically created blocker note]


# ========== Day 2: Analyze Code Quality ==========

# 1. Overall quality analysis
response = maintainer.analyze()
"""
📊 Analyzing code quality...
🤖 Thinking...

🤖 Assistant: Code quality analysis report:

📊 Code statistics:
- Total lines: 3,542 lines
- Average file length: approximately 150 lines
- Largest file: services/order_service.py (456 lines)

⚠️ Issues found:

1. Code duplication (HIGH):
   - user_service.py and product_service.py have similar CRUD logic
   - Recommendation: Extract base class BaseService

2. Excessive complexity (MEDIUM):
   - order_service.py::process_order method has 8 levels of nesting
   - Recommendation: Split into multiple small methods

3. Lack of tests (HIGH):
   - Test coverage only 45%
   - services/ directory has no tests at all

4. TODO/FIXME (12 items):
   - Most are "optimize later" type comments
   - Recommendation: Clean up or convert to formal tasks

I have recorded these issues as notes and marked priorities.
"""
# [Automatically created multiple blocker notes]

# 2. View specific issue
response = maintainer.run("Please review the process_order method in order_service.py and provide refactoring recommendations")
"""
🤖 Assistant: I reviewed the code of the process_order method:

【Original Code】
def process_order(self, order_id):
    order = Order.query.get(order_id)
    if order:
        if order.status == 'pending':
            product = Product.query.get(order.product_id)
            if product:
                if product.stock >= order.quantity:
                    # ... 8 levels of nesting continue ...

【Problem Analysis】
1. Deep nesting leads to poor readability
2. Lack of error handling
3. Business logic mixed with data access

【Refactoring Recommendations】
def process_order(self, order_id):
    # 1. Early return pattern, reduce nesting
    order = self._get_order(order_id)
    if not order or order.status != 'pending':
        return False

    # 2. Extract methods, single responsibility
    if not self._check_stock(order):
        return False

    # 3. Use transactions to ensure consistency
    with db.transaction():
        self._update_stock(order)
        self._update_order_status(order, 'processing')

    return True

【Next Steps】
1. I have created a refactoring plan note
2. Recommend writing test cases first
3. Refactor gradually, keeping functionality unchanged
"""
# [Automatically created action note]


# ========== Day 3: Plan Refactoring Tasks ==========

# 1. Review progress
response = maintainer.plan_next_steps()
"""
📋 Loading task planning...
🤖 Thinking...

🤖 Assistant: Based on the exploration and analysis from the past two days, I have organized refactoring task priorities:

🔴 High priority (complete this week):
1. [blocker] Add unique constraint to User.email
   - Risk: Low, just database migration
   - Effort: 1 hour

2. [blocker] Extract BaseService to reduce code duplication
   - Risk: Medium, affects multiple service classes
   - Effort: Half a day

3. [blocker] Refactor process_order method
   - Risk: High, core business logic
   - Effort: 1 day (including writing tests)

🟡 Medium priority (next week):
4. Add unit tests for services/
5. Clean up TODO comments
6. Add time fields to Order

🟢 Low priority (TBD):
7. Performance optimization
8. Documentation updates

Recommendations:
- This week, first resolve data model issues (tasks 1, 6)
- Next week, conduct code refactoring (tasks 2, 3)
- Continuously increase test coverage (task 4)

I have updated task status notes, you can check progress anytime.
"""

# 2. Manually create detailed refactoring plan
maintainer.create_note(
    title="Weekly Refactoring Plan - Week 1",
    content="""## Objectives
Complete optimization of data model layer

## Task Checklist
- [ ] Add unique constraint to User.email
- [ ] Add created_at, updated_at fields to Order
- [ ] Write database migration scripts
- [ ] Update related test cases

## Schedule
- Monday: Design migration scripts
- Tuesday-Wednesday: Execute migration and test
- Thursday: Update test cases
- Friday: Code Review

## Risks
- Database migration may affect production environment, needs to be executed during off-peak hours
- Existing data may have duplicate emails, need to clean up first
""",
    note_type="task_state",
    tags=["refactoring", "week1", "high_priority"]
)

print("✅ Created detailed refactoring plan")


# ========== One Week Later: Check Progress ==========

# View note summary
summary = maintainer.note_tool.run({"action": "summary"})
print("📊 Note summary:")
print(json.dumps(summary, indent=2, ensure_ascii=False))
"""
{
  "total_notes": 8,
  "type_distribution": {
    "blocker": 3,
    "action": 2,
    "task_state": 2,
    "conclusion": 1
  },
  "recent_notes": [
    {
      "id": "note_20250119_160000_7",
      "title": "Weekly Refactoring Plan - Week 1",
      "type": "task_state",
      "updated_at": "2025-01-19T16:00:00"
    },
    ...
  ]
}
"""

# Generate complete report
report = maintainer.generate_report()
print("\n📄 Session report:")
print(json.dumps(report, indent=2, ensure_ascii=False))
"""
{
  "session_info": {
    "session_id": "session_20250119_150000",
    "project": "my_flask_app",
    "duration_seconds": 172800  # 2 days
  },
  "activity": {
    "commands_executed": 24,
    "notes_created": 8,
    "issues_found": 3
  },
  "notes": { ... }
}
"""
```

9.6.5 실행 효과 분석

이 완전한 사례 연구를 통해 우리는 장거리 에이전트의 몇 가지 주요 특성을 확인할 수 있습니다. 첫 번째는 세션 간 일관성입니다. 에이전트는 NoteTool을 통해 여러 날과 세션에 걸쳐 작업 일관성을 유지합니다. 첫째 날에 탐색된 문제는 둘째 날 분석에서 자동으로 고려되고, 셋째 날 계획은 이전 이틀 동안의 모든 발견을 종합할 수 있으며, 일주일 후 확인할 때 전체 기록이 보존됩니다. 두 번째는 지능형 컨텍스트 관리입니다. ContextBuilder는 각 대화에 대한 고품질 컨텍스트를 보장하고, 관련 메모(특히 차단기 유형)를 자동으로 수집하고, 대화 모드에 따라 전처리 전략을 동적으로 조정하고, 토큰 예산 내에서 가장 관련성이 높은 정보를 선택합니다.

세 번째 특징은 즉각적인 파일 시스템 액세스입니다. TerminalTool은 전체 코드베이스를 미리 인덱싱할 필요 없이 유연한 코드 탐색을 지원하고, 특정 파일 콘텐츠를 즉시 볼 수 있으며, 복잡한 텍스트 처리((grep, awk, etc.))를 지원합니다. 넷째는 자동화된 지식 관리입니다. 시스템은 발견된 지식을 자동으로 관리하고, 문제가 발견되면 차단 메모를 자동으로 생성하고, 계획을 논의할 때 조치 메모를 자동으로 생성하고, 주요 정보를 메모리 시스템에 자동으로 저장합니다. 마지막으로 인간-기계 협업입니다. 이 시스템은 유연한 인간-기계 협업 모드를 지원합니다. 이 모드에서는 에이전트가 자동으로 탐색 및 분석을 완료하고, 인간이 개입하여 메모 시스템을 통해 안내할 수 있으며, 세부 계획 메모를 수동으로 생성할 수 있습니다.

RAGTool을 통합하여 의미 검색과 결합된 코드베이스용 벡터 인덱스를 구축하고, 특수 탐색기, 분석기 및 플래너로 분할하여 다중 에이전트 협업을 구현하고, 테스트 도구를 통합하여 리팩토링 결과를 자동으로 확인하고, TerminalTool을 통해 git 명령을 실행하여 코드 변경 사항을 추적하거나, Gradio/Streamlit을 사용하여 시각적 인터페이스를 구축하는 등 이 기본 프레임워크를 더욱 확장할 수 있습니다.

## 9.7 장 요약

이 장에서 우리는 컨텍스트 엔지니어링의 이론적 기초와 엔지니어링 실무를 깊이 탐구했습니다.

### 이론적인 수준

1. **컨텍스트 엔지니어링의 본질**: "프롬프트 엔지니어링"에서 "컨텍스트 엔지니어링"으로의 진화, 핵심은 제한된 관심 예산을 관리하는 것입니다.
2. **컨텍스트 부패**: 긴 컨텍스트로 인한 성능 저하 이해, 컨텍스트를 희소 자원으로 인식
3. **세 가지 주요 전략**: 압축, 구조화된 메모 작성, 하위 에이전트 아키텍처

### 엔지니어링 실습

1. **ContextBuilder**: GSSC 파이프라인 구현, 통합 컨텍스트 관리 인터페이스 제공
2. **NoteTool**: Markdown+YAML의 하이브리드 형식, 구조화된 장기 메모리 지원
3. **TerminalTool**: 보안 명령줄 도구, 즉각적인 파일 시스템 액세스 지원
4. **Long-Horizon Agent**: 세 가지 주요 도구를 통합하고 세션 간 코드베이스 유지 관리 도우미를 구축합니다.

### 핵심 시사점

- **계층형 디자인**: 즉각적인 액세스(TerminalTool) + 세션 메모리(MemoryTool) + 영구 메모(NoteTool)
- **지능형 필터링**: 관련성과 최근성을 기반으로 한 점수 매기기 메커니즘
- **보안 우선**: 다계층 보안 메커니즘으로 시스템 안정성 보장
- **인간-기계 협업**: 자동화와 제어 가능성 사이의 균형

이 장의 학습을 통해 컨텍스트 엔지니어링의 핵심 기술을 숙달했을 뿐만 아니라 더 중요하게는 장기간에 걸쳐 일관성과 효율성을 유지할 수 있는 에이전트 시스템을 구축하는 방법을 이해했습니다. 이러한 기술은 프로덕션 수준 에이전트 애플리케이션을 구축하는 데 중요한 기반이 됩니다.

다음 장에서는 에이전트 통신 프로토콜을 살펴보고 에이전트가 외부 세계와 보다 광범위하게 상호 작용할 수 있도록 하는 방법을 알아봅니다.

## 연습

> **참고**: 일부 연습에는 표준 답변이 없습니다. 맥락공학과 장기적 과제관리에 대한 학습자의 종합적인 이해와 실무능력을 함양하는데 중점을 두고 있다.

1. 이 장에서는 컨텍스트 엔지니어링과 프롬프트 엔지니어링의 차이점을 소개했습니다. 분석해 주십시오:

- 섹션 9.1에서는 "컨텍스트는 한계 수익이 감소하는 제한된 리소스로 간주되어야 합니다"라고 언급했습니다. "컨텍스트 부패" 현상이 무엇인지 설명해 주세요. 모델이 100K 또는 심지어 200K 컨텍스트 창을 지원하는 경우에도 여전히 컨텍스트를 신중하게 관리해야 하는 이유는 무엇입니까?
   - 50개의 파일이 포함된 코드베이스를 분석해야 하는 "코드 검토 도우미"를 구축한다고 가정해 보겠습니다. 두 가지 전략을 비교해 보십시오. (1) 모든 파일 콘텐츠를 한 번에 컨텍스트에 로드합니다. (2) JIT(Just-in-time) 컨텍스트를 사용하여 도구를 통해 요청 시 파일을 검색합니다. 각각의 장점, 단점 및 적용 가능한 시나리오를 분석합니다.
   - 섹션 9.2.1에서는 시스템 프롬프트의 두 가지 극단적인 함정, 즉 "과도한 하드코딩"과 "너무 모호함"을 언급했습니다. 각각의 실제 사례를 제시하고 올바른 균형을 찾는 방법을 설명해주세요.

2. GSSC(Gather-Select-Structure-Compress) 파이프라인은 이 장의 핵심 기술입니다. 깊이 생각해주세요:

> **참고**: 이것은 실습 연습 문제이므로 실제 작동을 권장합니다.

- 섹션 9.3의 ContextBuilder 구현에서 네 단계는 각각 서로 다른 책임을 갖습니다. 분석하십시오: 특정 단계가 실패하는 경우(예: 관련 없는 정보를 선택하는 선택 단계 또는 정보 손실로 이어지는 압축 단계의 과도한 압축), 최종 에이전트 성능에 어떤 영향을 미치나요?
   - 섹션 9.3.4의 코드를 기반으로 ContextBuilder에 "컨텍스트 품질 평가" 기능을 추가합니다. 각 컨텍스트 빌드 후 컨텍스트의 정보 밀도, 관련성 및 완전성을 자동으로 평가하고 최적화 제안을 제공합니다.
   - GSSC 파이프라인의 "압축" 단계에서는 지능형 요약을 위해 LLM을 사용합니다. 생각해 보십시오. 어떤 상황에서 LLM 요약보다 단순한 절단 또는 슬라이딩 윈도우 전략이 더 적합할까요? 여러 압축 방법의 장점을 결합한 하이브리드 압축 전략을 설계합니다.

3. NoteTool 및 TerminalTool은 장거리 작업을 지원하는 핵심 도구입니다. 섹션 9.4 및 9.5에 따라 다음 확장 사례를 완료하십시오.

> **참고**: 이것은 실습 연습 문제이므로 실제 작동을 권장합니다.

- NoteTool은 계층적 노트 시스템(프로젝트 노트, 작업 노트, 임시 노트)을 사용합니다. "자동 노트 정리" 메커니즘을 설계해 주세요. 임시 노트가 특정 개수까지 쌓이면 상담원이 자동으로 이러한 노트를 분석하고, 중요한 정보를 작업 노트나 프로젝트 노트로 승격하고, 중복된 내용을 정리할 수 있습니다.
   - TerminalTool은 파일 시스템 운영 기능을 제공하지만 9.5.2절에서는 보안 설계를 강조합니다. 분석해 보십시오. 현재 보안 메커니즘(경로 유효성 검사, 명령 허용 목록, 권한 확인)이 충분한가요? 에이전트가 민감한 파일에 액세스하거나 위험한 작업을 실행해야 하는 경우 "인간-기계 공동 승인" 프로세스를 어떻게 설계해야 합니까?
   - NoteTool과 TerminalTool을 결합하여 "지능형 코드 리팩토링 도우미" 설계: 코드베이스 구조를 분석하고, 리팩토링 계획을 기록하고, 리팩토링 작업을 단계별로 실행하고, 진행 상황과 발생한 문제를 노트에서 추적할 수 있습니다. 완전한 작업 흐름 다이어그램을 그려주세요.

4. 섹션 9.6의 "장거리 작업 관리" 사례에서 우리는 실제 응용 분야에서 컨텍스트 엔지니어링의 가치를 확인했습니다. 심층적으로 분석해 보세요.

- 케이스는 "계층화된 컨텍스트 관리" 전략을 사용합니다: 즉각적인 액세스(TerminalTool) + 세션 메모리(MemoryTool) + 영구 메모(NoteTool). 분석해 보십시오. 이 세 가지 계층이 어떻게 조화를 이루어야 합니까? 어떤 정보가 어느 레이어에 배치되어야 할까요? 정보 중복과 불일치를 방지하는 방법은 무엇입니까?
   - 작업 실행 중에 중단이 발생한다고 가정하면(예: 시스템 충돌, 네트워크 연결 끊김) 에이전트는 메모에서 상태를 복구하고 실행을 계속해야 합니다. "중단점에서 재개" 메커니즘을 설계하십시오. 노트에 충분한 상태 정보를 기록하는 방법은 무엇입니까? 복구된 상태가 올바른지 확인하는 방법은 무엇입니까?
   - 장거리 작업에는 여러 하위 작업의 병렬 또는 직렬 실행이 포함되는 경우가 많습니다. "작업 종속성 관리" 시스템을 설계하십시오. 작업 간 종속 관계(예: "작업 A가 완료된 후 작업 B를 실행해야 합니다")를 표현하고 작업 실행 순서를 자동으로 예약할 수 있습니다. 이 시스템은 NoteTool과 어떻게 통합되어야 합니까?

5. 본 장에서는 '점진적 공개' 개념이 반복적으로 언급되었습니다. 생각해 보십시오:

- 섹션 9.2.2에서 점진적 공개는 "각 상호작용 단계가 새로운 맥락을 생성하고 이는 차례로 다음 결정을 안내합니다"라고 설명됩니다. 점진적인 공개가 상담원이 작업을 보다 효율적으로 완료하는 데 어떻게 도움이 되는지 보여주는 특정 애플리케이션 시나리오(예: 학술 논문 작성, 복잡한 문제 디버깅)를 설계하십시오.
   - 점진적 공개의 잠재적 위험은 "비효율적인 탐색"입니다. 에이전트는 중요하지 않은 세부 사항에 시간을 낭비하거나 주요 정보를 놓칠 수 있습니다. '탐색 안내' 메커니즘을 설계하세요. 휴리스틱 규칙이나 메타인지 전략을 통해 에이전트가 '다음에 탐색할 항목'에 대해 더 현명한 결정을 내릴 수 있도록 도와주세요.
   - "점진적 공개"를 전통적인 "모든 컨텍스트를 한 번에 로드"와 비교: 어떤 유형의 작업에서 전자가 확실한 이점을 가지고 있습니까? 어떤 유형의 작업에 후자가 더 적합할까요? 다양한 유형의 작업에 대한 예를 3개 이상 제공하세요.

## 참고자료

[1] 인류. AI 에이전트를 위한 효과적인 컨텍스트 엔지니어링. `https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents`

[2] 데이비드 김. 컨텍스트 엔지니어링(GitHub). `https://github.com/davidkimai/Context-Engineering`

