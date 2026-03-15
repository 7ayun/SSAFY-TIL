# [Problem Solving] 벌꿀채취

> **핵심 알고리즘:** #조합 #itertools #완전탐색 #제곱수익 #슬라이싱 #2차원리스트

| 항목 | 내용 |
|------|------|
| 출처 | SWEA |
| 핵심 유형 | 완전 탐색, 조합 (A형 수준) |
| 관련 개념 | 07_Exhaustive_Search |

---

## 문제 요약

N x N 벌통에서 두 명의 일꾼이 각각 가로로 연속된 M개의 벌통을 선택한다. 두 일꾼의 영역은 겹치면 안 된다. 각 일꾼은 선택한 M개 중에서 꿀의 합이 C를 넘지 않도록 일부 또는 전부를 채취하며, 수익은 채취한 각 꿀의 양의 제곱의 합이다. 두 일꾼의 수익 합의 최대값을 구한다.

---

## 접근 방식

전체 흐름은 세 단계로 나뉜다.

첫째, 일꾼 1의 위치를 고정한다. 2차원 리스트의 모든 위치에서 가로 M칸을 확보할 수 있는 곳을 순회한다.

둘째, 일꾼 2의 위치를 일꾼 1 이후부터 순회한다. 같은 행에서는 일꾼 1이 확보한 영역 뒤부터, 다음 행부터는 처음부터 탐색한다. 앞부분은 이미 일꾼 1/2 역할을 바꿔서 계산한 것과 동일하므로 중복을 피할 수 있다.

셋째, 각 일꾼의 M개 벌통에서 "꿀 합이 C 이하인 조합 중 제곱합이 최대인 것"을 구한다. M개 중 1개 선택, 2개 선택, ..., M개 전부 선택하는 모든 조합을 구해서 C 제한을 만족하는 것들의 제곱합 최대를 찾아야 한다.

> **강사님 강조**: 조합이 포함된 A형 수준 문제다. 조건이 많은 문제에서는 종이에 조건을 하나하나 나열하고 빠짐없이 구현했는지 확인하는 습관이 중요하다.

---

## 풀이

```python
from itertools import combinations

T = int(input())
for tc in range(1, T + 1):
    N, M, C = map(int, input().split())
    honey = [list(map(int, input().split())) for _ in range(N)]

    max_sum = 0

    for fi in range(N):
        for fj in range(N - M + 1):
            first_honey = honey[fi][fj:fj + M]

            first_max = 0
            for cnt in range(1, M + 1):
                for comb in combinations(first_honey, cnt):
                    if sum(comb) <= C:
                        result = sum(c ** 2 for c in comb)
                        first_max = max(first_max, result)

            for si in range(fi, N):
                for sj in range(N - M + 1):
                    if si == fi and sj < fj + M:
                        continue

                    second_honey = honey[si][sj:sj + M]

                    second_max = 0
                    for cnt in range(1, M + 1):
                        for comb in combinations(second_honey, cnt):
                            if sum(comb) <= C:
                                result = sum(c ** 2 for c in comb)
                                second_max = max(second_max, result)

                    max_sum = max(max_sum, first_max + second_max)

    print(f'#{tc} {max_sum}')
```

`combinations(리스트, 개수)`는 주어진 리스트에서 지정한 개수만큼 뽑는 모든 조합을 반환한다.

> **강사님 팁**: itertools.combinations는 파이썬 표준 라이브러리에 포함되어 있으므로 SWEA에서 사용 가능하다. 조합을 직접 구현하는 방법(재귀, DFS)은 이후 완전 탐색 단원에서 다룬다.

---

## 핵심 패턴

### itertools.combinations 활용

`combinations(iterable, r)`은 iterable에서 r개를 순서 없이 뽑는 모든 조합을 튜플로 반환한다.

### 일꾼 2의 탐색 범위 제한

일꾼 1과 2의 위치를 바꿔도 결과는 같으므로, 일꾼 2는 항상 일꾼 1 이후부터만 탐색한다. 탐색 공간을 절반으로 줄일 수 있다.

---

## 정리

| 구분 | 핵심 포인트 |
|------|-------------|
| 난이도 | A형 수준. 조합 개념이 필요 |
| 영역 선택 | 가로 연속 M칸. 두 일꾼은 겹치면 안 됨 |
| 수익 계산 | 채취한 각 꿀의 제곱합 (합 자체가 아님) |
| C 제한 | 선택한 꿀의 합(제곱이 아닌 원래 값의 합)이 C 이하 |
| 중복 방지 | 일꾼 2는 일꾼 1 이후부터만 탐색 |
| 조합 | combinations(리스트, 개수)로 모든 선택 경우 생성 |
