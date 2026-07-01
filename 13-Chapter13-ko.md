# 13장 지능형 여행 도우미

이전 장에서는 다양한 에이전트 패러다임, 도구 시스템, 메모리 메커니즘, 프로토콜 통신 및 성능 평가를 포함한 핵심 기능을 구현하여 HelloAgents 프레임워크를 처음부터 구축했습니다. 이 장부터 우리는 완전히 새로운 단계에 진입하게 됩니다. **학습된 모든 지식을 통합하여 완전한 실제 응용 프로그램을 구축합니다.**

1장에서 우리가 만든 첫 번째 에이전트를 기억하시나요? `Thought-Action-Observation` 루프의 기본 원리를 보여주는 간단한 지능형 여행 도우미였습니다. 이 장의 지능형 여행 도우미는 다음 핵심 기능을 포함하는 완전한 프로젝트입니다.

**(1) 지능형 여행 일정 계획**: 사용자가 목적지, 날짜, 선호 사항 및 기타 정보를 입력하면 시스템이 명소, 식사, 호텔을 포함한 완전한 여행 일정을 자동으로 생성합니다.

**(2) 지도 시각화**: 지도에 명소 위치를 표시하고 여행 경로를 그려 여행 일정을 한눈에 알 수 있습니다.

**(3) 예산 계산**: 티켓, 호텔, 식사, 교통 비용을 자동으로 계산하여 예산 세부 정보를 표시합니다.

**(4) 일정 편집**: 명소 추가, 삭제, 조정을 지원하고 지도를 실시간으로 업데이트합니다.

**(5) 내보내기 기능**: PDF 또는 이미지로 내보내기를 지원하여 저장 및 공유에 편리합니다.

## 13.1 프로젝트 개요 및 아키텍처 설계

13.1.1 지능형 여행 도우미가 필요한 이유

여행을 계획하는 것은 흥미롭기도 하고 답답하기도 합니다. 온라인으로 명소 정보를 검색하고, 다양한 가이드를 비교하고, 일기예보를 확인하고, 호텔을 예약하고, 예산을 계산하고, 경로를 계획해야 합니다. 이 프로세스는 몇 시간 또는 며칠이 걸릴 수 있습니다. 그리고 그렇게 많은 시간을 보낸 후에도 계획한 여행 일정이 합리적인지, 중요한 명소를 놓쳤는지, 예산이 정확한지 확신할 수 없습니다.

전통적인 여행 계획 방법에는 몇 가지 문제점이 있습니다. 첫 번째는 **흩어진 정보**입니다. 명소 정보는 여행 웹사이트에 있고, 날씨 정보는 날씨 웹사이트에 있고, 호텔 정보는 예약 웹사이트에 있습니다. 여러 웹사이트 간에 전환하고 이 정보를 수동으로 통합해야 합니다. 두 번째는 **개인화 부족**입니다. 대부분의 가이드는 일반적이며 개인 취향, 예산 제약, 이동 시간 및 기타 요소를 고려하지 않습니다. 마지막으로 **조정의 어려움**입니다. 여행 일정을 수정하고 싶을 때 관광지 순서, 시간 배치, 예산 등이 모두 서로 연결되어 있기 때문에 여행 전체를 다시 계획해야 할 수도 있습니다.

AI 기술은 이러한 문제를 해결할 수 있는 새로운 가능성을 제시합니다. 시스템에 "역사와 문화, 중간 예산 등 3일 동안 베이징을 방문하고 싶습니다"라고 말하면 시스템이 매일 방문할 명소, 식사할 곳, 숙박할 호텔, 예산이 얼마나 필요한지 등을 포함한 완전한 여행 일정을 자동으로 생성할 수 있다고 상상해 보세요. 또한, 이 계획은 조정 가능합니다. 마음에 들지 않는 명소를 삭제하고 투어 순서를 조정할 수 있으며 시스템이 자동으로 지도와 예산을 업데이트합니다.

이것이 우리가 만들고 싶은 지능형 여행 도우미입니다. 단순한 기술 시연이 아니라 정말 유용한 애플리케이션입니다. 본 프로젝트를 통해 실제 문제에 AI 기술을 적용하는 방법, 멀티 에이전트 시스템을 설계하는 방법, 완전한 웹 애플리케이션을 구축하는 방법을 배우게 됩니다.

### 13.1.2 기술 아키텍처 개요

시스템은 그림 13.1과 같이 4개 계층으로 구분된 고전적인 **프런트엔드 및 백엔드 분리 아키텍처**를 채택합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-1.png" alt="" width="85%"/>
<p>그림 13.1 지능형 여행 보조 기술 아키텍처</p>
</div>

**(1) 프런트 엔드 레이어(Vue3+TypeScript)**: 양식 입력, 결과 표시 및 지도 시각화를 포함한 사용자 상호 작용 및 데이터 표시를 담당합니다.

**(2) 백엔드 계층(FastAPI)**: API 라우팅, 데이터 검증 및 비즈니스 로직을 담당합니다.

**(3) 에이전트 계층(HelloAgents)**: 작업 분해, 도구 호출 및 결과 통합을 담당합니다. 전문 요원 4명이 포함되어 있습니다.

**(4) 외부 서비스 계층**: Amap API, Unsplash API 및 LLM API를 포함한 데이터 및 기능을 제공합니다.

데이터 흐름 프로세스는 다음과 같습니다. 사용자가 프런트엔드에서 양식 작성 → 백엔드에서 데이터 검증 → 에이전트 시스템 호출 → 에이전트가 관광지 검색, 날씨 쿼리, 호텔 추천, 여행 일정 계획 순차 호출 에이전트 → 각 에이전트가 MCP 프로토콜을 통해 외부 API 호출 → 결과 통합 및 프런트엔드로 복귀 → 프런트엔드 렌더링 및 표시.

프로젝트 구조 참조는 다음과 같으며 소스 코드를 쉽게 찾을 수 있도록 제공됩니다.
```
helloagents-trip-planner/
├── backend/                    # Backend code
│   ├── app/
│   │   ├── agents/            # Agent implementation
│   │   ├── api/               # API routes
│   │   ├── models/            # Data models
│   │   ├── services/          # Service layer
│   │   └── config.py          # Configuration file
│   └── requirements.txt       # Python dependencies
│
└── frontend/                   # Frontend code
    ├── src/
    │   ├── views/             # Page components
    │   ├── services/          # API services
    │   ├── types/             # Type definitions
    │   └── router/            # Route configuration
    └── package.json           # npm dependencies
```

자세한 아키텍처 설계 및 데이터 흐름은 후속 섹션에서 소개됩니다.

### 13.1.3 빠른 경험: 5분 안에 프로젝트 실행

구현 세부 사항을 살펴보기 전에 먼저 프로젝트를 실행하여 최종 효과를 살펴보겠습니다. 이렇게 하면 전체 시스템을 직관적으로 이해할 수 있습니다.

**환경 요구 사항:**

- 파이썬 3.10 이상
- Node.js 16.0 이상
- npm 8.0 이상

**API 키 얻기:**

다음 API 키를 준비해야 합니다.

- LLM API (OpenAI, DeepSeek, etc.)
- Amap 웹 서비스 키: https://console.amap.com/를 방문하여 애플리케이션을 등록하고 생성하세요.
- Unsplash 액세스 키: https://unsplash.com/developers를 방문하여 애플리케이션을 등록하고 생성하세요.

모든 API 키를 `.env` 파일에 넣으세요.

백엔드를 시작합니다.

```bash
# 1. Enter backend directory
cd helloagents-trip-planner/backend

# 2. Install dependencies
pip install -r requirements.txt

# 3. Configure environment variables
cp .env.example .env
# Edit .env file, fill in your API keys

# 4. Start backend service
uvicorn app.api.main:app --reload
# or
python run.py
```

성공적으로 시작한 후 http://localhost:8000/docs를 방문하여 API 문서를 확인하세요.

새 터미널 창을 엽니다.

```bash
# 1. Enter frontend directory
cd helloagents-trip-planner/frontend

# 2. Install dependencies
npm install

# 3. Start frontend service
npm run dev
```

성공적으로 시작되면 http://localhost:5173를 방문하여 애플리케이션을 사용하세요.

핵심 기능을 경험해보세요:

먼저 홈페이지 양식에 목적지 도시, 여행 날짜, 선호도, 예산, 교통수단, 숙박 유형을 입력합니다. "계획 시작" 버튼을 클릭하면 시스템은 로딩 진행률 표시줄을 표시하고 그림 13.2와 같이 결과 페이지를 빠르게 생성합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-2.png" alt="" width="85%"/>
<p>그림 13.2 여행 도우미 계획 진행 페이지</p>
</div>

로드가 성공적으로 완료되면 페이지에는 그림 13.3 및 13.4와 같이 여행 일정 개요, 예산 세부정보, 명소 지도, 일일 여행 일정 세부정보 및 날씨 정보가 명확하게 표시됩니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-3.png" alt="" width="85%"/>
<p>그림 13.3 여행 도우미 계획 완료 페이지</p>
</div>

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-4.png" alt="" width="85%"/>
<p>그림 13.4 여행 도우미 계획 완료 페이지</p>
</div>

사용자가 개인화된 조정이 필요한 경우 "여행 일정 편집" 버튼을 클릭하여 그림 13.5와 같이 명소의 순서를 자유롭게 조정하거나 특정 명소를 삭제할 수 있습니다. 계획이 완료된 후 "일정 내보내기" 드롭다운 메뉴를 통해 최종 계획을 이미지나 PDF 파일로 쉽게 저장할 수 있어 언제든지 편리하게 참조할 수 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-5.png" alt="" width="85%"/>
<p>그림 13.5 여행 도우미 계획 완료 페이지</p>
</div>

## 13.2 데이터 모델 설계

### 13.2.1 웹 애플리케이션의 데이터 흐름

지능형 여행 도우미를 구축할 때 핵심 문제인 **여행 계획 데이터를 어떻게 표현하고 전송합니까?**를 해결해야 합니다.

우리는 완전한 웹 애플리케이션에서 데이터가 어떻게 흐르는지 이해해야 합니다. 사용자가 브라우저에서 "계획 시작" 버튼을 클릭하면 어떤 일이 일어날지 상상해 보십시오.

프런트엔드 (destination, dates, budget, etc.)에서 사용자가 작성한 양식 데이터는 HTTP 요청을 통해 백엔드 서버로 전송되어야 합니다. 백엔드가 데이터를 수신한 후 처리를 위해 에이전트 시스템을 호출합니다. 그런 다음 에이전트는 Amap API 및 Unsplash API와 같은 외부 서비스를 호출하여 데이터를 얻습니다. 이러한 외부 API에서 반환되는 데이터 형식은 다릅니다. 일부는 `lng`을 사용하고 일부는 `lon`을 사용하며 일부는 `longitude`을 사용합니다. 마지막으로 백엔드는 처리된 데이터를 프런트엔드로 반환해야 하며, 프런트엔드는 이를 사용자에게 표시되는 페이지로 렌더링합니다.

이 프로세스에서 데이터는 프런트엔드 양식 → HTTP 요청 → 백엔드 Python 개체 → 외부 API 응답 → 백엔드 Python 개체 → HTTP 응답 → 프런트엔드 TypeScript 개체 → 페이지 표시 등 여러 가지 변환을 거칩니다. 통합된 데이터 형식이 없으면 각 변환 단계가 잘못될 수 있습니다. 이것이 바로 **데이터 모델**이 필요한 이유입니다.

