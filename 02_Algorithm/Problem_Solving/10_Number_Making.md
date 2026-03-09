# [Problem Solving] 숫자 만들기

> **핵심 알고리즘:** #DFS #재귀 #순열 #연산자배치 #visited복원 #가지치기 #최대최소

| 항목 | 내용 |
|------|------|
| 출처 | SWEA 4008. 숫자 만들기 |
| 핵심 유형 | DFS 재귀 — 연산자 순열 탐색 |
| 관련 개념 | 10_Backtracking.md |

---

## 문제 요약

N개의 숫자가 적혀 있는 게임판이 있고, 덧셈·뺄셈·곱셈·나눗셈 연산자 카드가 각각 일정 개수 주어진다. 숫자 사이에 연산자를 하나씩 넣어 계산할 때(연산자 우선순위 무시, 왼쪽부터 순서대로 계산), 결과의 최대값과 최소값의 차이를 구하라.

---

## 접근 방식

이 문제는 N-1개의 빈칸에 주어진 연산자 카드를 배치하는 **순열** 문제다. 각 빈칸에 플러스·마이너스·곱셈·나눗셈 중 하나를 넣되, 카드 개수가 남아 있어야 한다.

DFS 재귀로 접근할 때 핵심 설계는 다음과 같다.

**종료 조건 변수(depth):** 현재 계산하려는 숫자의 인덱스다. depth가 N에 도달하면 모든 숫자를 계산한 것이므로 최대·최소값을 갱신한다. 첫 번째 숫자는 연산자 영향을 받지 않으므로 초기값으로 세팅하고, depth=1부터 시작한다는 점에 주의한다.

**누적값(result):** 여태까지 선택한 연산자에 따라 계산된 중간 결과다. 연산자를 선택할 때마다 해당 연산을 적용한 값을 다음 재귀에 전달한다.

**카드 개수 관리:** 연산자를 선택하면 해당 카드 개수를 1 감소시키고, 재귀가 돌아오면 다시 1 복원한다. 이것이 순열에서 visited 복원과 같은 원리다. 한 순열에서 사용된 카드 상태가 다른 순열에 영향을 주면 안 되기 때문이다.

for문으로 0~3 인덱스를 순회하며 카드가 남아 있는 연산자만 선택하는 방식이 가장 깔끔하다. 나눗셈은 `//` 연산자가 아닌 `int(a / b)` 형태로 처리해야 음수에서 올바른 결과를 얻는다.

> **강사님 팁**: 파라미터를 뭘 넣을지 모르겠으면 일단 다 넣어라. 문제를 많이 풀다 보면 필요 없는 것들을 자연스럽게 줄여가게 된다.

---

## 풀이

```python
def solve():
    N = int(input())
    op_list = list(map(int, input().split()))
    numbers = list(map(int, input().split()))

    max_num = -1e9
    min_num = 1e9

    def dfs(op, depth, result):
        nonlocal max_num, min_num

        if depth == N:
            max_num = max(max_num, result)
            min_num = min(min_num, result)
            return

        for op_idx in range(4):
            if op[op_idx] == 0:
                continue
            if op_idx == 0:
                temp = result + numbers[depth]
            elif op_idx == 1:
                temp = result - numbers[depth]
            elif op_idx == 2:
                temp = result * numbers[depth]
            else:
                if numbers[depth] == 0:
                    continue
                temp = int(result / numbers[depth])

            op[op_idx] -= 1
            dfs(op, depth + 1, temp)
            op[op_idx] += 1

    dfs(op_list, 1, numbers[0])
    print(max_num - min_num)

T = int(input())
for tc in range(1, T + 1):
    print(f'#{tc}', end=' ')
    solve()
```

---

## 핵심 패턴

이 문제의 DFS 구조는 햄버거 다이어트(부분집합)와 본질적으로 같은 꼴이다.

| 요소 | 햄버거 다이어트 | 숫자 만들기 |
|------|---------------|------------|
| 선택지 | 토핑을 선택 O / 선택 X | +, -, *, / 중 하나 선택 |
| depth | 토핑 인덱스 | 숫자 인덱스 |
| 누적값 | 칼로리, 점수 | 계산 결과 |
| 복원 | 불필요 | 카드 개수 복원 필수 |

> **강사님 강조**: 이 코드와 햄버거 다이어트 코드가 사실 그렇게 다르지 않다. 종료 조건 적고, 선택한 경우 치고 넘기고, 다른 선택도 치고 넘기고. 이 꼴만 깨달으면 DFS 문제는 다 풀 수 있다.

---

## 정리

| 구분 | 핵심 포인트 |
|------|-------------|
| 문제 유형 | 연산자 순열 배치 → DFS 재귀 |
| depth | 현재 계산할 숫자 인덱스, N 도달 시 종료 |
| 누적값 | 연산자 적용한 중간 결과를 다음 재귀에 전달 |
| 카드 복원 | 재귀 후 op[idx] += 1 필수 (독립적 순열 보장) |
| 스켈레톤 | 종료 조건 → 4가지 연산자 for문 → 카드 확인 → 계산 → 재귀 → 복원 |
