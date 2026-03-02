# [Python] 함수 — 매개변수, 스코프, 재귀, 패킹/언패킹

> **핵심 키워드:** #함수 #def #return #매개변수 #인자 #스코프 #LEGB #global #재귀 #map #zip #lambda #패킹 #언패킹

---

## 학습 목표

* 함수의 정의(def)와 호출 구조를 이해하고 직접 작성
* 매개변수와 인자의 차이, 다양한 인자 종류(위치/기본/키워드/가변) 구분
* 스코프(LEGB 룰)를 이해하고 변수 유효 범위 추적
* map, zip 내장함수의 동작 원리와 이터레이터 개념 파악
* 패킹/언패킹, 람다 표현식의 기본 문법 이해

---

## 1. 함수란

함수(Function)는 특정 작업을 수행하는 코드를 하나로 묶어 재사용할 수 있게 만든 것이다. 함수를 사용하는 이유는 크게 세 가지다.

**재사용성** — 같은 코드를 반복 작성하지 않고 함수 이름으로 호출하여 사용한다. **유지보수성** — 로직이 변경되면 함수 내부만 수정하면 모든 호출부에 반영된다. **가독성** — 함수 이름만 보고도 해당 코드 묶음의 기능을 유추할 수 있다.

```python
# 함수 없이 반복 작성
result1 = 3 + 5
result2 = 10 + 20
result3 = 7 + 8

# 함수로 묶으면 한 번 정의, 여러 번 사용
def get_sum(num1, num2):
    return num1 + num2

result1 = get_sum(3, 5)
result2 = get_sum(10, 20)
result3 = get_sum(7, 8)
```

> **Tip:** 함수는 코드가 익숙해진 후 도입해도 괜찮다. 기본 로직을 짜는 데 어려움이 없어지면 그때 함수화를 연습하자.

---

## 2. 함수 구조

```python
def 함수명(매개변수1, 매개변수2):   # ① 정의 (선언)
    """함수 설명 (Docstring)"""     # ② 설명 (선택)
    코드 블럭                        # ③ 바디
    return 반환값                    # ④ 반환

result = 함수명(인자1, 인자2)        # ⑤ 호출
```

**def** — 함수 정의를 시작하는 키워드. 예약어이므로 변수명으로 사용 불가.

**함수명** — 스네이크 케이스로 작성. `동사 + 명사` 형태 권장. 예: `get_user_info`, `calculate_sum`.

**매개변수(parameter)** — 정의부에서 함수가 요구하는 변수. 괄호 안에 작성.

**Docstring** — 함수 설명을 `"""..."""`로 작성. 생략 가능하지만, 다른 사람이 함수를 사용할 때 설명서 역할.

**return** — 함수의 실행을 종료하고 결과값을 호출부로 반환. return이 없으면 함수는 `None`을 반환한다.

**호출** — 함수명과 소괄호를 사용. 정의만으로는 실행되지 않는다. 정의는 설계도, 호출이 실행이다.

```python
def greet(name):
    """인사 메시지를 반환하는 함수"""
    return f"안녕하세요, {name}님!"

message = greet("Alice")
print(message)  # 안녕하세요, Alice님!
```

> **주의:** `def`와 `return`은 예약어(키워드)다. `def = 10`이나 `return = [1, 2, 3]` 같은 변수 할당은 불가능하다. 문법을 배우다 보면 자연스럽게 인지하게 된다.

---

## 3. 매개변수와 인자

| 용어 | 영어 | 위치 | 의미 |
|------|------|------|------|
| 매개변수 | parameter | 함수 **정의**부 | 함수가 요구하는 변수 |
| 인자 | argument | 함수 **호출**부 | 실제로 전달하는 값 |

```python
def add(a, b):      # a, b → 매개변수 (parameter)
    return a + b

add(3, 5)            # 3, 5 → 인자 (argument)
```

---

## 4. 인자의 종류

### 4-1. 위치 인자 (Positional Argument)

