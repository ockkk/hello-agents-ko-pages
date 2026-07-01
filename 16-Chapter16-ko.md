# 16장: 졸업 프로젝트 - 나만의 다중 에이전트 애플리케이션 구축

Hello-Agents 튜토리얼의 마지막 장에 도달한 것을 축하합니다! 이전 15개 장에서 우리는 HelloAgents 프레임워크를 처음부터 구축하고 핵심 에이전트 개념, 다중 패러다임, 도구 시스템, 메모리 메커니즘, 통신 프로토콜, 강화 학습 훈련 및 성능 평가에 대해 배웠습니다. 13~15장에서는 세 가지 완전한 실제 프로젝트(지능형 여행 도우미, 자동화된 심층 연구 에이전트 및 사이버 타운)를 통해 학습된 모든 지식을 통합하는 방법도 시연했습니다.

이제 진정한 에이전트 시스템 빌더가 될 때입니다! 이 장에서는 **자신만의 다중 에이전트 애플리케이션**을 구축하고 오픈 소스 협업을 통해 성과를 커뮤니티와 공유하는 방법을 안내합니다.

## 16.1 졸업 프로젝트의 의의

### 16.1.1 졸업 프로젝트를 하는 이유

기술을 배우는 가장 좋은 방법은 튜토리얼을 읽는 것이 아니라 **직접 연습**하는 것입니다. 이전 장을 통해 에이전트 시스템 구축을 위한 이론적 지식과 기술 도구를 마스터했습니다. 그러나 실제 과제는 **이 지식을 실제 문제에 어떻게 적용할 것인가?입니다. 완전한 시스템을 설계하는 방법은 무엇입니까? 다양한 극단적인 경우와 예외를 처리하는 방법은 무엇입니까?**

졸업 프로젝트의 핵심 가치는 이전에 학습한 모든 지식을 선택적으로 통합하여((agent paradigms, tool systems, memory mechanisms, communication protocols, etc.)) 포괄적인 응용 능력을 배양하는 것입니다.

이 장의 학습과 실습을 통해 완전한 에이전트 애플리케이션을 독립적으로 설계 및 구현하고, HelloAgents 프레임워크의 다양한 기능을 능숙하게 사용하고, 기본 Git 및 GitHub 작업을 마스터하고, 명확한 프로젝트 문서 작성 방법을 배우고, 오픈 소스 커뮤니티 공동 개발에 참여하고, 궁극적으로 선보일 수 있는 기술 작업을 얻을 수 있기를 바랍니다.

### 16.1.2 졸업 프로젝트의 형태

귀하의 졸업 프로젝트는 **오픈소스 프로젝트** 형식으로 Hello-Agents 공동작성 프로젝트 저장소(`Co-creation-projects` 디렉터리)에 제출됩니다. 구체적인 요구사항은 다음과 같습니다.

1. **프로젝트 이름 지정**: `{your-GitHub-username}-{project-name}` 형식을 사용합니다(예: `jjyaoao-CodeReviewAgent`).

2. **프로젝트 내용**:
   - 실행 가능한 Jupyter Notebook(`.ipynb` 파일) 또는 Python 스크립트
   - 전체 종속성 목록(`requirements.txt`)
   - README 문서 지우기(`README.md`)
   - 선택사항: 데모 동영상, 스크린샷, 데이터세트 등

3. **제출 방법**: GitHub Pull Request(PR)를 통해 제출

4. **검토 프로세스**: 커뮤니티 구성원이 코드를 검토하고 개선 제안을 제공하며 승인 후 기본 저장소에 병합됩니다.

## 16.2 프로젝트 주제 선정 가이드

16.2.1 주제 선택 원칙

좋은 졸업 프로젝트는 기술을 위한 기술보다는 실제 문제를 해결하는 실용적이어야 합니다. 제한된 시간과 자원 내에서 완성도를 추구하면서 기술력을 확실하게 입증해야 합니다.

### 16.2.2 추천 주제 방향

다음은 권장되는 프로젝트 방향입니다. 하나를 선택하거나 자신만의 아이디어를 제안할 수 있습니다.

**(1) 생산성 도구**

