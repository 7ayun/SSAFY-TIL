# 문제풀이/개념 정리

---

## 0) 강사님 메시지
- 알고리즘은 **안 풀면 감이 떨어짐** → SSAFY 기간에 **내용 숙지의 고점**을 찍어두면 나중에 복구가 쉽습니다.
- 문제 접근 기본 순서
  1) **완전탐색(가능한 모든 경우 고려)**
  2) 유형이 보이면 최적화(그리디/DP/분할정복 등)
- 그리디는 **공식이 없고 증명도 어려워** 유형을 많이 경험해야 합니다.

---

## 1) 채점 시스템

### 핵심 규칙
- 학생 답안은 `0/1`
  - `1`: 정답, `0`: 오답
- 연속 정답이면 점수 증가
  - 예: 연속 3번 맞으면 `1 + 2 + 3`
- 오답이면 연속 카운트 0으로 초기화
- 목표: **학생 점수의 최댓값 - 최솟값**

### 입력에서 자주 터지는 포인트
- 첫 줄: `N M` (학생 수, 문항 수)
- 다음 줄: **정답지**
- 다음 N줄: **학생 답안지**

### 구현 코드(주석 포함)
```python
# 채점 시스템
# 입력: N(학생 수), M(문항 수)
N, M = map(int, input().split())

# 정답지(길이 M)
correct = list(map(int, input().split()))

# 전체 학생 점수 중 최댓값/최솟값 초기화
max_score = 0
min_score = 10**9

# 학생 N명에 대해 반복
for _ in range(N):
    # 현재 학생의 답안지(길이 M)
    sheet = list(map(int, input().split()))

    # 연속 정답 개수(맞으면 +1, 틀리면 0)
    streak = 0

    # 현재 학생 누적 점수
    total = 0

    # 문항 M개를 채점
    for i in range(M):
        if sheet[i] == correct[i]:
            # 정답이면 연속 정답 증가
            streak += 1
            # 연속 정답 수만큼 점수 추가
            total += streak
        else:
            # 오답이면 연속 카운트 초기화
            streak = 0

    # 전체 최댓값/최솟값 갱신
    max_score = max(max_score, total)
    min_score = min(min_score, total)

# 정답: 최고점 - 최저점
print(max_score - min_score)
```

---

## 2) 탑을 쌓자 (최소 비용)

### 핵심 아이디어
- 비용 = **층수 × 화물 무게**
- 최소 비용을 만들려면: **무거운 화물을 낮은 층(작은 층수)에 배치**

### 풀이 A: 정렬 매칭(가장 단순)
1) 화물 무게: **내림차순**
2) 가능한 층수 목록: 탑1의 `[1..h1]` + 탑2의 `[1..h2]`
3) 층수 목록: **오름차순**
4) 같은 인덱스끼리 곱해서 합

```python
# 탑을 쌓자 - 정렬 매칭 풀이
# 입력: n(화물 개수), h1(탑1 높이), h2(탑2 높이)
n, h1, h2 = map(int, input().split())

# 화물 무게 리스트
weights = list(map(int, input().split()))

# 1) 무거운 화물부터 처리해야 하므로 내림차순 정렬
weights.sort(reverse=True)

# 2) 가능한 모든 층수를 리스트로 생성
floors = []

# 탑1의 층수 1..h1 추가
for i in range(1, h1 + 1):
    floors.append(i)

# 탑2의 층수 1..h2 추가
for i in range(1, h2 + 1):
    floors.append(i)

# 3) 낮은 층부터 배치해야 하므로 오름차순 정렬
floors.sort()

# 4) 무거운 화물(내림차순) × 낮은 층(오름차순)을 1:1 매칭
#    이 매칭이 최소 비용을 만듦
cost = 0
for i in range(n):
    cost += weights[i] * floors[i]

print(cost)
```

> 풀이 B(시뮬레이션): 두 탑의 “다음에 놓을 층”을 비교하며 진행하는 방법도 가능하지만, 조건 분기가 길어져 실수 위험이 커서 **정렬 매칭**이 시험에서 안전합니다.

---

## 3) 비밀번호 (연속 쌍 소거)

### 규칙
- 인접한 같은 숫자 쌍은 삭제
- 삭제 후 새로 인접한 쌍이 생기면 연쇄 삭제

### 입력 함정
- 어떤 문제는 `T`를 입력으로 주지 않고 **문제에서 10개라고만 말한 뒤** input.txt에 10줄이 들어있기도 함

### 스택 풀이(정석)
- 문자를 왼쪽부터 보면서
  - 스택 top과 같으면 pop(쌍 삭제)
  - 다르면 push

```python
# 비밀번호 - 스택 풀이
# 입력: 숫자 문자열 s
s = input().strip()

stack = []

# 문자열을 왼쪽부터 순회
for ch in s:
    # 스택이 비어있지 않고, top과 현재 문자가 같으면
    # -> 인접 쌍이므로 top을 제거(pop)하고 현재 문자는 넣지 않음
    if stack and stack[-1] == ch:
        stack.pop()
    else:
        # 다르면 현재 문자를 스택에 추가
        stack.append(ch)

# 스택에 남은 문자가 최종 비밀번호
print(''.join(stack))
```

