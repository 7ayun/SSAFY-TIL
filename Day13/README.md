# Queue(큐) 자료구조 학습정리

---

## 1. 오늘 수업의 큰 흐름

1. 시험 피드백 및 IM 대비 마인드
2. 스택 복습
3. 큐(Queue) 개념 학습
4. 선형 큐 구현
5. 큐의 한계 이해
6. 마이쮸 시뮬레이션 문제
7. 원형 큐(Circular Queue) 개념 및 구현

---

## 2. 스택 복습 핵심

### 스택 특징
- 후입선출 (LIFO: Last In First Out)
- 가장 나중에 들어간 데이터가 가장 먼저 나온다.

### 핵심 연산
- push: top을 증가시키고 값 삽입
- pop: top 위치의 값을 꺼내고 top 감소

---

## 3. 큐(Queue) 개념

### 큐의 특징
- 선입선출 (FIFO: First In First Out)
- 먼저 들어온 데이터가 먼저 나간다.

### 실생활 예시
- 은행 대기 줄
- 프린터 출력 대기열
- 유튜브 버퍼링

---

## 4. 큐의 핵심 구조

큐는 삽입과 삭제 위치가 다르다.

| 동작 | 위치 |
|------|------|
| 삽입(enqueue) | 뒤(rear) |
| 삭제(dequeue) | 앞(front) |

따라서 포인터가 2개 필요하다.
- front: 삭제 위치 관리
- rear: 삽입 위치 관리

---

## 5. 선형 큐(Linear Queue) 구현

### 특징
- front와 rear를 이용해 구현
- 초기값: front = -1, rear = -1

### 상태 판단
- 비어있음: front == rear
- 가득참: rear == capacity - 1

---

### 선형 큐 코드

```python
class Queue:
    def __init__(self, capacity=10):
        # 큐의 최대 크기
        self.capacity = capacity

        # 실제 데이터를 저장할 리스트
        self.items = [None] * capacity

        # front: 마지막으로 삭제된 위치
        # rear: 마지막으로 삽입된 위치
        self.front = -1
        self.rear = -1

    def is_full(self):
        # rear가 마지막 인덱스를 가리키면 가득 찬 상태
        return self.rear == self.capacity - 1

    def is_empty(self):
        # front와 rear가 같으면 비어있는 상태
        return self.front == self.rear

    def enqueue(self, item):
        # 삽입 전, 가득 찼는지 확인
        if self.is_full():
            raise IndexError("Queue is full")

        # rear를 한 칸 이동 후 데이터 삽입
        self.rear += 1
        self.items[self.rear] = item

    def dequeue(self):
        # 삭제 전, 비어있는지 확인
        if self.is_empty():
            raise IndexError("Queue is empty")

        # front를 한 칸 이동 (실제 데이터 위치로 이동)
        self.front += 1

        # 해당 위치의 데이터를 꺼냄
        item = self.items[self.front]

        # 시각적 확인을 위해 삭제 처리
        self.items[self.front] = None

        return item

    def peek(self):
        if self.is_empty():
            raise IndexError("Queue is empty")

        # front 다음 칸이 실제 첫 데이터
        return self.items[self.front + 1]

    def size(self):
        # 현재 들어있는 데이터 개수
        return self.rear - self.front
```

---

## 6. 선형 큐의 문제점

### 문제 상황
- dequeue를 여러 번 수행하면
- 앞쪽 공간이 비게 된다.
- 하지만 rear는 계속 뒤로 이동한다.

결과:
- 배열 앞쪽이 비어 있어도
- rear가 끝에 도달하면 enqueue 불가능

→ 공간 낭비 발생

---

## 7. 마이쮸 문제 (큐 시뮬레이션)

### 문제 핵심 규칙
1. 처음에 1번이 1개를 받는다.
2. 누군가 마이쮸를 받으면:
   - 그 사람은 (받은 개수 +1)로 다시 줄 선다.
   - 새로운 사람이 1개 받겠다고 줄 선다.
3. 마지막 마이쮸를 가져가는 사람을 구한다.

---

### 마이쮸 코드

```python
def last_person_mycandy(total_candy=20):
    # 줄 서 있는 사람들 (큐 역할)
    q = []

    # 새로 들어오는 사람 번호
    next_person = 1

    # 처음에는 1번이 1개 받겠다고 줄을 선다
    q.append((next_person, 1))

    # 마지막 마이쮸를 가져간 사람
    last_person = 0

    # 마이쮸가 남아 있는 동안 반복
    while total_candy > 0:

        # 줄 맨 앞 사람 꺼냄
        person, count = q.pop(0)

        # 이번에 주려는 개수만큼 줬더니 끝나면
        # 이 사람이 마지막 마이쮸를 가져감
        if total_candy - count <= 0:
            last_person = person
            break

        # 마이쮸 지급
        total_candy -= count

        # 받은 사람은 다음에 +1개 받겠다고 다시 줄 선다
        q.append((person, count + 1))

        # 새로운 사람이 1개 받겠다고 줄 선다
        next_person += 1
        q.append((next_person, 1))

    return last_person
```

---

## 8. 원형 큐(Circular Queue)

### 선형 큐 문제 해결 아이디어
- 인덱스를 순환시키면 앞 공간을 재사용 가능
- 방법: `(index + 1) % capacity`

### 핵심 규칙 (한 칸 비우기)
- 비어 있음: front == rear
- 가득 참: (rear + 1) % capacity == front

---

### 원형 큐 코드

```python
class CircularQueue:
    def __init__(self, capacity=10):
        # 원형 큐는 한 칸을 비워두기 때문에 +1
        self.capacity = capacity + 1
        self.items = [None] * self.capacity

        # front: 마지막으로 삭제된 위치
        # rear: 마지막으로 삽입된 위치
        self.front = 0
        self.rear = 0

    def is_empty(self):
        return self.front == self.rear

    def is_full(self):
        return (self.rear + 1) % self.capacity == self.front

    def enqueue(self, item):
        if self.is_full():
            raise IndexError("Queue is full")

        self.rear = (self.rear + 1) % self.capacity
        self.items[self.rear] = item

    def dequeue(self):
        if self.is_empty():
            raise IndexError("Queue is empty")

        self.front = (self.front + 1) % self.capacity
        item = self.items[self.front]
        self.items[self.front] = None
        return item

    def peek(self):
        if self.is_empty():
            raise IndexError("Queue is empty")

        return self.items[(self.front + 1) % self.capacity]

    def size(self):
        return (self.rear - self.front + self.capacity) % self.capacity
```

---

## 9. 오늘 핵심 정리

- 스택은 LIFO 구조이며 top 하나로 관리한다.
- 큐는 FIFO 구조이며 front와 rear 두 포인터가 필요하다.
- 선형 큐는 앞 공간이 비어도 재사용하지 못하는 문제가 있다.
- 원형 큐는 인덱스를 순환시켜 공간 낭비를 해결한다.
- 원형 큐의 상태 조건
  - 비어 있음: front == rear
  - 가득 참: (rear + 1) % capacity == front

---

## 10. 다음 수업 예고

- 연결 리스트(Linked List)
- deque 구조
- 이후: 그리디, 완전탐색 등 본격 알고리즘 진입

