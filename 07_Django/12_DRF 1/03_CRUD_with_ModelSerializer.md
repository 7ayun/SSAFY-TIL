# [Django] CRUD with ModelSerializer

---

## 1. URL과 HTTP Method 설계

RESTful API에서는 **URL이 자원의 위치만 표현**하고, **행위는 HTTP 메서드로 구분**한다.

> URL에 동작 이름(`get`, `create`)을 넣지 말고, 자원 중심으로 설계한다.

| URL | GET | POST | PUT | DELETE |
|---|---|---|---|---|
| `articles/` | 전체 글 조회 | 글 작성 | - | - |
| `articles/1/` | 1번 글 조회 | - | 1번 글 수정 | 1번 글 삭제 |

**URL 설계 팁**
- URL에 동작명(`get`, `create`) 사용 금지
- 복수형/단수형 혼용 금지 (일관되게 사용)
- 깊은 중첩 구조 피하기
- 기능이 아닌 자원의 위치만 URL로 표현하고, 동작은 HTTP 메서드로 구분

---

## 2. Serializer 구성

```python
# articles/serializers.py
from rest_framework import serializers
from .models import Article


# 목록 조회용 (id, title, content만 제공)
class ArticleListSerializer(serializers.ModelSerializer):
    class Meta:
        model = Article
        fields = (
            'id',
            'title',
            'content',
        )


# 상세 조회 / 생성 / 수정용 (전체 필드 제공)
class ArticleSerializer(serializers.ModelSerializer):
    class Meta:
        model = Article
        fields = '__all__'
```

- 기능에 따라 **별도의 Serializer 클래스**를 만들어 사용
- `ArticleListSerializer`: 목록에서는 핵심 필드만 제공
- `ArticleSerializer`: 상세 조회/수정 시 전체 필드 제공

---

## 3. GET method — 조회

### 전체 목록 조회 (GET `articles/`)

```python
# articles/serializers.py 에 ArticleListSerializer 정의 후

# articles/views.py
from rest_framework.response import Response
from rest_framework.decorators import api_view

from .models import Article
from .serializers import ArticleListSerializer, ArticleSerializer


@api_view(['GET'])
def article_list(request):
    articles = Article.objects.all()                         # QuerySet 조회
    serializer = ArticleListSerializer(articles, many=True)  # 직렬화
    return Response(serializer.data)                         # 응답

# articles/urls.py
urlpatterns = [
    path('articles/', views.article_list),
    ...
]
```

### 단일 게시글 조회 (GET `articles/<int:article_pk>/`)

```python
# articles/views.py
@api_view(['GET'])
def article_detail(request, article_pk):
    article = Article.objects.get(pk=article_pk)  # 단일 객체 조회
    serializer = ArticleSerializer(article)        # many=True 없음
    return Response(serializer.data)

# articles/urls.py
urlpatterns = [
    ...
    path('articles/<int:article_pk>/', views.article_detail),
]
```

### 과거 view 함수와 비교

```python
# 과거: HTML 페이지로 응답
def index(request):
    articles = Article.objects.all()
    context = {'articles': articles}
    return render(request, 'articles/index.html', context)

# 현재: JSON 데이터로 응답 (페이지 없음)
@api_view(['GET'])
def article_list(request):
    articles = Article.objects.all()
    serializer = ArticleListSerializer(articles, many=True)
    return Response(serializer.data)
```

### ModelSerializer 주요 인자 및 속성

| 항목 | 설명 |
|---|---|
| `many=True` | Serialize 대상이 QuerySet(다수)인 경우 필수. 없으면 단일 객체로 처리 |
| `.data` | Serialized data 객체에서 실제 데이터를 추출 |

### `@api_view` 데코레이터

- DRF view 함수에서 **필수**로 작성
- view 함수를 실행하기 전 HTTP 메서드를 확인
- 허용한 메서드 이외의 요청에는 **405 Method Not Allowed** 응답
- `@api_view` 없이 일반 Django view로 인식되면 → 500 에러 or HTML 응답 발생

```python
@api_view(['GET'])          # GET만 허용
@api_view(['GET', 'POST'])  # GET, POST 허용 (리스트 형태)
```

> 요청이 실패할 경우 `@api_view` 데코레이터 누락 여부를 가장 먼저 확인!

---

## 4. POST method — 생성

```python
from rest_framework import status


@api_view(['GET', 'POST'])
def article_list(request):
    if request.method == 'GET':
        articles = Article.objects.all()
        serializer = ArticleListSerializer(articles, many=True)
        return Response(serializer.data)

    elif request.method == 'POST':
        serializer = ArticleSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

- `@api_view`에 허용할 메서드를 리스트로 추가
- `request.data`: POST 요청의 데이터
- `serializer.is_valid()`: 데이터 유효성 검사
  - 성공: `201 Created` 응답
  - 실패: `400 Bad Request` 응답
- Postman에서 Body > **form-data** 탭으로 데이터 전송

> POST 에러 발생 시 `is_valid()`로 어떤 필드가 누락되었는지 확인.  
> 400 오류는 대부분 입력 데이터 문제.

---

## 5. DELETE method — 삭제

### 기본 방식 (204 No Content)

```python
@api_view(['GET', 'DELETE'])
def article_detail(request, article_pk):
    article = Article.objects.get(pk=article_pk)

    if request.method == 'GET':
        serializer = ArticleSerializer(article)
        return Response(serializer.data)

    elif request.method == 'DELETE':
        article.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

