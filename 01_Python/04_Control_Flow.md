# [Python] 제어문 & 모듈 (Control of Flow & Modules)
> **핵심 키워드:** #Python #모듈 #패키지 #pip #조건문 #if #반복문 #for #while #break #continue #리스트컴프리헨션

---

## 🎯 학습 목표
* 모듈·패키지·라이브러리의 관계 및 import 방법 숙지
* `pip`를 이용한 외부 패키지 설치 방법 이해
* `if` / `elif` / `else` 조건문 흐름 완벽 이해
* `for`문으로 다양한 자료형 순회 구현
* `for`문이 스코프를 만들지 않는 파이썬 특성 인지
* 중첩 조건문·중첩 반복문 작성 및 흐름 추적
* `while`문의 조건 기반 반복 원리 및 무한루프 위험 이해
* `break` / `continue` / `pass` 반복 제어 키워드 구분
* `for-else` 구문 (파이썬 전용 문법) 이해
* 리스트 컴프리헨션 기본 문법 이해
* `enumerate()` 내장함수 활용법 이해

---

## 💡 주요 개념 정리

### 1. 모듈(Module)이란?
* **한 파일(`.py`)로 묶인 변수와 함수의 모음**
* 다른 개발자가 만들어 놓은 코드를 재사용하는 개념
* 모듈을 사용하는 이유: 효율성 / 안정성(검증된 코드) / 확장성

### 2. 패키지(Package)와 라이브러리(Library) 관계

```
라이브러리(Library)
└── 패키지(Package) — 연관된 모듈들을 담은 폴더(디렉토리)
    └── 모듈(Module) — .py 파일 하나
        └── 함수(Function) / 변수(Variable)
```

* **모듈** → `.py` 파일 단위
* **패키지** → 폴더(디렉토리) 단위, 관련 모듈들의 묶음
* **라이브러리** → 패키지와 모듈의 총집합

---

## 💻 기능 구현 및 코드 실습

### 🔧 모듈 불러오기 — `import` vs `from ... import`

#### ① `import` 방식 (권장)
```python
import math

print(math.pi)          # 3.141592653589793
print(math.sqrt(16))    # 4.0
print(math.fabs(-5))    # 5.0 — 절댓값
```
* 장점: 어느 모듈에서 왔는지 **명확** / 이름 충돌 방지
* 단점: 코드가 길어짐 (`모듈명.함수명` 형태 필수)

#### ② `from ... import` 방식
```python
from math import pi, sqrt

print(pi)       # 3.141592...
print(sqrt(16)) # 4.0
```
* 장점: 코드가 **간결**
* 단점: 이름 충돌 가능성 / 함수 출처 추적 어려움

#### ③ 와일드카드 import — 지양
```python
from math import *   # math 내 모든 요소 가져옴 → 충돌 위험, 비권장
```

#### ④ `as` — 별칭(alias) 지정
```python
import math as m           # 이름 충돌 방지 or 줄여쓰기
from math import sqrt as sq

print(m.pi)     # 3.141592...
print(sq(25))   # 5.0
```

> 💡 강사님: *"실무에서는 버그를 내지 않기 위해 `import 모듈명` 후 `모듈명.함수명()` 형태를 선호한다."*

---

### 🔧 커스텀 모듈 만들기

```python
# my_math.py — 직접 만든 모듈
PI = 3.141592

def add(x, y):
    return x + y

def remainder(x, y):
    return x % y
```

```python
# test.py — 모듈 사용 파일 (같은 폴더에 위치)
import my_math

print(my_math.PI)           # 3.141592
print(my_math.add(10, 20))  # 30
```

---

### 🔧 패키지 구조 및 사용

```
my_package/                    ← 패키지 (폴더)
├── math/
│   └── my_math.py             ← 모듈
└── statistics/
    └── tools.py               ← 모듈
```

```python
# 패키지에서 특정 모듈만 가져오기
from my_package.math import my_math

print(my_math.add(1, 2))   # 3
```

> 핵심: **`from 패키지 import 모듈` → `모듈.함수명()` 형태**로 사용

---

### 🔧 pip — 외부 패키지 관리

