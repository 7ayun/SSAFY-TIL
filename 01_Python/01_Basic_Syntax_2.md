# Day04 - 1월 27일 수업 정리

---

# Part 1. 문자열 메서드 (String Methods)

문자열 메서드는 **문자열을 가공, 변환, 분리, 결합**하기 위해 사용하는 함수들로,
입력 처리, 데이터 정제, 출력 포맷팅에서 매우 핵심적인 역할을 한다.

---

## 1. find / index

### 개념
- 문자열 내부에서 **특정 문자 또는 문자열의 위치(인덱스)를 탐색**

```python
s = "hello"
s.find('e')   # 1
s.index('e')  # 1
```

| 메서드 | 특징 |
|----------|--------|
| find | 없으면 -1 반환 (안전) |
| index | 없으면 오류 발생 (엄격) |

👉 **존재 여부만 확인 → find / 반드시 존재해야 → index**

---

## 2. replace

### 개념
- 문자열 내부의 **특정 부분을 다른 값으로 치환**

```python
"hello".replace("h", "H")  # Hello
```

### 특징
- 문자열은 **immutable(불변)** → 항상 **새 문자열 생성**

---

## 3. split ⭐

### 개념
- 문자열을 **구분자를 기준으로 분리 → 리스트로 변환**

```python
"10 20 30".split()  # ['10','20','30']
```

### 실전 활용

```python
nums = list(map(int, input().split()))
```

👉 **알고리즘 입력 처리 핵심 공식**

---

## 4. join ⭐

### 개념
- 리스트 요소들을 **구분자로 연결 → 하나의 문자열 생성**

```python
" ".join(['hello','world'])  # 'hello world'
```

👉 **출력 형식 맞추기 핵심**

---

## 5. strip / lstrip / rstrip

### 개념
- 문자열 **양쪽 / 왼쪽 / 오른쪽 공백 제거**

```python
"  hi  ".strip()   # "hi"
```

---

# Part 2. 리스트 메서드 (List Methods)

## 6. append

- **리스트 맨 뒤에 요소 하나 추가**

```python
lst.append(x)
```

⚠️ **원본 변경, 반환값 없음(None)**

---

## 7. extend ⭐

- iterable 내부 **요소들을 풀어서 추가**

```python
lst.extend([1,2,3])
```

### append vs extend

| 구분 | append | extend |
|--------|----------|----------|
| 추가 방식 | 객체 자체 | 요소 풀어서 |
| 결과 | [1,2,[3,4]] | [1,2,3,4] |

👉 **실습 & 시험 최다 실수 구간**

---

## 8. insert

```python
lst.insert(1, 100)
```

- **중간 삽입 → 성능 저하 → 알고리즘 사용 지양**

---

## 9. pop

```python
x = lst.pop()   # 마지막 요소 제거 + 반환
y = lst.pop(2)  # 특정 인덱스 제거 + 반환
```

👉 **스택 / 큐 구조 핵심 연산**

---

## 10. remove

```python
lst.remove(3)
```

- **값 기준 삭제**

---

## 11. sort / reverse

```python
lst.sort()      # 원본 변경
lst.reverse()   # 뒤집기
```

⚠️ **sort()는 반환값 없음 → 체이닝 불가**

---

# Part 3. 얕은 복사 vs 깊은 복사 ⭐⭐⭐

## 얕은 복사 (Shallow Copy)

```python
b = a[:]
b = list(a)
b = a.copy()
```

- **겉만 복사 / 내부 가변 객체 공유**

### 문제 예시

```python
a = [[1,2],[3,4]]
b = a.copy()
b[0][0] = 100
print(a)   # [[100,2],[3,4]]
```

---

## 깊은 복사 (Deep Copy)

```python
import copy
b = copy.deepcopy(a)
```

- **중첩 구조까지 완전 복사 → 안전**

---

# Part 4. 딕셔너리 메서드 (Dictionary Methods) ⭐⭐⭐

딕셔너리는 **키(key)로 값(value)에 바로 접근**하는 자료구조입니다.
리스트처럼 순서(인덱스)로 찾지 않기 때문에, 많은 상황에서 **탐색/추가/삭제가 빠르게** 동작합니다.

---

## 13. clear

### 기능
- 딕셔너리의 **모든 키/값을 한 번에 삭제(초기화)**

```python
d = {'a': 1, 'b': 2}
d.clear()
print(d)  # {}
```

---

## 14. get ⭐

### 기능
- 키가 없을 때도 **에러 없이 안전하게 값 조회**

```python
person = {'name': 'Alice', 'age': 25}
print(person.get('name'))          # Alice
print(person.get('country'))       # None
print(person.get('country', 'KR')) # KR
```

### 언제 get을 쓰나?
- **키가 없을 수도 있는 상황**에서 안전하게 처리하고 싶을 때

### 언제 []를 쓰나?
- **반드시 존재해야 하는 키**라서, 없으면 즉시 버그를 잡고 싶을 때

```python
# 키가 없으면 KeyError → 로직 이상을 빨리 감지
username = person['username']
```

---

## 15. keys / values / items ⭐

### 기능
- `keys()`   : 키들만 모아둔 **뷰(view)** 반환
- `values()` : 값들만 모아둔 **뷰(view)** 반환
- `items()`  : (키, 값) 쌍을 모아둔 **뷰(view)** 반환

```python
person = {'name': 'Alice', 'age': 25}

for k in person.keys():
    print(k)

for v in person.values():
    print(v)

for k, v in person.items():
    print(k, v)
```

