# [Python] 데이터 구조 (Data Structure)
> **핵심 키워드:** #Python #데이터구조 #문자열메서드 #리스트메서드 #딕셔너리메서드 #세트메서드 #해시테이블 #얕은복사 #깊은복사

---

## 🎯 학습 목표
* 메서드의 정의와 호출 방식 이해 (`객체.메서드()` 형태)
* 문자열 조회 메서드 `find()` vs `index()` 차이 및 적절한 선택 기준 파악
* 문자열 조작 핵심 메서드(`replace`, `strip`, `split`, `join`) 활용
* 리스트 메서드(`append`, `extend`, `insert`, `remove`, `pop`, `sort`) 특성 및 반환값 구분
* 딕셔너리 메서드(`get`, `keys`, `values`, `items`, `update`, `pop`) 안전한 접근 방법 숙지
* 세트 메서드 및 집합 연산자 활용
* 해시 테이블 구조 기반 `dict`/`set` 탐색 속도 우위(O(1)) 이해
* 얕은 복사(`copy`, `[:]`) vs 깊은 복사(`copy.deepcopy`) 차이 명확히 구분

---

## 💡 주요 개념 정리

### 1. 메서드(Method)란?
* **특정 객체에 속한 함수** — 해당 객체의 상태를 조작하거나 동작 수행
* 호출 방식: `데이터.메서드명(인자)`
* 파이썬의 모든 데이터(문자열, 리스트, 딕셔너리 등) → 객체(Object)
* **데이터 타입별 고유한 메서드 존재** → 다른 타입의 메서드 공유 불가

```python
# 객체.메서드() 호출 기본 구조
"hello".upper()       # str 객체의 메서드
.append(4)   # list 객체의 메서드[2][3][1]
```

> 💡 강사님: *"메서드를 전부 외우는 게 목표가 아니라 '이런 게 있구나'를 인지하는 게 목표. 나중에 필요할 때 검색해서 쓸 수 있으면 충분. 존재조차 모르면 직접 알고리즘으로 구현하는 낭비 발생."*

---

### 2. 자료구조 개요 (CS 기초)
```
자료구조(Data Structure)
├── 단순 구조: 정수, 실수, 문자, 문자열
├── 선형 구조: 리스트(배열), 연결 리스트, 스택, 큐
└── 비선형 구조: 트리, 그래프
```
* **파이썬 파트**: 정수·실수·문자열·리스트·딕셔너리·세트·튜플 학습
* **알고리즘 파트**: 연결 리스트·스택·큐·트리·그래프 학습 예정

---

## 💻 기능 구현 및 코드 실습

### 🔧 문자열 메서드 — 조회 및 탐색

#### `find(x)` vs `index(x)`

| 메서드 | 없을 때 반환 | 사용 케이스 |
|---|---|---|
| `find(x)` | `-1` 반환 | 값이 없어도 **무방**한 경우 |
| `index(x)` | `ValueError` 발생 | 값이 **반드시 존재**해야 하는 경우 |

```python
text = "banana"

# find — 값 없으면 -1 반환, 에러 없음
print(text.find("a"))   # 1 (첫 번째 위치, 0-indexed)
print(text.find("z"))   # -1 (없으면 -1 반환)
print(text.find("an"))  # 1 (문자열도 탐색 가능)

# index — 값 없으면 ValueError 발생
print(text.index("a"))  # 1
print(text.index("z"))  # ValueError: substring not found
```

> 💡 강사님: *"어떤 게 더 좋냐 흑백 논리로 생각하면 안 됨. 상황에 따라 다름. 예: 전화번호부에서 '010'이 반드시 있어야 하는데 없으면 → `index`로 에러 발생시켜 개발자에게 알려야 함. 그냥 `-1` 반환하고 넘어가면 잘못된 데이터가 DB에 쌓이는 스노우볼 사태 발생."*

#### 문자열 검증 메서드