```bash
# 외부 패키지 설치 (터미널 명령어 — $ 는 터미널 입력 의미)
$ pip install requests          # 최신 버전 설치
$ pip install requests==2.28.0  # 특정 버전 설치
$ pip install requests>=2.0.0   # 최소 버전 지정 설치

# 설치된 패키지 목록 확인
$ pip freeze

# 특정 패키지 정보 확인 (라이선스 포함)
$ pip show requests
```

* `pip` = Python Package Installer — 파이썬 설치 시 함께 설치
* **의존성(Dependency):** 패키지 실행에 필요한 다른 패키지들 → pip가 자동으로 함께 설치
* **버전 충돌** 문제 → `venv` (가상환경)로 해결

> 💡 `$` 기호 = 터미널에 입력하라는 표시 (달러 기호 자체를 입력하는 게 아님)

---

### 🔧 조건문 (if / elif / else)

* 조건식이 **참(True)**일 때만 해당 코드 블록 실행
* `if` → 필수 / `elif` → 선택(복수 조건) / `else` → 선택(나머지 전부)
* 위에서부터 순차적으로 검사 → 참인 블록 하나 실행 후 나머지 **건너뜀**

```python
# 기본 구조
dust = 155

if dust > 150:
    print("매우 나쁨")
elif dust > 80:
    print("나쁨")
elif dust > 30:
    print("보통")
else:
    print("좋음")
# 출력: 매우 나쁨
```

```python
# 중첩 조건문
dust = 480

if dust > 150:
    print("매우 나쁨")
    if dust > 300:         # 중첩 if — 위 조건 통과 후 추가 검사
        print("오늘은 외출 금지!")
elif dust > 80:
    print("나쁨")
else:
    print("보통 이하")
# 출력: 매우 나쁨 → 오늘은 외출 금지!
```

> ⚠️ **들여쓰기 레벨 = 코드 블록 범위** → 반드시 정확히 맞춰야 함

---

### 🔧 for 반복문

* **이터러블(iterable)한 객체**를 순서대로 하나씩 순회
* 이터러블: `str`, `list`, `tuple`, `range`, `dict`, `set` 등

```python
# 기본 구조
for 변수 in 이터러블:
    # 코드 블록

# 리스트 순회
fruits = ['apple', 'banana', 'coconut']
for item in fruits:
    print(item)
# apple → banana → coconut

# 문자열 순회
for char in "KOREA":
    print(char)
# K → O → R → E → A

# range로 횟수 지정 순회
for i in range(5):       # 0, 1, 2, 3, 4
    print(i)

for i in range(1, 6):    # 1, 2, 3, 4, 5
    print(i)

# 딕셔너리 순회 — 기본적으로 key만 반환
my_dict = {'x': 1, 'y': 2, 'z': 3}
for key in my_dict:
    print(key, my_dict[key])   # key와 value 함께 출력
# x 1 → y 2 → z 3
```

#### 🔑 파이썬 특성: for문은 스코프를 만들지 않음
```python
for i in range(3):
    pass

print(i)   # 2 — 에러 없이 마지막 값 출력 (파이썬 특유)
# ⚠️ 다른 언어는 for문 외부에서 i 접근 시 에러

# for문 실행 전에는 에러
# print(i)  → NameError: name 'i' is not defined
```

> 💡 강사님: *"`for`문은 함수·클래스와 달리 지역 스코프를 만들지 않는다. 파이썬만의 특이한 특성."*

---

### 🔧 range() 활용 — 인덱스로 리스트 순회

```python
numbers = [4, 6, 10, 8, 2]

# 값 순회 (일반)
for num in numbers:
    print(num)

# 인덱스로 순회 — 값 수정 시 필요
for i in range(len(numbers)):     # range(5) → 0, 1, 2, 3, 4
    numbers[i] = numbers[i] * 2   # 각 요소를 2배로 재할당

print(numbers)   # [8, 12, 20, 16, 4]
```

---

### 🔧 중첩 for문 (이중 포문)

* 바깥 반복 1회 동안 안쪽 반복이 **전체 순회**
* 총 반복 횟수 = 바깥 횟수 × 안쪽 횟수

