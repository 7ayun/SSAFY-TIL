# [관통 PJT] Unit Test

---

## 1. 단위 테스트(Unit Test)란

소스 코드의 특정 모듈(함수)이 **의도대로 작동하는지 검증하는 가장 작은 단위의 테스트**

> TDD(개발 방법론)와 Unit Test(테스트 작성 도구)는 엄연히 다르다.  
> - **TDD**: 기획 → 테스트 케이스 작성 → 개발 순서의 **개발 방법론**  
> - **Unit Test**: 테스트 코드를 어떻게 작성하는지에 관한 **도구/방법**  
> 유닛 테스트는 전통적인 개발 방식이든 TDD든 언제든 도입 가능하다.

### 단위 테스트의 특징 (F.I.R.S.T 원칙)

| 원칙 | 설명 |
|------|------|
| **F**ast (빠르게) | 테스트가 빠르게 진행되어야 함 |
| **I**ndependent (독립적) | 각 테스트가 서로 의존하지 않아야 함 |
| **R**epeatable (반복 가능) | 로컬, 서버 환경 등 **어디서든** 똑같이 성공/실패 |
| **S**elf-Validating (자가 검증) | 테스트 스스로가 결과를 판단 |
| **T**imely (적시에) | 실제 코드를 구현하기 직전에 작성 |

> **독립성이 중요한 이유**: VIP 테스트 결과가 실버 테스트에 영향을 주면 안 된다. 각 테스트가 실패해도 다른 테스트에 영향을 주어서는 안 된다.

---

## 2. 단위 테스트의 표준 구조: AAA 패턴

```python
def test_calculate_discount_rate():
    # Arrange: 테스트에 필요한 데이터 설정
    original_price = 10000
    discount_amount = 2000

    # Act: 실제로 검증하려는 메서드 호출
    result = calculate_discount(original_price, discount_amount)

    # Assert: 결과값이 예상과 일치하는지 확인
    assert result == 0.2
```

---

## 3. 테스트해야 할 내용

### 1. 비즈니스 로직

서비스의 규칙이 코드에 잘 적용됐는지 검증

```python
def calculate_discount(price, member_level):
    if member_level == "GOLD":
        return price * 0.2
    return 0

def test_calculate_discount_for_gold_member():
    # 'GOLD 등급은 20% 할인'이라는 비즈니스 로직 검증
    assert calculate_discount(10000, "GOLD") == 2000
```

### 2. 경계값

버그가 가장 많이 발생하는 구간 — 경계에 해당하는 부분에서 제대로 동작하는지 검증

```python
def can_join_service(age):
    return age >= 19

def test_can_join_service_boundary():
    assert can_join_service(18) == False  # 경계값 바로 아래
    assert can_join_service(19) == True   # 걸치는 경계값
    assert can_join_service(20) == True   # 경계값 바로 위
```

### 3. 예외 상황

잘못된 입력에 대한 응답을 검증

```python
def square(n):
    if n is None:
        raise ValueError("Input cannot be None")
    return n * n

def test_square_should_raise_error_for_none():
    with pytest.raises(ValueError):
        square(None)
```

---

## 4. 테스트하지 말아야 할 내용

### 1. 내부 구현 디테일

내부 로직이 잘 동작하는지 확인하지 말고, **결과가 제대로 나오지를 검증**하라

```python
def _internal_format(text):  # 테스트 X (내부 함수)
    return text.strip().lower()

def get_clean_username(username):  # 테스트 O (공개 함수)
    return _internal_format(username)

def test_get_clean_username():
    assert get_clean_username("  User123  ") == "user123"
```

### 2. 외부 라이브러리

파이썬 기본 기능이나 이미 잘 만들어진 라이브러리는 믿기 — `math.sqrt`가 잘 작동하는지 테스트할 필요 없음

### 3. 단순한 코드

로직이 아예 없는 단순한 값 전달은 검증할 필요 없음

---

## 5. pytest

파이썬 코드가 의도한 대로 잘 돌아가는지 검사해 주는 도구

- **장점**: 문법이 쉽고, 실패 원인을 자세하게 알려줌; 스스로 테스트 코드를 찾아 실행
- **단점**: 외부 의존성(설치 필요), 깊게 들어가면 학습 곡선 존재
- 파이썬에는 기본 내장 `unittest`가 있지만, **실무에서는 pytest를 대부분 사용**

### 설치 및 실행

```bash
$ pip install pytest

$ pytest              # 현재 디렉토리 전체 테스트
$ pytest 특정파일.py  # 특정 파일만 테스트
```

### 파일/함수 명명 규칙

