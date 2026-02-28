# [Python] 함수 (Function) 및 스코프 (Scope)

> **핵심 키워드:** #Function #Parameter #Argument #Scope #LEGB #Recursive #Lambda #Packing #Unpacking

---

## 🎯 학습 목표
* 함수의 정의 및 호출 원리 파악을 통한 코드 재사용성 및 유지보수성 향상
* 매개변수(Parameter)와 인자(Argument)의 종류별 특성 및 데이터 전달 메커니즘 숙달
* 변수 유효 범위인 스코프(Scope)의 개념과 LEGB 규칙에 따른 참조 우선순위 이해
* 재귀 함수, 내장 함수(map, zip), 람다 표현식 등 고급 함수 활용 기법 습득

---

## 💡 주요 개념 정리

### 1. 함수의 구조 및 정의
* **정의(Define):** `def` 키워드 사용 및 `snake_case` 기반 '동사_명사' 형태 함수명 명명 권장
* **반환(Return):** 함수 실행 종료 및 결과값 호출부 전달 절차 (명시적 `return` 부재 시 `None` 자동 반환)
* **설명(Docstring):** 함수 내부 상단 `""" """` 구문 활용 기능 및 매개변수 정보 기록

### 2. 매개변수(Parameter)와 인자(Argument)
* **위치 인자 (Positional Arguments):** 함수 호출 시 전달 순서에 따른 매개변수 자동 매핑
* **기본 인자값 (Default Argument Values):** 호출 시 인자 생략 가능하도록 설정된 초기값 (위치 인자 뒤 배치 필수)
* **키워드 인자 (Keyword Arguments):** 인자 이름 직접 명시를 통한 순서 무관 데이터 전달 기법
* **가변 인자 목록 (*args):** 정해지지 않은 개수의 인자들을 튜플(Tuple) 형태로 일괄 수집
* **가변 키워드 인자 (**kwargs):** 정해지지 않은 키워드 인자들을 딕셔너리(Dict) 형태로 일괄 수집

### 3. 변수 유효 범위 (Scope) 및 수명 주기
* **로컬 스코프 (Local):** 함수 내부 독립 영역으로 해당 함수 종료 시 즉시 소멸
* **글로벌 스코프 (Global):** 모듈 전체 참조 가능 영역으로 프로그램 종료 시까지 유지
* **LEGB 규칙:** 식별자 참조 우선순위 (Local → Enclosing → Global → Built-in)
* **전역 키워드 (global):** 로컬 스코프 내 전역 변수 직접 수정을 위한 명시적 선언

---

## 💻 기능 구현 및 코드 실습

### 1. 매개변수 활용 및 기본값 설정
다양한 인자 전달 방식 조합 함수 정의 및 호출 예시

```python
def greet(name, age=30):
    # 인사말과 나이 조합 반환
    return f"안녕하세요 {name}님, {age}살이군요!"

# 위치 인자 기반 호출
print(greet("Patrick", 25))

# 기본 인자값 활용 (age 생략 시 설정값 30 적용)
print(greet("Minkyu")) 

# 키워드 인자 활용 (순서 무관 명시적 전달)
print(greet(age=28, name="Shark"))
```

### 2. 가변 인자 처리 (Packing & Unpacking)
유연한 데이터 수집용 패킹(Packing) 및 개별 인자 분리용 언패킹(Unpacking) 기법

```python
# 가변 인자 (*args): 전달 인자들을 튜플로 묶어서 처리
def calculate_sum(*args):
    return sum(args)

print(calculate_sum(1, 2, 3, 4, 5)) # 결과: 15

# 언패킹 (*): 리스트 요소를 분리하여 함수 인자로 전달
numbers = [10, 20, 30]
print(calculate_sum(*numbers)) # calculate_sum(10, 20, 30) 실행
```

### 3. 재귀 함수 (Recursive Function)
자기 자신 반복 호출을 통한 복합 문제의 계층적 해결 구조

```python
def factorial(n):
    # 종료 조건 (Base Case): 재귀 호출 중단점 설정
    if n == 1:
        return 1
    # 재귀 단계 (Recursive Step): n * (n-1)! 연산 수행
    return n * factorial(n - 1)

print(factorial(5)) # 결과: 120
```

### 4. 람다(Lambda) 및 map 활용
익명 함수를 통한 간결한 일회성 로직 처리 및 반복 가능 객체 대상 함수 적용 기법

```python
numbers = [1, 2, 3, 4, 5]

# map과 lambda 연계 활용 각 요소의 제곱 계산
# 결과 확인용 list 형변환 수행
result = list(map(lambda x: x**2, numbers))

print(result) # 결과: [1, 4, 9, 16, 25] 출력
```

---

## 🚀 복습 및 AI 인사이트
* **전역 변수 관리:** 로컬 영역 내 전역 변수 직접 수정 지양 및 인자 활용 데이터 교환 방식 권장
* **단일 책임 원칙:** 유지보수성 향상을 위한 단일 함수당 하나의 명확한 기능 할당 구조
* **지연 평가 (Lazy Evaluation):** `map`, `zip` 등의 연산을 실제 사용 시점까지 지연시키는 메모리 최적화 메커니즘
```