- **지능형 코드 검토 도우미**: 자동으로 코드 품질을 분석하고, 잠재적인 버그를 발견하고, 최적화 제안을 제공합니다.
- **지능형 문서 생성기**: 코드를 기반으로 API 문서 및 사용자 매뉴얼을 자동으로 생성합니다.
- **지능형 회의 도우미**: 회의 내용 기록, 회의록 생성, 작업 항목 추출
- **지능형 이메일 도우미**: 이메일 자동 분류, 회신 초안 생성, 중요한 사항 알림

**(2) 학습 지원**

- **지능형 학습 파트너**: 학습 진행 상황에 따른 학습 리소스 추천, 연습 문제 생성, 질문 답변
- **지능형 논문 도우미**: 문헌 검색, 논문 요약, 인용 생성 지원
- **지능형 프로그래밍 튜터**: 프로그래밍 연습, 코드 검토, 학습 경로 계획 제공
- **지능형 언어 학습 도우미**: 대화 연습, 문법 교정, 어휘 확장 제공

**(3) 창의적인 엔터테인먼트**

- **지능형 스토리 생성기**: 사용자 입력을 기반으로 소설, 대본, 시 생성
- **지능형 게임 NPC**: 플레이어와 자연스럽게 대화할 수 있는 개성 넘치는 게임 캐릭터 생성
- **지능형 음악 추천**: 분위기와 장면에 따른 음악 추천, 재생목록 생성
- **지능형 레시피 도우미**: 재료와 맛을 바탕으로 레시피 추천, 쇼핑 목록 생성

**(4) 데이터 분석**

- **지능형 데이터 분석가**: 자동으로 데이터 분석, 시각화 차트 생성, 분석 보고서 작성
- **지능형 주식 분석**: 주식 데이터 및 뉴스 심리를 분석하고 투자 조언을 제공합니다.
- **지능형 여론 모니터링**: 소셜 미디어 및 뉴스 웹사이트 모니터링, 여론 동향 분석
- **지능형 경쟁 분석**: 경쟁사 정보 수집, 비교 분석, 보고서 생성

**(5) 생활 서비스**

- **지능형 건강 도우미**: 건강 데이터 기록, 건강 조언 제공, 운동 계획 수립
- **지능형 재정 보조원**: 수입과 지출 기록, 지출 습관 분석, 재정 조언 제공
- **지능형 쇼핑 도우미**: 가격 비교, 제품 추천, 쇼핑 목록 생성
- **지능형 홈 제어**: 자연어를 통해 스마트 홈 기기를 제어합니다.

### 16.2.3 주제 선택 예시

구체적인 예를 통해 주제를 선택하고 프로젝트를 디자인하는 방법을 설명하겠습니다.

**프로젝트 이름**: 지능형 코드 검토 도우미(CodeReviewAgent)

**문제 분석**: 코드 검토는 소프트웨어 개발의 중요한 부분이지만 수동 검토에는 시간이 많이 걸리고 문제가 누락되기 쉽습니다. 기존 정적 분석 도구는 구문 오류만 찾아낼 수 있고 코드 논리를 이해할 수 없기 때문에 코드 의미를 이해하고 심층적인 분석을 제공할 수 있는 지능형 비서가 필요합니다.

**핵심 기능**: 이 프로젝트는 코드 품질 분석(코드 스타일, 명명 규칙, 주석 완전성 확인), 잠재적 버그 감지(논리 오류, 경계 조건 문제, 리소스 누수 발견), 성능 최적화 제안(성능 병목 현상 식별, 최적화 솔루션 제안), 보안 취약성 검사(SQL 주입, XSS 및 기타 보안 문제 감지), 모범 사례 권장 사항(언어 기능 및 디자인 패턴을 기반으로 개선 제안)을 구현합니다.

**예상 결과**: 최종 결과물은 전체 검토 프로세스를 시연하고, Python 및 JavaScript와 같은 주류 언어를 지원하고, 구조화된 Markdown 형식 검토 보고서를 생성할 수 있으며, 특정 코드 예제 및 개선 제안을 제공하는 실행 가능한 Jupyter Notebook이 될 것입니다.

## 16.3 개발 환경 준비

### 16.3.1 필요한 도구 설치

개발을 시작하기 전에 개발 환경에 다음 도구가 설치되어 있는지 확인하십시오.

**(1) Python 환경**

```bash
# Install HelloAgents
pip install "hello-agents[all]"
```

**(2) Git 및 GitHub**

