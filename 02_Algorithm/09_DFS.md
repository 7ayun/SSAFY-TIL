# [Algorithm] DFS — 그래프 탐색, 방문 처리, 인접 리스트·행렬, 부분집합 응용, 위상 정렬, 공통 조상

> **핵심 키워드:** #DFS #DepthFirstSearch #깊이우선탐색 #그래프 #비선형 #NxM #정점 #간선 #인접리스트 #인접행렬 #간선리스트 #visited #방문처리 #사이클 #연결요소 #재귀 #스택 #부분집합 #가지치기 #위상정렬 #진입차수 #TopologicalSort #LCA #공통조상 #서브트리

---

## 학습 목표

* 그래프의 특징(비선형, N대 M, 사이클)을 이해하고 트리와의 차이 구분
* 간선 리스트를 인접 리스트로 변환하여 DFS를 구현하고, visited로 중복 방문 방지
* 모든 정점에서 DFS를 실행해야 하는 이유(끊어진 그래프) 이해
* 부분집합 문제를 DFS 재귀로 풀고, 가지치기로 최적화하는 전략 습득
* 위상 정렬과 최소 공통 조상(LCA)의 원리와 구현 방법 파악

---

## 1. 그래프와 DFS 기초

### 1-1. 그래프 복습

그래프는 비선형 자료구조로, 데이터(정점, Vertex) 간에 **N대 M의 복잡한 관계**를 표현한다. 트리가 1대 N 계층 구조인 것과 달리, 그래프는 서로 얽히고 설킨 관계가 가능하며 **사이클(자기 자신으로 되돌아오는 경로)** 이 존재할 수 있다.

| 용어 | 의미 |
|------|------|
| 정점(Vertex) | 그래프의 데이터 하나하나 |
| 간선(Edge) | 정점과 정점을 연결하는 선 |
| 인접(Adjacent) | 두 정점이 간선으로 직접 연결된 상태 |

### 1-2. 그래프 표현 방법

**간선 리스트**는 문제에서 입력으로 주어지는 형태다. `(0, 1), (0, 2), (1, 3)` 처럼 연결된 두 정점 쌍의 나열이다.

**인접 리스트**는 각 정점에 연결된 정점 목록을 저장하는 형태로, 실제 DFS 구현에 사용한다.

```python
# 간선 리스트 → 인접 리스트 변환
edges = [(0, 1), (0, 2), (1, 3), (1, 4), (2, 4), (3, 5), (4, 5), (5, 6)]
graph = {}
for u, v in edges:
    graph.setdefault(u, []).append(v)
    graph.setdefault(v, []).append(u)  # 양방향 그래프인 경우
```

**인접 행렬**은 N×N 2차원 리스트로 연결 여부를 0/1로 표현한다. 정점이 많고 간선이 적으면 메모리 낭비가 크지만, 두 정점의 연결 여부를 O(1)로 확인할 수 있다.

> **강사님 강조**: 대부분의 문제는 간선 리스트로 주어지고, 우리는 이것을 인접 리스트로 바꿔서 풀어야 한다. 이 변환 과정이 중요하다.

### 1-3. 그래프에서 DFS를 할 때 유의할 점

트리와 달리 그래프에서 DFS를 적용할 때는 두 가지를 반드시 고려해야 한다.

**첫째, 방문 처리(visited)가 필수다.** N대 M 관계와 사이클 때문에 같은 정점을 중복 방문할 수 있으므로, 한 번 방문한 정점은 다시 방문하지 않도록 표시해야 한다.

**둘째, 모든 정점에서 DFS를 실행해야 한다.** 그래프는 끊어져 있을 수 있다. 한 시작점에서 DFS를 돌리면 연결된 정점만 방문하므로, 떨어져 있는 다른 그룹은 탐색하지 못한다. 따라서 모든 정점에 대해 DFS를 시도하되, 이미 방문한 곳은 즉시 종료한다.

---

## 2. 그래프 DFS 구현

### 2-1. 인접 리스트 + set 방식

정점이 문자열처럼 인덱스로 활용하기 어려운 경우, 방문 여부를 **set**으로 관리하면 편하다. set의 포함 여부 확인과 추가 모두 O(1)이다.

```python
graph = {
    'A': ['B', 'C'],
    'B': ['A', 'D', 'E'],
    'C': ['A', 'E'],
    'D': ['B', 'F'],
    'E': ['B', 'C', 'F'],
    'F': ['D', 'E', 'G'],
    'G': ['C', 'F']
}

def dfs(graph, start, visited, result):
    visited.add(start)          # 방문 처리
    result.append(start)        # 경로에 추가

    for neighbor in graph[start]:
        if neighbor in visited:
            continue            # 이미 방문한 정점은 스킵
        dfs(graph, neighbor, visited, result)

visited = set()
result = []
dfs(graph, 'A', visited, result)
print(' → '.join(result))
# A → B → D → F → E → C → G
```

