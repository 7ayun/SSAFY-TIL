# [Algorithm] 리스트와 행렬 기초 (List & Matrix Fundamentals)

> **핵심 키워드:** #Big_O #Sorting #2D_Array #Delta_Search #Snail_Array

---

## 🎯 학습 목표
* 알고리즘의 정의 및 좋은 알고리즘의 평가 기준(정확성, 연산량, 메모리, 단순성, 최적성) 정립
* 시간 복잡도(Big-O)의 정량적 계산 및 공간 복잡도와의 트레이드 오프(Trade-off) 관계 이해
* 기초 정렬 알고리즘 4종(Bubble, Selection, Insertion, Counting)의 개별 원리 및 특성 분석
* 2차원 리스트의 행/열/지그재그 순회 기법 및 델타(Delta) 탐색 알고리즘 숙달
* 달팽이 숫자 배열 등 복합적인 매트릭스 제어 로직의 코드 구현 능력 배양

---

## 💡 주요 개념 정리 (전체 내용 포함)

### 1. 알고리즘 및 복잡도 분석
* **좋은 알고리즘 기준:** 정확한 동작(예외 처리), 적은 연산, 메모리 절약, 단순성, 최적성 등
* **Big-O 표기법:** 가장 큰 영향력을 주는 최고차항만 표시하며 계수와 상수는 생략하여 최악의 성능을 보장함
* **주요 시간 복잡도 사례:**
    | 복잡도 | 실제 비유 및 예시 |
    | :--- | :--- |
    | **O(1)** | 노래방 번호 입력 즉시 재생, 딕셔너리 키 조회 |
    | **O(log N)** | 업앤다운 게임, 술병뚜껑 숫자 맞추기 (이진 탐색) |
    | **O(N)** | 1차원 리스트 선형 탐색 (브루트포스) |
    | **O(N log N)** | 퀵/병합 정렬 및 파이썬 내장 정렬(TimSort) |
    | **O(N²)** | 이중 반복문을 사용하는 기초 정렬 기법 |

### 2. 기초 정렬 알고리즘 상세 분석
| 정렬 기법 | 평균 복잡도 | 안정성(Stable) | 적응성(Adaptive) | 제자리(In-place) | 핵심 원리 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **버블(Bubble)** | O(N²) | **Yes** | **Yes** | **Yes** | 인접 원소 비교 및 교환을 통한 최대값 후방 밀어내기 |
| **선택(Selection)** | O(N²) | No | No | **Yes** | 구간 내 최소값 인덱스를 찾아 최전방 원소와 교환 |
| **삽입(Insertion)** | O(N²) | **Yes** | **Yes** | **Yes** | 정렬된 부분에 미정렬 원소의 적정 위치 탐색 후 삽입 |
| **카운팅(Counting)** | O(N+K) | **Yes** | No | No | 정수 데이터의 빈도수 및 누적합을 이용한 위치 할당 |

### 3. 매트릭스 순회 및 델타 탐색
* **표준 좌표계:** 상하 대칭된 `[행 i][열 j]` 구조 활용 및 수학 좌표계 지양
* **지그재그 순회:** 짝수 행은 정방향(`j`), 홀수 행은 역방향(`m-1-j`) 수식 적용
* **델타 탐색:** 특정 좌표 기준 상하좌우 이동을 위한 `dx`, `dy` 배열 기반 탐색
* **경계 조건 처리:** `ni`, `nj` 생성 직후 `0 <= ni < n and 0 <= nj < m` 범위 검증 필수

---

## 💻 기능 구현 및 코드 실습

### 1. 버블 및 선택 정렬 구현
```python
# [Bubble Sort] 인접 요소 스왑 기반 정렬
def bubble_sort(arr):
    n = len(arr)
    for i in range(n): # 패스 반복
        for j in range(0, n - i - 1): # 정렬된 끝부분 제외
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j] # 위치 교환
    return arr

# [Selection Sort] 최소값 인덱스 추적 정렬
def selection_sort(arr):
    n = len(arr)
    for i in range(n - 1):
        min_idx = i # 현재 위치를 최소값으로 가정
        for j in range(i + 1, n):
            if arr[j] < arr[min_idx]:
                min_idx = j # 더 작은 값 발견 시 갱신
        arr[i], arr[min_idx] = arr[min_idx], arr[i] # 최종 교환
    return arr
```

### 2. 삽입 및 카운팅 정렬 구현
```python
# [Insertion Sort] 정렬된 부분집합 내 삽입
def insertion_sort(arr):
    for i in range(1, len(arr)):
        for j in range(i, 0, -1):
            if arr[j] < arr[j - 1]:
                arr[j], arr[j - 1] = arr[j - 1], arr[j]
            else: # 본인 자리를 찾으면 즉시 중단 (적응성)
                break
    return arr

# [Counting Sort] 안정성 확보를 위한 누적합 방식
def counting_sort(arr, max_val):
    counts = [0] * (max_val + 1)
    result = [0] * len(arr)
    for num in arr: counts[num] += 1 # 빈도 기록
    for i in range(1, len(counts)): counts[i] += counts[i-1] # 누적합
    for i in range(len(arr) - 1, -1, -1): # 역순 순회로 안정성 보존
        value = arr[i]
        counts[value] -= 1
        result[counts[value]] = value
    return result
```

### 3. 달팽이 배열 (Snail Array)
> **활용 알고리즘:** `Delta_Search`, `Modulo_Direction_Toggle`
> **난이도:** ★★★☆☆

```python
def solve_snail(N):
    matrix = [[0] * N for _ in range(N)]
    di, dj = [0, 1, 0, -1], [1, 0, -1, 0] # 우-하-좌-위 순서
    i, j, dr = 0, 0, 0
    for num in range(1, N*N + 1):
        matrix[i][j] = num
        ni, nj = i + di[dr], j + dj[dr]
        # 벽에 부딪히거나 이미 숫자가 있는 경우 방향 전환
        if not (0 <= ni < N and 0 <= nj < N) or matrix[ni][nj] != 0:
            dr = (dr + 1) % 4
            ni, nj = i + di[dr], j + dj[dr]
        i, j = ni, nj
    return matrix
```

---

## 🚀 복습 및 AI 인사이트
* **헷갈렸던 점:**
    * 델타 탐색에서 아래 방향 이동 시 인덱스가 `+1` 되는 좌표계 특성 혼동 주의
    * 카운팅 정렬에서 단순히 개수만큼 출력하는 것이 아니라 누적합과 역순 순회를 써야 안정성이 보장됨을 인지
* **AI 활용 팁:**
    * `IndexError` 발생 시 AI에게 경계 조건(`if not (0 <= ni < N)`)의 논리적 오류 검토 요청
    * 복잡한 알고리즘의 최선/최악 시간 복잡도 유도 과정을 시각화 자료와 함께 질문