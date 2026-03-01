# [Python] 함수 (Functions)
> **핵심 키워드:** #Python #함수 #Function #스코프 #재귀함수 #내장함수 #람다 #패킹 #언패킹

---

## 🎯 학습 목표
* 함수의 정의, 선언, 호출 구조 이해 및 직접 구현
* 위치인자 / 기본인자 / 키워드인자 / 가변인자 차이 숙지
* 로컬 스코프 vs 글로벌 스코프 개념 및 LEGB 룰 이해
* `global` 키워드 동작 원리 파악
* 재귀함수 개념 및 팩토리얼 예시 이해
* `map()` / `zip()` 내장함수 동작 원리 이해
* 패킹 / 언패킹 (`*`, `**`) 활용법 이해
* 람다(lambda) 표현식 기본 문법 이해

---

## 💡 주요 개념 정리

### 1. 함수(Function)란?
* **특정 작업을 수행하기 위한 재사용 가능한 코드 묶음**
* 핵심은 **재사용성** — 귀찮음을 없애기 위해 탄생한 개념
* 함수를 사용하는 이유: 코드 중복 방지 / 유지보수성 향상 / 가독성 향상

> 💡 강사님 표현: *"개발자들은 귀찮아하기 때문에 누구보다 성실하게 코드를 짠다. 귀찮지 않기 위해서."*

* **주의:** 함수는 **정의(선언)** 와 **호출** 을 반드시 구분해야 함
  * 정의만 해서는 실행되지 않음 → 설계도만 그린 상태
  * 실제로 코드가 실행되는 건 **호출 시점**

### 2. 함수 구조

```python
def 함수명(매개변수1, 매개변수2):
    """
    함수 설명 (docstring) — 생략 가능하지만 협업 시 권장
    """
    # 함수 바디: 들여쓰기된 블록이 함수 범위
    결과 = 매개변수1 + 매개변수2
    return 결과   # 반환값 — 없으면 None 반환
```

* `def` : 함수 정의 키워드 (예약어 → 변수명으로 사용 불가)
* `return` : 함수 실행 **종료** + 값 **반환** 동시 수행
  * return 없으면 → `None` 반환
  * return 이후 코드는 **실행되지 않음**
* **들여쓰기 범위** = 함수 범위 (코드 블록) → 들여쓰기 벗어나면 함수 외부

### 3. 함수 네이밍 컨벤션 (스네이크 케이스)
* **소문자 + 언더바(`_`)** 로 구성 — 파이썬 표준 (클래스·예외처리 제외)
* 동사 + 명사 형태로 작성 → 기능을 직관적으로 표현

```python
# 좋은 예
def get_user_info():    # 동사(get) + 명사(user_info)
def calculate_sum():
def send_email():

# 나쁜 예
def Data():             # 대문자, 명사만 사용
def f():                # 의미 불명확
```

### 4. 매개변수(Parameter) vs 인자(Argument)
* **매개변수(Parameter):** 함수를 **정의**할 때 함수가 받을 값을 나타내는 변수
* **인자(Argument):** 함수를 **호출**할 때 실제로 전달되는 값

```python
def greet(name, age):   # name, age → 매개변수
    print(name, age)

greet("Alice", 25)      # "Alice", 25 → 인자
```

> 💡 강사님 표현: *"매개변수에 인자를 대입한다고 생각하면 정확하다."*

---

## 💻 기능 구현 및 코드 실습

### 🔧 인자 종류 4가지

#### ① 위치인자 (Positional Argument)
* 함수 호출 시 **위치(순서)** 에 따라 매개변수에 대입
* 가장 간단하고 직관적 — 개수 2~3개 이하일 때 권장

```python
def greet(name, age):
    print(f"안녕하세요, {name}님! {age}살이시군요.")

greet("Alice", 25)   # name="Alice", age=25
greet(25, "Alice")   # name=25, age="Alice" → 순서가 중요!

# 인자 개수 불일치 → 에러 발생
greet("Alice")       # TypeError: missing 1 required positional argument: 'age'
```

#### ② 기본인자 값 (Default Argument Value)
* 매개변수에 **기본값** 을 미리 설정 → 인자 미전달 시 기본값 사용
* **반드시 위치인자보다 뒤에 위치**해야 함

