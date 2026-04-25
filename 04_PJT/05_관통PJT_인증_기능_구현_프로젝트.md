# [관통 PJT] 인증 기능 구현 프로젝트 — 리팩토링 / Structured Output / AbstractUser

> 📌 핵심 키워드: #리팩토링 #기술부채 #클린코드 #StructuredOutput #OpenAI #AbstractUser #ModelField #choices

---

## 학습 목표

* 좋은 코드와 돌아가는 코드의 차이를 이해하고, 리팩토링이 필요한 이유를 설명할 수 있다
* 리팩토링 6가지 기법(이름 바꾸기, 매직넘버 교체, 함수 추출, 조건문 분해, 매개변수 객체, 주석달기)을 적용할 수 있다
* OpenAI Structured Output을 활용하여 일관된 형식의 JSON 응답을 받을 수 있다
* `form.save(commit=False)`를 이용해 OpenAI 응답 데이터를 Django 모델에 함께 저장할 수 있다
* `AbstractUser`와 `ModelField choices`를 활용해 사용자 모델과 필드를 확장할 수 있다

---

## 1. 리팩토링

### 1-1. 좋은 코드란 무엇인가

좋은 코드라는 개념은 매우 주관적이다. 중요한 것은 **돌아가는 코드와 좋은 코드는 다르다**는 점이다. 좋은 코드도 돌아가고 안 좋은 코드도 돌아간다. 결국 좋은 코드를 짠다는 건 **미래의 나와 팀원과의 의사소통 수단**이다. 코드를 대충 짜면 미래의 내가 고스란히 피해를 본다.

복잡하고 지저분하며 대충 짠 코드를 **스파게티 코드**라고 한다. 면이 꼬이는 것처럼 어디에 뭐가 있는지 알 수 없고 버그가 발생하기 쉽다.

| 구분 | 설명 |
|------|------|
| 돌아가는 코드 | 결과값만 맞으면 되는 코드 |
| 좋은 코드 | 가독성, 유지보수성, 협업 가능성을 갖춘 코드 |
| 스파게티 코드 | 복잡하고 지저분하게 얽혀 유지보수가 어려운 코드 |

### 1-2. 기술 부채

기술 부채(Technical Debt)는 시간이 없어서 퀄리티를 낮춰 개발할 때 쌓인다. 마감 기한이 쫓겨 대충 짜면 그 빚이 나중에 3~4배의 작업 시간으로 돌아온다.

일정이 빠듯할 때 개발자가 취할 수 있는 현명한 방법은 다음과 같다. 팀장이나 대표에게 못하겠다고 무작정 얘기하는 것이 아니라, 총 소요 시간을 계산해서 수치로 보고하고 우선순위가 낮은 기능을 빼달라고 협상하는 것이다.

### 1-3. 리팩토링이란

리팩토링은 **기존의 동작은 그대로 두고 내부 구조를 개선**하는 작업이다. 리팩토링을 주기적으로 해야 기술 부채를 해결하고, 잠재적인 버그를 미리 발견하며, 추후 기능 개발 속도를 높일 수 있다.

오늘 강의의 리팩토링 내용은 **리팩토링(마틴 파울러)**과 **클린코드(로버트 마틴)** 두 권을 종합하여 만든 내용이다. 절대적인 정답이 아니며, 여러분 각자의 개발 철학에 따라 유동적으로 흡수해야 한다.

---

## 2. 리팩토링 6가지 기법

### 2-1. 변수와 함수 이름 바꾸기

좋은 이름이란 주석 없이도 의도를 전달하는 이름이다. 이름 짓기가 어렵다면 아직 그 코드의 역할이 명확하지 않다는 신호다.

**파이썬 관례 준수:**