```python
s1 = "HELLO"
s2 = "Hello"
s3 = "hello"

print(s1.isupper())   # True  — 모두 대문자인지 확인
print(s3.islower())   # True  — 모두 소문자인지 확인
print(s2.isupper())   # False — H만 대문자이므로 False
print(s2.islower())   # False

print("hello".isalpha())   # True  — 알파벳으로만 구성 확인
print("hello1".isalpha())  # False — 숫자 포함 시 False
```

---

### 🔧 문자열 메서드 — 조작 ⭐

#### `replace(old, new[, count])`

```python
text = "hello world world world"

# 기본: 모든 해당 문자열을 교체
new_text = text.replace("world", "Python")
print(new_text)   # "hello Python Python Python"
print(text)       # "hello world world world" — 원본 유지 (불변)

# count 인자: 교체 횟수 제한
new_text2 = text.replace("world", "Python", 1)
print(new_text2)  # "hello Python world world" — 1개만 교체
```

> ⚠️ **문자열 불변(Immutable)의 핵심**: `replace`는 원본 문자열을 변경하지 않고, 교체된 **새 문자열을 반환**. 반드시 변수에 할당 후 사용.

#### `strip()` / `lstrip()` / `rstrip()`

```python
text = "  hello world  "

print(text.strip())   # "hello world"  — 양쪽 공백 제거
print(text.lstrip())  # "hello world  " — 왼쪽만 제거
print(text.rstrip())  # "  hello world" — 오른쪽만 제거

# 지정 문자 제거
text2 = "...hello..."
print(text2.strip("."))  # "hello" — 지정 문자 제거
```

> 💡 강사님: *"웹 서비스 로그인 시 이메일 뒤에 공백 입력해도 로그인 되는 이유가 서버에서 `strip()` 적용하기 때문. 알고리즘 문제 풀 때 코드가 완벽한데 안 풀리면 입력값에 `strip()` 적용해볼 것. 출제자 실수로 공백이 들어간 경우 있음 — 직접 경험."*

#### `split([sep])` ⭐ 알고리즘 필수

```python
# 공백 기준 분리 (알고리즘 입력값 처리 시 표준 패턴)
text = "10 20 30 40 50"
numbers = text.split()      # ['10', '20', '30', '40', '50']

# 지정 구분자 기준 분리
csv = "hello,world,python"
words = csv.split(",")       # ['hello', 'world', 'python']
```

> 💡 강사님: *"알고리즘 사이트의 입력값은 전부 문자열(텍스트 파일)로 제공됨. '10 20 30' 같은 문자열을 for문으로 순회하면 '1', '0', ' ', '2'... 처럼 한 글자씩 돌아감. `split()`으로 리스트로 변환 후 처리해야 함 — 알고리즘 매 문제 무조건 사용."*

#### `join(iterable)` ⭐

```python
words = ["hello", "world", "python"]

# 구분자.join(이터러블) — 이터러블을 하나의 문자열로 결합
result = "-".join(words)     # "hello-world-python"
result2 = " ".join(words)    # "hello world python"
result3 = "".join(words)     # "helloworldpython"
```

#### `split()` + `join()` 콜라보 패턴 ⭐

```python
# 문자열에서 특정 단어 교체 패턴 (불변인 문자열을 간접적으로 조작)
sentence = "안녕 나는 공부하고 싶어"

words = sentence.split()          # ['안녕', '나는', '공부하고', '싶어']
words = "너는"                  # 리스트는 가변 → 직접 수정 가능[3]
new_sentence = " ".join(words)    # "안녕 너는 공부하고 싶어"

print(sentence)       # "안녕 나는 공부하고 싶어" — 원본 유지
print(new_sentence)   # "안녕 너는 공부하고 싶어"
```

> 💡 강사님: *"`replace()`로 못 처리하는 상황(토큰 단위로 분리 후 특정 위치만 수정, 불필요한 다중 공백 제거 등)에서 `split()` + `join()` 조합 활용. 실무에서도 굉장히 많이 사용."*

#### 대소문자 변환 메서드

