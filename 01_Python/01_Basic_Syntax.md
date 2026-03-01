# [Python] 기본 문법 1 · 2 — 데이터 타입 · 연산자 · 메모리 구조

> **핵심 키워드:** #Python #변수 #데이터타입 #연산자 #메모리 #Truthy #Falsy #부동소수점 #컬렉션

---

## 🎯 학습 목표

* 프로그래밍의 개념과 고수준 / 저수준 언어의 차이 이해
* Python 인터프리터 동작 원리 및 실행 환경 구성 파악
* 변수 할당 시 내부 메모리 주소 참조 방식 이해
* Numeric · Sequence · Non-sequence 자료형 전체 파악
* `==` vs `is` 비교, 단축 평가, 멤버십 연산자 정확히 구분
* Truthy / Falsy 판별 규칙 및 암시적 · 명시적 형변환 적용

---

## 💡 주요 개념 정리

### 1. 프로그래밍이란?

* **프로그램** = 명령어들의 집합
* 컴퓨터는 주어진 명령어를 **그대로** 순서대로 실행 — 임의 판단 없음
* 기초 연산 예시: 더하기, 빼기, 곱하기, 출력 등
* **주의** — 명령이 모호하면 의도와 다른 결과 발생 ("우유 1개 사와. 달걀 있으면 6개 사와." → 우유 6개 구매)

---

### 2. 고수준 vs 저수준 언어

| 구분 | 특징 | 예시 |
|---|---|---|
| **고수준(High-level)** | 사람이 읽기 쉬운 언어 | Python, Java, JavaScript |
| **저수준(Low-level)** | 컴퓨터가 직접 이해하는 언어 | 기계어(0·1), 어셈블리어 |

* 고수준 언어 → 컴퓨터 실행을 위해 **번역 과정** 필수
* Python에서 번역 담당 → **인터프리터(Interpreter)**
* 인터프리터 = 실시간 통역사 역할; 운영체제(Windows · Linux · macOS) 간 이식 가능

---

### 3. Python 소개

* **창시자:** 귀도 반 로썸(Guido van Rossum) — 크리스마스 취미 프로젝트로 제작
* **설계 철학:** "프로그래밍은 쉬워야 한다(Easy)"
* **특징:**
  * 쉽고 간결한 문법
  * 커뮤니티 규모 세계 최대 → 망할 위험 낮음, 레퍼런스 풍부
  * 알고리즘 구현에 유리 (타입 선언 불필요, 핵심 로직 집중 가능)
  * 순열 · 조합 · 우선순위 큐 등 라이브러리 풍부 (`itertools.combinations` 등)
  * **SSAFY 시험:** A형까지 Python 필수, 기업 코딩테스트 시 Python이 가장 유리

* **귀도 반 로썸 타이틀:** "자비로운 종신 독재자(BDFL)" — 파이썬 생태계 논쟁 결정권 보유, 현재는 은퇴

---

### 4. Python 실행 원리 및 환경 변수

* Python 파일 확장자: **`.py`**
* 터미널 실행 단축키: `Ctrl` + `` ` `` (백틱, 숫자 1 왼쪽)
* `python test.py` 입력 시 동작 과정:
  1. OS가 PATH 환경 변수에서 `python` 실행 파일 위치 탐색
  2. 해당 경로의 `python.exe` 실행 → 인터프리터 가동
  3. `.py` 파일 번역 및 실행
* **환경 변수(PATH):** Python 설치 시 "Add to PATH" 체크 필수
  * 미체크 시 → `python` 명령어 미인식 에러 발생
  * 수동 등록 경로: 윈도우키 → "시스템 환경 변수 편집" → PATH 항목에 추가
* Python 버전: **SSAFY 기준 3.11** 필수

---

### 5. 변수와 메모리 — 파이썬만의 핵심 동작

* **변수:** 데이터를 저장하는 메모리 공간의 **별명(alias)**
* 변수명 규칙: 영문 · 숫자 · 언더스코어 조합, 숫자 시작 불가

#### ⚠️ 파이썬 변수 = 주소 저장 (다른 언어와 핵심 차이)
```python
# 일반 언어 (C 등): 변수에 값 자체 저장
# 파이썬: 변수에 "값이 저장된 메모리 주소" 저장