| 대상 | 케이스 | 예시 |
|------|--------|------|
| 변수, 함수 | snake_case | `get_user`, `total_amount` |
| 상수 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT`, `DELIVERY_MIN_PRICE` |
| 클래스 | PascalCase | `UserInfo`, `ArticleForm` |

**좋은 이름의 기준:**

- **의도를 드러낼 것**: `d = 10` → `elapsed_days = 10`. 변수명을 보고 주석이 필요하면 실패한 변수다.
- **발음하기 쉽고 검색 가능할 것**: `genYMDHMS` → `generate_timestamp`. 한 글자 변수는 Ctrl+F로 검색이 불가능하다.
- **문맥에 맞는 품사 사용**: 변수/객체는 명사 또는 형용사(`user`, `total_amount`), 함수/메서드는 동사(`get_user()`, `calculate_score()`), Boolean은 `is_`, `has_`, `can_` 접두사 사용(`is_valid`, `has_token`, `is_authenticated`).
- **자료구조를 변수명에 명시하지 말 것**: `user_list` → `users`, `user_dict` → `user_by_id`. 자료구조가 바뀌면 변수명이 거짓말을 하게 된다. (단, 이는 저자 권장이며 팀 컨벤션에 따라 유동적으로 적용한다.)
- **한 개념에 한 단어만 사용**: 같은 의미라면 프로젝트 전체에서 `get` 또는 `fetch` 중 하나만 사용한다. 여러 개발자가 다른 단어를 쓰면 혼란이 생긴다.

**안 좋은 예 → 좋은 예:**

```python
# 안 좋은 예
class userinfo:           # PascalCase 미준수
    pass

fetchUserList = []        # camelCase 사용
d = 10                    # 의미 불명확
genYMDHMS = "2024-01-01" # 발음·검색 불가

# 좋은 예
class UserInfo:
    pass

users = []
elapsed_days = 10
generate_timestamp = "2024-01-01"
user_by_id = {}           # 딕셔너리임을 구조 대신 의미로 표현
```

### 2-2. 매직넘버를 상수로 교체하기

**매직넘버**란 코드 속에 의미 없이 박혀 있는 숫자 또는 문자열을 뜻한다. 하드코딩과 동의어다.

매직넘버의 문제점은 두 가지다. 첫째, 볼 때마다 무슨 의미인지 파악하기 위해 로직을 분석해야 한다. 둘째, 같은 값이 여러 곳에 흩어져 있어 하나라도 수정을 빠뜨리면 버그가 터진다.

```python
# 안 좋은 예 - 30이 뭔지, 0.1이 뭔지 알 수 없음
def calculate_discount(days):
    if days > 30:
        return price * 0.1
    return 0

# 좋은 예
MAX_RENTAL_DAYS = 30
DISCOUNT_RATE = 0.1

def calculate_discount(days):
    if days > MAX_RENTAL_DAYS:
        return price * DISCOUNT_RATE
    return 0
```

### 2-3. 함수 추출 및 분리하기

**원칙:** 하나의 함수는 하나의 기능만 담당한다. 함수는 작게 작성하고, 함수 내 코드는 동일한 추상화 수준으로 구성한다.

```python
# 안 좋은 예 - 하나의 함수가 재고 확인, 결제, 이메일 발송을 모두 처리
def process_order(order_items):
    for item in order_items:
        if item.stock < 1:
            raise Exception("재고 없음")
    # 결제 처리 로직 30줄...
    # 이메일 발송 로직 20줄...

# 좋은 예 - 역할별로 분리
def process_order(order_items):
    check_stock(order_items)
    process_payment(order_items)
    send_confirmation_email(order_items)

def check_stock(order_items):
    for item in order_items:
        if item.stock < 1:
            raise Exception("재고 없음")

def process_payment(order_items):
    # 결제 처리

def send_confirmation_email(order_items):
    # 이메일 발송
```

**함수 분리의 장점:** 가독성 향상, 재사용성 증가, 독립적인 테스트 가능, 버그 추적 용이.

**함수 분리의 트레이드오프:** 지나친 분리는 추적 지옥(함수 클릭 → 또 다른 함수 → 또 다른 함수)을 만들고, 과도한 추상화로 전체 구조 파악이 어려워진다. 함수 이름이 내부 코드보다 길어지면 오버하는 것이다.

**분리 기준:**
- 재사용되는 코드라면 분리
- 한 곳에서만 쓰이고 로직도 단순하면 굳이 분리 불필요
- 너무 긴밀하게 연결된 로직은 굳이 분리 불필요

### 2-4. 조건문 분해하기

복잡한 조건식이나 깊게 중첩된 if문은 코드의 흐름을 파악하기 어렵게 만든다.

**방법 1: 조건식을 함수로 추출**

```python
# 안 좋은 예
if date < SUMMER_START or date > SUMMER_END:
    charge = BASE_CHARGE

# 좋은 예
def is_summer(date):
    return SUMMER_START <= date <= SUMMER_END

if is_summer(date):
    charge = SUMMER_CHARGE
