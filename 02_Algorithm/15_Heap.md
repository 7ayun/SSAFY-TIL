# [Algorithm] 힙 — 완전이진트리, 최대힙, 최소힙, heapq

> **핵심 키워드:** #힙 #Heap #완전이진트리 #힙속성 #최대힙 #최소힙 #ShiftUp #ShiftDown #heapq #우선순위큐 #배열표현 #인덱스공식 #프림 #다익스트라

---

## 학습 목표

* 힙의 2가지 핵심 규칙(완전이진트리, 힙 속성)을 설명할 수 있다
* 완전이진트리와 일반 이진트리의 차이를 구분할 수 있다
* 힙 삽입(Shift Up)과 삭제(Shift Down)의 동작 원리를 설명할 수 있다
* 배열로 힙을 표현하고 인덱스 공식을 활용할 수 있다
* Python heapq 라이브러리를 올바르게 사용할 수 있다

---

## 1. 힙이란

### 1-1. 자료구조로서의 힙

힙은 알고리즘이 아니라 자료구조다. 스택, 큐처럼 데이터를 저장하고 관리하는 방식이다.

**힙의 목적**: 여러 데이터 중 가장 크거나 가장 작은 값을 빠르게 꺼내기 위해 사용한다.

> **강사님 강조**: 힙큐의 목적은 정렬이 아니다. 루트(최대/최솟값)에 있는 애만 빠르게 꺼내는 것이 전부다.

### 1-2. 2가지 핵심 규칙

힙이 되려면 아래 두 조건을 모두 만족해야 한다.

```
조건 1: 완전 이진트리 형태여야 한다
조건 2: 힙 속성(Heap Property)을 만족해야 한다
```

---

## 2. 완전 이진트리

### 2-1. 완전 이진트리 조건

마지막 레벨을 제외하고 모든 레벨이 가득 차 있어야 하며, 마지막 레벨은 왼쪽부터 순서대로 채워야 한다.

```
완전 이진트리:              완전 이진트리 아님:

     O                           O
   /   \                       /   \
  O     O          vs          O     O
 / \   / \                   / \
O   O O   O                  O       O  ← 왼쪽 먼저 채워야 함
O   O                        (왼쪽 빈 자리 있음)
```

> **강사님 강조**: 완전 이진트리 구조 덕분에 삽입 위치를 고민할 필요가 없다. 항상 맨 마지막에 추가하면 된다.

### 2-2. 완전 이진트리를 요구하는 이유

완전 이진트리 구조여야만 배열로 효율적으로 표현할 수 있고, 삽입/삭제 위치가 명확해진다. 이 구조가 힙의 성능을 보장하는 핵심 이유다.

---

## 3. 힙 속성과 종류

### 3-1. 힙 속성

```
최대 힙: 부모 노드의 키값 >= 자식 노드의 키값
최소 힙: 부모 노드의 키값 <= 자식 노드의 키값
```

### 3-2. BST 순서 속성과의 차이

| 구분 | BST 순서 속성 | 힙 속성 |
|------|-------------|--------|
| 규칙 | 왼쪽 < 부모 < 오른쪽 | 부모 > 자식 (모두) |
| 목적 | 특정 값 탐색 | 최대/최솟값 추출 |
| 자식 간 정렬 | 보장됨 | 보장 안 됨 |

> **강사님 주의**: 힙에서 자식 노드끼리는 순서가 보장되지 않는다. `heapq`로 만든 리스트에서 두 번째 인덱스가 두 번째로 작은 값이라고 단정하면 안 된다.

### 3-3. 힙이 아닌 예시

```
완전 이진트리 위반:          힙 속성 위반 (최대힙 기준):
      10                          10
     /  \                        /  \
    8    6                        8    12  ← 부모(10)보다 큼
        / \
     (비어있음)                   힙 아님
     ↑
  왼쪽부터 채워야 함
```

---