### 13.2.2 딕셔너리에서 피단틱 모델로

1장의 간단한 프로토타입부터 시작하겠습니다. 해당 프로토타입에서는 Python 사전을 사용하여 명소 데이터를 표현했습니다.

```python
# Chapter 1 approach: using dictionaries
attraction = {
    "name": "Forbidden City",
    "location": {"lng": 116.397128, "lat": 39.916527},
    "price": 60
}

# Access data
lng = attraction["location"]["lng"]
```

이 접근 방식은 프로토타입 단계에서는 편리하지만 실제 프로젝트에서는 많은 문제에 직면하게 됩니다. 첫 번째는 **일관되지 않은 필드 이름** 문제입니다. Amap API에서 반환된 위치 데이터는 `"116.397128,39.916527"`와 같은 문자열이며 수동으로 경도와 위도로 분할해야 합니다. Unsplash API는 `longitude` 및 `latitude`를 사용할 수 있습니다. 코드의 모든 곳에서 사전을 사용한다면 모든 곳에서 이러한 차이점을 처리해야 합니다.

두 번째는 **타입 안전성** 문제입니다. 실수로 `price`를 문자열 `"60"`로 설정했다고 가정해 보겠습니다. 이는 Python에서 즉시 오류가 발생하지는 않지만 총 예산을 계산할 때 문제를 일으킬 것입니다. 게다가 이러한 종류의 오류는 런타임에만 발견될 수 있으며 오류 메시지를 찾기 어려울 수도 있습니다.

마지막으로 **유지보수성** 문제입니다. 명소에 새 필드(예: `rating`)를 추가해야 하는 경우 코드의 여러 위치를 수정해야 합니다. 어딘가를 놓치면 데이터 불일치가 발생합니다.

Pydantic이 솔루션을 제공합니다. 클래스를 사용하여 데이터 구조를 정의하고 유효성 검사, 변환 및 직렬화를 자동으로 처리할 수 있는 Python 데이터 유효성 검사 라이브러리입니다. 간단한 예를 살펴보겠습니다.

```python
from pydantic import BaseModel, Field

class Location(BaseModel):
    longitude: float = Field(..., description="Longitude")
    latitude: float = Field(..., description="Latitude")

class Attraction(BaseModel):
    name: str
    location: Location
    ticket_price: int = 0

# Create object
attraction = Attraction(
    name="Forbidden City",
    location=Location(longitude=116.397128, latitude=39.916527),
    ticket_price=60
)

# Type-safe access
lng = attraction.location.longitude  # IDE will provide code completion
```

이 접근 방식에는 여러 가지 이점이 있습니다. 첫째, 잘못된 유형을 전달하는 경우(예: `ticket_price`을 문자열로 설정), Pydantic은 즉시 오류가 있는 위치를 알려주는 예외를 발생시킵니다. 둘째, IDE는 유형 정의를 기반으로 코드 완성 및 유형 검사를 제공하여 철자 오류를 크게 줄일 수 있습니다. 마지막으로 데이터 구조를 수정해야 할 때 클래스 정의만 수정하면 이 클래스를 사용하는 모든 장소가 자동으로 업데이트됩니다.

### 13.2.3 Pydantic의 핵심 개념

데이터 모델 설계를 시작하기 전에 먼저 Pydantic의 몇 가지 핵심 개념을 이해해 보겠습니다. Pydantic의 기초는 `BaseModel` 클래스이며 모든 데이터 모델은 이 클래스에서 상속되어야 합니다. 각 필드는 유형을 지정할 수 있으며 Pydantic은 자동으로 유형 확인 및 변환을 수행합니다.

필드 정의는 기본값, 설명, 유효성 검사 규칙 등을 지정할 수 있는 `Field` 함수를 사용합니다. `...`는 이 필드가 필수임을 나타냅니다. 객체를 생성할 때 이 필드가 제공되지 않으면 Pydantic은 예외를 발생시킵니다. `Optional`를 사용하여 선택적 필드를 표시하거나 기본값을 직접 제공할 수도 있습니다.

```python
from pydantic import BaseModel, Field
from typing import Optional, List

class Attraction(BaseModel):
    name: str = Field(..., description="Attraction name")  # Required
    rating: float = Field(default=0.0, ge=0, le=5)  # Default value, range validation
    visit_duration: int = Field(default=60, gt=0)  # Greater than 0
    description: Optional[str] = None  # Optional field
```

Pydantic은 중첩 모델과 목록도 지원합니다. 한 모델의 필드 유형으로 다른 모델을 사용할 수 있으므로 복잡한 데이터 구조를 구축할 수 있습니다. 예를 들어 관광명소에는 위치 정보가 포함되어 있고 여행 일정에는 여러 관광명소가 포함되어 있습니다.

```python
class DayPlan(BaseModel):
    date: str
    attractions: List[Attraction]  # Attraction list
    hotel: Optional[Hotel] = None  # Optional hotel information
```

가장 강력한 기능 중 하나는 **맞춤형 유효성 검사기**입니다. 때로는 외부 API에서 반환된 데이터 형식이 요구 사항을 충족하지 않는 경우 `field_validator` 데코레이터를 사용하여 유효성 검사 및 변환 논리를 맞춤설정할 수 있습니다. 예를 들어, Amap이 반환한 온도는 `"16°C"`과 같은 문자열이므로 이를 숫자로 변환해야 합니다.

```python
from pydantic import field_validator

class WeatherInfo(BaseModel):
    temperature: int

    @field_validator('temperature', mode='before')
    def parse_temperature(cls, v):
        """Parse temperature string: "16°C" -> 16"""
        if isinstance(v, str):
            v = v.replace('°C', '').replace('℃', '').strip()
            return int(v)
        return v
```

이 유효성 검사기는 객체를 생성하기 전에 자동으로 실행되어 문자열을 정수로 변환합니다. 이렇게 하면 코드의 모든 위치에서 온도 형식을 수동으로 처리할 필요가 없습니다.

### 13.2.4 상향식 모델 설계

이제 지능형 여행 도우미를 위한 데이터 모델 설계를 시작하겠습니다. 좋은 설계 원칙은 **상향식**입니다. 먼저 가장 기본적인 모델을 정의한 다음 점차적으로 복잡한 구조로 결합합니다. 이 접근 방식의 장점은 각 모델이 간단하고 이해 및 유지 관리가 쉽다는 것입니다.

가장 기본적인 모델은 **위치정보**입니다. 관광지든, 호텔이든, 레스토랑이든 모두 위치 정보가 필요합니다. 경도와 위도 좌표를 나타내기 위해 `Location` 클래스를 정의합니다.

```python
class Location(BaseModel):
    """Location information (longitude and latitude coordinates)"""
    longitude: float = Field(..., description="Longitude", ge=-180, le=180)
    latitude: float = Field(..., description="Latitude", ge=-90, le=90)
```

여기서는 범위 확인(`ge`은 크거나 같음을 의미, `le`은 작거나 같음을 의미)을 사용하여 경도 및 위도 값이 합리적인 범위 내에 있는지 확인합니다.

다음은 **명소정보**입니다. 명소에는 이름, 주소, 위치, 방문 시간, 설명, 평점, 이미지, 티켓 가격 정보가 포함됩니다. 중첩 모델인 필드 유형으로 `Location`을 사용합니다.

```python
class Attraction(BaseModel):
    """Attraction information"""
    name: str = Field(..., description="Attraction name")
    address: str = Field(..., description="Address")
    location: Location = Field(..., description="Longitude and latitude coordinates")
    visit_duration: int = Field(..., description="Recommended visit duration (minutes)", gt=0)
    description: str = Field(..., description="Attraction description")
    category: Optional[str] = Field(default="Attraction", description="Attraction category")
    rating: Optional[float] = Field(default=None, ge=0, le=5, description="Rating")
    image_url: Optional[str] = Field(default=None, description="Image URL")
    ticket_price: int = Field(default=0, ge=0, description="Ticket price (yuan)")
```

마찬가지로 **식사 정보** 및 **호텔 정보**를 정의합니다. 이러한 모델은 모두 이름, 주소, 위치, 비용과 같은 기본 정보를 포함하는 비슷한 구조를 가지고 있습니다.

```python
class Meal(BaseModel):
    """Meal information"""
    type: str = Field(..., description="Meal type: breakfast/lunch/dinner/snack")
    name: str = Field(..., description="Meal name")
    address: Optional[str] = Field(default=None, description="Address")
    location: Optional[Location] = Field(default=None, description="Longitude and latitude coordinates")
    description: Optional[str] = Field(default=None, description="Description")
    estimated_cost: int = Field(default=0, description="Estimated cost (yuan)")

class Hotel(BaseModel):
    """Hotel information"""
    name: str = Field(..., description="Hotel name")
    address: str = Field(default="", description="Hotel address")
    location: Optional[Location] = Field(default=None, description="Hotel location")
    price_range: str = Field(default="", description="Price range")
    rating: str = Field(default="", description="Rating")
    distance: str = Field(default="", description="Distance to attractions")
    type: str = Field(default="", description="Hotel type")
    estimated_cost: int = Field(default=0, description="Estimated cost (yuan/night)")
```

**예산 정보**는 위치 정보는 포함하지 않지만 다양한 비용에 대한 요약을 포함하는 특수 모델입니다.

```python
class Budget(BaseModel):
    """Budget information"""
    total_attractions: int = Field(default=0, description="Total attraction ticket cost")
    total_hotels: int = Field(default=0, description="Total hotel cost")
    total_meals: int = Field(default=0, description="Total meal cost")
    total_transportation: int = Field(default=0, description="Total transportation cost")
    total: int = Field(default=0, description="Total cost")
```

이제 이러한 기본 모델을 결합하여 **일일 일정**을 구축할 수 있습니다. 일일 일정에는 날짜, 설명, 교통 방법, 숙박 시설, 호텔, 명소 목록 및 식사 목록이 포함됩니다.

```python
class DayPlan(BaseModel):
    """Daily itinerary"""
    date: str = Field(..., description="Date")
    day_index: int = Field(..., description="Day number (starting from 0)")
    description: str = Field(..., description="Daily itinerary description")
    transportation: str = Field(..., description="Transportation method")
    accommodation: str = Field(..., description="Accommodation arrangement")
    hotel: Optional[Hotel] = Field(default=None, description="Hotel information")
    attractions: List[Attraction] = Field(default_factory=list, description="Attraction list")
    meals: List[Meal] = Field(default_factory=list, description="Meal arrangements")
```

명소 목록을 나타내기 위해 `List[Attraction]`을 사용하고, `default_factory=list`은 기본값이 빈 목록임을 의미합니다.

**날씨 정보**는 Amap에서 반환한 온도 형식이 표준이 아니기 때문에 특별한 처리가 필요합니다. 이를 처리하기 위해 사용자 정의 유효성 검사기를 사용합니다.