else:
    charge = BASE_CHARGE
```

**방법 2: 얼리 리턴(Early Return)으로 depth 줄이기**

if/else가 계속 중첩되면 depth가 깊어진다. 예외 케이스를 먼저 처리하고 return해서 중첩을 줄이는 방식이다.

```python
# 안 좋은 예
def process(user):
    if user.is_active:
        if user.has_permission:
            if user.balance > 0:
                # 실제 로직
                return result
        else:
            return None
    else:
        return None

# 좋은 예 - 얼리 리턴
def process(user):
    if not user.is_active:
        return None
    if not user.has_permission:
        return None
    if user.balance <= 0:
        return None
    # 실제 로직
    return result
```

### 2-5. 매개변수 객체 도입하기

함수의 매개변수는 적을수록 좋다. 리팩토링 책에서는 **0개가 이상적이며, 3개 이상은 피하도록** 권장한다. 매개변수가 많을 때는 클래스나 객체로 묶어서 전달한다.

```python
# 안 좋은 예 - 파라미터 순서를 기억해야 함
def search_booking(date, location, guests, min_price, max_price):
    pass

# 호출부
search_booking("2024-07-01", "서울", 2, 50000, 200000)

# 좋은 예 - 객체로 묶기
class SearchBookingRequest:
    def __init__(self, date, location, guests, min_price, max_price):
        self.date = date
        self.location = location
        self.guests = guests
        self.min_price = min_price
        self.max_price = max_price

def search_booking(request: SearchBookingRequest):
    pass

# 호출부
request = SearchBookingRequest("2024-07-01", "서울", 2, 50000, 200000)
search_booking(request)
```

매개변수를 객체로 묶으면 다음 장점이 생긴다. 객체 이름만 보고도 어떤 파라미터 세트인지 알 수 있고, 내부에 파라미터를 추가해도 호출부를 전부 수정할 필요가 없으며, 유효성 검사 로직을 객체 내부에 담을 수 있다.

### 2-6. 주석달기

주석은 **코드로 의도를 표현하지 못했을 때 작성하는 것**이다. (리팩토링 책 기준. 강사님은 잘 짠 코드에도 주석이 필요할 수 있다는 입장이다.)

**반드시 주석을 코드와 싱크를 맞춰야 한다.** 코드를 수정했는데 주석을 안 바꾸면 한 달 뒤에 코드가 진짜인지 주석이 진짜인지 헷갈린다.

| 좋은 주석 | 나쁜 주석 |
|-----------|-----------|
| 법적 정보 | 코드를 그대로 번역한 주석 |
| 의도 설명 (왜 이렇게 짰는가) | 코드는 바꿨는데 내용이 다른 주석 |
| 결과에 대한 경고 | 감상문 |
| 향후 개선 필요 사항 TO-DO | 주석 처리된 사용하지 않는 코드 |

```python
# 나쁜 예 - 코드를 그대로 번역
for i in range(10):  # i를 10번 반복합니다

# 좋은 예 - 의도를 설명
# 최대 10명 중 랜덤으로 한 명을 뽑는다. 결과가 항상 일정하지 않을 수 있음.
for i in range(10):
    ...
