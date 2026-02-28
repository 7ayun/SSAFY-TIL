## [문제] 채점 시스템

> **활용 알고리즘:** `Simulation`, `1D_Array`
> **난이도:** ★★☆☆☆

### 🔍 문제 설명
* 정답지와 $N$명의 학생 답안지를 비교하여 점수를 매김
* 연속으로 정답을 맞출 경우 1점, 2점, 3점 식으로 가중치가 붙어 누적되며, 틀리면 다시 가중치가 0으로 초기화됨
* 학생들 중 최고 점수와 최저 점수의 차이를 구하기

### 💡 풀이 포인트
* **가중치 변수 분리:** 현재 학생의 총점을 저장할 `student_score`와 연속 정답 횟수를 저장할 `correct_count` 변수를 분리하여 관리
* **조건 분기:** 정답일 경우 `correct_count`를 1 증가시키고 그 값을 `student_score`에 더함. 오답일 경우 즉시 `correct_count`를 0으로 초기화
* **최대/최소 갱신:** 한 학생의 채점이 끝날 때마다 `max()`와 `min()` 함수를 활용하여 최고점과 최저점 갱신

### 💻 구현 코드 (Python)
```python
def solve_scoring(N, M, correct_sheet, student_sheets):
    max_score = 0
    min_score = float('inf') # 최저점 갱신을 위해 무한대로 초기화
    
    # 1. 각 학생의 답안지 순회
    for student_sheet in student_sheets:
        student_score = 0
        correct_count = 0
        
        # 2. M개의 문항에 대해 정답지와 비교
        for i in range(M):
            if student_sheet[i] == correct_sheet[i]:
                correct_count += 1        # 연속 정답 카운트 증가
                student_score += correct_count # 가중치가 반영된 점수 누적
            else:
                correct_count = 0         # 틀리면 즉시 가중치 초기화
                
        # 3. 한 학생의 채점이 끝나면 최대/최소 점수 갱신
        max_score = max(max_score, student_score)
        min_score = min(min_score, student_score)
        
    return max_score - min_score
```