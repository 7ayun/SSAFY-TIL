# [Python] OOP — 프로그래밍 패러다임, 클래스 기초, 메서드, 상속

> **핵심 키워드:** #OOP #절차지향 #객체지향 #클래스 #인스턴스 #생성자 #self #인스턴스변수 #클래스변수 #인스턴스메서드 #클래스메서드 #스태틱메서드 #매직메서드 #상속 #오버라이딩 #다중상속 #MRO #super

---

## 학습 목표

* 절차지향과 객체지향 프로그래밍의 차이와 관계를 이해
* 클래스, 인스턴스, 생성자 메서드(`__init__`), `self`의 역할 파악
* 인스턴스 변수와 클래스 변수의 차이 및 탐색 순서 이해
* 인스턴스 메서드 · 클래스 메서드 · 스태틱 메서드의 구분과 활용
* 상속, 메서드 오버라이딩, 다중 상속(MRO), `super()`의 동작 원리 이해

---

## 1. 프로그래밍 패러다임

### 1-1. 절차지향 프로그래밍 (Procedural Programming)

위에서 아래로 코드가 순차적으로 실행되는 방식이다. 변수와 함수를 별개로 다루며, 대표적으로 C언어가 있다.

```python
# 절차지향 방식
name = "Alice"
age = 25

def introduce(name, age):
    print(f"저는 {name}이고, {age}살입니다.")

introduce(name, age)
```

장점은 코드가 직관적이고, 실행 속도가 빠르다는 것이다. 단점은 코드가 길어질수록 복잡성이 증가하고, 변수·함수 관리가 어려워진다는 것이다.

### 1-2. 객체지향 프로그래밍 (Object-Oriented Programming)

같은 쓰임새의 **데이터와 함수를 하나의 단위(객체)로 묶어서** 관리하는 방식이다. 대표적으로 Java, C++이 있으며, 파이썬도 객체지향을 메인으로 지원한다.

```python
# 객체지향 방식
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def introduce(self):
        print(f"저는 {self.name}이고, {self.age}살입니다.")

p1 = Person("Alice", 25)
p1.introduce()
```

> **강사님 강조**: 절차지향과 객체지향은 **대조되는 개념이 아니다**. 절차지향으로 개발할 수 없는 사람이 객체지향으로 바로 넘어갈 수는 없다. 알고리즘 문제는 절차지향으로 풀되, 프로젝트에서는 객체지향을 적극 활용하면 된다.

### 1-3. 두 패러다임 비교

| 구분 | 절차지향 | 객체지향 |
|------|----------|----------|
| 중심 | 함수 호출의 흐름 (논리적 순서) | 객체 간 상호작용 |
| 데이터 관리 | 변수·함수 별개 관리 | 데이터+기능을 객체로 묶음 |
| 장점 | 직관적, 속도 빠름 | 재사용성, 유지보수 용이 |
| 대표 언어 | C | Java, C++ |

---

## 2. 클래스와 인스턴스

### 2-1. 핵심 개념

**클래스(Class)** 는 객체를 만들기 위한 설계도(틀)이다. 클래스를 정의하는 것만으로는 아무 일도 일어나지 않는다.

**인스턴스(Instance)** 는 클래스를 기반으로 실제로 만들어진 개별 객체이다.

```
붕어빵 틀 → 클래스
붕어빵    → 인스턴스
```

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def introduce(self):
        print(f"안녕하세요, 저는 {self.name}이고 {self.age}살입니다.")

p1 = Person("Alice", 25)   # 인스턴스 생성
p2 = Person("Bella", 30)   # 또 다른 인스턴스