pytest는 파일/함수의 **이름**을 보고 테스트인지 판단한다:

- **파일명**: 반드시 `test_`로 시작하거나 `_test.py`로 끝나야 함
- **함수명**: 반드시 `test_`로 시작해야 함

---

## 6. assert

뒤에 오는 조건이 참인지 거짓인지 판별

- **참이면**: 아무 일도 없다는 듯 조용히 다음 코드로 넘어감
- **거짓이면**: `AssertionError`를 던지면서 프로그램을 즉시 중단시킴

```python
# pytest assert vs 파이썬 내장 assert
# pytest는 왜 틀렸는지를 알려주고, 내장 assert는 '틀렸다'만 알려줌

def test_assert():
    assert 1 + 1 == 3
# 출력: AssertionError: assert (1 + 1) == 3
```

### assert 작성 규칙 3가지

**1. 하나의 테스트에는 하나의 검증 로직만 작성**

```python
# Bad Case - 중간에 실패하면 이후 assert는 실행조차 안 됨
def test_user_all_info():
    user = {"name": "Gemini", "age": 20, "email": "ai@google.com"}
    assert user["name"] == "Gemini"
    assert user["age"] == 25        # 여기서 실패
    assert user["email"] == "ai@google.com"  # 실행조차 안 됨

# Good Case - 각각 독립적으로 검증
def test_user_name_is_correct():
    user = {"name": "Gemini", "age": 20}
    assert user["name"] == "Gemini"

def test_user_age_is_correct():
    user = {"name": "Gemini", "age": 20}
    assert user["age"] == 20
```

**2. 비교 가능한 상태로 작성할 것**

```python
# Bad case - 0이 아니기만 하면 통과, 틀린 값도 통과될 위험
def test_calculate_fee_exists():
    result = get_delivery_fee(60000)
    assert result  # 0이 아니기만 하면 통과됨!

# Good case - 정확한 값 비교
def test_calculate_fee_is_free():
    result = get_delivery_fee(60000)
    assert result == 0  # 무료 배송이므로 반드시 0이어야 함
```

**3. 주석이나 에러 메시지 작성할 것**

```python
def test_discount_logic():
    price = 10000
    discounted = apply_coupon(price, "SUMMER_50")

    # 실패 시 Pytest의 기본 리포팅 뒤에 아래 메시지가 추가됨
    assert discounted == 5000, f"50% 쿠폰 적용 실패: {price}원에서 {discounted}원이 나옴"
```

---

## 7. Pytest 기능

### Parametrize: 여러 케이스 한 번에 검증

동일한 테스트 로직에 여러 데이터를 한 번에 검증하는 기능

```python
# Before: 동일한 테스트를 3번 반복
def test_gold_discount():
    assert calculate_discount(10000, "GOLD") == 2000

def test_silver_discount():
    assert calculate_discount(10000, "SILVER") == 1000

# After: Parametrize로 한번에 처리
@pytest.mark.parametrize("grade, expected", [
    ("GOLD", 2000),
    ("SILVER", 1000),
    ("BRONZE", 500),
], ids=["good_grade", "silver_grade", "bronze_grade"])
def test_calculate_discount_by_grade(grade, expected):
    assert calculate_discount(10000, grade) == expected
```

- **변수 이름**: 테스트 함수에서 사용할 인자 이름을 쉼표로 구분해서 작성
- **데이터 리스트**: 각 테스트 케이스를 튜플 형태로 리스트에 작성
- `ids`를 활용해 테스트 결과에 이름을 붙일 수 있음

### Fixture: 테스트 전 환경 준비

테스트를 시작하기 전에 미리 세팅이 필요한 데이터나 **환경을 만들어주는 역할**

```python
# 1. Fixture 정의
@pytest.fixture
def sample_data():
    return {"id": 1, "name": "Unit Test"}

# 2. 테스트 함수에서 이름으로 호출 (Dependency Injection)
def test_data_name(sample_data):
    assert sample_data["name"] == "Unit Test"
```

**Fixture의 장점:**

- 중복된 준비 코드를 한 곳에서 관리 (준비 과정이 바뀌면 Fixture만 고치면 됨)
- 테스트 함수 본문에는 "검증 로직"만 남게 되어 코드가 깔끔
- 테스트 로직과 데이터 준비 로직이 분리되어 유지보수가 쉬움

```python
# conftest.py - 같은 폴더의 모든 테스트 파일에서 import 없이 사용 가능
@pytest.fixture
def global_admin_user():
    return {"name": "Manager", "role": "ADMIN"}
```