```python
def greet(name, age=30):   # age의 기본값 = 30
    print(f"{name}님, {age}살이시군요.")

greet("Bob")         # age 미전달 → 기본값 30 사용
greet("Bob", 35)     # age 전달 → 기본값 덮어쓰기, 35 사용
greet("Bob", None)   # None도 의도적 전달 → 기본값 무시, None 사용

# 잘못된 순서 → SyntaxError
def greet(age=30, name):   # 기본인자가 위치인자보다 앞 → 에러
    pass
```

#### ③ 키워드인자 (Keyword Argument)
* 호출 시 **매개변수명을 직접 명시** → 순서 무관
* 코드 가독성 향상, 의도 명확 전달
* 단점: 코드가 길어짐 / 매개변수명 정확히 알아야 함
* **위치인자보다 뒤에 위치**해야 함

```python
def greet(name, age):
    print(f"{name}님, {age}살")

greet(age=35, name="Dave")   # 순서 달라도 키워드로 매칭
greet("Dave", age=35)        # 위치인자 + 키워드인자 혼합 가능
# greet(name="Dave", 35)     # 키워드인자 뒤 위치인자 → 에러
```

#### ④ 가변인자 (Arbitrary Argument)
* 전달될 인자 개수가 **불확실**할 때 사용
* `*args` : 여러 인자를 **튜플** 형태로 수신
* `**kwargs` : 키워드 인자를 **딕셔너리** 형태로 수신

```python
# *args — 개수 제한 없이 튜플로 수신
def calculate_sum(*args):
    print(args)          # (1, 2, 3, 4) — 튜플 형태
    return sum(args)

calculate_sum(1, 2, 3)       # args = (1, 2, 3)
calculate_sum(1, 2, 3, 4, 5) # args = (1, 2, 3, 4, 5)

# **kwargs — 키워드 인자를 딕셔너리로 수신
def print_info(**kwargs):
    print(kwargs)        # {'name': 'Alice', 'age': 25}

print_info(name="Alice", age=25)
```

* 장점: **확장성** — 개수 무관하게 유연하게 커버
* 단점: 다양한 타입 혼입 시 로직 처리 **복잡도** 증가

#### ⑤ 현업 권장 작성 순서

```python
# 위치인자 → 기본인자 → 키워드인자 → 가변인자 → 가변키워드인자
def example(a, b, c=10, *args, **kwargs):
    pass

# 현업에서는 위치인자 + 기본인자 + 키워드인자 세 가지를 함께 활용하는 게 가장 안전
```

> 💡 강사님: *"매개변수가 너무 많으면 함수가 여러 가지 기능을 하고 있다는 신호. 함수를 분리해야 한다."*

---

### 🔧 재귀함수 (Recursive Function)
* **함수 내부에서 자기 자신을 호출**하는 함수
* 알고리즘 파트에서 심화 학습 예정 → 지금은 개념만 숙지

```python
# 팩토리얼 예시
# n! = n × (n-1)! → 자기 자신의 범위를 줄여 반복 호출
def factorial(n):
    if n == 1:          # 종료 조건 (Base Case) — 반드시 필요
        return 1
    return n * factorial(n - 1)   # 자기 자신 호출, 범위 축소

print(factorial(4))   # 4 × 3 × 2 × 1 = 24
```

* **필수 조건 2가지**
  * **종료 조건(Base Case)** 반드시 존재 → 없으면 무한 루프
  * **범위가 점점 축소**되는 방향으로 설계

> 💡 구글에 'recursion' 검색 시 *"이것을 찾으셨나요?"* 이스터에그 → 재귀적으로 반복

---

### 🔧 내장함수 (Built-in Functions)
* 파이썬 설치만으로 별도 `import` 없이 바로 사용 가능한 함수
* **학습 단계에서는 내장함수 사용 자제** → 직접 구현 먼저 연습

```python
numbers = [3, 1, 4, 1, 5]

len(numbers)              # 5 — 길이(개수)
max(numbers)              # 5 — 최댓값
min(numbers)              # 1 — 최솟값
sum(numbers)              # 14 — 합계
sorted(numbers)           # [1, 1, 3, 4, 5] — 오름차순 정렬
sorted(numbers, reverse=True)  # [5, 4, 3, 1, 1] — 내림차순 정렬
```

> ⚠️ 강사님: *"내장함수가 어떻게 동작하는지 구현할 줄 알고 나서 써야 한다. 시험에서도 내장함수 사용 금지 조건이 붙는다."*

---

### 🔧 map() 함수
* **순회 가능한 자료형의 각 요소에 함수를 하나하나 적용** 후 결과 반환
* 반환값은 **map 오브젝트(이터레이터)** → 사용 시 `list()` 형변환 필요