p1.introduce()  # 안녕하세요, 저는 Alice이고 25살입니다.
p2.introduce()  # 안녕하세요, 저는 Bella이고 30살입니다.
```

동일한 메서드를 호출하지만, 인스턴스가 갖고 있는 데이터에 따라 결과가 달라진다.

### 2-2. 객체 vs 인스턴스

**객체(Object)** 는 전체를 뭉뚱그려서 말하는 표현이다. **인스턴스**는 "어떤 클래스의 인스턴스"인지 주체를 명확히 해야 한다.

```python
# "아이유는 가수 클래스의 인스턴스다" (O)
# "아이유는 인스턴스다" (△ — 어떤 클래스인지 불명확)
```

### 2-3. 파이썬의 모든 데이터는 클래스의 인스턴스

```python
name = "Alice"       # str 클래스의 인스턴스
numbers = [1, 2, 3]  # list 클래스의 인스턴스
age = 25             # int 클래스의 인스턴스
```

우리가 사용해 온 `.upper()`, `.append()` 등의 메서드는 이미 정의된 클래스 내부의 인스턴스 메서드를 호출한 것이다.

---

## 3. 클래스 구성 요소

### 3-1. 클래스 정의 문법

```python
class MyClass:    # class 키워드 + 파스칼케이스(PascalCase) 이름
    pass
```

클래스 이름은 **파스칼케이스**(각 단어 첫 글자 대문자)로 작성하는 것이 관례다.

### 3-2. 생성자 메서드 (`__init__`)

인스턴스가 생성될 때 **자동으로 호출**되는 메서드이다. 인스턴스 변수의 초기값을 설정하는 역할을 한다.

```python
class Circle:
    def __init__(self, radius):
        self.radius = radius    # 인스턴스 변수 초기화

c1 = Circle(1)   # __init__ 자동 실행 → c1.radius = 1
c2 = Circle(2)   # __init__ 자동 실행 → c2.radius = 2
```

### 3-3. 인스턴스 변수 vs 클래스 변수

```python
class Circle:
    pi = 3.14               # 클래스 변수: 모든 인스턴스가 공유

    def __init__(self, radius):
        self.radius = radius # 인스턴스 변수: 인스턴스별 독립

c1 = Circle(1)
c2 = Circle(2)

print(c1.radius)       # 1 (인스턴스 변수)
print(c2.radius)       # 2 (인스턴스 변수)
print(Circle.pi)       # 3.14 (클래스 변수)
```

| 구분 | 인스턴스 변수 | 클래스 변수 |
|------|--------------|------------|
| 정의 위치 | `__init__` 내부 (`self.변수`) | 클래스 바로 아래 |
| 범위 | 인스턴스별 독립 | 전체 인스턴스 공유 |
| 접근 방법 | `인스턴스.변수` | `클래스.변수` (권장) |

인스턴스에서도 클래스 변수에 접근할 수 있지만, 가독성을 위해 `클래스명.변수`로 접근하는 것이 권장된다. 탐색 순서는 **인스턴스 → 클래스** 순이다 (로컬 → 글로벌 스코프와 동일한 원리).

---

## 4. 메서드 종류

### 4-1. 인스턴스 메서드

가장 많이 쓰이는 메서드로, 인스턴스가 호출하며 첫 번째 매개변수로 `self`(호출한 인스턴스)를 자동으로 받는다.

```python
class Counter:
    def __init__(self):
        self.count = 0          # 인스턴스 변수 초기화

    def increment(self):        # 인스턴스 메서드
        self.count += 1

c = Counter()
c.increment()
print(c.count)   # 1
c.increment()
print(c.count)   # 2
```

`self`는 **메서드를 호출한 인스턴스 자체를 인지하기 위한** 매개변수이다. `self`라는 이름은 관례일 뿐이지만, **필수라고 생각하고** 반드시 `self`로 작성한다.

**동작 원리**: `c.increment()`를 호출하면 내부적으로 `Counter.increment(c)`와 동일하게 동작한다.

```python
# 단축형 호출 (우리가 쓰는 방식)
"hello".upper()

# 실제 내부 동작
str.upper("hello")
```

### 4-2. 클래스 메서드

클래스가 호출하며, 첫 번째 매개변수로 `cls`(클래스 자체)를 받는다. **클래스 변수를 조작**할 때 사용한다.

```python
class Person:
    population = 0              # 클래스 변수

    def __init__(self, name):
        self.name = name
        Person.increase_population()

    @classmethod                # 데코레이터 필수
    def increase_population(cls):
        cls.population += 1

