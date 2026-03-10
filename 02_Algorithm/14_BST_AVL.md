# [Algorithm] 이진 탐색 트리 — BST, 자가균형트리, AVL

> **핵심 키워드:** #BST #이진탐색트리 #순서속성 #탐색 #삽입 #삭제 #중위후속자 #편향트리 #자가균형트리 #AVL #균형인수 #LL회전 #RR회전 #LR회전 #RL회전 #레드블랙트리

---

## 학습 목표

* BST의 등장 배경과 순서 속성을 설명할 수 있다
* BST의 탐색, 삽입, 삭제 알고리즘을 구현할 수 있다
* BST 편향 트리 문제를 이해하고 자가균형트리의 필요성을 설명할 수 있다
* AVL 트리의 균형 인수와 4가지 회전 케이스를 구분할 수 있다
* AVL 트리와 레드블랙트리의 차이를 설명할 수 있다

---

## 1. BST 등장 배경

### 1-1. 기존 자료구조의 한계

데이터가 많아질수록 탐색 속도가 핵심이다. 그런데 기존 자료구조는 한계가 있다.

| 자료구조 | 탐색 | 삽입/삭제 | 문제점 |
|---------|------|----------|-------|
| 배열 | O(N) | O(N) | 중간 삽입/삭제 시 전체를 밀거나 당겨야 함 |
| 연결 리스트 | O(N) | O(1) | 인덱스 접근 불가, 처음부터 순차 탐색 |
| **BST** | **O(log N)** | **O(log N)** | 균형 깨지면 O(N) |

> **강사님 강조**: 데이터가 10억 개여도 BST는 단 30번 만에 찾을 수 있다. 2의 30승이 약 10억이기 때문이다.

### 1-2. BST란

BST(Binary Search Tree, 이진 탐색 트리)는 데이터의 저장, 검색, 삽입, 삭제를 효율적으로 처리하기 위한 자료구조다.

BST가 되려면 아래 2가지 조건을 반드시 만족해야 한다.

```
조건 1: 각 노드는 최대 2개의 자식을 가진다 (이진 트리)
조건 2: 순서 속성 — 왼쪽 자식 < 부모 노드 < 오른쪽 자식
```

```
        10
       /  \
      5    15
     / \   / \
    3   7  12  17
```

이 트리는 두 조건을 모두 만족한다. 10 기준으로 왼쪽(5, 3, 7) 전부 < 10, 오른쪽(15, 12, 17) 전부 > 10.

> **강사님 강조**: BST에서 중복 값은 허용하지 않는 것이 일반적이다. 굳이 중복을 허용하려면 노드에 개수를 따로 저장하거나 별도 규칙을 정의해야 한다.

---

## 2. BST 탐색

### 2-1. 탐색 원리

순서 속성 덕분에 루트에서 시작해 키값을 비교하며 범위를 절반씩 줄여나간다.

1. 루트 노드에서 시작
2. 찾는 값이 현재 노드보다 작으면 왼쪽 자식으로 이동
3. 찾는 값이 현재 노드보다 크면 오른쪽 자식으로 이동
4. 같으면 탐색 성공, None이면 탐색 실패

```
12를 찾는 경우:

        10
       /  \
      5    15
          /
         12

① 12 > 10 → 오른쪽
② 12 < 15 → 왼쪽
③ 12 == 12 → 탐색 성공
```

### 2-2. 탐색 코드

외부에서 호출하는 `search`와 내부 재귀 로직을 담당하는 `_search`를 분리한다.

```python
class BSTNode:
    def __init__(self, key):
        self.key = key
        self.left = None
        self.right = None

class BST:
    def __init__(self):
        self.root = None

    def search(self, key):
        return self._search(self.root, key)

    def _search(self, node, key):
        if node is None or node.key == key:
            return node
        if key < node.key:
            return self._search(node.left, key)
        else:
            return self._search(node.right, key)
```

> **강사님 팁**: 언더바(`_`)로 시작하는 메서드는 내부 구현용이라는 관례적 표시다. Python은 private/public 구분이 없어서 이렇게 약속으로 구분한다. 현업에서는 이런 식으로 많이 작성한다.

### 2-3. 시간 복잡도

| 상태 | 시간 복잡도 |
|------|-----------|
| 균형 잡힌 BST | O(log N) |
| 편향된 BST (최악) | O(N) |

---

## 3. BST 삽입

### 3-1. 삽입 원리

삽입도 탐색과 동일하게 진행한다. 다만 None인 자리를 만나는 순간 그 자리가 새 노드의 위치다.