```python
outers = ['A', 'B']
inners = ['C', 'D']

for outer in outers:
    for inner in inners:
        print(outer, inner)
# A C → A D → B C → B D (총 4회)
```

```python
# 중첩 리스트 순회
matrix = [['A', 'B'], ['C', 'D']]

for element in matrix:          # element = ['A','B'] → ['C','D']
    for item in element:        # item = 'A' → 'B' → 'C' → 'D'
        print(item)
# A → B → C → D
```

> ⚠️ 강사님: *"3중 포문 이상은 잘못 짠 코드일 확률이 높다. N×M×K회 반복으로 성능 급저하."*

---

### 🔧 while 반복문

* **조건식이 참(True)인 동안** 코드 블록 반복 실행
* 반복 횟수가 불확실하거나, 특정 조건 달성 시 종료할 때 사용
* **종료 조건 필수** — 없으면 무한루프 발생

```python
# 기본 구조
a = 0
while a < 3:        # 조건식: a가 3 미만이면 반복
    print(a)        # 0 → 1 → 2
    a += 1          # 반드시 종료 조건 방향으로 업데이트
# 출력: 0 1 2

# 사용자 입력 기반 반복 — while 대표 활용 패턴
while True:
    n = int(input("양의 정수를 입력하세요: "))
    if n <= 0:
        if n < 0:
            print("음수를 입력했습니다.")
        else:
            print("0은 양의 정수가 아닙니다.")
    else:
        print("잘했습니다!")
        break       # 양의 정수 입력 시 탈출
```

> 💡 강사님: *"실무에서 while문을 선호하지 않는다. 종료 조건 실수 시 무한루프로 서비스 장애 발생 가능. 불가피할 때는 무한루프 탈출 코드를 반드시 함께 작성."*
>
> 💡 평가 팁: *"최근 SSAFY IM 시험 추세에서 while문 비중이 증가하고 있어 연습 필요."*

---

### 🔧 반복 제어 — break / continue / pass

| 키워드 | 역할 | 적용 대상 |
|---|---|---|
| `break` | 반복문 **즉시 종료** | 가장 가까운 for/while |
| `continue` | 현재 반복 **건너뜀**, 다음 반복 진행 | 가장 가까운 for/while |
| `pass` | **아무것도 안 함** (자리 채우기용) | if/for/while/함수 등 |

```python
# break — 특정 조건에서 반복 완전 종료
for i in range(10):
    if i == 5:
        break       # i=5 도달 시 for문 종료
    print(i)        # 0 1 2 3 4

# continue — 해당 반복 건너뛰고 다음 반복 진행
for i in range(10):
    if i % 2 == 0:  # 짝수 건너뜀
        continue
    print(i)        # 1 3 5 7 9

# pass — 코드 블록 자리를 채워두기만 함 (미완성 코드 표시용)
for i in range(5):
    if i == 3:
        pass        # 나중에 채울 로직 자리 확보
    print(i)        # 0 1 2 3 4 — pass는 아무 영향 없음
```

> ⚠️ `break`는 **if문이 아닌 반복문(for/while)을 탈출**한다는 점에 주의

---

### 🔧 for-else 구문 (파이썬 전용)

* `for`문이 **한 번도 `break` 없이** 끝까지 순회 완료했을 때 `else` 실행
* `break`로 탈출한 경우 `else` 실행 안 됨

```python
# 다른 언어 방식 — flag 변수 필요
numbers = [1, 3, 5, 7]
found_even = False
for num in numbers:
    if num % 2 == 0:
        found_even = True
        break
if not found_even:
    print("짝수를 찾지 못했습니다.")

# 파이썬 for-else 방식 — flag 변수 불필요
for num in numbers:
    if num % 2 == 0:
        print(f"짝수 {num} 발견!")
        break
else:
    # break 없이 for문이 끝까지 돌았을 때 실행
    print("짝수를 찾지 못했습니다.")
# 출력: 짝수를 찾지 못했습니다.
```

> 💡 강사님: *"실무에서 이 코드 짜는 게 진짜 어려웠다. for-else를 알면 코드가 훨씬 깔끔해진다."*

---

### 🔧 리스트 컴프리헨션 (List Comprehension)

