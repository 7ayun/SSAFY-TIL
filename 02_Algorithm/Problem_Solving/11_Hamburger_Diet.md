# [Problem Solving] 햄버거 다이어트

> **핵심 알고리즘:** #부분집합 #DFS #재귀 #가지치기 #백트래킹 #선택미선택 #칼로리제한

| 항목 | 내용 |
|------|------|
| 출처 | SWEA 5215. 햄버거 다이어트 |
| 핵심 유형 | 부분집합 — DFS 재귀 + 가지치기 |
| 관련 개념 | 10_Backtracking |

---

## 문제 요약

N개의 재료가 있고, 각 재료에 맛 점수와 칼로리가 주어진다. 제한 칼로리(L) 이하의 조합 중에서 맛 점수 합이 가장 높은 햄버거를 구하라.

---

## 접근 방식

재료를 1개만 고르는 경우, 2개 고르는 경우, ..., N개 모두 고르는 경우를 전부 살펴봐야 하므로 **부분집합** 문제다. 경우의 수는 2^N이며, N이 20 이하이면 약 100만으로 충분히 완전 탐색 가능하다.

DFS 재귀의 핵심 설계는 다음과 같다.

**종료 조건 변수(depth):** 현재 선택 여부를 결정할 재료의 인덱스다. depth가 N에 도달하면 모든 재료를 고려한 것이므로 최대 맛 점수를 갱신한다.

**누적값:** 여태까지 선택한 재료들의 누적 칼로리와 누적 맛 점수를 함께 전달한다.

**분기 구조:** 각 재료에 대해 "선택한다"와 "선택하지 않는다" 두 갈래로 재귀 호출한다. 선택한 경우에는 해당 재료의 점수와 칼로리를 누적값에 더해서 전달하고, 선택하지 않은 경우에는 기존 값을 그대로 전달한다.

**가지치기(백트래킹):** 누적 칼로리가 이미 제한 칼로리를 초과했으면, 이후 재료를 더 선택해도 의미가 없으므로 즉시 리턴한다. 이 두 줄의 가지치기를 추가하기 위해 itertools 대신 DFS 재귀로 구현하는 것이다.

이 문제는 모든 DFS 부분집합 문제의 **기본 스켈레톤**이다. itertools의 combinations로도 풀 수 있지만, 모든 경우를 다 구한 뒤 확인하므로 가지치기가 불가능하다. DFS 재귀 방식은 중간에 개입하여 경로를 자를 수 있다는 것이 핵심 차이다.

> **강사님 강조**: 부분집합은 외우는 게 아니라, "선택하냐 마냐"의 분기를 재귀로 펼치는 원리를 이해해야 한다. 이 코드를 그대로 암기해도 도움이 된다.

---

## 풀이

```python
def solve():
    N, L = map(int, input().split())
    items = []
    for _ in range(N):
        score, cal = map(int, input().split())
        items.append((score, cal))

    result = 0

    def dfs(depth, total_score, total_cal):
        nonlocal result

        if total_cal > L:
            return

        if depth == N:
            result = max(result, total_score)
            return

        dfs(depth + 1,
            total_score + items[depth][0],
            total_cal + items[depth][1])

        dfs(depth + 1, total_score, total_cal)

    dfs(0, 0, 0)
    return result

T = int(input())
for tc in range(1, T + 1):
    print(f'#{tc} {solve()}')
```

---

## 핵심 패턴

부분집합 DFS의 스켈레톤 코드:

```python
def dfs(depth, 누적값):
    if 누적값이_제한_초과:
        return
    if depth == N:
        결과_갱신
        return
    dfs(depth + 1, 누적값 + 현재_원소_값)  # 선택 O
    dfs(depth + 1, 누적값)                  # 선택 X
```

이 구조에서 "선택한다/안 한다"를 다른 조건으로 바꾸면 거의 모든 DFS 문제에 적용 가능하다.

---

## 정리

| 구분 | 핵심 포인트 |
|------|-------------|
| 문제 유형 | 부분집합 — 칼로리 제한 하에 맛 점수 최대화 |
| 분기 구조 | 선택 O (누적값 변경) / 선택 X (누적값 유지) |
| 가지치기 | 칼로리 초과 시 즉시 return. itertools 대비 핵심 장점 |
| 복원 불필요 | 선택/미선택 분기라 visited 복원이 필요 없음 |
| 스켈레톤 | 가지치기 → 종료 조건 → 선택 O 재귀 → 선택 X 재귀 |