```python
class WeatherInfo(BaseModel):
    """Weather information"""
    date: str = Field(..., description="Date")
    day_weather: str = Field(..., description="Daytime weather")
    night_weather: str = Field(..., description="Nighttime weather")
    day_temp: int = Field(..., description="Daytime temperature (Celsius)")
    night_temp: int = Field(..., description="Nighttime temperature (Celsius)")
    wind_direction: str = Field(..., description="Wind direction")
    wind_power: str = Field(..., description="Wind power")

    @field_validator('day_temp', 'night_temp', mode='before')
    def parse_temperature(cls, v):
        """Parse temperature string: "16°C" -> 16"""
        if isinstance(v, str):
            v = v.replace('°C', '').replace('℃', '').replace('°', '').strip()
            try:
                return int(v)
            except ValueError:
                return 0  # Error tolerance
        return v
```

마지막으로 **완전한 여행 계획**을 정의합니다. 이는 모든 정보를 포함하는 최상위 모델입니다.

```python
class TripPlan(BaseModel):
    """Travel plan"""
    city: str = Field(..., description="Destination city")
    start_date: str = Field(..., description="Start date")
    end_date: str = Field(..., description="End date")
    days: List[DayPlan] = Field(default_factory=list, description="Daily itinerary")
    weather_info: List[WeatherInfo] = Field(default_factory=list, description="Weather information")
    overall_suggestions: str = Field(..., description="Overall suggestions")
    budget: Optional[Budget] = Field(default=None, description="Budget information")
```

이로써 전체 데이터 모델의 설계가 완성되었습니다. 가장 기본적인 `Location`부터 `Attraction`, `Meal`, `Hotel`, 그리고 `DayPlan`, 마지막으로 `TripPlan`까지 명확한 계층 구조를 형성합니다.

13.2.5 웹 애플리케이션에서 데이터 모델 적용

이제 이러한 데이터 모델이 실제 웹 애플리케이션에서 어떻게 사용되는지 살펴보겠습니다. FastAPI에서 Pydantic 모델은 요청 및 응답에 대한 유형 정의로 직접 사용될 수 있습니다. FastAPI는 데이터 검증, 직렬화 및 문서 생성을 자동으로 수행합니다.

```python
from fastapi import FastAPI
from app.models.schemas import TripPlanRequest, TripPlan

app = FastAPI()

@app.post("/api/trip/plan", response_model=TripPlan)
async def create_trip_plan(request: TripPlanRequest) -> TripPlan:
    """
    Create travel plan

    FastAPI automatically:
    1. Validates request data (TripPlanRequest)
    2. Validates response data (TripPlan)
    3. Generates OpenAPI documentation
    """
    trip_plan = await generate_trip_plan(request)
    return trip_plan
```

사용자가 `/api/trip/plan`에 POST 요청을 보내면 FastAPI는 자동으로 JSON 데이터를 `TripPlanRequest` 개체로 변환합니다. 데이터 형식이 잘못된 경우(예: 필수 필드 누락 또는 유형 불일치) FastAPI는 자동으로 400 오류를 반환하고 사용자에게 오류가 있는 위치를 알려줍니다.

프런트엔드에서는 해당 TypeScript 유형도 정의해야 합니다. TypeScript와 Python은 서로 다른 언어이지만 데이터 구조는 동일합니다.

```typescript
interface Location {
  longitude: number;
  latitude: number;
}

interface Attraction {
  name: string;
  address: string;
  location: Location;
  visit_duration: number;
  ticket_price: number;
}

interface TripPlan {
  city: string;
  start_date: string;
  end_date: string;
  days: DayPlan[];
}
```

이런 방식으로 프런트엔드와 백엔드는 통합된 데이터 형식을 사용합니다. 백엔드가 `TripPlan` 객체를 반환하면 프런트엔드는 변환 없이 이를 직접 사용할 수 있습니다. TypeScript의 유형 검사는 많은 오류를 피하는 데에도 도움이 될 수 있습니다.

## 13.3 다중 에이전트 협업 설계

13.3.1 다중 에이전트가 필요한 이유

7장에서는 SimpleAgent를 사용하여 에이전트를 구축하는 방법을 배웠습니다. SimpleAgent의 디자인 철학은 간단하고 직접적입니다. `run()` 메소드가 호출될 때마다 에이전트는 사용자의 질문을 분석하고 도구 호출 여부를 결정한 후 결과를 반환합니다. 이 디자인은 간단한 작업을 처리할 때 매우 효과적이지만 여행 계획과 같은 작업을 수행할 때는 몇 가지 문제가 발생합니다.

여행 계획을 완료하기 위해 단일 에이전트를 사용하는 경우 이 에이전트는 무엇을 해야 합니까? 먼저, 명소 정보를 검색해야 하는데, 이를 위해서는 Amap의 POI 검색 도구를 호출해야 합니다. 그런 다음 날씨 정보를 쿼리해야 하며, 이를 위해서는 날씨 쿼리 도구를 호출해야 합니다. 다음으로 호텔 정보를 검색해야 하며, 이를 위해서는 다시 POI 검색 도구를 호출해야 합니다. 마지막으로, 완전한 여행 계획을 생성하려면 이 모든 정보를 통합해야 합니다.

간단해 보이지만 실제 작업에서는 **도구 호출 제한**이라는 첫 번째 문제가 발생합니다. SimpleAgent는 `run()` 호출당 하나의 도구만 실행할 수 있습니다. 즉, `run()` 메서드를 여러 번 호출해야 하며 각 호출은 하나의 작업을 처리해야 합니다. 하지만 이로 인해 새로운 문제가 발생합니다. 여러 호출 간에 정보를 전달하는 방법은 무엇입니까? 첫 번째 통화에서 얻은 명소 정보를 두 번째 통화에 어떻게 전달하나요? 이러한 중간 결과를 수동으로 관리해야 하므로 코드가 매우 복잡해집니다.

물론 ReactAgent를 사용하여 이 문제를 해결할 수 있습니다. ReactAgent는 한 번의 호출로 여러 도구를 실행할 수 있으며 여러 라운드의 사고와 작업을 자동으로 수행합니다. 그러나 이로 인해 **시간 비용**이라는 새로운 문제가 발생합니다. ReactAgent의 각 사고 단계에는 LLM 호출이 필요합니다. 세 가지 도구를 호출해야 하는 경우 최소한 세 번의 사고가 필요하며 이는 최소 세 번의 LLM 호출을 의미합니다. 더욱이 이러한 호출은 연속적입니다. 다음 호출은 이전 호출이 완료된 후에만 시작할 수 있으므로 총 시간이 매우 길어집니다.

두 번째 문제는 **즉각적인 복잡성**입니다. 하나의 에이전트가 모든 작업을 완료하도록 하려면 프롬프트에서 각 작업의 실행 논리를 자세히 설명해야 합니다. 예를 들어:

```python
COMPLEX_PROMPT = """You are a travel planning assistant. You need to:
1. Use maps_text_search to search for attractions, keywords determined by user preferences
2. Use maps_weather to query weather, get weather forecast for the next few days
3. Use maps_text_search to search for hotels, type determined by user needs
4. Integrate all information to generate travel plan, including daily attractions, dining, accommodation arrangements
Note: Must execute in order, each tool can only be called once, output must be in JSON format...
"""
```

이런 종류의 프롬프트에는 몇 가지 문제가 있습니다. 첫 번째는 **유지관리가 어렵다**는 것입니다. 명소 검색 로직(예: 평점 필터링 추가)을 수정하려면 전체 프롬프트를 수정해야 하며 이는 다른 부분에 쉽게 영향을 미칠 수 있습니다. 두 번째는 **오류가 발생하기 쉽습니다**. LLM은 여러 작업의 요구 사항을 동시에 이해해야 하며 다양한 작업의 형식과 매개 변수를 쉽게 혼동할 수 있습니다. 마지막으로 **디버깅이 어렵습니다**. 생성된 계획이 기대에 미치지 못하는 경우 어느 부분이 잘못되었는지 알기 어렵습니다. 명소 검색이 정확하지 않은 것인지, 날씨 쿼리가 실패한 것인지, 아니면 통합 로직에 문제가 있는 것인지?

이러한 문제에 직면했을 때 자연스러운 아이디어는 다음과 같습니다. 복잡한 작업을 여러 개의 간단한 작업으로 분해하고 서로 다른 에이전트가 각자 자신의 작업을 수행하도록 할 수 있습니까? 이것이 다중 에이전트 협업의 핵심 아이디어입니다.

현실 세계의 여행사를 상상해 보세요. 여행사에 가서 여행 계획을 상담할 때, 한 사람의 도움만 받는 것이 아닙니다. 일반적으로 명소 추천을 담당하는 전담 명소 컨설턴트가 있습니다. 호텔 예약을 담당하는 호텔 컨설턴트 그리고 모든 정보를 완전한 여행 일정으로 통합하는 일을 담당하는 여행 일정 플래너입니다. 각 사람은 자신의 전문 분야에 집중하고, 마지막으로 여행 일정 플래너가 모든 정보를 요약합니다. 이러한 노동 분업과 협업은 한 사람이 모든 일을 하는 것보다 훨씬 더 효율적입니다.

### 13.3.2 상담원 역할 설계

작업 분해 원칙에 따라 그림 13.6과 같이 4개의 특수 에이전트를 설계했습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-6.png" alt="" width="85%"/>
<p>그림 13.6 다중 에이전트 협업 흐름</p>
</div>

- **AttractionSearchAgent(명소 검색 전문가)**는 명소 정보 검색에 중점을 둡니다. 사용자 선호도(예: "역사 및 문화", "자연 경관")를 이해한 다음 Amap의 POI 검색 도구를 호출하고 관련 명소 목록을 반환하기만 하면 됩니다. 프롬프트는 매우 간단하여 선호도에 따라 키워드를 선택하는 방법과 도구를 호출하는 방법만 설명하면 됩니다.

- **WeatherQueryAgent(Weather Query Expert)**는 날씨 정보를 쿼리하는 데 중점을 둡니다. 도시 이름만 알고 날씨 쿼리 도구를 호출하여 다음 며칠 동안의 일기 예보를 반환하면 됩니다. 그 작업은 매우 명확하고 오류가 거의 없습니다.

- **HotelAgent(호텔 추천 전문가)**는 호텔 정보 검색에 집중합니다. 사용자 숙박 시설 요구 사항(예: "예산", "고급")을 이해한 다음 POI 검색 도구를 호출하고 요구 사항을 충족하는 호텔 목록을 반환해야 합니다.

- **PlannerAgent(여정 계획 전문가)**는 모든 정보를 통합하는 역할을 담당합니다. 처음 세 에이전트의 출력과 사용자의 원래 요구 사항 (dates, budget, etc.)을 수신한 다음 완전한 여행 계획을 생성합니다. 외부 도구를 호출할 필요가 없으며 정보 통합 및 일정 조정에만 집중하면 됩니다.

이제 각 Agent의 역할과 프롬프트를 세부적으로 설계해 보겠습니다. 프롬프트를 디자인할 때 다음과 같은 몇 가지 주요 질문을 고려해야 합니다. 이 에이전트에는 어떤 입력이 필요합니까? 어떤 결과물을 생산해야 합니까? 호출하려면 어떤 도구가 필요합니까? 어떤 문제가 발생할 수 있나요?