---

## 4) 의석이의 세로로 말해요 (5줄 고정)

### 핵심
- 입력은 **항상 5줄**
- 각 줄 길이는 다를 수 있음
- 읽기는 **열(col) 기준으로 왼쪽부터**
- 특정 줄이 짧아 col 인덱스가 없으면 **건너뜀**

```python
# 의석이의 세로로 말해요
# 5개의 문자열을 입력
words = [input().rstrip() for _ in range(5)]

# 가장 긴 문자열 길이 계산
max_len = 0
for w in words:
    if len(w) > max_len:
        max_len = len(w)

result = []

# 열(col)을 0..max_len-1까지 순회
for col in range(max_len):
    # 행(row)은 0..4(항상 5개)
    for row in range(5):
        # 현재 col이 해당 단어 길이보다 작을 때만 접근 가능
        # (그 외는 글자가 없으므로 건너뜀)
        if col < len(words[row]):
            result.append(words[row][col])

print(''.join(result))
```

---

## 5) 벌꿀채취 (A형 도전용)

### 조건 정리(핵심만)
- `N×N` 벌통
- 일꾼 2명
- 각 일꾼은 **가로로 연속된 M칸** 선택(겹치면 안 됨)
- 각 일꾼은 선택한 M칸 중에서 **일부만 채취 가능**
  - 단, 채취한 꿀 합 ≤ `C`
- 수익 = 채취한 각 칸 꿀의 **제곱합**

### 풀이 구조(시험에서 안전한 방식)
1) 모든 가로 구간(길이 M)에 대해
   - 그 구간에서 만들 수 있는 **최대 수익(부분집합 중 합≤C인 것의 제곱합 최대)**을 미리 계산
2) 겹치지 않는 두 구간을 골라
   - (구간1 최대 수익 + 구간2 최대 수익)의 최댓값 갱신

### (핵심 함수) 한 구간의 최대 수익 계산
- 길이 M 리스트 `seg`에서
- 부분집합을 전부 보며(DFS/비트마스크)
  - 합이 C 이하면 제곱합 최대 갱신

```python
# 한 구간(seg)에서 얻을 수 있는 최대 수익을 계산
# seg: 길이 M의 꿀 양 리스트
# C: 채취 가능한 최대 꿀 양

def best_profit(seg, C):
    best = 0
    M = len(seg)

    # 부분집합 탐색(비트마스크)
    # mask의 i번째 비트가 1이면 seg[i]를 선택
    for mask in range(1 << M):
        total = 0
        profit = 0

        # 현재 부분집합의 합/제곱합 계산
        for i in range(M):
            if mask & (1 << i):
                total += seg[i]
                profit += seg[i] * seg[i]

        # 합 제한 C를 만족하면 최대 수익 갱신
        if total <= C:
            if profit > best:
                best = profit

    return best
```

### 전체 풀이(완성 형태)
```python
# 벌꿀채취 - 완전탐색 + 구간별 최대수익(부분집합)
# 입력: N(격자 크기), M(연속 선택 길이), C(최대 채취량)
N, M, C = map(int, input().split())
board = [list(map(int, input().split())) for _ in range(N)]

# 모든 가로 구간에 대해 최대 수익을 미리 계산해 둠
# profits[r][c] = (r행, c열 시작)에서 길이 M 구간의 최대 수익
profits = [[0] * (N - M + 1) for _ in range(N)]

for r in range(N):
    for c in range(N - M + 1):
        seg = board[r][c:c + M]           # 가로로 연속 M칸 구간
        profits[r][c] = best_profit(seg, C)  # 그 구간에서 만들 수 있는 최대 수익

answer = 0

# 1) 첫 번째 일꾼 구간 선택
for r1 in range(N):
    for c1 in range(N - M + 1):
        p1 = profits[r1][c1]

        # 2) 두 번째 일꾼 구간 선택
        #    중복 계산을 줄이기 위해 (r1,c1) 이후만 탐색하는 방식
        for r2 in range(r1, N):
            start_c = 0

            # 같은 행이면 겹치지 않도록 c2 시작점을 조정
            # (c1..c1+M-1)과 겹치면 안 되므로 같은 행에서는 c2 >= c1+M
            if r2 == r1:
                start_c = c1 + M

            for c2 in range(start_c, N - M + 1):
                p2 = profits[r2][c2]

                # 두 구간은 (같은 행일 때) 위 start_c 처리로 겹침을 방지함
                # 다른 행이면 애초에 겹칠 수 없음
                if p1 + p2 > answer:
                    answer = p1 + p2

print(answer)
```

---

## 오늘 실수 방지 체크리스트
- 입력 구조를 끝까지 확인: **N/M**, **정답지 vs 학생 답안**, **T가 실제로 입력으로 주어지는지**
- 범위: `N - M + 1` / `len(arr)-1` 같은 **끝 인덱스** 실수 주의
- 기본 테스트케이스만 믿지 말고 작은 반례를 직접 만들어보기

---