> **주의**: conftest.py를 남용하면 출처를 확인하기 어려워져 명시성이 저하될 수 있다. 또한 conftest에 정의된 이름과 개별 테스트 파일의 fixture 이름이 같으면 덮어씌워질 수 있다.

### Marks: 테스트에 레이블 붙이기

테스트 케이스에 요구사항을 작성하는 방법

**내장 Marks:**

```python
@pytest.mark.skip(reason="아직 로직이 확정되지 않음")
def test_new_feature():
    assert False

@pytest.mark.skipif(sys.platform == "win32", reason="윈도우에서는 돌아가지 않는 기능입니다")
def test_linux_specific_logic():
    pass

@pytest.mark.xfail(reason="API 서버 점검 중이라 실패할 예정")
def test_server_connection():
    assert connect_to_server() == 200
```

**커스텀 Marks:**

나만의 이름을 붙여서 특정 그룹만 골라서 실행할 수 있음

```python
@pytest.mark.slow     # 실행 시간이 오래 걸리는 테스트
@pytest.mark.smoke    # 핵심 기능 테스트 (배포 전 필수)
@pytest.mark.db       # 데이터베이스를 사용하는 테스트
@pytest.mark.api      # 외부 API 연결하는 테스트
@pytest.mark.wip      # 현재 개발 진행중인 테스트
```

```bash
$ pytest -m slow          # slow 마크 테스트만 실행
$ pytest -m "not db"      # db 마크가 아닌 테스트만 실행
```

### pytest.ini: 설정 파일

Pytest 도구의 규칙을 설정하는 파일 (마커 등록, 기본 옵션 등)

```ini
[pytest]
markers =
    smoke: 핵심 기능 테스트 (배포 전 필수)
    slow: 실행 시간이 오래 걸리는 테스트
    db: 데이터베이스를 사용하는 테스트
    api: 외부 api 연결하는 테스트
    wip: 현재 개발 진행중인 테스트

# -v: 상세 보고, -s: print문 출력 허용, --durations=3: 가장 느린 테스트 3개 표시
addopts = -v -s --durations=3

python_files = test_*.py check_*.py
python_classes = Test* *Tests
python_functions = test_* check_*
```

### monkeypatch: 가짜 데이터 치환

Pytest에서 기본으로 제공하는 내장 fixture — **테스트 환경에서 가짜 데이터를 사용할 수 있도록 하는 설정**

- 테스트 환경이 종료되는 순간 모든 설정이 돌아오며, **실제 프로그램에 영향을 주지 않음**

```python
def test_db_config(monkeypatch):
    # 테스트 동안만 DB_URL을 가짜로 설정
    monkeypatch.setenv("DATABASE_URL", "sqlite:///:memory:")

    import os
    assert os.getenv("DATABASE_URL") == "sqlite:///:memory:"
```

**자주 사용하는 메서드:**

| 메서드 | 용도 |
|--------|------|
| `setattr` | 객체의 속성이나 함수를 교체할 때 |
| `setenv` | DB 비밀번호나 API 키를 테스트용으로 바꿀 때 |
| `setitem` | 딕셔너리의 값을 교체할 때 |

---

## 8. 유닛 테스트 작성 전략 (TDD 전략 활용)

### 테스트 작성 전략(추상적): 사다리 타기 패턴

"0 → 1 → 다수"의 흐름으로 작성해 나가며, 추상적인 개념을 조금 더 구체화할 수 있음

예) 장바구니 합계 계산 기능:
1. 장바구니가 비었을 때 (0)
2. 상품이 1개일 때 (1)
3. 상품이 여러 개일 때 (다수)

### 테스트 작성 전술(구체적)

**방법 1) 명백한 구현**

너무 쉽고 뻔한 경우에는 바로 정답(테스트 케이스)을 작성

```python
# 장바구니가 비었을 때 → 0 반환은 명백하므로 바로 작성
def get_total_price(items):
    return 0  # 명백하니까 바로 작성!

def test_get_total_price_empty():
    items = []
    assert get_total_price(items) == 0
```

**방법 2) 가짜 구현 → 삼각 측량**

일단 가짜(하드코딩)로 구현하고, 다른 데이터를 넣어 일반화 코드를 작성

```python
# Step 1: 가짜 구현 (하드코딩)
def get_total_price(items):
    if not items: return 0
    return 1000  # 가짜 구현

def test_get_total_price_one_item():
    items = [{'name': '사과', 'price': 1000}]
    assert get_total_price(items) == 1000

# Step 2: 삼각 측량 - 다른 데이터로 일반화
def test_get_total_price_another_item():
    items = [{'name': '포도', 'price': 5000}]
    assert get_total_price(items) == 5000  # 여기서 실패

# 결과: 하드코딩으로 안 되는 걸 깨닫고 일반화
def get_total_price(items):
    if not items: return 0
    return items[0]['price']  # 데이터를 보고 일반화

# Step 3: 여러 개일 때도 통과하는 최종 코드
def get_total_price(items):
    return sum(item['price'] for item in items)
```