**AttractionSearchAgent**의 임무는 사용자 선호도에 따라 명소를 검색하는 것입니다. 입력은 도시 이름과 사용자 선호도(예: "역사 및 문화", "자연 경관")입니다. 키워드와 도시를 매개변수로 사용하여 `amap_maps_text_search` 도구를 호출해야 합니다. 이름, 주소, 등급 및 기타 정보를 포함한 명소 목록이 출력됩니다.

```python
ATTRACTION_AGENT_PROMPT = """You are an attraction search expert.

**Tool Call Format:**
`[TOOL_CALL:amap_maps_text_search:keywords=attraction,city=city_name]`

**Examples:**
- `[TOOL_CALL:amap_maps_text_search:keywords=attraction,city=Beijing]`
- `[TOOL_CALL:amap_maps_text_search:keywords=museum,city=Shanghai]`

**Important:**
- Must use tools to search, don't fabricate information
- Search for attractions in {city} based on user preferences ({preferences})
"""
```

이 프롬프트는 간결하지만 필요한 모든 정보를 포함하고 있습니다. 도구 호출 형식을 명확하게 설명하고 구체적인 예를 제공하며 도구를 사용해야 함(조작할 수 없음)과 사용자 기본 설정에 따라 검색해야 한다는 두 가지 중요한 원칙을 강조합니다.

**WeatherQueryAgent**의 작업은 날씨만 쿼리하면 되므로 더 간단합니다. 입력은 도시 이름이고 출력은 날씨 정보입니다.

```python
WEATHER_AGENT_PROMPT = """You are a weather query expert.

**Tool Call Format:**
`[TOOL_CALL:amap_maps_weather:city=city_name]`

Please query weather information for {city}.
"""
```

**HotelAgent**의 임무는 호텔을 검색하는 것입니다. 입력은 도시 이름과 숙박 시설 유형이고 출력은 호텔 목록입니다.

```python
HOTEL_AGENT_PROMPT = """You are a hotel recommendation expert.

**Tool Call Format:**
`[TOOL_CALL:amap_maps_text_search:keywords=hotel,city=city_name]`

Please search for {accommodation} hotels in {city}.
"""
```

**PlannerAgent**는 모든 정보를 통합해야 하기 때문에 가장 복잡합니다. 입력은 사용자 요구 사항과 처음 세 에이전트의 출력이며 출력은 완전한 여행 계획(JSON 형식)입니다.

```python
PLANNER_AGENT_PROMPT = """You are an itinerary planning expert.

**Output Format:**
Strictly return in the following JSON format:
{
  "city": "city name",
  "start_date": "YYYY-MM-DD",
  "end_date": "YYYY-MM-DD",
  "days": [...],
  "weather_info": [...],
  "overall_suggestions": "overall suggestions",
  "budget": {...}
}

**Planning Requirements:**
1. weather_info must include weather for each day
2. Temperature as pure numbers (without °C)
3. Arrange 2-3 attractions per day
4. Consider attraction distance and visit time
5. Include breakfast, lunch, and dinner
6. Provide practical suggestions
7. Include budget information
"""
```

### 13.3.3 상담원 협업 흐름

이제 이 네 명의 에이전트가 어떻게 협력하여 여행 계획 작업을 완료하는지 살펴보겠습니다. 전체 흐름은 다섯 단계로 나눌 수 있습니다.

```python
class TripPlannerAgent:
    def __init__(self):
        self.attraction_agent = SimpleAgent(name="Attraction Search", prompt=ATTRACTION_PROMPT)
        self.weather_agent = SimpleAgent(name="Weather Query", prompt=WEATHER_PROMPT)
        self.hotel_agent = SimpleAgent(name="Hotel Recommendation", prompt=HOTEL_PROMPT)
        self.planner_agent = SimpleAgent(name="Itinerary Planning", prompt=PLANNER_PROMPT)

    def plan_trip(self, request: TripPlanRequest) -> TripPlan:
        # Step 1: Attraction search
        attraction_response = self.attraction_agent.run(
            f"Please search for {request.preferences} attractions in {request.city}"
        )

        # Step 2: Weather query
        weather_response = self.weather_agent.run(
            f"Please query weather for {request.city}"
        )

        # Step 3: Hotel recommendation
        hotel_response = self.hotel_agent.run(
            f"Please search for {request.accommodation} hotels in {request.city}"
        )

        # Step 4: Integrate and generate plan
        planner_query = self._build_planner_query(
            request, attraction_response, weather_response, hotel_response
        )
        planner_response = self.planner_agent.run(planner_query)

        # Step 5: Parse JSON
        trip_plan = self._parse_trip_plan(planner_response)
        return trip_plan
```

이 흐름은 4단계를 순차적으로 실행하며 각 단계의 출력은 다음 단계의 입력으로 사용됩니다. 섹션 13.2에 정의된 `TripPlanRequest` 및 `TripPlan` Pydantic 모델을 사용한다는 점에 유의하세요.

### 13.3.4 쿼리 구성

PlannerAgent는 모든 정보를 통합해야 합니다. 이 쿼리에는 필요한 모든 정보가 포함되어야 하며 LLM이 정확하게 이해할 수 있도록 명확하고 질서 있게 구성되어야 합니다.

```python
def _build_planner_query(
    self,
    request: TripPlanRequest,
    attraction_response: str,
    weather_response: str,
    hotel_response: str
) -> str:
    """Build query for planning Agent"""
    return f"""
Please generate a {request.days}-day travel plan for {request.city} based on the following information:

**User Requirements:**
- Destination: {request.city}
- Dates: {request.start_date} to {request.end_date}
- Days: {request.days} days
- Preferences: {request.preferences}
- Budget: {request.budget}
- Transportation: {request.transportation}
- Accommodation: {request.accommodation}

**Attraction Information:**
{attraction_response}

**Weather Information:**
{weather_response}

**Hotel Information:**
{hotel_response}

Please generate a detailed travel plan, including daily attraction arrangements, dining recommendations, accommodation information, and budget details.
"""
```

이 다중 에이전트 협업 설계를 통해 복잡한 여행 계획 작업을 4개의 간단한 하위 작업으로 분해합니다. 각 에이전트는 자신의 전문 분야에 중점을 두고 있으며 향후 기능 확장(예: 레스토랑 추천 에이전트, 교통 계획 에이전트 추가)을 위한 좋은 기반을 마련합니다.

## 13.4 MCP 도구 통합 세부사항

### 13.4.1 API를 직접 호출하면 안되는 이유

섹션 13.3에서는 여행 계획 작업에 협력할 4개의 에이전트를 설계했습니다. 그 중 AttractionSearchAgent, WeatherQueryAgent, HotelAgent는 모두 Amap의 API를 호출하여 데이터를 얻어야 합니다. 자연스러운 질문은 다음과 같습니다. 에이전트에서 Amap의 HTTP API를 직접 호출하면 어떨까요?

먼저 API를 직접 호출하는 방법을 살펴보겠습니다. Amap은 POI 검색 API를 제공하며 HTTP 요청을 구성하고 매개변수를 전달하고 응답을 구문 분석해야 합니다.

```python
import requests

def search_poi(keywords: str, city: str, api_key: str):
    """Directly call Amap POI search API"""
    url = "https://restapi.amap.com/v3/place/text"
    params = {
        "keywords": keywords,
        "city": city,
        "key": api_key,
        "output": "json"
    }
    response = requests.get(url, params=params)
    data = response.json()
    return data
```

이 접근 방식은 간단해 보이지만 실제 사용에서는 몇 가지 문제에 직면하게 됩니다. 첫 번째는 **상담원이 자율적으로 전화를 걸 수 없습니다**입니다. HelloAgents 프레임워크에서 에이전트는 프롬프트의 도구 호출 표시(예: `[TOOL_CALL:tool_name:arg1=value1]`)를 인식하여 도구를 호출합니다. 코드에서 직접 API를 호출하면 에이전트는 자율적인 의사 결정 능력을 잃고 단순한 함수 호출이 됩니다.

두 번째는 **복잡한 매개변수 전달**입니다. Amap의 API에는 많은 매개변수가 있습니다. 예를 들어 POI 검색에는 `keywords`, `city`, `types`, `offset`, `page` 등과 같은 12개 이상의 매개변수가 있습니다. 에이전트가 이러한 매개변수를 유연하게 사용하도록 하려면 프롬프트에서 각 매개변수의 의미와 형식을 자세히 설명해야 하므로 프롬프트가 매우 복잡해집니다.

세 번째는 **어려운 응답 구문 분석**입니다. Amap API가 반환하는 데이터는 상대적으로 복잡한 구조를 지닌 JSON 형식입니다. 이 데이터를 구문 분석하고 필요한 필드를 추출하려면 코드를 작성해야 합니다. API의 응답 형식이 변경되면 구문 분석 코드를 수정해야 합니다.

마지막으로 **혼돈스러운 도구 관리**입니다. Amap은 12개 이상의 다양한 API (POI search, weather query, route planning, etc.)를 제공합니다. 각 API에 대한 함수를 작성한 후 에이전트의 도구 목록에 수동으로 등록하면 코드가 매우 길어집니다. 그리고 새로운 API를 추가하려면 여러 위치를 수정해야 합니다.

### 13.4.2 Amap MCP 통합

MCP(Model Context Protocol)는 LLM과 외부 도구를 연결하기 위해 Anthropic에서 제안한 표준화된 프로토콜입니다. 이 섹션에서는 Amap MCP 서버를 프로젝트에 통합하는 방법을 소개합니다. 우리 프로젝트는 Node.js에 구현된 MCP 서버인 `amap-mcp-server`를 사용합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-7.png" alt="" width="85%"/>
<p>그림 13.7 amap-mcp-server 도구</p>
</div>

Amap MCP 서버는 표 13.1에 표시된 대로 주로 다음 범주로 구분된 다양한 도구를 제공합니다.

<div align="center">
<p>표 13.1 Amap MCP 도구 카테고리</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-table-1.png" alt="" width="85%"/>
</div>

MCP 프로토콜을 통해 HelloAgents에 쉽게 통합할 수 있습니다.

```python
from hello_agents.tools import MCPTool
from app.config import get_settings

settings = get_settings()

# Create MCP tool
mcp_tool = MCPTool(
    name="amap_mcp",
    command="npx",
    args=["-y", "@sugarforever/amap-mcp-server"],
    env={"AMAP_API_KEY": settings.amap_api_key},
    auto_expand=True
)
```

이 코드는 무엇을 합니까? 먼저 `command` 및 `args`은 MCP 서버를 시작하는 방법을 지정합니다. `npx -y @sugarforever/amap-mcp-server`는 npm 저장소에서 `amap-mcp-server` 패키지를 다운로드하고 실행합니다. `env` 매개변수는 환경 변수를 전달하며 여기서는 Amap API 키를 전달합니다.

**참고:** 이 문서의 일부 예에서는 `npx`을 사용하여 MCP(모델 컨텍스트 프로토콜) 서비스를 시작합니다. 그러나 이 콘텐츠 섹션에 해당하는 코드 저장소에서는 실제로 `uvx`을 사용합니다. `npx`와 `uvx`은 거의 동일한 설계 원칙을 공유한다는 점에 유의하는 것이 중요합니다. 유일한 차이점은 생태계에 있습니다. `npx`는 JavaScript/Node.js(npm의 패키지)를 대상으로 하고 `uvx`는 Python(PyPI의 패키지)을 대상으로 합니다. 두 방법 사이에는 우월성이나 열등함이 없습니다. 이용시 필요에 따라 선택하시기 바랍니다.