> **강사님 팁**: 재귀 함수의 파라미터를 뭘 넣어야 할지 모르겠으면 처음에는 위에 만든 변수를 전부 넣어라. 문제를 많이 풀다 보면 필요 없는 것들을 자연스럽게 줄여가게 된다.

### 2-2. 인접 행렬 + 리스트 방식

정점이 숫자(0~N-1)인 경우 인접 행렬과 boolean 리스트로 구현한다.

```python
N = 5
adj_matrix = [
    [0, 1, 1, 0, 0],  # 0번: 1, 2와 연결
    [1, 0, 0, 1, 1],  # 1번: 0, 3, 4와 연결
    [1, 0, 0, 0, 1],  # 2번: 0, 4와 연결
    [0, 1, 0, 0, 1],  # 3번: 1, 4와 연결
    [0, 1, 1, 1, 0],  # 4번: 1, 2, 3과 연결
]
visited = [False] * N

def dfs(current, adj_matrix, visited):
    visited[current] = True
    print(current, end=' ')

    for i in range(len(adj_matrix[current])):
        if adj_matrix[current][i] == 1 and not visited[i]:
            dfs(i, adj_matrix, visited)

# 모든 정점에서 DFS 실행 (끊어진 그래프 대비)
for i in range(N):
    if not visited[i]:
        dfs(i, adj_matrix, visited)
```

모든 정점에서 DFS를 실행하되, 이미 visited가 True인 곳은 바로 넘어가므로 중복 탐색 비용은 거의 없다.

> **강사님 강조**: 샘플 테스트 케이스에서는 그래프가 다 연결되어 있는 척을 해놓고, 내부 테스트 케이스에서 끊어진 그래프를 던지는 경우가 정말 많다. 모든 정점에서 DFS를 실행하는 코드를 빠뜨리면 샘플만 맞고 내부에서 꽝이 된다.

---

## 3. 부분집합과 DFS 응용

### 3-1. 부분집합 문제 — itertools 방식

N명의 점원 중 일부를 골라 탑을 쌓아 선반 높이 B 이상에 도달하되, 가장 낮은 탑을 구하는 문제다. 1명~N명을 고르는 모든 조합(부분집합)을 확인해야 한다.

```python
from itertools import combinations

heights = [3, 1, 3, 5, 6]
B = 16
min_height = float('inf')

for r in range(1, len(heights) + 1):
    for comb in combinations(heights, r):
        total = sum(comb)
        if total >= B and total < min_height:
            min_height = total

print(min_height)
```

> **강사님 강조**: A형 역량평가는 완전탐색 레벨을 벗어나지 않는다. N 자체가 작으므로 itertools의 combinations로 모든 경우를 구해도 시간 안에 돈다. 구현 못하겠으면 쿨하게 itertools 쓰자. 답을 구하는 게 가장 중요하다.

### 3-2. 부분집합 문제 — DFS 재귀 방식

DFS로 "선택한다 / 선택하지 않는다"를 재귀적으로 분기하면 가지치기가 가능하다.

```python
heights = [3, 1, 3, 5, 6]
B = 16
min_height = float('inf')

def dfs(idx, current_sum):
    global min_height

    # 종료 조건: 모든 점원을 고려했을 때
    if idx == len(heights):
        if current_sum >= B:
            min_height = min(min_height, current_sum)
        return

    # 가지치기: 이미 최솟값보다 크면 더 볼 필요 없음
    if current_sum >= min_height:
        return

    # 선택 O
    dfs(idx + 1, current_sum + heights[idx])
    # 선택 X
    dfs(idx + 1, current_sum)

dfs(0, 0)
print(min_height)
```

재귀 함수의 파라미터를 설계할 때 가장 먼저 고민할 것은 **종료 조건을 제어할 변수**(여기서는 idx)다. 그 다음으로 재귀가 진행되면서 누적·변화하는 값(여기서는 current_sum)을 파라미터로 설정한다.

> **강사님 팁**: DFS 코드가 짧아 보이지만 내부적으로 엄청나게 많은 호출이 일어난다. A4 용지에 직접 재귀 호출 과정을 쭉 그려보면 깨달음을 얻는 순간이 온다. 디버거의 Call Stack을 한 줄씩 따라가며 변수 변화를 보는 것도 좋은 방법이다.

---

## 4. 위상 정렬 (Topological Sort)

### 4-1. 위상 정렬이란

방향 비순환 그래프(DAG)에서 **선행 관계를 지키면서** 모든 정점을 한 줄로 나열하는 알고리즘이다. "작업 A를 끝내야 작업 B를 시작할 수 있다" 같은 선후 관계가 있을 때 실행 순서를 결정한다.