## 4. 배열로 힙 표현하기

### 4-1. 인덱스 공식

힙은 완전 이진트리 구조 덕분에 추가 포인터 없이 배열로 표현할 수 있다.

```python
# 현재 노드 인덱스: i
부모 인덱스     = (i - 1) // 2
왼쪽 자식 인덱스 = 2 * i + 1
오른쪽 자식 인덱스 = 2 * i + 2
```

### 4-2. 예시

```
트리 구조:              배열 표현:
     33                [33, 31, 27, 21, 22, 18, 23]
    /  \               인덱스: 0   1   2   3   4   5   6
   31   27
  / \   / \
 21 22 18  23

- 인덱스 1(31)의 부모: (1-1)//2 = 0 → 33  (확인)
- 인덱스 1(31)의 왼쪽 자식: 2*1+1 = 3 → 21  (확인)
- 인덱스 1(31)의 오른쪽 자식: 2*1+2 = 4 → 22  (확인)
```

---

## 5. 힙 삽입 (Shift Up)

### 5-1. 삽입 원리

완전 이진트리를 유지해야 하므로 새 원소는 반드시 맨 마지막 위치에 추가한다. 이후 힙 속성을 만족할 때까지 부모와 비교하며 위로 올라간다.

```
최대 힙에 23 삽입:

초기 상태:         마지막에 추가:       Shift Up:
     20                20                  23
    /  \              /  \               /  \
   17   19            17   19             17   20
  /                  /  \              /  \
 13                 13   23            13   19

                  23 > 19 교환 →  23 > 20 교환 → 완료
```

### 5-2. 삽입 코드

```python
def push(self, item):
    self.heap.append(item)      # 맨 마지막에 추가
    self._shift_up(len(self.heap) - 1)

def _shift_up(self, idx):
    while idx > 0:
        parent = (idx - 1) // 2
        if self.heap[idx] > self.heap[parent]:  # 최대 힙 조건
            self.heap[idx], self.heap[parent] = self.heap[parent], self.heap[idx]
            idx = parent
        else:
            break
```

삽입의 시간 복잡도는 최악의 경우 트리 높이만큼 올라가므로 O(log N)이다.

---

## 6. 힙 삭제 (Shift Down)

### 6-1. 삭제 원리

힙에서는 루트 노드만 삭제할 수 있다. 힙의 목적 자체가 최대/최솟값을 꺼내는 것이기 때문이다.

1. 루트 값을 꺼내서 반환
2. 마지막 노드를 루트로 올림 (완전 이진트리 유지)
3. 힙 속성을 만족할 때까지 더 큰(최대힙) 자식과 교환하며 내려감

```
최대 힙에서 루트(19) 삭제:

     19               11                  15
    /  \             /  \              /  \
   15   11   →      15   (삭제) →     11   (없음)
  /  \             /  \            /  \
 4   13            4   13           4   13

루트 꺼냄    마지막 노드(11)을     11 < 15 → 교환
             루트로 올림           완료
```

### 6-2. 삭제 코드

```python
def pop(self):
    if not self.heap:
        return None
    if len(self.heap) == 1:
        return self.heap.pop()

    root = self.heap[0]
    self.heap[0] = self.heap.pop()  # 마지막 노드 → 루트로
    self._shift_down(0)
    return root

def _shift_down(self, idx):
    size = len(self.heap)
    largest = idx

    left = 2 * idx + 1
    right = 2 * idx + 2

    if left < size and self.heap[left] > self.heap[largest]:
        largest = left
    if right < size and self.heap[right] > self.heap[largest]:
        largest = right

    if largest != idx:
        self.heap[idx], self.heap[largest] = self.heap[largest], self.heap[idx]
        self._shift_down(largest)
```

삭제의 시간 복잡도도 최악의 경우 트리 높이만큼 내려가므로 O(log N)이다.

---

## 7. Python heapq 라이브러리