`MCPTool` 객체를 생성하면 백그라운드에서 MCP 서버 프로세스를 시작하고 표준 입출력 (stdin/stdout)을 통해 서버와 통신합니다. 이는 MCP 프로토콜의 기능입니다. HTTP 대신 프로세스 간 통신을 사용하므로 더 효율적이고 관리하기 쉽습니다.

가장 중요한 매개변수는 `auto_expand=True`입니다. True로 설정하면 `MCPTool`는 MCP 서버가 제공하는 도구가 무엇인지 자동으로 쿼리한 다음 각 도구에 대해 독립적인 도구 개체를 만듭니다. 이것이 바로 우리가 `MCPTool` 하나만 생성했지만 에이전트에는 16개의 도구가 있는 이유입니다. 이 과정을 살펴보겠습니다.

```python
# Create one MCPTool
mcp_tool = MCPTool(..., auto_expand=True)
agent.add_tool(mcp_tool)

# Agent actually gets 16 tools!
print(list(agent.tools.keys()))
# ['amap_maps_text_search', 'amap_maps_weather', ...]
```

그림 13.8에서 볼 수 있듯이 사용자가 베이징의 명소를 검색하려고 한다고 가정합니다. AttractionSearchAgent는 "베이징의 역사, 문화 명소를 검색해 주세요"라는 쿼리를 수신합니다. 에이전트는 이 쿼리를 분석하고 매개변수 `keywords=attraction, city=Beijing`을 사용하여 `amap_maps_text_search` 도구를 호출하기로 결정합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-8.png" alt="" width="85%"/>
<p>그림 13.8 MCP 도구 호출 흐름</p>
</div>

에이전트는 도구 호출 마커 `[TOOL_CALL:amap_maps_text_search:keywords=attraction,city=Beijing]`를 생성합니다. HelloAgents 프레임워크는 이 마커를 구문 분석하고 도구 이름과 매개변수를 추출한 다음 해당 도구 개체를 호출합니다.

도구 개체는 `MCPTool`에 의해 자동으로 생성되며 MCP 서버에 호출 요청을 보냅니다. 특히 JSON-RPC 형식 메시지를 구성하고 이를 stdin을 통해 서버 프로세스로 보냅니다.

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "amap_maps_text_search",
    "arguments": {
      "keywords": "attraction",
      "city": "Beijing"
    }
  }
}
```

MCP 서버는 이 메시지를 수신하고 매개변수를 구문 분석한 다음 Amap의 HTTP API를 호출합니다. HTTP 요청을 구성하고, API 키를 추가하고, 요청을 보내고, 응답을 받습니다.

Amap API는 명소 목록, 주소, 좌표 및 기타 정보가 포함된 JSON 형식 데이터를 반환합니다. MCP 서버는 이 데이터를 구문 분석하고, 주요 필드를 추출한 다음 응답 메시지를 구성하여 stdout을 통해 `MCPTool`에 반환합니다.

```json
{
  "jsonrpc": "2.0",
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Found the following attractions:\n1. Forbidden City Museum - Address: No. 4 Jingshan Front Street, Dongcheng District\n2. Temple of Heaven Park - Address: Tiantan Road, Dongcheng District\n..."
      }
    ]
  }
}
```

`MCPTool`는 응답을 받고 텍스트 내용을 추출하여 에이전트에 반환합니다. 에이전트는 이 결과를 도구 호출의 출력으로 사용하고 계속해서 최종 응답을 생성합니다.

이 과정은 복잡해 보이지만 Agent의 경우 명소를 검색할 수 있는 `amap_maps_text_search`라는 도구가 있다는 것만 알면 됩니다. 모든 기본 세부 정보는 MCP 프로토콜과 `MCPTool`에 의해 캡슐화됩니다.

### 13.4.3 MCP 인스턴스 공유

다중 에이전트 시스템에서는 세 명의 에이전트가 모두 Amap 도구를 사용해야 합니다. 그러면 각 에이전트가 자체 `MCPTool` 인스턴스를 생성해야 할까요, 아니면 동일한 인스턴스를 공유해야 할까요?

각 에이전트가 `MCPTool` 인스턴스를 생성하면 3개의 서버 프로세스가 동시에 실행된다는 의미입니다. 각 프로세스는 독립적으로 Amap API를 호출하며 이는 API의 속도 제한을 초과할 수 있습니다. 또한 여러 프로세스가 더 많은 메모리와 CPU 리소스를 차지합니다.

더 나은 접근 방식은 모든 에이전트가 동일한 `MCPTool` 인스턴스를 공유하도록 하는 것입니다. 이렇게 하면 하나의 MCP 서버 프로세스만 시작하면 되며 모든 API 호출은 이 프로세스를 거칩니다. 이를 통해 리소스를 절약할 수 있을 뿐만 아니라 API 호출 빈도를 더 효과적으로 제어할 수 있습니다.

코드에서는 `TripPlannerAgent` 생성자에 `MCPTool` 인스턴스를 생성한 다음 이를 각 하위 에이전트의 도구 목록에 추가합니다.

```python
class TripPlannerAgent:
    def __init__(self):
        settings = get_settings()
        self.llm = HelloAgentsLLM()

        # Create shared MCP tool instance (create only once)
        self.mcp_tool = MCPTool(
            name="amap_mcp",
            command="npx",
            args=["-y", "@sugarforever/amap-mcp-server"],
            env={"AMAP_API_KEY": settings.amap_api_key},
            auto_expand=True
        )

        # Create multiple Agents, sharing the same MCP tool
        self.attraction_agent = SimpleAgent(
            name="AttractionSearchAgent",
            llm=self.llm,
            system_prompt=ATTRACTION_AGENT_PROMPT
        )
        self.attraction_agent.add_tool(self.mcp_tool)  # Share

        self.weather_agent = SimpleAgent(
            name="WeatherQueryAgent",
            llm=self.llm,
            system_prompt=WEATHER_AGENT_PROMPT
        )
        self.weather_agent.add_tool(self.mcp_tool)  # Share

        self.hotel_agent = SimpleAgent(
            name="HotelAgent",
            llm=self.llm,
            system_prompt=HOTEL_AGENT_PROMPT
        )
        self.hotel_agent.add_tool(self.mcp_tool)  # Share
```

이런 방식으로 세 에이전트 모두 Amap의 16개 도구를 사용할 수 있지만 그 아래에서는 하나의 MCP 서버 프로세스만 실행됩니다. `TripPlannerAgent`의 `plan_trip` 메소드를 호출하면 세 에이전트가 도구를 차례로 호출하고 모든 요청은 동일한 MCP 서버를 통해 Amap API로 전송됩니다.

### 13.4.4 Unsplash 이미지 API 통합

여행 계획을 보다 생생하고 직관적으로 만들기 위해서는 Amap 외에도 명소에 대한 이미지도 획득해야 합니다. Unsplash API를 사용하여 명소 이미지를 검색합니다. Unsplash는 외국 서비스이며 무료로 사용할 수 있는 몇 안 되는 이미지 API 중 하나이므로 검색 결과가 충분히 정확하지 않을 수 있습니다. 실제 프로젝트에서는 Bing, Baidu 또는 Amap의 POI 이미지 API 사용을 고려할 수 있지만 이러한 서비스에는 일반적으로 결제가 필요합니다.

Unsplash API의 통합은 비교적 간단합니다. API 호출을 캡슐화하기 위해 `UnsplashService` 클래스를 만듭니다.

```python
# backend/app/services/unsplash_service.py
import requests
from typing import Optional, List, Dict
import logging

logger = logging.getLogger(__name__)

class UnsplashService:
    """Unsplash image service"""

    def __init__(self, access_key: str):
        self.access_key = access_key
        self.base_url = "https://api.unsplash.com"

    def search_photos(self, query: str, per_page: int = 10) -> List[Dict]:
        """Search for images"""
        try:
            url = f"{self.base_url}/search/photos"
            params = {
                "query": query,
                "per_page": per_page,
                "client_id": self.access_key
            }

            response = requests.get(url, params=params, timeout=10)
            response.raise_for_status()

            data = response.json()
            results = data.get("results", [])

            # Extract image URLs
            photos = []
            for result in results:
                photos.append({
                    "url": result["urls"]["regular"],
                    "description": result.get("description", ""),
                    "photographer": result["user"]["name"]
                })

            return photos

        except Exception as e:
            logger.error(f"Image search failed: {e}")
            return []

    def get_photo_url(self, query: str) -> Optional[str]:
        """Get single image URL"""
        photos = self.search_photos(query, per_page=1)
        return photos[0].get("url") if photos else None
```

이 서비스 클래스는 두 가지 방법을 제공합니다. `search_photos`는 여러 이미지를 검색하고, `get_photo_url`은 단일 이미지의 URL을 가져옵니다. 우리는 API 경로에서 이 서비스를 사용하여 각 명소에 대한 이미지를 얻습니다.

```python
# backend/app/api/routes/trip.py
from app.services.unsplash_service import UnsplashService

unsplash_service = UnsplashService(settings.unsplash_access_key)

@router.post("/plan", response_model=TripPlan)
async def create_trip_plan(request: TripPlanRequest) -> TripPlan:
    # Generate travel plan
    trip_plan = trip_planner_agent.plan_trip(request)

    # Get images for each attraction
    for day in trip_plan.days:
        for attraction in day.attractions:
            if not attraction.image_url:
                image_url = unsplash_service.get_photo_url(
                    f"{attraction.name} {trip_plan.city}"
                )
                attraction.image_url = image_url

    return trip_plan
