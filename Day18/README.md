# 조합(Combination) 수업 정리 + 코드 주석

## 1) 오늘 강의 핵심 요약

### 핵심 메시지 3개
1. **조합/순열/부분집합은 완전탐색의 핵심 도구**이며 A형(더 나아가 B형)까지 연결
2. 문제를 풀 때 **입력 범위(제약)**를 먼저 보고, 완전탐색(브루트포스)이 가능한지 판단
3. **조합은 “순서가 중요하지 않은 선택”**이고, 재귀(DFS)로 구현하면 강력

---

## 2) 순열 vs 조합

- **순열(Permutation)**: 순서가 다르면 다른 경우
  - 예) (1,2) ≠ (2,1)
- **조합(Combination)**: 순서가 달라도 같은 경우
  - 예) (1,2) = (2,1)

### 예시: [1,2,3]에서 2개 선택
- 순열(3P2) → (1,2) (1,3) (2,1) (2,3) (3,1) (3,2) = 6개
- 조합(3C2) → (1,2) (1,3) (2,3) = 3개

---

## 3) 조합 재귀식(가장 중요)

### 조합의 재귀적 표현
**nCr = (n-1)C(r-1) + (n-1)Cr**

### 코딩 사고로 해석
특정 원소 X에 대해
- **X를 포함한다** → 남은 (n-1)개 중 (r-1)개 선택
- **X를 포함하지 않는다** → 남은 (n-1)개 중 r개 선택

> 이 “포함/미포함” 이분법 사고가 내일 **부분집합**, 이후 **DFS/백트래킹**의 뼈대입니다.

---

## 4) 반복문으로 조합 규칙 잡기 (i+1 규칙)

조합은 중복(역순)을 없애기 위해 **다음 선택은 항상 더 뒤 인덱스에서만** 고릅니다.

예) 1~4에서 2개 조합
- 첫 번째가 1이면 두 번째는 2~4만 가능
- 첫 번째가 2이면 두 번째는 3~4만 가능
- 첫 번째가 3이면 두 번째는 4만 가능

즉, 2중 루프라면
- 두 번째 반복문의 시작이 **i+1**

---

## 5) 재귀로 조합 만들기(규칙 기반: 슬라이싱)

### 조합(중복 없음): 다음 후보를 arr[i+1:]로 제한
```python
def combination(arr, r):
    """
    arr: 후보 리스트(예: [0,1,2,3])
    r  : 뽑을 개수
    return: r개를 고르는 조합들의 리스트(각 원소는 리스트)
    """

    # 종료 조건 1) 더 뽑을 게 0개면: "아무것도 안 뽑는 경우" 1가지 반환
    # -> 상위 호출에서 [pick] + rest 로 결합할 때 끊기지 않도록 [[]] 반환
    if r == 0:
        return [[]]

    # 종료 조건 2) 후보가 없는데 뽑아야 한다면: 불가능 -> 빈 결과
    if not arr:
        return []

    result = []

    # 현재 단계에서 하나를 선택하고, 다음 단계 후보를 "선택한 것 이후"로 제한
    for i in range(len(arr)):
        pick = arr[i]

        # 조합은 순서가 없으므로 다음 후보는 arr[i+1:]만 허용
        # -> (2,1) 같은 역순 중복을 원천 차단
        for rest in combination(arr[i+1:], r-1):
            # 현재 선택(pick) + 나머지(rest)
            result.append([pick] + rest)

    return result
```

### 왜 이 코드가 맞는가?
- `arr[i+1:]` : 이전에 뽑은 원소보다 **뒤에서만** 고르게 만들어 중복 제거
- `r-1` : 이번에 1개 뽑았으니 남은 개수만큼만 더 뽑기
- `r==0`에서 `[[]]` : 결합용 중립 원소(재귀 결합이 끊기지 않도록)

---

## 6) 한 줄만 바꾸면 4종류(순열/조합/중복순열/중복조합)

### 공통 템플릿
```python
def select(arr, r, next_candidates_fn):
    """
    arr: 후보 리스트
    r  : 뽑을 개수
    next_candidates_fn(arr, i): i번째를 뽑았을 때 다음 후보 리스트를 만드는 함수
    """
    if r == 0:
        return [[]]
    if not arr:
        return []

    result = []
    for i in range(len(arr)):
        pick = arr[i]

        # ★ 여기만 바꾸면 종류가 바뀜
        nxt = next_candidates_fn(arr, i)

        for rest in select(nxt, r-1, next_candidates_fn):
            result.append([pick] + rest)

    return result
```

