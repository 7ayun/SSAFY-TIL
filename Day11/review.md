# Day11 Stack 1·2 / 괄호검사 / 계산기 / 재귀 복습 노트

---

### 1) 스택 기본

**Q1. 스택의 핵심 구조**
- 스택 = (      )

<details>
<summary>정답</summary>

LIFO
</details>

---

**Q2. top 초기값**
- `top = (      )`

<details>
<summary>정답</summary>

-1
</details>

---

### 2) 괄호 검사

**Q3. 닫는 괄호를 만났을 때 해야 할 것 2가지**
1. 스택이 (      )인지 확인
2. top과 (      ) 확인

<details>
<summary>정답</summary>

1) 비었는지
2) 매칭되는지
</details>

---

### 3) 전치/회전

**Q4. 전치 핵심 코드**
- `zip(      )`

<details>
<summary>정답</summary>

*matrix
</details>

---

**Q5. 90도 시계 회전 원리**
- (      ) 후 (      )

<details>
<summary>정답</summary>

행 뒤집기, 전치
</details>

---

### 4) 계산기

**Q6. 처리 순서**
- (      ) → (      ) → 계산

<details>
<summary>정답</summary>

중위표기식, 후위표기식
</details>

---

**Q7. 후위표기식에서 숫자를 만나면?**
- (      )

<details>
<summary>정답</summary>

스택에 push
</details>

---

**Q8. 연산자를 만나면?**
- 스택에서 (      )개 pop 후 계산

<details>
<summary>정답</summary>

2
</details>

---

**Q9. 후위 계산 pop 순서**
- `op2 = pop()` 은 (      ) 피연산자
- `op1 = pop()` 은 (      ) 피연산자

<details>
<summary>정답</summary>

오른쪽, 왼쪽
</details>

---

### 5) 재귀

**Q10. 재귀의 필수 요소 2가지**
1. (      )
2. (      )

<details>
<summary>정답</summary>

종료조건(Base case)
재귀호출
</details>

---

**Q11. factorial 종료조건**
```python
if n <= (      ):
    return (      )
```

<details>
<summary>정답</summary>

1, 1
</details>

---

**Q12. 피보나치 점화식**
- `f(n) = f(n-1) + (      )`

<details>
<summary>정답</summary>

f(n-2)
</details>

---

**Q13. 거듭제곱 최적화 시간복잡도**
- `O(      )`

<details>
<summary>정답</summary>

log n
</details>

---

### 이동
👉 [학습 정리](./README.md)
👉 [메인 README](../README.md)