degree = 36.5   # degree 변수 안에는 36.5가 아닌, 36.5가 저장된 주소가 들어감
```

* 이 구조 덕분에 타입 선언 불필요 → 어떤 크기 데이터든 주소(고정 크기)만 저장
* **삭제(del)의 실체:** 값 자체가 지워지는 게 아니라 주소(접근 경로)만 제거 — 복원 회사가 빠르면 복구 가능한 이유

#### 변수 할당 흐름 예시
```python
number = 10       # ① 10을 메모리에 저장 → 주소 982651 부여
                  # ② number 변수에 주소 982651 저장

double = 2 * number   # ③ 20 계산 → 새 주소 부여 후 저장
                      # ④ double에 20의 주소 저장

number = 5        # ⑤ 5를 새 주소에 저장 → number에 새 주소 저장
print(double)     # → 20 출력 (double은 변경 없음, 그대로 20 가리킴)
```

---

### 6. 데이터 타입 (Data Types)

#### 6-1. Numeric 타입

**① int (정수형)**
```python
a = 10       # 10진수 (기본)
b = 0b1010   # 2진수 (Binary) → 10
c = 0o12     # 8진수 (Octal)  → 10
d = 0xA      # 16진수 (Hex)   → 10

# 16진수: 0~9 이후 A(10), B(11), C(12), D(13), E(14), F(15)
```

**② float (실수형 = 부동소수점)**
```python
x = 3.14
y = 3.14e-2   # 지수 표현: 3.14 × 10⁻² = 0.0314

# ⚠️ 부동소수점 에러 — 반드시 알아야 할 함정
print(3.2 - 3.1 == 1.2 - 1.1)   # → False (!)
# 이유: 소수를 2진수(0·1)로 저장 시 무한 반복 → 중간에서 잘라 저장
#       → 미세한 오차 발생 (예: 0.0999...3 ≠ 0.1000...2)

# 해결책: decimal 모듈 사용 (정밀 소수 연산)
from decimal import Decimal
print(Decimal('3.2') - Decimal('3.1'))   # → 0.1 정확 출력
# 단, 트레이드오프: 메모리 사용량 증가 + 속도 저하
```

* **부동소수점 핵심 요약:**
  * 메모리 유한 → 무한 소수 저장 불가 → 잘라서 저장
  * 정밀 계산 필요 시 `decimal` 모듈 활용
  * `float` 구조: 1비트(부호) + 11비트(지수부) + 52비트(가수부) = 64비트

---

#### 6-2. Sequence 타입 — 순서 있는 자료형

> 공통 특징: **순서 존재** → 인덱싱 · 슬라이싱 · 길이(`len()`) 가능

**③ str (문자열) — 불변(Immutable)**
```python
# 선언 방식
s1 = "Hello"
s2 = 'Hello'   # 동일, 따옴표 통일성 유지 권장

# 중첩 따옴표: 문자열 내 따옴표 포함 시
s3 = "I'm a student"   # 내부 ' → 외부 " 사용
s4 = 'He said "Hi"'   # 내부 " → 외부 ' 사용

# Escape Sequence (외울 필요 없음, 필요 시 참조)
s5 = "Hello\nWorld"    # \n: 개행 (줄바꿈)
s6 = "Tab\there"       # \t: 탭 삽입
s7 = "Back\\slash"     # \\: 백슬래시 문자 자체 출력
s8 = "It\'s OK"        # \': 작은따옴표 출력 (동일 따옴표 내부)

# f-string (Python 3.6+) — 문자열 내 변수 삽입
name = "Jaden"
count = 13
print(f"안녕하세요 {name}님, 접속 횟수: {count}")   # → 안녕하세요 Jaden님, 접속 횟수: 13

# 인덱싱: 컴퓨터는 0부터 카운트
word = "Hello"
print(word[0])    # → H (인덱스 0)
print(word[-1])   # → o (뒤에서 첫 번째)

# 슬라이싱: [시작:끝:간격] — 시작 포함, 끝 미포함
print(word[1:4])    # → ell
print(word[::2])    # → Hlo (2칸 간격)
print(word[::-1])   # → olleH (역순)

# ⚠️ 불변 특성: 인덱스로 부분 수정 불가
word[0] = 'h'   # → TypeError 발생!
# 단, 전체 재할당은 가능: word = "hello"
```

**④ list (리스트) — 가변(Mutable)**
```python
my_list = [1, "Hello", 3.14, True, [4, 5]]   # 다양한 자료형 혼합 저장 가능

