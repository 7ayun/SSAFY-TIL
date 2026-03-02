# [Python] 에러와 예외 처리 — 에러 종류, 예외 처리(try/except)

> **핵심 키워드:** #버그 #디버깅 #SyntaxError #NameError #TypeError #ValueError #IndexError #KeyError #예외처리 #try #except #else #finally #as #예외계층 #BaseException

---

## 학습 목표

* 버그와 디버깅의 개념을 이해하고, print문을 활용한 디버깅 습관 형성
* 문법 에러(Syntax Error)와 예외(Exception)의 차이 구분
* 주요 예외 종류(ZeroDivision, Name, Type, Value, Index, Key 등)를 에러 메시지로 파악
* try / except / else / finally 구문으로 예외 처리 작성
* 예외 계층 구조를 이해하고, except 배치 순서의 중요성 인지

---

## 1. 버그와 디버깅

### 1-1. 버그 (Bug)

소프트웨어에서 발생하는 오류나 결함으로, 프로그램의 **예상된 동작과 실제 동작 사이의 불일치**를 의미한다. 실제로 1940년대에 컴퓨터에 나방(bug)이 끼어 합선을 일으킨 사건에서 유래했다.

### 1-2. 디버깅 (Debugging)

버그를 찾아내고 수정하는 과정이다.

**디버깅 방법**

- **print문 활용**: 에러가 나는 지점 주변에 `print()`를 찍어서 변수 값이 예상대로인지 확인한다. 가장 기본적이고 효과적인 방법이다.
- **디버거 도구**: PyCharm 등의 IDE에서 제공하는 디버깅 기능(브레이크포인트, 스텝 실행 등)을 활용한다.

> **강사님 강조**: 에러가 나면 손만 비비고 있지 말고, 에러 지점 위에 `print()`를 찍어서 내가 생각한 대로 값이 출력되는지 확인하라. 또한 에러 메시지를 GPT에 바로 던지지 말고, **직접 읽는 연습**을 하라. 익숙해지면 에러 메시지를 보자마자 바로 해결할 수 있다.

---

## 2. 에러의 종류

### 2-1. 문법 에러 (Syntax Error)

코드 실행 전에 파이썬이 감지하는, 문법 규칙 위반 에러이다.

```python
# SyntaxError: invalid syntax — 올바르지 않은 문법
while                       # 조건식 누락, 콜론 누락

# SyntaxError: cannot assign to literal — 값에 할당 불가
5 = 3                       # 변수가 아닌 값에 할당 시도

# SyntaxError: EOL while scanning string literal — 문자열 미종료
print("hello)               # 닫는 따옴표 누락

# SyntaxError: unexpected EOF while parsing — 괄호 미닫힘
print("hello"               # 닫는 괄호 누락
```

### 2-2. 예외 (Exception)

프로그램 실행 중에 발생하는 에러이다. 문법적으로는 문제가 없지만, 실행 과정에서 문제가 생긴 것이다.

**ZeroDivisionError** — 0으로 나눌 때

```python
10 / 0    # ZeroDivisionError: division by zero
```

**NameError** — 정의하지 않은 변수를 사용할 때

```python
print(x)  # NameError: name 'x' is not defined
```

**TypeError** — 호환되지 않는 타입 간 연산, 인자 개수 불일치

```python
"2" + 2           # TypeError: can only concatenate str to str
sum()              # TypeError: sum expected at least 1 argument, got 0
sum(1, 2, 3)       # TypeError: sum expected at most 2 arguments, got 3
```

**ValueError** — 타입은 맞지만 값이 부적절할 때

```python
int("1.5")         # ValueError: invalid literal for int()
                    # 소수점 문자열은 int로 직접 변환 불가 (float 거쳐야 함)

[1, 2, 3].index(6) # ValueError: 6 is not in list
```

**IndexError** — 리스트 인덱스 범위 초과

```python
lst = []
lst[2]             # IndexError: list index out of range
```

> 알고리즘에서 가장 많이 보는 에러. 리스트 크기를 제대로 파악하지 못하고 범위를 넘어서는 인덱스에 접근했을 때 발생한다.

**KeyError** — 딕셔너리에 없는 키로 접근할 때

```python
d = {"name": "Alice"}
d["age"]           # KeyError: 'age'
```

`get()` 메서드를 사용하면 KeyError 없이 안전하게 접근할 수 있다.

**ModuleNotFoundError** — 존재하지 않는 모듈을 import할 때

```python
import hahaha      # ModuleNotFoundError: No module named 'hahaha'
```

**IndentationError** — 들여쓰기가 잘못됐을 때

```python
for i in range(3):
print(i)           # IndentationError: expected an indented block
```

**KeyboardInterrupt** — `Ctrl + C`로 프로그램을 강제 종료할 때

```python
while True:
    pass
# Ctrl + C → KeyboardInterrupt
```

> **팁**: 터미널에서 프로그램이 무한루프에 빠지거나 멈추면 `Ctrl + C`를 누르면 된다. 그래도 안 되면 터미널 자체를 종료한다.

---

## 3. 에러 메시지 읽는 법

에러 메시지는 크게 **에러 타입**과 **상세 설명**으로 구성된다. 영어 단어만 이해하면 대부분 원인을 파악할 수 있다.

```
TypeError: can only concatenate str (not "int") to str
│         └─ 상세 설명: 문자열은 문자열끼리만 결합 가능, int는 안 됨
└─ 에러 타입: 타입 에러
```

```
IndexError: list index out of range
│           └─ 리스트 인덱스가 범위를 벗어남
└─ 인덱스 에러
```

> **강사님 강조**: 에러 메시지의 기술 영어 단어들은 알아둬야 한다. `invalid`(무효한), `defined`(정의된), `unexpected`(예상치 못한), `expected`(기대된), `literal`(문자 리터럴), `concatenate`(결합) 등.