함수 호출 시 인자의 **순서**에 따라 매개변수에 대입된다.

```python
def greet(name, age):
    print(f"안녕하세요, {name}님! {age}살이시군요.")

greet("Alice", 25)   # name="Alice", age=25
greet(25, "Alice")   # name=25, age="Alice" → 순서 바뀌면 의미 달라짐
```

요구하는 매개변수 개수를 맞추지 않으면 에러가 발생한다. 간단하고 직관적이지만, 매개변수가 많아지면(3개 이상) 관리가 어려워진다.

### 4-2. 기본 인자 값 (Default Argument Value)

정의부에서 매개변수에 기본값을 할당한다. 호출 시 해당 인자를 생략하면 기본값이 사용되고, 값을 전달하면 기본값은 덮어쓰기된다.

```python
def greet(name, age=30):
    print(f"안녕하세요, {name}님! {age}살이시군요.")

greet("Bob")          # age 생략 → 기본값 30 사용
greet("Alice", 25)    # age=25 전달 → 기본값 무시, 25 사용
```

> **주의:** 기본 인자 값은 반드시 위치 인자보다 **뒤에** 와야 한다. `def greet(age=30, name)` 은 에러.

### 4-3. 키워드 인자 (Keyword Argument)

호출 시 매개변수 이름을 직접 지정하여 값을 전달한다. 순서를 외울 필요가 없어진다.

```python
def greet(name, age):
    print(f"안녕하세요, {name}님! {age}살이시군요.")

greet(name="Dave", age=35)    # 이름 지정 → 순서 무관
greet(age=35, name="Dave")    # 순서 바꿔도 정확히 매칭
```

매개변수명을 정확히 알아야 하고 코드가 길어지는 단점이 있지만, 어떤 값이 어디에 들어가는지 명확하므로 안정성이 높다.

> **주의:** 키워드 인자도 위치 인자보다 뒤에 위치해야 한다. `greet(name="Dave", 35)` 은 에러.

### 4-4. 임의의 인자 목록 (`*args`)

매개변수 앞에 애스터리스크(`*`)를 붙이면 개수 제한 없이 인자를 받을 수 있다. 전달된 값들은 **튜플** 형태로 저장된다.

```python
def calculate_sum(*args):
    print(args)        # (1, 2, 3, 4)
    return sum(args)

calculate_sum(1, 2, 3, 4)  # 몇 개든 전달 가능
```

### 4-5. 임의의 키워드 인자 목록 (`**kwargs`)

애스터리스크 두 개(`**`)를 붙이면 키=값 형태의 인자를 개수 제한 없이 받는다. **딕셔너리** 형태로 저장된다.

```python
def print_info(**kwargs):
    print(kwargs)

print_info(name="Alice", age=25)
# {'name': 'Alice', 'age': 25}
```

> **Tip:** `*args`와 `**kwargs`는 확장성이 좋지만, 어떤 타입의 데이터가 들어올지 보장할 수 없다는 단점이 있다. 실무에서 자주 쓰이는 것은 아니며, 정해진 매개변수가 충분한 경우가 대부분이다.

### 4-6. 인자 작성 순서와 실무 권장

정의 시 작성 순서: `위치 인자` → `기본 인자` → `*args` → `**kwargs`

```python
def example(a, b, c=3, *args, **kwargs):
    print(a, b, c, args, kwargs)

example(1, 2, 3, 4, 5, 6, key="value")
# 1 2 3 (4, 5, 6) {'key': 'value'}
```

실무에서는 위치 + 기본값 + 키워드 인자를 함께 사용하는 것이 가장 안전하다. 핵심 원칙은 매개변수를 **최대한 적게** 구성하는 것이다. 매개변수가 많다면 함수가 여러 기능을 담당하고 있을 확률이 높다.

---

## 5. 재귀 함수

함수 내부에서 자기 자신을 호출하는 함수다. 알고리즘 파트에서 본격적으로 다루며, 여기서는 개념 소개만 한다.