```python
text = "hello WORLD"

print(text.capitalize())  # "Hello world"  — 첫 글자만 대문자, 나머지 소문자
print(text.title())       # "Hello World"  — 공백 기준 각 단어 첫 글자 대문자
print(text.upper())       # "HELLO WORLD"  — 전체 대문자
print(text.lower())       # "hello world"  — 전체 소문자
print(text.swapcase())    # "HELLO world"  — 대소문자 반전
```

---

### 🔧 리스트 메서드 — 추가

#### `append(x)` vs `extend(iterable)` ⭐

```python
my_list =[1][2][3]

# append — 단일 항목을 마지막에 추가 (어떤 타입이든 가능)
my_list.append(4)          #[2][3][1]
my_list.append()     # [1, 2, 3, 4, ] — 리스트 자체가 하나의 항목으로 삽입

my_list2 =[3][1][2]
# extend — 이터러블의 각 항목을 풀어서 추가 (정수 등 비이터러블 불가)
my_list2.extend() #  — 개별 항목으로 풀려서 삽입[1][2][3]
my_list2.extend("안녕")    # [1, 2, 3, 4, 5, 6, '안', '녕'] — 문자열도 한 글자씩 삽입
```

> ⚠️ **가변(Mutable) 리스트 핵심**: `append()` 반환값 = `None`. 원본을 직접 수정하므로 `new_list = my_list.append(4)` 형태 사용 금지.

#### `insert(idx, x)`

```python
my_list =[2][3][1]

# insert — 지정 인덱스 위치에 항목 삽입
my_list.insert(1, 5)   #  — 인덱스 1 위치에 5 삽입[3][1][2]

# insert(0, x) — 맨 앞에 삽입
my_list.insert(0, 10)  #[1][2][3]
```

> ⚠️ 강사님: *"리스트는 연속된 메모리 공간 사용. 중간 `insert` 시 뒤에 있는 모든 요소를 한 칸씩 뒤로 재배치해야 함 → O(n) 비용 발생. 데이터 많을수록 느려짐. 알고리즘에서 성능 고려 필요."*

---

### 🔧 리스트 메서드 — 삭제

```python
my_list =[2][3][1]

# remove(x) — 첫 번째 x 항목 제거 (없으면 ValueError)
my_list.remove(2)    #  — 첫 번째 2만 제거[3][1][2]

# pop(idx) — 인덱스 항목 제거 후 반환 (기본값: 마지막 요소)
val = my_list.pop()      # val = 4, my_list =[1][2][3]
val2 = my_list.pop(0)    # val2 = 1, my_list =[2][1]
```

---

### 🔧 리스트 메서드 — 기타

```python
my_list =[3][1][2]

# count(x) — x의 등장 횟수 반환
print(my_list.count(1))   # 2
print(my_list.count(5))   # 2

# index(x) — x의 첫 번째 인덱스 반환 (없으면 ValueError)
print(my_list.index(4))   # 2

# sort() — 원본을 오름차순 정렬 (반환값 None)
my_list.sort()
print(my_list)   #[1][2][3]

my_list.sort(reverse=True)
print(my_list)   #[2][3][1]
```

> 💡 강사님: *"`sort()`와 `sorted()` 혼동 주의. `sort()`는 원본 리스트를 직접 정렬하고 반환값은 `None`. `sorted()`는 정렬된 **새 리스트를 반환**하고 원본 유지. `result = my_list.sort()` 형태 작성 금지."*

---

### 🔧 딕셔너리 메서드 ⭐

#### `get(key[, default])`

```python
person = {"name": "Alice", "age": 25}

# 일반 접근 — 없는 키 접근 시 KeyError 발생
print(person["name"])    # "Alice"
# print(person["country"])  → KeyError!

# get() — 없는 키 접근 시 None 반환 (에러 없음)
print(person.get("name"))      # "Alice"
print(person.get("country"))   # None (기본값)
print(person.get("country", "unknown"))  # "unknown" (기본값 지정)
```

> 💡 강사님: *"실무에서 `get()`은 정말 많이 씀. 키가 있는지 없는지 불확실한 상황에서 안전하게 접근 가능. 없으면 `None` 반환하므로 `if`문으로 처리하기도 용이."*

