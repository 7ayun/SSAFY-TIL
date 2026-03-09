# [Algorithm] BFS — 너비 우선 탐색, 큐 활용, 최단 거리, 2D 격자 탐색, DFS와 비교

> **핵심 키워드:** #BFS #BreadthFirstSearch #너비우선탐색 #큐 #deque #popleft #선입선출 #FIFO #최단거리 #레벨탐색 #2D격자 #섬찾기 #방문처리 #visited #완전탐색 #DFS비교

---

## 학습 목표

* BFS의 원리(가까운 노드부터 레벨별로 탐색)와 큐 기반 구현 이해
* 트리에서의 BFS와 그래프에서의 BFS 차이(방문 처리 필요 여부) 구분
* 2D 격자에서 BFS로 최단 거리와 영역(섬) 탐색 문제를 해결
* 같은 문제를 BFS와 DFS로 풀어보며 각각의 장단점 비교

---

## 1. BFS의 개념

### 1-1. BFS란

BFS(Breadth First Search, 너비 우선 탐색)는 시작 노드에서 **가까운 노드부터 레벨별로** 탐색하는 완전 탐색 방법이다. DFS가 한 방향으로 끝까지 깊이 파고드는 것과 달리, BFS는 같은 거리에 있는 노드를 모두 방문한 뒤 다음 레벨로 넘어간다.

### 1-2. 왜 큐를 사용하는가

루트의 자식 B, C, D를 탐색할 때, B의 자식들은 B보다 늦게 들어왔으므로 C, D를 먼저 처리한 뒤에 나와야 한다. **먼저 들어간 것이 먼저 나오는** 선입선출(FIFO) 구조가 필요하므로 큐를 사용한다.

| 탐색 방법 | 자료구조 | 탐색 방향 |
|----------|---------|----------|
| DFS (깊이 우선) | 스택 / 재귀 | 한 방향으로 끝까지 깊이 진입 |
| BFS (너비 우선) | 큐 | 같은 레벨을 먼저, 그 다음 레벨로 |

### 1-3. DFS와 속도 비교

DFS와 BFS는 둘 다 완전 탐색이므로 기본적인 탐색 경우의 수는 동일하다. 차이는 탐색 순서이며, 문제 유형에 따라 적합한 쪽이 다르다.

---

## 2. 트리에서의 BFS

### 2-1. 동작 과정

트리는 1대 N이고 사이클이 없으므로 중복 방문을 걱정할 필요가 없다. 코드가 간단하다.

```
Q에 루트 노드 삽입
↓
Q가 빌 때까지 반복:
  ├ Q에서 노드 하나 꺼냄 (방문)
  └ 해당 노드의 자식들을 Q에 삽입
```

트리 `A → [B, C, D], B → [E, F], D → [G, H, I]`에서 BFS 순서:

```
Q: [A]           → A 꺼냄, B·C·D 삽입
Q: [B, C, D]     → B 꺼냄, E·F 삽입
Q: [C, D, E, F]  → C 꺼냄 (자식 없음)
Q: [D, E, F]     → D 꺼냄, G·H·I 삽입
Q: [E, F, G, H, I] → 순서대로 꺼냄

방문 순서: A → B → C → D → E → F → G → H → I
```

### 2-2. 코드 구현

```python
from collections import deque

tree = {
    'A': ['B', 'C', 'D'],
    'B': ['E', 'F'],
    'D': ['G', 'H', 'I']
}

def bfs_tree(root, tree):
    queue = deque([root])
    result = []

    while queue:
        node = queue.popleft()
        result.append(node)

        if node not in tree:    # 리프 노드면 자식 없음
            continue

        for child in tree[node]:
            queue.append(child)

    return result

print(bfs_tree('A', tree))
# ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I']
```

---

## 3. 그래프에서의 BFS

### 3-1. 핵심 차이 — 방문 처리 필수

그래프는 N대 M 관계이고 사이클이 있을 수 있으므로, **큐에 넣을 때 반드시 방문 처리**를 해야 한다. 방문 처리를 빠뜨리면 같은 노드가 큐에 중복으로 들어가 무한 루프에 빠지거나 답이 틀린다.

```python
from collections import deque

def bfs_graph(start, graph, n):
    visited = [False] * n
    queue = deque([start])
    visited[start] = True       # 큐에 넣을 때 방문 처리
    result = []

    while queue:
        node = queue.popleft()
        result.append(node)

        for neighbor in graph[node]:
            if not visited[neighbor]:
                visited[neighbor] = True    # 큐에 넣을 때 방문 처리
                queue.append(neighbor)

    return result
```

> **강사님 강조**: 방문 처리는 큐에서 꺼낼 때가 아니라 **큐에 넣을 때** 해야 한다. 꺼낼 때 하면 같은 노드가 큐에 여러 번 들어가서 중복 탐색이 발생한다. 이 실수는 무조건 한 번쯤 당한다.

### 3-2. BFS가 적합한 문제 유형

BFS는 가까운 노드부터 레벨별로 탐색하므로, **최단 거리·최소 횟수** 문제에 강하다.

