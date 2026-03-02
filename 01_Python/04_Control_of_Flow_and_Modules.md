# [Python] 제어문과 모듈 — if, for, while, break/continue, 모듈/패키지

> **핵심 키워드:** #모듈 #import #from #패키지 #pip #조건문 #if #elif #else #반복문 #for #while #break #continue #range #enumerate #리스트컴프리헨션

---

## 학습 목표

* 모듈, 패키지, 라이브러리의 관계를 이해하고 import/from 구문 사용
* pip을 통한 외부 패키지 설치와 의존성 개념 파악
* 조건문(if/elif/else)으로 프로그램 흐름 분기 처리
* 반복문(for/while)으로 코드를 반복 실행하고, break/continue로 반복 제어
* 중첩 반복문, enumerate, 리스트 컴프리헨션 활용

---

## 1. 모듈

### 1-1. 모듈이란

모듈(Module)은 한 파일(.py)로 묶인 변수와 함수의 모음이다. 다른 사람이 만든 코드를 가져다 쓸 수도 있고, 내가 직접 만들 수도 있다.

```python
# my_math.py  ← 이 파일 자체가 하나의 모듈
def add(x, y):
    return x + y

def subtract(x, y):
    return x - y
```

> **주의:** 내부가 어떻게 돌아가는지 이해하지 못하고 가져다 쓰기만 하면, 그 모듈에 잡아먹힌다. 원리를 알고 쓰자.

### 1-2. import와 from

**import** — 모듈 전체를 가져온다. 사용할 때 `모듈명.함수명` 형태로 호출.

```python
import math
print(math.sqrt(4))    # 2.0
print(math.pi)         # 3.141592...
```

장점: 어떤 모듈에서 온 함수인지 명확하므로 이름 충돌이 없다. 단점: 매번 모듈명을 적어야 해서 코드가 길어진다.

**from ... import** — 모듈에서 특정 함수/변수만 가져온다. 모듈명 없이 바로 사용 가능.

```python
from math import sqrt, pi
print(sqrt(4))         # 2.0
print(pi)              # 3.141592...
```

장점: 코드가 간결해진다. 단점: 다른 모듈에서 같은 이름의 함수를 가져오면 이름 충돌이 발생할 수 있다.

> **Tip:** 이름 충돌이 우려될 때는 별칭(alias)을 사용한다. `import math as m` → `m.sqrt(4)`

### 1-3. 직접 모듈 만들기

파이썬 파일을 만들면 그 자체가 모듈이다. 같은 디렉토리에 있으면 파일명으로 import할 수 있다.

```python
# test.py에서 my_math 모듈 사용
import my_math
print(my_math.add(1, 2))       # 3
```

---

## 2. 패키지와 라이브러리

### 2-1. 모듈 → 패키지 → 라이브러리

| 개념 | 단위 | 설명 |
|------|------|------|
| 모듈 | .py 파일 | 함수와 변수를 담은 단일 파일 |
| 패키지 | 폴더(디렉토리) | 연관된 모듈들을 모아놓은 폴더 |
| 라이브러리 | 모듈+패키지 모음 | 파이썬과 함께 제공되는 표준 라이브러리(math, os, json 등) |

```
my_package/             ← 패키지 (폴더)
├── math_tools/         ← 하위 패키지
│   └── my_math.py      ← 모듈 (파일)
└── statistics/
    └── tools.py        ← 모듈 (파일)
```

패키지에서 특정 모듈을 가져올 때는 `from 패키지 import 모듈` 형태를 사용한다.

```python
from my_package.math_tools import my_math
print(my_math.add(1, 2))
```

핵심 패턴: `from 패키지 import 모듈` → `모듈.함수()` 형태로 사용한다.

### 2-2. pip — 외부 패키지 설치

pip(Package Installer for Python)은 외부 패키지를 다운로드하고 설치하는 파이썬 패키지 관리자다. PyPI(Python Package Index)에 등록된 패키지를 가져온다.

```bash
$ pip install requests          # 최신 버전 설치
$ pip install requests==2.31.0  # 특정 버전 설치
$ pip list                      # 설치된 패키지 목록
$ pip show requests             # 패키지 상세 정보 (라이선스 등)
```

> **Tip:** 명령어 앞의 `$` 기호는 "터미널에 입력하라"는 의미다. `$`를 직접 타이핑하는 것이 아니다.