```
16 삽입:

        10
       /  \
      5    15
          /  \
         12   17
              /
            (16 삽입 위치)

① 16 > 10 → 오른쪽
② 16 > 15 → 오른쪽
③ 16 < 17 → 왼쪽
④ 왼쪽이 None → 여기가 내 자리
```

> **강사님 주의**: 1, 2, 3, 4, 5 순서로 삽입하면 완전히 오른쪽으로 편향된 트리가 만들어진다. 현실의 데이터는 은근히 정렬된 상태가 많아서 편향 문제가 자주 발생한다.

### 3-2. 삽입 코드

```python
def insert(self, key):
    if self.root is None:
        self.root = BSTNode(key)
    else:
        self._insert(self.root, key)

def _insert(self, node, key):
    if key < node.key:
        if node.left is None:
            node.left = BSTNode(key)
        else:
            self._insert(node.left, key)
    else:
        if node.right is None:
            node.right = BSTNode(key)
        else:
            self._insert(node.right, key)
```

---

## 4. BST 삭제

### 4-1. 3가지 케이스

삭제는 대상 노드의 자식 수에 따라 처리 방법이 달라진다.

**Case 1: 자식이 없는 경우 (리프 노드)**

그냥 삭제하면 된다. 순서 속성이 자동으로 유지된다.

```
5를 삭제:
    10              10
   /  \    →      /  \
  5    15         3    15
 /
3
```

**Case 2: 자식이 1개인 경우**

삭제할 노드의 자식을 삭제할 노드의 부모에 직접 연결한다.

```
5를 삭제 (자식이 7 하나):
    10              10
   /  \    →      /  \
  5    15         7    15
   \
    7
```

순서 속성상 5의 자식(7)은 무조건 5의 부모(10)보다 작으므로, 7을 10의 왼쪽 자식으로 연결해도 순서 속성이 유지된다.

**Case 3: 자식이 2개인 경우**

중위 후속자(오른쪽 서브트리에서 가장 작은 값)로 대체한다.

```
15를 삭제 (중위 후속자 = 16):

        10                    10
       /  \                  /  \
      5    15    →           5    16
          /  \                   /  \
         12   17                 12   17
             /
            16 (중위 후속자)
```

### 4-2. 삭제 코드

```python
def delete(self, key):
    self.root = self._delete(self.root, key)

def _delete(self, node, key):
    if node is None:
        return node

    if key < node.key:
        node.left = self._delete(node.left, key)
    elif key > node.key:
        node.right = self._delete(node.right, key)
    else:
        # Case 1, 2: 자식이 0개 또는 1개
        if node.left is None:
            return node.right
        elif node.right is None:
            return node.left

        # Case 3: 자식이 2개 → 중위 후속자 찾기
        temp = node.right
        while temp.left is not None:
            temp = temp.left

        node.key = temp.key
        node.right = self._delete(node.right, temp.key)

    return node
```

> **강사님 강조**: 코드 구현보다 원리가 중요하다. "오른쪽 서브트리에서 가장 작은 값을 가져와서 대체하고, 그 자리는 기존 삭제 규칙으로 처리한다"는 흐름만 이해하면 된다.

---

## 5. BST 한계와 자가균형트리

### 5-1. 편향 트리 문제

BST의 구조는 데이터 삽입 순서에 따라 결정된다. 정렬된 데이터를 순서대로 삽입하면 최악의 편향 트리가 된다.

```
1, 2, 3, 4, 5 순서로 삽입:

1
 \
  2
   \
    3         → 그냥 연결 리스트와 다름없다. O(N)
     \
      4
       \
        5
```

### 5-2. 자가균형트리

편향 문제를 해결하기 위해 등장한 것이 자가균형트리다. BST의 모든 속성을 유지하면서 균형까지 자동으로 잡는다.

| 자료구조 | 설명 | 활용 |
|---------|------|------|
| **AVL 트리** | 높이 차를 최대 1로 유지. 엄격한 균형 | 탐색이 잦은 경우 |
| **레드블랙트리** | 색상 개념으로 유연하게 균형 유지 | C++ STL, Java TreeMap |

> **강사님 강조**: AVL과 레드블랙트리 모두 BST의 상위호환이다. AVL의 원리를 이해하면 레드블랙트리는 거기에 색상 규칙만 추가한 것이라 자연스럽게 이해된다.

---

## 6. AVL 트리

### 6-1. 균형 인수

AVL 트리의 핵심은 각 노드마다 균형 인수(Balance Factor)를 관리하는 것이다.

```
균형 인수 = 왼쪽 서브트리 높이 - 오른쪽 서브트리 높이

허용 범위: -1, 0, 1
위반 범위: ±2 이상 → 즉시 회전으로 복구
```

> **강사님 강조**: 균형 인수가 ±2 이상이 되는 순간 회전이 발동한다. 이것만 기억하면 된다.