- **204 No Content**: 요청은 성공했지만, 응답으로 보낼 본문 데이터가 없음
- `Response()`는 기본적으로 `data` 인자를 필요로 하지 않음
  - 데이터 없이 상태 코드만 전달할 때는 `status=` 키워드 인자 형태로 전달

### 삭제 후 메시지 반환하는 방식 (200 OK)

```python
elif request.method == 'DELETE':
    pk = article.pk
    title = article.title
    article.delete()  # delete() 이후에는 객체 접근 불가
    data = {
        'message': f'{pk}번 게시글 "{title}"이 삭제되었습니다.'
    }
    return Response(data, status=status.HTTP_200_OK)
```

- `delete()` 실행 후에는 해당 객체에 접근할 수 없으므로, **삭제 전에 필요한 값을 변수에 저장**
- 클라이언트에서 삭제 대상 데이터를 확인하거나 UI 알림에 활용할 때 사용

> REST 원칙상 기본은 응답 없음(204).  
> 목적이 명확할 때만 200으로 삭제된 정보 반환.

---

## 6. PUT method — 수정

```python
@api_view(['GET', 'DELETE', 'PUT'])
def article_detail(request, article_pk):
    article = Article.objects.get(pk=article_pk)

    if request.method == 'GET':
        ...

    elif request.method == 'DELETE':
        ...

    elif request.method == 'PUT':
        serializer = ArticleSerializer(article, data=request.data)
        # serializer = ArticleSerializer(instance=article, data=request.data, partial=False)
        if serializer.is_valid(raise_exception=True):
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

- 수정 시 Serializer 첫 번째 인자로 **기존 인스턴스**를 전달
- 수정 성공: `200 OK` 응답

### `partial` 인자

```python
# partial 기본값: False (전체 수정)
serializer = ArticleSerializer(article, data=request.data)            # PUT: 전체 수정

# partial=True (부분 수정)
serializer = ArticleSerializer(article, data=request.data, partial=True)  # PATCH: 부분 수정
```

- `partial=False`(기본): 모든 필수 필드의 값이 전달되었는지 확인
  - `title`만 수정하려 해도 `content`도 함께 전송해야 함
- `partial=True`: 일부 필드만 전달해도 수정 허용

### PUT vs PATCH

| 항목 | PUT | PATCH |
|---|---|---|
| 수정 대상 | 전체 리소스 | 리소스의 일부 필드 |
| 요청 데이터 요구 | 모든 필수 필드 포함 | 수정할 필드만 포함 가능 |
| 사용 목적 | 전체 덮어쓰기 (교체) | 부분 수정 (일부 필드만 갱신) |
| DRF 설정 | 기본 (partial=False) | 반드시 partial=True 필요 |

```python
# PATCH 처리 예시
@api_view(['GET', 'DELETE', 'PATCH'])
def article_detail(request, article_pk):
    ...
    elif request.method == 'PATCH':
        serializer = ArticleSerializer(article, data=request.data, partial=True)
        if serializer.is_valid(raise_exception=True):
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

> 일부만 수정할 때는 **PATCH**를 사용하는 것이 RESTful한 설계.  
> PUT에서 `partial=True`를 쓰는 건 기능적으로는 동작하지만 REST 원칙에 어긋남.

---

## 7. `raise_exception` 옵션 (참고)

```python
if serializer.is_valid(raise_exception=True):
    serializer.save()
    return Response(serializer.data)
# raise_exception=True를 쓰면 is_valid 실패 시 자동으로 400 응답을 반환하므로
# 아래 return Response(serializer.errors, ...) 생략 가능
```

- `is_valid(raise_exception=True)`: 유효성 검사 실패 시 자동으로 **400 Bad Request** 응답 반환
- 별도의 실패 처리 코드를 작성할 필요 없어 코드가 간결해짐

---

## 💡 한 줄 요약

> DRF CRUD는 URL을 자원 중심으로 설계하고, `@api_view`로 허용 메서드를 지정하며, ModelSerializer로 직렬화해 JSON으로 응답하는 패턴이다.

## ❓ 더 찾아볼 것

- DRF `raise_exception` 심화 동작 방식
- `status` 모듈 주요 상태 코드 목록
- DRF Class-Based View (CBV) — `APIView`, `generics`
- DRF Pagination 처리
- `form-data` vs `raw JSON` 요청 방식 차이
