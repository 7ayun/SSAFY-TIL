# AI 특강 Ⅰ: AI 오케스트레이션 (Langflow 실습) 요약 정리

## 1. AI 오케스트레이션 (AI Orchestration) 이란?
단일 AI 모델이 모든 것을 처리하는 것이 아니라, **"AI가 다른 AI(에이전트)들을 지휘하고 통제하는 시스템"**입니다.
* **필요성:** 에이전트가 강력해지고 도구(API, DB 등)가 많아질수록 실행 경로가 복잡해집니다. 사용자가 대충 질문(게으른 프롬프트)을 던져도, 상위 지휘자(오케스트레이터)가 이를 분석하여 적절한 하위 에이전트(플래너, 제너레이터, 리뷰어 등)에게 역할을 분배하고 순서를 제어하여 최종 결과를 도출해야 높은 퀄리티의 서비스가 완성됩니다.
* **핵심 구조:** 한 명의 지휘자(Orchestrator) ↔ 다수의 역할별 하위 에이전트(Sub-Agents)

---

## 2. 랭플로우 (Langflow) 개요 및 핵심 컴포넌트
AI 에이전트와 파이프라인을 코딩 없이(No-code/Low-code) 시각적(GUI)으로 설계하고 테스트할 수 있는 오픈소스 도구입니다. 랭체인(Langchain) 기반으로 작동합니다.

### 주요 컴포넌트 (Components)
* **Chat Input / Output:** 사용자의 입력을 받고, 최종 AI의 답변을 출력하는 UI 역할.
* **Prompt Template:** LLM에게 페르소나와 제약조건을 부여하는 지시문.
* **Language Model (LLM):** 실제 두뇌 역할을 하는 모델 (실습에서는 OpenAI의 `GPT-4o-nano` 사용).
* **Message History:** LLM에게 이전 대화 기록(기억력)을 주입하는 컴포넌트.
* **API Request:** 외부 API(위키피디아 등)에 데이터를 요청하여 가져오는 도구.
* **Smart Transform:** API로 받아온 복잡한 데이터(JSON 등)를 LLM을 이용해 내가 원하는 형태의 자연어로 가공(전처리)하는 컴포넌트.
* **Agent:** 단순 텍스트 생성을 넘어, 스스로 판단(Reasoning)하고 주어진 **도구(Tool)**를 자율적으로 사용하는 똑똑한 모델.

---

## 3. 실습 1: 기본 챗봇 및 기억력(Memory) 추가
LLM은 기본적으로 **단일 요청-응답(Stateless)** 구조라 이전 대화를 기억하지 못합니다. 랭플로우에서는 이를 해결하기 위해 `Message History` 컴포넌트를 부착합니다.

> **주의사항:** 대화 기록이 쌓일수록 API 요청 시 전송되는 토큰(Token)량이 급증하므로, 실제 서비스 구현 시에는 적절한 시점에 과거 대화를 요약(Compress)하는 기술이 필수적입니다.

### [보충 설명] 파이썬(Langchain) 코드로 구현하는 메모리 추가
랭플로우의 GUI 연결은 내부적으로 아래와 같은 파이썬 랭체인 코드로 동작합니다.

```python
from langchain.chat_models import ChatOpenAI
from langchain.memory import ConversationBufferMemory
from langchain.chains import ConversationChain

# 1. LLM 모델 생성 (실습의 GPT-4o-nano 역할)
llm = ChatOpenAI(model_name="gpt-4o-mini", temperature=0.7)

# 2. 메모리 객체 생성 (Message History 컴포넌트 역할)
# 이전 대화 기록을 변수(history)에 저장하여 LLM에게 함께 전달함
memory = ConversationBufferMemory()

# 3. 체인 생성 (컴포넌트들을 선으로 연결하는 역할)
conversation = ConversationChain(
    llm=llm, 
    memory=memory,
    verbose=True
)

# 4. 대화 실행
response1 = conversation.predict(input="내 이름은 패트릭이야.")
response2 = conversation.predict(input="내 이름이 뭐라고 했지?") # 패트릭이라고 정확히 대답함
```

---

## 4. 실습 2: 위키피디아 API 연동 및 에이전트(Agent) 활용
일반 LLM 모델을 **에이전트(Agent)** 로 업그레이드하여, 사용자의 질문에 따라 자율적으로 위키피디아 검색 도구를 사용하도록 설계합니다.

