# [Problem Solving] 초심자의 회문 검사

> **핵심 알고리즘:** #문자열 #투포인터 #Palindrome #재귀 #Recursion #strip #인덱스비교

| 항목 | 내용 |
|------|------|
| 출처 | SWEA |
| 핵심 유형 | 1차원 리스트, 문자열 비교 |
| 관련 개념 | 01_List_and_Sorting, 09_DFS |

---

## 문제 요약

단어를 입력받아 거꾸로 읽어도 원래와 같은 회문(Palindrome)이면 1, 아니면 0을 출력한다. 예를 들어 "level"은 거꾸로 읽어도 "level"이므로 회문이다.

---

## 접근 방식

슬라이싱(`word[::-1]`)으로 뒤집어서 비교하면 한 줄에 해결되지만, 학습 목적으로 인덱스를 활용한 풀이를 먼저 익힌다.

핵심 아이디어는 문자열의 앞(front)과 뒤(rear)에서 동시에 출발하여 가운데로 좁혀오면서 비교하는 것이다. front 인덱스가 0일 때 rear 인덱스는 `n - 1 - 0`, front가 1이면 rear는 `n - 1 - 1`이 된다. 비교 횟수는 문자열 길이의 절반이면 충분하다.

중간에 하나라도 다르면 회문이 아니므로 즉시 break할 수 있다. 이 방식은 슬라이싱보다 유리한 점이 있는데, 10만 글자에서 첫 글자와 마지막 글자가 다르면 비교 1회로 끝나기 때문이다.

---

## 풀이 1 — 반복문 (투 포인터)

```python
T = int(input())
for tc in range(1, T + 1):
    word = input().strip()  # 양쪽 공백 제거 필수
    n = len(word)
    result = 1  # 회문이라고 가정하고 시작

    for front_idx in range(n // 2):
        rear_idx = n - 1 - front_idx

        if word[front_idx] != word[rear_idx]:
            result = 0  # 하나라도 다르면 회문 아님
            break        # 더 비교할 필요 없음

    print(f'#{tc} {result}')
```

`result`를 1로 초기화해 놓고, 다른 글자를 발견했을 때만 0으로 바꾸는 방식이다. `input().strip()`은 SWEA 입력에 간혹 포함되는 양쪽 공백을 제거하기 위해 습관적으로 붙인다.

> **강사님 주의**: SWEA 문제에서 문자열 입력 시 양쪽에 공백이 포함되어 있는 경우가 있다. 논리가 완벽한데 안 되면 strip()을 확인하라.

---

## 풀이 2 — 재귀

```python
def is_palindrome(word, left, right):
    # 종료 조건: left가 right 이상이면 모든 비교 완료
    if left >= right:
        return 1

    # 다르면 회문 아님
    if word[left] != word[right]:
        return 0

    # 범위를 좁혀서 재귀 호출
    return is_palindrome(word, left + 1, right - 1)

T = int(input())
for tc in range(1, T + 1):
    word = input().strip()
    result = is_palindrome(word, 0, len(word) - 1)
    print(f'#{tc} {result}')
```

재귀 함수의 매개변수 설계가 핵심이다. word(검사 대상), left(앞 인덱스), right(뒤 인덱스) 세 가지를 넘긴다. 매 호출마다 left는 +1, right는 -1로 범위를 좁히며, left >= right가 되면 모든 비교를 통과한 것이므로 1을 반환한다.

> **강사님 강조**: 재귀 함수를 쓸 때 반드시 종료 조건을 설정하라. 범위가 점점 줄어드는 형태로 설계해야 무한 호출을 피할 수 있다. 재귀는 A형 시험에서 거의 필수로 사용되므로 반드시 이해해야 한다.

---

## 핵심 패턴

### 투 포인터 비교

front와 rear 인덱스를 양쪽 끝에서 출발시켜 가운데로 좁혀오는 패턴이다. 회문 검사 외에도 정렬된 배열에서 두 수의 합을 찾는 문제 등에 활용된다. rear 인덱스 공식은 `n - 1 - front`이며, 반복 횟수는 `n // 2`로 충분하다.

### 문제 풀이 전 그림 그리기

> **강사님 강조**: 바로 코드로 손이 가면 아무것도 못 푼다. 예시를 적어보고 인덱스의 규칙을 찾은 뒤 코드를 작성하라.

---

## 정리

| 구분 | 핵심 포인트 |
|------|-------------|
| 투 포인터 | front(0→)와 rear(n-1←)를 가운데로 좁히며 비교. 비교 횟수는 n // 2 |
| 재귀 풀이 | left, right를 매개변수로 넘기고 범위를 줄여가며 호출. left >= right가 종료 조건 |
| strip() | SWEA 문자열 입력 시 양쪽 공백 제거 습관화 |
| 성능 비교 | 슬라이싱은 전체를 뒤집고, 투 포인터는 불일치 시 즉시 종료 가능 |