```

---

## 3. 리팩토링을 대하는 자세

### 3-1. 조금씩 개선하기

시간이 없어서 코드가 엉망이 됐는데, 리팩토링을 하려면 시간이 필요하다는 모순이 있다. 이 모순에서 벗어나는 방법은 **조금씩 개선**하는 것이다.

하루에 변수명 하나, 하드코딩 하나만 없애도 성공적인 리팩토링이다. 오늘 코드를 짤 때 조금 더 잘 짜려고 노력하는 것 자체가 리팩토링이다.

### 3-2. 테스트 코드 작성

리팩토링의 본질은 기존 기능이 그대로 동작해야 한다는 것이다. 리팩토링 후 기존 기능이 망가질 확률이 높기 때문에 **테스트 코드**를 작성하여 안전망을 만든다. AI 시대에는 AI가 테스트 케이스를 빠르게 만들어주기 때문에 테스트 코드 작성의 부담이 많이 줄었다.

### 3-3. 실용주의적 접근

완벽함보다 개선을 목표로 한다.

- 같은 코드가 3번 이상 반복될 때 리팩토링을 고려한다. 4번부터는 골치가 아파진다.
- 개발 흐름 속에서 자연스럽게 생각나면 미루지 말고 바로 진행한다.
- 버그가 한 번 발생한 함수는 다른 버그도 품고 있을 가능성이 높다. 그 함수를 점검한다.
- 코드 리뷰를 받는 타이밍이 리팩토링의 좋은 기회다.

### 3-4. AI 리팩토링의 한계

AI는 아직 유지보수에 있어 완벽하지 않다. 비즈니스 도메인 지식이 없는 상태에서 리팩토링을 하면 의도적으로 넣어놓은 로직(예: 예약 시스템의 1초 딜레이)을 제거해버릴 수 있다. 유지보수는 도메인 지식을 가진 사람이 직접 해야 한다.

---

## 4. OpenAI Structured Output

### 4-1. 문제: LLM 응답의 형식 불일치

LLM은 확률 기반으로 답변을 생성하므로, 동일한 질문을 해도 매번 다른 형식으로 응답한다. JSON 형식으로 반환하라고 지시해도 키값, 단위, 표현 방식이 매번 달라진다.

```python
# 기존 방식 (v1 - Chat Completions)
response = client.chat.completions.create(
    model="gpt-4o",
    response_format={"type": "json_object"},  # JSON으로 달라고 요청해도
    messages=[
        {"role": "system", "content": "당신은 데이터를 JSON 형식으로 출력하는 비서입니다."},
        {"role": "user", "content": "대한민국의 수도와 인구를 JSON으로 알려줘"}
    ]
)
# 결과: 키값이 매번 다름 (capital vs 수도 vs center_city, population이 숫자/문자 등 형식 불일치)
```

### 4-2. 해결: Structured Output (v2)

OpenAI API v2(`responses` 엔드포인트)에서 도입된 기능으로, **BaseModel 클래스를 이용해 응답 형식을 고정**할 수 있다. `text_format` 파라미터에 클래스를 전달하면 해당 형식으로만 응답을 강제한다.

```python
from openai import OpenAI
from pydantic import BaseModel

client = OpenAI()

# 응답 형식을 클래스로 정의
class CountryInfo(BaseModel):
    name: str         # 국가명 (문자열 강제)
    capital: str      # 수도 (문자열 강제)
    population: int   # 인구 (정수 강제)
    cities: list[str] # 도시 목록 (문자열 리스트 강제)

# v2 방식 (Responses API)
response = client.responses.parse(
    model="gpt-4o",
    input=[{"role": "user", "content": "대한민국의 정보를 알려줘"}],
    text_format=CountryInfo,  # 응답 형식 지정
)

result = response.output_parsed
print(result.name)       # "대한민국"
print(result.capital)    # "서울"
print(result.population) # 51000000 (정수로 일관되게 반환)
```

> ⚠️ `text_format`은 GPT-4o 이상의 모델에서만 지원된다. v1 방식은 2028년쯤 지원 중단 예정이므로 v2로 마이그레이션을 권장한다.

### 4-3. description으로 응답 품질 높이기

`description`은 해당 필드에 어떤 값이 들어와야 하는지를 LLM에게 더 구체적으로 설명하는 필드다. 사실상 필드 수준의 프롬프트라고 생각하면 된다.

```python
from pydantic import BaseModel, Field

class MovieInfo(BaseModel):
    title: str = Field(description="영화 제목을 한국어로 반환하시오")
    director: str = Field(description="감독의 이름을 한국어로 반환하시오")
    release_year: int = Field(
        description="영화 개봉 연도",
        ge=1800,   # Greater than or Equal: 이상
        le=2026    # Less than or Equal: 이하
    )
    actors: list[str] = Field(description="주연 배우 목록을 한국어 이름으로 반환하시오")
```

### 4-4. Literal로 선택지 제한하기

응답값을 정해진 선택지 중에서만 고르도록 강제할 수 있다.

```python
from typing import Literal
from pydantic import BaseModel, Field

class MovieInfo(BaseModel):
    title: str
    genre: Literal["스릴러", "멜로", "코미디"]  # 세 가지 중에서만 선택
```

### 4-5. 중첩 구조 만들기

클래스 안에 다른 클래스를 중첩해서 복잡한 데이터 구조를 정의할 수 있다.

```python
from pydantic import BaseModel, Field

class ActorInfo(BaseModel):
    name: str = Field(description="배우 이름을 한국어로 반환하시오")
    age: int = Field(description="배우의 나이", ge=0, le=120)

