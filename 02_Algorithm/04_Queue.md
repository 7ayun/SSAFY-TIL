# [Algorithm] 큐 — 선형 큐, 원형 큐, deque, 버퍼, 마이쮸·피자 굽기

> **핵심 키워드:** #큐 #FIFO #선입선출 #front #rear #enqueue #dequeue #선형큐 #원형큐 #모듈러연산 #버퍼 #deque #popleft #시간복잡도 #마이쮸 #피자굽기 #시뮬레이션

---

## 학습 목표

* 큐의 FIFO 구조를 이해하고 front·rear 포인터 기반으로 직접 구현
* 선형 큐의 한계(앞공간 낭비)를 파악하고, 원형 큐로 해결하는 원리 이해
* 원형 큐에서 모듈러(%) 연산으로 인덱스를 순환시키고, 한 칸 비우기로 빈 상태와 포화 상태를 구분
* `collections.deque`의 `popleft()`로 O(n²) → O(n) 개선 원리를 이해하고 실전에 적용
* 큐를 활용한 시뮬레이션 문제(마이쮸, 피자 굽기) 풀이 전략 습득

---

## 1. 큐의 구조와 작동 원리

### 1-1. 큐란

큐(Queue)는 스택과 마찬가지로 삽입과 삭제의 위치가 제한적인 자료구조다. **선입선출(FIFO, First In First Out)** — 가장 먼저 넣은 데이터를 가장 먼저 꺼낸다.

스택은 한쪽(top)에서만 삽입·삭제가 일어나지만, 큐는 **뒤에서 삽입하고 앞에서 삭제**한다. 일상에서의 예시로는 줄 서기, 은행 대기표, 편의점 음료 진열(뒤에서 넣고 앞에서 꺼냄) 등이 있다.

### 1-2. 큐의 주요 연산

| 연산 | 설명 |
|------|------|
| `enqueue` | 큐의 맨 뒤(rear)에 데이터 삽입 |
| `dequeue` | 큐의 맨 앞(front)에서 데이터를 꺼내고 반환 |
| `peek` | 맨 앞(front) 데이터를 제거하지 않고 반환만 |
| `isEmpty` | 큐가 비어 있는지 확인 |
| `isFull` | 큐가 가득 찼는지 확인 |

### 1-3. front와 rear 포인터

스택은 top 하나로 관리했지만, 큐는 삽입과 삭제가 일어나는 위치가 다르므로 **두 개의 포인터**가 필요하다. front는 꺼낼 위치(앞), rear는 넣을 위치(뒤)를 가리킨다.

> **강사님 강조**: 큐에서 집중해야 할 것은 두 가지뿐이다. **선입선출(FIFO)**과 **front·rear 포인터**. 이 두 가지만 지켜서 구현하면 어떤 방식이든 올바른 큐다.

---

## 2. 선형 큐 구현

### 2-1. 저수준 구현

```python
class Queue:
    def __init__(self, capacity=10):
        self.capacity = capacity
        self.data = [None] * capacity
        self.front = -1
        self.rear = -1

    def is_empty(self):
        return self.front == self.rear

    def is_full(self):
        return self.rear == self.capacity - 1

    def enqueue(self, item):
        if self.is_full():
            raise IndexError("Queue is full")
        self.rear += 1
        self.data[self.rear] = item

    def dequeue(self):
        if self.is_empty():
            raise IndexError("Queue is empty")
        self.front += 1
        item = self.data[self.front]
        self.data[self.front] = None
        return item

    def peek(self):
        if self.is_empty():
            raise IndexError("Queue is empty")
        return self.data[self.front + 1]
```

### 2-2. 선형 큐의 한계

enqueue를 반복하면 rear가 계속 뒤로 이동하고, dequeue를 반복하면 front도 뒤를 쫓아간다. rear가 배열 끝에 도달하면, 앞쪽에 빈 공간이 있어도 더 이상 삽입할 수 없다. 이것이 선형 큐의 핵심 문제점이다.

해결 방법으로 매번 앞으로 당기는 방식(`pop(0)`)이 있지만, 이는 O(n) 비용이 발생하여 비효율적이다.

> **강사님 강조**: `pop(0)`, `list.index()`, `list.find()` 같은 함수는 내부적으로 for문처럼 전체를 순회하므로 하나당 O(n)이다. while문 안에서 반복 호출하면 O(n²)이 되어 시간 초과의 원인이 된다.

---

## 3. 원형 큐

### 3-1. 핵심 아이디어 — 모듈러(%) 연산

원형 큐는 선형 큐의 앞공간 낭비 문제를 해결한다. 실제 배열은 1차원 그대로 사용하되, 인덱스를 이동할 때마다 **배열 크기로 나눈 나머지**를 인덱스로 사용한다. 이렇게 하면 인덱스가 배열 끝을 넘어갈 때 자동으로 0으로 돌아오면서 순환 구조가 만들어진다.

```
인덱스:  0  1  2  3  4  5  6  7  ...
% 4:    0  1  2  3  0  1  2  3  ...  → 0~3을 무한 순환
```

