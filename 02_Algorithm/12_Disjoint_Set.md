# [Algorithm] 서로소 집합 — Make Set, Find Set, Union, 경로 압축, 랭크 최적화

> **핵심 키워드:** #서로소집합 #DisjointSet #Union-Find #사이클검출 #MakeSet #FindSet #Union #경로압축 #PathCompression #랭크최적화 #그래프 #트리표현

---

## 학습 목표

* 서로소 집합(Disjoint Set)의 개념과 사이클 검출 원리를 설명할 수 있다
* Make Set / Find Set / Union 세 연산을 코드로 구현할 수 있다
* 경로 압축과 랭크 최적화를 적용하여 시간 복잡도를 개선할 수 있다
* 서로소 집합을 그래프 문제(그룹 나누기)에 응용할 수 있다

---

## 1. 서로소 집합이란

### 1-1. 개념 정의

서로소 집합(Disjoint Set)은 공통 원소가 없는 집합들의 자료구조다. 두 집합 A, B에 대해 `A ∩ B = ∅`인 상태를 의미하며, 상호 배타 집합(Mutual Exclusive Set)이라고도 부른다.

> **강사님 강조**: 자료구조와 알고리즘의 차이를 구분하자. 서로소 집합은 데이터를 저장·표현하기 위한 **자료구조**다.

### 1-2. 왜 배우는가

그래프에서 **사이클 검출**을 위해 사용한다. 두 노드를 연결하기 전, 같은 집합에 속해 있으면 연결 시 사이클이 생긴다. 같은 집합인지 판단하는 기준은 **대표자(루트)**가 동일한지 여부다.

> **강사님 강조**: 사이클이 있으면 무한루프, 위상정렬 불가, 최단거리 계산 불가 등 문제가 생긴다. 사이클 검출은 그래프 알고리즘의 핵심이다.

---

## 2. 세 가지 연산

### 2-1. Make Set

모든 원소를 **스스로를 대표자로** 하는 독립 집합으로 초기화한다. 처음에는 그룹을 나눌 기준이 없으므로 모두 자기 자신이 대장이 된다. 오직 초기화 시 한 번만 실행된다.

```python
# parent[i] = i의 직속 부모 (초기에는 자기 자신)
def make_set(n):
    parent = list(range(n + 1))  # 1-indexed
    return parent
```

### 2-2. Find Set

원소 `x`가 속한 집합의 **대표자(루트)**를 반환한다. `parent[x] == x`이면 자기 자신이 대표자이므로 반환하고, 아니라면 부모를 대상으로 재귀 호출한다.

```python
# 기본 구현 — O(N)
def find_set(x):
    if x == parent[x]:
        return x
    return find_set(parent[x])
```

> **강사님 팁**: `parent` 리스트는 대표자가 아니라 **직속 부모**를 저장한다. 대표자는 Find Set을 타고 올라가야 알 수 있다.

### 2-3. Union

두 집합을 합친다. **대표자들끼리** 붙는 것이 핵심이다. 한쪽 대표자가 지면 그 부하 전체가 상대 집합으로 편입된다.

```python
# 기본 구현 — O(N)
def union(x, y):
    px = find_set(x)
    py = find_set(y)
    if px != py:
        parent[py] = px  # py 집합을 px 집합으로 편입
```

> **강사님 강조**: Union 내부에서 반드시 Find Set이 실행된다. 대표자를 먼저 구해야 합칠 수 있기 때문이다.

---

## 3. 표현 방식 비교

서로소 집합은 연결 리스트 또는 트리로 표현할 수 있다.

| 표현 방식 | Find Set | Union | 특징 |
|-----------|----------|-------|------|
| 연결 리스트 | O(1) — 대표자 주소 직접 참조 | O(N) — 모든 원소의 포인터 갱신 | 구현 간단, Union이 느림 |
| 트리 (기본) | O(N) — 루트까지 순회 | O(N) — Find 포함 | 편향 트리 시 성능 저하 |
| 트리 (최적화) | O(1) | O(1) | 경로 압축 + 랭크 적용 |

실제 구현에서는 **리스트 기반 트리**를 사용하며, 최적화까지 적용한다.

---

## 4. 최적화

### 4-1. 경로 압축 (Path Compression)

