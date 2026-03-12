# [Algorithm] 분할 정복 — 이진 탐색, 파라메트릭 서치, 병합 정렬, 퀵 정렬

> **핵심 키워드:** #분할정복 #이진탐색 #파라메트릭서치 #병합정렬 #퀵정렬 #재귀

---

## 학습 목표

* 분할 정복의 개념과 탐욕 알고리즘, DP와의 차이를 설명할 수 있다
* 이진 탐색의 동작 원리와 low/high/mid 포인터 관계를 코드로 구현할 수 있다
* 파라메트릭 서치의 핵심 아이디어를 이해하고 최적값 탐색 문제에 적용할 수 있다
* 병합 정렬의 분할 및 병합 단계를 재귀적으로 구현할 수 있다
* 퀵 정렬의 피벗 기반 분할 과정을 이해하고 성능 특성을 비교할 수 있다

---

## 1. 분할 정복 개요

### 1-1. 개념

복잡한 문제를 더 작은 하위 문제들로 나누어 각각 해결한 후, 그 결과를 결합해 전체 해를 구하는 **문제 해결 기법**이다.
특정 알고리즘이 아니라 완전 탐색, 탐욕 알고리즘처럼 문제를 바라보는 방식에 해당한다.

> **강사님 강조**: 분할 정복은 알고리즘이 아닌 문제 해결 기법이다. 헷갈리면 안 된다.

핵심 3단계:
* **분할 (Divide)**: 문제를 동일한 유형의 작은 하위 문제로 쪼갠다
* **정복 (Conquer)**: 하위 문제를 각각 독립적으로 해결한다
* **결합 (Combine)**: 하위 문제의 해를 합쳐 원래 문제의 해를 구한다

### 1-2. 탐욕 / 분할 정복 / DP 비교

세 기법 모두 **최적 부분 구조**(작은 문제의 답을 모아 큰 문제의 답을 만드는 구조)를 가지지만, 접근 방식이 다르다.

| 기법 | 부분 문제 관계 | 선택 방식 |
|------|--------------|---------| 
| 탐욕 알고리즘 | 독립적 | 각 순간 최적 선택 |
| 분할 정복 | 독립적 | 하위 문제를 각각 해결 후 결합 |
| DP | 중복 (연관) | 이전에 구한 답을 재사용 |

> **강사님 강조**: 분할 정복은 하위 문제가 **독립적**, DP는 하위 문제가 **중복**된다. 이 차이가 핵심이다.

---

## 2. 이진 탐색 (Binary Search)

### 2-1. 개념과 조건

정렬된 배열에서 탐색 범위를 절반씩 줄여가며 원하는 값을 찾는 알고리즘이다.
시간복잡도는 O(log N)이며, **반드시 정렬된 상태**여야 동작한다.

* `low`: 탐색 시작 범위 인덱스
* `high`: 탐색 끝 범위 인덱스
* `mid = (low + high) // 2`: 현재 탐색 중앙 인덱스
* `low > high`가 되면 탐색 실패 (값 없음)

### 2-2. 구현 (반복문 / 재귀)

```python
# 반복문 방식
def binary_search_iter(arr, key):
    low, high = 0, len(arr) - 1
    while low <= high:
        mid = (low + high) // 2
        if arr[mid] == key:
            return mid
        elif arr[mid] < key:
            low = mid + 1   # 오른쪽 탐색
        else:
            high = mid - 1  # 왼쪽 탐색
    return -1  # 탐색 실패
```

```python
# 재귀 방식
def binary_search_rec(arr, key, low, high):
    if low > high:          # 종료 조건: 탐색 실패
        return -1
    mid = (low + high) // 2
    if arr[mid] == key:
        return mid
    elif arr[mid] < key:
        return binary_search_rec(arr, key, mid + 1, high)
    else:
        return binary_search_rec(arr, key, low, mid - 1)
```

> **강사님 팁**: `low > high` 역전이 발생하면 해당 값이 배열에 없다는 의미다. 이 종료 조건이 핵심이다.

---

## 3. 파라메트릭 서치 (Parametric Search)

### 3-1. 개념

이진 탐색을 최적값 탐색 문제에 응용한 기법이다.
"X 시간 안에 조건을 만족하는가?"라는 **결정 문제(예/아니오)** 를 이진 탐색으로 반복 판단하여 최적값을 찾는다.

> **강사님 강조**: 입력값이 억 단위를 넘기면 거의 무조건 파라메트릭 서치다. 기업 코테에서 필수 유형이다.

핵심 발상:
* 정답이 나올 수 있는 **범위(low~high)** 를 먼저 설정한다
* 범위의 중간값(`mid`)이 조건을 만족하면 → 범위를 더 줄여 최적화 시도
* 조건 불만족이면 → 반대 방향으로 범위 이동

### 3-2. 구현 예시 — 기계 제품 생산 최소 시간

N개의 기계(각 기계당 제품 1개 생산 시간 Ti), M개의 제품을 생산하는 **최소 시간** 탐색