```bash
# Check Git version
git --version

# Configure Git user information
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Configure GitHub SSH key (recommended)
# 1. Generate SSH key
ssh-keygen -t ed25519 -C "your.email@example.com"

# 2. Add public key to GitHub
# Copy the content of ~/.ssh/id_ed25519.pub
# Add in GitHub Settings > SSH and GPG keys

# 3. Test connection
ssh -T git@github.com
```

**(3) 주피터 노트북**

```bash
# Install Jupyter
pip install jupyter notebook

# Or use JupyterLab (recommended)
pip install jupyterlab

# Start Jupyter
jupyter lab
```

### 16.3.2 프로젝트 저장소 포크

**1단계: 저장소 포크**

1. Hello-Agents 저장소를 방문하세요: https://github.com/datawhalechina/hello-agents
2. 그림 16.1의 빨간색 상자에 표시된 대로 오른쪽 상단 모서리에 있는 "포크" 버튼을 클릭합니다.
3. GitHub 계정을 선택하고 포크를 생성합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/hello-agents/main/docs/images/16-figures/16-1.png" alt="" width="85%"/>
<p>그림 16.1 포크 저장소 단계</p>
</div>

**2단계: 로컬에 복제**

```bash
# As shown in Figure 16.2, clone your forked repository
git clone git@github.com:your-username/hello-agents.git

# Enter project directory
cd Hello-Agents

# Add upstream repository (for syncing updates)
git remote add upstream https://github.com/datawhalechina/hello-agents.git

# View remote repositories
git remote -v
```

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/hello-agents/main/docs/images/16-figures/16-2.png" alt="" width="85%"/>
<p>그림 16.2 로컬에 저장소 복제</p>
</div>

**3단계: 개발 분기 생성**

```bash
# Create and switch to new branch
git checkout -b feature/your-project-name

# For example:
git checkout -b feature/code-review-agent
```

### 16.3.3 프로젝트 디렉토리 구조

`Co-creation-projects` 디렉터리에 프로젝트 폴더를 만듭니다.

```bash
# Enter co-creation projects directory
cd Co-creation-projects

# Create project folder (format: GitHub-username-project-name)
mkdir your-username-project-name

# For example:
mkdir jjyaoao-CodeReviewAgent

# Enter project directory
cd jjyaoao-CodeReviewAgent
```

권장 프로젝트 구조:

```
jjyaoao-CodeReviewAgent/
├── README.md              # Project documentation
├── requirements.txt       # Python dependency list
├── main.ipynb            # Main Jupyter Notebook
├── data/                 # Data files (optional)
│   ├── sample_code.py
│   └── test_cases.json
├── outputs/              # Output results (optional)
│   ├── review_report.md
│   └── screenshots/
├── src/                  # Source code (optional, if code is extensive)
│   ├── agents/
│   ├── tools/
│   └── utils/
└── .env.example          # Environment variable template
```

## 16.4 프로젝트 개발 가이드

### 16.4.1 README 문서 작성

README는 프로젝트의 얼굴입니다. 좋은 README에는 다음 내용이 포함되어야 합니다.

```markdown
# Project Name

> One-sentence description of your project

## 📝 Project Introduction

Detailed introduction to your project:
- What problem does it solve?
- What are its special features?
- What scenarios is it suitable for?

## ✨ Core Features

- [ ] Feature 1: Description
- [ ] Feature 2: Description
- [ ] Feature 3: Description

## 🛠️ Technology Stack

- HelloAgents framework
- Agent paradigms used (e.g., ReAct, Plan-and-Solve, etc.)
- Tools and APIs used
- Other dependency libraries

## 🚀 Quick Start

### Environment Requirements

- Python 3.10+
- Other requirements

### Install Dependencies


pip install -r requirements.txt


### Configure API Keys


# Create .env file
cp .env.example .env

# Edit .env file and fill in your API keys


### Run Project


# Start Jupyter Notebook
jupyter lab

# Open main.ipynb and run


## 📖 Usage Examples

Show how to use your project, preferably with code examples and results.

## 🎯 Project Highlights

- Highlight 1: Explanation
- Highlight 2: Explanation
- Highlight 3: Explanation

## 📊 Performance Evaluation

If you have evaluation results, display them here:
- Accuracy: XX%
- Response time: XX seconds
- Other metrics

## 🔮 Future Plans

- [ ] Feature 1 to be implemented
- [ ] Feature 2 to be implemented
- [ ] Parts to be optimized

## 🤝 Contribution Guidelines

Issues and Pull Requests are welcome!

## 📄 License

MIT License

## 👤 Author

- GitHub: [@your-username](https://github.com/your-username)
- Email: your.email@example.com (optional)

## 🙏 Acknowledgments

Thanks to the Datawhale community and Hello-Agents project!
```

