# 📝 Day10 문제풀이 복습 노트

> **목표**: 코드를 직접 완성하며 개념을 점검한다.  
> 빈칸을 채우면서 *손으로 로직을 복기*할 것.

---

## 1️⃣ 회문 검사 (Palindrome)

### ✔️ 구현 흐름 채우기

- 문자열 입력 시 반드시 처리해야 하는 것: `__________`
- 비교는 문자열의 절반만 수행한다 → 이유: `__________`

```python
word = input().strip()
n = len(word)
result = 1

for i in range( ________ ):
    if word[i] != word[ ________ ]:
        result = 0
        ________

print(result)
```

### ❗ 체크 포인트
- 뒤쪽 인덱스 공식: `n - 1 - i`
- 하나라도 다르면 바로 종료

---

## 2️⃣ 파리 퇴치 (Fly Swatter)

### ✔️ 시작점 범위 채우기

```python
for i in range( __________ ):
    for j in range( __________ ):
        temp_sum = 0
        for x in range(M):
            for y in range(M):
                temp_sum += grid[i+x][j+y]

        max_kill = max(max_kill, temp_sum)
```

### ❗ 체크 포인트
- `temp_sum` 초기화 위치는 `(i, j)` 내부
- 시작점 범위는 반드시 `N-M+1`

---

## 3️⃣ 풍선팡 1 (Balloon Pang 1)

### ✔️ 델타 + 거리 확장 로직

```python
sum_value = arr[i][j]

for dx, dy in dxy:
    for dist in range( __________ ):
        ni = i + dx * dist
        nj = j + dy * dist

        if ni < 0 or ni >= N or nj < 0 or nj >= N:
            ________

        sum_value += arr[ni][nj]
```

### ❗ 체크 포인트
- 거리 범위는 `1 ~ K`
- 범위 벗어나면 해당 방향 즉시 종료
- 본인 값은 이미 포함되어 있음

---

## 4️⃣ Ladder1 (사다리)

### ✔️ 테스트케이스 반복 구조

```python
for _ in range( ____ ):
    tc = int(input())
    data = [list(map(int, input().split())) for _ in range( ____ )]
```

### ✔️ visited 배열 생성

```python
visited = [[0]*N for _ in range(N)]
```

### ✔️ 이동 조건 점검 (Early Continue)

```python
if nx < 0 or nx >= N or ny < 0 or ny >= N:
    continue
if data[nx][ny] != 1:
    continue
if visited[nx][ny] == 1:
    continue
```

### ❗ 체크 포인트
- `[[0]*N]*N` 사용 ❌
- while 종료 조건 명확히 설정
- 원본 배열 수정 금지

---

## 🧠 시험 직전 자가 점검

- 회문: `strip()` 했는가?
- 파리퇴치: 시작점 범위 맞는가?
- 풍선팡1: dist 시작값 1인가?
- Ladder1: TC=10 고정 기억했는가?

---

✍️ **이 노트는 반드시 직접 채워서 완성할 것.**

### 이동
👉 [학습 정리](./README.md)
👉 [메인 README](../README.md)