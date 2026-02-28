# [Python] 객체지향 프로그래밍 및 예외 처리 (OOP & Exception Handling)

> **핵심 키워드:** #OOP #Class #Instance #Inheritance #MRO #super #Overriding #Exception #Try_Except

---

## 🎯 학습 목표
* 절차 지향과 객체지향 프로그래밍의 패러다임 차이 및 장단점 이해
* 클래스(Class) 설계도와 인스턴스(Instance) 객체 간의 관계 파악
* 상속(Inheritance) 및 다중 상속 구조에서의 메서드 탐색 순서(MRO) 및 `super()` 활용 숙달
* 프로그램 실행 중 발생하는 에러 유형 파악 및 예외 처리(Exception Handling) 기법 습득

---

## 💡 주요 개념 정리

### 1. 객체지향 프로그래밍 (OOP) 개요
* **정의:** 데이터(속성)와 기능(메서드)을 객체라는 하나의 단위로 묶어 관리하는 방식
* **핵심 이점:** 코드의 재사용성 향상, 유지보수 용이성 및 현실 세계 모델링의 적합성
* **대비 개념:** 순차적인 로직 흐름 중심의 절차 지향 프로그래밍(C 언어 등)

### 2. 클래스(Class)와 인스턴스(Instance)
* **클래스:** 객체를 만들기 위한 설계도 혹은 붕어빵 틀
* **인스턴스:** 클래스라는 틀을 통해 실제 생성된 고유한 개체(붕어빵)
* **네임스페이스:** 각 인스턴스는 독립적인 이름 공간을 가지며, 인스턴스 내 부재 시 클래스 공간 탐색

### 3. 상속 (Inheritance)
* **메서드 오버라이딩:** 부모 클래스의 메서드를 자식 클래스에서 동일한 이름으로 재정의
* **다중 상속:** 둘 이상의 상위 클래스로부터 속성과 기능을 물려받는 기법
* **MRO (Method Resolution Order):** 다중 상속 시 메서드를 찾는 우선순위 규칙 (깊이 우선, 왼쪽에서 오른쪽)

---

## 💻 기능 구현 및 코드 실습

### 1. 클래스 구성 및 메서드 유형
인스턴스, 클래스, 정적 메서드의 명확한 구분 및 활용 기법

```python
class BankAccount:
    interest_rate = 0.03  # 클래스 변수 (모든 인스턴스 공유)

    def __init__(self, owner, balance):
        self.owner = owner    # 인스턴스 변수 (개별 독립 데이터)
        self.balance = balance

    # 1. 인스턴스 메서드: 인스턴스 상태 조작 (첫 인자 self)
    def deposit(self, amount):
        self.balance += amount
        return self.balance

    # 2. 클래스 메서드: 클래스 변수 조작 (첫 인자 cls)
    @classmethod
    def set_interest_rate(cls, rate):
        cls.interest_rate = rate

    # 3. 스태틱 메서드: 클래스/인스턴스와 독립적인 로직 (인자 없음)
    @staticmethod
    def is_positive(amount):
        return amount > 0
```

### 2. 상속 및 `super()` 활용
부모 클래스의 기능을 확장하고 중복 코드를 제거하는 설계 기법

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

class Student(Person):
    def __init__(self, name, age, gpa):
        # 부모 클래스의 생성자 호출을 통한 초기화 위임
        super().__init__(name, age) 
        self.gpa = gpa

    def talk(self): # 메서드 오버라이딩
        print(f"안녕하세요, {self.name}입니다. 학점은 {self.gpa}입니다.")
```

### 3. 예외 처리 구문 (Try-Except)
예상치 못한 에러 상황에서도 프로그램 종료를 방지하는 방어적 코딩

```python
try:
    num = int(input("숫자 입력: "))
    result = 10 / num
except ValueError:        # 타입 에러 대응
    print("정수만 입력 가능")
except ZeroDivisionError: # 0 나누기 에러 대응
    print("0으로 나눌 수 없음")
except Exception as e:    # 기타 모든 에러 대응
    print(f"예상치 못한 에러: {e}")
else:                     # 에러 없을 때 실행
    print(f"결과: {result}")
finally:                  # 발생 여부 무관 무조건 실행
    print("연산 종료")
```

---

## 🚀 복습 및 AI 인사이트
* **MRO의 중요성:** 다중 상속 시 `super()`는 단순히 부모를 부르는 것이 아닌, MRO 순서상의 다음 클래스를 호출함에 유의
* **에러 메시지 독해:** 에러 발생 시 즉시 AI에 의존하기보다 `SyntaxError`, `IndexError` 등 메시지 하단을 직접 읽고 원인을 파악하는 습관 강조
* **설계 철학:** 상속은 코드 재사용을 위한 강력한 도구이나, 과도한 깊이의 상속은 복잡성을 증대시키므로 적절한 계층 구조 설계 필요성
* **매직 메서드:** `__str__`, `__len__` 등 특수 메서드 활용을 통한 사용자 정의 객체의 파이썬 표준 동작 구현 기법