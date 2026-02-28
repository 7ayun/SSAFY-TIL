# [Python] 함수 (Function) 및 스코프 (Scope)

> **핵심 키워드:** #Function #Parameter #Argument #Scope #LEGB #Recursive #Lambda #Packing #Unpacking

---

## 🎯 학습 목표
* 함수의 정의 및 호출 원리와 재사용을 통한 유지보수성 향상 이해
* 매개변수(Parameter)와 인자(Argument)의 다양한 종류 및 전달 방식 마스터
* 변수 유효 범위인 스코프(Scope)의 개념과 LEGB 룰을 통한 참조 우선순위 파악
* 재귀 함수, 내장 함수(map, zip), 람다 표현식 등 고급 함수 기법 습득

---

## 💡 주요 개념 정리

### 1. 함수의 구조 및 정의
* **정의(Define):** `def` 키워드 사용, 함수명은 `snake_case`와 `동사_명사` 형태 권장
* **반환(Return):** 함수 실행 종료 및 결과를 호출부로 전달하며, 명시적 `return` 없을 시 `None` 반환
* **독스트링(Docstring):** 함수 내부 상단에 `""" """`를 이용한 기능 설명 기록

### 2. 매개변수(Parameter)와 인자(Argument)
* **위치 인자 (Positional Arguments):** 함수 호출 시 인자의 순서에 맞춰 매개변수에 할당
* **기본 인자값 (Default Argument Values):** 매개변수에 기본값을 설정하여 호출 시 생략 가능 (위치 인자 뒤에 위치 필수)
* **키워드 인자 (Keyword Arguments):** 인자의 이름과 값을 명시하여 순서와 상관없이 전달
* **가변 인자 목록 (*args):** 정해지지 않은 개수의 인자를 튜플(Tuple) 형태로 처리
* **가변 키워드 인자 (**kwargs):** 정해지지 않은 키워드 인자들을 딕셔너리(Dict) 형태로 처리

### 3. 변수 유효 범위 (Scope) 및 수명 주기
* **로컬 스코프 (Local):** 함수 내부 영역으로 함수 종료 시 소멸
* **글로벌 스코프 (Global):** 모듈 전체에서 참조 가능하며 프로그램 종료 시까지 유지
* **LEGB 룰:** 이름 참조 순서 (Local → Enclosing → Global → Built-in)
* **global 키워드:** 로컬 스코프에서 전역 변수를 수정하기 위해 사용 (남발 지양)

---

## 💻 기능 구현 및 코드 실습

### 1. 매개변수 활용 및 기본값 설정
다양한 인자 전달 방식을 조합한 함수 구현 예시임.

```python
def greet(name, age=30):
    # 인사말과 나이를 반환하는 함수
    return f"안녕하세요 {name}님, {age}살이군요!"

# 위치 인자 사용
print(greet("Patrick", 25))

# 기본 인자 사용 (age 생략)
print(greet("Minkyu")) # age는 30으로 처리됨

# 키워드 인자 사용 (순서 무관)
print(greet(age=28, name="Shark"))
```

### 2. 가변 인자 처리 (Packing & Unpacking)
개수가 정해지지 않은 데이터를 유연하게 처리하는 기법임.

```python
# 가변 인자 (*args): 여러 인자를 튜플로 패킹
def calculate_sum(*args):
    # 전달받은 모든 숫자의 합 계산
    return sum(args)

print(calculate_sum(1, 2, 3, 4, 5)) # 15

# 언패킹 (*): 리스트 요소를 개별 인자로 풀어서 전달
numbers = [10, 20, 30]
print(calculate_sum(*numbers)) # calculate_sum(10, 20, 30)과 동일
```

### 3. 재귀 함수 (Recursive Function)
자기 자신을 호출하여 복잡한 로직을 간결하게 구현함.

```python
def factorial(n):
    # 종료 조건 (Base Case)
    if n == 1:
        return 1
    # 재귀 단계 (Recursive Step): n * (n-1)!
    return n * factorial(n - 1)

print(factorial(5)) # 120
```

### 4. 람다(Lambda) 및 map 활용
익명 함수를 사용하여 한 줄로 로직을 처리함.

```python
numbers = [1, 2, 3, 4, 5]

# map과 lambda를 활용하여 각 요소의 제곱 계산
# map(함수, 반복가능데이터) -> 각 요소에 함수 적용
result = list(map(lambda x: x**2, numbers))

print(result) # [1, 4, 9, 16, 25]
```

---

## 🚀 복습 및 AI 인사이트
* **주의 사항:** 함수 내에서 전역 변수를 수정할 때는 `global` 선언이 필요하지만, 가급적 함수의 인자로 값을 받아 처리하는 방식 권장
* **단일 책임 원칙:** 하나의 함수는 하나의 명확한 기능만 담당하도록 설계하여 유지보수성 극대화
* **게으른 평가 (Lazy Evaluation):** `map`, `zip` 등의 결과가 `object`로 반환되는 이유는 메모리 효율을 위해 실제 사용 시점까지 연산을 늦추기 때문임