# 인덱싱 · 슬라이싱: str과 동일
print(my_list[0])      # → 1
print(my_list[-1][1])  # → 5 (중첩 리스트 접근)

# 가변 특성: 인덱스로 값 수정 가능
my_list[0] = 100       # → [100, "Hello", 3.14, True, [4, 5]]

# 리스트 내부 메모리 구조
# ┌───────┬───────┬───────┐
# │ 주소1  │ 주소2 │  주소3 │  ← 실제 값이 아닌 각 원소의 주소 저장
# └───────┴───────┴───────┘
# 인덱스 접근: 시작주소 + (인덱스 × 8바이트) 계산 → O(1) 속도

# ⚠️ 알고리즘 지옥 함정: 리스트 복사는 주소 복사
c = [7, 8, 9]
d = c             # d는 c와 같은 주소(같은 객체)를 가리킴
c[0] = 10
print(d)          # → [10, 8, 9] (d도 변경됨!)
# 해결: 깊은 복사 필요 (추후 학습)
```

**⑤ tuple (튜플) — 불변(Immutable)**
```python
# 선언: 소괄호 또는 쉼표만으로 가능 (쉼표가 본체!)
t1 = (1, 2, 3)
t2 = 1, 2, 3      # 소괄호 없이도 튜플 생성
t3 = (1,)         # ⚠️ 단일 원소 튜플: 반드시 후행 쉼표 필요
t4 = (1)          # → int 타입 (튜플 아님!)
t5 = ()           # 빈 튜플: 소괄호 필수

# ⚠️ 핵심: 콤마가 튜플의 본체, 소괄호는 단순 그룹화 도구

# 인덱스 수정 불가
t1[0] = 100   # → TypeError: 'tuple' object does not support item assignment

# 주요 활용
# ① 불변 데이터 보관 (RGB 색상, 좌표 등 변경 불필요한 값)
color = (255, 128, 0)
point = (37.5665, 126.9780)   # 서울 좌표

# ② 다중 할당
x, y = 10, 20     # x=10, y=20

# ③ 값 교환 (파이썬만의 강점 — C언어는 temp 변수 필요)
x, y = y, x       # 내부적으로 튜플 처리, temp 없이 교환
```

**⑥ range (정수 시퀀스 생성기) — 불변(Immutable)**
```python
# 문법: range(시작, 끝, 간격)
# 규칙: 시작 포함, 끝 미포함 (슬라이싱과 동일!)

range(5)          # → 0, 1, 2, 3, 4 (시작 생략 시 0부터)
range(1, 10)      # → 1, 2, 3, 4, 5, 6, 7, 8, 9
range(2, 4)       # → 2, 3
range(5, 0, -1)   # → 5, 4, 3, 2, 1 (역순)
range(1, 5, -1)   # → 빈 시퀀스 (방향 불일치)
# range(1, 10, 0)  → ValueError: range() arg 3 must not be zero

# for문과 연계 (다음 주 학습)
for i in range(1, 10):
    print(i)   # 1~9 출력
# for문만 잘 써도 IM 통과, A형 통과 가능
```

---

#### 6-3. Non-sequence 타입 — 순서 없는 자료형

**⑦ dict (딕셔너리) — 가변(Mutable), 키 중복 불가**
```python
# 사전(Dictionary) 구조: 키(Key) - 값(Value) 쌍으로 저장
# 키: 변경 불가능한 타입만 사용 가능 (str, int, tuple, range)
# 값: 모든 자료형 사용 가능

my_dict = {
    "apple": 12,           # str 키
    "list": [1, 2, 3],     # value에 list 가능
    "hello": "안녕",
}

# 접근: 키로 직접 호명
print(my_dict["apple"])       # → 12
print(my_dict["list"])        # → [1, 2, 3]

# 추가 / 수정
my_dict["banana"] = 50        # 새 키-값 쌍 추가
my_dict["apple"] = 100        # 기존 키 값 수정

# ⚠️ 없는 키 접근 → KeyError 발생
print(my_dict["jaden"])       # → KeyError: 'jaden'

# ⚠️ 리스트는 키로 사용 불가 (가변이므로)
# my_dict[[1, 2, 3]] = "test"  → TypeError

