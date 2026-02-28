## [문제] 벌꿀채취

> **활용 알고리즘:** `Brute_Force`, `itertools.combinations`
> **난이도:** ★★★★☆

### 🔍 문제 설명
* $N \times N$ 벌통에서 두 명의 일꾼이 겹치지 않게 가로로 연속된 $M$개의 벌통을 각각 선택함
* 선택한 벌통 중 최대 용량 $C$를 초과하지 않게 꿀을 채취하며, 채취한 꿀의 '제곱의 합'이 최대가 되는 수익 구하기

### 💡 풀이 포인트
* **1. 일꾼 영역 분리 (이중 순회):** 일꾼1이 $(i, j)$에서 $M$칸을 선택하면, 일꾼2는 일꾼1의 영역과 겹치지 않도록 일꾼1의 끝점 이후나 다음 행부터 탐색을 시작해야 함
* **2. 선택 영역 내 최대 수익 도출 (조합):** * 선택한 $M$개의 벌통 꿀 합이 $C$를 넘을 경우, $M$개 중 1개, 2개... $M$개를 고르는 모든 조합(Combination)을 구해야 함
    * 각 조합의 합이 $C$ 이하인 경우에 한해 제곱의 합을 구하고 최대 수익을 갱신함

### 💻 개념 구현 로직 (Python)
```python
from itertools import combinations

def get_max_profit(honey_list, C):
    max_profit = 0
    M = len(honey_list)
    # 1. M개 중 1개~M개를 뽑는 모든 조합 탐색
    for select_count in range(1, M + 1):
        for combo in combinations(honey_list, select_count):
            # 2. 채취량 합이 C 이하일 때만 수익 계산
            if sum(combo) <= C:
                profit = sum(x ** 2 for x in combo)
                max_profit = max(max_profit, profit)
    return max_profit

# 본 로직에서는 일꾼1의 위치를 고정하고, 겹치지 않게 일꾼2의 위치를 잡아
# 각각 get_max_profit()을 호출한 합의 최댓값을 구하는 방식으로 완전 탐색을 진행함
```