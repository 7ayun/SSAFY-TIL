# [Algorithm] 백트래킹 — 가지치기, N-Queen, 부분집합 최적화, 순열 DFS

> **핵심 키워드:** #백트래킹 #Backtracking #가지치기 #Pruning #DFS응용 #NQueen #유망성검사 #부분집합 #순열 #visited복원 #경우의수축소 #완전탐색최적화 #결정문제

---

## 학습 목표

* 백트래킹의 개념(해가 아닌 경로를 조기에 포기하고 되돌아가는 기법)을 이해
* N-Queen 문제를 통해 유망성 검사(같은 열·대각선 체크)와 가지치기 원리 습득
* 부분집합 문제에서 DFS 재귀 + 가지치기로 경우의 수를 줄이는 전략 적용
* 순열을 DFS로 구현하고, visited 복원을 통해 독립적인 세계관을 유지하는 방법 이해

---

## 1. 백트래킹의 개념

### 1-1. 백트래킹이란

해를 찾는 도중에 **"이 경로는 해가 될 수 없다"** 는 결론이 나면, 더 깊이 들어가지 않고 **되돌아가서(Backtrack)** 다른 경로를 탐색하는 기법이다.

사실상 DFS에 가지치기(Pruning)를 추가한 것이므로, DFS3라고 봐도 무방하다. 새로운 알고리즘이라기보다 DFS의 응용이라고 이해하면 된다.

### 1-2. 문제 해결 기법 분류에서의 위치

| 기법 | 설명 | A형 출제 빈도 |
|------|------|:---:|
| 완전 탐색 | 모든 경우의 수 확인 (순열·조합·부분집합·DFS·BFS) | 높음 |
| 탐욕 (Greedy) | 현재 최선의 선택을 반복 | 낮음 |
| 분할 정복 | 문제를 쪼개서 해결 후 합치기 | 낮음 |
| DP | 중복 부분 문제를 저장하여 재활용 | 낮음 |

A형 역량평가는 거의 완전 탐색 레벨에서 출제되며, 백트래킹은 완전 탐색의 핵심 최적화 기법이다.

> **강사님 강조**: DFS는 쉬운데 백트래킹은 어렵다고 말하면 모순이다. 백트래킹은 DFS 코드에 가지치기 조건 몇 줄을 추가하는 것뿐이다. 범주 분류를 명확히 하자.

---

## 2. N-Queen 문제

### 2-1. 문제 설명

N×N 체스판에 N개의 퀸을 서로 공격할 수 없도록 배치하는 문제다. 퀸은 상하좌우 + 대각선 4방향을 거리 제한 없이 이동할 수 있으므로, 같은 행·같은 열·같은 대각선에 퀸이 2개 이상 놓이면 안 된다.

### 2-2. 핵심 아이디어

행 단위로 한 행에 퀸 하나씩 배치하면 같은 행 충돌은 자동으로 방지된다. 남은 검사는 **같은 열에 있는지**와 **대각선에 있는지**만 확인하면 된다.

대각선 판별: 두 퀸의 행 차이와 열 차이의 절대값이 같으면 같은 대각선이다. `abs(row1 - row2) == abs(col1 - col2)`

### 2-3. 구현

```python
def n_queen(n):
    cols = [0] * n      # cols[i] = i번 행의 퀸이 놓인 열 번호
    count = 0

    def is_promising(row):
        """현재까지 놓은 퀸들과 충돌하지 않는지 검사"""
        for prev_row in range(row):
            # 같은 열 체크
            if cols[prev_row] == cols[row]:
                return False
            # 대각선 체크
            if abs(prev_row - row) == abs(cols[prev_row] - cols[row]):
                return False
        return True

    def dfs(row):
        nonlocal count
        if row == n:        # 모든 행에 퀸 배치 완료
            count += 1
            return

        for col in range(n):
            cols[row] = col             # row번 행에 col열에 퀸 배치
            if is_promising(row):       # 유망한 경우만 다음 행으로 진행
                dfs(row + 1)

    dfs(0)
    return count

print(n_queen(8))   # 92
```

`is_promising()`이 False를 반환하면 해당 열에 퀸을 놓지 않고 다음 열로 넘어간다. 이것이 **가지치기**다. 유망하지 않은 경로를 조기에 차단하여 탐색 범위를 대폭 줄인다.

> **강사님 팁**: N-Queen은 DFS 없이 처음 마주하면 정말 어렵다. 핵심은 "행 단위로 놓으면 행 충돌 제거, 열·대각선만 체크"라는 아이디어다. 한번 이해하면 백트래킹의 뼈대가 잡힌다.

---

## 3. 부분집합과 백트래킹

### 3-1. 부분집합 복습 — DFS 재귀

N개 원소에서 부분집합을 구하는 것은 각 원소를 "선택한다 / 선택하지 않는다"로 분기하는 DFS다.

```python
def subset_sum(items, target):
    min_over = float('inf')

    def dfs(idx, current_sum):
        nonlocal min_over

        if idx == len(items):
            if current_sum >= target:
                min_over = min(min_over, current_sum)
            return

        # 가지치기: 이미 최솟값보다 크면 더 볼 필요 없음
        if current_sum >= min_over:
            return

        dfs(idx + 1, current_sum + items[idx])   # 선택 O
        dfs(idx + 1, current_sum)                 # 선택 X

    dfs(0, 0)
    return min_over
```