### 2-3. 의존성

의존성(Dependency)이란 어떤 패키지가 실행되기 위해 먼저 설치되어야 하는 다른 패키지를 말한다. pip은 의존성을 자동으로 함께 설치해 준다.

비유하자면, 롤을 실행하려면 라이엇 클라이언트가 먼저 설치되어야 하는 것과 같다. `pip install requests` 하나만 실행해도 requests가 필요로 하는 certifi, urllib3 등이 함께 설치된다.

두 패키지가 서로 다른 버전의 같은 의존성을 요구하면 **버전 충돌**이 발생한다. 이를 해결하기 위해 **가상환경(venv)** 을 사용하여 프로젝트별로 독립적인 패키지 환경을 구성한다.

---

## 3. 조건문 (if / elif / else)

주어진 조건식을 평가하여 참(True)인 경우에만 해당 코드 블록을 실행한다.

```python
if 조건식1:
    # 조건식1이 참일 때 실행
elif 조건식2:
    # 조건식1이 거짓이고, 조건식2가 참일 때 실행
else:
    # 위 조건 모두 거짓일 때 실행
```

**if** 는 반드시 필요하다. **elif** 는 추가 조건이 필요할 때 여러 개 사용 가능. **else** 는 생략 가능하며, 위 조건이 전부 거짓일 때 실행된다. if/elif/else는 한 세트로 묶여 있어서, 하나가 참이면 나머지는 전부 건너뛴다.

### 3-1. 기본 예시

```python
a = 5
if a > 3:
    print("3 초과")     # ← 실행됨 (True)
else:
    print("3 이하")     # ← 건너뜀
```

### 3-2. 복수 조건문

조건은 위에서 아래로 **순차적**으로 비교된다. 먼저 참이 되는 조건의 코드만 실행하고 나머지는 건너뛴다.

```python
dust = 35
if dust > 150:
    print("매우 나쁨")
elif dust > 80:
    print("나쁨")
elif dust > 30:
    print("보통")       # ← 35 > 30 → True → 실행
else:
    print("좋음")       # ← 건너뜀
```

> **주의:** 조건 순서가 중요하다. 만약 `dust > 30`을 맨 위에 놓으면 170을 입력해도 "보통"이 출력된다. 큰 범위의 조건을 위에 배치해야 한다.

### 3-3. 중첩 조건문

if문 안에 또 다른 if문을 넣을 수 있다. 들여쓰기 레벨로 어디에 속하는지 구분한다.

```python
dust = 480
if dust > 150:
    print("매우 나쁨")           # ← 실행
    if dust > 300:
        print("위험! 나가지 마세요")  # ← 실행
elif dust > 80:
    print("나쁨")               # ← 건너뜀
```

---

## 4. 반복문 — for

임의의 시퀀스(리스트, 문자열, range 등) 항목들을 순서대로 하나씩 꺼내어 반복 실행한다.

```python
for 변수 in 반복가능한객체:
    # 변수를 활용한 코드 블록
```

### 4-1. 리스트 순회

```python
fruits = ["apple", "banana", "coconut"]
for item in fruits:
    print(item)
# apple
# banana
# coconut
```

동작 원리: 첫 번째 항목 "apple"이 item에 할당 → 코드 블록 실행 → 두 번째 항목 "banana"가 item에 할당 → 코드 블록 실행 → ... → 마지막 항목까지 순회하면 for문 종료.

### 4-2. 문자열 순회

문자열도 반복 가능한 객체이므로 한 글자씩 순회할 수 있다.

```python
for char in "Korea":
    print(char)
# K, o, r, e, a 각각 출력
```

### 4-3. range와 함께 사용

```python
for i in range(5):      # 0, 1, 2, 3, 4
    print(i)
```

인덱스로 리스트에 접근하고 싶을 때 `range(len(리스트))` 패턴을 사용한다.

```python
numbers = [4, 6, 10, 8, 5]
for i in range(len(numbers)):
    numbers[i] = numbers[i] * 2    # 각 요소를 2배로 재할당
```

### 4-4. 딕셔너리 순회

딕셔너리를 for문에 넣으면 **키(key)** 만 순회한다. 값이 필요하면 키로 접근한다.

