## [문제] 풍선팡

> **활용 알고리즘:** `Dynamic_Delta_Search`
> **난이도:** ★★★☆☆

### 🔍 문제 설명
* $N \times M$ 격자에 배치된 풍선 중 하나를 터뜨릴 때, 해당 풍선에 적힌 숫자만큼 상하좌우 방향으로 추가 폭발이 일어남. 이때 가장 많은 꽃가루가 터지는 개수 구하기

### 💡 풀이 포인트
* **거리 비례 탐색:** 풍선에 적힌 숫자(`burst_range`)만큼 뻗어나감. `dx`, `dy` 배열에 반복문 변수 `dist`를 곱하여 동적 점프(`i + dx * dist`) 구현
* **탐색 최적화 (Break):** 특정 방향으로 뻗어나가던 중 격자판 범위를 벗어나면, 그 이후의 거리는 확인할 필요가 없으므로 `break`를 걸어 불필요한 반복 방지

### 💻 구현 코드 (Python)
```python
def balloon_pang(N, M, grid):
    dx_dy = [(0, 1), (0, -1), (1, 0), (-1, 0)] # 우, 좌, 하, 상
    max_pollen = 0
    
    for i in range(N):
        for j in range(M):
            # 1. 자기 자신의 꽃가루 수로 초기화
            temp_sum = grid[i][j]
            burst_range = grid[i][j]
            
            # 2. 4방향 델타 탐색
            for dx, dy in dx_dy:
                # 3. 터진 풍선의 숫자만큼 거리 연장 탐색
                for dist in range(1, burst_range + 1):
                    ni = i + dx * dist
                    nj = j + dy * dist
                    
                    # 4. 범위를 벗어나면 해당 방향 탐색 즉시 중단 (최적화)
                    if not (0 <= ni < N and 0 <= nj < M):
                        break
                        
                    temp_sum += grid[ni][nj]
                    
            # 5. 최대 꽃가루 수 갱신
            max_pollen = max(max_pollen, temp_sum)
            
    return max_pollen
```