```python
def factorial(n):
    if n == 1:          # 종료 조건 (Base Case)
        return 1
    return n * factorial(n - 1)   # 자기 자신 호출

print(factorial(4))     # 4 * 3 * 2 * 1 = 24
```

핵심 원리: `n! = n × (n-1)!` 이라는 점을 이용하여 n이 1에 도달할 때까지 자기 자신을 호출한다. 1에 도달하면 return 값이 거꾸로 올라가면서 최종 결과가 계산된다.

```
factorial(4) = 4 * factorial(3)
             = 4 * 3 * factorial(2)
             = 4 * 3 * 2 * factorial(1)
             = 4 * 3 * 2 * 1
             = 24
```

재귀 함수 작성 시 반드시 **종료 조건**이 있어야 하고, 호출할 때마다 범위가 **점점 줄어드는** 형태로 설계해야 한다.

> **Tip:** 구글에서 "recursion"을 검색하면 "이것을 찾으셨나요: recursion"이라는 이스터에그가 나온다. 클릭하면 무한으로 반복되는 재귀의 개념 그 자체.

---

## 6. 내장 함수

파이썬이 기본 제공하는 함수로, import 없이 바로 사용할 수 있다.

```python
numbers = [1, 2, 3, 4, 5]

len(numbers)        # 5      — 길이(개수)
max(numbers)        # 5      — 최댓값
min(numbers)        # 1      — 최솟값
sum(numbers)        # 15     — 합계
sorted(numbers, reverse=True)  # [5, 4, 3, 2, 1] — 내림차순 정렬
```

> **주의:** 학습 단계에서는 내장 함수 사용을 자제하고 직접 구현하는 연습이 중요하다. `max`를 쓸 줄 아는 것이 아니라 `max`를 구현할 줄 아는 것이 실력이다. 시험에서도 내장 함수 사용 금지 조건이 붙는 경우가 많다.

---

## 7. map과 zip

### 7-1. map — 함수를 각 요소에 적용

`map(함수, 반복가능한자료)`는 왼쪽 함수를 오른쪽 자료의 **각 요소에 하나씩 적용**하고 결과를 반환한다.

```python
numbers = [1, 2, 3]
result = map(str, numbers)   # 각 요소에 str() 적용
print(list(result))          # ['1', '2', '3']
```

알고리즘에서 가장 많이 쓰이는 패턴은 입력값을 정수로 변환하는 것이다.

```python
# "1 2 3" 같은 입력을 정수 리스트로 변환
data = "1 2 3".split()        # ['1', '2', '3']
numbers = list(map(int, data)) # [1, 2, 3]
```

### 7-2. zip — 같은 위치의 요소끼리 묶기

여러 리스트에서 같은 인덱스의 요소들을 **세로로 묶어** 튜플로 반환한다.

```python
names = ["Jane", "Peter"]
scores = [90, 85]

result = list(zip(names, scores))
# [('Jane', 90), ('Peter', 85)]
```

3개 이상도 묶을 수 있다.

```python
kr = [10, 20, 30]
math = [20, 40, 50]
en = [40, 20, 30]

list(zip(kr, math, en))
# [(10, 20, 40), (20, 40, 20), (30, 50, 30)]
```

### 7-3. 이터레이터와 게으른 평가

`map`과 `zip`의 결과를 바로 출력하면 `<map object>`, `<zip object>` 같은 형태가 나온다. 이것이 **이터레이터(iterator)** 다.

이터레이터는 **게으른 평가(Lazy Evaluation)** 방식을 사용한다. 결과를 즉시 계산하지 않고, 실제로 사용될 때(list 변환, for문 순회 등) 그때서야 메모리에 올린다. 1억 개 요소를 변환했는데 아직 사용하지 않았다면, 메모리를 차지하지 않는다는 뜻이다.

```python
result = map(str, [1, 2, 3])
print(result)           # <map object at 0x...>  ← 아직 계산 안 됨
print(list(result))     # ['1', '2', '3']        ← 사용 시점에 계산
```