p1 = Person("Alice")
p2 = Person("Bella")
print(Person.population)   # 2
```

### 4-3. 스태틱 메서드

클래스·인스턴스와 **무관하게 독립적으로 동작**하는 메서드이다. `self`도 `cls`도 받지 않으며, 사실상 일반 함수와 동일하다.

```python
class MathUtils:
    @staticmethod               # 데코레이터 필수
    def add(a, b):
        return a + b

    @staticmethod
    def is_positive(num):
        return num > 0

print(MathUtils.add(3, 5))          # 8
print(MathUtils.is_positive(-1))    # False
```

### 4-4. 세 메서드 비교

| 구분 | 인스턴스 메서드 | 클래스 메서드 | 스태틱 메서드 |
|------|----------------|-------------|-------------|
| 첫 번째 인자 | `self` (인스턴스) | `cls` (클래스) | 없음 |
| 데코레이터 | 없음 | `@classmethod` | `@staticmethod` |
| 호출 주체 | 인스턴스 | 클래스 | 클래스 |
| 용도 | 인스턴스 데이터 조작 | 클래스 변수 조작 | 독립적 유틸 기능 |

> **강사님 강조**: 클래스는 클래스 메서드·스태틱 메서드만, 인스턴스는 인스턴스 메서드만 사용하자. **"할 수 있다 ≠ 써도 된다"** — 객체지향에서는 행동의 주체가 명확해야 한다.

### 4-5. 종합 예제: 은행 계좌

```python
class BankAccount:
    interest_rate = 0.05           # 클래스 변수 (이자율은 은행 전체 공통)

    def __init__(self, owner, balance):
        self.owner = owner
        self.balance = balance

    def deposit(self, amount):     # 인스턴스 메서드 (계좌별 독립)
        self.balance += amount

    def withdraw(self, amount):    # 인스턴스 메서드
        if self.balance >= amount:
            self.balance -= amount
        else:
            print("잔액 부족")

    @classmethod
    def set_interest_rate(cls, rate):   # 클래스 메서드 (은행 레벨)
        cls.interest_rate = rate

    @staticmethod
    def is_positive(amount):            # 스태틱 메서드 (독립 유틸)
        return amount > 0
```

---

## 5. 매직 메서드와 데코레이터

### 5-1. 매직 메서드 (Special Methods)

`__이름__` 형태의 메서드로, 특수한 동작을 위해 파이썬이 미리 정의해 둔 것이다. 인스턴스 메서드에 해당한다.

```python
class Circle:
    def __init__(self, radius):
        self.radius = radius

    def __str__(self):                 # print() 시 자동 호출
        return f"원의 반지름: {self.radius}"

    def __len__(self):                 # len() 시 자동 호출
        return self.radius

c = Circle(5)
print(c)        # 원의 반지름: 5    (__str__ 호출)
print(len(c))   # 5                 (__len__ 호출)
```

우리가 `len([1,2,3])`을 쓸 수 있는 이유도 `list` 클래스 내부에 `__len__` 매직 메서드가 정의되어 있기 때문이다.

### 5-2. 데코레이터 (Decorator)

다른 함수의 코드를 유지한 채 기능을 수정·확장하는 함수이다. 함수 위에 `@데코레이터명`을 작성하면 해당 함수 실행 전에 데코레이터가 먼저 실행된다.

```python
@classmethod      # "이 메서드는 클래스 메서드입니다"
@staticmethod     # "이 메서드는 스태틱 메서드입니다"
```

직접 만드는 것은 고급 스킬이므로, 현재는 **기존 데코레이터를 끼워넣어 사용하는 것**으로 충분하다.

---

## 6. 상속 (Inheritance)

### 6-1. 개념

한 클래스(부모)의 속성과 메서드를 다른 클래스(자식)가 그대로 물려받는 것이다. 코드 재사용과 계층 구조 형성이 목적이다.

```python
class Animal:                    # 부모 클래스
    def eat(self):
        print("먹는 중")

class Dog(Animal):               # 자식 클래스 (Animal 상속)
    def bark(self):
        print("멍멍!")

