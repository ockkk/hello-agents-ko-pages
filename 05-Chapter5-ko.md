# 5장: 로우 코드 플랫폼으로 에이전트 구축

이전 장에서는 Python 코드를 작성하여 ReAct, Plan-and-Solve 및 Reflection을 포함한 다양한 클래식 에이전트 워크플로를 처음부터 구현했습니다. 이 프로세스는 우리에게 탄탄한 기술적 기반을 마련했으며 에이전트의 내부 메커니즘에 대한 깊은 이해를 제공했습니다. 그러나 빠르게 발전하는 분야의 경우 순수 코드 개발이 항상 가장 효율적인 선택은 아닙니다. 특히 아이디어를 신속하게 검증해야 하거나 전문 개발자가 아닌 개발자가 구축에 참여하려는 시나리오에서는 더욱 그렇습니다.

## 5.1 플랫폼 기반 구축의 부상

기술이 발전함에 따라 점점 더 많은 기능이 "플랫폼화"되는 것을 볼 수 있습니다. 웹 사이트 개발이 HTML/CSS/JS를 직접 작성하는 것에서 WordPress 및 Wix와 같은 웹 사이트 구축 플랫폼을 사용하는 것으로 발전한 것처럼 에이전트 구성도 플랫폼화의 물결을 가져왔습니다. 이 장에서는 그래픽, 모듈식 로우 코드 플랫폼을 사용하여 에이전트 애플리케이션을 빠르고 직관적으로 구축, 디버깅 및 배포하는 방법에 중점을 두고 초점을 "구현 세부 사항"에서 "비즈니스 로직"으로 전환합니다.

5.1.1 로우코드 플랫폼이 필요한 이유

딥러닝에서는 "바퀴를 재발명하는 것"이 ​​중요하지만, 엔지니어링 효율성과 혁신을 추구하는 실무에서는 거인의 어깨 위에 서야 하는 경우가 많습니다. 4장에서 `ReActAgent` 및 `PlanAndSolveAgent`과 같은 재사용 가능한 클래스를 캡슐화했지만 비즈니스 로직이 복잡해지면 순수 코드의 유지 관리 비용과 개발 주기가 급격히 증가합니다. 로우코드 플랫폼의 등장은 바로 이러한 문제점을 해결하기 위한 것입니다.

그들의 핵심 가치는 주로 다음 측면에 반영됩니다.

1. **기술 장벽 낮추기**: 로우 코드 플랫폼은 복잡한 기술 세부 사항(예: API 호출, 상태 관리, 동시성 제어)을 이해하기 쉬운 "노드" 또는 "모듈"로 캡슐화합니다. 사용자는 프로그래밍에 능숙할 필요가 없습니다. 강력한 작업 흐름을 구축하려면 이러한 노드를 끌어서 연결하기만 하면 됩니다. 이를 통해 제품 관리자, 디자이너, 비즈니스 전문가 등 비기술 인력도 에이전트 설계 및 제작에 참여할 수 있어 혁신의 범위가 크게 확대됩니다.
2. **개발 효율성 향상**: 전문 개발자에게 플랫폼은 효율성을 크게 향상시킬 수도 있습니다. 프로젝트 초기 단계에서 아이디어를 신속하게 검증해야 하거나 프로토타입을 구축해야 할 때 로우 코드 플랫폼을 사용하면 원래 코딩에 며칠이 걸리던 작업을 몇 시간, 심지어 몇 분 만에 완료할 수 있습니다. 개발자는 낮은 수준의 엔지니어링 구현보다는 비즈니스 로직 구성과 신속한 엔지니어링 최적화에 더 많은 에너지를 투자할 수 있습니다.
3. **더 나은 시각화 및 관찰 가능성 제공**: 터미널에서 로그를 인쇄하는 것과 비교하여 그래픽 플랫폼은 자연스럽게 에이전트 실행 궤적의 엔드투엔드 시각화를 제공합니다. 각 노드 간에 데이터가 어떻게 흐르는지, 어떤 링크가 가장 오래 걸리는지, 어떤 도구 호출이 실패하는지 명확하게 확인할 수 있습니다. 이러한 직관적인 디버깅 경험은 순수한 코드 개발과 비교할 수 없습니다.
4. **표준화 및 모범 사례 축적**: 우수한 로우 코드 플랫폼에는 일반적으로 많은 업계 모범 사례가 내장되어 있습니다. 예를 들어 사전 설정된 ReAct 템플릿, 최적화된 지식 기반 검색 엔진, 표준화된 도구 통합 사양 등을 제공합니다. 이는 개발자가 "지뢰를 밟는" 것을 방지할 뿐만 아니라 모든 사람이 동일한 표준 및 구성 요소 세트를 기반으로 개발하기 때문에 팀 협업을 더욱 원활하게 만듭니다.

간단히 말해서, 로우코드 플랫폼은 코드를 대체하기 위한 것이 아니라 더 높은 수준의 추상화를 제공하기 위한 것입니다. 이를 통해 우리는 지루한 하위 수준 구현에서 벗어나 에이전트의 "생각"과 "행동" 자체의 논리에 더 집중할 수 있으므로 아이디어를 더 빠르고 더 나은 현실로 만들 수 있습니다.

### 5.1.2 로우코드 플랫폼 선택

현재 에이전트 및 LLM 애플리케이션을 위한 로우 코드 플랫폼 시장은 각 플랫폼마다 고유한 포지셔닝과 장점을 갖고 있어 번성하고 있습니다. 선택할 플랫폼은 핵심 요구 사항, 기술 배경 및 프로젝트의 최종 목표에 따라 결정되는 경우가 많습니다. 이 장의 다음 내용에서는 Coze, Dify, FastGPT, n8n 등 여러 대표적인 플랫폼을 소개하고 실습하는 데 중점을 둘 것입니다. 그 전에, 그들에 대해 간략하게 소개하겠습니다.

**코즈**

- **코어 포지셔닝**: ByteDance에서 출시한 Coze<sup>[1]</sup>은 제로 코드/로우 코드 에이전트 구축 경험에 중점을 두어 프로그래밍 배경 지식이 없는 사용자도 쉽게 생성할 수 있습니다.
- **기능 분석**: Coze는 매우 친숙한 시각적 인터페이스를 갖추고 있습니다. 사용자는 LEGO 블록을 구축하는 것과 마찬가지로 플러그인을 끌어다 놓고, 지식 기반을 구성하고, 워크플로를 설정하여 에이전트를 만들 수 있습니다. 매우 풍부한 플러그인 라이브러리가 내장되어 있으며 Douyin, Feishu 및 WeChat 공식 계정과 같은 주류 플랫폼에 대한 원클릭 게시를 지원하여 배포 프로세스를 크게 단순화합니다.
- **대상**: 아이디어를 빠르게 인터랙티브 제품으로 전환하려는 AI 애플리케이션 초급 사용자, 제품 관리자, 운영 담당자 및 개인 창작자입니다.

**디파이**

- **코어 포지셔닝**: Dify는 개발자에게 프로토타입 제작부터 생산 배포까지 원스톱 솔루션을 제공하는 것을 목표로 하는 모든 기능을 갖춘 오픈 소스 LLM 애플리케이션 개발 및 운영 플랫폼<sup>[2]</sup>입니다.
- **기능 분석**: 백엔드 서비스와 모델 운영의 개념을 통합하여 에이전트 워크플로, RAG 파이프라인, 데이터 주석 및 미세 조정과 같은 다양한 기능을 지원합니다. 전문성, 안정성, 확장성을 추구하는 엔터프라이즈급 애플리케이션을 위해 Dify는 탄탄한 기반을 제공합니다.
- **대상**: 기술적 배경을 어느 정도 갖춘 개발자, 확장 가능한 엔터프라이즈급 AI 애플리케이션을 구축해야 하는 팀.

**빠른GPT**

- **핵심 포지셔닝**: FastGPT는 오픈 소스 LLM 기반 지식 기반 Q&A 플랫폼이자 에이전트 구축 도구<sup>[3]</sup>로, 사용하기 쉬운 RAG(검색 증강 생성) 솔루션과 시각적 워크플로 조정 기능을 제공하는 데 중점을 둡니다.
- **기능 분석**: FastGPT의 핵심 장점은 지식 기반 Q&A 시나리오에 대한 최고의 최적화에 있습니다. 데이터 가져오기, 자동 텍스트 청킹, 벡터화된 저장에서 지능형 검색에 이르기까지 완전한 RAG 파이프라인을 제공하고 직관적인 시각적 인터페이스(흐름 모듈)를 통해 복잡한 대화 흐름 및 에이전트 워크플로 조정을 지원합니다. 이 플랫폼은 모델 중립적 설계를 채택하여 OpenAI, Claude 및 Tongyi Qianwen과 같은 다양한 국내 및 국제 대형 모델에 유연하게 연결하는 동시에 WeChat Work, DingTalk 및 Feishu와 같은 기존 시스템과 빠르게 통합할 수 있는 포괄적인 API 인터페이스와 플러그인 시장을 제공합니다.
- **대상**: 지능형 고객 서비스, 내부 지식 도우미 또는 개인 지식 기반을 기반으로 한 문서 Q&A 로봇을 신속하게 구축하려는 개발자 및 중소기업 팀은 물론 구현 장벽을 낮추고 싶은 RAG에 관심이 있는 기술 매니아.

**n8n**

- **핵심 포지셔닝**: n8n은 본질적으로 순수한 LLM 플랫폼이 아닌 오픈 소스 작업 흐름 자동화 도구<sup>[4]</sup>입니다. 최근에는 AI 기능을 적극적으로 통합했습니다.

- **기능 분석**: n8n의 강점은 '연결'에 있습니다. 다양한 SaaS 서비스, 데이터베이스 및 API를 복잡하고 자동화된 비즈니스 프로세스에 쉽게 연결할 수 있는 수백 개의 사전 설정된 노드가 있습니다. 이 프로세스에 LLM 노드를 삽입하여 전체 자동화 체인의 일부로 만들 수 있습니다. 처음 두 개만큼 LLM 기능에 특화되어 있지는 않지만 일반적인 자동화 기능은 독특합니다. 그러나 학습 곡선도 상대적으로 가파르다.

- **대상**: AI 기능을 기존 비즈니스 프로세스에 심층적으로 통합하고 고도로 맞춤화된 자동화를 달성해야 하는 개발자 및 기업.

다음 하위 섹션에서는 이러한 플랫폼을 하나씩 직접 경험하고 실제 운영을 통해 각각의 매력을 보다 직관적으로 느낄 것입니다.

## 5.2 플랫폼 1: 코즈
Coze는 정말 멋진 AI 에이전트 생성 도구입니다! 또한 현재 시장에서 가장 널리 사용되는 에이전트 플랫폼이기도 합니다. 직관적인 시각적 인터페이스와 풍부한 기능 모듈을 통해 플랫폼을 통해 사용자는 채팅이 가능한 챗봇, 자동으로 스토리를 작성하는 크리에이티브 머신, 심지어 스토리를 영화 MV로 직접 변환하는 데 도움을 주는 다양한 유형의 에이전트 애플리케이션을 쉽게 만들 수 있습니다! 하이라이트 중 하나는 강력한 생태계 통합 기능입니다. 개발된 에이전트는 클릭 한 번으로 WeChat, Feishu, Doubao와 같은 주류 플랫폼에 게시할 수 있어 원활한 크로스 플랫폼 배포가 가능합니다. 기업 사용자를 위해 Coze는 에이전트 기능을 기존 비즈니스 시스템에 통합하여 "빌딩 블록 스타일" AI 애플리케이션 구축을 달성하는 유연한 API 인터페이스도 제공합니다.
### 5.2.1 코즈의 기능 모듈
(1) 플랫폼 인터페이스 개요

전체 레이아웃 소개: 최근 Coze는 그림 5.1과 같이 UI 인터페이스를 다시 업데이트했습니다. 이제 가장 왼쪽 사이드 바는 핵심 프로젝트 개발, 리소스 라이브러리, 효과 평가 및 공간 구성을 포함하는 Coze 플랫폼 홈페이지의 개발 작업 공간입니다. 아래 영역은 원클릭 복사를 위한 공식 템플릿, Coze의 가장 큰 장점인 풍부하고 다양한 플러그인 스토어, 눈부신 배열을 갖춘 최대 에이전트 커뮤니티, API 테스트를 위한 API 관리, 기업을 위한 자세한 튜토리얼 문서 및 일반 관리를 포함하여 Coze 개발을 위한 지원 자료 공간입니다. 오른쪽에는 4개의 템플릿이 있습니다. 상단에는 Coze의 최신 업데이트 공지가 있으며, Coze의 최신 진행 상황을 알려주므로 최신 도구와 기능에 대해 알아볼 수 있습니다. 그 아래에는 초보자 튜토리얼이 있습니다. 이를 클릭하면 초보자 튜토리얼 문서를 찾을 수 있으며 몇 분 안에 에이전트 구축을 시작할 수 있습니다. 다음은 귀하의 팔로우와 상담원 권장사항입니다. 여기에서 좋아하는 AI 개발자를 팔로우하고 에이전트를 즐겨찾기에 추가하여 사용할 수도 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-01.png" alt="Image description" width="90%"/>
<p>그림 5.1 Coze 에이전트 플랫폼의 전체 도식</p>
</div>

(2) 핵심 기능 소개

먼저 왼쪽 사이드바의 더하기 기호를 클릭하여 에이전트 생성을 위한 진입점을 확인합니다. 현재 AI 애플리케이션에는 두 가지 유형이 있습니다. 하나는 에이전트를 생성하는 것이고 다른 하나는 애플리케이션이라고 합니다. 그 중 에이전트는 단일 에이전트 자율 계획 모드, 단일 에이전트 대화 흐름 모드 및 다중 에이전트 모드로 구분됩니다. AI 애플리케이션도 두 가지 유형으로 나뉜다. 그림 5.2와 같이 데스크톱과 웹용 사용자 인터페이스를 디자인할 수 있을 뿐만 아니라 미니 프로그램과 H5용 인터페이스도 쉽게 구축할 수 있다.
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-02.png" alt="Image description" width="90%"/>
<p>그림 5.2 Coze Agent 생성 항목</p>
</div>
프로젝트 공간은 귀하가 개발했거나 복사한 모든 에이전트 또는 애플리케이션이 저장되는 에이전트 저장소입니다. 그림 5.3에서 볼 수 있듯이 Coze에서 Agent를 개발할 때 가장 자주 방문하게 되는 곳이기도 합니다.
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-03.png" alt="Image description" width="90%"/>
<p>그림 5.3 Coze Agent 프로젝트 공간</p>
</div>
리소스 라이브러리는 Coze 에이전트 개발을 위한 핵심 무기고입니다. 리소스 라이브러리에는 에이전트 개발을 위한 워크플로, 기술 자료, 카드, 프롬프트 라이브러리 및 기타 일련의 도구가 저장되어 있습니다. 어떤 종류의 에이전트를 만들 수 있는지는 먼저 모델의 능력에 따라 다르지만, 가장 중요한 것은 에이전트에 "장비와 스킬"을 어떻게 갖추느냐에 달려 있습니다. 모델은 에이전트의 하한을 결정하지만 Coze 리소스 라이브러리는 에이전트의 능력에 대해 무한한 상한을 제공하므로 그림 5.4와 같이 자신의 아이디어, 상상력 및 창의성에 따라 개발할 수 있습니다.
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-04.png" alt="Image description" width="90%"/>
<p>그림 5.4 Coze 에이전트 리소스 라이브러리</p>
</div>
공간 구성에는 그림 5.5와 같이 에이전트, 플러그인, 워크플로 및 게시 채널을 위한 통합 관리 채널과 호출하는 다양한 대형 모델을 볼 수 있는 모델 관리가 포함됩니다.
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-05.png" alt="Image description" width="90%"/>
<p>그림 5.5 Coze Agent 게시 채널</p>
</div>
Coze의 에이전트 개발을 간단하게 요약한다면 이를 게임의 다양한 구성 요소에 비유하겠습니다. 훌륭한 에이전트를 만들기 위한 각 부분의 조합은 마치 "게임"을 하는 것과 매우 흡사합니다. 에이전트를 완성할 때마다 보스를 물리치고 '경험치'든 '장비'든 많은 것을 얻는 것과 같습니다.