---

## 9. Django TestCode

### 초기 세팅

```ini
# pytest.ini (manage.py와 같은 레벨에 생성)
[pytest]
DJANGO_SETTINGS_MODULE = <프로젝트이름>.settings
pythonpath = .
```

```bash
$ pip install pytest pytest-django
$ pytest articles/tests/           # 특정 폴더 테스트
$ pytest articles/tests/test_models.py  # 특정 파일 테스트
```

App 내부의 `tests.py` 파일을 지우고, `tests` 폴더를 만들어 테스트 유형에 따른 파이썬 파일에 테스트 코드를 작성한다.

### 1. DB 모델 테스트

```python
# articles/tests/test_models.py
import pytest
from articles.models import Article

@pytest.mark.django_db  # 테스트 DB를 활용하겠다는 마커 설정
def test_article_creation():
    # 1. 데이터 준비
    title = "테스트 제목"
    content = "테스트 내용입니다."

    # 2. 로직 작성
    article = Article.objects.create(title=title, content=content)

    # 3. 검증
    assert article.id is not None         # 객체가 진짜 생성되었는지 ID로 확인
    assert article.content == "테스트 내용입니다."
    assert article.title == "테스트 제목"
```

> 테스트 DB는 메모리에 저장되기 때문에 눈으로 확인할 수 없다. `@pytest.mark.django_db`가 없으면 DB 접근이 차단되어 RuntimeError 발생.

```python
# Article과 Comment 관계 테스트
@pytest.mark.django_db
def test_comment_creation_with_article():
    article = Article.objects.create(title="부모 게시글", content="댓글을 달아주세요.")
    comment = Comment.objects.create(article=article, content="첫 번째 댓글입니다.")

    assert comment.id is not None
    assert comment.article.title == "부모 게시글"     # 참조 확인
    assert article.comment_set.count() == 1           # 역참조 확인
    assert article.comment_set.first().content == "첫 번째 댓글입니다."
```

### 2. Serializer 테스트

데이터가 외부에 제공되기 전, 가공한 로직이 정확한지를 검증

```python
# articles/tests/test_serializers.py
from django.db.models import Count
from articles.models import Article
from articles.serializers import ArticleSerializer

@pytest.mark.django_db
def test_article_serializer_num_of_comments_one():
    article = Article.objects.create(title="테스트", content="내용")
    Comment.objects.create(article=article, content="댓글 1")

    # 실제 뷰와 같은 환경: annotate로 num_of_comments 주입
    annotated_article = Article.objects.annotate(
        num_of_comments=Count('comment')
    ).get(pk=article.pk)

    serializer = ArticleSerializer(annotated_article)
    assert serializer.data['num_of_comments'] == 1
```

### 3. View 통합 테스트

URL로 요청을 보냈을 때, 뷰가 데이터를 잘 가져와서 Serializer로 포장해 잘 응답하는지 테스트

```python
# articles/tests/test_views.py
from rest_framework.test import APIClient

@pytest.mark.django_db
def test_article_detail_api():
    # 1. 데이터 준비
    article = Article.objects.create(title="통합 테스트", content="내용")
    Comment.objects.create(article=article, content="댓글 1")
    Comment.objects.create(article=article, content="댓글 2")

    # 2. 가상의 클라이언트(Postman 역할) 생성
    client = APIClient()

    # 3. 실제 API 경로로 GET 요청
    url = f'/api/v1/articles/{article.id}/'
    response = client.get(url)

    # 4. 검증
    assert response.status_code == 200
    assert response.data['title'] == "통합 테스트"
    assert response.data['num_of_comments'] == 2
```

---

## 💡 한 줄 요약

> 단위 테스트는 F.I.R.S.T 원칙 하에 AAA 패턴으로 작성하고, pytest의 Parametrize·Fixture·Marks·monkeypatch 등의 기능을 활용해 비즈니스 로직·경계값·예외 상황을 체계적으로 검증한다.

## ❓ 더 찾아볼 것

- `pytest-cov`: 테스트 커버리지 측정
- `pytest-django` 공식 문서
- `conftest.py` 심화 활용 패턴
- Mock vs monkeypatch 차이점
- Django의 `TestCase` vs `pytest-django` 비교