### (1) 순열(Permutation): 뽑은 것만 제거
```python
def next_perm(arr, i):
    # 순열은 순서가 있으므로 앞/뒤 모두 후보가 될 수 있음
    # 단, 같은 원소 중복 선택은 안 되므로 i번째만 제거
    return arr[:i] + arr[i+1:]
```

### (2) 조합(Combination): 뒤만 허용
```python
def next_comb(arr, i):
    # 조합은 순서 중복 제거를 위해 i 이후만 후보로 남김
    return arr[i+1:]
```

### (3) 중복 순열(Repeated Permutation): 제거 없음
```python
def next_rperm(arr, i):
    # 중복 순열은 같은 원소를 또 뽑을 수 있음
    return arr
```

### (4) 중복 조합(Repeated Combination): 자기 포함 + 뒤
```python
def next_rcomb(arr, i):
    # 중복 조합은 같은 원소 재선택 허용 + 순서 중복 제거
    return arr[i:]
```

사용 예:
```python
arr = [1,2,3,4]
print(select(arr, 2, next_comb))   # 조합
print(select(arr, 2, next_perm))   # 순열
```

---

## 7) 요리사 문제 정리 (조합을 2번 쓰는 대표 문제)

### 문제 구조
- N개의 재료를 **N/2개씩** 두 음식(A,B)로 나눔 (N은 짝수)
- 음식의 맛 = 같은 음식 안에서 발생하는 시너지 합
  - i, j가 함께 있으면 `S[i][j]` 발생
  - **방향이 다르면 값이 다르므로** `S[i][j] + S[j][i]`를 더해야 함
- 목표: `abs(taste(A) - taste(B))` 최소

### 풀이 로직
1. A가 가질 재료를 `N C (N/2)`로 모두 뽑는다 (조합 1번)
2. B는 전체에서 A를 뺀 나머지
3. A 내부에서 2개씩 뽑아 시너지 합산 (조합 2번)
4. B도 동일하게 합산
5. 최소 차이 갱신

---

## 8) 요리사 문제 코드(제출형, 주석 포함)

```python
from itertools import combinations


def taste(food, S):
    """
    food: 한 음식에 들어간 재료 인덱스 리스트(또는 튜플)
    S   : 시너지 행렬
    return: 해당 음식의 맛(시너지 총합)
    """
    total = 0

    # 음식에 포함된 재료들 중 "2개씩" 뽑아 시너지 계산
    # -> 문제 정의상 S[i][j] 뿐 아니라 S[j][i]도 더해야 함
    for i, j in combinations(food, 2):
        total += S[i][j] + S[j][i]

    return total


def solve():
    T = int(input())
    for tc in range(1, T + 1):
        N = int(input())
        S = [list(map(int, input().split())) for _ in range(N)]

        ingredients = list(range(N))
        half = N // 2

        ans = float('inf')

        # A가 가질 재료를 조합으로 모두 생성
        for A in combinations(ingredients, half):
            A_set = set(A)

            # B는 A에 없는 나머지 재료들
            B = [x for x in ingredients if x not in A_set]

            # 각 음식 맛 계산(내부에서 다시 조합 2개씩 사용)
            taste_A = taste(A, S)
            taste_B = taste(B, S)

            diff = abs(taste_A - taste_B)
            if diff < ans:
                ans = diff

                # 이미 0이면 더 내려갈 수 없으므로 종료 가능
                if ans == 0:
                    break

        print(f"#{tc} {ans}")


# 실행
# solve()
```

---

## 9) 실전 체크리스트

### 문제 접근 단계
- 입력 범위를 먼저 확인했는가? (완전탐색 가능 여부 판단)
- 순서가 결과에 영향을 주는가?
  - YES → 순열
  - NO  → 조합
- 선택/미선택(포함/미포함) 구조로 나눌 수 있는가?

### 조합 구현 체크
- 다음 후보를 `i+1` 이후로 제한했는가? (중복 제거 핵심)
- 재귀에서 `r-1`로 정확히 감소시켰는가?
- `r == 0` 종료 조건을 정확히 처리했는가?
- 반환값 구조(`[[]]`)의 의미를 이해했는가?

### 요리사 문제 전용 체크
- A는 `N C (N/2)`로 정확히 생성했는가?
- B는 A를 제외한 나머지로 정확히 구성했는가?
- 시너지 계산 시 `S[i][j] + S[j][i]`를 모두 더했는가?
- 최소값 갱신 로직이 정확한가?


> 조합을 이해하면 부분집합, DFS, 백트래킹까지 연결
> 오늘 내용은 이후 알고리즘 응용 파트의 기초 뼈대