---

## 8. 스코프와 LEGB 룰

### 8-1. 로컬 스코프 vs 글로벌 스코프

**로컬 스코프(Local Scope)** 는 함수 내부의 고유한 영역이다. 여기서 선언된 변수는 함수가 종료되면 사라진다.

**글로벌 스코프(Global Scope)** 는 함수 바깥, 파일 전체에서 접근 가능한 영역이다.

핵심 규칙: 안쪽(로컬)에서는 바깥(글로벌) 변수에 **접근은 가능**하지만 **수정은 불가능**하다.

```python
num = 10               # 글로벌 변수

def my_func():
    num = 20           # 로컬 변수 (글로벌의 num과 별개)
    print(num)         # 20

my_func()
print(num)             # 10 (글로벌 변수는 변하지 않음)
```

> **Tip:** 비유하자면 로컬 스코프는 사투리, 글로벌 스코프는 표준어다. 사투리 쓰는 사람(로컬)은 표준어(글로벌)를 알아듣지만, 표준어 쓰는 사람(글로벌)은 사투리(로컬)를 못 알아듣는다.

### 8-2. LEGB 룰

파이썬이 변수를 찾는 순서: **L → E → G → B** (안쪽부터 바깥으로)

| 순서 | 스코프 | 설명 |
|------|--------|------|
| L | Local | 현재 함수 내부 |
| E | Enclosed | 감싸고 있는 바깥 함수 (중첩 함수 시) |
| G | Global | 파일 전체 |
| B | Built-in | 파이썬 내장 (len, max 등) |

```python
sum = 5                    # 글로벌에서 sum을 덮어씀
print(sum([1, 2, 3]))     # TypeError! sum은 이제 정수 5이므로 함수가 아님
```

위 예시에서 `sum`을 글로벌 변수로 할당하면, LEGB 순서에 따라 글로벌(G)의 `sum=5`가 빌트인(B)의 `sum()` 함수보다 먼저 발견되어 내장 함수가 가려진다.

### 8-3. 스코프 종합 예시

```python
a = 1
b = 2

def enclosed():
    a = 10
    c = 3

    def local(c):
        print(a, b, c)    # ① → 10, 2, 500

    local(500)
    print(a, b, c)         # ② → 10, 2, 3

enclosed()
print(a, b)                # ③ → 1, 2
```

**①** `local` 함수 안에서: a는 로컬에 없으므로 Enclosed의 10 사용, b는 글로벌의 2 사용, c는 매개변수로 전달된 500 사용.

**②** `enclosed` 함수 안에서: `local` 함수가 끝나면 그 안의 c=500은 소멸. enclosed의 c=3이 살아있다.

**③** `enclosed` 함수가 끝나면: enclosed 안의 a=10, c=3은 전부 소멸. 글로벌의 a=1, b=2만 남는다.

### 8-4. global 키워드

로컬 스코프에서 글로벌 변수를 직접 수정하려면 `global` 키워드를 선언해야 한다.

```python
num = 0

def increment():
    global num        # 글로벌 변수 num을 직접 수정하겠다고 선언
    num += 1

increment()
print(num)            # 1
```

> **주의:** `global`은 스코프 분리 원칙을 위반하는 것이므로 남발하면 안 된다. 알고리즘에서 재귀 함수 내부에서 외부 변수에 접근해야 할 때 제한적으로 사용한다.

---

## 9. 함수 스타일 가이드

**네이밍** — 스네이크 케이스 사용. `동사 + 명사` 조합. 약어보다는 명확한 이름 권장.

```python
# 좋은 예
def get_user_info():
def calculate_total():
def validate_password():

# 나쁜 예
def gui():        # 약어 → 의미 불분명
def process():    # 너무 추상적
```

**단일 책임 원칙** — 하나의 함수는 하나의 기능만 담당한다. 매개변수가 많아진다면 함수를 분리해야 한다는 신호다.