1. **API Request를 도구(Tool)로 변환:** API Request 컴포넌트의 설정을 'Tool Mode'로 변경.
2. **에이전트 연결:** 에이전트 컴포넌트에 Prompt와 Tool(API Request)을 연결.
3. **자율 동작 (Reasoning):** 사용자가 "세종대왕에 대해 알려줘"라고 하면, 에이전트가 프롬프트를 분석 ➔ 위키피디아 API 도구를 호출(Action) ➔ 결과를 받아와서 답변을 생성.

---

## 5. 실습 3: AI 오케스트레이터 구축 (마스터 에이전트)
이번 실습의 핵심으로, 상황에 따라 다른 도구(서브 플로우)를 호출하는 상위 지휘자(Orchestrator)를 만듭니다.

* **서브 에이전트 1 (Ugly Agent):** 사용자가 '취미'를 말하면 무조건 부정적으로 답변하는 못난이 에이전트 (분기 테스트용).
* **서브 에이전트 2 (Wiki Agent):** 위키피디아 API를 사용해 정보를 찾아주는 플로우를 `Run Flow` 컴포넌트를 이용해 도구(Tool)로 가져옴.
* **오케스트레이터 에이전트:** * **프롬프트:** "직접 대답하지 말고, 상황에 따라 적절한 도구(Sub-Agent)를 선택해 그 답변을 그대로 반환해."
  * **결과:** 사용자가 취미를 물어보면 Ugly Agent를 호출하고, 역사적 사실을 물어보면 Wiki Agent를 호출하는 라우팅(Routing) 기능을 완벽히 수행함.

### [보충 설명] 파이썬 코드로 구현하는 오케스트레이터 (도구 선택)
오케스트레이터가 도구를 선택하고 사용하는 내부 로직의 파이썬 구현 예시입니다.

```python
from langchain.agents import initialize_agent, Tool, AgentType
from langchain.chat_models import ChatOpenAI

# 1. 하위 에이전트(도구) 역할 정의 함수
def ugly_hobby_response(query):
    return "그런 취미는 정말 시간 낭비인 것 같네요."

def wiki_search_response(query):
    # 실제 API 연동 로직이 들어가는 곳
    return f"위키피디아 검색 결과: {query}는 조선의 제4대 왕입니다."

# 2. 오케스트레이터에게 쥐어줄 도구(Tool) 리스트 세팅
tools = [
    Tool(
        name="Hobby_Critic_Tool",
        func=ugly_hobby_response,
        description="사용자가 본인의 '취미'나 '여가 활동'에 대해 이야기할 때 반드시 이 도구를 사용하세요."
    ),
    Tool(
        name="Wikipedia_Search_Tool",
        func=wiki_search_response,
        description="사용자가 역사, 인물, 지식 등에 대해 질문할 때 검색을 위해 사용하세요."
    )
]

# 3. 오케스트레이터 에이전트 초기화
llm = ChatOpenAI(model_name="gpt-4o-mini", temperature=0)
orchestrator_agent = initialize_agent(
    tools, 
    llm, 
    agent=AgentType.OPENAI_FUNCTIONS, # 함수(도구) 호출에 특화된 에이전트 타입
    verbose=True
)

# 4. 오케스트레이터 실행 (자동 라우팅)
# 취미를 물어보면 Hobby_Critic_Tool을, 정보를 물어보면 Wikipedia_Search_Tool을 자동 선택함
orchestrator_agent.run("내 취미는 그림 그리기야.") 
orchestrator_agent.run("세종대왕에 대해 알려줘.")
```

---

## 6. 랭플로우의 한계와 향후 활용 방안 (최종 프로젝트 연계)
* **한계:** 랭플로우는 빠르고 시각적인 **'설계 및 검증(Blueprint)' 용도**이지, 실제 프로덕션 서버로 배포하거나 복잡한 백엔드 비즈니스 로직을 모두 감당하기 위한 툴은 아닙니다. (실제 서비스는 파이썬 등 코드로 구현해야 함)
* **활용 방안 (1학기 최종 프로젝트):** 1. 팀원들과 프로젝트 기획 시, 복잡한 LLM 아키텍처(플래너-제너레이터-리뷰어 등)를 랭플로우로 먼저 그려보고 동작을 검증합니다.
  2. 잘 작동하는 플로우를 **JSON 형태로 Export(내보내기)** 하여 팀원들과 공유합니다.
  3. 설계도가 확정되면, 이를 바탕으로 파이썬 백엔드(Django, FastAPI 등)에 실제 랭체인(Langchain) 코드로 이식하여 SSAFY 최종 프로젝트 서비스를 완성합니다.