### 7-1. 기본 사용법

Python의 `heapq`는 기본적으로 최소 힙으로 동작한다.

```python
import heapq

heap = []
heapq.heappush(heap, 5)
heapq.heappush(heap, 3)
heapq.heappush(heap, 8)

print(heap[0])               # 3 (최솟값 조회, O(1))
print(heapq.heappop(heap))   # 3 (최솟값 꺼내기, O(log N))
print(heapq.heappop(heap))   # 5
```

기존 리스트를 힙으로 변환할 때는 `heapify`를 사용한다.

```python
data = [3, 1, 4, 1, 5, 9, 2]
heapq.heapify(data)    # O(N)
print(data[0])         # 1
```

### 7-2. 최대 힙 구현

`heapq`는 최소 힙만 지원하므로 값에 음수를 붙여 최대 힙처럼 사용한다.

```python
heap = []
for val in [3, 1, 4, 1, 5, 9]:
    heapq.heappush(heap, -val)        # 음수로 삽입

max_val = -heapq.heappop(heap)        # 꺼낼 때 다시 음수 처리
print(max_val)                         # 9
```

### 7-3. 튜플로 우선순위 지정

```python
heap = []
heapq.heappush(heap, (2, "작업B"))
heapq.heappush(heap, (1, "작업A"))
heapq.heappush(heap, (3, "작업C"))

priority, task = heapq.heappop(heap)
print(task)    # "작업A" (우선순위 1이 가장 먼저)
```

> **강사님 주의**: `heapify` 후 리스트 인덱스 순서를 정렬된 결과로 오해하면 안 된다. 정렬된 순서로 꺼내고 싶으면 `heappop()`을 반복 호출해야 한다.

```python
data = [3, 2, 1]
heapq.heapify(data)
# data가 [1, 2, 3]처럼 보여도 자식 간 정렬은 보장 안 됨

result = []
while data:
    result.append(heapq.heappop(data))
print(result)    # [1, 2, 3] ← 이렇게 해야 정렬 보장
```

---

## 8. 참고 — 힙의 활용 (읽을거리)

힙은 단독으로 쓰이기보다 다른 알고리즘의 우선순위 큐로 활용되는 경우가 많다.

**프림 알고리즘 (MST)**

```python
import heapq

def prim(graph, start):
    visited = set()
    heap = [(0, start)]
    total = 0
    while heap:
        weight, node = heapq.heappop(heap)
        if node in visited:
            continue
        visited.add(node)
        total += weight
        for next_weight, next_node in graph[node]:
            if next_node not in visited:
                heapq.heappush(heap, (next_weight, next_node))
    return total
```

**다익스트라 알고리즘 (최단경로)**

```python
import heapq

def dijkstra(graph, start):
    dist = {node: float('inf') for node in graph}
    dist[start] = 0
    heap = [(0, start)]
    while heap:
        d, node = heapq.heappop(heap)
        if d > dist[node]:
            continue
        for next_node, weight in graph[node]:
            new_dist = d + weight
            if new_dist < dist[next_node]:
                dist[next_node] = new_dist
                heapq.heappush(heap, (new_dist, next_node))
    return dist
```

---

## 정리

| 구분 | 핵심 포인트 |
|------|------------|
| 힙 2가지 규칙 | 완전이진트리 + 힙 속성 (부모 >= 자식 or 부모 <= 자식) |
| BST와 차이 | 힙은 자식 간 정렬 없음, 최대/최솟값 추출이 목적 |
| 삽입 | 맨 마지막 추가 → Shift Up → O(log N) |
| 삭제 | 루트 꺼냄 → 마지막 노드를 루트로 → Shift Down → O(log N) |
| 인덱스 공식 | 부모: (i-1)//2, 왼쪽: 2i+1, 오른쪽: 2i+2 |
| heapq | 최소 힙 기본, 최대 힙은 음수값으로 구현 |