```python
def can_produce(machines, mid_time, m):
    # 주어진 시간(mid_time)에 M개 이상 생산 가능한지 확인
    total = sum(mid_time // t for t in machines)
    return total >= m

def parametric_search(machines, m):
    low = 1
    high = min(machines) * m   # 가장 빠른 기계만으로 M개 생산하는 시간

    result = high
    while low <= high:
        mid = (low + high) // 2
        if can_produce(machines, mid, m):
            result = mid       # 조건 만족 → 결과 저장, 범위 축소
            high = mid - 1
        else:
            low = mid + 1      # 조건 불만족 → 시간 늘려야 함
    return result

machines = [2, 3, 7]
m = 10
print(parametric_search(machines, m))  # 12
```

> **강사님 강조**: O(N)으로 풀 수 없는 억 단위 문제는 정답 범위를 이진 탐색으로 좁히는 발상으로 접근해야 한다.

---

## 4. 병합 정렬 (Merge Sort)

### 4-1. 개념과 시간복잡도

분할 정복을 활용해 배열을 절반씩 쪼개고, 정렬하면서 다시 합치는 정렬 알고리즘이다.

* 분할 단계: O(log N) — 절반씩 나누기
* 병합 단계: O(N log N) — 전체 데이터를 log N번 비교
* 총 시간복잡도: **O(N log N)** (최선/평균/최악 동일)

| 속성 | 여부 | 이유 |
|------|------|------|
| 안정성 | O | 병합 시 순서대로 비교하므로 동일값 순서 유지 |
| 적응성 | X | 정렬 여부와 무관하게 항상 분할-병합 수행 |
| 제자리 정렬 | X | 병합 과정에서 외부 메모리(임시 리스트) 사용 |

> **강사님 팁**: 파이썬 내장 `sorted()` / `.sort()`는 **Timsort** (병합 정렬 + 삽입 정렬)로 구현되어 있다.

### 4-2. 구현

```python
def merge_sort(arr):
    if len(arr) <= 1:           # 종료 조건: 원소 1개
        return arr

    mid = len(arr) // 2
    left = merge_sort(arr[:mid])    # 왼쪽 재귀 분할
    right = merge_sort(arr[mid:])   # 오른쪽 재귀 분할

    return merge(left, right)

def merge(left, right):
    result = []
    while left and right:
        if left[0] <= right[0]:     # 안정성 보장: <= 사용
            result.append(left.pop(0))
        else:
            result.append(right.pop(0))
    result.extend(left)             # 남은 원소 그대로 추가
    result.extend(right)            # (이미 정렬된 상태이므로 안전)
    return result
```

---

## 5. 퀵 정렬 (Quick Sort)

### 5-1. 개념과 시간복잡도

피벗(기준 원소)을 선택해 피벗보다 작은 원소는 왼쪽, 큰 원소는 오른쪽으로 배치한 뒤, 양쪽 배열에 대해 재귀적으로 동일 과정을 반복하는 정렬이다.

* 피벗 선택 방식에 따라 성능이 크게 달라진다
* 최선/평균: **O(N log N)**, 최악(이미 정렬된 배열 + 맨 앞/뒤 피벗): **O(N²)**

| 속성 | 여부 | 이유 |
|------|------|------|
| 안정성 | X | left/right 포인터 교차 시 원소 교환으로 동일값 상대 순서 변경 가능 |
| 적응성 | O | 피벗 선택에 따라 최선 O(N log N) ~ 최악 O(N²)로 성능이 달라짐 |
| 제자리 정렬 | O | left/right 두 포인터만으로 원소 교환, 외부 메모리 불필요 |

> **강사님 강조**: 퀵 정렬은 기술 면접에도 자주 나온다. 피벗 기준 좌우 분할 개념이 핵심이며, 변형이 많아도 이 원칙은 동일하다.

### 5-2. 구현

```python
def quick_sort(arr):
    if len(arr) <= 1:
        return arr

    pivot = arr[len(arr) // 2]          # 중간값을 피벗으로 선택
    left = [x for x in arr if x < pivot]
    middle = [x for x in arr if x == pivot]
    right = [x for x in arr if x > pivot]

    return quick_sort(left) + middle + quick_sort(right)

print(quick_sort([3, 6, 8, 10, 1, 2, 1]))
# [1, 1, 2, 3, 6, 8, 10]
```

> **강사님 주의**: 피벗을 항상 맨 앞/뒤로 선택하면 이미 정렬된 배열에서 O(N²)로 성능 저하가 발생한다. 중간값 또는 랜덤 선택을 권장한다.

---

## 정리

| 구분 | 핵심 포인트 |
|------|-----------| 
| 분할 정복 | 문제 해결 기법, 하위 문제 독립적 해결 후 결합 |
| vs DP | 분할 정복은 독립적 부분 문제, DP는 중복 부분 문제 |
| 이진 탐색 | 정렬 배열 필수, low/high 역전 시 탐색 실패, O(log N) |
| 파라메트릭 서치 | 이진 탐색으로 정답 범위 좁히기, 억 단위 입력값에서 필수 |
| 병합 정렬 | O(N log N) 안정적, 외부 메모리 사용, 적응성 없음 |
| 퀵 정렬 | 피벗 기준 좌우 분할, 평균 O(N log N), 피벗 선택이 성능 좌우 |