```

Unsplash를 도구 또는 MCP 도구로 캡슐화하지 않고 API 경로에서 직접 호출했습니다. 이는 이미지 검색에는 에이전트의 지능적인 의사 결정이 필요하지 않고 단순한 데이터 향상 단계이기 때문입니다. 에이전트가 이미지가 필요한지 여부를 자동으로 결정하거나 다른 이미지 소스를 선택하도록 하려면 이를 도구로 캡슐화하는 것을 고려할 수 있습니다.

## 13.5 프론트엔드 개발 세부사항

### 13.5.1 프런트엔드와 백엔드 분리 웹 아키텍처

프런트 엔드 개발을 시작하기 전에 최신 웹 애플리케이션의 아키텍처 패턴을 이해해야 합니다. 초기 웹 개발에서는 프런트엔드와 백엔드가 혼합되어 있었습니다. 예를 들어, PHP 및 JSP와 같은 기술에는 HTML 템플릿과 비즈니스 논리 코드가 동일한 파일에 작성되어 있습니다. 이 접근 방식은 소규모 프로젝트에서는 편리하지만 대규모 프로젝트에서는 많은 문제에 직면합니다. 프런트엔드와 백엔드 개발자는 빈번한 조정이 필요하고 코드를 재사용하기 어렵고 테스트가 어렵습니다.

최신 웹 애플리케이션은 일반적으로 **프런트엔드와 백엔드 분리** 아키텍처를 채택합니다. 백엔드는 API 인터페이스를 제공하고 JSON 형식으로 데이터를 반환하는 역할만 담당합니다. 프런트엔드는 HTTP 요청을 통해 백엔드 API를 호출하고 데이터를 얻은 다음 페이지를 렌더링하는 독립적인 애플리케이션입니다. 이 아키텍처에는 몇 가지 확실한 장점이 있습니다. 프런트엔드와 백엔드를 독립적으로 개발, 배포 및 테스트할 수 있습니다. 프런트엔드는 동일한 백엔드 API 세트를 사용하는 웹 애플리케이션, 모바일 애플리케이션 또는 데스크톱 애플리케이션일 수 있습니다. 프런트엔드는 최신 프레임워크와 툴체인을 사용하여 더 나은 사용자 경험을 제공할 수 있습니다.

지능형 여행 도우미 프로젝트에서 백엔드는 Python 및 FastAPI로 구현되어 여행 요구 사항을 수신하고 여행 계획을 반환하는 핵심 API 인터페이스 `POST /api/trip/plan`를 제공합니다. 프런트엔드는 Vue 3 및 TypeScript로 구현되며 단일 페이지 애플리케이션(SPA)입니다. 사용자가 브라우저에서 양식을 작성하고 "계획 시작" 버튼을 클릭하면 프런트 엔드가 백엔드에 HTTP 요청을 보내고 응답을 기다린 다음 결과 페이지를 렌더링합니다. 이 프로세스 전체에서 페이지가 새로 고쳐지지 않으며 사용자 경험이 매우 원활합니다.

프런트엔드 기술 스택을 선택할 때는 개발 효율성, 성능, 생태계, 학습 곡선 등 여러 요소를 고려해야 합니다. 표 13.2에 표시된 대로 프로젝트에서는 다음 기술 스택을 선택했습니다.

<div align="center">
<p>표 13.2 프런트엔드 기술 스택</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-table-2.png" alt="" width="85%"/>
</div>

프로젝트의 디렉토리 구조는 다음과 같습니다.

```
frontend/
├── src/
│   ├── views/              # Page components
│   │   ├── Home.vue        # Home page (form)
│   │   └── Result.vue      # Result page
│   ├── services/           # API services
│   │   └── api.ts
│   ├── types/              # Type definitions
│   │   └── index.ts
│   ├── router/             # Router configuration
│   │   └── index.ts
│   ├── App.vue
│   └── main.ts
├── package.json
├── vite.config.ts
└── tsconfig.json
```

`views` 디렉터리는 페이지 구성 요소를 저장하고, `services` 디렉터리는 API 호출 논리를 저장하고, `types` 디렉터리는 TypeScript 유형 정의를 저장하고, `router` 디렉터리는 라우터 구성을 저장합니다.

### 13.5.2 유형 정의

섹션 13.2에서는 Pydantic을 사용하여 백엔드에서 `Location`, `Attraction`, `DayPlan`, `TripPlan` 등과 같은 데이터 모델을 정의했습니다. 프런트엔드에서는 해당 TypeScript 유형을 정의해야 합니다.

이러한 유형을 정의하는 방법을 살펴보겠습니다. 첫 번째는 경도와 위도 좌표를 나타내는 가장 기본적인 `Location` 유형입니다.

```typescript
// frontend/src/types/index.ts
export interface Location {
  longitude: number
  latitude: number
}
```

이 유형 정의는 백엔드 Pydantic 모델과 정확히 일치합니다. TypeScript는 `interface` 키워드를 사용하여 유형을 정의하고 필드 유형은 콜론으로 구분되며 기본값이 필요하지 않습니다.

다음은 명소 정보를 나타내는 `Attraction` 유형입니다.

```typescript
export interface Attraction {
  name: string
  address: string
  location: Location
  visit_duration: number
  description: string
  category?: string
  rating?: number
  image_url?: string
  ticket_price?: number
}
```

여기서는 중첩 유형인 `Location` 유형을 필드 유형으로 사용합니다. 물음표 `?`는 백엔드 Pydantic 모델의 `Optional`에 해당하는 선택적 필드를 나타냅니다.

마찬가지로 `Meal`, `Hotel`, `Budget`, `WeatherInfo` 등과 같은 유형을 정의합니다. 마지막으로 최상위 수준 `TripPlan` 유형은 다음과 같습니다.

```typescript
export interface TripPlan {
  city: string
  start_date: string
  end_date: string
  days: DayPlan[]
  weather_info: WeatherInfo[]
  overall_suggestions: string
  budget?: Budget
}
```

백엔드 요청 모델에 해당하는 요청 유형 `TripPlanRequest`도 있습니다.

```typescript
export interface TripPlanRequest {
  city: string
  start_date: string
  end_date: string
  days: number
  preferences: string
  budget: string
  transportation: string
  accommodation: string
}
```

이러한 유형 정의는 무엇을 위한 것인가요? 먼저, API를 호출하면 TypeScript는 우리가 전달하는 데이터가 `TripPlanRequest` 유형을 준수하는지 확인합니다. 실수로 `days`을 문자열로 쓰면 TypeScript는 즉시 오류를 보고합니다. 둘째, API 응답을 받으면 TypeScript는 응답 데이터가 `TripPlan` 유형을 준수하는지 확인합니다. 백엔드의 데이터 구조가 변경되면 프런트엔드가 즉시 이를 발견합니다. 마지막으로 IDE는 유형 정의를 기반으로 코드 완성 기능을 제공할 수 있습니다. `tripPlan.`를 입력하면 IDE가 자동으로 사용 가능한 모든 필드를 나열합니다.

### 13.5.3 API 서비스 캡슐화

유형 정의를 사용하면 API 호출을 캡슐화할 수 있습니다. `api.ts` 파일을 생성하고 Axios를 사용하여 HTTP 요청을 보냅니다.

```typescript
import axios from 'axios'
import type { TripPlanRequest, TripPlan } from '../types'

const api = axios.create({
  baseURL: 'http://localhost:8000/api',
  timeout: 120000, // 2-minute timeout
  headers: {
    'Content-Type': 'application/json'
  }
})
```

여기서는 Axios 인스턴스를 생성하고 기본 URL, 시간 초과 및 요청 헤더를 구성합니다. 시간 초과가 2분으로 설정된 이유는 무엇입니까? 여행 계획을 생성하려면 여러 에이전트를 호출해야 하기 때문에 각 에이전트는 LLM 및 외부 API를 호출해야 하며 전체 프로세스는 10~30초 정도 걸릴 수 있습니다. 제한 시간이 너무 짧으면 요청이 중단됩니다.

다음으로 인터셉터를 추가합니다. 인터셉터는 요청을 보내기 전과 응답을 받은 후에 로깅, 오류 처리, 인증 등과 같은 몇 가지 일반적인 논리를 실행할 수 있습니다.

```typescript
// Request interceptor
api.interceptors.request.use(
  config => {
    console.log('Sending request:', config)
    return config
  },
  error => Promise.reject(error)
)

// Response interceptor
api.interceptors.response.use(
  response => {
    console.log('Received response:', response)
    return response
  },
  error => {
    console.error('Request failed:', error)
    return Promise.reject(error)
  }
)
```

마지막으로 프런트엔드가 백엔드를 호출하는 유일한 진입점인 API 함수를 정의합니다.

```typescript
// Generate travel plan
export const generateTripPlan = async (request: TripPlanRequest): Promise<TripPlan> => {
  const response = await api.post<TripPlan>('/trip/plan', request)
  return response.data
}
```

이 함수의 유형 서명에 유의하세요. 매개변수는 `TripPlanRequest` 유형이고 반환 값은 `Promise<TripPlan>` 유형입니다. 이는 TypeScript가 호출자가 전달한 매개변수가 요구 사항을 충족하는지 확인하고 반환 값의 사용이 올바른지 여부도 확인한다는 의미입니다.

### 13.5.4 홈 폼 디자인

홈 페이지는 사용자가 여행 요구 사항을 입력하는 양식이 포함된 사용자의 진입점입니다. 우리는 Vue 3의 Composition API를 사용하여 코드를 구성합니다.

```vue
<script setup lang="ts">
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { message } from 'ant-design-vue'
import { generateTripPlan } from '@/services/api'
import type { TripPlanRequest } from '@/types'

const router = useRouter()
const loading = ref(false)
const loadingProgress = ref(0)
const loadingStatus = ref('')

const formData = ref<TripPlanRequest>({
  city: '',
  start_date: '',
  end_date: '',
  days: 3,
  preferences: 'History and Culture',
  budget: 'Medium',
  transportation: 'Public Transportation',
  accommodation: 'Budget Hotel'
})
</script>
```

여기서는 `ref`을 사용하여 반응형 변수를 생성합니다. `formData`은 `TripPlanRequest` 유형의 양식 데이터입니다. `loading`는 로딩 여부를 나타내고, `loadingProgress`는 로딩 진행 상황을 나타내며, `loadingStatus`는 로딩 상태 텍스트를 나타냅니다.

양식 제출 논리는 다음과 같습니다.

```typescript
const handleSubmit = async () => {
  loading.value = true
  loadingProgress.value = 0

  // Simulate progress updates
  const progressInterval = setInterval(() => {
    if (loadingProgress.value < 90) {
      loadingProgress.value += 10
      if (loadingProgress.value <= 30) loadingStatus.value = '🔍 Searching for attractions...'
      else if (loadingProgress.value <= 50) loadingStatus.value = '🌤️ Querying weather...'
      else if (loadingProgress.value <= 70) loadingStatus.value = '🏨 Recommending hotels...'
      else loadingStatus.value = '📋 Generating itinerary...'
    }
  }, 500)

  try {
    const response = await generateTripPlan(formData.value)
    clearInterval(progressInterval)
    loadingProgress.value = 100
    router.push({ name: 'result', state: { tripPlan: response } })
  } catch (error) {
    clearInterval(progressInterval)
    message.error('Failed to generate plan, please try again')
  } finally {
    loading.value = false
  }
}
```

이 코드는 여러 가지 작업을 수행합니다. 먼저 `loading`을 true로 설정하여 로딩 상태를 표시합니다. 그런 다음 500밀리초마다 진행률 표시줄과 상태 텍스트를 업데이트하는 타이머를 시작합니다. 백엔드의 처리 진행 상황을 정확하게 알 수 없기 때문에 시뮬레이션된 진행 상황입니다. 그러나 이를 통해 사용자는 시스템이 정지되지 않고 작동 중임을 알 수 있습니다.

다음으로 `generateTripPlan` 함수를 호출하여 API 요청을 보냅니다. 이는 비동기 작업이며 `await`을 사용하여 응답을 기다립니다. 요청이 성공하면 타이머를 지우고 진행률을 100%로 설정한 다음 결과 페이지로 이동하여 여행 계획 데이터를 전달합니다. 요청이 실패하면 오류 메시지를 표시합니다. 마지막으로 성공 여부에 관계없이 `loading`를 false로 설정하여 로딩 상태를 숨깁니다.

템플릿 부분은 Ant Design Vue 구성 요소를 사용합니다.

```vue
<template>
  <div class="home-container">
    <div class="page-header">
      <h1 class="page-title">✈️ Intelligent Travel Assistant</h1>
      <p class="page-subtitle">AI-Powered Personalized Travel Planning</p>
    </div>

    <a-card class="form-card">
      <a-form :model="formData" @finish="handleSubmit">
        <a-form-item label="Destination City" name="city" :rules="[{ required: true }]">
          <a-input v-model:value="formData.city" placeholder="e.g., Beijing" />
        </a-form-item>

        <!-- More form items... -->

        <a-form-item>
          <a-button type="primary" html-type="submit" size="large" :loading="loading">
            Start Planning
          </a-button>
        </a-form-item>

        <!-- Loading progress bar -->
        <a-form-item v-if="loading">
          <a-progress :percent="loadingProgress" status="active" />
          <p>{{ loadingStatus }}</p>
        </a-form-item>
      </a-form>
    </a-card>
  </div>