my_dog = Dog()
my_dog.bark()   # 멍멍!
my_dog.eat()    # 먹는 중  ← Dog에 eat이 없지만 Animal에서 상속받아 사용
```

### 6-2. 상속 없이 vs 상속으로 구현

```python
# 상속 없이 — 중복 코드 발생
class Professor:
    def __init__(self, name, age, department):
        self.name = name
        self.age = age
        self.department = department
    def talk(self):
        print(f"{self.name}, {self.age}살")

class Student:
    def __init__(self, name, age, gpa):
        self.name = name       # 중복
        self.age = age         # 중복
        self.gpa = gpa
    def talk(self):            # 중복
        print(f"{self.name}, {self.age}살")
```

```python
# 상속 활용 — 공통 부분을 부모 클래스로
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    def talk(self):
        print(f"{self.name}, {self.age}살")

class Professor(Person):
    def __init__(self, name, age, department):
        super().__init__(name, age)
        self.department = department

class Student(Person):
    def __init__(self, name, age, gpa):
        super().__init__(name, age)
        self.gpa = gpa
```

> **강사님 강조**: 상속의 첫 번째 목표는 **코드를 해석할 수 있는 것**이다. 직접 상속 구조를 설계하는 건 경험이 쌓여야 가능하다. 장고(Django)에서 상속을 많이 활용하므로 개념 이해가 중요하다.

---

## 7. 메서드 오버라이딩과 오버로딩

### 7-1. 메서드 오버라이딩 (Method Overriding)

자식 클래스가 부모 클래스의 메서드를 **동일한 이름으로 재정의**하는 것이다. 자식 클래스의 메서드가 우선 적용된다 (함수 스코프에서 로컬 변수가 글로벌보다 우선인 것과 같은 원리).

```python
class Animal:
    def eat(self):
        print("먹는 중")

class Dog(Animal):
    def eat(self):              # 부모의 eat을 덮어쓰기(오버라이딩)
        print("멍멍! 먹는 중")

my_dog = Dog()
my_dog.eat()   # 멍멍! 먹는 중  ← Dog의 eat이 우선
```

### 7-2. 메서드 오버로딩 (Method Overloading)

함수 이름이 같아도 **매개변수가 다르면 다른 함수**로 동작하는 것이다. Java, C++에서 지원하며, **파이썬은 지원하지 않는다**.

```python
# Java 스타일 (파이썬에서는 불가)
def add(a, b): ...
def add(a, b, c): ...    # 파이썬에서는 마지막 정의만 유효

# 파이썬에서 유사하게 구현
def add(*nums):
    if len(nums) == 2:
        return nums[0] + nums[1]
    elif len(nums) == 3:
        return nums[0] + nums[1] + nums[2]
```

> **면접 빈출**: 오버라이딩 vs 오버로딩의 차이. 오버라이딩은 상속 관계에서 메서드 재정의, 오버로딩은 매개변수에 따라 다른 함수로 동작하는 것이다.

---

## 8. 다중 상속과 MRO

### 8-1. 다중 상속

둘 이상의 부모 클래스로부터 상속받는 것이다. 상속받은 모든 속성·메서드를 사용할 수 있지만, **이름이 겹칠 경우** 어떤 부모를 따를지 문제가 생긴다 (다이아몬드 문제).

```python
class Mom:
    gene = "XX"
    def swim(self):
        print("엄마 수영")

class Dad:
    gene = "XY"
    def walk(self):
        print("아빠 걷기")

class Child(Dad, Mom):          # Dad, Mom 순서로 상속
    def cry(self):
        print("응애")

baby = Child()
baby.cry()              # 응애      ← 자기 자신
baby.walk()             # 아빠 걷기  ← Dad에서 상속
print(baby.gene)        # XY       ← Dad가 먼저 (상속 순서)
```

### 8-2. MRO (Method Resolution Order)

파이썬이 다중 상속에서 메서드를 찾는 규칙이다.

**규칙**: 깊이 우선, 왼쪽에서 오른쪽으로, 같은 클래스는 두 번 검색하지 않는다.

```python
class Child(Dad, Mom):   # 탐색 순서: Child → Dad → Mom → object
    pass

