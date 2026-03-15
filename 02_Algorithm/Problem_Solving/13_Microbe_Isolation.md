# [Problem Solving] 미생물 격리

> **핵심 알고리즘:** #구현 #시뮬레이션 #딕셔너리 #defaultdict #델타탐색 #군집병합

| 항목 | 내용 |
|------|------|
| 출처 | SWEA |
| 핵심 유형 | 구현 / 시뮬레이션 |
| 관련 개념 | 02_Matrix, 04_Queue |

---

## 문제 요약

N×N 격자에서 K개의 미생물 군집이 매 시간마다 정해진 방향으로 한 칸씩 이동한다. 가장자리(약품 지대)에 도달하면 미생물 수가 절반(내림)으로 줄고 방향이 반대로 바뀐다. 수가 0이 되면 군집이 사라진다. 이동 후 같은 칸에 여러 군집이 모이면 수를 합산하고 가장 많은 수를 가진 군집의 방향을 따른다. M 시간 후 남아 있는 미생물의 총합을 구한다.

---

## 접근 방식

매 시간마다 세 단계를 반복하는 구조다.

1. **이동 단계**: 군집 리스트를 순회하며 각 방향에 따라 한 칸씩 이동한다. 델타 배열을 인덱스 1~4로 구성해 문제에서 주어지는 방향 번호를 그대로 사용한다.
2. **약품 지대 처리**: 이동 후 가장자리(x==0, x==N-1, y==0, y==N-1)이면 수를 //2로 줄이고 방향을 반대로 바꾼다. 수가 0이 되면 딕셔너리에 추가하지 않고 건너뛴다.
3. **군집 병합**: 이동 결과를 `defaultdict(list)`에 저장하고, 한 칸에 여러 군집이 있으면 수를 합산하고 가장 큰 군집의 방향을 선택한다.

> **강사님 강조**: 매 시간 처리 순서는 "이동 → 약품 지대 → 병합" 순서를 반드시 지켜야 한다.

> **강사님 팁**: 이동 결과를 리스트가 아닌 `defaultdict(list)`로 저장하면 키 초기화 없이 군집 정보를 바로 append할 수 있다.

> **강사님 주의**: 방향 반대 변환은 딕셔너리로 미리 준비한다. 1↔2, 3↔4.

---

## 풀이

이동 방향 1~4를 그대로 인덱스로 쓸 수 있도록 0번 자리는 더미로 채운다.

```python
from collections import defaultdict

N, M, K = map(int, input().split())
groups = [list(map(int, input().split())) for _ in range(K)]
# groups[i] = [x, y, 수, 방향]

dxy = [(0,0), (-1,0), (1,0), (0,-1), (0,1)]  # 0:더미, 1:상, 2:하, 3:좌, 4:우
opposite = {1:2, 2:1, 3:4, 4:3}

for _ in range(M):
    cell = defaultdict(list)

    for cx, cy, k, d in groups:
        nx = cx + dxy[d][0]
        ny = cy + dxy[d][1]

        # 약품 지대 처리
        if nx == 0 or nx == N-1 or ny == 0 or ny == N-1:
            k //= 2
            d = opposite[d]

        if k == 0:
            continue

        cell[(nx, ny)].append((k, d))

    # 군집 병합 후 groups 갱신
    groups = []
    for (x, y), lst in cell.items():
        if len(lst) == 1:
            groups.append([x, y, lst[0][0], lst[0][1]])
        else:
            total_k = sum(v[0] for v in lst)
            _, max_d = max(lst, key=lambda v: v[0])
            groups.append([x, y, total_k, max_d])

print(sum(g[2] for g in groups))
```

---

## 핵심 패턴

### 델타 배열 인덱스 일치

문제에서 방향을 1~4로 주면 델타 배열의 0번을 더미로 채워 인덱스를 맞춘다. 불필요한 -1 변환을 없애 코드가 간결해진다.

### defaultdict로 충돌 관리

이동 후 좌표를 key, 군집 정보 리스트를 value로 저장하면 같은 칸에 여러 군집이 모여도 자동으로 누적된다. 병합 여부는 `len(lst)`로 판단한다.

---

## 정리

| 구분 | 핵심 포인트 |
|------|-------------|
| 이동 | 델타 인덱스 = 방향 번호로 직접 매핑 |
| 약품 지대 | 수 //2, 방향 반전, 수==0이면 저장 스킵 |
| 병합 | defaultdict(list) → 수 합산 + 최대 수 방향 선택 |
| groups 갱신 | 매 시간마다 cell 딕셔너리 → groups 리스트 재생성 |
| 결과 | sum(g[2] for g in groups) |