- 작업 흐름: 레벨 클리어 루트 맵
- 대화 흐름 : NPC 대화 정리
- 플러그인: 캐릭터 스킬 카드
- 지식 기반: 게임 백과사전
- 카드: 퀵아이템바
- 프롬프트: 캐릭터 이동 키
- 데이터베이스: "클라우드 저장"
- 출판관리 : 레벨리뷰어
- 모델 관리: 게임 캐릭터 라이브러리 또는 캐릭터 생성 시스템
- 효과 평가 : 레벨 채점 시스템




### 5.2.2 "Daily AI Brief" 도우미 구축

**사례 설명:** 이 실제 사례는 Coze 플랫폼의 플러그인 통합 기능을 심층적으로 분석하고 독자가 처음부터 강력한 "Daily AI Brief" 에이전트를 구축하도록 안내하는 것을 목표로 합니다. 이 에이전트는 여러 정보 소스(36Kr, Huxiu, IT Home, InfoQ, GitHub, arXiv 포함)에서 최신 AI 분야 헤드라인, 학술 논문 및 오픈 소스 프로젝트 업데이트를 자동으로 캡처하여 체계적이고 전문적인 방식으로 생생하고 간결한 브리핑으로 통합할 수 있습니다.

이 사례를 통해 귀하는 다음과 같은 핵심 기술을 체계적으로 습득하게 됩니다.

* **다중 소스 정보 집계:** Coze의 플러그인 에코시스템을 사용하여 크로스 플랫폼, 크로스 유형 데이터 흐름의 원활한 통합을 달성합니다.
  * **에이전트 행동 정의:** 역할 설정 및 신속한 엔지니어링을 통해 에이전트의 작업 실행 및 콘텐츠 생성을 정밀하게 제어하여 출력이 미리 설정된 전문 표준을 충족하도록 합니다.
  * **자동화된 워크플로 구성:** 데이터 수집, 콘텐츠 처리, 형식화된 출력 등 여러 단계를 효율적이고 자동화된 워크플로에 연결하는 방법을 알아보세요.



**1단계: 정보 소스 플러그인 추가 및 구성**

"Daily AI Brief" 에이전트를 구축하는 주요 작업은 이를 풍부하고 권위 있는 정보 소스에 연결하는 것입니다. Coze 플랫폼에서는 해당 플러그인을 추가하고 구성하여 이를 달성합니다.

1. **플러그인 통합:** Coze의 플러그인 라이브러리에서 필요한 플러그인을 검색하여 추가하세요. 예를 들어 **RSS** 플러그인 (as shown in Figure 5.6)을 통해 미디어 플랫폼의 RSS 피드를 구독하고, **GitHub** 플러그인 (as shown in Figure 5.7)을 통해 오픈 소스 프로젝트를 추적하고, **arXiv** 플러그인 (as shown in Figure 5.8)을 통해 최신 학술 연구 결과를 얻으세요.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-06.png" alt="Image description" width="90%"/>
<p>그림 5.6 미디어 플랫폼용 RSS 소스 플러그인</p>
</div>
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-07.png" alt="Image description" width="90%"/>
<p>그림 5.7 GitHub 플러그인</p>
</div>
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-08.png" alt="Image description" width="90%"/>
<p>그림 5.8 Arxiv 플러그인</p>
</div>

2. **개인화된 구성:** 각 플러그인에 대해 세부적인 구성을 수행하여 필요한 데이터를 정확하게 얻을 수 있는지 확인하세요. 예를 들어 RSS 플러그인에서 36Kr 및 Huxiu와 같은 웹사이트에 대한 특정 RSS 구독 링크를 입력하세요. GitHub 플러그인에서 모니터링할 키워드 쿼리 수량 및 최신 업데이트 설정을 설정합니다. arXiv 플러그인에서는 "LLM", "AI" 등 관심 키워드를 정의하고 수량 및 최신 업데이트 설정을 정의합니다.

```
RSS Link Configuration

- **36Kr:** https://www.36kr.com/feed
- **Huxiu:** https://rss.huxiu.com/
- **IT Home:** http://www.ithome.com/rss/
- **InfoQ:** https://feed.infoq.com/ai-ml-data-eng/

GitHub Plugin Configuration

- q:AI
- per_page:10
- sort:updated

Arxiv Plugin Configuration

- count: 5
- search_query: AI
- sort_by: 2
```

3. **오케스트레이션 및 연결:** 에이전트의 시각적 오케스트레이션 인터페이스에서 구성된 정보 소스 플러그인 (such as ⟪P0⟫, ⟪P1⟫, ⟪P2⟫, etc.)을 데이터 입력 노드로 사용하고 이를 후속 논리 처리 모듈(예: **대형 모델** 모듈)에 연결하여 그림 5.9와 같이 완전한 데이터 처리 경로를 구축합니다.
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-09.png" alt="Image description" width="90%"/>
<p>그림 5.9 일일 AI 브리핑 오케스트레이션 흐름도</p>
</div>


**2단계: 상담원 역할 및 프롬프트 설정**

역할 설정 및 프롬프트 작성은 상담원 행동 및 출력 품질을 정의하는 핵심 단계입니다. 이 단계의 목표는 추상 지침을 에이전트가 이해하고 실행할 수 있는 특정 작업으로 변환하는 것입니다.

(1) 역할 설정

우리는 에이전트를 **선임되고 권위 있는 기술 미디어 편집자**로 설정했습니다. 이 역할은 에이전트에게 명확한 전문적 포지셔닝을 제공하여 후속 콘텐츠 생성 시 전문 편집자의 사고 방식을 모방하고 효율적인 정보 선별, 통합 및 요약을 수행할 수 있도록 합니다.

(2) 신속한 작성 및 구조화

프롬프트는 에이전트가 작업을 실행하는 데 필요한 지침 매뉴얼입니다. 지침이 명확하고 완전하며 제어 가능하도록 이를 **시스템 프롬프트와 사용자 프롬프트**로 나눕니다.

**시스템 프롬프트**

시스템 프롬프트는 상담원의 장기적인 행동 지침과 출력 형식 사양을 정의하는 데 사용됩니다.

```
# Role
You are a senior and authoritative technology media editor, skilled at efficiently and precisely integrating and creating highly professional technology briefs, with deep analytical and integration capabilities especially in AI field technical developments, cutting-edge academic research results, and popular open-source projects.

## Workflow
### Daily Report Output Format
1. The daily report should prominently display "AI Daily Report", "by@jasonhuang", and the current date at the beginning, for example: "AI Daily Report | September 24, 2025 | by@jasonhuang".
2. <!!!important!!!> Add a unique Emoji symbol at the beginning of each title based on the different content of each AI technology news, each AI academic paper, and each AI open-source project.
3. All output content must be highly relevant to AI, LLM, AIGC, large models, and other technical topics, firmly excluding any irrelevant information, advertisements, and marketing content.
4. Must provide the original link for each item (including AI technology news, AI academic papers, AI open-source projects).
5. Provide a brief and precise summary description for each news item or project output.
```

**사용자 프롬프트**

사용자 프롬프트는 특정 작업 지침과 데이터 소스를 정의하는 데 사용됩니다.

```
- **Information Extraction and Integration:** From input sources `{{articles}}`, `{{articles1}}`, `{{articles2}}`, and `{{articles3}}`, filter and extract article titles and corresponding links related to AI, large models, AIGC, LLM, and other topics, and organize them into the **"AI Technology News"** module.
- **Academic Paper Summary:** From input source `{{arxiv}}`, based on fields `arxiv_title` and `arxiv_link`, summarize and organize the latest paper content to form the **"AI Academic Papers"** module.
- **Open-Source Project Filtering:** From input source `{{GitHub}}`, filter out the **5 most prominent and influential AI open-source projects**. Extract the titles and corresponding links of these projects and organize them into the **"AI Open-Source Projects"** module.

# Attention
- Strictly follow the daily report output format defined in the system prompt.
- The total output content should be: **10 AI technology news items, 5 AI academic papers, 5 AI open-source projects**.
```



**3단계: 테스트, 디버깅 및 다중 채널 게시**

에이전트의 핵심 로직 구성을 완료한 후에는 출력이 기대치를 충족하는지 확인하기 위해 엄격한 테스트 및 디버깅을 수행해야 합니다.

**미리보기 실행:** Coze 플랫폼의 미리보기 인터페이스에서 에이전트를 실행하고 생성되는 간략한 콘텐츠를 관찰합니다.

```
# AI Daily Report by@jasonhuang 2025-09-24

## 🚀 AI Technology News

🤖 **Zhiyuan Robot GO-1 Universal Embodied Foundation Model Fully Open-Sourced**
Link: https://36kr.com/p/3479085489708163?f=rss
Summary: Zhiyuan Robot announced the full open-sourcing of its GO-1 universal embodied foundation model, providing powerful AI foundational capabilities for the robotics field.

🔬 **Microsoft Overcomes Data Center Chip Cooling Bottleneck: Microfluidics + AI Precision Cooling**
Link: https://www.ithome.com/0/885/391.htm
Summary: Microsoft achieves precise temperature control of data center chips through the combination of microfluidic technology and AI algorithms, improving energy efficiency.
......

## 📚 AI Academic Papers

🧪 **Lyra: Generative 3D Scene Reconstruction via Video Diffusion Model Self-Distillation**
Link: http://arxiv.org/pdf/2509.19296v1
Summary: Proposes an innovative framework for 3D scene generation through video diffusion model self-distillation, without requiring multi-view training data.

📊 **The ICML 2023 Ranking Experiment: Examining Author Self-Assessment in ML/AI Peer Review**
Link: http://arxiv.org/pdf/2408.13430v3
Summary: Studies the effectiveness of author self-assessment in machine learning conference review processes and proposes methods to improve review mechanisms.
......

## 💻 AI Open-Source Projects

🤖 **llmling-agent - Multi-Agent Workflow Framework**
Link: https://github.com/phil65/llmling-agent
Summary: Multi-agent interaction framework supporting YAML configuration and programming methods, integrating MCP and ACP protocol support.

🚌 **College_EV_AI_Transportation - Campus AI Electric Transportation System**
Link: https://github.com/LuisMc2005v/College_EV_AI_Transportation
Summary: AI-driven campus electric transportation optimization system, achieving real-time tracking and efficient carpooling services.
......
```

내용의 정확성, 형식의 완전성, 언어 스타일을 주의 깊게 확인하세요. 기대에 미치지 못하는 부분이 발견되면 프롬프트 또는 플러그인 구성 단계로 돌아가 세부 조정을 수행하세요. 예를 들어 콘텐츠가 충분히 간결하지 않은 경우 프롬프트에서 요약 요구 사항을 수정합니다. 데이터 수집이 정확하지 않은 경우 플러그인 구성 매개변수를 확인하세요.

다중 채널 게시: Coze는 한 번의 클릭으로 에이전트를 여러 주류 애플리케이션 플랫폼 (such as WeChat, Doubao, Feishu, etc.)에 게시하는 기능을 제공하여 그림 5.10과 같이 에이전트의 애플리케이션 시나리오를 크게 확장합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-10.png" alt="Image description" width="90%"/>
<p>그림 5.10 코즈 플랫폼의 다양한 퍼블리싱 채널</p>
</div>