```python
numbers = [1, 2, 3]

# 각 요소를 문자열로 형변환
result = map(str, numbers)
print(list(result))    # ['1', '2', '3']

# 알고리즘 입력값 처리 시 자주 활용
# 예: "1 2 3" 입력 → 정수 리스트로 변환
nums = list(map(int, input().split()))   # [1, 2, 3]
```

* **이터레이터(Iterator)** 반환 이유 → **게으른 평가(Lazy Evaluation)**
  * 즉시 계산하지 않고, **실제 사용 시점에** 메모리에 올림
  * 데이터가 1억 개여도 미리 메모리 점유 X → 메모리 효율 향상
  * `list()` 형변환 or `for`문 순회 시 실제 계산 발생

---

### 🔧 zip() 함수
* **여러 이터러블을 세로(열) 방향으로 묶어** 튜플 형태로 반환
* 반환값도 zip 오브젝트(이터레이터) → `list()` 형변환 필요

```python
a_scores = [10, 20, 30]
b_scores = [40, 50, 60]

result = list(zip(a_scores, b_scores))
print(result)   # [(10, 40), (20, 50), (30, 60)]

# 활용 예시: 학생 이름 + 점수 묶기
students = ["Jane", "Ashley", "Peter"]
scores   = [90, 85, 78]

for name, score in zip(students, scores):
    print(f"{name}: {score}점")
```

---

### 🔧 스코프 (Scope) & LEGB 룰

| 스코프 범위 | 설명 | 유지 기간 |
|---|---|---|
| **Built-in** | 파이썬 실행 시부터 내장된 영역 (`len`, `print` 등) | 프로그램 전체 |
| **Global** | 파일(모듈) 최상위 레벨 | 파일 종료 시까지 |
| **Enclosed** | 중첩 함수의 외부 함수 범위 | 외부 함수 종료 시 |
| **Local** | 함수 내부의 독자적 범위 | 함수 종료 시 |

* **안쪽(Local)에서 바깥(Global)에 접근 가능** but **수정 불가**
* **바깥에서 안쪽(Local)에 접근 불가**
* 동일 변수명 존재 시 → **가장 안쪽(Local)이 우선**

```python
a = 1   # 글로벌 변수
b = 2

def outer():
    a = 10    # 로컬 변수 — 글로벌 a 덮어쓰기(로컬 내에서만)
    c = 3

    def inner(c):   # c는 매개변수로 500 수신
        print(a, b, c)   # a=10(outer 로컬), b=2(글로벌), c=500

    inner(500)
    print(a, c)     # 10, 3

outer()
print(a, b)         # 1, 2 — outer 내부 변경 영향 없음
```

> 💡 강사님 비유: *"로컬 스코프는 사투리, 글로벌 스코프는 표준어. 부산에 가면 부산 사투리가 디폴트."*

---

### 🔧 global 키워드
* 함수 내부(로컬)에서 **전역 변수(글로벌)를 직접 수정**할 때 사용
* 남발 금지 — 스코프 구분을 의도적으로 무너뜨리는 행위
* 실무보다는 **알고리즘 문제**에서 제한적으로 활용

```python
count = 0

def increment():
    global count    # 글로벌 변수 count를 직접 수정 선언
    count += 1      # global 없이는 UnboundLocalError 발생

increment()
print(count)   # 1
```

> ⚠️ global 키워드 규칙
> * **매개변수에 사용 불가**
> * **글로벌 키워드 선언 전 해당 변수 참조 불가**

---

### 🔧 패킹(Packing) & 언패킹(Unpacking)

#### 패킹 — 여러 값을 하나의 변수(튜플)에 묶기

```python
# 기본 패킹
numbers = 1, 2, 3, 4, 5   # 자동으로 튜플로 패킹

# 에스터리스크(*) 활용 패킹 — 나머지를 리스트로 수집
a, *b, c = 1, 2, 3, 4, 5
# a=1, b=[2, 3, 4], c=5

first, *rest = [1, 2, 3, 4, 5]
# first=1, rest=[2, 3, 4, 5]
```

#### 언패킹 — 묶인 값을 개별 변수로 분리

```python
def my_function(x, y, z):
    print(x, y, z)

# * 언패킹 — 리스트/튜플을 개별 인자로 분리
names = [1, 2, 3]
my_function(*names)   # my_function(1, 2, 3) 과 동일

# ** 언패킹 — 딕셔너리를 키워드 인자로 분리
data = {'x': 1, 'y': 2, 'z': 3}
my_function(**data)   # my_function(x=1, y=2, z=3) 과 동일

# print() 함수도 내부적으로 *args로 가변인자 수신
print("안녕", "하세요", "!!")   # 인자 개수 제한 없음
```

