# [Problem Solving] 등산로 조성

> **핵심 알고리즘:** #DFS #재귀 #백트래킹 #완전탐색 #visited #공사

| 항목 | 내용 |
|------|------|
| 출처 | SWEA |
| 핵심 유형 | DFS / 백트래킹 |
| 관련 개념 | 09_DFS, 10_Backtracking |

---

## 문제 요약

N×N 격자의 각 칸에 높이가 있다. 가장 높은 봉우리(최댓값, 복수 가능)에서 출발해 반드시 낮은 칸으로만 이동하며 가장 긴 등산로를 만든다. 단, 경로 중 딱 한 번 최대 K 깊이만큼 인접 칸을 깎아서 이동할 수 있다. 가장 긴 등산로의 길이를 구한다.

---

## 접근 방식

가장 높은 봉우리 전부에서 DFS를 실행하고 최대 길이를 갱신한다.

- 방향은 4방향(상하좌우), 대각선 이동 없음
- 이동 조건: 다음 칸 높이 < 현재 칸 높이
- 공사 조건: 이동 불가 상황(다음 칸 ≥ 현재 칸)에서 공사 이용권이 남아 있고, 최대 K만큼 깎았을 때 현재 높이보다 낮아지는 경우에만 사용
- 공사 후 높이는 `현재 높이 - 1`로 설정해야 이후 경로의 모든 경우를 커버할 수 있다

> **강사님 주의**: K만큼 무조건 다 깎는 게 아니라 "최대 K만큼 자유롭게" 깎을 수 있다. 현재 높이 - 1로 설정해야 모든 경로를 탐색할 수 있다.

> **강사님 팁**: visited 배열이 필요하다. 공사로 낮아진 칸에서 반대편으로 돌아가는 경우를 막아야 한다.

---

## 풀이

```python
import sys
input = sys.stdin.readline

def dfs(cx, cy, depth, can_dig, visited):
    global result
    result = max(result, depth)

    for dx, dy in [(-1,0),(1,0),(0,-1),(0,1)]:
        nx, ny = cx + dx, cy + dy
        if nx < 0 or nx >= N or ny < 0 or ny >= N:
            continue
        if visited[nx][ny]:
            continue

        cur_h = arr[cx][cy]
        nxt_h = arr[nx][ny]

        if nxt_h < cur_h:
            visited[nx][ny] = True
            dfs(nx, ny, depth + 1, can_dig, visited)
            visited[nx][ny] = False

        elif can_dig and nxt_h - K < cur_h:
            original = arr[nx][ny]
            arr[nx][ny] = cur_h - 1
            visited[nx][ny] = True
            dfs(nx, ny, depth + 1, False, visited)
            visited[nx][ny] = False
            arr[nx][ny] = original

N, K = map(int, input().split())
arr = [list(map(int, input().split())) for _ in range(N)]

max_h = max(max(row) for row in arr)
starts = [(i, j) for i in range(N) for j in range(N) if arr[i][j] == max_h]

result = 0
for sx, sy in starts:
    visited = [[False]*N for _ in range(N)]
    visited[sx][sy] = True
    dfs(sx, sy, 1, True, visited)

print(result)
```

---

## 핵심 패턴

### 공사 후 높이를 `cur_h - 1`로 설정

K만큼 깎되 실제 값은 현재 높이보다 딱 1 낮게 설정한다. 이렇게 해야 이후 경로에서 이 칸 주변의 모든 이동 가능 범위를 최대로 확보할 수 있다.

### 상태 복원 (백트래킹)

공사를 사용한 경우 DFS가 돌아온 후 배열 값을 원래대로 복원한다. visited도 마찬가지로 해제한다.

---

## 정리

| 구분 | 핵심 포인트 |
|------|-------------|
| 시작점 | 최댓값과 동일한 모든 봉우리에서 DFS 실행 |
| 이동 조건 | 다음 칸 높이 < 현재 칸 높이 |
| 공사 조건 | can_dig==True AND nxt_h - K < cur_h |
| 공사 높이 | arr[nx][ny] = cur_h - 1 (최대 범위 확보) |
| 복원 | DFS 복귀 후 visited 해제 + 배열 값 원복 |