</template>
```

양방향 데이터 바인딩을 구현하는 `v-model:value` 지시문에 유의하세요. 사용자가 입력 상자에 입력하면 `formData.city`이 자동으로 업데이트됩니다. `formData.city` 값이 변경되면 입력 상자 내용도 자동으로 업데이트됩니다.

### 13.5.5 결과 페이지 표시

결과 페이지는 생성된 여행 계획을 표시하는 전체 애플리케이션의 핵심입니다. 이 페이지에는 여행 일정 개요, 예산 세부정보, 지도 시각화, 일일 여행 일정 세부정보, 날씨 정보 등 여러 부분이 포함되어 있습니다.

첫 번째는 지도 시각화입니다. 우리는 Amap JS API를 사용하여 지도에 명소 위치를 표시합니다.

```typescript
import AMapLoader from '@amap/amap-jsapi-loader'

const initMap = async () => {
  const AMap = await AMapLoader.load({
    key: 'your_amap_web_key',
    version: '2.0'
  })

  map = new AMap.Map('amap-container', {
    zoom: 12,
    center: [116.397128, 39.916527]
  })

  // Add attraction markers
  tripPlan.value.days.forEach((day) => {
    day.attractions.forEach((attraction, index) => {
      const marker = new AMap.Marker({
        position: [attraction.location.longitude, attraction.location.latitude],
        title: attraction.name,
        label: { content: `${index + 1}`, direction: 'top' }
      })
      map.add(marker)
    })
  })
}
```

이 코드는 먼저 Amap SDK를 로드한 다음 지도 인스턴스를 생성하고 마지막으로 모든 명소를 반복하여 각각에 대한 마커를 생성합니다. 마커의 위치는 백엔드의 `Attraction` 객체에서 얻은 명소의 경도 및 위도 좌표입니다.

내보내기 기능은 `html2canvas` 및 `jsPDF` 라이브러리를 사용합니다. `html2canvas`는 DOM 요소를 Canvas로 변환한 다음 Canvas를 이미지나 PDF로 내보낼 수 있습니다.

```typescript
import html2canvas from 'html2canvas'
import jsPDF from 'jspdf'

// Export as image
const exportAsImage = async () => {
  const element = document.getElementById('trip-plan-content')
  const canvas = await html2canvas(element, { scale: 2 })
  const link = document.createElement('a')
  link.download = `${tripPlan.value.city} Travel Plan.png`
  link.href = canvas.toDataURL()
  link.click()
}

// Export as PDF
const exportAsPDF = async () => {
  const element = document.getElementById('trip-plan-content')
  const canvas = await html2canvas(element, { scale: 2 })
  const imgData = canvas.toDataURL('image/png')
  const pdf = new jsPDF('p', 'mm', 'a4')
  const imgWidth = 210
  const imgHeight = (canvas.height * imgWidth) / canvas.width
  pdf.addImage(imgData, 'PNG', 0, 0, imgWidth, imgHeight)
  pdf.save(`${tripPlan.value.city} Travel Plan.pdf`)
}
```

이러한 프런트엔드 기술을 통해 우리는 완전한 웹 애플리케이션을 구현했습니다. 사용자는 브라우저에서 양식을 작성하고, 요청을 제출하고, AI가 여행 계획을 생성할 때까지 기다린 다음, 자세한 여행 일정을 확인하고, 지도에서 명소 위치를 확인하고, 이미지나 PDF로 내보낼 수 있습니다. 전체 프로세스가 원활하고 자연스럽습니다. 이것이 현대 웹 애플리케이션의 매력입니다.

## 13.6 기능 구현 세부 사항

이 섹션에서는 예산 계산, 진행률 표시줄 로드, 여행 일정 편집, 내보내기 기능, 측면 탐색 등 지능형 여행 도우미의 핵심 기능 구현을 소개합니다.

### 13.6.1 예산 계산 기능

여행을 계획할 때 예산은 매우 중요한 고려 사항입니다. 사용자는 이 여행에 드는 비용과 돈이 어디에 사용될지 대략적으로 알아야 합니다. 우리의 지능형 여행 도우미는 비용을 명소 티켓, 호텔 숙박, 식사, 교통의 네 가지 주요 범주로 분류하는 자동 예산 계산 기능을 제공합니다.

예산 계산 로직은 어디에 구현되나요? 우리는 백엔드의 PlannerAgent에서 이를 구현하기로 결정했습니다. 왜 프런트엔드에서 계산하지 않나요? 예산 추정은 명소 티켓 가격, 호텔 가격 범위, 식사 표준 및 기타 정보를 기반으로 해야 하기 때문에 이 모든 정보는 여행 일정을 생성할 때 PlannerAgent에서 이미 획득한 것입니다. 프런트 엔드에서 계산하는 경우 이 논리를 복제해야 하며 정확하지 않을 수 있습니다.

PlannerAgent의 프롬프트에서는 LLM이 예산 정보를 생성하도록 명시적으로 요구합니다.

```python
PLANNER_AGENT_PROMPT = """
You are an itinerary planning expert.

**Output Format:**
Strictly return in the following JSON format:
{
  ...
  "budget": {
    "total_attractions": 180,
    "total_hotels": 1200,
    "total_meals": 480,
    "total_transportation": 200,
    "total": 2060
  }
}

**Planning Requirements:**
...
7. Include budget information, estimate based on attraction tickets, hotel prices, dining standards, and transportation methods
"""
```

LLM은 여행 일정의 명소, 호텔 및 식사 준비를 기반으로 각 항목의 비용을 추정합니다. 예를 들어 여행 일정에 자금성(티켓 60위안), 천단(티켓 15위안), 이화원(티켓 30위안)이 포함되어 있다면 총 관광지 티켓 비용은 105위안입니다. 2박 3일 저가 호텔 여행(1박당 300위안)이라면 총 호텔 비용은 600위안입니다.

프런트 엔드에서는 Ant Design Vue의 Statistic 구성 요소를 사용하여 예산 정보를 표시합니다. 이 구성 요소는 통계 데이터를 표시하기 위해 특별히 설계되었으며 숫자 애니메이션, 접두사/접미사, 사용자 정의 스타일 등을 지원합니다.

```vue
<a-card v-if="tripPlan.budget" title="💰 Budget Details">
  <a-row :gutter="16">
    <a-col :span="6">
      <a-statistic title="Attraction Tickets" :value="tripPlan.budget.total_attractions" suffix="yuan" />
    </a-col>
    <a-col :span="6">
      <a-statistic title="Hotel Accommodation" :value="tripPlan.budget.total_hotels" suffix="yuan" />
    </a-col>
    <a-col :span="6">
      <a-statistic title="Dining Expenses" :value="tripPlan.budget.total_meals" suffix="yuan" />
    </a-col>
    <a-col :span="6">
      <a-statistic title="Transportation" :value="tripPlan.budget.total_transportation" suffix="yuan" />
    </a-col>
  </a-row>
  <a-divider />
  <a-row>
    <a-col :span="24" style="text-align: center;">
      <a-statistic
        title="Estimated Total Cost"
        :value="tripPlan.budget.total"
        suffix="yuan"
        :value-style="{ color: '#cf1322', fontSize: '32px', fontWeight: 'bold' }"
      />
    </a-col>
  </a-row>
</a-card>
```

이 코드는 그리드 레이아웃(`a-row` 및 `a-col`)을 사용하여 4개의 비용 항목을 나란히 표시합니다. 각 비용 항목은 `a-statistic` 구성요소를 사용하여 제목과 값을 표시합니다. 마지막으로 구분선(`a-divider`)으로 구분하며 아래에는 강조를 위해 총 비용을 큰 빨간색 글꼴로 표시합니다.

조건부 렌더링 `v-if="tripPlan.budget"`을 참고하세요. 예산 정보는 선택 사항이므로(Pydantic 모델에서 `Optional[Budget]`로 정의됨) LLM이 예산 정보를 생성하지 않으면 이 카드가 표시되지 않습니다. 이는 데이터에 대한 프런트 엔드의 오류 허용 범위를 반영합니다.

### 13.6.2 진행률 표시줄 로드

여행 계획을 세우는 것은 시간이 많이 걸리는 작업입니다. 백엔드는 AttractionSearchAgent, WeatherQueryAgent, HotelAgent, PlannerAgent를 순차적으로 호출해야 하며, 각 에이전트는 LLM 및 외부 API를 호출해야 합니다. 전체 과정은 10~30초 정도 소요될 수 있습니다. 사용자가 "계획 시작" 버튼을 클릭했는데 페이지에 피드백이 없으면 사용자는 시스템이 멈췄다고 생각하고 페이지를 새로 고치거나 반복적으로 클릭할 수 있습니다.

사용자 경험을 개선하기 위해 로딩 진행률 표시줄과 상태 프롬프트를 추가했습니다. 현재는 시뮬레이션된 진행 상황일 뿐이지만 사용자에게 시스템이 작동 중임을 알립니다.

```typescript
const loading = ref(false)
const loadingProgress = ref(0)
const loadingStatus = ref('')

const handleSubmit = async () => {
  loading.value = true
  loadingProgress.value = 0

  // Simulate progress updates
  const progressInterval = setInterval(() => {
    if (loadingProgress.value < 90) {
      loadingProgress.value += 10
      if (loadingProgress.value <= 30) loadingStatus.value = '🔍 Searching for attractions...'
      else if (loadingProgress.value <= 50) loadingStatus.value = '🌤️ Querying weather...'
      else if (loadingProgress.value <= 70) loadingStatus.value = '🏨 Recommending hotels...'
      else loadingStatus.value = '📋 Generating itinerary...'
    }
  }, 500)

  try {
    const response = await generateTripPlan(formData.value)
    clearInterval(progressInterval)
    loadingProgress.value = 100
    loadingStatus.value = '✅ Complete!'
    router.push({ name: 'result', state: { tripPlan: response } })
  } catch (error) {
    clearInterval(progressInterval)
    message.error('Failed to generate plan')
  } finally {
    loading.value = false
  }
}
```

### 13.6.3 여행일정 편집 기능

AI가 생성한 여행 계획은 지능적이지만 사용자의 개인적인 요구를 완전히 충족시키지 못할 수 있습니다. 예를 들어, 사용자는 특정 명소가 마음에 들지 않아 삭제하고 싶을 수도 있고, 명소 순서를 조정하고 싶을 수도 있습니다. 우리는 사용자가 여행 일정을 맞춤 설정할 수 있는 여행 일정 편집 기능을 제공합니다.

편집 기능의 핵심은 **상태 관리**입니다. 현재 여행 일정 계획과 원래 여행 일정 계획이라는 두 가지 상태를 유지해야 합니다. 사용자가 편집 모드에 들어가면 원래 계획의 사본이 저장됩니다. 사용자가 편집을 취소하면 원래 계획이 복원됩니다. 사용자가 변경 사항을 저장하면 현재 계획이 업데이트됩니다.

```typescript
const editMode = ref(false)
const originalPlan = ref<TripPlan | null>(null)

