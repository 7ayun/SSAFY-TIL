# Day06 — 상속 · super · MRO · 예외 처리 복습 노트

> **목표:** 구조 이해 + 실행 흐름 예측

---

## 1️⃣ 상속 기본

1. 상속은 기존 클래스의 ______을 재사용하기 위한 구조이다.
2. 부모 클래스 → ______ 클래스


---

## 2️⃣ super() 핵심

1. super()는 ______ 기준으로 다음 클래스를 호출한다.
2. super()는 ______를 직접 호출하지 않는다.


---

## 3️⃣ MRO 구조

```python
class A: pass
class B(A): pass
class C(A): pass
class D(B, C): pass
```

1. `D.mro()`의 두 번째 요소는 ______이다.
2. 전체 순서: D → ______ → ______ → A


---

## 4️⃣ 실행 흐름 예측

```python
class A:
    def show(self): print('A')

class B(A):
    def show(self):
        super().show()
        print('B')

class C(A):
    def show(self):
        super().show()
        print('C')

class D(B, C):
    def show(self):
        super().show()
        print('D')

D().show()
```

출력 순서:

1. ______
2. ______
3. ______
4. ______


---

## 5️⃣ Error vs Exception

1. 문법 자체 오류 → ______
2. 실행 중 발생 → ______


---

## 6️⃣ 예외 처리 구조

```python
try:
    ...
except ______:
    ...
else:
    ...
finally:
    ...
```

빈칸에 들어갈 키워드: ______


---

## 7️⃣ 시험 함정

1. super()를 ______ 호출로 해석하면 틀린다.
2. MRO는 항상 ______ 기준으로 해석한다.


---

### 이동
👉 [학습 정리](./README.md)
👉 [메인 README](../README.md)