```python
my_dict = {"x": 10, "y": 20, "z": 30}
for key in my_dict:
    print(key, my_dict[key])
# x 10 / y 20 / z 30
```

### 4-5. 중첩 반복문 (이중 for문)

바깥 for문이 한 번 돌 때마다 안쪽 for문이 전체를 순회한다. 총 실행 횟수는 `바깥 개수 × 안쪽 개수`.

```python
outers = ["A", "B"]
inners = ["C", "D"]
for outer in outers:
    for inner in inners:
        print(outer, inner)
# A C → A D → B C → B D
```

중첩 리스트의 내부 요소에 접근할 때도 이중 for문을 사용한다.

```python
elements = [["A", "B"], ["C", "D"]]
for element in elements:        # element = ["A", "B"] → ["C", "D"]
    for item in element:        # item = "A" → "B" → "C" → "D"
        print(item)
```

> **주의:** 3중 for문 이상은 속도가 급격히 느려지므로, 코드를 잘못 짰을 확률이 높다. 이중 for문까지가 일반적이다.

### 4-6. for문은 스코프를 만들지 않는다

파이썬의 for문은 함수와 달리 독자적인 로컬 스코프를 생성하지 않는다. for문이 끝난 후에도 반복 변수는 마지막 값을 유지한다.

```python
for item in ["apple", "banana", "coconut"]:
    pass
print(item)    # coconut  ← 에러가 아니라 마지막 값이 출력됨
```

이것은 파이썬 고유의 특성이다. 다른 언어에서는 for문 바깥에서 반복 변수에 접근하면 에러가 발생한다. 알고리즘 풀이 시 이 점을 모르면 의도치 않은 버그가 생길 수 있으니 주의하자.

---

## 5. 반복문 — while

조건식이 참(True)인 동안 코드 블록을 반복 실행한다. 조건이 거짓(False)이 되면 즉시 종료.

```python
a = 0
while a < 3:
    print(a)       # 0, 1, 2 출력
    a += 1         # 종료 조건으로 수렴
# a가 3이 되면 조건 False → while 종료
```

while문에는 **반드시 종료 조건**이 있어야 한다. 조건이 영원히 True인 구조라면 무한루프에 빠져 프로그램이 멈추지 않는다.

### for문 vs while문

| 구분 | for문 | while문 |
|------|-------|---------|
| 반복 기준 | 리스트, range 등 정해진 항목 | 조건식이 거짓이 될 때까지 |
| 반복 횟수 | 명확하게 정해져 있음 | 불확실 (조건 의존) |
| 주 사용처 | 리스트 순회, 횟수 반복 | 사용자 입력, 조건 도달까지 반복 |
| 위험 요소 | 적음 | 종료 조건 실수 시 무한루프 |

> **Tip:** 대부분의 상황에서는 for문이 적합하다. while문은 "언제 끝날지 모르지만 이 조건을 만족할 때까지 반복"해야 하는 경우에 사용한다. while문을 쓸 때는 종료 조건을 탄탄하게 설계하자.

---

## 6. 반복 제어 — break, continue, pass

### 6-1. break — 반복 즉시 종료

break가 실행되면 자신이 속한 가장 가까운 for문 또는 while문을 즉시 탈출한다. if문을 빠져나가는 것이 아니다.

```python
for i in range(10):       # 0 ~ 9
    if i == 5:
        break             # for문 자체를 종료
    print(i)
# 0, 1, 2, 3, 4 출력 (5부터는 출력되지 않음)
```

> **주의:** break는 if문이 아니라 반복문을 탈출한다. 초보자가 자주 혼동하는 부분이므로 주의.

### 6-2. continue — 다음 순회로 건너뛰기

continue가 실행되면 아래 코드를 건너뛰고 다음 반복으로 넘어간다. 반복문 자체를 종료하지는 않는다.

```python
for i in range(10):
    if i % 2 == 0:        # 짝수일 경우
        continue          # 아래 print를 건너뛰고 다음 순회
    print(i)
# 1, 3, 5, 7, 9 출력 (홀수만)
```

`i % 2 == 0`은 i를 2로 나눈 나머지가 0, 즉 짝수를 의미한다. 알고리즘에서 매우 자주 쓰이는 패턴이다.

### 6-3. pass — 아무것도 하지 않음