// Enter edit mode
const toggleEditMode = () => {
  editMode.value = true
  originalPlan.value = JSON.parse(JSON.stringify(tripPlan.value))
}
```

객체를 깊이 복사하려면 `JSON.parse(JSON.stringify(...))`을 사용합니다. 왜 직접 할당하지 않습니까? JavaScript의 객체는 참조 유형이기 때문에 직접 할당하면 `originalPlan`과 `tripPlan`가 동일한 객체를 가리키며 하나를 수정하면 다른 객체에도 영향을 미칩니다. 전체 복사는 완전히 독립적인 복사본을 생성합니다.

어트랙션을 이동하는 논리는 배열에 있는 두 요소의 위치를 ​​바꾸는 것입니다.

```typescript
// Move attraction
const moveAttraction = (dayIndex: number, attractionIndex: number, direction: 'up' | 'down') => {
  const attractions = tripPlan.value.days[dayIndex].attractions
  const newIndex = direction === 'up' ? attractionIndex - 1 : attractionIndex + 1

  if (newIndex >= 0 && newIndex < attractions.length) {
    [attractions[attractionIndex], attractions[newIndex]] =
    [attractions[newIndex], attractions[attractionIndex]]
  }
}
```

이는 ES6의 구조 분해 할당 구문을 사용하여 두 요소를 교환합니다. `[a, b] = [b, a]`은 임시 변수 없이 교체하는 우아한 방법입니다.

명소를 삭제하려면 배열의 `splice` 메서드를 사용합니다.

```typescript
// Delete attraction
const deleteAttraction = (dayIndex: number, attractionIndex: number) => {
  tripPlan.value.days[dayIndex].attractions.splice(attractionIndex, 1)
}
```

변경 사항을 저장할 때 명소 위치가 변경되었을 수 있으므로 지도를 다시 초기화해야 합니다.

```typescript
// Save changes
const saveChanges = () => {
  editMode.value = false
  message.success('Changes saved')
  initMap()  // Reinitialize map
}

// Cancel editing
const cancelEdit = () => {
  if (originalPlan.value) {
    tripPlan.value = originalPlan.value
  }
  editMode.value = false
}
```

템플릿에서는 `editMode` 값에 따라 다른 UI를 표시합니다. 편집 모드에서는 각 명소 옆에 위, 아래, 삭제 버튼이 표시됩니다.

```vue
<div v-if="editMode" class="edit-buttons">
  <a-button size="small" @click="moveAttraction(dayIndex, index, 'up')">Up</a-button>
  <a-button size="small" @click="moveAttraction(dayIndex, index, 'down')">Down</a-button>
  <a-button size="small" danger @click="deleteAttraction(dayIndex, index)">Delete</a-button>
</div>
```

### 13.6.4 내보내기 기능

사용자는 만족스러운 여행 계획을 작성한 후 이를 저장하거나 친구들과 공유할 수 있습니다. 이미지로 내보내기와 PDF로 내보내기라는 두 가지 내보내기 방법을 제공합니다.

내보내기 기능의 핵심은 `html2canvas` 라이브러리입니다. 이 라이브러리는 DOM 요소를 Canvas로 변환한 다음 Canvas를 이미지로 내보낼 수 있습니다. 하지만 여기에는 기술적인 문제가 있습니다. 지도는 캔버스를 사용하여 렌더링되며 `html2canvas`에는 중첩된 캔버스를 처리할 때 호환성 문제가 있습니다.

내보내기 전에 지도 Canvas를 이미지로 변환하는 것을 포함하여 여러 솔루션을 시도했지만 Amap의 Canvas 렌더링 메커니즘과 교차 출처 제한으로 인해 이 솔루션은 문제를 완전히 해결하지 못했습니다. 실제 프로젝트에서는 다음과 같은 대체 솔루션을 고려해야 할 수도 있습니다.

1. **Amap의 정적 지도 API 사용**: `maps_staticmap` 도구를 호출하여 동적 지도를 대체할 정적 지도 이미지를 생성합니다.
2. **별도 내보내기**: 지도와 여행 일정 콘텐츠를 별도로 내보낸 후 백엔드에서 병합합니다.
3. **스크린샷 서비스 사용**: Puppeteer와 같은 헤드리스 브라우저를 사용하여 서버 측에서 스크린샷을 찍습니다.
4. **컨텐츠 내보내기 단순화**: 내보낼 때 지도를 숨기고 텍스트 컨텐츠만 내보냅니다.

현재 구현에서는 내보낼 때 지도 부분을 일시적으로 숨기고 여행 일정의 텍스트 콘텐츠와 명소 정보만 내보내는 단순화된 접근 방식을 채택했습니다. 이것이 이상적인 솔루션은 아니지만 내보내기 기능을 사용할 수 있도록 보장합니다.

이미지로 내보내는 논리는 간단합니다.

```typescript
import html2canvas from 'html2canvas'

const exportAsImage = async () => {
  const element = document.getElementById('trip-plan-content')
  if (!element) return

  const canvas = await html2canvas(element, {
    backgroundColor: '#ffffff',
    scale: 2,
    useCORS: true
  })

  const link = document.createElement('a')
  link.download = `${tripPlan.value.city} Travel Plan.png`
  link.href = canvas.toDataURL('image/png')
  link.click()
  message.success('Export successful!')
}
```

`scale: 2`은 2배 해상도를 사용하여 내보낸 이미지를 더 선명하게 만드는 것을 의미합니다. `useCORS: true`은 출처 간 이미지 로딩을 허용하는데, 이는 명소 이미지(Unsplash의)에 중요합니다.

PDF로 내보내려면 추가 단계가 필요합니다. 먼저 Canvas로 변환한 다음 이미지로 변환하고 마지막으로 PDF에 추가합니다.

```typescript
import jsPDF from 'jspdf'

const exportAsPDF = async () => {
  // First capture map image
  await captureMapImage()

  const element = document.getElementById('trip-plan-content')
  if (!element) return

  const canvas = await html2canvas(element, {
    backgroundColor: '#ffffff',
    scale: 2,
    useCORS: true,
    allowTaint: true
  })

  // Restore map
  restoreMap()

  const pdf = new jsPDF('p', 'mm', 'a4')
  const imgData = canvas.toDataURL('image/png')
  const imgWidth = 210  // A4 width
  const imgHeight = (canvas.height * imgWidth) / canvas.width

  pdf.addImage(imgData, 'PNG', 0, 0, imgWidth, imgHeight)
  pdf.save(`${tripPlan.value.city} Travel Plan.pdf`)
  message.success('Export successful!')
}
```

여기서는 종횡비를 유지하기 위해 이미지 높이를 계산해야 합니다. A4 용지의 너비는 210mm이며 캔버스 종횡비를 기준으로 해당 높이를 계산합니다.

### 13.6.5 측면 탐색 및 앵커 점프

결과 페이지에는 여행 일정 개요, 예산 세부정보, 지도, 일일 여행 일정, 날씨 정보 등 많은 콘텐츠가 포함되어 있습니다. 사용자가 특정 섹션으로 빠르게 이동하려면 먼 거리를 스크롤해야 합니다. 사용자가 빠르게 찾을 수 있도록 측면 탐색 및 앵커 점프 기능을 제공합니다.

측면 탐색은 Ant Design Vue의 메뉴 구성요소를 사용합니다.

```vue
<a-menu
  v-model:selectedKeys="[activeSection]"
  mode="inline"
  @click="scrollToSection"
>
  <a-menu-item key="overview">📋 Itinerary Overview</a-menu-item>
  <a-menu-item key="budget">💰 Budget Details</a-menu-item>
  <a-menu-item key="map">🗺️ Map</a-menu-item>
  <a-menu-item key="days">📅 Daily Itinerary</a-menu-item>
  <a-menu-item key="weather">🌤️ Weather</a-menu-item>
</a-menu>
```

메뉴 항목을 클릭하면 `scrollToSection` 함수를 호출합니다.

```typescript
const activeSection = ref('overview')

// Scroll to specified section
const scrollToSection = ({ key }: { key: string }) => {
  activeSection.value = key
  const element = document.getElementById(key)
  if (element) {
    element.scrollIntoView({ behavior: 'smooth', block: 'start' })
  }
}
```

`scrollIntoView`은 요소를 표시 영역으로 스크롤할 수 있는 기본 브라우저 API입니다. `behavior: 'smooth'`은 즉각적인 점프보다는 부드러운 스크롤을 의미합니다. `block: 'start'`는 요소의 상단이 가시 영역의 상단과 정렬됨을 의미합니다.

페이지의 다양한 부분에서 해당 ID를 추가해야 합니다.

```vue
<div id="overview">
  <!-- Itinerary overview content -->
</div>

<div id="budget">
  <!-- Budget details content -->
</div>

<div id="map">
  <!-- Map content -->
</div>
```

이렇게 하면 사용자가 측면 탐색에서 메뉴 항목을 클릭할 때 페이지가 해당 섹션으로 원활하게 스크롤됩니다.

이러한 기능의 구현을 통해 당사의 지능형 여행 도우미는 여행 계획을 생성할 뿐만 아니라 풍부한 대화형 기능도 제공합니다. 예산 계산을 통해 사용자는 비용을 이해할 수 있고, 진행률 표시줄을 로드하면 기다리는 시간이 덜 불안해지며, 일정 편집을 통해 계획이 더욱 개인화되고, 내보내기 기능을 통해 계획을 공유 및 저장할 수 있으며, 측면 탐색을 통해 긴 페이지를 쉽게 탐색할 수 있습니다. 이러한 기능의 조합은 완전하고 사용자 친화적이며 실용적인 웹 애플리케이션을 형성합니다.

## 13.7 결론

13장을 완료한 것을 축하합니다!

이 장을 통해 완전한 지능형 여행 도우미 애플리케이션을 구축하는 방법을 배웠을 뿐만 아니라 더 중요하게는 다음 사항을 숙달했습니다.

1. **시스템 디자인 사고**: 복잡한 문제를 여러 개의 간단한 작업으로 분해하는 방법
2. **엔지니어링 실습 능력**: 이론적 지식을 실행 가능한 코드로 변환하는 방법
3. **풀스택 개발 능력**: 프론트엔드와 백엔드 기술 스택을 통합하는 방법
4. **AI 애플리케이션 개발**: LLM을 사용하여 실용적인 애플리케이션을 구축하는 방법

이 프로젝트는 끝점이 아닌 시작점입니다. 이 프로젝트를 기반으로 다음을 수행할 수 있습니다.

- 더 많은 기능 추가
- 사용자 경험 최적화
- 다른 도메인으로 확장 (such as intelligent shopping assistant, intelligent learning assistant, etc.)
- 실제 사용자에게 서비스를 제공하기 위해 프로덕션 환경에 배포

학습하는 가장 좋은 방법은 연습을 통해서입니다. 코드를 읽기만 하지 말고 직접 수정하고 확장하고 최적화하세요. 각 실습을 통해 다중 에이전트 시스템에 대한 이해가 깊어집니다.

귀하의 AI 애플리케이션 개발 여정에서 성공을 기원합니다!

