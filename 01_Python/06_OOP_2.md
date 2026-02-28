# Day06 — 상속(Inheritance) · super() · MRO · 예외 처리

> **학습 목표**
> - 상속 구조를 해석할 수 있다.
> - `super()`의 실제 동작 원리를 MRO 기반으로 이해한다.
> - 다중 상속에서 메서드 탐색 순서를 예측할 수 있다.
> - 에러(Error)와 예외(Exception)의 차이를 이해하고, 기본적인 예외 처리를 수행할 수 있다.

---

## 1. 오늘의 핵심 요약
- 상속은 **기존 클래스 기능을 재사용·확장**하기 위한 구조이다.
- `super()`는 **부모 클래스 직접 지정이 아니라 MRO 기반 다음 대상 호출**이다.
- MRO(Method Resolution Order)는 **속성과 메서드를 찾는 순서 규칙**이다.
- 다중 상속에서는 **직관과 다른 순서로 호출**될 수 있다.
- 예외 처리는 **프로그램 비정상 종료 방지 + 안정성 확보**가 목적이다.

---

## 2. 개념 구조 정리

```
객체지향 심화
├─ 상속 (Inheritance)
│   ├─ 단일 상속
│   ├─ 다중 상속
│   └─ super()
│
├─ MRO (Method Resolution Order)
│   ├─ 탐색 순서 규칙
│   └─ 다중 상속 충돌 해결
│
└─ 에러 & 예외
    ├─ 에러(Error)
    ├─ 예외(Exception)
    └─ try / except
```

---

## 3. 상속(Inheritance) 기본 개념

### 3-1. 상속이란?

> **부모 클래스의 속성과 메서드를 자식 클래스가 물려받아 사용하는 구조**

```python
class Parent:
    def greet(self):
        print('Hello')

class Child(Parent):
    pass

c = Child()
c.greet()   # Hello
```

### 3-2. 상속의 목적

- 코드 재사용
- 유지보수 용이
- 공통 기능 중앙 관리

---

## 4. super()의 본질

### 4-1. super()는 "부모"가 아니다

```python
super().method()
```

➡ **의미:**
> MRO 기준으로 **다음에 호출되어야 할 클래스의 메서드 실행**

---

### 4-2. 단일 상속에서 super()

```python
class A:
    def show(self):
        print('A')

class B(A):
    def show(self):
        super().show()
        print('B')

b = B()
b.show()
```

**출력 순서**
```
A
B
```

---

## 5. 다중 상속 & MRO 핵심

### 5-1. MRO란?

> **Method Resolution Order**
> → 속성과 메서드를 탐색하는 공식적인 순서 규칙

```python
print(ClassName.mro())
```

---

### 5-2. 다중 상속 구조

```python
class A:
    pass

class B(A):
    pass

class C(A):
    pass

class D(B, C):
    pass

print(D.mro())
```

**출력 구조**
```
[D, B, C, A, object]
```

➡ **핵심 원칙**
- 항상 **자식 → 부모 → 조상**
- **중복 탐색 방지**

---

## 6. super() + MRO 실행 흐름 (시험 핵심)

```python
class A:
    def show(self): print('A')

class B(A):
    def show(self):
        super().show()
        print('B')

class C(A):
    def show(self):
        super().show()
        print('C')

class D(B, C):
    def show(self):
        super().show()
        print('D')

D().show()
```

**실제 실행 순서**
```
A
C
B
D
```

➡ **이유:**
> `super()`는 **부모가 아니라 D의 MRO 기준 다음 대상 호출**

---

## 7. MRO 사고 흐름 정리

> **MRO는 항상 "호출한 클래스 기준"으로만 해석한다.**

1. D 호출
2. → B
3. → C
4. → A

---

## 8. 에러(Error) vs 예외(Exception)

| 구분 | 의미 | 예시 |
|--------|--------|------|
| Error | 프로그램 자체 문제 | SyntaxError |
| Exception | 실행 중 발생 상황 | ZeroDivisionError |

---

## 9. 예외 처리 기본 구조

```python
try:
    x = int(input())
    print(10 / x)
except ZeroDivisionError:
    print('0으로 나눌 수 없습니다')
except ValueError:
    print('숫자만 입력하세요')
else:
    print('정상 실행')
finally:
    print('종료')
```

---

## 10. 실수 & 감점 포인트

| 실수 | 결과 |
|---------|--------|
| super() = 부모 호출로 오해 | MRO 해석 실패 |
| 다중 상속 순서 암기 | 구조 바뀌면 오답 |
| 예외 처리 누락 | 런타임 종료 |

---

## 11. 시험 & 알고리즘 연결

- MRO 출력 결과 해석
- super 실행 순서 추론
- 다중 상속 구조 해석
- try/except 구조 완성

---

## 12. 5분 요약

- 상속 → 기능 확장
- super → MRO 기준 다음 호출
- MRO → 탐색 순서 규칙
- 예외 → 프로그램 보호 장치

---

### 이동
👉 [복습 노트](./review.md)
👉 [메인 README](../README.md)