이 방식은 델타 탐색에서 방향을 순환할 때 `% 4`를 사용했던 것과 동일한 원리다.

### 3-2. 빈 상태와 포화 상태 구분

모듈러 연산으로 순환은 해결했지만 새로운 문제가 생긴다. front와 rear가 같은 위치에 있을 때 "비어 있는 것"인지 "가득 찬 것"인지 구분할 수 없다.

해결법은 **의도적으로 한 칸을 비워 두는 것**이다.

| 조건 | 의미 |
|------|------|
| `front == rear` | 큐가 비어 있음 |
| `(rear + 1) % size == front` | 큐가 가득 참 (한 칸 비워둠) |

10개의 데이터를 저장하고 싶으면 배열 크기를 11로 잡아서 한 칸을 비워 두는 것이다.

### 3-3. 원형 큐 구현

```python
class CircularQueue:
    def __init__(self, capacity=10):
        self.size = capacity + 1       # 한 칸을 비워 두기 위해 +1
        self.data = [None] * self.size
        self.front = 0
        self.rear = 0

    def is_empty(self):
        return self.front == self.rear

    def is_full(self):
        return (self.rear + 1) % self.size == self.front

    def enqueue(self, item):
        if self.is_full():
            raise IndexError("Queue is full")
        self.rear = (self.rear + 1) % self.size
        self.data[self.rear] = item

    def dequeue(self):
        if self.is_empty():
            raise IndexError("Queue is empty")
        self.front = (self.front + 1) % self.size
        item = self.data[self.front]
        self.data[self.front] = None
        return item
```

> **강사님 강조**: 원형 큐를 왜 쓰는지가 가장 중요하다. 선형 큐는 rear가 뒤로만 가니까 앞공간이 버려지고, 그걸 해결하려고 모듈러 연산으로 인덱스를 순환시키는 것이 원형 큐다. 빈 것과 찬 것을 구분하려면 한 칸을 비워 둔다. 이 흐름만 기억하면 된다.

---

## 4. 큐의 응용 — 버퍼

유튜브에서 영상 재생 시 데이터가 도착하는 대로 바로 보여주는 것이 아니라, 일정량을 모아 둔 뒤 순서대로 재생한다. 이 임시 저장 공간이 버퍼이며, 먼저 들어온 데이터를 먼저 내보내므로 전형적인 FIFO 구조다. 인터넷이 끊겨도 영상이 바로 멈추지 않는 이유가 버퍼에 10~15초 분량이 저장되어 있기 때문이다. 키보드 입력 버퍼도 동일한 원리다.

---

## 5. deque — 큐의 실전 도구

### 5-1. deque란

`collections.deque`는 Python 표준 라이브러리에서 제공하는 양방향 큐로, 내부적으로 **이중 연결 리스트**로 구현되어 있다. 앞에서 빼는 연산(`popleft`)이 O(1)이므로, `list.pop(0)`의 O(n)과 비교하면 큐 용도로 사용할 때 성능 차이가 크다.

### 5-2. 주요 메서드

| 메서드 | 설명 |
|--------|------|
| `append(x)` | 오른쪽(뒤)에 삽입 |
| `appendleft(x)` | 왼쪽(앞)에 삽입 |
| `pop()` | 오른쪽(뒤)에서 꺼냄 |
| `popleft()` | 왼쪽(앞)에서 꺼냄 — 핵심 |
| `rotate(n)` | 양수면 오른쪽, 음수면 왼쪽으로 회전 |

> **강사님 강조**: 큐를 써야 하는데 앞에서 데이터를 빼야 한다면 **무조건 deque**를 쓰자. A형 시험에서는 deque를 쓰지 않으면 시간 초과로 틀릴 정도로 의미 있는 차이다. deque에서 인덱스 접근(`deque[3]`)은 문법상 가능하지만 내부적으로 순차 탐색(O(n))이 일어나므로 주의해야 한다.

---

## 6. 큐 응용 문제 — 마이쮸

### 6-1. 문제 설명

마이쮸를 나눠주는 시뮬레이션 문제다. 사람이 마이쮸를 받으면 다시 줄을 서고 그 뒤에 새로운 사람이 줄을 서는 구조를 큐로 관리한다. 받을 때마다 받는 양이 1씩 누적되며, 마이쮸가 소진되는 시점에 마지막으로 받은 사람을 구한다.

### 6-2. list 버전 (기본)

```python
total_candy = 20
queue = []
next_person = 1
queue.append((next_person, 1))   # (번호, 받을 개수) — 1번이 1개 받겠다고 줄 서기
next_person += 1
last_person = None

while total_candy > 0:
    person, count = queue.pop(0)             # 맨 앞 사람 꺼내기

    if total_candy - count <= 0:             # 줬더니 캔디가 0 이하
        last_person = person                 # 이 사람이 마지막 마이쮸 주인
        break

    total_candy -= count                     # 캔디 나눠주기
    queue.append((person, count + 1))        # 받은 사람 다시 줄 서기 (개수 +1)
    queue.append((next_person, 1))           # 새로운 사람 1개 받겠다고 줄 서기
    next_person += 1

print(last_person)   # 2
```