---

## 4. 예외 처리 (try / except / else / finally)

### 4-1. 기본 구조

```python
try:
    # 에러가 발생할 수 있는 코드
except 에러타입:
    # 해당 에러 발생 시 실행할 코드
else:
    # 에러가 발생하지 않았을 때 실행할 코드
finally:
    # 에러 발생 여부와 상관없이 무조건 실행할 코드
```

| 블록 | 실행 조건 |
|------|----------|
| `try` | 에러 발생 여부를 검사하는 영역 |
| `except` | 에러 발생 시 실행 |
| `else` | 에러 미발생 시 실행 |
| `finally` | 에러 여부 무관, **무조건** 실행 |

### 4-2. 기본 예제

```python
try:
    result = 10 / 0
except ZeroDivisionError:
    print("0으로 나눌 수 없습니다")
# 출력: 0으로 나눌 수 없습니다
# 프로그램이 종료되지 않고 계속 실행됨
```

예외 처리를 하지 않았다면 에러 메시지가 뜨고 프로그램이 종료됐을 것이다.

### 4-3. 복수 예외 처리

```python
try:
    num = int(input("숫자 입력: "))
    result = 100 / num
except ValueError:
    print("숫자가 아닙니다")
except ZeroDivisionError:
    print("0으로 나눌 수 없습니다")
except:                          # 그 외 모든 에러
    print("알 수 없는 에러 발생")
else:
    print(f"결과: {result}")
finally:
    print("계산 종료")
```

실무에서는 모든 에러 타입을 외우기 어려우므로, 구체적인 에러 몇 개를 처리한 뒤 마지막에 `except:`로 나머지를 포괄하는 방식이 일반적이다.

### 4-4. else와 finally 활용

```python
try:
    num = int(input("숫자 입력: "))
except ValueError:
    print("잘못된 입력입니다")
else:
    print(f"입력한 숫자: {num}")    # 에러 없을 때만 실행
finally:
    print("프로그램 종료")           # 항상 실행
```

---

## 5. 예외 객체 확인 (as 키워드)

`as` 키워드로 발생한 예외 객체를 변수에 담아 상세 정보를 확인할 수 있다.

```python
try:
    lst = [1, 2, 3]
    print(lst[5])
except IndexError as e:
    print(f"에러 발생: {e}")
# 출력: 에러 발생: list index out of range
```

어떤 에러가 발생했는지 구체적으로 모를 때, `except Exception as e`로 포괄적으로 잡아서 에러 내용을 출력하면 원인 파악에 도움이 된다.

```python
try:
    # 어떤 코드
    ...
except Exception as e:
    print(f"에러: {e}")
```

---

## 6. 예외 계층 구조

파이썬의 예외 클래스는 **상속 구조**로 이루어져 있다. 상위 클래스를 `except`에 넣으면 하위 에러까지 모두 잡아버리므로, **구체적인 에러를 먼저**, 포괄적인 에러를 나중에 배치해야 한다.

```
BaseException
 └── Exception
      ├── ValueError
      ├── TypeError
      ├── IndexError
      ├── KeyError
      ├── ZeroDivisionError
      └── ...
```

```python
# 잘못된 순서 — BaseException이 모든 에러를 가져감
try:
    ...
except BaseException:        # 여기서 모든 에러가 처리됨
    print("숫자를 넣어주세요")
except ValueError:           # 도달 불가
    print("값 에러")

# 올바른 순서 — 구체적 에러를 먼저
try:
    ...
except ValueError:
    print("값 에러")
except ZeroDivisionError:
    print("0 나누기 에러")
except Exception:            # 나머지 포괄
    print("기타 에러")
```

---

## 7. 실전 팁: 알고리즘 시험에서의 예외 처리

> **강사님 팁**: 알고리즘 채점 시스템은 테스트 케이스 중간에 에러가 발생하면 **프로그램을 종료**시킨다. 예를 들어 50개 테스트 케이스 중 20번에서 에러가 나면, 21~50번은 채점되지 않아 19점만 받는다. 에러가 날 법한 곳에 `try-except`를 넣으면 프로그램이 종료되지 않으므로, 틀린 부분만 넘어가고 나머지는 정상 채점된다.

```python
# 알고리즘 시험에서 안전하게 처리하는 패턴
try:
    # 에러가 날 수 있는 코드
    result = solve(test_case)
except:
    pass    # 에러 나도 프로그램 종료 방지
```

---

## 정리

| 구분 | 핵심 포인트 |
|------|-------------|
| 버그 / 디버깅 | 버그 = 예상과 실제의 불일치, 디버깅 = 원인 식별 후 수정 |
| 문법 에러 (Syntax Error) | 실행 전 감지 — 콜론 누락, 따옴표·괄호 미닫힘, 값에 할당 등 |
| 예외 (Exception) | 실행 중 발생 — `NameError`, `TypeError`, `ValueError`, `IndexError` 등 |
| `IndexError` | 알고리즘에서 가장 많이 만나는 에러, 리스트 범위 초과 접근 |
| 에러 메시지 읽기 | 에러 타입 + 상세 설명으로 구성, 영어 단어만 이해하면 원인 파악 가능 |
| `try` / `except` | `try` = 에러 발생 가능 코드, `except` = 에러 시 실행 코드 |
| `else` / `finally` | `else` = 에러 미발생 시 실행, `finally` = 에러 여부 무관 무조건 실행 |
| 예외 계층 주의 | 상위 예외(`BaseException`)를 먼저 쓰면 하위 `except`에 도달 불가 |
| `as` 키워드 | `except Exception as e`로 에러 상세 정보 확인 가능 |