# 딕셔너리 속도 우위 (리스트 대비)
# 리스트 포함 여부 확인: 최대 N번 순회 필요 → O(N)
# 딕셔너리 키 확인: 해시(Hash)값으로 즉시 접근 → O(1)
```

**⑧ set (세트) — 가변(Mutable), 순서 없음, 중복 없음**
```python
# 수학의 집합과 동일한 구조
my_set = {1, 2, 3, 3, 3}
print(my_set)   # → {1, 2, 3} (중복 자동 제거)

# ⚠️ 출력 순서 비보장: 삽입 순서 일치처럼 보이나 믿으면 안 됨
# 은행 대기열 등 순서 중요한 곳에 set 사용 금지

# 집합 연산
a = {1, 2, 3}
b = {3, 6, 9}
print(a | b)    # 합집합 → {1, 2, 3, 6, 9}
print(a - b)    # 차집합 → {1, 2}
print(a & b)    # 교집합 → {3}

# 알고리즘 활용: 중복 제거
numbers = [1, 2, 2, 3, 3, 3]
unique = set(numbers)   # → {1, 2, 3}
# 포함 여부 확인도 O(1) (딕셔너리와 동일한 해시 기반)
```

---

#### 6-4. None 타입
```python
# 의도적인 값 없음 표현 (null, undefined와 동일 개념)
baby_name = None      # 아직 이름 미결정 상태 (버그 아님, 의도적)
# ≠ "None" (문자열 None과 구분 필수)

baby_name = "아린"    # 값 결정 후 할당
```

---

#### 6-5. bool (불린 타입) — Truthy / Falsy
```python
# 참(True) / 거짓(False) 두 값만 존재
is_member = True
is_adult = False

# ⚠️ Truthy / Falsy 규칙 (자동 형변환)
# False 취급 목록 (외울 필요 없음, 빈 것 & 0이면 False)
# 0, 0.0, "", [], {}, set(), None, ()

# True 취급: 위 목록 외 모든 값
# 정수: 0만 False, 나머지(음수 포함) 전부 True
bool(0)       # → False
bool(1)       # → True
bool(-5)      # → True   ← 음수도 True!
bool(3)       # → True

# 문자열: 빈 문자열만 False
bool("")      # → False
bool("0")     # → True   ← 문자열 "0"은 True!
bool("False") # → True   ← 문자열 "False"도 True!

# 컨테이너: 비어있으면 False
bool([])      # → False
bool([0])     # → True
bool({})      # → False
bool(None)    # → False
```

---

### 7. 컬렉션 비교 정리

| 자료형 | 가변 | 순서 | 중복 | 표기 |
|---|---|---|---|---|
| **str** | ✗ 불변 | ✓ | ✓ | `""` `''` |
| **list** | ✓ 가변 | ✓ | ✓ | `[]` |
| **tuple** | ✗ 불변 | ✓ | ✓ | `()` 또는 쉼표 |
| **range** | ✗ 불변 | ✓ | — | `range()` |
| **dict** | ✓ 가변 | ✗ | 키 ✗ / 값 ✓ | `{key: value}` |
| **set** | ✓ 가변 | ✗ | ✗ | `{}` |

---

### 8. 불변 vs 가변 메모리 원리
```python
# 불변(str): 내부 값 자체 변경 불가
s = "hello"
# s[0] = 'H'  → TypeError
# 이유: "hello" 객체 자체가 메모리에 고정 → 내부 수정 불가
# 변경하려면 → 새 객체 생성 + 재할당

# 가변(list): 내부 주소 교체 가능
lst = ["hello", "a", "b"]
lst[0] = "HELLO"   # OK
# 이유: list는 원소들의 주소를 저장 → "HELLO" 새 객체 생성 후
#       해당 슬롯의 주소만 교체 → 리스트 자체는 그대로

# 핵심 원칙
# - 객체 본연의 값을 직접 수정하는 것: 불가 (str, int, float, tuple 등)
# - 리스트 슬롯의 주소를 바꿔치기하는 것: 가능
```

---

### 9. 타입 변환 (Type Conversion)
```python
# ① 암시적 형변환 (파이썬 자동 처리 — 데이터 손실 없는 방향)
3 + 5.0       # → 8.0 (int + float → float 자동 변환)
True + 3      # → 4   (bool + int → int, True=1)
True + False  # → 1   (1 + 0)

# "hello" + 17  → TypeError (문자열 + 정수: 자동 변환 불가)

