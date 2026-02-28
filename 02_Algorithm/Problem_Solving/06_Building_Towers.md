## [문제] 탑을 쌓자

> **활용 알고리즘:** `Greedy`, `Sorting`
> **난이도:** ★★☆☆☆

### 🔍 문제 설명
* 높이가 정해진 두 개의 탑(Tower1, Tower2)에 $N$개의 화물을 나누어 쌓아야 함
* 비용은 `탑의 층수 * 화물의 무게`로 계산됨. 총 비용이 최소가 되도록 화물을 배치하는 방법 구하기

### 💡 풀이 포인트
* **그리디(탐욕) 핵심 논리:** '무거운 화물을 최대한 낮은 층에 배치해야' 총 비용이 최소화됨
* **발상의 전환 (1차원 배열 매핑):**
    * 각 탑에 번갈아 가며 배치하는 복잡한 시뮬레이션 대신, 두 탑에서 사용 가능한 '모든 층수'를 하나의 리스트로 모음
    * 층수 리스트는 오름차순(낮은 층부터) 정렬하고, 화물 리스트는 내림차순(무거운 것부터) 정렬한 뒤 인덱스끼리 곱하여 합산하면 최적해가 도출됨

### 💻 구현 코드 (Python)
```python
def build_towers(N, tower1_height, tower2_height, freights):
    # 1. 화물은 무거운 순으로 내림차순 정렬
    freights.sort(reverse=True)
    
    # 2. 두 탑의 모든 층수를 담을 리스트 생성
    floors = []
    for i in range(1, tower1_height + 1):
        floors.append(i)
    for i in range(1, tower2_height + 1):
        floors.append(i)
        
    # 3. 층수 리스트를 낮은 순으로 오름차순 정렬
    floors.sort()
    
    total_cost = 0
    # 4. 무거운 화물(내림차순)과 낮은 층(오름차순)을 1:1 매칭하여 비용 합산
    for i in range(N):
        total_cost += freights[i] * floors[i]
        
    return total_cost
```