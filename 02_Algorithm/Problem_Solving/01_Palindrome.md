## [문제] 초심자의 회문 검사 (Palindrome)

> **활용 알고리즘:** `Two_Pointers`, `Recursion`
> **난이도:** ★☆☆☆☆

### 🔍 문제 설명
* 주어진 단어가 거꾸로 읽어도 똑같은 회문(Palindrome)인지 판별하여 맞으면 1, 틀리면 0을 반환

### 💡 풀이 포인트
* **슬라이싱 기법:** `word == word[::-1]`을 활용한 파이썬 특화 초간단 판별법
* **투 포인터 기법:** 문자열의 양끝(Front, Rear) 인덱스를 설정하고 중앙으로 좁혀가며 비교. 길이가 $N$일 때 판별 횟수는 $N // 2$번으로 충분함
* **재귀(Recursion) 기법:** `left`와 `right` 인덱스를 파라미터로 넘겨 스스로를 호출. `left >= right`가 교차하는 순간을 펠린드롬 성공(종료 조건)으로 판별

### 💻 구현 코드 (Python)
```python
# 1. [투 포인터 방식]
def check_palindrome_iterative(word):
    n = len(word)
    # 문자열 길이의 절반만 순회
    for front in range(n // 2):
        rear = n - 1 - front
        # 앞뒤 문자가 다르면 즉시 0 반환
        if word[front] != word[rear]:
            return 0
    return 1

# 2. [재귀 방식]
def check_palindrome_recursive(word, left, right):
    # 종료 조건: 포인터가 교차하거나 같아지면 회문 인정
    if left >= right:
        return 1
    # 다르면 즉시 0 반환
    if word[left] != word[right]:
        return 0
    # 양끝을 한 칸씩 좁히며 재귀 호출
    return check_palindrome_recursive(word, left + 1, right - 1)
```