---

### 🔧 람다(Lambda) 표현식
* **익명함수(Anonymous Function)** — 이름 없이 한 줄로 정의하는 일회성 함수
* `return` 키워드 생략 → 표현식 결과 자동 반환

```python
# 일반 함수
def addition(x, y):
    return x + y

# 람다 표현식으로 동일 기능 표현
addition_lambda = lambda x, y: x + y
print(addition_lambda(3, 5))   # 8

# map()과 람다 조합 — 가장 많이 쓰이는 패턴
numbers = [1, 2, 3, 4, 5]

# 일반 함수 사용
def square(x):
    return x ** 2
result = list(map(square, numbers))

# 람다 사용 — 코드 간결화
result = list(map(lambda x: x ** 2, numbers))
print(result)   # [1, 4, 9, 16, 25]
```

> 💡 강사님 의견: *"람다는 가독성이 떨어지고 재사용성도 낮아서 개인적으로 잘 안 쓴다. 오픈소스 코드를 읽을 수 있을 정도만 익혀두면 충분하다."*

---

### 🔧 단일 책임 원칙 (Single Responsibility Principle)

```python
# ❌ 나쁜 예 — 하나의 함수가 여러 기능 담당
def process_user_data(user):
    # 비밀번호 검사 로직
    # 유저 데이터 저장 로직
    # 이메일 발송 로직
    pass

# ✅ 좋은 예 — 기능별로 함수 분리
def validate_password(password):
    """비밀번호 유효성 검사"""
    pass

def save_user(user):
    """유저 데이터 저장"""
    pass

def send_welcome_email(email):
    """회원가입 이메일 발송"""
    pass

def register_user(user):
    """위 함수들을 순서대로 실행"""
    validate_password(user['password'])
    save_user(user)
    send_welcome_email(user['email'])
```

> 💡 강사님: *"잘게 쪼갤수록 유지보수와 개발 속도가 올라간다. 단, 너무 쪼개면 함수가 많아져 혼란스럽다. 트레이드오프를 적절히 결정하는 게 중요."*

---

## 🚀 복습 및 AI 인사이트

### ✅ 핵심 체크포인트
* **정의 ≠ 실행:** `def`로 정의한 코드는 **호출**해야 비로소 실행
* **return 없으면 None 반환:** 할당 시 `None`이 들어감 → 예기치 않은 버그 원인
* **return은 함수 종료:** return 이후 코드는 절대 실행되지 않음
* **인자 작성 순서:** 위치인자 → 기본인자 → 키워드인자 → `*args` → `**kwargs`
* **기본인자는 위치인자 뒤:** 반드시 위치인자 이후에 작성
* **스코프 우선순위 LEGB:** Local → Enclosed → Global → Built-in 순으로 탐색
* **global 남발 금지:** 스코프 분리 원칙 훼손 → 알고리즘에서만 제한적 사용
* **map 반환값은 이터레이터:** `list()` 형변환 필요 / 게으른 평가(Lazy Evaluation)
* **재귀함수 2대 조건:** 종료 조건(Base Case) 필수 + 범위가 점점 축소되는 구조
* **스네이크 케이스:** 파이썬 함수/변수명은 소문자 + 언더바 (클래스 제외)
* **단일 책임 원칙:** 하나의 함수 = 하나의 기능 → 유지보수성·확장성 향상

### 🤖 AI 활용 팁
* **프롬프트 예시 (스코프 이해):** `"Python LEGB 룰에서 동일한 변수명이 글로벌과 로컬 스코프에 동시에 존재할 때 어떤 값이 출력되는지 단계별로 추적해줘 (코드 포함)"`
* **프롬프트 예시 (재귀함수):** `"Python에서 팩토리얼을 재귀함수로 구현할 때 각 호출 단계에서 스택이 어떻게 쌓이고 반환되는지 시각적으로 설명해줘"`
* **프롬프트 예시 (람다 vs 일반 함수):** `"Python에서 lambda와 def 함수의 차이점을 가독성, 재사용성, 성능 관점에서 비교해줘"`
* **프롬프트 예시 (map 활용):** `"Python map() 함수의 게으른 평가(Lazy Evaluation) 원리를 메모리 측면에서 설명하고, 언제 실제 계산이 발생하는지 예시로 보여줘"`