### 6-2. 4가지 회전 케이스

불균형 패턴에 따라 회전 방식이 달라진다. LL/RR 2개만 구현하면 LR/RL은 이 둘의 조합으로 해결된다.

| 케이스 | 발생 조건 | 처리 방법 |
|--------|----------|----------|
| **LL** | 부모 균형인수 +2, 왼쪽 자식 균형인수 +1 or 0 | 오른쪽 회전 1번 |
| **RR** | 부모 균형인수 -2, 오른쪽 자식 균형인수 -1 or 0 | 왼쪽 회전 1번 |
| **LR** | 부모 균형인수 +2, 왼쪽 자식 균형인수 -1 | 왼쪽 자식에 RR → 부모에 LL |
| **RL** | 부모 균형인수 -2, 오른쪽 자식 균형인수 +1 | 오른쪽 자식에 LL → 부모에 RR |

LL 회전 예시:

```
삽입 후 불균형:    LL 회전 후:
      9                7
     /                / \
    7        →        5   9
   /
  5
```

LR 회전 예시:

```
삽입 후:     1단계 RR:    2단계 LL:
   9              9             8
  /              /             / \
 7       →      8      →      7   9
  \            /
   8           7
```

### 6-3. AVL 트리 코드

```python
class AVLNode:
    def __init__(self, key):
        self.key = key
        self.left = None
        self.right = None
        self.height = 1

class AVLTree:
    def get_height(self, node):
        return node.height if node else 0

    def get_balance(self, node):
        return self.get_height(node.left) - self.get_height(node.right) if node else 0

    def rotate_right(self, y):          # LL 케이스
        x = y.left
        y.left = x.right
        x.right = y
        y.height = 1 + max(self.get_height(y.left), self.get_height(y.right))
        x.height = 1 + max(self.get_height(x.left), self.get_height(x.right))
        return x

    def rotate_left(self, x):           # RR 케이스
        y = x.right
        x.right = y.left
        y.left = x
        x.height = 1 + max(self.get_height(x.left), self.get_height(x.right))
        y.height = 1 + max(self.get_height(y.left), self.get_height(y.right))
        return y

    def insert(self, node, key):
        # 1단계: 일반 BST 삽입
        if not node:
            return AVLNode(key)
        if key < node.key:
            node.left = self.insert(node.left, key)
        else:
            node.right = self.insert(node.right, key)

        # 2단계: 높이 갱신
        node.height = 1 + max(self.get_height(node.left), self.get_height(node.right))

        # 3단계: 균형 인수 확인 및 회전
        balance = self.get_balance(node)

        if balance > 1 and key < node.left.key:      # LL
            return self.rotate_right(node)
        if balance < -1 and key > node.right.key:    # RR
            return self.rotate_left(node)
        if balance > 1 and key > node.left.key:      # LR
            node.left = self.rotate_left(node.left)
            return self.rotate_right(node)
        if balance < -1 and key < node.right.key:    # RL
            node.right = self.rotate_right(node.right)
            return self.rotate_left(node)

        return node
```

> **강사님 팁**: 회전 케이스를 외울 필요 없다. 균형 인수 조건을 if문으로 분기시켜놓으면 자동으로 돌아간다.

---

## 7. 참고 — AVL vs 레드블랙트리 (읽을거리)

레드블랙트리는 AVL에 색상 규칙을 추가해 삽입/삭제 시 회전 빈도를 줄인 자료구조다.

| 구분 | AVL 트리 | 레드블랙트리 |
|------|---------|------------|
| 균형 기준 | 엄격 (균형인수 ±1) | 유연 (색상 규칙 기반) |
| 탐색 성능 | 더 빠름 | 상대적으로 느림 |
| 삽입/삭제 성능 | 회전이 자주 발생 | 회전이 적어 더 빠름 |
| 활용 | 탐색 중심 시스템 | C++ STL, Java TreeMap, Python dict |

---

## 정리

| 구분 | 핵심 포인트 |
|------|------------|
| BST 2가지 규칙 | 최대 2개 자식 + 순서 속성 (왼쪽 < 부모 < 오른쪽) |
| 탐색/삽입 | 루트부터 순서 속성 따라 이동, None 만나면 종료 |
| 삭제 Case 3 | 오른쪽 서브트리의 최솟값(중위 후속자)으로 대체 후 삭제 |
| BST 한계 | 정렬된 데이터 삽입 시 편향 트리 → O(N) |
| AVL 균형 인수 | 왼쪽 높이 - 오른쪽 높이, 절댓값 ≤ 1 항상 유지 |
| 회전 조합 | LL/RR 기본, LR = RR 후 LL, RL = LL 후 RR |
