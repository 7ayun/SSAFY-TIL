# [관통 PJT] TDD (Test-Driven Development)

---

## 1. TDD란 무엇인가

**TDD(Test-Driven Development)** 는 테스트 주도 개발로, 하나의 **개발 방법론**이다.

- 일반적인 개발 순서: 기획 → 개발 → 테스트
- TDD 개발 순서: 기획 → **테스트 케이스 먼저 작성** → 테스트를 통과하는 코드 개발

즉 Test(결과)를 먼저 정의하고, 그 Test를 달성하기 위한 코드를 작성하는 설계 방식이다.

### TDD의 본질

- **요구사항에 집중**해서 먼저 코드를 정의할 수 있다
- 테스트를 정의하기 위해 "어떻게 함수를 호출할까?", "어떤 값을 반환할까?" 와 같은 고민을 먼저 해야 하며, 이는 **좋은 코드 작성을 유도**한다

> **Kent Beck**의 저서 *Test-Driven Development: By Example*에서 정립된 개념

---

## 2. TDD에 대한 오해

| 오해 | 실제 |
|------|------|
| 작업 시간이 2배 걸린다 | 초기에는 늘어나지만, **유지보수에서 많은 시간을 확보**할 수 있다 |
| 모든 코드에 100% 테스트 진행 | **핵심 비즈니스 로직과 복잡한 설계**에만 적용한다 |
| 완벽하게 설계해야 한다 | 오히려 설계가 불확실할 때, **작은 테스트부터 시작**하며 설계를 다듬는 기술이다 |
| TDD는 테스트 코드를 작성하는 기술 | TDD는 하나의 **설계 기술**(개발 방법론)이다 |

> TDD 방법론 자체는 아직 물음표지만, 유닛 테스트 작성은 AI 시대에 필수로 자리잡았다고 본다.

---

## 3. TDD vs 전통적인 개발 방식 비교

| 구분 | 전통적인 방식 | TDD |
|------|-------------|-----|
| 우선순위 | 일단 돌아가는 코드 최우선 | "어떻게 사용할 것인가"(요구사항) 결정 후 코드 작성 |
| 장점 | 빠르게 단순 기능 구현 가능, 직관적 | 깔끔한 설계 자연스럽게 도출, 리팩토링/수정 자유 |
| 단점 | 마지막에 몰아서 테스트 → 수정 비용 큼, **리팩토링의 공포**, 부실하거나 과도한 설계 | 초기 개발 속도 느려짐(약 1.5~2.5배), 레거시·외부 API 처리 학습 곡선 존재, 테스트 코드 유지보수 비용 |

### 리팩토링의 공포란?

테스트 케이스가 없으면 코드를 수정했을 때 기존 기능이 망가질까 봐 리팩토링을 꺼리게 된다. 결국 스파게티 코드가 쌓이고 나중에 2.0 버전을 새로 내는 상황이 된다.

TDD가 있으면 리팩토링 후 테스트를 돌려 바로 확인할 수 있어 **리팩토링에 자유롭다.**

---

## 4. TDD 샘플 살펴보기

두 수를 더하는 함수 `add`에 대한 TDD 예시:

```python
# 기능 코드 (나중에 작성)
def add(a, b):
    return a + b

# 테스트 케이스 (먼저 작성)
def test_add_two_positive_numbers():
    assert add(2, 3) == 5

def test_add_negative_and_positive_number():
    assert add(-1, 2) == 1

def test_add_zero():
    assert add(7, 0) == 7
```

```
collected 3 items

test.py::test_add_two_positive_numbers PASSED    [ 33%]
test.py::test_add_negative_and_positive_number PASSED    [ 66%]
test.py::test_add_zero PASSED    [100%]

========================== 3 passed in 0.03s ==========================
```

---

## 5. TDD의 핵심 가치 3가지

### 1. 유지보수성

- 기능을 추가/수정했을 때, **다른 기능이 문제없음을 바로 검증**할 수 있다
- 테스트 코드만 봐도 "어떤 의도/사용 방법"을 알 수 있는 **가장 정확한 설명서**가 된다
- 요구사항이 변경되어도 과감하게 구조를 변경할 수 있다

```python
def test_20_percent_discount_vip():
    assert apply_discount(price=10000, user_type="VIP") == 8000

def test_10_percent_discount_normal():
    assert apply_discount(price=10000, user_type="NORMAL") == 9000
```

### 2. 코드 퀄리티

- 테스트 작성 시 객체나 함수를 분리해야 하므로 **자연스럽게 모듈화**로 이어진다
- 테스트를 통과할 만큼만 코드를 작성하게 되므로 **오버 엔지니어링 방지**

### 3. 심리적 안정감

- "테스트 통과했으니 괜찮아!"라는 심리적 안정감으로 배포할 때 떨리지 않는다
- **버그가 버그를 낳는 사건** (버그 고치다 새 버그 생기는 상황)을 막을 수 있다
- CI/CD와 연계하면 테스트 통과 시에만 자동 배포되도록 설정 가능

---

## 6. TDD의 3단계: 문제 정의 → 문제 해결 → 코드 개선

### 1단계: 문제 정의 (Red)

- 요구사항을 정의하는 단계
- **반드시 실패**해야 하며, 전체적인 구조(함수명, 파라미터, 반환 타입 등)만 작성

```python
# 테스트 먼저 작성 (함수는 아직 로직 없음)
def format_string(text):
    return text  # 아직 구현 전

def test_format_string_should_trim_and_uppercase():
    input_str = "  hello world  "
    expected = "HELLO WORLD"
    result = format_string(input_str)
    assert result == expected  # 실패!
```

### 2단계: 문제 해결 (Green)

- 테스트 통과만을 목적으로 코드를 작성
- **상수, 하드코딩도 상관없음** - 일단 돌아가기만 하면 됨

```python
def format_string(text):
    return text.strip().upper()  # 테스트 통과!
```

### 3단계: 코드 개선 (Refactor)

- 기능은 그대로 유지하되, **내부 코드를 개선**하는 단계
- 중복 코드 제거, 변수/함수명 다듬기, 함수 분리 등
- 실수하면 테스트가 실패하므로 **과감하게 리팩토링 가능**
- 리팩토링이 끝나도 **테스트는 여전히 성공**이어야 함

```python
import pytest

def format_string(text):
    """
    주어진 문자열을 대문자로 바꿔주는 함수
    """
    return text.strip().upper()

@pytest.mark.parametrize("input_str, expected", [
    ("  hello  ", "HELLO"),
    ("python", "PYTHON"),
    ("  t d d  ", "T D D"),
])
def test_format_string_should_trim_and_uppercase(input_str, expected):
    result = format_string(input_str)
    assert result == expected
```

---

## 💡 한 줄 요약

> TDD는 테스트 코드를 **먼저** 작성하고 이를 통과하는 코드를 짜는 개발 **방법론**으로, 유지보수성·코드 퀄리티·심리적 안정감이라는 3가지 핵심 가치를 제공한다.

## ❓ 더 찾아볼 것

- Kent Beck, *Test-Driven Development: By Example* (원서)
- Red-Green-Refactor 사이클 심화
- TDD와 BDD(Behavior-Driven Development)의 차이
- CI/CD 파이프라인에서 테스트 자동화 설정 방법
