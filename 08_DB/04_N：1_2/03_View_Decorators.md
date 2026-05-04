# [DB] View Decorators

---

## 1. View Decorator란

View Decorator는 View 함수의 동작을 수정하거나 추가 기능을 부여하는 **Python 데코레이터**이다. View 함수가 실행되기 전에 먼저 동작하는 **사전 체크리스트** 역할을 한다.

이미 사용해 온 `@login_required`도 View Decorator의 일종으로, 비로그인 사용자를 로그인 페이지로 리다이렉트시킨다. 이 밖에도 권한 검사, HTTP 요청 방식 제한 등 다양한 데코레이터가 있다.

> 지금까지는 View 함수 내부의 `if request.method == 'POST'` 분기로 요청 방식을 구분했지만, 애초에 허용되지 않는 메서드를 데코레이터 단계에서 차단하면 코드가 더 명확해지고 보안도 강화된다.

---

## 2. Allowed HTTP methods

특정 HTTP 메서드로만 View 함수에 접근할 수 있도록 제한하는 데코레이터들이다. 허용되지 않은 메서드로 요청 시 **405 Method Not Allowed** 응답을 반환한다.

| 데코레이터 | 허용 메서드 | import 경로 |
|---|---|---|
| `@require_http_methods(["GET", "POST"])` | 지정한 메서드만 허용 | `django.views.decorators.http` |
| `@require_safe` | GET, HEAD | `django.views.decorators.http` |
| `@require_POST` | POST | `django.views.decorators.http` |

### require_http_methods

리스트 형태로 허용할 메서드를 **대문자 문자열**로 지정한다. GET과 POST를 모두 처리하는 create, update 함수에 적합하다.

```python
from django.views.decorators.http import require_http_methods

@login_required
@require_http_methods(['GET', 'POST'])
def create(request):
    ...

@login_required
@require_http_methods(['GET', 'POST'])
def update(request, pk):
    ...
```

### require_safe

GET과 HEAD 메서드만 허용한다. `@require_GET`은 곧 deprecated 예정이므로, 조회 전용 View에는 `@require_safe`를 권장한다.

> HEAD 요청은 브라우저가 실제 데이터 없이 서버의 메타데이터만 조회할 때 사용하는 메서드이다. GET과 함께 "안전한(safe) 메서드"로 분류되어 `require_safe`에 묶여 있다.

```python
from django.views.decorators.http import require_safe

@require_safe
def index(request):
    ...

@require_safe
def detail(request, pk):
    ...
```

### require_POST

POST 메서드만 허용한다. 별도의 페이지를 렌더링하지 않고 처리만 하는 delete, comments_create, comments_delete 함수에 적합하다.

```python
from django.views.decorators.http import require_POST

@login_required
@require_POST
def delete(request, pk):
    ...

@login_required
@require_POST
def comments_create(request, pk):
    ...

@login_required
@require_POST
def comments_delete(request, article_pk, comment_pk):
    ...
```

---

## 3. 데코레이터 적용 순서

데코레이터가 여러 개일 때는 **위에서 아래 순서**로 실행된다.

```python
@login_required        # 1순위: 로그인 여부 확인
@require_POST          # 2순위: POST 요청 여부 확인
def delete(request, pk):
    ...
```

`@login_required`가 먼저 실행되어 비로그인 사용자를 차단하고, 통과하면 `@require_POST`가 메서드를 검사한다. 현재 코드에서 두 데코레이터는 독립적인 조건을 검사하므로 순서의 영향이 크지 않지만, 상황에 따라 순서가 중요할 수 있다.

---

## 4. 각 View 함수별 데코레이터 적용 정리

| View 함수 | 적용 데코레이터 | 이유 |
|---|---|---|
| `index` | `@require_safe` | 게시글 목록 조회 전용 (GET만) |
| `detail` | `@require_safe` | 게시글 상세 조회 전용 (GET만) |
| `create` | `@login_required`, `@require_http_methods(['GET', 'POST'])` | GET(폼 렌더링) + POST(저장) 모두 필요 |
| `update` | `@login_required`, `@require_http_methods(['GET', 'POST'])` | GET(폼 렌더링) + POST(수정) 모두 필요 |
| `delete` | `@login_required`, `@require_POST` | POST 처리만, 별도 페이지 없음 |
| `comments_create` | `@login_required`, `@require_POST` | POST 처리만, 별도 페이지 없음 |
| `comments_delete` | `@login_required`, `@require_POST` | POST 처리만, 별도 페이지 없음 |

> 프로젝트 초반에는 View 함수의 핵심 로직에 집중하고, 기능 구현이 완료된 후에 데코레이터를 추가하는 방식으로 접근하면 좋다.

---

## 💡 한 줄 요약
> View Decorator는 View 함수 실행 전에 HTTP 메서드나 로그인 여부를 사전 차단하는 체크리스트로, `@require_safe`(조회), `@require_POST`(처리), `@require_http_methods`(복합) 세 가지를 용도에 맞게 적용한다.

## ❓ 더 찾아볼 것
- 405 Method Not Allowed 외 다른 HTTP 상태 코드 정리 (400번대)
- Django 공식 View Decorator 전체 목록: https://docs.djangoproject.com/en/5.2/topics/http/decorators/
- `Conditional view processing` 데코레이터 (`condition`, `etag`, `last_modified`)
- GZip compression 데코레이터