#### `keys()` / `values()` / `items()`

```python
person = {"name": "Alice", "age": 25, "country": "Korea"}

# keys() — 키 목록 반환 (dict_keys 객체)
for key in person.keys():
    print(key)   # name, age, country

# values() — 값 목록 반환 (dict_values 객체)
for val in person.values():
    print(val)   # Alice, 25, Korea

# items() — (키, 값) 튜플 쌍 반환 (dict_items 객체) ⭐ 가장 많이 활용
for key, value in person.items():
    print(key, value)   # name Alice / age 25 / country Korea
```

#### `update(other_dict)` / `pop(key)`

```python
person = {"name": "Alice", "age": 25}

# update() — 딕셔너리 병합 (기존 키는 덮어쓰기)
person.update({"age": 30, "country": "Korea"})
print(person)   # {"name": "Alice", "age": 30, "country": "Korea"}

# pop(key) — 키 제거 후 해당 값 반환
removed = person.pop("age")
print(removed)  # 30
print(person)   # {"name": "Alice", "country": "Korea"}
```

---

### 🔧 세트 메서드

```python
my_set = {1, 2, 3}

# add(x) — 단일 요소 추가 (중복 시 무시)
my_set.add(4)    # {1, 2, 3, 4}
my_set.add(2)    # {1, 2, 3, 4} — 중복 무시

# remove(x) — 요소 제거 (없으면 KeyError)
my_set.remove(2)  # {1, 3, 4}

# discard(x) — 요소 제거 (없어도 에러 없음)
my_set.discard(99)  # {1, 3, 4} — 에러 없음

# pop() — 임의 요소 제거 후 반환 (실제로는 해시 테이블 입력 순 반환)
val = my_set.pop()

# update(iterable) — 이터러블 요소 전체 추가 (extend의 세트 버전)
my_set.update()

# clear() — 전체 초기화
my_set.clear()   # set()
```

#### 집합 연산

```python
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

print(a | b)    # {1, 2, 3, 4, 5, 6} — 합집합
print(a & b)    # {3, 4}             — 교집합
print(a - b)    # {1, 2}             — 차집합
print(a ^ b)    # {1, 2, 5, 6}       — 대칭 차집합

# 메서드 방식
print(a.union(b))        # 합집합
print(a.intersection(b)) # 교집합
print(a.difference(b))   # 차집합
```

> 💡 강사님: *"세트는 알고리즘에서 속도가 필요할 때 핵심 자료구조. `add()`·`remove()` 연산이 O(1)로 매우 빠름. 리스트는 특정 요소 포함 여부 확인 시 최악 O(n) — 데이터 1000개면 1000번 탐색 필요."*

---

### 🔧 해시 테이블 & 해시어블 (Hashable)

#### 핵심 개념

* **해시 테이블**: `dict`·`set`의 내부 저장 구조 — 키를 해시값으로 변환 후 저장
* **O(1) 탐색**: 해시값으로 직접 접근 → 데이터 크기와 무관하게 빠른 속도
* **해시어블(Hashable)**: 해시값으로 변환 가능한 객체 = **불변형 타입**
  * 해시어블: `int`, `float`, `str`, `tuple` (내부에 가변형 미포함 시)
  * 해시 불가: `list`, `dict`, `set` (가변형)
* **딕셔너리 키 조건**: 반드시 해시어블한 값만 사용 가능

```python
# 해시어블 타입은 dict 키로 사용 가능
d = {
    "name": "Alice",      # str → 해시어블 ✅
    1: "one",             # int → 해시어블 ✅
    (1, 2): "tuple key",  # tuple (불변 요소만) → 해시어블 ✅
}

# 가변형은 키로 사용 불가
# d[] = "list key"   → TypeError: unhashable type: 'list'[3][2]
# d[{1, 2}] = "set key"    → TypeError: unhashable type: 'set'

# 튜플 내부에 가변형 포함 시 해시어블 불가
# d[(, 3)] = "val"  → TypeError[2][3]
```