### 16.4.2 요구사항.txt 작성

프로젝트에 필요한 모든 Python 종속성을 나열합니다.

```txt
# Core dependencies
hello-agents[all]>=0.2.7

# Visualization (if needed)
matplotlib>=3.7.0
plotly>=5.14.0

# Web framework (if needed)
fastapi>=0.109.0
uvicorn>=0.27.0
```

### 16.4.3 주피터 노트북 개발하기

**(1) 노트북 구조 권장 사항**

좋은 Jupyter Notebook에는 다음 부분이 포함되어야 합니다.

```python
# ========================================
# Part 1: Project Introduction
# ========================================

"""
# Project Name

## Project Introduction
Brief introduction to project goals and features

## Author Information
- Name: XXX
- GitHub: @XXX
- Date: 2025-XX-XX
"""

# ========================================
# Part 2: Environment Configuration
# ========================================

# Install dependencies
!pip install -q hello-agents[all]

# Import necessary libraries
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import BaseTool
import os
from dotenv import load_dotenv

# Load environment variables
load_dotenv()

# ========================================
# Part 3: Tool Definition
# ========================================

class CustomTool(BaseTool):
    """Custom tool class"""

    name = "tool_name"
    description = "Tool description"

    def run(self, query: str) -> str:
        """Tool execution logic"""
        # Implement your tool logic
        return "Result"

# ========================================
# Part 4: Agent Construction
# ========================================

# Create LLM
llm = HelloAgentsLLM()

# Create agent
agent = SimpleAgent(
    name="Agent Name",
    llm=llm,
    system_prompt="System prompt"
)

# Add tools
agent.add_tool(CustomTool())

# ========================================
# Part 5: Feature Demonstration
# ========================================

# Example 1: Basic functionality
print("=== Example 1: Basic Functionality ===")
result = agent.run("User input")
print(result)

# Example 2: Complex scenario
print("\n=== Example 2: Complex Scenario ===")
result = agent.run("Complex user input")
print(result)

# ========================================
# Part 6: Performance Evaluation (Optional)
# ========================================

# Evaluation code
# ...

# ========================================
# Part 7: Summary and Outlook
# ========================================

"""
## Project Summary

### Implemented Features
- Feature 1
- Feature 2

### Challenges Encountered
- Challenge 1 and solution
- Challenge 2 and solution

### Future Improvement Directions
- Improvement 1
- Improvement 2
"""
```

### 16.4.4 프로젝트 테스트

제출하기 전에 이 체크리스트를 사용하여 프로젝트가 제출 요구 사항을 충족하는지 확인하세요.

```markdown
- [ ] Code runs normally without errors
- [ ] README documentation is complete with clear instructions
- [ ] requirements.txt contains all dependencies
- [ ] Clear usage examples provided
- [ ] Code has appropriate comments
- [ ] Output results meet expectations
- [ ] Common exception cases handled
- [ ] Project structure is clear with standardized file naming
- [ ] Large files properly handled (see next section)
```

### 16.4.5 대용량 파일 처리 가이드

**⚠️ 중요: 너무 큰 메인 저장소는 피하세요**

Hello-Agents 기본 저장소를 경량으로 유지하려면 다음 대용량 파일 처리 지침을 따르십시오.

**(1) 파일 크기 제한**

- **총 프로젝트 크기**: 5MB 이하
- **직접 제출 금지**: 동영상 파일, 대용량 데이터세트, 모델 파일

**(2) 대용량 파일 처리 솔루션**

프로젝트에 대용량 파일((datasets, videos, models, etc.))이 포함된 경우 다음 해결 방법을 사용하세요.

**해결책 1: 외부 링크 사용(권장)**

외부 플랫폼에 대용량 파일을 업로드하고 README에 다운로드 링크를 제공하세요.

