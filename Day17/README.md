# Day17. 완전탐색과 순열

---

## 1. 오늘 수업의 전체 흐름

1. 영양평가 시험 전략
2. 완전탐색(Brute Force) 개념
3. 조합적 문제 3종
   - 순열
   - 조합
   - 부분집합
4. 순열 개념과 시간복잡도
5. 순열 구현
   - 반복문 방식
   - 재귀 방식
6. itertools 활용
7. 베이비진 문제에 순열 적용

---

## 2. 영양평가 시험 전략 핵심

### (1) 문제는 반드시 천천히 읽기
- 조건 하나 잘못 읽으면 전체 로직이 틀어짐
- 특히 주의:
  - 초과 / 이하 / 이상
  - 인덱스 범위
  - 초기값 조건

→ 조건은 종이에 따로 적어두는 습관

---

### (2) 테스트케이스 일부만 틀릴 때
대부분 원인은 다음과 같음

- 인덱스 범위 오류
- 초기화 위치 오류
- 들여쓰기 오류
- 0 처리 실수

이 경우:
1. 코드 전부 지우기
2. 문제 다시 읽기
3. 처음부터 다시 작성

---

### (3) A형 문제 전략
- 최적화보다 **일단 구현이 우선**
- 시간초과가 나도 괜찮음
- 이후 가지치기로 개선

---

## 3. 완전탐색(Brute Force)

### 정의
- 가능한 모든 경우를 시도하는 방법

### 특징
장점:
- 정답을 반드시 찾을 수 있음

단점:
- 경우의 수가 많으면 시간초과

### 시험 접근 순서
1. 완전탐색으로 설계
2. 경우의 수가 너무 크면
   → 그리디, 투포인터, 백트래킹 등으로 개선

---

## 4. 조합적 문제 3가지 유형

### (1) 순열 (Permutation)
- 서로 다른 것 중에서
- **순서를 고려하여** 나열

예:
- 순위
- 경로
- 순서가 결과에 영향 주는 문제

공식:
```
nPr = n × (n-1) × ... × (n-r+1)
```

시간복잡도:
```
O(n!)
```

→ n이 12만 넘어도 위험

---

### (2) 조합 (Combination)
- 순서 상관 없이 선택만 하는 경우

예:
- 6개 중 3개 선택

---

### (3) 부분집합 (Subset)
- 0개부터 N개까지 전부 고려

경우의 수:
```
2^n
```

→ n이 30이면 약 10억 수준

---

## 5. 순열 구현 (반복문 방식)

예: 1,2,3의 모든 순열

```python
for i in range(1, 4):          # 첫 번째 자리
    for j in range(1, 4):      # 두 번째 자리
        if i == j:
            continue
        for k in range(1, 4):  # 세 번째 자리
            if k == i or k == j:
                continue
            print(i, j, k)
```

### 핵심 개념
- 앞에서 사용한 숫자는 뒤에서 사용 불가
- 조건문으로 중복 제거

---

## 6. 순열 구현 (재귀 방식)

### 핵심 아이디어
- selected: 지금까지 고른 값
- remain: 앞으로 고를 값들

### 종료 조건
- remain이 비면
→ selected 출력 후 종료

---

### 재귀 순열 코드

```python
def permutation(selected, remain):
    # 종료 조건
    if not remain:
        print(selected)
        return

    # remain에서 하나씩 선택
    for i in range(len(remain)):
        pick = remain[i]

        # 선택한 요소 제외한 새로운 리스트
        new_remain = remain[:i] + remain[i+1:]

        # 원본을 건드리지 않고 새 리스트로 전달
        permutation(selected + [pick], new_remain)


permutation([], [1, 2, 3])
```

---

## 7. extend를 쓰지 않는 이유 (중요)

### 문제 상황
재귀에서 extend를 쓰면:

- 원본 리스트가 직접 수정됨
- 상위 호출로 돌아왔을 때
- selected 값이 이미 바뀌어 있음

→ 다음 순열이 정상적으로 생성되지 않음

### 해결 방법
원본을 유지하고
새 리스트를 만들어 넘김

```
selected + [pick]
```

---

## 8. itertools로 순열 사용

```python
import itertools

nums = [1, 2, 3]

for p in itertools.permutations(nums):
    print(p)
```

---

### 중복순열 (비밀번호 예시)

```python
import itertools

nums = [1, 2, 3]

for p in itertools.product(nums, repeat=3):
    print(p)
```

---

## 9. 베이비진(Baby-gin) 문제

### 조건
6장의 카드가
- run(연속 3장)
- triplet(같은 숫자 3장)

이 두 덩어리로 구성되면 베이비진

---

### 함수 분리

```python
def isRun(cards):
    return cards[0] + 1 == cards[1] and cards[1] + 1 == cards[2]

def isTriplet(cards):
    return cards[0] == cards[1] and cards[1] == cards[2]
```

---

### 순열로 베이비진 검사

```python
import itertools

def isBabyGin(numbers):
    for p in itertools.permutations(numbers):
        first = p[:3]
        second = p[3:]

        first_ok = isRun(first) or isTriplet(first)
        second_ok = isRun(second) or isTriplet(second)

        if first_ok and second_ok:
            return True
    return False
```

---

## 10. 핵심 정리

- 완전탐색은 모든 경우를 시도하는 기본 전략
- 조합적 문제는 세 가지로 나뉜다
  - 순열: 순서 중요
  - 조합: 순서 무관
  - 부분집합: 모든 선택 경우
- 순열은 시간복잡도가 매우 큼 (팩토리얼)
- 재귀 순열의 핵심
  - selected / remain 구조
  - 원본 리스트 훼손 금지