2D 격자에서 시작점(S)에서 도착점(E)까지의 최단 거리를 구할 때, BFS로 한 칸씩 퍼져나가면 E에 처음 도달한 시점의 거리가 곧 최단 거리다.

```
S에서 1칸 → 상하좌우 탐색
S에서 2칸 → 그 다음 레벨 탐색
S에서 3칸 → ...
S에서 4칸 → E 도달! → 최단 거리 = 4
```

---

## 4. 2D 격자 — 섬 찾기 (BFS)

### 4-1. 문제

2D 격자에서 1은 땅, 0은 바다다. 상하좌우 및 대각선으로 연결된 1의 묶음이 하나의 섬이다. 섬의 개수를 구하라.

### 4-2. 접근 — 모든 지점에서 BFS

모든 칸을 순회하면서 땅(1)을 만나면 BFS를 실행한다. BFS가 한 번 실행될 때마다 연결된 모든 땅을 방문 처리하므로, BFS 실행 횟수가 곧 섬의 개수다.

방문 처리는 별도 visited 배열 대신 **원본 격자의 값을 0(바다)으로 변경**하면 간결하다.

```python
from collections import deque

def count_islands(grid):
    n, m = len(grid), len(grid[0])
    # 상하좌우 + 대각선 8방향
    dxy = [(0,1),(0,-1),(1,0),(-1,0),(1,1),(1,-1),(-1,1),(-1,-1)]
    count = 0

    def bfs(x, y):
        queue = deque([(x, y)])
        grid[x][y] = 0                 # 방문 처리 (바다로 변경)

        while queue:
            cx, cy = queue.popleft()
            for dx, dy in dxy:
                nx, ny = cx + dx, cy + dy
                if nx < 0 or nx >= n or ny < 0 or ny >= m:
                    continue
                if grid[nx][ny] == 0:   # 바다면 스킵
                    continue
                grid[nx][ny] = 0        # 방문 처리
                queue.append((nx, ny))

    for i in range(n):
        for j in range(m):
            if grid[i][j] == 1:         # 땅이면 BFS 시작
                bfs(i, j)
                count += 1              # BFS 1회 = 섬 1개

    return count
```

---

## 5. 같은 문제를 DFS로 풀기

### 5-1. 섬 찾기 (DFS 버전)

BFS와 로직은 동일하고, 큐 대신 재귀로 구현한다. 시간 복잡도도 동일하다.

```python
def count_islands_dfs(grid):
    n, m = len(grid), len(grid[0])
    dxy = [(0,1),(0,-1),(1,0),(-1,0),(1,1),(1,-1),(-1,1),(-1,-1)]
    count = 0

    def dfs(x, y):
        grid[x][y] = 0                 # 방문 처리
        for dx, dy in dxy:
            nx, ny = x + dx, y + dy
            if nx < 0 or nx >= n or ny < 0 or ny >= m:
                continue
            if grid[nx][ny] == 0:
                continue
            dfs(nx, ny)

    for i in range(n):
        for j in range(m):
            if grid[i][j] == 1:
                dfs(i, j)
                count += 1

    return count
```

### 5-2. BFS vs DFS 선택 기준

| 상황 | 추천 | 이유 |
|------|------|------|
| 최단 거리·최소 횟수 | BFS | 가까운 것부터 탐색하므로 처음 도달 = 최단 |
| 경로 추적·과정 확인 | DFS | 재귀로 경로를 누적하면서 탐색 가능 |
| 가지치기 필요 | DFS | 중간에 조건 확인 후 분기 차단 가능 |
| 단순 완전 탐색 (섬 찾기 등) | 둘 다 가능 | 시간 복잡도 동일, 취향 차이 |

> **강사님 강조**: BFS가 시험에 단독으로 나오는 경우는 드물고, DFS와 합쳐서 나오거나 완전 탐색 뒤에 최단 거리를 구하는 식으로 출제된다. BFS가 나오면 무조건 맞아야 한다. 난이도가 낮아서 틀리면 아깝다.

---

## 정리

| 구분 | 핵심 포인트 |
|------|-------------|
| BFS | 너비 우선 탐색, 가까운 노드부터 레벨별 탐색, 큐(FIFO)로 구현 |
| 트리 BFS | 사이클 없어 방문 처리 불필요, Q에 자식 넣고 꺼내기 반복 |
| 그래프 BFS | 방문 처리 필수, **큐에 넣을 때** visited 체크 (꺼낼 때 X) |
| 최단 거리 | BFS로 한 칸씩 퍼져나가면 처음 도달 시점 = 최단 거리 |
| 섬 찾기 | 모든 칸 순회 → 땅이면 BFS 실행 → 연결된 땅 방문 처리, BFS 횟수 = 섬 개수 |
| 방문 처리 팁 | 별도 visited 대신 원본 격자를 0으로 변경하면 간결 |
| BFS vs DFS | 최단 거리 → BFS, 경로 추적·가지치기 → DFS, 단순 완탐 → 취향 |
