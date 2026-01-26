# Day03 - 1월 26일 수업 정리

---

# Part 1. 함수(Function) 기본 개념

## 1. 함수란?

- **특정 기능을 수행하는 코드 묶음**
- 코드 **재사용**, **가독성 향상**, **유지보수성 증가** 목적

```python
def add(a, b):
    return a + b
```

---

## 2. 함수 구조

```python
def 함수명(매개변수):
    실행 코드
    return 반환값
```

- `def` : 함수 정의 키워드
- 들여쓰기 영역 = 함수 내부

---

## 3. 함수 호출

```python
add(3, 5)
```

---

## 4. return & None

```python
def f():
    print(10)

x = f()   # None
```

- return 생략 → **None 자동 반환**

---

# Part 2. 매개변수 & 인자

## 5. 개념 정리

| 구분 | 의미 |
|------|------|
| 매개변수(Parameter) | 함수가 받는 변수 |
| 인자(Argument) | 실제 전달되는 값 |

---

## 6. 인자 전달 방식

### 위치 인자

```python
f(3, 5)
```

### 기본 인자

```python
def f(a, b=10):
    pass
```

### 키워드 인자

```python
f(b=5, a=3)
```

---

## 7. 가변 인자

```python
def f(*args):
    pass

def g(**kwargs):
    pass
```

- `*args` → tuple
- `**kwargs` → dict

---

# Part 3. 재귀 함수 (Recursion)

## 8. 개념

- **함수가 자기 자신을 다시 호출**
- 반드시 **종료 조건(Base Case)** 필요

```python
def factorial(n):
    if n == 1:
        return 1
    return n * factorial(n-1)
```

- **개념만 이해, 구현은 알고리즘 파트에서 심화 학습**

---

# Part 4. 내장 함수 (Built-in Functions)

| 함수 | 설명 |
|------|------|
| len | 길이 |
| max | 최댓값 |
| min | 최솟값 |
| sum | 합계 |
| sorted | 정렬 |

⚠️ **학습 단계에서는 직접 구현 → 내장 함수 의존 금지**

---

# Part 5. map & zip

## 9. map

```python
list(map(str, [1,2,3]))
```

- 모든 요소에 함수 적용
- 결과 → **iterator → list 변환 필요**

---

## 10. zip

```python
list(zip([1,2],[3,4]))
```

- 여러 iterable을 **세로 방향으로 묶음**

---

# Part 6. Scope & LEGB Rule (매우 중요)

## 11. 스코프 종류

| 범위 | 설명 |
|--------|------|
| Local | 함수 내부 |
| Enclosed | 중첩 함수 외부 |
| Global | 파일 전체 |
| Built-in | 파이썬 기본 |

→ **LEGB 순서로 탐색**

---

## 12. global 키워드

```python
x = 10

def f():
    global x
    x += 1
```

- 전역 변수 수정 시 사용
- **남용 금지**

---

# Part 7. Packing & Unpacking

```python
a, b, *c = [1,2,3,4,5]
```

---

# Part 8. Lambda

```python
lambda x, y: x + y
```

---

# Part 9. 모듈(Module)

## 13. 모듈 개념

- **함수와 변수들의 묶음 파일**

---

## 14. import 방식

### import 모듈

```python
import math
math.sqrt(4)
```

### from 모듈 import 함수

```python
from math import sqrt
sqrt(4)
```

| 방식 | 장점 | 단점 |
|------|------|------|
| import | 충돌 방지, 명확 | 코드 길어짐 |
| from | 간결 | 이름 충돌 위험 |

→ **실무 권장: import 방식**

---

# Part 10. 조건문 (if / elif / else)

```python
if a > 10:
    print('big')
elif a > 5:
    print('mid')
else:
    print('small')
```

- **위에서부터 순차 비교 → 처음 참인 블록만 실행**

---

## 15. 중첩 조건문

```python
if a > 10:
    if a > 100:
        print('huge')
```

---

# Part 11. 반복문 (for / while)

## 16. for

```python
for i in range(5):
    print(i)
```

## 17. while

```python
while 조건:
    실행
```

---

# Part 12. Day03 핵심 요약

- 함수 구조 & 인자 전달 방식
- 재귀 개념
- 내장 함수 & map / zip
- **Scope & LEGB Rule**
- 모듈 import 방식
- 조건문 & 반복문 → **알고리즘 핵심 기반**

---

### 🔗 이동

- 📝 복습 노트: [review.md](./review.md)
- ⬆ 메인: [README](../README.md)