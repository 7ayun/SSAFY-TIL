## [문제] 의석이의 세로로 말해요

> **활용 알고리즘:** `2D_Array_Traversal`
> **난이도:** ★★☆☆☆

### 🔍 문제 설명
* 5줄의 영어/숫자 문자열이 주어짐. 각 줄의 길이는 들쭉날쭉함
* 이 문자들을 열(Column) 방향으로 위에서 아래로, 왼쪽 열부터 오른쪽 열 순서대로 읽어서 하나의 문자열로 출력하기. (빈 공간은 건너뜀)

### 💡 풀이 포인트
* **최대 길이 확보:** 5개의 문자열 중 가장 긴 문자열의 길이(`max_length`)를 먼저 구하여 열 반복문의 한계값으로 설정
* **인덱스 보호 (안전장치):** 열을 고정(`col`)하고 5개의 행(`row`)을 순회할 때, 현재 행의 문자열 길이(`len(word)`)가 `col`보다 클 때만 접근해야 `IndexError`를 방지할 수 있음

### 💻 구현 코드 (Python)
```python
def read_vertically(words):
    # 1. 5개 단어 중 가장 긴 문자열의 길이 구하기
    max_length = 0
    for word in words:
        max_length = max(max_length, len(word))
        
    result = []
    
    # 2. 열(Column)을 먼저 고정
    for col in range(max_length):
        # 3. 행(Row)을 순회하며 글자 추출 (5줄 고정)
        for row in range(5):
            # 4. 현재 접근하려는 행의 단어 길이가 col보다 커야만 인덱스 접근 가능
            if len(words[row]) > col:
                result.append(words[row][col])
                
    return ''.join(result)
```