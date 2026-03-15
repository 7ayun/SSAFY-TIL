# [Problem Solving] 파핑파핑 지뢰찾기

> **핵심 알고리즘:** #BFS #구현 #시뮬레이션 #지뢰찾기 #8방향 #최소클릭

| 항목 | 내용 |
|------|------|
| 출처 | SWEA |
| 핵심 유형 | BFS / 구현 |
| 관련 개념 | 02_Matrix, 11_BFS |

---

## 문제 요약

R×C 격자에서 지뢰 위치가 주어진다. 지뢰가 없는 칸을 클릭하면 주변 8칸의 지뢰 수가 숫자로 표시된다. 주변에 지뢰가 하나도 없는 칸(값=0)을 클릭하면 인접한 0인 칸들이 연쇄적으로 자동으로 열린다. 지뢰를 제외한 모든 칸을 열기 위한 최소 클릭 수를 구한다.

---

## 접근 방식

두 단계로 나눠 풀이한다.

**1단계 — 숫자 격자 사전 계산**: 모든 칸에 대해 주변 8방향의 지뢰 수를 세어 정수 격자를 만든다. 지뢰 칸은 -1로 표시한다.

**2단계 — BFS로 0 영역 확장**: 값이 0인 칸 중 아직 방문하지 않은 칸을 찾아 BFS를 실행한다. BFS 내부에서 0인 인접 칸은 큐에 추가(연쇄 오픈), 0이 아닌 칸은 큐에 추가하지 않는다(경계에서 멈춤). 클릭 횟수를 1 증가시킨다.

**3단계 — 나머지 숫자 칸 카운트**: 전체 순회 후 방문하지 않은 칸(숫자 칸)의 수를 클릭 횟수에 더한다.

> **강사님 강조**: 숫자 칸은 직접 하나하나 클릭해야 하므로 방문 여부를 기준으로 미방문 칸 수를 세면 된다.

---

## 풀이

```python
from collections import deque

def solution(R, C, board):
    # 1단계: 숫자 격자 계산
    arr = [[-1]*C for _ in range(R)]
    dirs8 = [(-1,-1),(-1,0),(-1,1),(0,-1),(0,1),(1,-1),(1,0),(1,1)]
    dirs4_idx = [(-1,0),(1,0),(0,-1),(0,1)]

    for i in range(R):
        for j in range(C):
            if board[i][j] == '*':
                continue
            cnt = 0
            for di, dj in dirs8:
                ni, nj = i+di, j+dj
                if 0 <= ni < R and 0 <= nj < C and board[ni][nj] == '*':
                    cnt += 1
            arr[i][j] = cnt

    visited = [[False]*C for _ in range(R)]

    # 지뢰 칸 방문 처리
    for i in range(R):
        for j in range(C):
            if arr[i][j] == -1:
                visited[i][j] = True

    result = 0

    # 2단계: 0 영역 BFS
    for i in range(R):
        for j in range(C):
            if arr[i][j] == 0 and not visited[i][j]:
                visited[i][j] = True
                q = deque([(i, j)])
                result += 1
                while q:
                    ci, cj = q.popleft()
                    for di, dj in dirs8:
                        ni, nj = ci+di, cj+dj
                        if 0 <= ni < R and 0 <= nj < C and not visited[ni][nj]:
                            visited[ni][nj] = True
                            if arr[ni][nj] == 0:
                                q.append((ni, nj))

    # 3단계: 나머지 숫자 칸 카운트
    for i in range(R):
        for j in range(C):
            if not visited[i][j]:
                result += 1

    return result
```

---

## 핵심 패턴

### 0 영역 BFS 경계 처리

BFS 큐에 추가할 때 숫자가 0인 칸만 큐에 넣는다. 숫자가 있는 칸은 visited 처리만 하고 큐에는 넣지 않아 자동으로 경계가 형성된다.

### 나머지 칸 = 방문 안 한 칸

0 영역 BFS로 처리된 칸은 모두 visited 처리되므로, 순회 후 `not visited`인 칸이 곧 개별 클릭이 필요한 숫자 칸이다.

---

## 정리

| 구분 | 핵심 포인트 |
|------|-------------|
| 사전 계산 | 8방향 지뢰 수 카운트 → 정수 격자 생성 |
| 0 영역 BFS | 0인 미방문 칸 발견 → BFS로 연결된 0 전체 오픈 |
| BFS 큐 조건 | 인접 칸이 0일 때만 큐 추가 (숫자 칸은 방문만) |
| 숫자 칸 처리 | BFS 종료 후 미방문 칸 수 = 개별 클릭 횟수 |
| 결과 | 0 영역 클릭 수 + 미방문 숫자 칸 수 |