* **`for`문을 한 줄로** 작성하는 파이썬 전용 문법
* 메모리·속도 효율 우수 (이터레이터 방식) → 알고리즘에서 유용

```python
# 일반 for문
squares = []
for i in range(1, 6):
    squares.append(i ** 2)
print(squares)   # [1, 4, 9, 16, 25]

# 리스트 컴프리헨션 — 한 줄로
squares = [i ** 2 for i in range(1, 6)]
print(squares)   # [1, 4, 9, 16, 25]

# 조건 포함
even_squares = [i ** 2 for i in range(1, 11) if i % 2 == 0]
print(even_squares)   # [4, 16, 36, 64, 100]
```

> 💡 강사님: *"가독성이 다소 떨어질 수 있어 개인적으로 선호하지 않지만, 알고리즘 시험에서 속도 이점이 크다. 읽을 수 있는 수준까지는 반드시 익혀두자."*

---

### 🔧 enumerate() — 인덱스 + 값 동시 순회

```python
fruits = ['apple', 'banana', 'coconut']

# 일반 방식 — 인덱스 + 값 동시 출력 시 번거로움
for i in range(len(fruits)):
    print(i, fruits[i])

# enumerate 방식 — 인덱스와 값을 동시에 간결하게
for idx, fruit in enumerate(fruits):
    print(idx, fruit)
# 0 apple → 1 banana → 2 coconut

# 시작 인덱스 지정
for idx, fruit in enumerate(fruits, start=1):
    print(idx, fruit)
# 1 apple → 2 banana → 3 coconut
```

> 💡 강사님: *"enumerate는 꼭 써야 한다. 쓰임새가 정말 많은 중요한 내장함수."*

---

## 🚀 복습 및 AI 인사이트

### ✅ 핵심 체크포인트
* **import 권장 방식:** `import 모듈명` → `모듈명.함수명()` 형태 (출처 명확, 충돌 방지)
* **모듈 vs 패키지:** 모듈 = `.py` 파일 / 패키지 = 폴더(관련 모듈 묶음)
* **`pip`:** 외부 패키지 설치 도구, 파이썬 설치 시 함께 제공, `$` 기호는 터미널 입력 의미
* **의존성:** 패키지 실행에 필요한 다른 패키지 → pip가 자동 설치
* **if/elif/else:** 위에서부터 순차 검사, 참인 블록 하나만 실행, 나머지 건너뜀
* **for문 스코프:** for문은 지역 스코프 생성 안 함 → 루프 변수가 for문 밖에서도 유효 (파이썬 특성)
* **for-else:** break 없이 순회 완료 시 else 실행 → flag 변수 없이 "못 찾음" 처리
* **range + len:** 인덱스로 리스트 요소 수정할 때 사용 (`for i in range(len(list))`)
* **break vs continue:** break = 반복문 종료 / continue = 현재 회차 건너뜀, 다음 회차 진행
* **break는 for/while 탈출:** if문 탈출이 아님 — 가장 가까운 반복문 탈출
* **while 종료 조건 필수:** 종료 조건 누락 → 무한루프 → 서비스 장애 위험
* **중첩 반복문 복잡도:** 이중 N×M, 3중 이상은 성능 이슈 주의
* **리스트 컴프리헨션:** `[표현식 for 변수 in 이터러블 if 조건]` — 메모리·속도 우수

### 🤖 AI 활용 팁
* **프롬프트 예시 (for문 흐름 추적):** `"Python 중첩 for문에서 outers=['A','B'], inners=['C','D']일 때 각 반복 단계별로 변수값이 어떻게 바뀌는지 단계별로 표로 보여줘"`
* **프롬프트 예시 (for-else):** `"Python for-else 구문이 실행되는 조건과 실행되지 않는 조건을 break 유무로 구분해서 예시 코드와 함께 설명해줘"`
* **프롬프트 예시 (while vs for):** `"Python에서 for문과 while문의 사용 기준을 상황별로 정리해줘. 각각 적합한 케이스와 코드 예시 포함"`
* **프롬프트 예시 (리스트 컴프리헨션 변환):** `"다음 for문을 리스트 컴프리헨션으로 바꿔줘: [코드 붙여넣기]. 변환 전후 가독성 차이도 설명해줘"`