# ② 명시적 형변환 (개발자가 직접 지정)
int("3")       # → 3
int(3.9)       # → 3 (소수점 버림)
float("3.14")  # → 3.14
str(17)        # → "17"
bool(0)        # → False

# "hello" + str(17)  → "hello17" (명시적 변환 후 결합)

# ⚠️ 명시적 형변환 불가 케이스
int("3.5")     # → ValueError (정수형으로 직접 변환 불가)
# 해결: int(float("3.5")) → 3

# 강사님 조언: 암시적 형변환에 의존 금지, 명시적 지정이 안전
```

---

### 10. 연산자

#### 10-1. 산술 연산자
```python
a, b = 10, 3

a + b    # → 13  덧셈
a - b    # → 7   뺄셈
a * b    # → 30  곱셈
a / b    # → 3.333... 나눗셈 (float 결과)
a // b   # → 3   몫 (정수 나눗셈)
a % b    # → 1   나머지
a ** b   # → 1000 거듭제곱
```

#### 10-2. 복합 대입 연산자
```python
# a op= b  →  a = a op b 의 축약 표현
a = 10
a += 5    # a = 15
a -= 4    # a = 11
a *= 2    # a = 22
a /= 4    # a = 5.5
a //= 2   # a = 2
a **= 3   # a = 8
# ⚠️ Python에는 a++ 없음

# 코드 철학: 간결함(+=) vs 가독성(a = a + b) — 팀 컨벤션 준수
```

#### 10-3. 비교 연산자 — `==` vs `is`
```python
# == : 값(Value) 비교
# is : 주소(Identity, 객체 동일 여부) 비교

a = [1, 2, 3]
b = [1, 2, 3]   # a와 같은 값이지만 다른 객체
c = a           # a와 동일한 주소 참조

print(a == b)   # → True  (값 같음)
print(a is b)   # → False (다른 주소)
print(a is c)   # → True  (같은 주소)

# 정수 비교 예시
print(1 == True)    # → True  (값 비교: 1 = True)
print(1 is True)    # → False (주소가 다른 객체)
```

#### 10-4. 논리 연산자 (`and`, `or`, `not`)
```python
# and: 양쪽 모두 True여야 True
# or:  둘 중 하나라도 True면 True
# not: 부정 반전

True and False   # → False
True or False    # → True
not True         # → False

# 활용 예시
num = 15
print(num > 10 and num % 2 == 0)   # → False (15는 홀수)

name = "Alice"
age = 25
print(name == "Alice" or age == 30)   # → True
```

#### 10-5. 단축 평가 (Short-circuit Evaluation) — ⚠️ 파이썬 고유 동작
```python
# and: 왼쪽이 False이면 더 볼 필요 없음 → 왼쪽 값 그대로 반환
# or:  왼쪽이 True이면 더 볼 필요 없음  → 왼쪽 값 그대로 반환
# (일반 언어는 True/False 반환, 파이썬은 실제 값 반환)

# and 단축 평가
print(3 and 5)    # → 5   (3=True → 5까지 봐야 함 → 마지막 값 반환)
print(3 and 0)    # → 0   (3=True → 0 확인 → 부정값 0 반환)
print(0 and 5)    # → 0   (첫 번째가 False → 즉시 0 반환, 5 미확인)

# or 단축 평가
print(5 or 0)     # → 5   (5=True → 즉시 5 반환, 0 미확인)
print(0 or 3)     # → 3   (0=False → 3 확인 → 긍정값 3 반환)

# 결론:
# and → False(부정) 만나면 그 값 반환 / 모두 True면 마지막 값 반환
# or  → True(긍정) 만나면 그 값 반환  / 모두 False면 마지막 값 반환

# ⚠️ 강사님 조언: 실무에서 단축 평가 활용 자제 권장
# (다른 언어와 동작 방식 달라 혼란 유발, 가독성 저하)
# 단, 시험에서는 출제 가능 — 규칙으로 이해하고 암기
```

#### 10-6. 멤버십 연산자 (`in`, `not in`)
```python
word = "Hello"
numbers = [1, 2, 3, 4]

print("H" in word)        # → True  (H가 Hello에 포함)
print("Z" in word)        # → False
print(4 not in numbers)   # → False (4가 numbers에 있으므로 not in은 False)
```

#### 10-7. 시퀀스형 연산자
```python
# 문자열/리스트 결합 (+)
print("길동" + "홍")        # → "길동홍"
print([1, 2] + [3, 4])     # → [1, 2, 3, 4]

