# [Algorithm] 스택 (Stack)

> **핵심 키워드:** #LIFO #Top_Pointer #Stack_Frame #Recursion #Postfix_Notation #Deepcopy

---

## 🎯 학습 목표
* 데이터를 효율적으로 저장하고 관리하는 자료구조의 개념 및 문제별 적합한 자료구조 설계 능력 배양
* 후입선출(LIFO) 구조의 스택 정의 및 선형 구조(1:1 관계)의 특징 완벽 파악
* 파이썬 내장 메서드(`append`, `pop`)에 의존하지 않는 저수준(C-level) 스택 구조체 및 `top` 포인터 직접 구현
* 괄호 검사, 함수 호출 관리, 웹 브라우저 뒤로가기 등 스택의 실전 응용 메커니즘 이해
* 중위 표기법(Infix)과 후위 표기법(Postfix)의 차이점 및 스택 기반의 수식 변환/계산 알고리즘 숙달
* 재귀 호출 시 메모리 내부의 스택 프레임(지역 변수, 복귀 주소) 할당 구조 및 `Stack Overflow` 원인 분석
* 가변/불변 객체의 인자 전달 방식(`Pass-by-value/reference`) 및 중첩 리스트의 독립성 확보를 위한 `deepcopy` 원리 정립

---

## 💡 주요 개념 정리

### 1. 스택(Stack)의 기본 원리 및 특성
* **핵심 정의:** 물건을 쌓아 올리듯 자료를 선형으로 저장하고 관리하는 후입선출(LIFO, Last-In-First-Out) 자료구조
* **주요 특징:**
    * **선형 구조:** 데이터 사이에 1:1 관계의 순서가 존재하며 앞뒤가 연결된 연속적 형태
    * **실생활 비유:** 접시 쌓기, 판 위에 호떡 쌓기(가장 늦게 구워진 호떡을 먼저 꺼내 고객에게 전달)
* **비선형 구조와의 비교:** 트리나 그래프처럼 친구 관계(1:N, N:M)와 같이 순서가 불규칙하고 복잡하게 얽힌 구조

### 2. 저수준 스택 구현 및 관리 지표
* **Top 포인터:** 스택 내 마지막 원소의 인덱스를 가리키는 지시자. 데이터 부재 시 `-1`로 초기화하며, 삽입 시 증가 및 삭제 시 감소 연산 수행
* **Capacity(용량):** 파이썬과 달리 저수준 언어(C 등)에서는 리스트 크기를 미리 지정해야 하며, 용량 초과 시 삽입 불가 상태(`isFull`)를 체크해야 함
* **핵심 연산:**
    | 연산명 | 기능 설명 |
    | :--- | :--- |
    | **Push** | `top`을 한 칸 올린 뒤 해당 위치에 신규 데이터 삽입 |
    | **Pop** | `top` 위치의 데이터를 반환 후 `None`으로 초기화 및 `top` 한 칸 내림 |
    | **Peek** | `top`의 데이터를 제거하지 않고 현재 마지막 원소의 값만 확인 |
    | **isEmpty** | `top == -1` 여부를 확인하여 데이터 존재 유무 판단 |

### 3. 스택 응용: 괄호 검사 및 수식 계산
* **괄호 짝짓기 조건:**
    1. 왼쪽/오른쪽 괄호의 총 개수 일치
    2. 왼쪽 괄호가 오른쪽보다 먼저 등장
    3. 동일 유형의 괄호 짝(소/중/대) 보존 및 포함 관계 유지
* **후위 표기법(Postfix) 변환:**
    * 컴퓨터 친화적 연산 방식으로 우선순위(괄호) 고려가 불필요함
    * **우선순위 전략:** 나보다 우선순위가 높은 '선배님' 연산자(곱셈/나눗셈 등)가 스택에 있으면 먼저 내보내고(pop) 본인은 입성함
    * **괄호 처리:** 닫힌 괄호를 만나면 열린 괄호가 나올 때까지 스택의 모든 연산자를 털어냄(pop)

### 4. 재귀 호출(Recursion)과 메모리 참조
* **재귀 원리:** 동일한 함수를 반복 호출하되 범위를 좁혀 수렴하게 만듦. 인턴(제일 아래)까지 일을 넘긴 뒤 결과값을 사장(최초 호출자)에게 역순 보고하는 구조
* **스택 프레임(Stack Frame):** 함수 호출 시 지역 변수와 복귀 주소를 저장하는 메모리 공간. 종료 조건 부재 시 `Stack Overflow` 에러 발생
* **참조 메커니즘:**
    * **Pass-by-value:** 정수 등 불변 객체 전달 시 값만 복사되어 원본이 보존됨
    * **Pass-by-reference:** 리스트 등 가변 객체 전달 시 주소값을 공유하여 원본이 훼손될 수 있음
* **Deepcopy:** 중첩 리스트 내부의 주소값까지 완벽하게 분리하여 복사하는 유일한 방법

---

## 💻 기능 구현 및 코드 실습

### [문제] 저수준 스택 클래스 구현
> **활용 알고리즘:** `Class_Architecture`, `Top_Pointer_Logic`
> **난이도:** ★★☆☆☆

```python
class Stack:
    def __init__(self, capacity=10):
        # 1. 초기화 단계: 용량 설정 및 -1로 top 포인터 세팅
        self.capacity = capacity
        self.items = [None] * capacity
        self.top = -1

    def push(self, item):
        # 2. 로직 처리: 가득 찼는지 검증 후 top 증가 및 데이터 투하
        if self.top == self.capacity - 1:
            raise IndexError("스택 가득 참")
        self.top += 1
        self.items[self.top] = item

    def pop(self):
        # 3. 데이터 추출: 비어있는지 검증 후 top 원소 반환 및 한 칸 내림
        if self.top == -1:
            raise IndexError("스택 비어있음")
        item = self.items[self.top]
        self.items[self.top] = None
        self.top -= 1
        return item
```

### [문제] 후위 표기법(Postfix) 계산기
> **활용 알고리즘:** `Postfix_Logic`, `O(N)`
> **난이도:** ★★★☆☆

```python
def evaluate_postfix(tokens):
    stack = []
    for token in tokens:
        if token.isdigit(): # 피연산자면 스택에 추가
            stack.append(int(token))
        else: # 연산자면 스택에서 2개 꺼내 계산 (순서 op1-op2 주의)
            op2 = stack.pop()
            op1 = stack.pop()
            if token == '+': stack.append(op1 + op2)
            elif token == '-': stack.append(op1 - op2)
            elif token == '*': stack.append(op1 * op2)
            elif token == '/': stack.append(op1 / op2)
    return stack.pop()
```

---

## 🚀 복습 및 AI 인사이트
* **헷갈렸던 점:**
    - 후위 표기법 계산 시 스택에서 먼저 나온 값이 수식의 뒤(`op2`)에 위치해야 결과값이 정확함
    - 재귀 함수는 단순히 자기 호출이 아니라, 반드시 해결 가능한 최소 단위(Base Case)로 일을 넘겨야 함을 인지
    - 리스트 인자 전달 시 `copy`만으로는 내부 리스트의 주소 공유 문제를 해결할 수 없으므로 `deepcopy` 필수 사용 인지
* **AI 활용 팁:**
    - 중위 표기법의 복잡한 괄호 수식을 AI에게 시각화된 후위 표기법 변환 과정을 요청하여 로직 검증 수행
    - 파이썬 PEP 8 가이드에 따른 클래스 메서드 간 공백 및 노란색 경고 줄에 대한 AI 가이드 참고 습관화