코드 블록에 반드시 무언가를 적어야 하지만 실행할 코드가 없을 때 자리를 채우는 역할이다. 미완성 코드의 플레이스홀더로도 사용한다.

```python
for i in range(10):
    pass              # 아직 로직 미구현, 나중에 채울 예정
```

### 6-4. for-else 구문 (파이썬 고유)

for문이 break 없이 끝까지 순회를 완료하면 else 블록이 실행된다. break가 한 번이라도 걸리면 else는 실행되지 않는다.

```python
numbers = [1, 3, 5, 7, 9]
for num in numbers:
    if num % 2 == 0:
        print(f"짝수 발견: {num}")
        break
else:
    print("짝수를 찾지 못했습니다")    # ← break 안 걸렸으므로 실행됨
```

다른 언어에서는 "찾았는지 여부"를 추적하는 별도 변수(found = False)가 필요하지만, 파이썬의 for-else를 쓰면 그 변수 없이 깔끔하게 처리할 수 있다.

---

## 7. enumerate — 인덱스와 값을 동시에

for문에서 인덱스와 값을 동시에 가져오고 싶을 때 사용하는 내장함수다.

```python
fruits = ["apple", "banana", "coconut"]

# 기존 방식: range(len(...))
for i in range(len(fruits)):
    print(i, fruits[i])

# enumerate 사용 (권장)
for index, value in enumerate(fruits):
    print(index, value)
# 0 apple / 1 banana / 2 coconut
```

쓰임새가 많으므로 반드시 익혀두자.

---

## 8. 리스트 컴프리헨션

for문을 한 줄로 작성하여 새로운 리스트를 생성하는 파이썬 고유 문법이다.

```python
# 일반 for문
result = []
for i in range(5):
    result.append(i * 2)

# 리스트 컴프리헨션 (동일 결과)
result = [i * 2 for i in range(5)]
# [0, 2, 4, 6, 8]
```

구조: `[표현식 for 변수 in 반복가능객체]`

메모리와 속도 면에서 일반 for문보다 효율적이지만, 복잡해지면 가독성이 떨어진다. 코드를 읽을 수 있는 수준까지는 알아두되, 익숙해지기 전까지는 일반 for문을 사용해도 괜찮다.

---

## 정리

**모듈/패키지**
* 모듈 = .py 파일 (함수+변수 모음), 패키지 = 폴더 (모듈 모음), 라이브러리 = 전체 모음
* `import 모듈` → `모듈.함수()` / `from 패키지 import 모듈` → `모듈.함수()`
* 외부 패키지는 `pip install 패키지명`으로 설치, 의존성은 pip이 자동 관리
* 버전 충돌은 가상환경(venv)으로 해결

**조건문 (if/elif/else)**
* if로 시작 → 조건이 참이면 코드 블록 실행, 거짓이면 다음 elif/else로 이동
* 한 세트에서 하나만 실행되고 나머지는 건너뜀
* 조건 순서가 중요 — 큰 범위를 위에 배치
* 중첩 조건문도 가능, 들여쓰기로 소속 구분

**반복문 (for/while)**
* for: 정해진 항목을 순서대로 순회. 리스트, 문자열, range, 딕셔너리 모두 가능
* while: 조건이 거짓이 될 때까지 무한 반복. 반드시 종료 조건 필요
* 이중 for문: 바깥 × 안쪽 횟수만큼 실행. IM 시험의 기본 소양
* for문은 스코프를 만들지 않음 — 반복 변수가 for문 밖에서도 마지막 값 유지

**반복 제어**
* break: 반복문 즉시 탈출 (if문이 아닌 for/while을 빠져나감)
* continue: 아래 코드 건너뛰고 다음 순회로
* pass: 아무것도 안 함 (자리 채우기)
* for-else: break 없이 끝까지 돌면 else 실행 (파이썬 고유)

**필수 도구**
* enumerate: 인덱스 + 값 동시 접근
* 리스트 컴프리헨션: for문 한 줄 표현, 효율적이지만 가독성 주의

> **강사님 조언:** if문과 for문은 코딩의 전부다. 함수를 몰라도 알고리즘은 할 수 있지만, if문과 for문을 모르면 아무것도 못한다. 여기 한 번 놓치면 끝도 없이 밀리므로, 오늘 이해 못했으면 집에 가서라도 반드시 공부하자.