역량평가 A형에서도 출제된 적 있는 중요한 알고리즘이다.

### 4-2. 진입 차수(In-degree) 기반 구현

각 정점으로 들어오는 간선의 수를 진입 차수라 한다. 진입 차수가 0인 정점(선행 조건 없는 작업)부터 큐에 넣고, 해당 정점을 제거하면서 연결된 정점의 진입 차수를 줄여가는 방식이다.

```python
from collections import deque

def topological_sort(n, edges):
    graph = [[] for _ in range(n)]
    in_degree = [0] * n

    for u, v in edges:
        graph[u].append(v)
        in_degree[v] += 1

    queue = deque()
    for i in range(n):
        if in_degree[i] == 0:
            queue.append(i)

    result = []
    while queue:
        node = queue.popleft()
        result.append(node)
        for neighbor in graph[node]:
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0:
                queue.append(neighbor)

    return result

# 예: 6개 작업, 선행 관계
edges = [(0, 1), (0, 2), (1, 3), (2, 3), (3, 4), (2, 5)]
print(topological_sort(6, edges))  # [0, 1, 2, 3, 5, 4] 등
```

진입 차수가 0인 정점이 여러 개면 순서가 달라질 수 있으므로, 정답이 유일하지 않을 수 있다.

---

## 5. 최소 공통 조상 (LCA)

### 5-1. LCA란

트리에서 두 노드의 **가장 가까운 공통 조상**을 찾는 알고리즘이다. 트리의 부모 정보와 깊이(depth)를 DFS로 미리 구해놓은 뒤, 두 노드의 깊이를 맞추고 함께 올라가며 만나는 지점을 찾는다.

### 5-2. 전처리 — DFS로 부모·깊이·서브트리 크기 저장

```python
def dfs_preprocess(node, d, tree, depth, parent, subtree_size):
    depth[node] = d
    subtree_size[node] = 1

    for child in tree[node]:
        if depth[child] == -1:        # 아직 방문하지 않은 자식만
            parent[child] = node
            dfs_preprocess(child, d + 1, tree, depth, parent, subtree_size)
            subtree_size[node] += subtree_size[child]
```

리프 노드에 도달하면 서브트리 크기가 1로 확정되고, 재귀가 돌아오면서 부모 노드에 자식들의 서브트리 크기가 누적된다. 피보나치 재귀에서 말단부터 값이 확정되어 올라오는 것과 같은 원리다.

### 5-3. LCA 탐색 — 깊이 맞추고 함께 올라가기

```python
def lca(a, b, depth, parent):
    # 1. 깊이 맞추기 (더 깊은 쪽을 올린다)
    while depth[a] != depth[b]:
        if depth[a] > depth[b]:
            a = parent[a]
        else:
            b = parent[b]

    # 2. 같은 조상을 만날 때까지 함께 올라가기
    while a != b:
        a = parent[a]
        b = parent[b]

    return a  # 최소 공통 조상
```

이 방식의 시간 복잡도는 O(N)이다. 트리 깊이가 매우 깊은 경우(10만 이상) 느릴 수 있으며, 이를 O(log N)으로 줄이는 **이진 리프팅(Binary Lifting)** 기법이 존재한다. 이진 리프팅은 각 노드에서 1, 2, 4, 8, ... 칸 위의 조상을 미리 저장해두고 지수적으로 점프하는 방식이다.

> **강사님 강조**: LCA 기본 구현은 충분히 이해하고, 이진 리프팅은 B형 이상 레벨이니 관심 있는 사람만 따로 공부하면 된다.

---

## 정리

| 구분 | 핵심 포인트 |
|------|-------------|
| 그래프 | 비선형, N대 M 관계, 사이클 가능, 데이터 = 정점(Vertex), 연결 = 간선(Edge) |
| DFS 핵심 규칙 | ① visited로 중복 방문 방지 ② 모든 정점에서 DFS 실행 (끊어진 그래프 대비) |
| 인접 리스트 | 간선 리스트를 딕셔너리/리스트로 변환, DFS 구현에 가장 많이 사용 |
| set vs 리스트 visited | 정점이 문자열이면 set, 정점이 숫자(0~N)이면 boolean 리스트 |
| 부분집합 (itertools) | combinations로 1개~N개 조합을 전부 구해서 조건 확인, A형에서 유효 |
| 부분집합 (DFS 재귀) | 선택 O / 선택 X 분기 + 가지치기로 최적화, 종료 변수(idx) 설계가 핵심 |
| 위상 정렬 | 진입 차수 0인 정점부터 큐에 넣고, 제거하며 차수 갱신, 작업 순서 결정 |
| LCA | DFS로 부모·깊이 전처리 → 깊이 맞추기 → 함께 올라가기 → 공통 조상 반환 |
