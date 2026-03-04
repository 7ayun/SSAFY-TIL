# [Problem Solving] 채점 시스템

> **핵심 알고리즘:** #연속카운트 #max갱신 #min갱신 #2차원리스트 #입력처리

| 항목 | 내용 |
|------|------|
| 출처 | SWEA |
| 핵심 유형 | 1차원 리스트 순회, 조건 분기 |
| 관련 개념 | 01_List_and_Sorting |

---

## 문제 요약

학생 N명, 문항 수 M이 주어진다. 첫 줄에 정답지, 이후 N줄에 각 학생의 답안이 주어진다. 연속 정답 시 이전 점수 + 1점을 누적하고, 오답 시 0으로 초기화한다. 학생들의 최고점과 최저점의 차이를 출력한다.

---

## 접근 방식

먼저 입력 구조를 정확히 파악해야 한다. 첫 줄에 학생 수(N)와 문항 수(M), 그 다음 줄에 정답지, 이후 N줄에 각 학생의 답안이 주어진다. 정답지를 별도로 입력받은 뒤, 각 학생의 시험지를 순회하면서 정답지와 문항별로 비교한다.

연속 정답 횟수를 별도 변수(correct_count)로 관리하고, 정답이면 카운트를 올려 점수에 누적, 오답이면 카운트를 0으로 초기화한다. 한 학생의 모든 문항 채점이 끝나면 최고점/최저점을 갱신하고, 모든 학생 채점이 끝난 후 차이를 출력한다.

> **강사님 주의**: 이 문제에서 많이 실수하는 부분은 입력 처리다. 첫 줄이 정답지이고 그 다음 줄부터 학생 답안이라는 것을 놓치면 기본 테스트 케이스는 통과하더라도 숨겨진 케이스에서 틀린다. 기본 테스트 케이스를 너무 믿지 마라.

---

## 풀이

```python
T = int(input())
for tc in range(1, T + 1):
    N, M = map(int, input().split())
    correct_sheet = list(map(int, input().split()))  # 정답지 (별도 입력)
    student_st = [list(map(int, input().split())) for _ in range(N)]

    max_score = 0
    min_score = float('inf')

    for st_sheet in student_st:        # 각 학생의 시험지 순회
        student_score = 0              # 현재 학생의 총 점수
        correct_count = 0              # 연속 정답 카운트

        for i in range(M):             # M개 문항 채점
            if st_sheet[i] == correct_sheet[i]:
                correct_count += 1
                student_score += correct_count
            else:
                correct_count = 0      # 틀리면 연속 카운트 초기화

        max_score = max(max_score, student_score)
        min_score = min(min_score, student_score)

    result = max_score - min_score
    print(f'#{tc} {result}')
```

정답지를 학생 답안과 별도로 먼저 입력받는 것이 핵심이다. 정답지 없이 바로 학생 답안을 2차원 리스트로 받으면 정답지가 첫 번째 학생 답안으로 들어가 오답이 발생한다.

correct_count는 연속 정답 횟수를 의미하며, 정답이면 1씩 증가하고 student_score에 그대로 누적된다. 첫 정답이면 1, 두 번째 연속 정답이면 2, 세 번째면 3이 더해지는 구조다.

---

## 핵심 패턴

### 연속 카운트 + 누적

연속 조건을 만족하는 횟수를 별도 변수로 관리하고, 조건이 깨지면 0으로 초기화하는 패턴이다. 카운트 자체를 점수에 누적하면 "연속할수록 높은 점수"를 자연스럽게 구현할 수 있다.

### float('inf')를 활용한 최솟값 초기화

최솟값을 구할 때 초기값을 `float('inf')`로 설정하면 어떤 값이 들어와도 첫 비교에서 갱신된다. 최대값은 0, 최솟값은 무한대로 초기화하는 것이 안전한 관행이다.

---

## 정리

| 구분 | 핵심 포인트 |
|------|-------------|
| 입력 주의 | 정답지가 학생 답안과 별도 줄로 주어짐. 놓치면 오답 |
| 연속 카운트 | 정답이면 +1 누적, 오답이면 0 초기화 |
| 갱신 위치 | max/min 갱신은 한 학생의 모든 문항 채점이 끝난 후 |
| 초기값 | max_score = 0, min_score = float('inf') |