> 💡 강사님: *"세트·딕셔너리가 속도가 빠른 이유는 해시 테이블 구조 때문. 리스트는 1·2·3... 하나하나 비교 → O(n). 세트·딕셔너리는 해시값으로 바로 찾아 들어감 → O(1). 알고리즘에서 탐색 빈도 높으면 세트·딕셔너리로 변환 고려."*

---

### 🔧 얕은 복사 vs 깊은 복사

```python
import copy

# 단순 할당 — 참조(Reference)만 복사, 같은 객체 가리킴
a =[1][3][2]
b = a          # b는 a와 동일한 메모리 주소 참조
b = 100
print(a)       #  — a도 변경됨[1][2]

# ── 얕은 복사 (Shallow Copy) ──────────────────────────
a = [1, 2, ][1]

b = a.copy()   # 방법 1: .copy() 메서드
c = a[:]       # 방법 2: 슬라이싱

b = 100
print(a)       # [1, 2, ] — 1단계 요소는 독립 복사[1]

b = 999  # 중첩 리스트 수정[2]
print(a)       # [1, 2, ] — 중첩 객체는 여전히 같은 참조! ⚠️

# ── 깊은 복사 (Deep Copy) ────────────────────────────
a = [1, 2, ][1]
d = copy.deepcopy(a)  # 중첩 구조까지 완전히 새로운 객체로 복사

d = 999[2]
print(a)       # [1, 2, ] — 원본 영향 없음 ✅[1]
```

> 💡 강사님: *"얕은 복사는 1단계만 독립 복사. 안에 리스트·딕셔너리 같은 가변 객체가 있으면 여전히 같은 주소를 가리킴. 중첩 구조를 완전히 분리하려면 `copy.deepcopy()` 필수."*

---

## 🚀 복습 및 AI 인사이트

### ✅ 핵심 체크포인트
* **메서드 호출**: `데이터.메서드()` 형태 — 해당 객체에만 사용 가능
* **find vs index**: `find` = 없으면 `-1`, `index` = 없으면 `ValueError` — 상황별 선택
* **불변 문자열 조작**: `replace`·`split` 등은 새 문자열 반환 → 반드시 변수에 할당
* **append vs extend**: `append` = 항목 그대로 삽입 / `extend` = 이터러블 풀어서 삽입
* **가변 리스트 메서드 반환값**: `append`·`extend`·`sort` 반환값 = `None` → 재할당 금지
* **sort vs sorted**: `sort()` = 원본 수정·반환 None / `sorted()` = 새 리스트 반환·원본 유지
* **dict.get()**: 없는 키 접근 시 KeyError 방지 — 기본값 지정 가능
* **dict.items()**: `for key, value in dict.items()` 패턴 — 딕셔너리 순회 표준
* **세트 탐색 O(1)**: 해시 테이블 기반 → 리스트 O(n) 대비 속도 우위
* **얕은/깊은 복사**: 중첩 구조 완전 분리 필요 시 `copy.deepcopy()` 사용
* **insert 비용**: 중간 삽입 시 뒤 요소 전체 재배치 → O(n) 비용 주의
* **strip() 알고리즘 팁**: 완벽한 코드인데 오답 시 입력값에 `strip()` 적용 시도

### 🤖 AI 활용 팁
* **프롬프트 예시 (split·join 패턴):** `"Python에서 split()과 join()을 함께 사용하는 패턴 3가지를 실제 알고리즘 입력 처리 예시와 함께 보여줘"`
* **프롬프트 예시 (얕은/깊은 복사 차이):** `"Python 얕은 복사와 깊은 복사의 차이를 중첩 리스트 예시 코드와 메모리 구조 설명으로 보여줘"`
* **프롬프트 예시 (해시 테이블):** `"Python dict와 list의 탐색 속도 차이를 시간 복잡도(Big-O)와 실제 실행 시간 측정 코드로 비교해줘"`
* **프롬프트 예시 (dict.get 활용):** `"Python dict.get()을 활용해서 KeyError 없이 딕셔너리를 안전하게 다루는 패턴을 카운팅 문제 예시와 함께 보여줘"`