print(Child.mro())
# [Child, Dad, Mom, object]
```

속성이나 메서드를 호출하면 MRO 순서대로 탐색하여 **먼저 찾은 것을 사용**한다.

---

## 9. super()

### 9-1. 개념

자식 클래스에서 **부모 클래스를 호출**하기 위한 내장 함수이다. MRO 순서에 따라 올바른 부모 클래스를 자동으로 찾아준다.

### 9-2. super()를 사용하는 이유

부모 클래스 이름을 직접 쓰면(하드코딩), 이름이 바뀌거나 상속 구조가 변경될 때 모든 코드를 수정해야 한다. `super()`를 쓰면 파이썬이 MRO에 따라 알아서 부모를 호출하므로 **유지보수가 용이**하다.

```python
# 하드코딩 (비권장)
class Student(Person):
    def __init__(self, name, age, gpa):
        Person.__init__(self, name, age)    # 부모 이름 직접 명시
        self.gpa = gpa

# super() 활용 (권장)
class Student(Person):
    def __init__(self, name, age, gpa):
        super().__init__(name, age)         # MRO에 따라 자동 호출
        self.gpa = gpa
```

### 9-3. 다중 상속에서의 super()

```python
class ParentA:
    def show_value(self):
        print("ParentA")

class ParentB:
    def show_value(self):
        print("ParentB")

class Child(ParentA, ParentB):
    def show_value(self):
        super().show_value()    # MRO에 의해 ParentA.show_value 호출
        print("Child")

c = Child()
c.show_value()
# ParentA
# Child
```

`super()`는 MRO 순서상 다음 클래스를 호출하므로, `ParentA`가 먼저 호출되고 `ParentB`까지 갈 필요가 없으면 가지 않는다.

---

## 10. 이름 공간 탐색 순서

인스턴스에서 속성이나 메서드를 호출하면, **인스턴스 → 클래스 → 부모 클래스** 순서로 탐색한다.

```python
class Person:
    name = "unknown"        # 클래스 변수

    def talk(self):
        print(f"안녕, {self.name}")

p1 = Person()
p1.talk()          # 안녕, unknown  ← 인스턴스 변수 없으니 클래스 변수 사용

p2 = Person()
p2.name = "Kim"    # 인스턴스 변수 생성
p2.talk()          # 안녕, Kim      ← 인스턴스 변수 우선
```

각 인스턴스는 **독립적인 이름 공간**을 가지므로, 서로 다른 인스턴스의 데이터에 영향을 주지 않는다.

---

## 정리

| 구분 | 핵심 포인트 |
|------|-------------|
| 절차지향 vs 객체지향 | 대조가 아닌 보완 관계, 알고리즘은 절차지향 · 프로젝트는 객체지향 |
| 클래스 / 인스턴스 | 클래스 = 설계도(틀), 인스턴스 = 설계도로 찍어낸 개별 객체 |
| `__init__` | 인스턴스 생성 시 자동 호출되는 생성자 메서드, 초기화 담당 |
| `self` | 인스턴스 메서드의 첫 번째 인자, 호출한 인스턴스 자체를 가리킴 |
| 인스턴스 변수 / 클래스 변수 | 인스턴스 변수는 개별 독립, 클래스 변수는 전체 공유 |
| 메서드 3종 | 인스턴스(`self`) · 클래스(`cls`, `@classmethod`) · 스태틱(`@staticmethod`) |
| 매직 메서드 | `__str__`, `__len__` 등 — `print()`, `len()` 호출 시 자동 실행 |
| 상속 | 부모 클래스의 속성·메서드를 자식이 물려받아 코드 재사용 |
| 오버라이딩 | 자식이 부모 메서드를 동일 이름으로 재정의 (덮어쓰기) |
| 다중 상속 / MRO | 여러 부모 상속 시 깊이 우선 · 왼쪽 우선으로 탐색 |
| `super()` | MRO에 따라 부모 클래스를 자동 호출, 하드코딩 방지 |
| 이름 공간 탐색 | 인스턴스 → 클래스 → 부모 클래스 순서로 탐색 |
