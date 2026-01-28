# Day05 — 객체지향 프로그래밍(OOP) 복습 노트

> **목표:** 개념 암기 ❌ → 구조 이해 + 사고 연결 ⭕

---

## 1️⃣ 절차지향 vs 객체지향

1. 절차지향은 ______ 중심, 객체지향은 ______ 중심이다.
2. 알고리즘 문제 풀이에는 ______ 방식이 적합하다.
3. 협업 프로젝트에는 ______ 방식이 적합하다.

<details>
<summary>정답</summary>

1. 흐름, 객체
2. 절차지향
3. 객체지향
</details>

---

## 2️⃣ 클래스 & 인스턴스

1. 클래스는 ______이고, 인스턴스는 ______이다.
2. 같은 클래스로 만든 인스턴스는 서로 ______적인 데이터를 가진다.

<details>
<summary>정답</summary>

1. 설계도, 실체
2. 독립
</details>

---

## 3️⃣ __init__ & 인스턴스 변수

```python
class Person:
    def __init__(self, name):
        self.name = name
```

1. `__init__`은 인스턴스 ______ 시 자동 실행된다.
2. self.name은 ______ 변수이다.

<details>
<summary>정답</summary>

1. 생성
2. 인스턴스
</details>

---

## 4️⃣ self 역할

```python
p1.introduce()
```

➡ 내부 동작: `Person.introduce(_____)`

1. self에는 ______가 전달된다.

<details>
<summary>정답</summary>

p1
인스턴스 자기 자신
</details>

---

## 5️⃣ 클래스 변수 vs 인스턴스 변수

```python
class Circle:
    pi = 3.14
    def __init__(self, r):
        self.r = r
```

1. pi는 ______ 변수, r은 ______ 변수이다.
2. pi는 모든 인스턴스가 ______한다.

<details>
<summary>정답</summary>

1. 클래스, 인스턴스
2. 공유
</details>

---

## 6️⃣ 사고 연결 문제 (시험형)

```python
class Counter:
    def __init__(self):
        self.count = 0
    def inc(self):
        self.count += 1

c1 = Counter()
c2 = Counter()

c1.inc()
c1.inc()
c2.inc()
```

1. `c1.count = ____`
2. `c2.count = ____`

<details>
<summary>정답</summary>

1. 2
2. 1
</details>

---

## 7️⃣ 시험 함정 포인트

1. self 누락 시 발생 오류: ______
2. 클래스 변수/인스턴스 변수 혼동 시 발생 문제: ______

<details>
<summary>정답</summary>

1. TypeError
2. 값 공유 오류
</details>

---

### 이동
👉 [학습 정리](./README.md)
👉 [메인 README](../README.md)

