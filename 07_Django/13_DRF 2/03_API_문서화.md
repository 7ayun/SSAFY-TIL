# [Django] API 문서화

---

## 1. OpenAPI Specification (OAS)

API의 세부 사항을 기술하는 공식 표준. 표준화된 방법을 따르면 자동화된 문서 생성이 가능하다.

대표적인 두 가지 표현 방식:

| 방식 | 특징 |
|------|------|
| **Swagger** | 컬러풀하고 인터랙티브한 UI, Try it out 기능 제공 |
| **Redoc** | 딱딱한 명세서 스타일, 좌우 패널 구성 |

둘 다 오픈소스 프레임워크이며, **drf-spectacular** 라이브러리 하나로 두 방식을 모두 지원한다.

---

## 2. drf-spectacular 설치 및 설정

### 설치

```bash
$ pip install drf-spectacular
```

### settings.py 등록

```python
# settings.py
INSTALLED_APPS = [
    ...
    'drf_spectacular',  # 설치 시 하이픈(-), 등록 시 언더바(_) 주의!
]

# REST_FRAMEWORK에 AutoSchema 등록 (필수)
REST_FRAMEWORK = {
    'DEFAULT_SCHEMA_CLASS': 'drf_spectacular.openapi.AutoSchema',
}

# 문서 메타 정보 설정 (필수는 아니지만 권장)
SPECTACULAR_SETTINGS = {
    'TITLE': 'Your Project API',
    'DESCRIPTION': 'Your project description',
    'VERSION': '1.0.0',
}
```

### urls.py에 문서화 URL 추가

drf-spectacular 공식 문서에서 제공하는 URL 패턴을 프로젝트 `urls.py`에 추가한다.

```python
# project/urls.py
from drf_spectacular.views import SpectacularAPIView, SpectacularRedocView, SpectacularSwaggerView

urlpatterns = [
    ...
    # Schema 파일 (JSON/YAML 형태의 API 명세)
    path('api/v1/schema/', SpectacularAPIView.as_view(), name='schema'),
    # Swagger UI
    path('api/v1/swagger/', SpectacularSwaggerView.as_view(url_name='schema'), name='swagger-ui'),
    # Redoc UI
    path('api/v1/redoc/', SpectacularRedocView.as_view(url_name='schema'), name='redoc'),
]
```

> URL 경로명은 자유롭게 변경 가능하다. 프로젝트 URL 컨벤션에 맞게 조정하면 된다.

---

## 3. 문서화 활용

서버를 실행한 후 브라우저에서 접속하면 자동 생성된 API 문서를 확인할 수 있다.

```
http://127.0.0.1:8000/api/v1/swagger/   → Swagger UI
http://127.0.0.1:8000/api/v1/redoc/     → Redoc UI
```

문서에는 우리가 이틀간 구현한 모든 URL이 자동으로 해석되어 표시된다:

- HTTP 메서드별 색상 구분
- 경로 파라미터 및 Request Body 구성 표시
- 필수/선택 인자 정보
- Try it out으로 직접 API 테스트 가능

> **API 설계 우선 접근법:** API를 먼저 설계하고 명세를 작성한 뒤 기능 구현으로 들어가는 흐름이 API의 일관성을 유지하고 사용자에게 더 직관적인 문서를 제공한다. Swagger/Redoc이 이를 지원하는 도구다.

---

## 4. 참고 — 올바르게 404 응답하기

### 기존 방식의 문제

```python
article = Article.objects.get(pk=article_pk)  # 없으면 500 Internal Server Error 발생
```

`objects.get()`은 조회 대상이 없을 때 `DoesNotExist` 예외를 발생시켜 **500 에러**를 반환한다. 그러나 500은 서버 내부 오류에 해당하며, "찾고자 하는 데이터가 없다"는 클라이언트 상황에는 **404 Not Found**가 더 정확하다.

### Django shortcut 함수 활용

Django는 `render`, `redirect` 외에도 다음 두 함수를 제공한다.

```python
from django.shortcuts import get_object_or_404, get_list_or_404

# objects.get() 대체 — 없으면 404 반환
article = get_object_or_404(Article, pk=article_pk)

# objects.all() 대체 — 빈 쿼리셋 대신 404 반환
articles = get_list_or_404(Article)
```

| 기존 방식 | 개선 방식 | 없을 때 응답 |
|-----------|-----------|-------------|
| `objects.get()` | `get_object_or_404()` | 500 → **404** |
| `objects.all()` | `get_list_or_404()` | 200(빈 리스트) → **404** |

> 클라이언트 입장에서 빈 리스트로 200 OK를 받으면 "내가 잘못 요청한 건가?" 혼란이 생긴다. 데이터가 없다는 사실을 명확히 전달하려면 404를 사용하는 것이 더 정확하다.

### annotate와 함께 사용

```python
from django.shortcuts import get_object_or_404
from django.db.models import Count

article = get_object_or_404(
    Article.objects.annotate(num_of_comments=Count('comment')),
    pk=article_pk
)
```

`get_object_or_404`는 모델 클래스뿐만 아니라 **쿼리셋도 첫 번째 인자로 받을 수 있어** `annotate`와 조합이 가능하다.

---

## 5. 참고 — View와 Serializer의 역할 분리

| 구성 요소 | 담당 역할 |
|-----------|-----------|
| **View** | 복잡한 DB 쿼리, annotate / select_related / prefetch_related 등 쿼리 최적화, 비즈니스 로직 처리 |
| **Serializer** | View가 준비한 결과물을 직렬화, 응답 데이터 구조 정의 |

Serializer도 기술적으로는 연산을 처리할 수 있지만, 역할을 혼재시키면 유지보수가 어려워진다. **View에서 로직을 완성하고, Serializer는 직렬화에만 집중**하는 구조를 권장한다.

---

## 💡 한 줄 요약

> drf-spectacular 하나로 Swagger와 Redoc 문서를 자동 생성할 수 있으며, get_object_or_404로 클라이언트에게 더 정확한 404 응답을 전달해야 한다.

## ❓ 더 찾아볼 것

- `drf-spectacular` 공식 문서 — 인증 시스템 연동, 커스텀 스키마 추가 방법
- OpenAPI 3.0 표준 명세 구조
- Swagger UI에서 JWT 토큰 인증 설정하는 방법
- `get_list_or_404`를 쓸 때 정말 빈 리스트에도 404를 줘야 하는지 (서비스 요구사항에 따라 다를 수 있음)
