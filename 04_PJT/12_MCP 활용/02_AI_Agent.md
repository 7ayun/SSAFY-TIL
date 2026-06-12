# [관통 PJT] AI Agent

---

## 1. AI Agent란

지금까지 만든 MCP 서버는 **팔다리만 있는 기계**와 같다. 호출하면 동작하지만 스스로 생각하고 움직이지는 않는다.

여기에 **내부에 저장한 데이터**와 **프롬프트**가 결합되면 에이전트는 스스로 생각하고 행동하게 된다. 이것이 AI Agent다.

> 단순 Tool 호출 → 데이터 저장소 + 프롬프트 = **스스로 생각하고 행동하는 AI Agent**

---

## 2. ReAct 패턴 (Reasoning + Acting)

AI Agent의 핵심 동작 사이클. 계획하고, 행동하고, 결과를 관찰하는 과정을 반복한다.

```
계획(Thought) → 행동(Action) → 관찰(Observation) → (반복)
```

### 뉴스 보고서 AI Agent 기준 동작 흐름

| 단계 | 내용 |
|---|---|
| 1. 입력 | "뉴스 보고서 작성 해줘" |
| 2. 계획 | 프롬프트에 따라 refresh_news 실행 계획 |
| 3. 행동 | refresh_news() 실행 |
| 4. 관찰 | 업데이트 완료 결과 확인 |
| 5. 계획 | 관찰 결과가 성공 → news://latest 읽기 계획 |
| 6. 행동 | read_news_from_memory() 실행 |
| 7. 관찰 | 최신 뉴스를 확인 |
| 8. 최종 답변 | "안녕하십니까, 오늘의 보고입니다. …" |

### 주의사항

- 잘못 쓰면 **토큰이 미친듯이 날아간다**
- **무한루프**에 빠질 수 있다
- 프롬프트로 행동 순서를 명확하게 지시해야 한다

---

## 3. 뉴스 보고서 AI Agent 구현

기존 뉴스 크롤링 MCP 서버를 리팩토링해서 뉴스 보고서 AI Agent를 만든다.

### 목표

"브리핑 시작해" 와 같은 짧은 프롬프트를 입력하면, AI가 알아서 조건에 부합한 보고서를 만들어준다.

**보고서 조건:**
1. GeekNews와 AISparkUp 모두 확인
2. 중복된 내용은 제거
3. 말투는 "안녕하십니까, 오늘의 보고입니다"로 시작하는 비즈니스 톤
4. 마지막에 "한 줄의 인사이트" 포함

---

## 4. 데이터 저장소 설계

AI가 가져온 뉴스를 매번 채팅창으로 건네주면 **토큰이 낭비**된다. 내부 저장소를 활용하면 AI가 컨텍스트 주입 없이 직접 읽어서 활용할 수 있다.

```python
import time
mcp = FastMCP("news-server")

HEADERS = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 ..."
}

# 뉴스 데이터를 임시 저장할 캐시(Cache) 공간
news_storage = {
    "geeknews": "최신 기사 내용이 들어갈 value",
    "aisparkup": "최신 기사 내용이 들어갈 value"
}
last_updated = 0        # 마지막 업데이트 시간을 기록할 변수
CACHE_DURATION = 3600   # 캐시 유효 시간 (1시간)
```

---

## 5. 데이터 저장소 저장 — refresh_news()

크롤링 결과를 내부 저장소에 저장하는 함수. `force=True`로 설정하면 캐시 시간 무관하게 강제 새로고침한다.

```python
@mcp.tool()
async def refresh_news(force: bool = False) -> str:
    """
    뉴스를 크롤링하여 서버 메모리를 최신화합니다.
    기본적으로 1시간 이내에 업데이트된 기록이 있다면 크롤링을 생략합니다.
    Args:
        force: True로 설정하면 시간과 상관없이 강제로 다시 크롤링합니다.
    """
    global last_updated
    current_time = time.time()

    if not force and (current_time - last_updated < CACHE_DURATION):
        # 크롤링하지 않고 바로 리턴
        time_ago = int((current_time - last_updated) / 60)
        return f"이미 최신 데이터가 있습니다. ({time_ago}분 전 업데이트됨). 리소스를 바로 읽으세요."

    # 크롤링 수행
    geek_popular_data = await get_geeknews_popular()
    geek_new_data = await get_geeknews_new()
    ai_data = await get_ai_sparkup()

    # 결과를 반환하지 않고 메모리에 저장
    news_storage["geeknews"] = f"""
=== 인기 글 ===\n{geek_popular_data}
\n\n=== 최신 글 ===\n{geek_new_data}
"""
    news_storage["aisparkup"] = ai_data

    last_updated = current_time
    return "뉴스 업데이트 완료"
```

---

## 6. 데이터 저장소 읽기 — @mcp.resource

저장된 데이터를 AI가 읽을 수 있도록 **Resource**로 정의한다.

**LLM은 로컬 딕셔너리(`news_storage`)에 직접 접근할 수 없다.** 따라서 우회해서 데이터를 읽어오는 방식이 필요하다.