```markdown
## Datasets

The datasets used in this project are large. Please download from the following links:

- Dataset 1: [Baidu Netdisk](link) Extraction code: xxxx
- Dataset 2: [Google Drive](link)
- Demo video: [Bilibili](link) / [YouTube](link)
```

권장되는 외부 플랫폼:
- **데이터세트**: Baidu Netdisk, Google Drive, Kaggle, HuggingFace 데이터세트
- **동영상**: Bilibili, YouTube, Tencent Video
- **모델**: HuggingFace 모델, ModelScope
- **이미지**: GitHub 문제, 이미지 호스팅 서비스

**해결책 2: 독립 저장소 생성**

프로젝트에 리소스가 많은 경우 독립적인 데이터 저장소 생성을 고려하세요.

```markdown
## Project Resources

Due to the large amount of data and demo resources, a separate resource repository has been created:

- Resource repository: https://github.com/your-username/project-name-resources
- Contains: Datasets, demo videos, model files, test data, etc.

### Usage

\`\`\`bash
# Clone resource repository
git clone https://github.com/your-username/project-name-resources.git

# Copy data to project directory
cp -r project-name-resources/data ./data
\`\`\`
```

**해결책 3: 샘플 데이터 사용**

기본 저장소에는 소규모 샘플 데이터만 제공하십시오.

```python
# Explain in README
## Data Description

- `data/sample.csv`: Sample data (100 records)
- Complete dataset (100,000 records) download from [here](link)
```

**(3) 모범 사례 예**

```
your-username-project-name/
├── README.md              # Contains external resource links
├── requirements.txt
├── main.ipynb
├── .gitignore            # Ignore large files
├── data/
│   └── sample.csv        # Sample data only (<1MB)
└── outputs/
    └── demo_result.png   # Demo results only (<1MB)
```

읽어보기 설명:

```markdown
## Data and Resources

### Sample Data
Project includes small-scale sample data for quick testing (located in `data/sample.csv`)

### Complete Dataset
Complete dataset (500MB) download from the following link:
- Baidu Netdisk: [Link] Extraction code: xxxx
- Extract to `data/` directory after download

### Demo Video
- Bilibili: [Project Demo Video](link)
- YouTube: [Demo Video](link)
```

## 16.5 풀 요청 제출

16.5.1 GitHub에 코드 제출

**1단계: 수정 사항 확인**

```bash
# View modified files
git status
```

**2단계: 파일 추가**

```bash
# Add all modified files
git add .

# Or add specific files
git add Co-creation-projects/your-username-project-name/
```

**3단계: 변경사항 커밋**

커밋 메시지는 다음 형식을 따라야 합니다.

```bash
# Format: type: brief description
git commit -m "feat: Add XXX graduation project"
```

**커밋 유형 사양:**

- `feat`: 새로운 기능이나 프로젝트(졸업 프로젝트에 이 유형을 사용)
- `fix`: 버그 수정
- `docs`: 문서 업데이트
- `style`: 코드 형식 조정(기능에는 영향을 주지 않음)
- `refactor`: 코드 리팩토링
- `test`: 테스트 관련
- `chore`: 기타 수정 사항 (e.g., dependency updates)

**4단계: GitHub로 푸시**

```bash
# Push to your forked repository
git push origin feature/your-project-name
```

### 16.5.2 풀 리퀘스트 생성

**1단계: GitHub 방문**

1. 포크된 저장소를 방문하세요: `https://github.com/your-username/hello-agents`
2. 그림 16.3과 같이 "풀 요청" 탭을 클릭합니다.
3. "새 끌어오기 요청" 버튼을 클릭하세요.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/hello-agents/main/docs/images/16-figures/16-3.png" alt="" width="85%"/>
<p>그림 16.3 풀 요청 생성</p>
</div>

**2단계: 지점 선택**

- 기본 저장소: `datawhalechina/hello-agents`
- 기본 분기: `main`
- 헤드 저장소: `your-username/hello-agents`
- 브랜치 비교: `feature/your-project-name`

**3단계: 홍보 정보 입력**

**⚠️ 중요: 통합 PR 제목 형식**

간편한 관리 및 검색을 위해 모든 졸업 프로젝트 홍보 제목은 다음 형식을 따라야 합니다.

```
[Graduation Project] Project Name - Brief Description
```

예:
- `[Graduation Project] CodeReviewAgent - Intelligent Code Review Assistant`
- `[Graduation Project] StudyBuddy - AI Learning Partner`
- `[Graduation Project] DataAnalyst - Intelligent Data Analyst`