# 반복 (*)
print("Hi" * 3)            # → "HiHiHi"
print([1, 2] * 3)          # → [1, 2, 1, 2, 1, 2]
# AI 프롬프트 구분선 생성 등에 활용: "-" * 50
```

#### 10-8. 연산자 우선순위

* **헷갈리면 괄호 사용** → 명시적으로 처리 순서 지정
* 암기 불필요, 단 시험 대비 시 별도 확인

---

## 💻 기능 구현 및 코드 실습
```python
# ① 기본 출력 및 타입 확인
print("안녕하세요")           # 화면 출력
print(type(3.14))            # → <class 'float'> 타입 확인

# ② 변수 다중 할당
x, y, z = 1, 2, 3
a = b = c = 0                # 동일 값 다중 변수 할당

# ③ 값 교환 (파이썬 전용)
x, y = y, x

# ④ f-string 활용
user = "ssafy"
score = 95
print(f"사용자: {user}, 점수: {score}점")

# ⑤ 리스트 복사 주의 (얕은 복사 함정)
original = [1, 2, 3]
copy_ref = original      # 주소 복사 → 원본 변경 시 같이 변경
original[0] = 99
print(copy_ref)          # → [99, 2, 3] ← 주의!

# ⑥ 딕셔너리 활용
student = {"name": "홍길동", "age": 20, "major": "CS"}
print(student["name"])   # → 홍길동
student["gpa"] = 4.0     # 새 항목 추가

# ⑦ 단축 평가 실전 예시 (참고용)
value = None
result = value or "기본값"   # None(False)이면 "기본값" 반환
print(result)                # → "기본값"
# 강사님 권장: 명시적 if 조건문 사용이 가독성 우위

# ⑧ 세트로 중복 제거
raw = [1, 2, 2, 3, 3, 3, 4]
unique_sorted = sorted(set(raw))   # → [1, 2, 3, 4]

# ⑨ 진수 변환 확인
print(0b1010)    # → 10 (2진수)
print(0o12)      # → 10 (8진수)
print(0xA)       # → 10 (16진수)
print(hex(255))  # → 0xff (10진수 → 16진수 문자열)

# ⑩ bool 판별 실습
falsy_values = [0, 0.0, "", [], {}, set(), None, ()]
for v in falsy_values:
    print(f"{repr(v):10} → {bool(v)}")   # 모두 False
```

---

## 🚀 복습 및 AI 인사이트

* **핵심 체크포인트:**
  * `==` vs `is` 혼동 금지 — 값 비교 / 주소 비교 명확히 구분
  * 리스트 단순 복사 시 참조(주소) 공유 → 알고리즘에서 예상치 못한 값 변경 발생
  * `tuple` 단일 원소 선언 시 후행 쉼표 필수 (`(1,)` ← `(1)`은 int)
  * 부동소수점 비교 시 `==` 대신 `abs(a - b) < 1e-9` 또는 `decimal` 활용
  * `0`만 Falsy, 음수 포함 모든 0 아닌 정수 → Truthy
  * 딕셔너리 / 세트: 포함 여부 확인이 O(1) → 알고리즘 최적화 핵심 도구
  * 단축 평가는 파이썬 고유 동작 (True/False가 아닌 실제 값 반환)

* **AI 활용 팁:**
  * 부동소수점 에러 재현 및 해결책 탐색: "파이썬에서 3.2 - 3.1 == 1.2 - 1.1이 False인 이유를 IEEE 754 기준으로 설명하고, decimal 모듈로 해결하는 코드를 작성해줘"
  * 가변/불변 메모리 동작 시각화: "파이썬에서 리스트 얕은 복사(shallow copy)와 깊은 복사(deep copy)의 메모리 구조 차이를 id() 함수를 이용해 비교하는 예제 코드를 만들어줘"
  * 단축 평가 트러블슈팅: "파이썬 단축 평가에서 `0 and print('출력')` 실행 시 print가 호출되지 않는 이유를 설명하고, 실수하기 쉬운 패턴 3가지를 알려줘"
  * Falsy 값 완전 목록 확인: "파이썬에서 bool() 적용 시 False로 평가되는 모든 값을 각 타입별로 예시 코드와 함께 정리해줘"