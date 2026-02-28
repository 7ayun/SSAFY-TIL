## [문제] 파리퇴치

> **활용 알고리즘:** `2D_Array_Traversal`, `Nested_Loops`
> **난이도:** ★★☆☆☆

### 🔍 문제 설명
* $N \times N$ 배열 안에 파리의 개수가 주어질 때, $M \times M$ 크기의 파리채를 한 번 내리쳐 가장 많이 죽일 수 있는 파리의 수 구하기

### 💡 풀이 포인트
* **유효 범위 제한:** 파리채가 격자판 밖으로 나가지 않도록 시작점(기준점) 반복문의 범위를 `N - M + 1`까지만 순회하도록 제한
* **누적합 초기화:** 타격 기준점 `(i, j)`가 바뀔 때마다 누적 변수(`temp_sum`)를 0으로 초기화해야 각 구역의 합산이 독립적으로 보장됨

### 💻 구현 코드 (Python)
```python
def kill_flies(N, M, grid):
    max_flies = 0
    
    # 1. 파리채를 내리칠 수 있는 유효 기준점 탐색
    for i in range(N - M + 1):
        for j in range(N - M + 1):
            temp_sum = 0
            
            # 2. 기준점 (i, j)에서 M x M 범위의 파리 수 합산
            for r in range(M):
                for c in range(M):
                    temp_sum += grid[i + r][j + c]
                    
            # 3. 최댓값 갱신 (max 함수 활용)
            max_flies = max(max_flies, temp_sum)
            
    return max_flies
```