**PR 설명 템플릿:**

```markdown
## Project Information

- **Project Name**: XXX
- **Author**: @your-username
- **Project Type**: Productivity Tool/Learning Assistance/Creative Entertainment/Data Analysis/Life Service

## Project Introduction

Brief description of your project (2-3 sentences)

## Core Features

- [ ] Feature 1
- [ ] Feature 2
- [ ] Feature 3

## Technical Highlights

- Used XXX paradigm
- Implemented XXX functionality
- Optimized XXX performance

## Demo Effects

(Optional) Add screenshots or GIFs to showcase project effects

## Self-Check List

- [ ] Code runs normally
- [ ] README documentation complete
- [ ] requirements.txt complete
- [ ] Clear usage examples provided
- [ ] Code has appropriate comments

## Other Notes

(Optional) Other content that needs explanation
```

**4단계: PR 제출**

그림 16.4와 같이 "Create pull request" 버튼을 클릭하여 제출합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/hello-agents/main/docs/images/16-figures/16-4.png" alt="" width="85%"/>
<p>그림 16.4 풀 요청 제출</p>
</div>

### 16.5.3 리뷰 코멘트에 응답하기

PR을 제출하면 커뮤니티 구성원이 코드를 검토하고 제안을 제공합니다. 즉시 응답해 주십시오:

1. **댓글 보기**: PR 페이지에서 리뷰어 댓글을 확인하세요.
2. **코드 수정**: 제안에 따라 코드 수정
3. **업데이트 제출**:
   ```bash
   git add .
   git commit -m "fix: Modify XXX based on review comments"
   git push origin feature/your-project-name
   ```
4. **댓글에 답장**: GitHub의 리뷰어에게 수정 사항을 설명하는 답장을 보냅니다.

## 16.6 프로젝트 쇼케이스 예시

졸업 프로젝트 요구 사항을 더 잘 이해하는 데 도움이 되도록 전체 예제 프로젝트가 있습니다. 걱정하지 마세요. 작고 창의적인 아이디어도 포함될 수 있습니다. 당신이 직접 만든 모든 작품은 소중히 간직할 가치가 있습니다.

**프로젝트 정보**

- **프로젝트 이름**: CodeReviewAgent
- **작성자**: @jjyaoao
- **프로젝트 경로**: `Co-creation-projects/jjyaoao-CodeReviewAgent/`

**프로젝트 구조**

```
jjyaoao-CodeReviewAgent/
├── README.md              # Project documentation
├── requirements.txt       # Dependency list
├── main.ipynb            # Main program (includes quick demo and full features)
├── .env.example          # Environment variable example
├── .gitignore            # Git ignore rules
├── data/
│   └── sample_code.py    # Sample code
└── outputs/
    └── review_report.md  # Sample report
```

**핵심 코드 조각 (main.ipynb)**