에이전트가 게시된 후 Coze 스토어에서 생성한 AI 에이전트를 볼 수 있으며 그림 5.11 및 5.12와 같이 AI 애플리케이션에 통합되어 사용자에게 서비스를 제공할 수도 있습니다. [데일리 AI 뉴스에이전트 체험링크](https://www.coze.cn/store/agent/7506052197071962153?bot_id=true&bid=6hkt3je8o2g16))도 있어요

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-11.png" alt="Image description" width="90%"/>
<p>그림 5.11 AI 에이전트 - 일일 AI 뉴스</p>
</div>

또한, 이 [체험 링크](https://www.coze.cn/store/project/7458678213078777893?from=store_search_suggestion&bid=6gu3cmr7k5g1i))를 클릭하면 AI 애플리케이션에서 Daily AI News를 볼 수 있습니다.
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-12.png" alt="Image description" width="90%"/>
<p>그림 5.12 AI 애플리케이션의 일일 AI 뉴스</p>
</div>
**게시 구성:** 자체 에이전트를 게시하려면 그림 5.13 및 5.14에 표시된 것처럼 보다 친숙한 사용자 환경을 제공하기 위해 게시하기 전에 에이전트에 대한 적절한 이름, 아바타 및 환영 메시지도 구성해야 합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-13.png" alt="Image description" width="90%"/>
<p>그림 5.13 Agent 기본 정보 구성</p>
</div>
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-14.png" alt="Image description" width="90%"/>
<p>그림 5.14 Agent에 대한 시작 발언 및 사전 설정된 질문 구성</p>
</div>


### 5.2.3 코즈의 장점과 한계 분석

**장점:**

* **강력한 플러그인 생태계:** Coze 플랫폼의 핵심 장점은 에이전트가 외부 서비스 및 데이터 소스에 쉽게 액세스할 수 있도록 지원하는 풍부한 플러그인 라이브러리에 있으며 높은 기능 확장성을 달성합니다.
  * **직관적인 시각적 오케스트레이션:** 플랫폼은 낮은 임계값의 시각적 워크플로 오케스트레이션 인터페이스를 제공합니다. 사용자는 깊은 프로그래밍 지식 없이도 "드래그 앤 드롭"을 통해 복잡한 워크플로를 구축할 수 있어 개발 난이도가 크게 줄어듭니다.
  * **유연한 프롬프트 제어:** 정확한 역할 설정과 프롬프트 작성을 통해 사용자는 상담원 행동 및 콘텐츠 생성을 세밀하게 제어하여 고도로 맞춤화된 결과를 얻을 수 있습니다. 또한 신속한 관리 및 템플릿을 지원하여 개발자의 에이전트 개발을 크게 촉진합니다.
  * **편리한 다중 플랫폼 배포:** 동일한 에이전트를 다양한 애플리케이션 플랫폼에 게시하여 원활한 크로스 플랫폼 통합 및 애플리케이션을 달성합니다. 또한 Coze는 지속적으로 새로운 플랫폼을 생태계에 통합하고 있으며 점점 더 많은 휴대폰 제조업체와 하드웨어 제조업체가 점차 Coze 에이전트 게시를 지원하고 있습니다.

**제한사항:**

* **MCP를 지원하지 않습니다:** 이것이 가장 치명적이라고 생각합니다. Coze의 플러그인 시장은 매우 풍부하고 매력적이지만 MCP를 지원하지 않으면 개발을 제한하는 족쇄가 될 수 있습니다. 열리면 또 다른 킬러 기능이 될 것입니다.
  * **일부 플러그인 구성의 복잡성이 높음:** API 키 또는 기타 고급 매개변수가 필요한 플러그인의 경우 올바른 구성을 완료하려면 사용자에게 기술적 배경이 필요할 수 있습니다. 복잡한 워크플로 조정도 기초 없이 마스터할 수 있는 것이 아닙니다. JavaScript 또는 Python 기본 사항이 필요합니다.
  * **JSON 파일을 직접 가져올 수 없습니다:** 이전에는 앱에 내보내기/가져오기 기능이 없었지만 이제 유료 버전에는 있습니다. 그러나 내보내거나 가져온 파일은 Dify 또는 N8n과 같은 JSON 파일이 아닙니다. ZIP 파일이에요. 이는 앱에서만 내보낸 다음 ZIP 파일을 가져올 수 있음을 의미합니다. 그러나 해결 방법을 사용할 수 있습니다. 레이아웃 인터페이스에서 Ctrl+A를 눌러 모두 선택한 다음 Ctrl+C를 눌러 레이아웃을 복사한 다음 다른 빈 워크플로나 다른 워크플로에 붙여넣습니다.


## 5.3 플랫폼 2: Dify
### 5.3.1 Dify 및 생태계 소개

Dify는 그림 5.15와 같이 BaaS(Backend as a Service)와 LLMOps의 개념을 통합하여 프로토타입 설계부터 프로덕션 배포까지 전체 프로세스 지원을 제공하는 오픈 소스 LLM(대형 언어 모델) 애플리케이션 개발 플랫폼입니다. 데이터 계층, 개발 계층, 오케스트레이션 계층 및 기초 계층으로 구분된 계층형 모듈식 아키텍처를 채택하고 각 계층은 쉽게 확장할 수 있도록 분리됩니다.

Dify는 모델 중립성과 호환성이 매우 높습니다. 오픈 소스 모델이든 상용 모델이든 사용자는 간단한 구성을 통해 모델을 통합하고 통합 인터페이스를 통해 추론 기능을 호출할 수 있습니다. GPT, Deepseek, Llama와 같은 모델은 물론 OpenAI API와 호환되는 모든 모델을 포괄하는 수백 개의 오픈 소스 또는 독점 LLM과의 통합을 기본적으로 지원합니다.

동시에 Dify는 로컬 배포(공식 Docker Compose 원클릭 시작)와 클라우드 배포를 지원합니다. 사용자는 로컬/개인 환경에서 Dify를 자체 배포하거나(데이터 개인 정보 보호) 공식 SaaS 클라우드 서비스를 사용할 수 있습니다(아래 비즈니스 모델 섹션에 자세히 설명되어 있음). 이러한 배포 유연성 덕분에 보안 요구 사항이 있는 기업 인트라넷 환경이나 운영 편의성 요구 사항이 있는 개발자 그룹에 적합합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-01.png" alt="Image description" width="90%"/>
<p>그림 5.15 Dify 공식 홈페이지</p>
</div>

Marketplace 플러그인 생태계: Dify Marketplace는 원스톱 플러그인 관리 및 원클릭 배포 기능을 제공하여 개발자가 플러그인을 검색, 확장 또는 제출할 수 있도록 하여 그림 5.16과 같이 커뮤니티에 더 많은 가능성을 제공합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-02.png" alt="Image description" width="90%"/>
<p>그림 5.16 Dify Marketplace 플러그인 생태계</p>
</div>
마켓플레이스에는 다음이 포함됩니다.


- 모델
- 도구
- 에이전트 전략
- 확장
- 번들

현재 Dify Marketplace에는 다양한 기능과 애플리케이션 시나리오를 다루는 8,677개 이상의 플러그인이 있습니다. 그중 공식적으로 권장되는 플러그인은 다음과 같습니다.
- 구글 검색: langgenius/google
- Azure OpenAI: langgenius/azure_openai
- 개념: langgenius/notion
- DuckDuckGo: langgenius/duckduckgo


Dify는 최소한의 환경 설정만으로 널리 사용되는 IDE와 원활하게 협력하는 원격 디버깅 기능을 포함하여 플러그인 개발자를 위한 강력한 개발 지원을 제공합니다. 개발자는 테스트를 위해 모든 플러그인 작업을 로컬 환경으로 전달하면서 Dify의 SaaS 서비스에 연결할 수 있습니다. 이 개발자 친화적인 접근 방식은 플러그인 제작자에게 권한을 부여하고 Dify 생태계의 혁신을 가속화하는 것을 목표로 합니다. 이것이 바로 Dify가 현재 가장 성공적인 에이전트 플랫폼 중 하나가 될 수 있는 이유이기도 합니다. 모델을 모두 통합할 수 있고 프롬프트 및 오케스트레이션을 복사할 수 있지만 도구 플러그인의 존재 여부와 풍부함에 따라 에이전트가 더 나은 결과를 얻을 수 있는지 또는 예기치 않게 강력한 기능을 얻을 수 있는지 직접적으로 결정되기 때문입니다.

### 5.3.2 슈퍼 에이전트 개인 비서 구축
> **✨✨ 세부 운영 가이드**: **[Dify 에이전트 생성 단계별 튜토리얼](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra03-Dify智能体创建保姆级操作流程.md)**)을 참조하세요.

이전 Coze 사례에서는 일일 AI 간략 에이전트를 구축했습니다. 기능은 분명하지만 단일 간략 생성 기능은 다소 제한되어 있습니다. 이 섹션에서는 Dify를 사용하여 일일 Q&A, 카피라이팅 최적화, 다중 모드 생성 및 데이터 분석과 같은 여러 시나리오를 다루는 완전한 기능을 갖춘 슈퍼 에이전트 개인 비서를 구축합니다. 시작하기 전에 Dify의 주요 인터페이스와 기능 모듈에 대해 간략하게 알아보겠습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-14.png" alt="Image description" width="90%"/>
<p>그림 5.17 Dify 에이전트 구축 홈페이지</p>
</div>
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-18.png" alt="Image description" width="90%"/>
<p>그림 5.18 Dify 공식 템플릿 라이브러리</p>
</div>
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-15.png" alt="Image description" width="90%"/>
<p>그림 5.19 Dify 지식 베이스</p>
</div>
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-16.png" alt="Image description" width="90%"/>
<p>그림 5.20 Dify 플러그인 시장</p>
</div>
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-17.png" alt="Image description" width="90%"/>
<p>그림 5.21 대형 모델 구성 Dify</p>
</div>

**(1) 플러그인 생성 및 MCP 구성**

에이전트를 구축하기 전에 먼저 필요한 플러그인 설치 및 MCP 구성을 완료해야 합니다. 그림 5.22에 표시된 것처럼 이 경우에 필요한 핵심 플러그인은 다음과 같습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-19.png" alt="Image description" width="90%"/>
<p>그림 5.22 Dify 플러그인 설치 구성</p>
</div>

그림에서 빨간색 박스로 표시된 플러그인은 Dify 플러그인 마켓에서 검색하여 설치해야 합니다. 사용자는 클릭하여 세부 정보를 보고 각 플러그인의 특정 기능을 이해할 수 있습니다.

다음으로 MCP(Model Context Protocol)를 구성합니다. 여기서는 MCP의 세부 원칙을 확장하지 않습니다. 클라우드에 배포된 MCP 서비스를 사용하는 방법을 시연하는 데 중점을 둘 것입니다. 이 사례에서는 그림 5.23과 같이 국내 ModelScope 커뮤니티 MCP 시장을 시연용으로 사용했습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-20.png" alt="Image description" width="90%"/>
<p>그림 5.23 ModelScope 커뮤니티 MCP 시장</p>
</div>

ModelScope 커뮤니티 MCP 시장을 열고 호스팅 유형을 선택합니다. Amap MCP를 예로 들면, 홈페이지에 진입한 후 오른쪽에서 SSE 모드를 선택하고 연결 구성을 클릭하면 그림 5.24와 같이 전용 MCP 구성 JSON이 생성됩니다. MCP는 여러 통신 모드를 지원하지만 Dify에서는 SSE 모드 통신을 사용하는 것이 더 부드럽고 안정적이므로 SSE 모드를 권장합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-21.png" alt="Image description" width="90%"/>
<p>그림 5.24 Amap MCP 구성 예</p>
</div>

**(2) 에이전트 디자인 및 효과 표시**

이 사례는 다음 기능 모듈을 다루는 포괄적인 개인 비서를 생성합니다.

- 생활Q&A
- 카피라이팅 연마 및 최적화
- 다중 모드 콘텐츠 생성(이미지, 비디오)
- 데이터 쿼리 및 시각화 분석
- MCP 도구 통합(Amap, 식단 추천, 뉴스 정보)

전체 에이전트 조정 아키텍처는 그림 5.25에 나와 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-12.png" alt="Image description" width="90%"/>
<p>그림 5.25 에이전트 오케스트레이션</p>
</div>

다중 에이전트 아키텍처의 경우 지능형 라우팅을 위해 질문 분류기를 사용합니다. 분류자에서는 각 에이전트의 핵심 기능과 작업 범위를 정의하여 사용자 요청이 해당 처리 모듈에 정확하게 분배될 수 있도록 합니다.

**일일 보조 모듈**

대규모 언어 모델과 시간 도구로 구성된 기본 대화 모듈로 대체 일반 Q&A 서비스 역할을 합니다.

프롬프트 구성:
```
# Role: Daily Question Consultation Expert

## Profile
- language: Chinese
- description: Specializes in answering general questions in users' daily lives, providing practical, accurate, and easy-to-understand advice and answers
- background: Possesses rich life experience and extensive knowledge reserves, skilled at simplifying complex problems
- personality: Kind and friendly, patient and meticulous, pragmatic and reliable
- expertise: Daily life, health and wellness, family management, interpersonal relationships, practical tips


## Skills

1. Problem Analysis Ability
   - Quick Understanding: Rapidly grasp the core points of user questions
   - Classification Recognition: Accurately judge the life domain to which the question belongs
   - Demand Mining: Deeply understand users' potential needs
   - Priority Sorting: Reasonably assess the importance and urgency of problems

2. Answer Providing Ability
   - Knowledge Integration: Comprehensively apply multi-domain knowledge to provide answers
   - Solution Formulation: Provide specific and feasible solutions
   - Step Decomposition: Break down complex problems into simple steps
   - Alternative Solutions: Prepare multiple backup solutions for users to choose from

3. Communication and Expression Ability
   - Popular Language: Use simple and easy-to-understand everyday language
   - Clear Logic: Organize answer content in a well-organized manner
   - Illustrative Examples: Help understanding through specific cases
   - Highlight Key Points: Emphasize key information and precautions

## Rules

1. Answer Principles:
   - Practicality First: Ensure the advice provided is actionable
   - Accuracy Guarantee: Give answers based on reliable information and common sense
   - Neutral and Objective: Avoid personal bias and subjective assumptions
   - Moderate Advice: Provide appropriate depth of answers based on problem complexity

2. Code of Conduct:
   - Timely Response: Quickly respond to users' questions
   - Patient and Meticulous: Maintain patience with repetitive or simple questions
   - Active Guidance: Encourage users to provide more background information
   - Continuous Improvement: Optimize answer quality based on feedback


## Workflows

- Goal: Provide users with practical and reliable daily problem solutions
- Step 1: Carefully read and understand the daily questions raised by users
- Step 2: Analyze the problem type and users' potential needs
- Step 3: Provide specific and feasible suggestions based on common sense and experience
- Step 4: Organize answer content in easy-to-understand language
- Step 5: Check the practicality and safety of the answer


## Initialization
As a daily question consultation expert, you must abide by the above Rules and execute tasks according to Workflows.
```

효과 데모는 그림 5.26에 나와 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-03.png" alt="Image description" width="90%"/>
<p>그림 5.26 일일 도우미</p>
</div>

**카피라이팅 최적화 모듈**

OpenAI의 데이터 보고서에 따르면 사용자의 60% 이상이 다듬기, 수정, 확장, 축약을 포함한 텍스트 최적화 관련 작업에 ChatGPT를 사용합니다. 따라서 카피라이팅 최적화는 빈도가 높은 수요 시나리오이며 이를 두 번째 핵심 기능 모듈로 만듭니다.

프롬프트 구성:
```
# I. Role Setting (Role)
You are a professional copywriting optimization expert with rich experience in marketing copywriting and optimization, skilled at improving the attractiveness, conversion rate, and readability of copy. Your perspective is from the angle of the target audience and marketing goals, with professional boundaries limited to the copywriting optimization field, not involving technical implementation or product development.

# II. Background
The user has provided a piece of original copy that needs your optimization to improve its overall effectiveness. Background information includes: the copy may be used for marketing, brand promotion, or information communication scenarios, but the specific use is not detailed. The known condition is that the user hopes the copy is more attractive, clear, or persuasive, but has not provided the original copy content, so you need to work based on general optimization principles.

# III. Task Objectives (Task)
- Analyze and optimize the structure, language, and style of the copy to make it more in line with the preferences of the target audience.
- Improve the attractiveness, readability, and conversion potential of the copy, ensuring clear information delivery.
- Make adjustments according to common optimization principles (such as conciseness, emotional resonance, call to action, etc.), without content rewriting unless necessary.
- While maintaining core information, appropriately expand and enrich copy content to provide a more comprehensive optimized version.

# IV. Limitation Prompts (Limit)
- Avoid changing the core information or intent of the original copy unless explicitly requested by the user.
- Do not add fictional or irrelevant content, ensuring optimization is based on logic and best practices.
- Avoid using overly technical or professional terminology unless the target audience is professionals.
- Do not involve optimization of images, layouts, or other non-text elements.

# V. Output Format Requirements (Example)
The output should be optimized copy text with clear structure, fluent language, and substantial content. For example:
- If the original copy is "Our product is very good, come and buy it"
The optimized version can be: "In this era full of choices, what truly touches people's hearts is never exaggerated propaganda, but good products that can withstand the test of time and users. Our product is exactly that. It not only pays attention to details and quality in design but also continuously polishes and innovates in function, just to bring a better user experience to every user. Whether it's the texture of the appearance or the stability of performance, we always adhere to high standards and strict requirements, striving to make every customer who chooses us feel the surprise of value for money.
We deeply understand that purchasing a product is not just a simple consumption but a choice of lifestyle. Therefore, from material selection, craftsmanship to after-sales service, we have poured full sincerity and professionalism into every link, carefully guarding your every experience. Whether you pursue practicality, value quality, or want unique personalization, our products can provide you with ideal solutions.
Now, let us prove everything with action. A truly good product does not need too much embellishment; it itself is the best spokesperson. Act now, choose us, let quality change life, and have a different experience from now on!"
- The output should directly present optimized content without additional explanations or annotations unless requested by the user. Please ensure that the optimized copy content is richer and more complete, and the optimized copy text must exceed 500 words.
```

효과 데모는 그림 5.27에 나와 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-04.png" alt="Image description" width="90%"/>
<p>그림 5.27 카피라이팅 도우미</p>
</div>

**다중 모드 생성 모듈**

이미지 및 비디오 생성은 또 다른 고주파 애플리케이션 시나리오입니다. Jimeng 이미지 생성 및 Google Imagen과 같은 모델의 발전과 Keling, Google Veo 3, Seedance2.0 및 OpenAI Sora 2와 같은 비디오 생성 기술의 획기적인 발전으로 다중 모드 콘텐츠 생성의 품질이 실용적인 수준에 도달했습니다.

이 사례에서는 Jimeng 플러그인을 사용하여 이미지 및 비디오 생성을 구현합니다. 구성 단계는 다음과 같습니다.

1. 워크플로우에 Jimeng 이미지/비디오 생성 플러그인 추가
2. 매개변수 구성 (such as image ratio 1:1, model selection seedream4.5/5.0 and seedance1.5/2.0)
3. 생성된 파일을 출력합니다.

여기서는 Jimeng의 플러그인을 사용하여 최신 모델, 즉 Seedream5.0 및 Seedance2.0을 호출합니다. 이미지와 비디오의 품질이 크게 향상되었습니다. 또한 비디오 작업 검색을 위해 루프를 사용하고 매개변수 검색 및 조건부 판단으로 사례 데모를 보완합니다. 자세한 내용은 [전체 워크플로](code/chapter5/HelloAgent_difyCase.yml)를 참조하세요.

이미지 생성 구성 및 효과는 그림 5.28 및 5.29에 나와 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-13.png" alt="Image description" width="90%"/>
<p>그림 5.28 이미지 생성 설정</p>
</div>
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-05.png" alt="Image description" width="90%"/>
<p>그림 5.29 이미지 생성 도우미</p>
</div>

비디오 생성 효과는 그림 5.30에 나와 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-06.png" alt="Image description" width="90%"/>
<p>그림 5.30 비디오 어시스턴트</p>

<p><a href="⟪P0⟫ to watch video demo</a></p>
</div>

**데이터 쿼리 및 분석 모듈**

데이터 처리는 에이전트의 중요한 기능 중 하나입니다. 이 모듈에서는 데이터 쿼리 및 시각화 분석을 구현하기 위해 Dify의 데이터베이스에 연결하는 방법을 보여줍니다.

먼저 데이터 쿼리 도구 플러그인을 설치합니다. 이 경우에는 `rookie-text2data` 플러그인을 사용합니다. 데이터 쿼리의 핵심은 정확한 SQL 쿼리 문을 생성할 수 있도록 명확한 테이블 구조와 필드 정보를 대규모 모델에 제공하는 것입니다. 일반적인 관행은 다음과 같습니다.

- 데이터 테이블의 DDL 문을 직접 제공
- 테이블 이름과 필드 이름 간의 대응에 대한 설명 제공

그림 5.31과 같이 데이터베이스 연결 정보 (IP address, database name, port, account, password, etc.)를 구성합니다. 쿼리 결과는 대규모 모델 노드를 통해 구성되고 이해하기 쉬운 자연어 출력으로 변환되어야 합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-22.png" alt="Image description" width="90%"/>
<p>그림 5.31 데이터베이스 구성</p>
</div>

프롬프트 설정:

```
# I. Role Setting (Role)
You are a professional data query specialist, skilled at data organization, with clear logical thinking and concise expression ability.

# II. Background
The user has provided raw data queried from the database. This data may have issues such as inconsistent formats, missing fields, and duplicate records, and needs professional organization before effective display.

# III. Task Objectives (Task)
1. Summarize and organize raw data
2. Classify and sort data according to correct logic
3. Data display highlights key information and data insights
4. Provide easy-to-understand data display

# IV. Limitation Prompts (Limit)
1. Must not arbitrarily delete important data
2. Avoid using overly complex or professional statistical terminology
3. Must not tamper with the true values of raw data
4. Avoid displaying too much redundant information, keep it concise and clear
5. Must not leak sensitive data or personal privacy information

# V. Output Format Requirements (Example)
 Data Overview: Simply briefly explain the data content
```

효과 표시는 그림 5.32에 나와 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-07.png" alt="Image description" width="90%"/>
<p>그림 5.32 데이터 쿼리 도우미</p>
</div>

프롬프트 설정:

```
# I. Role Setting (Role)
You are a professional data analyst with data organization, cleaning, and visualization capabilities, able to extract key information from raw data and transform it into intuitive visual displays.

# II. Background
The user has queried a batch of raw data from the database. This data may contain multiple fields, missing values, or inconsistent formats, and needs to be organized before generating visualization charts.

# III. Task Objectives (Task)
# Workflow
1. Data Analysis
Analyze, organize, and summarize data according to reasonable rules
2. Analysis & Visualization
Generate at least 1 chart (choose one or more from bar / line / pie chart)
Can call tools: "generate_pie_chart" | "generate_column_chart" | "generate_line_chart"

# IV. Limitation Prompts (Limit)
1. Avoid using overly complex chart types, ensure visualization results are easy to understand
2. Do not ignore data quality issues, must perform necessary data cleaning
3. Avoid using too many colors or elements in visualization, keep it concise and clear
4. Do not omit labeling and explanation of key data
5. Must perform summary and chart generation, regardless of data volume

# V. Output Format Requirements (Example)
Please output in the following format:
1. Data overview summary (do not output field names, do not list points, just a short paragraph)
2. Display generated charts
```

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-08.png" alt="Image description" width="90%"/>
<p>그림 5.33 데이터 분석 도우미</p>
</div>

데이터 분석 도우미의 유일한 차이점은 데이터 시각화 도구, 즉 "generate_pie_chart" | "generate_column_chart" | "generate_line_chart" BI 차트 생성 도구 플러그인. 이전에 필요에 따라 설치한 경우 직접 추가하여 사용할 수 있으며 위 프롬프트와 같이 해당 설명을 추가할 수 있습니다.

**MCP 도구 통합**

마지막으로 MCP 도구의 통합 응용 프로그램입니다. 이전에 MCP 구성을 이미 완료했으므로 이제 이를 에이전트에 통합하겠습니다. 구성 단계는 다음과 같습니다.

1. MCP 통화를 지원하는 상담원 전략을 선택하세요.
2. 반응 모드를 선택하세요
3. MCP 서비스 구성(`mcp-server` 접두사를 삭제하려면 SSE 모드를 선택하세요.)
4. 해당 프롬프트를 입력하세요

구성 인터페이스는 그림 5.34에 나와 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-23.png" alt="Image description" width="90%"/>
<p>그림 5.34 에이전트 MCP 구성</p>
</div>

Amap 보조자, 식이 보조자 및 뉴스 보조자의 효과는 각각 그림 5.35, 5.36 및 5.37에 나와 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-09.png" alt="Image description" width="90%"/>
<p>그림 5.35 Amap 보조자</p>
</div>

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-10.png" alt="Image description" width="90%"/>
<p>그림 5.36 다이어트 보조원</p>
</div>

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-11.png" alt="Image description" width="90%"/>
<p>그림 5.37 뉴스 도우미</p>
</div>

이 시점에서 우리는 완전한 기능을 갖춘 슈퍼 에이전트 개인 비서를 완성했습니다. 이 도우미는 삶의 다양한 측면을 다룹니다. 새 옷이 필요할 때 Doubao에서 디자인을 생성하도록 할 수 있습니다. 외출하기 전에 Amap 보조원이 경로를 계획할 수 있습니다. 무엇을 먹어야 할지 모를 때 식단 추천을 받을 수 있습니다. 학습 상황을 이해하고 싶을 때 데이터 분석을 수행할 수 있습니다. 이 에이전트는 다양한 업무 및 생활 업무를 처리할 수 있으며, 모두가 더욱 창의적인 개인 에이전트 어시스턴트를 구축할 수 있기를 기대합니다.

### 5.3.3 Dify의 장점과 한계 분석

선도적인 AI 애플리케이션 개발 플랫폼인 Dify는 여러 측면에서 상당한 이점을 보여줍니다.

1. 핵심 장점

- 풀스택 개발 경험: Dify는 RAG 파이프라인, AI 워크플로우, 모델 관리 및 기타 기능을 하나의 플랫폼에 통합하여 원스톱 개발 경험을 제공합니다.
- 로우 코드와 높은 확장성 사이의 균형: Dify는 로우 코드 개발의 편의성과 전문 개발의 유연성 사이에서 적절한 균형을 유지합니다.

- 엔터프라이즈 수준 보안 및 규정 준수: Dify는 AES-256 암호화, RBAC 권한 제어 및 감사 로그를 제공하여 엄격한 보안 및 규정 준수 요구 사항을 충족합니다.

- 풍부한 도구 통합 기능: Dify는 9000개 이상의 도구 및 API 확장을 지원하여 광범위한 기능 확장성을 제공합니다.
- 활성 오픈 소스 커뮤니티: Dify에는 풍부한 학습 리소스와 지원을 제공하는 활성 오픈 소스 커뮤니티가 있습니다.



2. 주요 제한 사항
- 가파른 학습 곡선: 기술적 배경 지식이 전혀 없는 사용자의 경우 여전히 특정 학습 곡선이 있습니다.

- 성능 병목 현상: 적절한 최적화가 필요한 높은 동시성 시나리오에서 성능 문제에 직면할 수 있습니다. Dify 시스템의 핵심 서버 측 구성 요소는 Python으로 구현되어 C++, Golang, Rust와 같은 언어에 비해 성능이 상대적으로 낮습니다.

- 불충분한 멀티모달 지원: 현재 주로 텍스트 처리에 중점을 두고 있으며 이미지, 비디오, HTML 등에 대한 지원은 제한적입니다.

- 높은 Enterprise Edition 비용: Dify의 Enterprise Edition 가격은 상대적으로 높기 때문에 소규모 팀의 예산을 초과할 수 있습니다.

- API 호환성 문제: Dify의 API 형식은 OpenAI와 호환되지 않아 특정 타사 시스템과의 통합이 제한될 수 있습니다.


## 5.4 플랫폼 3: FastGPT

### 5.4.1 FastGPT 소개 및 핵심 기능

FastGPT는 오픈 소스 LLM 기반 지식 기반 Q&A 플랫폼이자 에이전트 구축 도구입니다. 핵심 포지셔닝은 사용하기 쉬운 RAG(Retrieval-Augmented Generation) 솔루션과 시각적 워크플로우 조정 기능을 제공하는 데 초점을 맞춘 "엔터프라이즈급 AI 생산성 엔진"입니다. Coze의 제로 코드 경험 및 Dify의 풀 스택 개발 기능과 달리 FastGPT는 지식 기반 Q&A를 일류 시민으로 취급하여 "데이터 가져오기 - 지능형 청크 - 벡터 검색 - 대화 상자 생성"의 전체 체인을 중심으로 심층적으로 최적화합니다.

FastGPT 공식 웹사이트를 방문하면 그림 5.38과 같이 안전하고 제어 가능한 엔터프라이즈급 AI 에이전트의 구축을 강조하는 간결하고 강력한 제품 선언문인 "엔터프라이즈급 AI 생산성 엔진"이 가장 먼저 보입니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-01.png" alt="Image description" width="90%"/>
<p>그림 5.38 FastGPT 공식 홈페이지 홈페이지</p>
</div>

플랫폼에 로그인하면 명확한 작업 공간 레이아웃을 볼 수 있습니다. 왼쪽 탐색 모음은 핵심 기능을 대화 포털, 작업 공간, 기술 자료 및 계정의 네 가지 모듈로 나눕니다. 그 중 Agent 모듈은 Workflow, Dialog Agent, Dialog Agent V2(Beta)의 세 가지 유형으로 더욱 세분화되어 사용자가 비즈니스 시나리오에 따라 적절한 구성 모드를 선택하는 것이 편리합니다. 메인 영역은 Sales Training Master, Document Translation Assistant, Industry Trend Insight Briefing과 같은 공식 템플릿이 내장되어 있어 "템플릿에서 생성"을 위한 빠른 진입점을 제공합니다. 아래는 그림 5.39와 같이 사용자 자신의 Agent 목록입니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-02.png" alt="Image description" width="90%"/>
<p>그림 5.39 FastGPT 플랫폼 메인 인터페이스</p>
</div>

계정 및 계획 옵션 측면에서 FastGPT는 개별 개발자가 사용해 볼 수 있는 무료 버전을 제공합니다. 무료 버전에는 그림 5.40에 표시된 대로 100크레딧, 600개 지식 베이스 색인, 1명의 팀원, 10명의 에이전트, 3개의 지식 베이스, 30일 대화 기록 보관, 30QPM 호출 속도, 한 ​​번에 50MB 크기의 파일 5개 업로드 기능이 포함되어 있습니다. 중소기업과 팀을 위해 플랫폼은 더 높은 동시성 및 스토리지 요구 사항을 충족하기 위해 유료 업그레이드 계획도 제공합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-03.png" alt="Image description" width="90%"/>
<p>그림 5.40 FastGPT 무료 계획 및 사용</p>
</div>

FastGPT의 핵심 경쟁력은 강력한 지식 기반 기능에 있습니다. 플랫폼은 Word, Markdown 및 PDF와 같은 일반적인 문서 유형을 포함하여 다양한 파일 형식 가져오기를 지원합니다. 그림 5.41과 같이 "test General Knowledge Base"에서는 Introduction to Deep Learning, Getting Started with Machine Learning, Bidding Document Text 등 여러 파일을 업로드할 수 있습니다. 시스템은 자동으로 파일을 청크하고 인덱싱하며, 상태가 "준비"로 표시되면 대화에서 해당 파일을 검색하고 참조할 수 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-12.png" alt="Image description" width="90%"/>
<p>그림 5.41 FastGPT 지식 기반 파일 관리</p>
</div>

파일 처리 수준에서 FastGPT는 세분화된 매개변수 구성을 제공합니다. 그림 5.42에 표시된 것처럼 사용자는 "청크 저장" 또는 "Q&A 쌍 추출" 처리 방법 중에서 선택하고, 청킹 조건을 설정하고(예: 원본 텍스트 길이가 1000자를 초과할 때 청킹 트리거), 색인에 제목 추가, 보조 색인 자동 생성, 자동 이미지 색인 생성을 포함한 다양한 색인 향상 옵션을 활성화할 수 있습니다. 텍스트와 이미지 콘텐츠가 광범위하게 혼합된 문서(예: 교과서 및 연구 보고서)의 경우 자동 이미지 인덱싱 기능이 특히 중요합니다. 이를 통해 대형 모델이 응답 시 문서의 시각적 정보를 이해하고 참조할 수 있기 때문입니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-14.png" alt="Image description" width="90%"/>
<p>그림 5.42 지식 베이스 데이터 처리 매개변수 설정</p>
</div>

업로드 후 사용자는 파일 청크의 특정 내용을 볼 수 있습니다. 그림 5.43에서 볼 수 있듯이 "English Grade 4 Lower Semester Full Electronic Book.pdf"를 예로 들면 플랫폼은 각 청크의 텍스트 미리보기를 표시하고 오른쪽 메타데이터 패널에는 파일 크기(62MB), 원본 텍스트 길이(37,797자), 처리 모드(청크 저장) 및 이미지 인덱싱 상태와 같은 주요 정보가 표시됩니다. 이 투명한 청크 디스플레이는 개발자가 지식 기반을 디버깅하고 최적화하는 데 도움을 줍니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-13.png" alt="Image description" width="90%"/>
<p>그림 5.43 지식 기반 파일 청크 세부정보 및 메타데이터</p>
</div>

지식 기반 외에도 FastGPT는 도구 통합의 생태계 추세를 따라잡습니다. 플랫폼은 기본적으로 MCP(Model Context Protocol) 도구를 지원하며, 사용자는 '내 도구' 모듈에서 다양한 MCP 서비스를 통일적으로 관리할 수 있습니다. 그림 5.44에서 볼 수 있듯이 "ai Finance" 폴더 아래에는 Chinese Trend Aggregation, Real-time Stock MCP, QieMan Fund MCP, Minimax-MCP 및 BI Chart Tool을 포함한 여러 MCP 도구를 구성했습니다. 이러한 도구를 사용하면 상담원이 외부 실시간 데이터 및 전문 서비스에 전화할 수 있는 능력을 갖게 됩니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-04.png" alt="Image description" width="90%"/>
<p>그림 5.44 FastGPT MCP 도구 관리</p>
</div>

### 5.4.2 '스마트 투자상담사 보조원' 구축

**사례 설명:** 이 사례에서는 지식 기반, MCP 도구 및 워크플로 조정과 결합된 FastGPT 플랫폼을 사용하여 전문적인 "스마트 투자 자문 도우미"를 구축합니다. 이 도우미는 금융 투자 이론 질문에 답하고, 실시간 주식 시세를 쿼리하고, 사용자 위험 프로필 평가를 수행하고, 개인화된 투자 전략 분석 보고서를 생성할 수 있습니다. 이 사례를 통해 FastGPT의 핵심 개발 패러다임인 지식 기반 구축, MCP 도구 통합, 시각적 워크플로 조정 및 다중 턴 대화 상호 작용 디자인을 마스터하게 됩니다.

**1단계: MCP 도구 구성**

스마트 투자 자문 도우미의 핵심 기능 중 하나는 실시간 금융 데이터를 얻는 것입니다. FastGPT에서는 MCP 도구를 연결하여 이를 달성합니다.

이 경우에는 다음 두 가지 유형의 MCP 서비스가 필요합니다.

1. **실시간 주가 조회**: 개별 주식의 실시간 가격, 가격 변동, 거래량 및 기타 데이터를 얻는 데 사용됩니다.
2. **재무 데이터 및 차트 생성**: 거시 경제 재무 데이터를 얻고 시각적 차트를 생성하는 데 사용됩니다.

그림 5.45에서 볼 수 있듯이 ModelScope 커뮤니티의 MCP 마켓플레이스에서 "Visual Chart MCP Server"를 찾을 수 있습니다. 이 서비스는 MCP 프로토콜과 호환되는 TypeScript를 기반으로 개발되었으며 영역 차트, 막대 차트, 파이 차트 및 기타 다양한 차트를 생성하는 기능을 제공하여 건조된 데이터를 직관적인 시각적 결과로 변환합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-05.png" alt="Image description" width="90%"/>
<p>그림 5.45 ModelScope 커뮤니티 시각적 차트 MCP 서버</p>
</div>

또한 그림 5.46에서 볼 수 있듯이 Alibaba Cloud Bailian 플랫폼은 풍부한 공식 MCP 서비스도 제공합니다. MCP 관리 페이지에서는 "오늘의 투자 - 금융...", "QieMan" 등의 금융 MCP 서비스와 실시간 주가 조회, Wanxiang - Video Generation 등의 도구를 만나보실 수 있습니다. FastGPT의 MCP 도구 라이브러리에 이러한 서비스를 추가한 후 에이전트는 대화 중에 필요할 때 해당 서비스를 호출할 수 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-06.png" alt="Image description" width="90%"/>
<p>그림 5.46 Alibaba Cloud Bailian MCP 관리</p>
</div>

FastGPT의 MCP 도구 구성 인터페이스에서 해당 서비스 주소와 인증 정보를 입력하면 도구 통합이 완료됩니다. 각 MCP 도구는 독립적인 설명과 호출 매개변수로 구성될 수 있으므로 에이전트가 결정을 내릴 때 각 도구의 목적을 더 쉽게 이해할 수 있습니다.

**2단계: 스마트 투자 자문 워크플로우 설계**

도구 구성을 완료한 후 핵심 워크플로 조정 단계로 들어갑니다. FastGPT는 사용자가 노드를 드래그하고 가장자리를 연결하여 복잡한 대화 흐름을 구축할 수 있는 시각적 흐름 조정 인터페이스를 제공합니다.

그림 5.47에 표시된 것처럼 "Smart Investment Advisor Assistant"의 전체 작업 흐름에는 사용자 의도 인식, 지식 기반 검색, 위험 설문지 수집, MCP 도구 호출 및 보고서 생성 등 여러 처리 분기가 포함됩니다. 전체 워크플로우는 데이터가 서로 다른 노드 간에 질서있게 흐르는 명확한 모듈식 구조를 제공합니다. 이러한 시각적 조정 접근 방식을 통해 개발자는 에이전트의 결정 경로를 직관적으로 이해하고 디버그할 수 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-07.png" alt="Image description" width="90%"/>
<p>그림 5.47 스마트 투자 자문 보조 워크플로 조정</p>
</div>

워크플로의 핵심 논리는 다음과 같습니다.

1. **의도 인식 노드**: 먼저 사용자 입력 유형을 결정합니다. 금융 개념 문의인 경우 지식 기반 검색 분기로 라우팅됩니다. 주식 쿼리인 경우 MCP 도구 호출 분기로 라우팅됩니다. 투자진단인 경우 리스크 설문조사 절차에 들어갑니다.
2. **투자 지식 교육 전문가**: 사전 구축된 금융 지식 베이스에 연결하여 관련 투자 이론, 개념 설명, 사례 연구를 검색합니다.
3. **위험 평가 분석가**: 연령, 투자 경험, 월 소득 수준, 위험 허용 범위 및 투자 목표와 같은 차원을 포함하는 양식 입력 노드를 통해 위험 평가 설문지를 통해 사용자를 안내합니다. 그런 다음 데이터는 시장 환경, 기본 및 뉴스 정서 분석을 위해 후속 대형 모델 노드로 전달되고 사용자 프로필, 시장 데이터 및 뉴스 정보를 통합하여 대형 모델을 호출하여 구조화된 투자 전략 분석 보고서를 생성합니다.
4. **시장 뉴스 인텔리전스 전문가**: 외부 데이터를 얻기 위해 사용자 요구에 따라 실시간 주식 MCP 또는 차트 생성 MCP를 호출합니다.
5. **일반 상담 전문가** : "안녕하세요", "잘 지내세요" 등의 간단한 사용자 문의에 답변합니다.

**3단계: 프롬프트 및 기술 자료 구성**

FastGPT에서는 시스템 프롬프트 구성도 똑같이 중요합니다. 현명한 투자 자문 보조원을 위해서는 전문적인 금융 투자 자문 역할을 설정해야 합니다.

```
# I. Role Setting (Role)
You are a professional financial investment advisor with rich experience in risk assessment and portfolio management. Your professional background includes finance, behavioral finance, and investment psychology, enabling you to analyze users' risk tolerance from multiple dimensions. Your tone should be professional, neutral, and easy to understand, avoiding overly complex financial jargon to ensure that ordinary investors can easily understand your analysis.

# II. Background
Users will provide the following information: age, investment experience, monthly income level, maximum tolerable loss, and investment goals. This information is the foundation for risk assessment. You need to analyze this data based on general financial principles (such as lifecycle theory and risk-return matching principles). The scenario limitation is that you cannot access other personal information about the user (such as total assets, family status), so the analysis should focus on the provided information and avoid excessive speculation.

# III. Task Objectives (Task)
Based on the user's provided age, investment experience, monthly income level, maximum tolerable loss, and investment goals, conduct a comprehensive risk assessment.
Output a clear risk level assessment result (e.g., conservative, moderate, aggressive, etc.), and briefly explain the reasoning.
Ensure the analysis logic is coherent and easy for users to understand and apply.

# IV. Limitation Prompts (Limit)
Avoid providing specific financial product recommendations (such as specific stock or fund names); only discuss general asset classes.
Do not make any guaranteed return promises or predictions; emphasize that investment carries risks.
Avoid using overly technical or specialized language to ensure output is friendly to non-professional investors.
Do not expand user information based on assumptions or speculation; only analyze using the provided age, investment experience, monthly income, maximum loss tolerance, and investment goals.
Output must not contain any discriminatory, biased, or strongly subjective statements; maintain objectivity and neutrality.

# V. Output Format Requirements (Example)
Output should be organized according to the following structure:
**Risk Level Assessment**: [e.g., Moderate]
**Assessment Reasoning**: Briefly explain the analysis based on user's age, investment experience, income, loss tolerance, and investment goals.
**Risk Reminder**: Reiterate investment risks and encourage users to adjust based on their own circumstances.
```

동시에 보조자를 위한 금융 지식 기반을 구성해야 합니다. 투자 기초, 재무제표 분석, 거시경제 지표 해석에 관한 문서를 지식베이스에 업로드하고 그림 5.41~5.43의 프로세스에 따라 청킹 및 인덱싱을 완료합니다. 이렇게 하면 사용자가 "P/E 비율과 P/B 비율의 차이점은 무엇입니까?"와 같은 개념적 질문을 하면 에이전트는 대규모 모델의 사전 학습 지식에 전적으로 의존하지 않고 지식 기반에서 정확한 정의와 비교 분석을 검색할 수 있어 환각 위험을 효과적으로 줄일 수 있습니다.

**4단계: 테스트 및 효과 시연**

워크플로우와 프롬프트 구성을 완료한 후 FastGPT의 대화 인터페이스에서 테스트할 수 있습니다. 그림 5.48에서 볼 수 있듯이 스마트 투자 자문 도우미의 시작 메시지는 금융 투자 이론 숙달, 실시간 시장 뉴스 및 데이터, 위험 프로필 평가에 따른 자산 배분 권장 사항이라는 세 가지 주요 기능을 명확하게 소개합니다. 인터페이스는 또한 사용자가 한 번의 클릭으로 일반적인 작업을 실행할 수 있는 빠른 작업 버튼을 제공합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-08.png" alt="Image description" width="50%"/>
<p>그림 5.48 스마트 투자 자문 보조 대화 인터페이스</p>
</div>

사용자가 '투자 조언을 위한 자산 평가 실시'를 클릭하면 도우미가 위험 평가 설문지를 순차적으로 펼쳐 사용자의 연령, 투자 경험, 월 소득 수준, 최대 허용 손실, 투자 목표에 대한 정보를 수집합니다. 이 정보를 바탕으로 보조자는 완전한 투자 전략 분석 보고서를 생성합니다.

그림 5.49에 표시된 대로 보고서에는 다음과 같은 핵심 모듈이 포함되어 있습니다.

- **사용자 위험 평가**: 설문 조사 결과를 바탕으로 사용자의 위험 허용 수준 (e.g., moderate)을 분석합니다.
- **자산배분비율 추천**: 주식, 채권, 현금 등 주요 자산군의 배분비율을 시각적 파이차트 (e.g., stocks 45%, bonds 40%, cash 15%) 형태로 제시합니다.
- **시장 기본 분석**: 현재 거시경제 환경과 업계 동향을 바탕으로 시장 판단을 제공합니다.
- **재조정 전략**: 재조정 주기 및 트리거 조건을 포함하여 주기적인 포트폴리오 재조정에 대한 권장 사항을 제공합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-09.png" alt="Image description" width="90%"/>
<p>그림 5.49 투자전략 분석 보고서</p>
</div>

실시간 데이터 조회 시나리오의 경우 그림 5.50과 같이 사용자가 "귀성마우타이 현재 주가정보를 조회해달라"고 요청하면 에이전트는 자동으로 MCP 툴(`get_stock_quote_realtime`)을 호출하여 실시간 시세 데이터를 획득한다. 반환된 결과에는 제목, 데이터 소스, 주요 하이라이트 (opening price, highest price, intraday price range, trading volume, total market capitalization, circulating market capitalization, etc.)는 물론 잠재적 영향 분석 및 제안 조치가 포함됩니다. 이 체계적이고 전문적인 출력은 에이전트 도구 호출 기능의 실질적인 가치를 반영합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-10.png" alt="Image description" width="50%"/>
<p>그림 5.50 실시간 주가 조회</p>
</div>

개념 설명 측면에서는 그림 5.51과 같이 사용자가 "PER과 P/B 비율의 차이점은 무엇입니까?"라고 질문하면 보조자는 지식 기반과 대규모 모델 이해를 바탕으로 체계적인 비교 분석을 제공합니다. 정의부터 시작하여 P/E 비율과 P/B 비율의 계산 방법을 자세히 설명합니다. 이를 4가지 차원(계산 기준, 적용 산업, 반영된 정보, 제한 사항)으로 비교합니다. 마지막으로 P/E 비율과 P/B 비율에 집중해야 할 시기에 대한 실용적인 적용 조언을 제공합니다. 이 잘 구조화되고 논리적으로 엄격한 출력은 수직 영역 Q&A에서 RAG 강화 대형 모델의 일반적인 장점입니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-11.png" alt="Image description" width="50%"/>
<p>그림 5.51 P/E 비율과 P/B 비율 개념 분석</p>
</div>

5.4.3 FastGPT의 장점과 한계 분석

위의 "스마트 투자 자문 도우미" 구축 실습을 통해 FastGPT 플랫폼에 대한 포괄적인 이해를 형성할 수 있습니다.

**장점:**

- **궁극적인 지식 기반 경험**: FastGPT의 핵심 장점은 RAG 파이프라인의 심층적인 개선에 있습니다. 파일 업로드, 지능형 청크, 인덱스 향상, 이미지 인식부터 검색 호출까지 각 링크는 세분화된 구성 옵션과 투명한 디버깅 인터페이스를 제공합니다. 개인 지식 기반(예: 기업 지식 도우미, 지능형 고객 서비스, 전문 도메인 컨설팅)을 기반으로 Q&A 시스템을 구축해야 하는 시나리오의 경우 FastGPT는 뛰어난 기본 경험을 제공합니다.
- **기본 MCP 지원**: Coze와 달리 FastGPT는 기본적으로 MCP 프로토콜을 지원하므로 ModelScope 및 Alibaba Cloud Bailian과 같은 생태계의 수많은 MCP 서비스와 원활하게 통합될 수 있습니다. 이는 에이전트의 도구 확장성이 더 이상 플랫폼에 내장된 플러그인 라이브러리로 제한되지 않음을 의미합니다. 개발자는 MCP 표준을 준수하는 모든 타사 도구를 자유롭게 통합할 수 있습니다.
- **모델 중립적 디자인**: FastGPT는 OpenAI, Claude, Tongyi Qianwen, DeepSeek 등 다양한 국내외 주류 대형 모델과의 유연한 통합을 지원합니다. 사용자는 비즈니스 요구 사항과 비용 고려 사항에 따라 기본 모델을 자유롭게 전환할 수 있으므로 단일 모델 제공자에 얽매이는 위험을 피할 수 있습니다.
- **시각적 워크플로 오케스트레이션**: Flow 모듈은 드래그 앤 드롭을 통해 복잡한 다중 분기 로직(예: 의도 인식, 설문지 수집 및 보고서 생성)을 신속하게 구축할 수 있는 직관적인 노드 기반 오케스트레이션 인터페이스를 제공하여 개발자가 아닌 사람들의 장벽을 낮춥니다.

**제한사항:**

- **상대적으로 약한 템플릿 생태계**: Coze의 풍부한 플러그인 스토어와 8,000개 이상의 플러그인이 있는 Dify의 마켓플레이스와 비교하면 FastGPT의 공식 템플릿과 내장 도구의 수는 상대적으로 제한되어 있습니다. MCP 프로토콜은 이러한 단점을 부분적으로 보완하지만 적절한 MCP 서비스를 찾고 구성하는 것은 비기술적인 사용자에게는 여전히 특정 임계값을 제시합니다.
- **엄격한 무료 버전 할당량**: 무료 버전은 100크레딧과 30QPM 호출 속도만 제공하므로 빈번한 테스트와 반복이 필요한 개발자에게는 빠르게 소진됩니다. 지식 기반 인덱스 수와 팀 구성원 수에 대한 제한으로 인해 무료 버전에서는 중간 규모의 팀 협업도 지원하기 어렵습니다.
- **커뮤니티 및 문서는 여전히 성숙 단계**: 상대적으로 젊은 오픈 소스 프로젝트인 FastGPT의 커뮤니티 활동 및 영어 문서 완성도는 여전히 Dify 및 n8n과 같은 성숙한 플랫폼에 비해 뒤떨어져 있습니다. 극단적인 경우가 발생하면 소스 코드를 자세히 살펴보거나 커뮤니티에서 도움을 구해야 할 수도 있습니다.

전반적으로 FastGPT는 지식 기반 Q&A 영역에서 경쟁이 매우 치열한 플랫폼입니다. 핵심 요구 사항이 개인 문서를 기반으로 지능형 Q&A 시스템을 구축하고 전체 RAG 파이프라인에 대한 세부적인 제어를 원하는 경우 FastGPT는 시도해 볼 가치가 있는 선택입니다. 강력한 플러그인 생태계나 복잡한 비즈니스 프로세스 자동화가 필요한 시나리오의 경우 Dify 또는 n8n으로 보완할 수 있습니다.


## 5.5 플랫폼 4: n8n

앞서 소개했듯이 n8n의 핵심 아이덴티티는 순수한 LLM 애플리케이션 구축 도구가 아닌 일반적인 워크플로 자동화 플랫폼입니다. 이것을 이해하는 것이 n8n을 마스터하는 열쇠입니다. 지능형 애플리케이션을 구축하기 위해 n8n을 사용할 때 우리는 실제로 더 큰 자동화 프로세스를 설계하고 있으며 대규모 언어 모델은 이 프로세스에서 단지 하나(또는 여러)의 강력한 "처리 노드"일 뿐입니다.

### 5.5.1 n8n의 노드 및 작업 흐름

n8n의 세계는 **노드**와 **워크플로**라는 두 가지 가장 기본적인 개념으로 구성됩니다.

- **노드**: 노드는 워크플로에서 특정 작업을 수행하는 가장 작은 단위입니다. 특정 기능을 갖춘 "빌딩 블록"이라고 생각하면 됩니다. n8n은 이메일 보내기, 데이터베이스 읽기 및 쓰기, API 호출, 파일 처리에 이르기까지 다양한 일반적인 작업을 다루는 수백 개의 사전 설정된 노드를 제공합니다. 각 노드에는 입력과 출력이 있으며 그래픽 구성 인터페이스를 제공합니다. 노드는 대략 두 가지 범주로 나눌 수 있습니다.
  - **트리거 노드**: 프로세스 시작을 담당하는 전체 워크플로의 시작점입니다. 예를 들어 '새 Gmail 이메일이 수신될 때', '한 시간에 한 번씩 트리거됨' 또는 '웹훅 요청이 수신될 때'입니다. 워크플로에는 트리거 노드가 하나만 있어야 합니다.
  - **일반 노드**: 특정 데이터 및 로직 처리를 담당합니다. 예를 들어 'Google 스프레드시트 읽기', 'OpenAI 모델 호출', '데이터베이스에 레코드 삽입' 등이 있습니다.
- **워크플로**: 워크플로는 연결된 여러 노드로 구성된 자동화 흐름도입니다. 이는 데이터가 트리거 노드에서 시작하여 여러 노드 사이를 단계별로 통과하고 처리되고 최종적으로 미리 설정된 작업을 완료하는 전체 경로를 정의합니다. 데이터는 구조화된 JSON 형식으로 노드 간에 전달되므로 각 링크의 입력과 출력을 정확하게 제어할 수 있습니다.


n8n의 진정한 힘은 강력한 "연결" 기능에 있습니다. 원래 격리된 애플리케이션과 서비스(예: 회사 내부 CRM, 외부 소셜 미디어 플랫폼, 데이터베이스, 대규모 언어 모델)를 연결하여 이전에는 복잡한 코딩이 필요했던 엔드투엔드 비즈니스 프로세스 자동화를 달성할 수 있습니다. 다가오는 실습에서는 이 노드와 워크플로우 시스템을 사용하여 AI 기능과 통합된 자동화된 애플리케이션을 구축하는 방법을 직접 경험하게 될 것입니다.

### 5.5.2 지능형 이메일 도우미 구축

n8n의 환경 구성과 가장 기본적인 사용법에 대해서는 프로젝트의 `Additional-Chapter` 폴더에 문서가 생성되어 있으므로 여기서는 자세히 소개하지 않겠습니다. 이전 섹션에서는 n8n의 기본 개념에 대해 배웠습니다. 이 사례는 최신 AI 에이전트와 기존 자동화 워크플로 간의 핵심 차이점을 명확하게 보여줍니다. 전통적인 프로세스는 선형적인 반면, 우리가 구축하려는 에이전트는 사용자 이메일을 수신하고, 핵심 **AI 에이전트 노드**를 통해 "생각"하고, 사용자 의도를 자율적으로 이해하고, 사용 가능한 여러 "도구" 중에서 결정과 선택을 내리고, 최종적으로 관련성이 높은 응답을 자동으로 생성하여 보낼 수 있습니다.

전체 프로세스는 그림 5.52에 표시된 대로 보다 발전된 결정 논리인 `Receive -> AI Agent (Think -> Decide -> Tool Call) -> Reply`를 시뮬레이션합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-01.png" alt="Image description" width="90%"/>
<p>그림 5.52 통합 지능형 ​​이메일 에이전트 아키텍처 다이어그램</p>
</div>

도구를 여러 하위 워크플로로 분할하는 기존 방법과 달리 n8n의 `AI Agent` 노드를 사용하면 LLM(대형 언어 모델), 메모리, 도구와 같은 구성 요소를 통합 인터페이스에 통합하여 구성 프로세스를 크게 단순화할 수 있습니다.

전체 건설 프로세스는 두 가지 핵심 단계로 나뉩니다.

1. **에이전트의 "메모리" 준비**: 에이전트에 대한 개인 지식 베이스를 로드하기 위한 독립적인 프로세스를 만듭니다.
2. **에이전트 본체 구축**: 이메일을 받고, 생각하고, 답장하는 기본 워크플로를 만듭니다.

### 5.5.3 에이전트의 개인 지식 베이스 구축

에이전트가 특정 도메인(예: 개인 정보 또는 프로젝트 문서)에 대한 질문에 답할 수 있도록 하려면 먼저 이에 대한 "외부 두뇌", 즉 벡터 지식 기반을 준비해야 합니다.

n8n에서는 `Simple Vector Store` 노드를 사용하여 메모리에 지식 기반을 빠르게 구축할 수 있습니다. 이 준비 프로세스는 일반적으로 지식을 업데이트할 때 한 번만 실행하면 됩니다.

**(1) 지식 출처 정의**

먼저 `Code` 노드를 사용하여 원시 지식 텍스트를 저장합니다. 이는 간단하고 빠른 방법입니다. 실제 프로젝트에서는 파일, 데이터베이스 등에서 데이터를 가져올 수도 있습니다.

- **노드**: `Code`
- **콘텐츠**: JSON 형식으로 지식을 작성합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-02.png" alt="Screenshot of knowledge base JSON text filled in Code node" width="90%"/>
<p>그림 5.53 코드 노드에서 지식 소스 정의</p>
</div>

```javascript
return [
  {
    "doc_id": "work-schedule-001",
    "content": "My working hours are Monday to Friday, 9 AM to 5 PM. The timezone is Australian Eastern Standard Time (AEST)."
  },
  {
    "doc_id": "off-hours-policy-001",
    "content": "During non-working hours (including weekends and public holidays), I cannot reply to emails immediately."
  },
  {
    "doc_id": "auto-reply-instruction-001",
    "content": "If an email is received during non-working hours, the AI assistant should inform the sender that the email has been received and I will process and reply as soon as possible between 9 AM and 5 PM on the next working day."
  }
];
```

**(2) 텍스트 벡터화(임베딩)**

컴퓨터는 텍스트를 직접 이해할 수 없으며 텍스트를 벡터로 변환해야 합니다. 우리는 이 "번역" 작업을 완료하기 위해 `Embeddings` 노드를 사용합니다.

- **노드**: `Embeddings Google Gemini`, 모델을 `gemini-embedding-exp-03-07`로 선택합니다. 여기서는 데모를 위해 Google API를 사용합니다. Google API를 얻는 방법을 모르는 경우 공식 문서를 참조하세요.
- **구성**: `Code` 노드 뒤에 연결하면 업스트림에서 전달된 텍스트를 벡터 데이터로 자동 변환합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-03.png" alt="" width="90%"/>
<p>그림 5.54 코드에서 데이터 벡터화</p>
</div>

**(3) 벡터 저장소에 저장**

마지막으로 그림 5.55와 같이 벡터화된 지식을 메모리 내 데이터베이스에 저장합니다.

- **노드**: `Simple Vector Store`
- **구성**:
  - **작동 모드**: `Insert Documents`(쓰기 모드).
  - **메모리 키**: 이 기술 자료에 고유한 이름을 지정합니다(예: `my-dailytime`). 이 키는 데이터베이스의 "테이블 이름"과 동일하며 에이전트는 나중에 정보를 찾는 데 이를 사용합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-04.png" alt="" width="90%"/>
<p>그림 5.55 코드의 데이터를 벡터 저장소에 저장</p>
</div>

구성을 완료한 후 **이 프로세스를 한 번 수동으로 실행**하세요. 성공 후에는 그림 5.56과 같이 귀하의 개인적인 지식이 n8n의 메모리에 로드됩니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-05.png" alt="" width="90%"/>
<p>그림 5.56 완전한 기술 자료 로딩 작업흐름</p>
</div>

### 5.5.4 에이전트 기본 워크플로 만들기

도구가 준비되었으므로 이제 에이전트의 기본 프로세스 구축을 시작합니다. 이메일을 받고, 생각하고 결정하고, 방금 만든 도구를 적시에 호출하고, 마지막으로 이메일 답장을 실행하는 일을 담당합니다.

(1) Gmail 트리거 구성

`Agent: Customer Support`이라는 새 워크플로를 만듭니다. `Gmail` 노드를 트리거로 사용하고 해당 **이벤트**를 `Message Received`로 설정하고 이메일 계정을 구성하세요. 이렇게 하면 새 이메일이 받은 편지함에 들어갈 때마다 그림 5.57과 같이 워크플로가 자동으로 트리거됩니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-06.png" alt="" width="90%"/>
<p>그림 5.57 Gmail 노드 만들기</p>
</div>

구성 프로세스는 [n8n 공식 문서](https://docs.n8n.io/integrations/builtin/credentials/google/oauth-single-service/?utm_source=n8n_app&utm_medium=credential_settings&utm_campaign=create_new_credentials_modal#enable-apis). Gmail의 API가 [여기]에서 구성됩니다.(https://console.cloud.google.com/apis/library/gmail.googleapis.com?project=apt-entropy-471905-b9). 자격 증명을 생성하고 웹 애플리케이션 유형을 선택한 후 마지막으로 필요한 클라이언트 ID와 클라이언트 비밀번호를 가져와야 합니다. 또한 n8n에서 제공한 OAuth 리디렉션 URL을 승인된 리디렉션 URI에 추가해야 합니다. 동시에 사용자 추가에서 자신의 이메일 주소도 추가해야 합니다. [관객](https://console.cloud.google.com/auth/audience?project=apt-entropy-471905-b9). 최종 구성된 페이지는 그림 5.58과 같습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-07.png" alt="" width="90%"/>
<p>그림 5.58 Gmail 계정이 성공적으로 로드되었습니다</p>
</div>

이제 그림 5.59와 같이 `Fetch Test Event`을 클릭하여 이메일을 받을 수 있습니다!

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-08.png" alt="" width="90%"/>
<p>그림 5.59 실시간 이메일 받기</p>
</div>

(2) AI 에이전트 노드 구성

이것이 전체 워크플로의 핵심입니다. 노드 메뉴에서 `AI Agent` 노드를 드래그하고 다음과 같이 구성합니다.

- **채팅 모델**: `Google Gemini Chat Model` 등 선택한 대형 언어 모델을 연결하세요. 이것이 에이전트의 "사고 코어"입니다.
- **메모리**: `Simple Memory` 노드를 연결합니다. 이를 통해 에이전트는 동일한 이메일 스레드에서 여러 개의 이메일을 주고받을 때 이전 대화 기록을 기억할 수 있습니다.
- **도구**: 여기에는 여러 도구를 연결할 수 있습니다. 우리의 경우 두 가지 도구를 연결합니다.
  1. `SerpAPI`: 이는 4장 사례에서 사용한 API로 에이전트가 온라인에서 공개 정보를 검색할 수 있는 기능을 제공합니다.
  2. `Simple Vector Store`: 첫 번째 부분에서 생성한 개인 지식 기반을 쿼리할 수 있는 기능을 에이전트에 제공합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-09.png" alt="" width="90%"/>
<p>그림 5.60 AI 에이전트 노드 설정</p>
</div>

이것이 Agent의 "생각"의 첫 번째 단계입니다. `Gemini` 노드(또는 다른 LLM 노드)를 추가하고 모드를 `Chat`로 설정합니다. 우리의 목표는 이메일 콘텐츠를 분석하고 사용자 의도를 판단하는 것입니다. 신속한 디자인이 중요합니다. 명확한 지시를 통해 LLM은 작업을 보다 정확하게 완료할 수 있습니다. 이메일 본문과 제목(`{{ $json.snippet }}{{ $json.Subject }}`)을 프롬프트에 변수로 전달합니다. API가 없다면 [Google AI Studio](https://aistudio.google.com/prompts/new_chat))로 이동하여 API 키 받기를 클릭하여 사용 가능한 API를 생성하세요.

AI 에이전트 노드의 경우 그림 5.61과 같이 주로 `User Message` 및 `System Message` 섹션을 채워야 합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-10.png" alt="" width="90%"/>
<p>그림 5.61 AI 에이전트 노드 세부정보</p>
</div>

우리의 경우에 사용된 프롬프트는 다음과 같습니다.

```json
# Prompt (User Message)
# Context Information
- Current Time: {{ new Date().toLocaleString('en-AU', { timeZone: 'Australia/Sydney', hour12: false }) }} (Sydney, Australia time)
- Sender: {{ $json.From }}
- Subject: {{ $json.Subject }}
- Email Body: {{ $json.snippet }}

# System Message
# Role and Goal
You are a 24/7 on-call, professional and efficient AI email assistant. Your task is: to do your best to answer all questions in emails using public information at the first opportunity, and add contextual status reminders at the beginning of replies based on my work schedule.

# Context Information
- Current Time: {{ new Date().toLocaleString('en-AU', { timeZone: 'Australia/Sydney', hour12: false }) }} (Sydney, Australia time)
- Email information is in the input data.

# Available Tools
- Simple Vector Store2: Used to query my exact working hours (e.g., Monday to Friday, 9 AM to 5 PM).
- SerpAPI: **[Primary Information Source]** Prioritize using this tool to search the internet to answer specific questions in emails.

# Execution Steps
1.  **Analyze the Question**: First, carefully read the email content and extract the sender's core question.

2.  **Parallel Information Gathering**: Execute the following two operations simultaneously to collect information:
    a. Use the `SerpAPI` tool to search online for answers to the sender's questions.
    b. Use the `Simple Vector Store2` tool to get my set exact working hours.

3.  **Draft Core Reply**: Based on the information collected by `SerpAPI`, clearly and directly answer the sender's question. This part will serve as the main body of the email reply.

4.  **Add Status Prefix and Integrate**:
    a. Compare "Current Time" with the working hours I obtained from the tool.
    b. **If currently "Non-working Hours"**: Create a status reminder prefix. This prefix **must include** the specific working hours obtained from `Simple Vector Store2`.
        * **Prefix Example**: "Hello, thank you for your email. You have contacted me during my non-working hours (my working hours are: [insert queried working hours here]). I will personally review this email on the next working day. In the meantime, here is a preliminary reply found for you based on public information:**<br><br>---<br><br>**"
    c. **If currently "Working Hours"**: Just use a simple greeting.
        * **Prefix Example**: "Hello, regarding your question, the reply is as follows:**<br><br>---<br><br>**"
    d. Concatenate the generated prefix and the core reply you drafted (result of step 3) to form the final email body.

5.  **Formatted Output**: You must output the finally generated email content in a strict JSON format. The format is as follows, do not add any additional explanations or text:
    {
      "shouldReply": true,
      "subject": "Re: [Original Email Subject]",
      "body": "[Here is the concatenated, complete email reply body, **all line breaks must use HTML <br> tags**]"
    }

# Rules and Restrictions
- **Always Try to Answer First**: At any time, your primary task is to use `SerpAPI` to provide valuable replies to users.
- **Must Declare Status**: If replying during non-working hours, you must clearly state this at the beginning of the email and attach my exact working hours.
- **Information Sources Must Be Accurate**: Working hours must strictly follow the results of `Simple Vector Store2`; question answers mainly come from `SerpAPI`, do not fabricate information.
- **Output Format**: **In the final output JSON, all line breaks in the `body` field must use `<br>` tags, not `\n`.**
```

(3) 에이전트 도구 구성

`Simple Vector Store` 도구의 경우 이전에 저장한 지식을 올바르게 "읽을" 수 있도록 주요 구성을 수행해야 합니다.

- **작동 모드**: `Retrieve Documents (As Tool for AI Agent)` (도구로서의 읽기 모드).
- **메모리 키**: 첫 번째 부분과 **정확히 동일한** 키를 입력해야 합니다(예: `my_private_knowledge`).
- **임베딩**: 첫 번째 부분과 **완전히 동일한** `Embeddings Google Gemini` 모델을 사용해야 합니다.

`Memory Key` 및 `Embeddings` 모델이 완전히 일치하는 경우에만 에이전트는 그림 5.62에 표시된 것처럼 올바른 "키"와 "언어"를 사용하여 지식 기반에 액세스할 수 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-11.png" alt="" width="90%"/>
<p>그림 5.62 간단한 벡터 저장 도구 구성</p>
</div>

설명 매개변수는 AI 에이전트가 도구를 호출할 때 도구에 대한 설명 정의입니다. 해당 프롬프트는 다음과 같습니다.

```json
This is the Simple Vector Store2 tool, used to query my personal information, especially my working hours and email reply policy. When you need to determine whether it is currently working hours, or need to inform the other party when I will reply to emails, you must use this tool.
```

메모리의 경우 주목해야 할 유일한 점은 여기에서 스토리지 고유성을 보장하기 위해 각 사서함의 스레드 이름을 고유 식별자로 사용한다는 것입니다. 키가 `{{ $('Gmail').item.json.threadId }}`로 설정되어 있습니다.



(4) 최종 답변 보내기

마지막 단계는 실행입니다. `AI Agent` 노드의 출력을 `Gmail` 노드에 연결하고 **Operation**을 `Send`로 설정합니다. 그림 5.63과 같이 n8n 표현식을 사용하여 수신자, 제목 및 본문을 `AI Agent`에 의해 출력된 JSON 데이터의 해당 필드와 연결하여 자동 이메일 회신을 달성합니다.

- **받는 사람**: `{{ $('Gmail').item.json.From }}` (또는 다른 트리거의 보낸 사람 필드)
- **제목**: `Re:  {{ $('Gmail').item.json.Subject }}`
- **메시지**: `{{ $json.output }}`

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-12.png" alt="" width="90%"/>
<p>그림 5.63 최종 응답 도구 다이어그램</p>
</div>

그리고 전송이 성공하면 그림 5.64와 같이 개인 메일함으로 실제 회신 이메일 정보를 받을 수도 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-13.png" alt="" width="90%"/>
<p>그림 5.64 개인 사서함 반송 이메일 형식</p>
</div>

이로써 `AI Agent` 노드 기반의 통합 지능형 ​​고객 서비스가 완성된다. 테스트 이메일을 보내 작업 결과를 확인할 수 있습니다. 이 아키텍처는 매우 강력한 확장성을 갖고 있습니다. 앞으로는 `AI Agent` 노드에 더 많은 도구 (such as calendars, databases, CRM, etc.)를 직접 추가할 수 있습니다. 보다 강력한 기능으로 에이전트에 지속적으로 권한을 부여하려면 프롬프트에서 에이전트를 사용하는 방법만 가르치면 됩니다.

### 5.5.5 n8n의 장점과 한계 분석

지능형 이메일 도우미를 처음부터 구축하는 실습을 통해 우리는 n8n의 작업 모드에 대한 직관적인 이해를 얻었습니다. 강력한 로우 코드 자동화 플랫폼인 n8n은 에이전트 애플리케이션 개발을 지원하는 데 탁월한 성능을 발휘하지만 전능하지는 않습니다. 표 5.2와 같이 그 장점과 잠재적인 한계를 객관적으로 분석해 보겠습니다.

<div align="center">
<p>표 5.2 n8n 플랫폼의 장점과 한계 요약</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-14.png" alt="" width="90%"/>
</div>

첫째, n8n의 가장 큰 장점은 **개발 효율성**에 있습니다. 복잡한 논리를 직관적인 시각적 워크플로로 추상화합니다. 이메일 수신, AI 의사결정, 도구 호출, 최종 응답 등 전체 데이터 흐름과 처리 체인이 캔버스에서 한 눈에 명확하게 표시됩니다. 이러한 로우 코드 특성은 기술 임계값을 크게 낮추어 개발자가 에이전트의 핵심 로직을 신속하게 구축 및 검증할 수 있도록 하여 아이디어에서 프로토타입까지의 거리를 크게 단축시킵니다.

둘째, 플랫폼은 **강력하고 고도로 통합**되어 있습니다. n8n에는 Gmail 및 Google Gemini와 같은 수백 가지 일반 서비스를 쉽게 연결할 수 있는 풍부한 내장 노드 라이브러리가 있습니다. 더 중요한 것은 고급 `AI Agent` 노드가 모델, 메모리 및 도구 관리를 고도로 통합하여 하나의 노드로 복잡한 자율적 의사 결정을 구현할 수 있다는 것입니다. 이는 기존 다중 노드 수동 라우팅보다 훨씬 우아하고 강력합니다. 동시에 내장 기능이 처리할 수 없는 시나리오의 경우 `Code` 노드는 사용자 정의 코드를 작성할 수 있는 유연성을 제공하여 기능의 상한을 보장합니다.

마지막으로 **배포 및 운영** 수준에서 n8n은 **비공개 배포**를 지원하며 현재 프로젝트의 전체 버전을 배포할 수 있는 비교적 간단한 비공개 에이전트 솔루션입니다. 이는 데이터 보안과 개인 정보 보호를 중요하게 생각하는 기업에 매우 중요합니다. 내부 이메일 및 고객 데이터와 같은 민감한 정보가 우리 환경을 벗어나지 않도록 전체 서비스를 자체 서버에 배포하여 에이전트 애플리케이션의 규정 준수를 위한 견고한 기반을 제공할 수 있습니다.

물론 모든 도구에는 장단점이 있습니다. n8n이 제공하는 편리함을 즐기면서도 한계도 인식해야 합니다.

**개발 효율성** 뒤에는 **상대적으로 번거로운 디버깅 및 오류 처리**가 있습니다. 워크플로가 복잡해지면 데이터 형식 오류가 발생하면 개발자는 문제를 찾기 위해 각 노드의 입력과 출력을 하나씩 확인해야 할 수 있습니다. 이는 코드에 중단점을 설정하는 것만큼 직접적이지 않은 경우도 있습니다.

기능적인 측면에서 가장 큰 한계는 **내장 스토리지의 비지속성**에 반영됩니다. 이 경우에 사용한 `Simple Memory` 및 `Simple Vector Store`은 모두 메모리 기반이므로 n8n 서비스가 다시 시작되면 모든 대화 기록과 지식 기반이 손실됩니다. 이는 프로덕션 환경 애플리케이션에 치명적입니다. 따라서 실제 배포에서는 Redis, Pinecone 등 외부 영구 데이터베이스로 교체해야 하며, 이로 인해 추가 구성 및 유지 관리 비용도 증가합니다.

또한 **배포 및 운영** 및 팀 협업 측면에서 n8n의 **버전 제어 및 다중 사용자 협업은 기존 코드**만큼 성숙하지 않습니다. 관리를 위해 워크플로를 JSON 파일로 내보낼 수 있지만 변경 사항을 비교하는 것은 `git diff` 코드보다 훨씬 명확하지 않으며 여러 사람이 동시에 동일한 워크플로를 편집하면 쉽게 충돌이 발생할 수 있습니다.

마지막으로 **성능**과 관련하여 n8n은 대부분의 기업 자동화 및 중간에서 낮은 빈도의 에이전트 작업을 완벽하게 충족할 수 있습니다. 그러나 매우 높은 동시 요청을 처리해야 하는 시나리오의 경우 노드 예약 메커니즘은 특정 성능 오버헤드를 가져올 수 있으며 이는 순수 코드로 구현된 서비스보다 약간 열등할 수 있습니다.

## 5.6 장 요약

이 장에서는 "손으로 작성한 코드"에서 "플랫폼 기반 개발"로의 중요한 전환을 표시하면서 로우 코드 플랫폼을 기반으로 에이전트 애플리케이션을 구축하는 개념, 방법 및 사례를 체계적으로 소개합니다.

첫 번째 섹션에서는 로우코드 플랫폼이 부상하게 된 배경과 가치에 대해 자세히 설명했습니다. 4장의 순수 코드 구현 에이전트와 비교하여 로우 코드 플랫폼은 기술 임계값을 크게 낮추고, 개발 효율성을 향상시키며, 그래픽 및 모듈식 접근 방식을 통해 더 나은 시각적 디버깅 경험을 제공합니다. 이 "더 높은 수준의 추상화"를 통해 개발자는 기본 구현 세부 사항보다는 비즈니스 논리와 신속한 엔지니어링에 에너지를 집중할 수 있습니다.

이후 우리는 네 가지 대표적인 대표 플랫폼을 심도 깊게 실천했습니다.

**Coze**는 코드가 필요 없는 환경과 풍부한 플러그인 생태계로 두각을 나타냅니다. 'Daily AI Brief' 사례를 통해 드래그 앤 드롭 구성을 통해 멀티 소스 정보를 빠르게 통합하고, 한 번의 클릭으로 여러 주류 플랫폼에 게시하는 방법을 경험했습니다. Coze는 기술적인 배경 지식이 없는 배경 사용자와 아이디어를 신속하게 검증해야 하는 시나리오에 특히 적합하지만 MCP를 지원하지 않는다는 한계와 표준화된 구성 파일을 내보낼 수 없다는 점도 주목할 가치가 있습니다.

**Dify**는 오픈소스 엔터프라이즈급 플랫폼으로 풀스택 개발 역량을 보여줍니다. "슈퍼 에이전트 개인 비서" 사례는 일일 Q&A, 카피라이팅 최적화, 다중 모달 생성, 데이터 분석 및 MCP 도구 통합과 같은 여러 모듈을 다루며 복잡한 비즈니스 시나리오에서 Dify의 강력한 오케스트레이션 기능을 완벽하게 보여줍니다. 풍부한 플러그인 시장(8000+), 유연한 배포 방법 및 엔터프라이즈 수준의 보안 기능을 통해 전문 개발자 및 엔터프라이즈 팀에게 이상적인 선택입니다. 그러나 동시성이 높은 시나리오에서는 상대적으로 가파른 학습 곡선과 성능 문제도 고려해야 합니다.

**FastGPT**는 최고의 RAG 지식 기반 경험으로 두각을 나타내며 수직 도메인 Q&A 시나리오에서 강력한 경쟁자가 되었습니다. "Smart Investment Advisor Assistant" 사례를 통해 지식 기반 구축, MCP 도구 통합, 시각적 워크플로우 조정까지 전체 개발 패러다임을 경험했습니다. 파일 청크, 색인 향상 및 이미지 인식에 대한 FastGPT의 세밀한 제어는 기업 지식 도우미 및 지능형 고객 서비스 시나리오에서 고유한 이점을 제공합니다. 그러나 상대적으로 약한 템플릿 생태계와 제한된 무료 버전 할당량으로 인해 더 복잡한 비즈니스 시나리오에서는 성능이 제한됩니다.

**n8n**은 고유한 "연결" 기능으로 또 다른 경로를 열어줍니다. "지능형 이메일 도우미" 사례를 통해 AI 기능을 복잡한 비즈니스 자동화 프로세스에 원활하게 포함시키는 방법을 살펴보았습니다. n8n의 AI 에이전트 노드는 모델, 메모리 및 도구를 고도로 통합하고 수백 개의 사전 설정 노드와 결합하여 고도로 맞춤화된 자동화 솔루션을 달성할 수 있습니다. 개인 배포에 대한 지원은 데이터 보안을 중시하는 기업에 특히 중요합니다. 그러나 내장 스토리지의 비지속성과 버전 관리의 미성숙으로 인해 프로덕션 환경에서는 추가적인 엔지니어링 처리가 필요합니다.

네 가지 플랫폼의 비교 실습을 통해 다음과 같은 선택 제안을 도출할 수 있습니다.
- **신속한 프로토타입 검증, 비기술 사용자**: Coze 우선순위
- **엔터프라이즈급 애플리케이션, 복잡한 비즈니스 로직, 멀티모달 생성**: Dify 우선순위 지정
- **개인 지식 기반 기반의 Q&A 시스템, 지능형 고객 서비스**: FastGPT 우선 순위 지정
- **심층적인 비즈니스 통합, 일반 자동화 프로세스**: n8n 우선순위 지정

로우코드 플랫폼은 코드 개발을 대체하는 것이 아니라 보완적인 선택을 제공한다는 점을 강조할 가치가 있습니다. 실제 프로젝트에서는 다양한 단계의 요구에 따라 유연하게 전환할 수 있습니다. 로우 코드 플랫폼을 사용하여 아이디어를 신속하게 검증하고, 코드를 사용하여 세밀한 제어를 달성합니다. 플랫폼을 사용하여 표준화된 프로세스를 처리하고 코드를 사용하여 특수 논리를 처리합니다. 이러한 "하이브리드 개발" 사고방식은 에이전트 엔지니어링의 모범 사례입니다.

다음 장에서는 독자가 보다 안정적이고 흥미로운 애플리케이션을 구축하는 데 도움이 되는 더 많은 기본 에이전트 프레임워크를 자세히 살펴보겠습니다.


## 연습

1. 이 장에서는 `Coze`, `Dify`, `FastGPT` 및 `n8n`이라는 네 가지 독특한 로우 코드 플랫폼을 소개합니다. 분석해 주십시오:

- 네 가지 플랫폼의 핵심 포지셔닝과 디자인 철학의 차이점은 무엇인가요? 에이전트 개발의 어떤 문제점을 각각 해결합니까?
   - 로우코드 플랫폼과 순수코드 개발은 각각 장점과 단점이 있습니다. 또한 일부 기능은 플랫폼을 사용하여 일부는 코드를 사용하여 구현되는 "하이브리드 개발" 모드도 있습니다. 세 가지 개발 모드가 각각 어떤 시나리오에 적합한지 생각해 보세요. 예를 들어주세요.

2. 섹션 5.2의 `Coze` 사례에서는 "Daily AI Brief" 에이전트를 구축했습니다. 이 사례를 바탕으로 생각을 확장해 보십시오.

> **팁**: 실습 문제이므로 실제 작동을 권장합니다.

- 현재 간략한 생성은 수동적으로 실행됩니다(사용자가 적극적으로 요청함). 자동으로 브리핑을 생성하고 매일 오전 8시에 지정된 Feishu 그룹 또는 WeChat 공식 계정으로 푸시할 수 있도록 이 에이전트를 변환하는 방법은 무엇입니까?
   - 신속한 디자인에 따라 브리핑의 질이 크게 좌우됩니다. 섹션 5.2.2의 프롬프트를 최적화하여 생성된 브리핑을 보다 전문적이고 명확한 구조로 만들거나 "핫스팟 분석" 및 "경향 예측"과 같은 새로운 기능을 추가해 보십시오.
   - `Coze`은 현재 `MCP` 프로토콜을 지원하지 않는 것이 중요한 제한 사항으로 간주됩니다(연습 작성 중에 `feature-mcp`는 [`Coze Studio Q4 2025 Product Roadmap`]에 있지만(https://github.com/coze-dev/coze-studio/issues/2218),은 아직 구현되지 않았습니다). `MCP` 프로토콜이 무엇인지 간략하게 설명해주세요. 왜 중요합니까? `Coze`가 지원하는 경우 `MCP` 미래에는 어떤 새로운 가능성을 가져올까요?

3. 섹션 5.3의 `Dify` 사례에서는 완전한 기능을 갖춘 "슈퍼 에이전트 개인 비서"를 구축했습니다. 심층적으로 분석해 보세요.

- 이 사례는 지능형 라우팅을 위해 "질문 분류자"를 사용하여 다양한 하위 에이전트에 다양한 유형의 요청을 배포합니다. 이 다중 에이전트 아키텍처의 장점은 무엇입니까? 분류자를 사용하지 않고 단일 에이전트가 모든 작업을 처리하게 하면 어떤 문제가 발생합니까?
   - 데이터 쿼리 모듈은 대형 모델에 명확한 테이블 구조 정보를 제공해야 합니다. 데이터베이스에 50개의 테이블이 있고 각각 20개의 필드가 있는 경우 모든 `DDL` 문을 프롬프트에 직접 입력하면 컨텍스트가 너무 길어집니다. 이 문제를 해결하기 위해 더 스마트한 솔루션을 설계하시기 바랍니다.
   - `Dify`는 로컬 배포와 클라우드 배포 모드를 모두 지원합니다. 데이터 보안, 비용, 성능 및 유지 관리 난이도 측면에서 두 가지 모드의 차이점을 비교하고 각각에 적용 가능한 시나리오를 설명하십시오.

4. 5.4절의 `FastGPT` 사례에서는 '스마트투자상담사보조'를 구축하였습니다. 심층적으로 분석해 보세요.

- FastGPT의 핵심 장점은 RAG 파이프라인의 심층적인 최적화입니다. FastGPT의 지식 기반 처리(파일 청크, 색인 향상, 이미지 인식)와 Dify의 지식 기반 기능을 비교해 보세요. 둘 사이의 디자인 철학과 적용 가능한 시나리오의 차이점은 무엇입니까?
   - 이 사례는 MCP 도구를 사용하여 실시간 주식 데이터를 얻고 시각적 차트를 생성합니다. FastGPT가 기본적으로 MCP를 지원하지 않는 경우 동일한 기능을 어떻게 달성할 수 있습니까? 대체 솔루션을 제안해주세요.
   - FastGPT 무료 버전에는 100크레딧과 30QPM만 있습니다. 1000명의 사용자에게 서비스를 제공해야 하는 스타트업 팀의 경우 비용과 성능의 균형을 맞추는 솔루션을 어떻게 설계하시겠습니까?

5. 섹션 5.5의 `n8n` 사례에서는 "지능형 이메일 도우미"를 구축했습니다. 다음 질문에 대해 생각해 보십시오.

> **팁**: 실습 문제이므로 실제 작동을 권장합니다.

- 해당 사례에 사용된 `Simple Vector Store`, `Simple Memory`은 모두 메모리 기반이므로 서비스 재시작 후 데이터가 손실됩니다. `n8n` 문서를 참조하여 영구 저장소 솔루션 (such as ⟪P3⟫, ⟪P4⟫, etc.)으로 교체해 보고 구성 프로세스를 설명하세요.
   - 현재 이메일 도우미는 텍스트 이메일만 처리할 수 있습니다. 사용자가 보낸 이메일에 첨부 파일(예: `PDF` 문서, 이미지)이 포함되어 있는 경우 상담원이 첨부 파일 내용을 이해하고 해당 답변을 할 수 있도록 이 워크플로를 어떻게 확장하시겠습니까?
   - `n8n`의 핵심 장점은 '연결' 기능에 있습니다. 보다 복잡한 자동화 시나리오를 설계하십시오. 고객이 전자 상거래 플랫폼에서 주문하면 일련의 작업(확인 이메일 보내기, 재고 데이터베이스 업데이트, 물류 시스템 알림, `CRM`에 고객 정보 기록)이 자동으로 실행됩니다. 워크플로의 노드 연결 다이어그램을 그리고 주요 구성을 설명해주세요.

6. 신속한 엔지니어링은 로우코드 플랫폼에서도 마찬가지로 중요합니다. 이 장에서는 다양한 플랫폼 프롬프트 디자인 사례를 보여줍니다. 분석해 주십시오:

- 섹션 5.2.2(`Coze`), 섹션 5.3.2(`Dify`), 섹션 5.4.2(`FastGPT`) 및 섹션 5.5.4(`n8n`)의 프롬프트 디자인을 비교하세요. 구조, 스타일, 초점의 차이점은 무엇입니까? 이러한 차이점은 플랫폼 특성과 관련이 있습니까?
   - `Dify`의 "카피라이팅 최적화 모듈"에서 프롬프트는 "500 단어 초과" 출력을 요구합니다. 출력 길이에 대한 이러한 엄격한 요구 사항이 합리적입니까? 어떤 상황에서 출력 길이를 제한해야 하며, 어떤 상황에서 모델이 자유롭게 표현하도록 허용해야 합니까?

7. 도구와 플러그인은 로우코드 플랫폼의 핵심 기능 확장 방법입니다. 생각해 보십시오:

- `Coze`에는 풍부한 플러그인 스토어가 있고, `Dify`에는 8000개 이상의 플러그인 시장이 있으며, `FastGPT`는 기본적으로 MCP 프로토콜을 지원하고, `n8n`에는 수백 개의 사전 설정 노드가 있습니다. 이 네 가지 플랫폼 중 어느 것도 필요한 특정 도구(예: "회사 내부 시스템에 연결 `API`")가 없는 경우 어떻게 해결하시겠습니까?
   - 섹션 5.3.2에서는 `MCP` 프로토콜을 사용하여 Amap 및 식단 추천과 같은 서비스를 통합했습니다. 조사하고 설명해 주세요. `MCP` 프로토콜과 기존 `RESTful API` 및 `Tool Calling` 프로토콜의 차이점은 무엇인가요? `MCP`가 에이전트 도구 호출의 "새로운 표준"이라고 불리는 이유는 무엇입니까?
   - 회사 내부 지식 기반 시스템을 호출할 수 있도록 `Dify`용 사용자 정의 플러그인을 개발한다고 가정해 보겠습니다. `Dify`의 플러그인 개발 문서를 참조하고 개발 프로세스와 주요 기술 사항을 간략하게 설명하세요.

8. 플랫폼 선택은 에이전트 제품의 성공을 위한 주요 결정 중 하나입니다. 당신이 스타트업 회사의 기술 리더이고 회사가 다음 세 가지 AI 애플리케이션을 개발할 계획이라고 가정해 보겠습니다. 각 애플리케이션(`Coze`, `Dify`, `FastGPT`, `n8n` 또는 순수 코드 개발)에 가장 적합한 플랫폼을 선택하고 자세히 설명해주세요.

**애플리케이션 A**: C-end 사용자를 위한 "AI Writing Assistant" 미니 프로그램은 제한된 예산으로 시장 수요를 확인하기 위해 신속하게 출시되어야 하며 팀에는 프런트 엔드 엔지니어 1명과 제품 관리자 1명만 있습니다.

**애플리케이션 B**: 기업 고객을 위한 "지능형 계약 검토 시스템"은 민감한 법률 문서를 처리해야 하고, 데이터가 고객의 개인 환경을 벗어날 수 없어야 하며, 고객의 기존 OA 시스템 및 문서 관리 시스템과의 긴밀한 통합이 필요합니다.

**애플리케이션 C**: 내부 "R&D 효율성 개선 도구"는 코드 검토, 테스트 보고서 생성, 버그 추적 및 프로젝트 진행 동기화와 같은 여러 R&D 프로세스 링크를 자동화해야 합니다. 팀은 강력한 기술 역량을 보유하고 있습니다.

각 애플리케이션에 대해 다음 차원을 분석하십시오(포함하되 이에 국한되지 않음).

> **팁**: 플랫폼 기능이 요구 사항을 충족하는지 여부, 출시 속도, 개발 비용, 운영 비용, 후속 반복의 어려움, 향후 기능 확장을 위한 공간

- 기술적 타당성
   - 개발 효율성
   - 비용 관리
   - 유지보수성
   - 확장성
   - 데이터 보안 및 규정 준수

## 참고자료

[1] 코제(Coze) - 차세대 AI 애플리케이션 개발 플랫폼. https://www.coze.cn/

[2] Dify - 오픈 소스 LLM 애플리케이션 개발 플랫폼입니다. https://dify.ai/

[3] FastGPT - 오픈 소스 지식 기반 Q&A 플랫폼 및 에이전트 구축 도구입니다. https://fastgpt.io/en/

[4] n8n - 워크플로 자동화 도구입니다. https://n8n.io/