```python
# 나쁜 예: 하나의 함수가 3가지 기능 담당
def process_user_data(user):
    validate_password(user)
    save_to_db(user)
    send_email(user)

# 좋은 예: 기능별 분리 → 조합해서 사용
def validate_password(user): ...
def save_to_db(user): ...
def send_email(user): ...

def register_user(user):
    validate_password(user)
    save_to_db(user)
    send_email(user)
```

분리하면 이메일 발송이 필요 없는 경로에서는 `send_email`만 빼면 된다. 단, 과도하게 쪼개면 함수가 너무 많아져 관리가 어려워지므로 적절한 균형이 중요하다.

---

## 10. 패킹과 언패킹

### 10-1. 패킹 (Packing)

여러 값을 하나의 변수에 묶어 담는다. 결과는 튜플 형태.

```python
packed = 1, 2, 3, 4, 5
print(packed)              # (1, 2, 3, 4, 5)
```

`*`를 사용하면 나머지를 묶어서 받을 수 있다.

```python
numbers = [1, 2, 3, 4, 5]
a, *b, c = numbers
print(a)    # 1
print(b)    # [2, 3, 4]
print(c)    # 5
```

`print()` 함수가 인자 개수 제한 없이 출력 가능한 이유도, 내부적으로 `*args`로 패킹하여 받기 때문이다.

### 10-2. 언패킹 (Unpacking)

묶여 있는 데이터를 개별 변수로 분리하여 전달한다.

```python
def my_func(x, y, z):
    print(x, y, z)

names = ["Alice", "Bob", "Charlie"]
my_func(*names)            # Alice Bob Charlie  ← 리스트 언패킹

info = {"x": 1, "y": 2, "z": 3}
my_func(**info)            # 1 2 3              ← 딕셔너리 언패킹 (키=매개변수명 매칭)
```

`*`는 리스트/튜플을 풀어서 위치 인자로, `**`는 딕셔너리를 풀어서 키워드 인자로 전달한다. 인덱싱으로 하나하나 꺼내 전달할 필요가 없어진다.

---

## 11. 람다 표현식

`lambda`는 이름 없는 일회성 함수를 한 줄로 정의한다. def로 함수를 만들 정도는 아닌 간단한 연산에 사용한다.

```python
# def 방식
def square(x):
    return x ** 2

# lambda 방식 (동일 기능)
square = lambda x: x ** 2
```

map과 조합하면 별도 함수 정의 없이 깔끔하게 처리할 수 있다.

```python
numbers = [1, 2, 3, 4, 5]

# def + map
def square(x):
    return x ** 2
result = list(map(square, numbers))

# lambda + map (한 줄)
result = list(map(lambda x: x ** 2, numbers))
# [1, 4, 9, 16, 25]
```

> **Tip:** 람다는 가독성이 떨어질 수 있어 쓰지 않는 개발자도 많다. 직접 작성하지 않더라도, 다른 사람의 코드에서 람다를 읽을 수 있는 수준까지는 공부해야 한다.

---

## 정리

* 함수는 재사용 가능한 코드 묶음이며, `def`로 정의하고 함수명으로 호출한다
* 정의는 설계도일 뿐 — 호출해야 실행된다
* 매개변수(parameter)는 정의부, 인자(argument)는 호출부 용어다
* 인자 작성 순서: 위치 → 기본값 → `*args` → `**kwargs`, 실무에서는 세 가지를 함께 쓰는 것이 가장 안전
* 하나의 함수는 하나의 기능만 담당하고, 매개변수는 최대한 적게 구성한다
* 스코프는 LEGB 순서로 탐색 — 안쪽 변수가 우선이며, 스코프를 벗어나면 해당 변수는 소멸
* `global` 키워드로 로컬에서 글로벌 변수를 수정할 수 있지만 남발 금지
* `map`은 함수를 각 요소에 적용, `zip`은 같은 위치 요소를 세로로 묶는다
* 내장 함수에 의존하지 말고 직접 구현할 줄 알아야 한다 — 구현 능력이 실력