```python
# ========================================
# Intelligent Code Review Assistant
# ========================================

from hello_agents import SimpleAgent, HelloAgentsLLM, ToolRegistry
from hello_agents.tools import Tool, ToolParameter
from typing import Dict, Any, List
import ast
import os

# ========================================
# 0. Configure LLM Parameters
# ========================================

os.environ["LLM_MODEL_ID"] = "Qwen/Qwen2.5-72B-Instruct"
os.environ["LLM_API_KEY"] = "your_api_key_here"
os.environ["LLM_BASE_URL"] = "https://api-inference.modelscope.cn/v1/"
os.environ["LLM_TIMEOUT"] = "60"

# ========================================
# 1. Define Code Analysis Tools
# ========================================

class CodeAnalysisTool(Tool):
    """Code static analysis tool"""

    def __init__(self):
        super().__init__(
            name="code_analysis",
            description="Analyze Python code structure, complexity, and potential issues"
        )

    def run(self, parameters: Dict[str, Any]) -> str:
        """Analyze code and return results"""
        code = parameters.get("code", "")
        if not code:
            return "Error: Code cannot be empty"

        try:
            tree = ast.parse(code)
            functions = [node for node in ast.walk(tree)
                        if isinstance(node, ast.FunctionDef)]
            classes = [node for node in ast.walk(tree)
                      if isinstance(node, ast.ClassDef)]

            result = {
                "Number of functions": len(functions),
                "Number of classes": len(classes),
                "Lines of code": len(code.split('\n')),
                "Function list": [f.name for f in functions],
                "Class list": [c.name for c in classes]
            }
            return str(result)
        except SyntaxError as e:
            return f"Syntax error: {str(e)}"

    def get_parameters(self) -> List[ToolParameter]:
        return [
            ToolParameter(
                name="code",
                type="string",
                description="Python code to analyze",
                required=True
            )
        ]

class StyleCheckTool(Tool):
    """Code style checking tool"""

    def __init__(self):
        super().__init__(
            name="style_check",
            description="Check if code complies with PEP 8 standards"
        )

    def run(self, parameters: Dict[str, Any]) -> str:
        """Check code style"""
        code = parameters.get("code", "")
        if not code:
            return "Error: Code cannot be empty"

        issues = []
        lines = code.split('\n')
        for i, line in enumerate(lines, 1):
            if len(line) > 79:
                issues.append(f"Line {i}: Exceeds 79 characters")
            if line.startswith(' ') and not line.startswith('    '):
                if len(line) - len(line.lstrip()) not in [0, 4, 8, 12]:
                    issues.append(f"Line {i}: Non-standard indentation")

        if not issues:
            return "Code style is good, complies with PEP 8 standards"
        return "Found the following issues:\n" + "\n".join(issues)

    def get_parameters(self) -> List[ToolParameter]:
        return [
            ToolParameter(
                name="code",
                type="string",
                description="Python code to check",
                required=True
            )
        ]

# ========================================
# 2. Create Tool Registry and Agent
# ========================================

# Create tool registry
tool_registry = ToolRegistry()
tool_registry.register_tool(CodeAnalysisTool())
tool_registry.register_tool(StyleCheckTool())

# Initialize LLM
llm = HelloAgentsLLM()

# Define system prompt
system_prompt = """You are an experienced code review expert. Your tasks are:

1. Use code_analysis tool to analyze code structure
2. Use style_check tool to check code style
3. Based on analysis results, provide detailed review report

The review report should include:
- Code structure analysis
- Style issues
- Potential bugs
- Performance optimization suggestions
- Best practice recommendations

Please output the report in Markdown format."""

# Create agent
agent = SimpleAgent(
    name="Code Review Assistant",
    llm=llm,
    system_prompt=system_prompt,
    tool_registry=tool_registry
)

# ========================================
# 3. Run Example
# ========================================

# Read sample code
with open("data/sample_code.py", "r", encoding="utf-8") as f:
    sample_code = f.read()

print("=== Code to Review ===")
print(sample_code)
print("\n" + "="*50 + "\n")

# Execute code review
print("=== Starting Code Review ===")
review_result = agent.run(f"Please review the following Python code:\n\n```python\n{sample_code}\n```")

print(review_result)

# Save review report
with open("outputs/review_report.md", "w", encoding="utf-8") as f:
    f.write(review_result)

print("\nReview report saved to outputs/review_report.md")
```

**README.md 예**

```markdown
# CodeReviewAgent - Intelligent Code Review Assistant

> Intelligent code review tool based on HelloAgents framework

## 📝 Project Introduction

CodeReviewAgent is an intelligent code review assistant that can automatically analyze Python code quality, discover potential issues, and provide optimization suggestions.

### Core Features

- ✅ Code structure analysis: Count functions, classes, lines of code, etc.
- ✅ Style checking: Check compliance with PEP 8 standards
- ✅ Intelligent suggestions: Provide in-depth analysis and optimization suggestions based on LLM
- ✅ Report generation: Generate review reports in Markdown format

## 🛠️ Technology Stack

- HelloAgents framework (SimpleAgent + ToolRegistry)
- Python AST module (code parsing)
- ModelScope API (Qwen2.5-72B model)

## 🚀 Quick Start

### Install Dependencies

\`\`\`bash
pip install -r requirements.txt
\`\`\`

### Configure LLM Parameters

**Method 1: Use .env file**

\`\`\`bash
cp .env.example .env
# Edit .env file and fill in your API key
\`\`\`

**Method 2: Set directly in Notebook**

The project is pre-configured with ModelScope API and can run directly. To modify, edit the configuration code in Part 1 of main.ipynb.

### Run Project

\`\`\`bash
jupyter lab
# Open main.ipynb and run all cells
\`\`\`

## 📖 Usage Example

1. Place code to review in `data/sample_code.py`
2. Run `main.ipynb`
3. View generated review report `outputs/review_report.md`

## 🎯 Project Highlights

- **Automation**: No need for manual line-by-line checking, automatically discovers issues
- **Intelligence**: Uses LLM to understand code semantics and provide in-depth suggestions
- **Extensibility**: Easy to add new checking rules and tools

## 👤 Author

- GitHub: [@jjyaoao](https://github.com/jjyaoao)
- Project link: [CodeReviewAgent](https://github.com/datawhalechina/hello-agents/tree/main/Co-creation-projects/jjyaoao-CodeReviewAgent)

## 🙏 Acknowledgments

Thanks to the Datawhale community and Hello-Agents project!
```

