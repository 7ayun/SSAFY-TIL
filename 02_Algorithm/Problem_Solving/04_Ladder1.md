## [문제] Ladder1 (사다리 타기)

> **활용 알고리즘:** `Simulation`, `Delta_Search`, `Early_Return`
> **난이도:** ★★★☆☆

### 🔍 문제 설명
* $100 \times 100$ 크기의 사다리 배열에서 도착점(2)에 도달할 수 있는 시작점(X)의 열(column) 인덱스 찾기

### 💡 풀이 포인트
* **방문 배열 (Visited):** 사다리의 좌우 이동 중 왔던 길로 무한히 되돌아가는 현상을 방지하기 위해 지나온 길을 1로 마킹하는 독립적인 2차원 배열 활용
* **얼리 리턴 (Early Return):** 1. 범위 이탈 검사, 2. 사다리 유무 검사, 3. 방문 여부 검사를 순차적으로 진행하며 부합하지 않으면 `continue`로 쳐내어 가독성 확보
* **역추적(Reverse Tracking) 아이디어:** 모든 시작점에서 출발해볼 필요 없이, 도착점(2)에서부터 거꾸로 위로 올라가며 시작점을 찾는 방식이 압도적으로 효율적임

### 💻 구현 코드 (Python)
```python
def search_ladder(start_j, grid):
    # 하강보다 좌우 이동을 우선하기 위한 델타 순서 고려 필요
    dx_dy = [(0, 1), (0, -1), (1, 0)] # 우, 좌, 하
    visited = [[0] * 100 for _ in range(100)]
    
    x, y = 0, start_j
    visited[x][y] = 1
    
    # 1. 맨 아래 행(99)에 도달할 때까지 무한 루프
    while x < 99:
        for dx, dy in dx_dy:
            nx, ny = x + dx, y + dy
            
            # 2. [얼리 리턴] 범위를 벗어나면 패스
            if not (0 <= nx < 100 and 0 <= ny < 100):
                continue
            # 3. [얼리 리턴] 사다리(1)가 아니면 패스
            if grid[nx][ny] == 0:
                continue
            # 4. [얼리 리턴] 이미 지나온 길이면 패스
            if visited[nx][ny] == 1:
                continue
                
            # 5. 이동 가능 시 좌표 갱신 및 방문 처리
            x, y = nx, ny
            visited[x][y] = 1
            break # 하나의 방향으로 이동했으므로 델타 반복문 탈출
            
    # 최종 도달한 곳이 목적지(2)인지 판별
    return grid[x][y] == 2
```