### ⚠️ 중요: dict_keys / dict_values / dict_items는 “복사본”이 아니라 뷰
- 뷰는 딕셔너리의 변경이 **반영**될 수 있습니다.
- 키 목록을 “고정”해서 쓰고 싶으면 list로 감싸서 복사해 두는 것이 안전합니다.

```python
keys_view = person.keys()
person['hello'] = 'world'
# keys_view에 'hello'가 보일 수 있음

keys_list = list(person.keys())
```

---

## 16. pop

### 기능
- **키를 삭제하면서 해당 값을 반환**

```python
person = {'name': 'Alice', 'age': 25}
age = person.pop('age')
print(age)    # 25
print(person) # {'name': 'Alice'}
```

⚠️ 없는 키를 pop하면 KeyError가 날 수 있으므로, 필요하면 기본값을 함께 사용합니다.

```python
x = person.pop('country', None)
```

---

## 17. setdefault

### 기능
- 키가 없으면 **(키, 기본값)을 먼저 넣어두고**, 그 값을 반환

```python
person = {'name': 'Alice'}
country = person.setdefault('country', 'KR')
print(country)  # KR
print(person)   # {'name': 'Alice', 'country': 'KR'}
```

---

## 18. update

### 기능
- 다른 딕셔너리(또는 키=값)로 **덮어쓰기/추가**

```python
person = {'name': 'Alice', 'age': 25}
person.update({'name': 'Jane'})
print(person)  # {'name': 'Jane', 'age': 25}
```

---

# Part 5. Set 메서드 (Set Methods) ⭐⭐⭐

세트(set)는 **중복이 없고 순서가 없는** 자료구조입니다.
- 중복 제거
- 포함 여부 확인
- 집합 연산
에서 매우 자주 사용합니다.

---

## 19. add

### 기능
- 원소 1개 추가

```python
s = {1, 2, 3}
s.add(4)
print(s)  # {1, 2, 3, 4} (출력 순서는 보장되지 않음)
```

⚠️ 같은 값을 add해도 중복이므로 변화 없음

---

## 20. remove / discard

### 기능
- `remove(x)` : x가 없으면 **KeyError**
- `discard(x)`: x가 없어도 **조용히 무시**

```python
s = {1, 2, 3}
s.remove(2)   # OK
# s.remove(9) # KeyError
s.discard(9)  # OK
```

---

## 21. pop

### 기능
- 세트는 순서가 없으므로, `pop()`은 **임의의 원소를 제거하고 반환**

```python
s = {1, 2, 3}
x = s.pop()
```

⚠️ “랜덤”처럼 보이지만, 내부 구현(해시 구조)에 의해 선택됩니다.

---

## 22. update

### 기능
- 다른 iterable의 요소들을 **펼쳐서 추가** (리스트의 extend 느낌)

```python
s = {1, 2, 3}
s.update([3, 4, 5])
print(s)  # {1, 2, 3, 4, 5}
```

---

## 23. 집합 연산

```python
A | B   # 합집합
A & B   # 교집합
A - B   # 차집합
```

---

# Part 6. 메서드 체이닝 (Method Chaining) ⭐⭐⭐

메서드 체이닝은 **여러 메서드를 연속으로 호출**해 코드를 간결하게 만드는 방식입니다.

```python
text = "  HeLLo "
result = text.strip().swapcase().replace('h', 'Z')
```

### 가능한 이유
- 앞 메서드의 결과가 **다음 메서드를 호출할 수 있는 객체(반환값)** 이기 때문입니다.

---

## 24. ⚠️ 체이닝이 깨지는 대표 케이스: None 반환 메서드

리스트 메서드 중 `append`, `sort` 같은 것들은 **원본을 바꾸고 None을 반환**합니다.
그래서 체이닝이 안 됩니다.

```python
nums = [3, 1, 2]
# nums.copy().sort()  # sort()가 None을 반환 → 여기서 끊김
```

### 정렬은 이렇게 구분

- `list.sort()`  : **원본 변경**, 반환값 None
- `sorted(list)` : **새 리스트 반환**

```python
nums = [3, 1, 2]
print(sorted(nums))  # [1, 2, 3]
print(nums)          # [3, 1, 2]

nums.sort()
print(nums)          # [1, 2, 3]
```

---

# Part 7. 해시 테이블(Hash Table) 맛보기 (왜 dict/set이 빠른가)

오늘 단계의 목표는 “완전한 구현”이 아니라,
**딕셔너리/세트가 왜 빠른지**와 **키 제약이 왜 있는지**를 이해하는 것입니다.

## 25. 핵심 아이디어

- dict/set은 내부적으로 **해시(hash) 기반 구조**를 사용합니다.
- 해시를 통해 “값이 들어갈 위치”를 빠르게 계산하여
  평균적으로 **빠른 탐색/추가/삭제**를 제공합니다.

## 26. 왜 dict의 키는 불변(immutable)이어야 하나?

- 키가 바뀌면 해시값이 바뀌어 **찾을 위치가 달라져 버립니다.**
- 그래서 키는 일반적으로 `int`, `float`, `str`, `tuple` 같은 **불변 타입**이어야 합니다.

---

# Part 8. Day04 핵심 요약

- 입력 처리 핵심 → **split**
- 출력 처리 핵심 → **join**
- 리스트 추가 → **append vs extend 차이 완전 이해**
- 딕셔너리 안전 접근 → **get**
- 딕셔너리 순회 핵심 → **items**
- 세트 안전 삭제 → **discard**
- 체이닝 주의 → **None 반환 메서드(append/sort)**
- dict/set이 빠른 이유 → **해시 기반**

---

### 🔗 이동

- 📝 복습 노트: [review.md](./review.md)
- ⬆ 메인: [README](../README.md)