Find Set을 수행할 때, 탐색 경로 위의 모든 노드가 **직접 대표자를 가리키도록** 갱신한다. 재귀 호출에서 돌아오면서 `parent[x]`를 대표자로 바꾸는 방식이다.

```python
# 경로 압축 적용 — Find Set 시 부모를 대표자로 갱신
def find_set(x):
    if x != parent[x]:
        parent[x] = find_set(parent[x])  # 돌아오면서 갱신
    return parent[x]
```

예시: `5 → 4 → 3 → 2(대표자)` 구조에서 `find_set(5)` 호출 시
→ 재귀로 대표자 2를 찾은 후, `parent[5]`, `parent[4]`, `parent[3]` 모두 2로 갱신됨

### 4-2. 랭크를 이용한 Union

각 노드의 서브트리 높이를 **랭크(rank)**로 저장한다. Union 시 **랭크가 낮은 집합을 높은 집합에 붙여** 트리가 한쪽으로 치우치는 것을 방지한다.

```python
def union(x, y):
    px, py = find_set(x), find_set(y)
    if px == py:
        return  # 이미 같은 집합 — 사이클 발생
    if rank[px] > rank[py]:
        parent[py] = px
    elif rank[px] < rank[py]:
        parent[px] = py
    else:                       # 랭크가 같으면
        parent[py] = px         # 아무 쪽이나 병합하고
        rank[px] += 1           # 대표자의 랭크 +1
```

> **강사님 팁**: 랭크가 같은 두 집합이 합쳐질 때만 랭크가 증가한다. 낮은 쪽이 높은 쪽으로 붙으면 전체 높이가 늘지 않아 시간 복잡도가 유지된다.

---

## 5. 응용 — 그룹 나누기

### 5-1. 문제 접근

N명의 학생 중 같은 조가 되고 싶은 쌍을 Union으로 연결한 뒤, 최종 대표자의 수를 세면 조의 개수를 구할 수 있다.

> **강사님 주의**: Union이 끝난 후 경로 압축이 되지 않은 노드가 있을 수 있다. 모든 원소에 대해 `find_set(i)`를 한 번 돌려서 `parent`를 전부 갱신한 뒤 대표자 수를 세야 한다.

### 5-2. 전체 코드

```python
import sys
input = sys.stdin.readline

def find_set(x):
    if x != parent[x]:
        parent[x] = find_set(parent[x])
    return parent[x]

def union(x, y):
    px, py = find_set(x), find_set(y)
    if px == py:
        return
    if rank[px] > rank[py]:
        parent[py] = px
    elif rank[px] < rank[py]:
        parent[px] = py
    else:
        parent[py] = px
        rank[px] += 1

T = int(input())
for _ in range(T):
    n, m = map(int, input().split())
    parent = list(range(n + 1))  # Make Set
    rank   =  * (n + 1)

    for _ in range(m):
        x, y = map(int, input().split())
        union(x, y)

    # 모든 노드의 대표자 갱신 (경로 압축 미적용 노드 처리)
    for i in range(1, n + 1):
        parent[i] = find_set(i)

    print(len(set(parent[1:])))  # 0번 인덱스 제외
```

> **강사님 주의**: `parent`를 1-indexed로 만들면 0번 인덱스가 의도치 않게 집합 대표자로 잡힌다. `set(parent[1:])`로 0번을 제외하거나, 결과에서 `-1` 처리해야 한다.

---

## 정리

| 구분 | 핵심 포인트 |
|------|-------------|
| 서로소 집합 | 교집합이 없는 집합들의 자료구조, 그래프 사이클 검출에 사용 |
| Make Set | 초기화 — 모든 원소가 스스로 대표자, 1회만 실행 |
| Find Set | 대표자 반환 — `parent[x] == x`이면 대표자, 아니면 재귀 |
| Union | 두 집합 합치기 — 대표자들끼리 연결, 내부에서 Find Set 실행 |
| 경로 압축 | Find Set 재귀 복귀 시 `parent[x]`를 대표자로 직접 갱신 → O(1) |
| 랭크 최적화 | 낮은 랭크가 높은 랭크에 편입, 같은 경우 합친 후 +1 → 트리 균형 유지 |
