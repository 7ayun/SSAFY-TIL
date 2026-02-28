# [Algorithm] 완전탐색과 조합적 문제 (Exhaustive Search & Combinatorics)

> **핵심 키워드:** #Brute_Force #Permutation #Combination #Subset #Binary_Counting #Pruning

---

## 🎯 학습 목표
* 시간 복잡도를 배제하고 모든 경우의 수를 확인하여 해답을 찾아내는 완전탐색(Brute Force)의 기조 및 접근 방식 확립
* 순서를 고려하는 순열(Permutation)과 순서를 무시하는 조합(Combination)의 수학적 원리 및 `itertools` 라이브러리 활용법 숙달
* 단 한 줄의 코드 수정으로 순열, 조합, 중복 순열, 중복 조합을 모두 구현하는 '재귀 통합 템플릿(필살기)' 원리 체화
* 배낭 짐싸기(Knapsack) 문제로 대표되는 부분집합(Subset)의 '포함/미포함' 이분법적 재귀 트리 구조 완벽 분석
* 불필요한 탐색 경로를 사전에 차단하여 실행 시간을 극적으로 단축시키는 가지치기(Pruning) 기법 도입
* 비트(Bit) 연산자(`&`, `<<` 등)의 컴퓨터 친화적 연산 속도 활용 및 바이너리 카운팅(Binary Counting) 메커니즘 이해
* 실전 A형 기출 유형인 '요리사(Chef)' 문제의 이중 조합 설계 로직 및 구현 능력 배양

---

## 💡 주요 개념 정리

### 1. 완전탐색 (Brute Force)과 베이비진 (Baby-gin)
* **완전탐색의 본질:** "Just do it". 최적화나 알고리즘 효율을 고민하기 전, 모든 가능한 시나리오를 나열하여 반드시 정답을 도출하는 무식하지만 가장 확실한 해법
* **베이비진 검증 원리:** 6자리 숫자 카드를 뽑아 런(Run: 3자리 연속 숫자)과 트리플렛(Triplet: 3자리 동일 숫자)의 조합만으로 이루어졌는지 판별
* **탐욕(정렬) 접근의 한계:** `123123`의 경우 정렬 시 `112233`이 되어 런 판별에 실패함. 따라서 순서를 고려한 모든 경우의 수(순열)를 뽑아 절반씩 나누어 검증하는 완전탐색이 필수적임

### 2. 순열과 조합 (Permutation & Combination)
* **순열 (Permutation, $N!$):**
    * 서로 다른 $N$개 중 $R$개를 '순서를 고려하여' 나열. (예: 52장 트럼프 카드 중 5장 뽑기 $\rightarrow 52 \times 51 \times 50 \times 49 \times 48$)
    * $N=12$ 이상만 되어도 4억 번 이상의 연산이 발생하므로 입력값이 매우 작을 때만 제한적으로 사용
* **조합 (Combination, $nCr$):**
    * 서로 다른 $N$개 중 '순서에 상관없이' $R$개를 고르는 경우의 수. (예: 로또 번호 추첨)
    * **재귀적 원리:** 특정 원소(예: 1)를 '반드시 포함하는 경우'(선택지 1 감소, 골라야 할 개수 1 감소)와 '절대 포함하지 않는 경우'(선택지만 1 감소)의 합으로 분기함
* **Python 라이브러리:** 실전에서는 재귀 직접 구현보다 `itertools.permutations` 및 `itertools.combinations` 사용을 적극 권장

### 3. 부분집합 (Subset)과 비트 연산 (Binary Counting)
* **부분집합 ($2^N$):**
    * 각 원소를 '선택(O)' 하거나 '미선택(X)' 하는 두 가지 경우의 수를 모든 원소에 적용. (예: A, B, C, D $\rightarrow 2^4 = 16$개)
* **비트 연산 기초:**
    * 컴퓨터 내부 CPU 단위 연산으로 압도적인 속도를 자랑함
    * `&`(AND: 둘 다 1일 때만 1), `|`(OR: 하나라도 1이면 1), `^`(XOR: 다르면 1), `~`(NOT: 반전), `<<`(Left Shift: 비트를 왼쪽으로 밀고 빈자리를 0으로 채움)
* **바이너리 카운팅 로직:**
    * 0부터 $2^N - 1$까지의 10진수(`i`)를 나열. 이는 부분집합의 총 경우의 수를 의미함
    * 마스킹 변수(`j`)를 0부터 $N-1$까지 순회하며 `1 << j`를 생성하고, `i & (1 << j)` 연산을 통해 해당 자리의 비트가 1인지 판별하여 부분집합 원소로 채택함

### 4. 재귀 호출과 세계관 보존 (Memory Reference)
* **파라미터 전달 시 주의점:**
    * 재귀 호출 시 리스트에 `.extend()`나 `.append()`를 사용하면 가변 객체의 주소가 참조되어 원본 데이터(세계관)가 영구적으로 훼손됨
    * 리스트 덧셈(`+`) 연산자를 활용하여 새로운 객체를 생성하고 넘겨야 이전 재귀 단계의 독립적인 세계관이 안전하게 보존됨

