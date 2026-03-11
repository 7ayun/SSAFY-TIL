# [Algorithm] 최소 신장 트리 — Kruskal, Prim, 귀류법 증명

> **핵심 키워드:** #MST #최소신장트리 #MinimumSpanningTree #Kruskal #Prim #그리디 #탐욕알고리즘 #우선순위큐 #힙 #사이클검출 #UnionFind #귀류법

---

## 학습 목표

* 신장 트리와 최소 신장 트리의 개념과 차이를 설명할 수 있다
* Kruskal 알고리즘의 동작 원리를 코드로 구현할 수 있다
* Prim 알고리즘의 동작 원리와 우선순위 큐 활용법을 이해할 수 있다
* 두 알고리즘의 차이(간선 중심 vs 정점 중심)를 비교하고 상황에 맞게 선택할 수 있다

---

## 1. 최소 신장 트리란

### 1-1. 용어 정의

| 용어 | 영문 | 정의 |
|------|------|------|
| 신장 트리 | Spanning Tree | N개의 정점을 N-1개의 간선으로 연결한 트리 (무향 그래프) |
| 최소 신장 트리 | Minimum Spanning Tree | 신장 트리 중 **간선 가중치 합이 최소**인 것 |

> **강사님 팁**: MST = **M**inimum **S**panning **T**ree. 약자보다 풀네임을 먼저 익혀야 내용이 기억된다. AI 분야에서도 약자 범람이 심하므로 풀네임 위주로 공부하는 습관을 들이자.

### 1-2. 대표 활용 사례

* 도로/수도관/회로 등 최소 비용으로 모든 지점을 연결하는 문제
* 네트워크 구성 최적화
* 섬을 가장 적은 다리로 연결하기 (백준 `다리 만들기 2` 유형)

---

## 2. Kruskal 알고리즘

### 2-1. 핵심 아이디어

**간선 중심** 탐욕 알고리즘이다. 가중치가 가장 작은 간선부터 선택하되, 선택 시 사이클이 생기면 건너뛴다. 이 과정을 N-1개의 간선이 선택될 때까지 반복한다.

> **강사님 강조**: 사이클 검출은 **서로소 집합(Union-Find)**으로 한다. 두 노드의 `find` 결과(대표자)가 같으면 사이클 발생 → Union 생략.

### 2-2. 동작 순서

1. 모든 간선을 가중치 기준 **오름차순 정렬**
2. 간선을 순서대로 순회하며:
   - `find(u) == find(v)` → 사이클 → 건너뜀
   - `find(u) != find(v)` → `union(u, v)` 후 MST 목록 추가
3. MST 간선이 N-1개가 되면 종료

### 2-3. 구현 코드

```python
import sys
input = sys.stdin.readline

def find(x):
    if x != parent[x]:
        parent[x] = find(parent[x])  # 경로 압축
    return parent[x]

def union(x, y):
    px, py = find(x), find(y)
    if px == py:
        return False  # 사이클 발생
    if rank[px] >= rank[py]:
        parent[py] = px
        if rank[px] == rank[py]:
            rank[px] += 1
    else:
        parent[px] = py
    return True

V, E = map(int, input().split())
parent = list(range(V + 1))  # Make Set
rank   =  * (V + 1)

edges = []
for _ in range(E):
    u, v, w = map(int, input().split())
    edges.append((w, u, v))

edges.sort()  # 가중치 오름차순 정렬

mst_cost = 0
mst_edges = []

for w, u, v in edges:
    if union(u, v):
        mst_cost += w
        mst_edges.append((u, v, w))
    if len(mst_edges) == V - 1:
        break

print(mst_cost)
```

### 2-4. 시간 복잡도

간선 정렬에 O(E log E), 간선 순회 + Union-Find에 O(E)이므로 전체 시간 복잡도는 **O(E log E)**다.

> **강사님 강조**: V개의 정점이 있으면 최대 간선 수는 V(V-1)/2이므로, E log E는 E log V로도 표현할 수 있다.

---

## 3. Kruskal 정확성 증명 (귀류법)

### 3-1. 귀류법이란

증명하려는 명제의 **반대를 참으로 가정**한 뒤, 그 안에서 **모순을 도출**해 원래 명제가 참임을 간접 증명하는 방법이다.