## 16.7 요약 및 전망

졸업 프로젝트를 완료하면 요구 사항에 따른 시스템 아키텍처 설계, HelloAgents 프레임워크의 다양한 기능과 구성 요소를 능숙하게 사용, 에이전트 기능 확장을 위한 사용자 정의 도구 개발, 요구 사항 분석에서 코드 구현까지 전체 프로젝트 개발 완료, 오픈 소스 협업을 위한 Git 및 GitHub 사용 방법 학습, 명확한 기술 문서 작성 등 에이전트 시스템 설계의 전체 프로세스를 마스터해야 합니다.

이 프로젝트에서는 HelloAgents 프레임워크를 처음부터 구축하고 이를 사용하여 여러 실제 애플리케이션을 구현했습니다. 졸업 프로젝트를 완료하는 것은 시작에 불과합니다. 더 많은 에이전트 패러다임 및 알고리즘, 프롬프트 엔지니어링 및 컨텍스트 엔지니어링, 다중 에이전트 협업 메커니즘 및 기타 이론적 지식에 대해 계속해서 심화 학습할 수 있습니다. 완전한 애플리케이션을 구축하기 위한 웹 개발, 데이터 지속성을 구현하기 위한 데이터베이스 학습, 온라인 애플리케이션 실행을 위한 배포 학습을 통해 기술 스택을 확장할 수도 있습니다. 또한 더 많은 기능을 추가하고, 성능 및 사용자 경험을 최적화하고, 테스트 및 문서화를 개선하여 프로젝트를 지속적으로 최적화할 수도 있습니다. 더 중요한 것은 다른 학습자를 돕고, Hello-Agents 프레임워크 개발에 참여하고, 경험과 통찰력을 공유함으로써 커뮤니티 기여에 적극적으로 참여하는 것입니다.

1장의 간단한 에이전트부터 이제 완전한 다중 에이전트 애플리케이션을 독립적으로 구축할 수 있게 되기까지 흥미로운 학습 여정을 거쳤습니다. 그러나 이것은 끝이 아닙니다. 새로운 시작입니다.

AI 기술은 빠르게 변화하고 있으며 에이전트 분야는 무한한 가능성으로 가득 차 있습니다. 호기심을 유지하고 새로운 기술을 지속적으로 학습하며, AI 기술을 용기 있게 활용하여 실질적인 문제를 해결하고 가치를 창출하며, 자신의 경험과 성과를 커뮤니티와 기꺼이 공유하며, 탁월함을 추구하기 위해 자신의 작업을 끊임없이 개선해 나가기를 바랍니다.

마지막으로, 이 프로젝트 전체를 읽어주셔서 감사합니다. 학습 과정을 통해 무엇인가를 얻었기를 바라며, 배운 내용을 실제 프로젝트에 적용하여 놀라운 에이전트 애플리케이션을 만들 수 있기를 바랍니다. AI의 미래는 무한한 가능성으로 가득 차 있습니다. 함께 탐험하고 창조해 보세요!

**기억하세요: 학습하는 가장 좋은 방법은 직접 실습하는 것입니다!**

이제 나만의 에이전트 애플리케이션 구축을 시작해 보세요! Co-creation-projects 디렉토리에서 여러분의 훌륭한 작품을 만나기를 기대합니다!

Hello-Agents 프로젝트가 도움이 되었다면 ⭐별표를 남겨주세요!

---
<div align="center">
<strong>🎓 Hello-Agents 튜토리얼 완료를 축하합니다! 🎉</strong>
</div>