> **강사님 강조**: 줄을 서고, 앞 차례가 되면 받고, 다시 뒤로 가서 줄 서는 문제가 나오면 기본적으로 큐를 떠올려야 한다. 변수 설계를 먼저 해야 한다 — 남은 캔디 수, 줄(큐), 마지막 사람, 새로 들어오는 번호. A4 용지에 논리를 다 세우고 나서 코드를 작성하자.

### 6-3. deque 버전 (개선)

6-2의 코드는 `queue.pop(0)`으로 맨 앞 사람을 꺼낸다. `pop(0)`은 호출할 때마다 뒤의 모든 원소를 한 칸씩 앞으로 당기므로 한 번에 O(n)이고, while문이 n번 도는 전체 루프에서는 **O(n²)** 이 된다. 수정은 딱 두 줄이면 충분하다.

```python
from collections import deque          # ① import 추가

total_candy = 20
queue = deque()                        # ② list → deque로 변경
next_person = 1
queue.append((next_person, 1))
next_person += 1
last_person = None

while total_candy > 0:
    person, count = queue.popleft()    # ③ pop(0) → popleft()  O(n) → O(1)

    if total_candy - count <= 0:
        last_person = person
        break

    total_candy -= count
    queue.append((person, count + 1))
    queue.append((next_person, 1))
    next_person += 1

print(last_person)   # 2
```

변경 포인트를 정리하면 다음과 같다.

| 변경 | 기존 (list) | 개선 (deque) | 효과 |
|------|------------|-------------|------|
| 자료구조 | `queue = []` | `queue = deque()` | 이중 연결 리스트 기반 |
| 앞에서 꺼내기 | `queue.pop(0)` — O(n) | `queue.popleft()` — O(1) | 매 꺼냄마다 n배 빠름 |
| 뒤에 넣기 | `queue.append()` — O(1) | `queue.append()` — O(1) | 동일 |
| 전체 루프 | O(n²) | **O(n)** | 데이터 커질수록 차이 극대화 |

나머지 로직(append, 조건문, 변수)은 **한 글자도 바꿀 필요 없다.** `deque`는 `append`를 그대로 지원하기 때문에, 기존 list 코드에서 선언부와 `pop(0)` → `popleft()`만 바꾸면 즉시 적용된다.

---

## 7. 큐 응용 문제 — 피자 굽기

화덕 크기 N, 피자 M개가 주어진다. 화덕에 N개를 넣고 순환하면서 한 바퀴 돌 때마다 치즈 양을 절반(정수 나눗셈)으로 줄인다. 치즈가 0이 되면 꺼내고, 대기 중인 피자를 넣는다. 마지막까지 남는 피자 번호를 구하는 문제다.

```python
from collections import deque

def pizza(n, cheeses):
    fire_q = deque()    # 화덕 (최대 n개)
    wait_q = deque()    # 대기 피자

    for i, c in enumerate(cheeses):
        if len(fire_q) < n:
            fire_q.append([i + 1, c])   # [번호, 치즈양]
        else:
            wait_q.append([i + 1, c])

    while len(fire_q) > 1:
        pizza = fire_q.popleft()        # 맨 앞 피자 꺼내기
        pizza[1] //= 2                  # 치즈 절반으로

        if pizza[1] > 0:
            fire_q.append(pizza)        # 아직 남았으면 다시 넣기
        elif wait_q:
            fire_q.append(wait_q.popleft())  # 빈자리에 대기 피자

    return fire_q[0][0]                 # 마지막 남은 피자 번호
```

> **강사님 팁**: 코드를 다 짜고 한 번에 실행하지 말고, 중간중간 print를 찍어서 "여기까지는 로직이 맞다"를 확인하며 진행해야 한다. 다 짜고 나서 틀리면 어디가 잘못됐는지 찾기가 매우 어렵다.

---

## 정리

| 구분 | 핵심 포인트 |
|------|-------------|
| 큐 | 선입선출(FIFO), front(앞)에서 삭제, rear(뒤)에서 삽입, 포인터 두 개로 관리 |
| 선형 큐의 한계 | rear가 끝에 도달하면 앞에 빈 공간이 있어도 삽입 불가 |
| 원형 큐 | `% size`로 인덱스 순환, 한 칸 비워서 빈 상태와 포화 상태 구분 |
| 버퍼 | 데이터를 임시 저장 후 순서대로 내보내는 FIFO 구조 (유튜브 버퍼링, 키보드 입력) |
| deque | `popleft()` O(1)로 큐 앞 추출, `pop(0)` O(n) 대비 성능 우위, A형 필수 |
| 마이쮸 개선 | `list` + `pop(0)` → `deque` + `popleft()`로 O(n²) → O(n), 변경 두 줄 |
| 피자 굽기 | 화덕(deque) + 대기(deque), 순환하며 치즈 절반 → 0이면 꺼내고 대기 피자 투입 |