### 3-2. 증명 과정

| 단계 | 내용 |
|------|------|
| 가정 | Kruskal로 만든 트리 T는 MST가 **아니다** |
| 전제 | 진짜 MST인 T*가 존재하고, T와 다른 간선 E를 포함한다 |
| 논리 | T는 E 대신 간선 F를 선택했고, Kruskal 특성상 `w(F) ≤ w(E)` |
| 모순 | T*에서 E를 F로 대체하면 비용이 같거나 더 작아짐 → T*가 MST라는 조건 붕괴 |
| 결론 | 가정이 틀렸으므로, Kruskal로 만든 T는 MST **맞다** |

> **강사님 강조**: 이 증명을 문제 풀 때마다 떠올릴 필요는 없다. "작은 걸 골랐는데 사이클이 생기면 굳이 넣을 이유가 없다"는 직관만 갖고 있으면 충분하다.

---

## 4. Prim 알고리즘

### 4-1. 핵심 아이디어

**정점 중심** 탐욕 알고리즘이다. 임의의 정점에서 시작해 현재 MST에서 갈 수 있는 간선 중 **가중치가 가장 작은 미방문 정점**을 계속 선택한다. 사이클 방지를 `visited` 배열로 처리한다.

> **강사님 강조**: Kruskal은 서로소집합으로 사이클을 막고, Prim은 `visited`로 막는다. 방식이 다를 뿐 최소 비용 결과는 항상 동일하다.

### 4-2. 동작 순서

1. 임의 정점 선택 → `visited` 처리
2. 해당 정점과 연결된 모든 간선을 **최소 힙(우선순위 큐)**에 삽입
3. 힙에서 가중치 최솟값 꺼내기:
   - 목적지 정점이 이미 방문 → 건너뜀
   - 미방문 → MST 추가, 해당 정점 연결 간선을 힙에 삽입
4. N-1개 선택 시 종료

### 4-3. 구현 코드

```python
import heapq
import sys
input = sys.stdin.readline

V, E = map(int, input().split())
graph = [[] for _ in range(V + 1)]

for _ in range(E):
    u, v, w = map(int, input().split())
    graph[u].append((w, v))
    graph[v].append((w, u))  # 무향 그래프

def prim(start):
    visited = [False] * (V + 1)
    heap = [(0, start)]  # (가중치, 정점)
    total = 0

    while heap:
        w, u = heapq.heappop(heap)
        if visited[u]:
            continue
        visited[u] = True
        total += w
        for cost, v in graph[u]:
            if not visited[v]:
                heapq.heappush(heap, (cost, v))
    return total

print(prim(1))
```

### 4-4. 시간 복잡도

힙 삽입/삭제가 O(log V)이고 간선 수만큼 반복하므로 총 시간 복잡도는 **O(E log V)**다. 밀집 그래프에서 Kruskal보다 유리할 수 있다.

---

## 5. Kruskal vs Prim 비교

| 구분 | Kruskal | Prim |
|------|---------|------|
| 중심 | 간선 | 정점 |
| 사이클 방지 | Union-Find | `visited` 배열 |
| 정렬/자료구조 | 간선 전체 오름차순 정렬 | 최소 힙(우선순위 큐) |
| 적합한 그래프 | 희소 그래프 (간선 ↓) | 밀집 그래프 (간선 ↑) |
| 음의 가중치 | 처리 가능 | 처리 가능 |
| 시간 복잡도 | O(E log E) | O(E log V) |

---

## 정리

| 구분 | 핵심 포인트 |
|------|-------------|
| 신장 트리 | N개 정점, N-1개 간선, 무향 그래프에서의 연결 트리 |
| MST | 신장 트리 중 간선 가중치 합이 최소인 것 |
| Kruskal | 간선 오름차순 정렬 → 사이클 없으면 Union, 있으면 스킵 |
| Prim | 임의 정점 출발 → 최소 힙으로 미방문 최소 간선 선택 반복 |
| 사이클 방지 | Kruskal: Union-Find 대표자 비교 / Prim: visited 배열 |
| 결과 | 두 알고리즘의 최소 비용은 항상 동일 |
| 그래프 선택 | 희소 그래프 → Kruskal, 밀집 그래프 → Prim |