class MovieInfo(BaseModel):
    title: str
    director: str
    actors: list[ActorInfo]  # 중첩 클래스 리스트
```

### 4-6. Optional 필드 (없으면 None 반환)

LLM이 정보가 없는 필드에 대해 값을 지어내는(환각, Hallucination) 문제를 방지할 수 있다.

```python
from typing import Optional
from pydantic import BaseModel

class MovieInfo(BaseModel):
    title: str
    production_days: Optional[int] = None  # 정보 없으면 None 반환
```

`Optional` 처리를 하면 해당 정보가 없을 때 억지로 값을 만들지 않고 `None`을 반환한다.

### 4-7. Structured Output 활용 예시

| 활용 분야 | 설명 |
|-----------|------|
| 이력서 파싱 | 다양한 형식의 PDF 이력서에서 이름, 이메일, 기술 스택, 학력 등을 일정한 형식으로 추출 → DB 직접 저장 |
| 뉴스 분석 | 뉴스 기사에서 키워드, 요약 등을 추출하여 DB에 저장 |
| 영수증 처리 | 영수증 이미지에서 금액, 날짜, 항목 등을 추출 |
| 동적 UI 생성 | AI 응답의 JSON 형식에 따라 UI 컴포넌트를 실시간으로 렌더링 |

### 4-8. Django에서 Structured Output 연동하기

`form.save(commit=False)`를 이용하면 폼 데이터로 인스턴스를 생성하되 DB에 즉시 저장하지 않는다. 이 시점에 OpenAI 응답 데이터를 인스턴스 필드에 추가한 뒤 최종적으로 `save()`를 호출한다.

```python
# views.py
def order_create(request):
    if request.method == "POST":
        form = OrderForm(request.POST)
        if form.is_valid():
            # commit=False: 인스턴스만 생성, DB 저장 보류
            order = form.save(commit=False)

            # OpenAI Structured Output으로 추가 데이터 추출
            ai_result = get_order_analysis(form.cleaned_data)

            # 인스턴스에 AI 분석 결과 추가
            order.ai_category = ai_result.category
            order.ai_summary = ai_result.summary

            # 최종 저장
            order.save()
            return redirect("order_list")
```

---

## 5. OpenAI TTS (Text-to-Speech)

OpenAI는 Structured Output 외에도 TTS(텍스트 → 음성 변환) 기능을 지원한다. 공식 문서의 Audio & Speech 섹션에서 확인할 수 있다.

```python
from openai import OpenAI

client = OpenAI()

# 텍스트를 자연스러운 톤의 음성으로 변환
response = client.audio.speech.create(
    model="gpt-4o-mini-tts",
    voice="alloy",
    input="오늘 소개할 책은 리팩토링입니다."
)

# 파일로 저장
response.stream_to_file("output.mp3")
```

공식 문서에서 제공하는 다양한 기능들(Structured Output 외에도 이미지 비전, 오디오 스피치, 함수 콜링, 웹서치, 스트리밍, 웹훅, MCP 등)을 한번씩 살펴보는 습관을 갖는 게 중요하다.

---

## 6. AbstractUser와 ModelField 심화

### 6-1. AbstractUser

`AbstractUser`는 Django 기본 User 모델을 직접 확장하기 위해 사용한다. 기본 User 모델에 없는 필드(나이, 성별, 프로필 이미지 등)를 추가해서 커스터마이징할 때 사용한다.

```python
# models.py
from django.contrib.auth.models import AbstractUser
from django.db import models

class User(AbstractUser):
    age = models.IntegerField(null=True, blank=True)
    gender = models.CharField(max_length=10, null=True, blank=True)
    profile_image = models.ImageField(upload_to="profiles/", null=True, blank=True)

# settings.py
AUTH_USER_MODEL = "accounts.User"
```

### 6-2. ModelField choices

모델 필드에 선택 가능한 값의 목록을 지정하는 기능이다. 잘못된 값이 DB에 저장되는 것을 방지(데이터 무결성)하고, 관리자 페이지나 폼에서 드롭다운 형태로 표시된다.

**TextChoices 사용:**

```python
from django.db import models