### 3-2. 가지치기의 효과

가지치기 없이 모든 부분집합을 구하면 경우의 수가 2^N이다. 가지치기 조건을 추가하면 불필요한 분기가 조기에 종료되어 실제 탐색 횟수가 크게 줄어든다.

> **강사님 강조**: itertools.combinations로도 풀 수 있지만, 모든 경우의 수를 다 구한 뒤 확인하므로 가지치기가 불가능하다. DFS 재귀 방식은 중간에 개입하여 경로를 자를 수 있다는 것이 핵심 차이다.

---

## 4. 순열과 DFS — 일 분배 문제

### 4-1. 문제 구조

N명의 직원에게 N개의 일을 1:1로 배정할 때, 각 직원이 각 일을 성공할 확률이 주어진다. 모든 일이 연속으로 성공할 확률의 최대값을 구하는 문제다. 이는 순열 문제이며, 모든 배정 순서를 탐색해야 한다.

### 4-2. 순열을 DFS로 구현

```python
def max_probability(n, prob):
    visited = [False] * n
    result = 0

    def dfs(depth, percent):
        nonlocal result

        # 종료 조건: 모든 직원에게 일 배정 완료
        if depth == n:
            result = max(result, percent)
            return

        # 가지치기: 현재 누적 확률이 이미 최대값 이하면 중단
        if percent <= result:
            return

        for i in range(n):
            if visited[i]:          # 이미 배정된 일은 스킵
                continue
            visited[i] = True       # i번 일을 depth번 직원에 배정
            dfs(depth + 1, percent * prob[depth][i])
            visited[i] = False      # 복원 (다른 순열을 위해)

    dfs(0, 1.0)
    return result
```

### 4-3. visited 복원의 의미

순열에서 핵심은 **재귀 호출이 끝난 후 visited를 False로 복원**하는 것이다. 한 순열(세계관)에서 선택한 상태가 다른 순열에 영향을 주면 안 되기 때문이다.

예를 들어 직원 A→일1, 직원 B→일2로 배정한 세계관이 끝나면, 직원 A→일1, 직원 B→일3을 시도할 때 일2는 "아무도 선택하지 않은 상태"여야 한다.

```
DFS 호출 전:  visited[i] = True   ← 선택
DFS 호출 후:  visited[i] = False  ← 복원 (다음 분기를 위해)
```

### 4-4. 가지치기 — 확률은 올라갈 수 없다

성공 확률은 0~1 사이의 값이므로, 곱할수록 줄어들거나 유지된다. 현재 누적 확률이 이미 구한 최대값 이하라면, 이후 어떤 일을 추가해도 최대값을 갱신할 수 없으므로 즉시 종료한다.

이 두 줄의 가지치기를 추가하기 위해 itertools.permutations 대신 DFS 재귀로 구현하는 것이다.

> **강사님 강조**: permutation을 써도 답은 나오지만, 이 두 줄의 가지치기를 추가하려고 힘들게 DFS 재귀로 구현하는 것이다. 그 두 줄 하나 때문에 시간 복잡도 차이가 많이 난다.

---

## 5. 백트래킹 정리 — 코드 패턴

백트래킹 문제는 대부분 아래 뼈대를 따른다.

```python
def dfs(depth, 누적값, ...):
    # 1. 종료 조건
    if depth == 전체크기:
        결과 갱신
        return

    # 2. 가지치기 (백트래킹 핵심)
    if 누적값이_이미_의미없으면:
        return

    # 3. 선택지 탐색
    for 선택 in 선택지:
        if 유망하지_않으면:
            continue
        선택_반영                     # visited[i] = True 등
        dfs(depth + 1, 갱신된_누적값)
        선택_복원                     # visited[i] = False 등
```

종료 변수(depth), 가지치기 조건, 선택-복원 사이클 — 이 세 가지를 잡으면 대부분의 백트래킹 문제를 풀 수 있다.

---

## 정리

| 구분 | 핵심 포인트 |
|------|-------------|
| 백트래킹 | DFS + 가지치기, 해가 될 수 없는 경로를 조기에 포기하고 되돌아감 |
| 가지치기 효과 | 불필요한 분기 조기 종료 → 탐색 범위 대폭 축소, 시간 복잡도 개선 |
| N-Queen | 행 단위 배치 → 열·대각선 충돌만 검사, `is_promising()`이 핵심 |
| 부분집합 + 백트래킹 | 선택 O/X 분기 DFS + 누적값 기반 가지치기 (이미 초과 시 중단) |
| 순열 DFS | visited로 중복 방지 + 재귀 후 visited 복원 (독립적 세계관 유지) |
| 확률 가지치기 | 곱셈 확률은 줄어들기만 하므로, 이미 최대값 이하면 즉시 종료 |
| 코드 패턴 | 종료 조건 → 가지치기 → 선택지 탐색(선택·재귀·복원)의 반복 |