### 5. 개발자 마인드셋 및 디버깅 요령
* **PyCharm 디버거 활용:** 무지성 클릭을 지양하고, 브레이크 포인트(Break Point)를 활용하여 예상되는 변수 값(`i`, `j`)을 미리 생각한 뒤 검증하는 방식으로 사용해야 논리력이 상승함
* **시험 멘탈 관리:** 1시간 이상 늪에 빠지면(Index 에러 등), 미련 없이 코드를 전면 초기화하고 화장실에 다녀와 백지상태에서 다시 시작하는 것이 훨씬 효율적임

---

## 💻 기능 구현 및 코드 실습

### [문제] 재귀 기반 통합 템플릿 (필살기 코드)
> **활용 알고리즘:** `Recursion`, `Array_Slicing`
> **난이도:** ★★★☆☆

```python
# [핵심] 재귀 호출 시 넘겨주는 remain(후보군) 배열의 슬라이싱 범위만 바꾸면 4가지 조합론이 모두 구현됨
def generate(selected, remain, pick_count):
    # 1. 종료 조건: 골라야 할 개수를 모두 채웠을 때 출력
    if pick_count == 0:
        print(selected)
        return

    # 2. 로직 처리: 남은 후보군 순회
    for i in range(len(remain)):
        select_i = remain[i]
        
        # ----------------------------------------------------
        # 💡 [필살기] 이 줄의 new_remain 슬라이싱만 바꾸면 됨!
        # 1. 순열: 본인(i)을 제외한 앞뒤 병합 (중복 불가, 순서 고려)
        new_remain = remain[:i] + remain[i+1:]
        
        # 2. 조합: 본인(i) 이후의 원소들만 넘김 (중복 불가, 순서 무시)
        # new_remain = remain[i+1:]
        
        # 3. 중복 순열: 후보군 전체를 다시 넘김 (중복 허용, 순서 고려)
        # new_remain = remain
        
        # 4. 중복 조합: 본인(i)을 포함하여 이후 원소들을 넘김 (중복 허용, 순서 무시)
        # new_remain = remain[i:]
        # ----------------------------------------------------
        
        # 3. 재귀 호출: 선택된 요소 리스트 병합(+) 및 선택 개수 - 1
        generate(selected + [select_i], new_remain, pick_count - 1)
```

### [문제] 요리사 (Chef) - 실전 A형 기출
> **활용 알고리즘:** `itertools.combinations`, `Exhaustive_Search`
> **난이도:** ★★★★☆

### 🔍 문제 설명
* N개의 식재료(짝수)를 N/2개씩 A음식과 B음식으로 분배
* 시너지 `S[i][j] + S[j][i]`의 합으로 맛을 산출하며, 두 음식의 맛 차이가 최소가 되는 값 구하기

### 💻 구현 코드 (Python)
```python
from itertools import combinations

def solve_chef(N, synergy_matrix):
    ingredients = list(range(N))
    min_diff = float('inf')
    
    # 1. 식재료를 N/2개로 나누는 모든 조합(A음식) 생성
    for a_food in combinations(ingredients, N // 2):
        # 2. A음식에 포함되지 않은 식재료를 B음식으로 할당
        b_food = [x for x in ingredients if x not in a_food]
        
        a_score = 0
        b_score = 0
        
        # 3. A음식 내에서 식재료 2개씩 뽑아 시너지 계산
        for i, j in combinations(a_food, 2):
            a_score += synergy_matrix[i][j] + synergy_matrix[j][i]
            
        # 4. B음식 내에서 식재료 2개씩 뽑아 시너지 계산
        for i, j in combinations(b_food, 2):
            b_score += synergy_matrix[i][j] + synergy_matrix[j][i]
            
        # 5. 맛의 차이 절댓값 도출 및 최소값 갱신
        diff = abs(a_score - b_score)
        if diff < min_diff:
            min_diff = diff
            
    return min_diff
```

### [문제] 비트 연산 기반 부분집합 생성 (Binary Counting)
> **활용 알고리즘:** `Bitwise_Shift`, `Masking`
> **난이도:** ★★★☆☆

```python
def binary_counting_subset(arr):
    n = len(arr)
    subsets = []
    
    # 1. 0부터 2^N - 1 까지의 10진수 순회 (1 << n은 2^n과 같음)
    for i in range(1 << n):
        current_subset = []
        # 2. 원소의 개수(자리수)만큼 비트 마스킹 검사
        for j in range(n):
            # i의 j번째 비트가 1인지(& 연산) 확인
            if i & (1 << j):
                current_subset.append(arr[j])
        subsets.append(current_subset)
        
    return subsets
```

---

## 🚀 복습 및 AI 인사이트
* **헷갈렸던 점:**
    * 재귀 호출 구조 설계 시 `depth`(선택 여부를 판단할 인덱스)와 `num_sum`(목표 확인용 누적값)을 파라미터로 넘기는 디자인 패턴 체화 필요성
    * 요리사 문제처럼 1차로 N/2 조합을 구하고, 그 내부에서 다시 2개짜리 조합을 구하는 **이중 조합 설계**의 논리적 흐름 파악의 어려움
* **AI 활용 팁:**
    * A형 모의고사 등 완전탐색 구현 시 시간 초과(Time Out)가 발생할 경우, AI에게 현재 로직의 중복 연산 탐지 및 `Pruning(가지치기)` 조건문 삽입을 요청하여 최적화 수행
    * 재귀로 짜인 부분집합 코드의 실행 흐름(Call Stack)이 머릿속에 그려지지 않을 때, AI에게 트리 구조 형태의 시각화 도표 생성을 요청하여 `include`와 `not include`의 분기점 파악