class Todo(models.Model):
    class StatusChoices(models.TextChoices):
        TODO = "TODO", "할 일"       # (DB 저장값, 화면 표시값)
        DOING = "DOING", "진행 중"
        DONE = "DONE", "완료"

    title = models.CharField(max_length=100)
    status = models.CharField(
        max_length=10,
        choices=StatusChoices.choices,
        default=StatusChoices.TODO
    )
```

**IntegerChoices 사용:**

```python
class Task(models.Model):
    class PriorityChoices(models.IntegerChoices):
        LOW = 1, "낮음"      # DB: 1, 화면: "낮음"
        MEDIUM = 2, "보통"
        HIGH = 3, "높음"

    title = models.CharField(max_length=100)
    priority = models.IntegerField(
        choices=PriorityChoices.choices,
        default=PriorityChoices.MEDIUM
    )
```

choices의 핵심: **첫 번째 값이 DB에 저장되는 값, 두 번째 값이 사용자에게 보여지는 값**이다. DB에는 간결한 코드값이 저장되고 사람이 읽을 수 있는 값은 별도로 관리한다.

---

## 7. 관통 PJT 안내

오늘 관통 프로젝트는 **어제 이어서 인증(Auth) 기능 구현**을 진행한다. 2인 팀으로 진행하며, 제출은 날짜별로 따로 메일로 보낸다.

추천 추가 구현 사항은 다음과 같다.

- OAuth (소셜 로그인): Sentry, 소켓 등 추가 도전 가능
- 브랜치 전략 적용 및 코드 리뷰
- `form.save(commit=False)` + Structured Output 연동 실습

오늘 수업 내용에서 권장하는 실습 우선순위: Structured Output 기본 실습 → 영화 정보 추출 실습 → Django 뷰에 연동.

---

## 📋 핵심 개념 정리

| 개념 | 설명 | 예시/명령어 |
|------|------|-------------|
| 기술 부채 | 빠른 개발로 쌓인 코드 품질 부채 | 나중에 3~4배 시간 소모 |
| 리팩토링 | 기존 동작 유지 + 내부 구조 개선 | 리팩토링(마틴 파울러), 클린코드 |
| 매직넘버 | 코드 속 의미 없는 숫자/문자 | `35000` → `DELIVERY_MIN_PRICE = 35000` |
| 얼리 리턴 | 예외 케이스를 먼저 처리해 depth 감소 | `if not condition: return` |
| BaseModel | pydantic 응답 형식 정의 클래스 | `class CountryInfo(BaseModel):` |
| Structured Output | OpenAI 응답 형식을 클래스로 강제 | `client.responses.parse(text_format=...)` |
| Field(description=) | 필드 수준 프롬프트 | `Field(description="한국어로 반환")` |
| Literal | 선택 가능한 값 목록 강제 | `Literal["스릴러", "멜로", "코미디"]` |
| Optional | 정보 없으면 None 반환 (환각 방지) | `Optional[int] = None` |
| form.save(commit=False) | 인스턴스 생성만 하고 DB 저장 보류 | `order = form.save(commit=False)` |
| AbstractUser | Django User 모델 직접 확장 | `class User(AbstractUser):` |
| TextChoices | 문자열 선택지 상수 정의 | `class StatusChoices(models.TextChoices):` |
| IntegerChoices | 정수 선택지 상수 정의 | `class PriorityChoices(models.IntegerChoices):` |
| TTS | 텍스트 → 음성 변환 | `client.audio.speech.create(...)` |

---

## 참고사항 (수업 후 읽기)

- OpenAI 공식 문서 Structured Output: https://platform.openai.com/docs/guides/structured-outputs
- OpenAI 공식 문서 Audio/TTS: https://platform.openai.com/docs/guides/text-to-speech
- OpenAI Responses API(v2)는 에이전트 및 툴 사용을 위한 새로운 API이며, Chat Completions API(v1)는 추후 지원 중단 예정이다.
- pydantic BaseModel은 Django 외부 환경에서도 데이터 검증 목적으로 폭넓게 사용된다.
- `form.save(commit=False)`는 트랜잭션(Transaction) 개념과 연결되며, DB에서의 commit과 동일한 개념이다.
- 이력서를 작성할 때 AI 채용 시스템이 키워드 기반으로 필터링할 가능성을 고려해 핵심 기술 키워드를 명확하게 포함시키는 것을 고려하자.
- Harness Engineering(에이전트를 잘 활용하기 위한 워크플로우 설계) 관련 내용은 추후 수업에서 다룰 예정이다.