| 데코레이터 | 역할 |
|---|---|
| `@mcp.tool()` | AI가 **호출**하는 함수 (Action) |
| `@mcp.resource()` | AI가 **참조**하는 데이터 (Read-Only, 컨텍스트 제공 목적) |

```python
@mcp.resource("news://latest")
def get_cached_news() -> str:
    """서버 메모리에 저장된 최신 뉴스를 반환합니다."""
    return f"""
=== GeekNews 데이터 ===
{news_storage['geeknews']}

=== AI SparkUp 데이터 ===
{news_storage['aisparkup']}
"""

@mcp.tool()
async def read_news_from_memory() -> str:
    """
    서버 메모리(news://latest)에 저장된 뉴스 데이터를 읽어옵니다.
    뉴스 업데이트(refresh_news)가 끝난 후, 내용을 파악하기 위해 이 도구를 사용하세요.
    """
    # Resource 함수를 재사용해서 텍스트를 리턴
    return get_cached_news()
```

---

## 7. 프롬프트 작성 — @mcp.prompt

AI에게 지시하는 프롬프트가 담긴 함수를 의미한다. 채팅창에서 `/`를 입력하면 등록된 프롬프트 목록이 자동 완성으로 나타난다.

```python
@mcp.prompt()
def news_briefing() -> str:
    return """
    너는 기업 임원에게 보고하는 'AI 비서'야. 아래 절차대로 보고서를 작성해.
    아래 [핵심 원칙]을 최우선으로 준수하여 보고서를 작성해.

    [핵심 원칙: 무조건적 도구 실행]
    1. 기억 소거: 이전 대화 내용이나 채팅 기록에 있는 뉴스 데이터를 절대 재사용하지 마.
    2. 강제 실행: 네가 보기에 데이터가 이미 있는 것 같아도, 무조건 `refresh_news` 도구를 실제로 실행해야 해.
    3. 검증: 도구를 실행했다는 로그(Observation)가 없다면 보고서를 작성하지 마.

    [너의 목표]
    위 원칙에 따라 확보한 '진짜 최신 데이터'를 바탕으로 '모닝 뉴스 브리핑 보고서'를 작성하는 것.

    [사용 가능한 자원]
    - 뉴스를 업데이트하는 도구(`refresh_news`)
    - 내용을 읽는 도구(`read_news_from_memory`)

    [행동 지침]
    - 현재 상황을 스스로 판단하되, "도구 실행 없는 보고"는 직무 유기로 간주한다.
    - 반드시 `refresh_news` → `내용 읽기` → `작성`의 순서를 밟아라.

    [보고서 작성 규칙]
    1. 시작: "안녕하십니까, 오늘의 보고입니다"로 시작할 것.
    2. 내용: 중복된 내용은 합치고, 핵심만 요약할 것.
    3. 어조: 정중한 비즈니스 톤 사용.
    4. 마무리: 맨 마지막에 "한 줄의 인사이트"를 반드시 포함할 것.
    """
```

---

## 8. 동작 확인

채팅창에 `/news`를 입력하면 `/mcp.news-agent.news_briefing` 프롬프트가 자동완성으로 나타난다. 선택하면 프롬프트가 채팅창에 주입된다.

AI Agent는 다음 순서로 ReAct 사이클을 수행한다.

1. `refresh_news(force=True)` 실행 → "뉴스 업데이트 완료"
2. `read_news_from_memory()` 실행 → 저장된 뉴스 읽기
3. 보고서 작성 → "안녕하십니까, 오늘의 보고입니다…"

재호출 시에는 캐시 기간(1시간) 이내라면 기존 메모리를 재사용한다.

---

## 9. 앞으로의 도전 과제

오늘 만든 AI Agent는 VSCode Copilot이 MCP Host와 Client 역할을 대신 해주는 구조다. 더 나아가면 다음과 같은 것들을 만들 수 있다.

- **벡터 DB를 MCP 리소스로 연결** — 수만 개의 사내 문서를 MCP Server로 검색
- **OAuth 인증 구현** — "내 일정을 브리핑해줘", "MM에 메시지 보내줘" 같은 에이전트
- **나만의 LLM 기반 AI Agent** — `LangChain 1.0`으로 MCP Client 역할까지 직접 구현
  - 복잡한 ReAct 과정을 헷갈리지 않도록 플로우를 효율적으로 작성 가능

---

## 💡 한 줄 요약
> AI Agent는 ReAct 패턴(계획→행동→관찰 반복)으로 동작하며, 캐시 저장소(@mcp.resource)와 행동 지침(@mcp.prompt)이 결합될 때 비로소 스스로 생각하고 움직이는 에이전트가 된다.

## ❓ 더 찾아볼 것
- ReAct 패턴 원문 논문: "ReAct: Synergizing Reasoning and Acting in Language Models"
- LangChain 1.0 MCP Client 구현 방법
- MCP Agent와 토큰 비용 최적화 전략
- `@mcp.resource`의 URI 스키마 설계 방법
