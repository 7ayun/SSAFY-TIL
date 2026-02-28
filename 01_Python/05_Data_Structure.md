# [Python] 데이터 구조 및 메서드 활용 (Data Structures & Methods)

> **핵심 키워드:** #Methods #String #List #Dictionary #Set #Copy #Hash_Table #Hashable

---

## 🎯 학습 목표
* 각 데이터 타입별 고유 메서드 파악 및 조작 능력 배양
* 가변(Mutable) 및 불변(Immutable) 객체의 메모리 참조 방식 이해
* 얕은 복사(Shallow)와 깊은 복사(Deep)의 차이점 및 발생 기전 파악
* 해시 테이블(Hash Table) 구조를 통한 딕셔너리/세트의 고속 탐색 원리 습득

---

## 💡 주요 개념 정리

### 1. 메서드(Method)의 정의
* **객체 고유 함수:** 특정 객체(데이터 타입)에 속해 그 상태를 조작하거나 동작을 수행하는 기능
* **호출 방식:** `객체.메서드()` 형태의 도트(.) 연산자 활용

### 2. 시퀀스 자료형 메서드 (String & List)
* **문자열(String):** 불변 객체 특성상 원본 수정이 아닌 수정된 **새로운 문자열 객체 반환**
* **리스트(List):** 가변 객체로 `append`, `sort` 등 사용 시 원본 데이터 직접 수정 (대부분 반환값 없음)

### 3. 비시퀀스 자료형 메서드 (Dictionary & Set)
* **해시 기반 관리:** 순서가 없으나 해시 함수를 통해 데이터 위치를 즉시 계산하여 O(1) 수준의 탐색 속도 확보
* **키(Key) 제약:** 해시값 생성을 위해 반드시 **불변(Immutable)** 객체만 키로 사용 가능

---

## 💻 기능 구현 및 코드 실습

### 1. 문자열 핵심 조작 기법
데이터 전처리 및 포맷팅에 필수적인 메서드 활용 예시

```python
text = "  hello world  "

# 공백 제거 및 분할
stripped = text.strip()           # 양끝 공백 제거
words = stripped.split()          # 공백 기준 리스트화 (['hello', 'world'])

# 리스트 결합 및 교체
joined = "-".join(words)          # 'hello-world' 반환
replaced = joined.replace("-", " ") # 특정 문자 치환

# 대소문자 반전 (메서드 체이닝)
result = text.strip().upper()     # 'HELLO WORLD' (연속 호출 가능)
```

### 2. 리스트 조작 및 성능 주의사항
데이터 추가/삭제 시 발생하는 메모리 비용 유의 기법

```python
mylist = [1, 2, 3]

# 데이터 추가
mylist.append(4)      # [1, 2, 3, 4] (맨 끝 추가, 효율적)
mylist.extend([5, 6]) # [1, 2, 3, 4, 5, 6] (반복 가능 객체 풀어서 추가)

# 성능 주의 (사용 지양)
mylist.insert(0, 99)  # 맨 앞 삽입 시 뒤 요소 전체 이동 발생 (O(N) 성능 저하)
mylist.pop(0)         # 맨 앞 삭제 시 요소 재배치 비용 발생

# 데이터 탐색 및 정렬
idx = mylist.index(3) # 값 3의 위치(인덱스) 반환
mylist.sort()         # 원본 직접 정렬
```

### 3. 객체 복사 (Shallow vs Deep)
중첩 데이터 구조에서의 참조 오류 방지 기법

```python
import copy

original = [1, 2, [3, 4]]

# 1. 얕은 복사: 외부 리스트만 새 객체, 내부 리스트 주소 공유
shallow = original[:] 
shallow[2][0] = 100   # 원본 original[2][0]도 100으로 변경됨

# 2. 깊은 복사: 모든 계층의 데이터를 완전히 새로운 객체로 복제
deep = copy.deepcopy(original)
deep[2][0] = 999      # 원본 영향 없음 (독립적)
```

### 4. 딕셔너리 및 세트 활용
안전한 키 접근 및 집합 연산 기법

```python
person = {'name': 'Patrick', 'age': 25}

# 안전한 접근 (KeyError 방지)
country = person.get('country', 'Unknown') # 키 없으면 기본값 반환

# 집합 연산
set_a = {1, 2, 3}
set_b = {3, 4, 5}
union = set_a | set_b        # 합집합 ({1, 2, 3, 4, 5})
inter = set_a & set_b        # 교집합 ({3})
diff = set_a - set_b         # 차집합 ({1, 2})
```

---

## 🚀 복습 및 AI 인사이트
* **성능 최적화:** 리스트 중간 삽입/삭제(`insert`, `pop(0)`) 대신 `collections.deque` 사용을 통한 속도 향상 권장
* **디버깅 팁:** 딕셔너리 키 추출 시 `list()`로 형변환하지 않으면 원본 변경 시 뷰(View) 객체 값도 실시간 연동됨에 유의
* **핵심 본질:** 모든 데이터 타입은 객체이며, 각 메서드의 **반환값 존재 여부(In-place vs Return)**를 정확히 구분하는 것이 에러 방지의 핵심성
* **알고리즘 힌트:** 중복 제거 및 고속 검색이 필요한 경우 리스트보다 세트(Set) 자료 구조 활용이 절대적으로 유리함