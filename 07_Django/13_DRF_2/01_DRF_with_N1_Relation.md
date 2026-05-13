# [Django] DRF with N:1 Relation

---

## 1. 사전 준비

Comment 모델을 정의하고 마이그레이션 및 fixtures 데이터를 로드한다.

```python
# articles/models.py

class Comment(models.Model):
    article = models.ForeignKey(Article, on_delete=models.CASCADE)
    content = models.CharField(max_length=200)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

```bash
$ python manage.py makemigrations
$ python manage.py migrate
$ python manage.py loaddata articles.json comments.json
# Installed 40 object(s) from 2 fixture(s)
```

오늘 수업에서 구현할 URL과 HTTP 메서드 구성은 아래와 같다.

| URL | GET | POST | PUT | DELETE |
|-----|-----|------|-----|--------|
| `comments/` | 댓글 목록 조회 | | | |
| `comments/<int:comment_pk>/` | 단일 댓글 조회 | | 단일 댓글 수정 | 단일 댓글 삭제 |
| `articles/<int:article_pk>/comments/` | | 댓글 생성 | | |

---

## 2. GET method — 댓글 목록 및 단일 댓글 조회

### CommentSerializer 정의

```python
# articles/serializers.py
from .models import Article, Comment

class CommentSerializer(serializers.ModelSerializer):
    class Meta:
        model = Comment
        fields = '__all__'
```

> **ModelSerializer**: Django 모델 구조를 바탕으로 자동으로 필드를 생성해주는 Serializer 클래스

### 댓글 목록 조회 (GET - List)

```python
# articles/urls.py
urlpatterns = [
    ...
    path('comments/', views.comment_list),
]
```

```python
# articles/views.py
from .models import Article, Comment
from .serializers import ArticleListSerializer, ArticleSerializer, CommentSerializer

@api_view(['GET'])
def comment_list(request):
    comments = Comment.objects.all()
    serializer = CommentSerializer(comments, many=True)  # 쿼리셋은 many=True 필수
    return Response(serializer.data)
```

### 단일 댓글 조회 (GET - Detail)

```python
# articles/urls.py
urlpatterns = [
    ...
    path('comments/<int:comment_pk>/', views.comment_detail),
]
```

```python
# articles/views.py
@api_view(['GET'])
def comment_detail(request, comment_pk):
    comment = Comment.objects.get(pk=comment_pk)
    serializer = CommentSerializer(comment)  # 단일 인스턴스는 many 옵션 불필요
    return Response(serializer.data)
```

---

## 3. POST method — 댓글 생성

댓글 생성은 **어떤 게시글에 달리는지**가 필요하기 때문에 별도의 URL을 사용한다.

```python
# articles/urls.py
urlpatterns = [
    ...
    path('articles/<int:article_pk>/comments/', views.comment_create),
]
```

```python
# articles/views.py
@api_view(['POST'])
def comment_create(request, article_pk):
    # 1. 어떤 게시글에 작성되는 댓글인지 조회
    article = Article.objects.get(pk=article_pk)
    # 2. 사용자 입력 데이터 직렬화
    serializer = CommentSerializer(data=request.data)
    # 3. 유효성 검사 후 저장
    if serializer.is_valid(raise_exception=True):
        # 외래키 데이터를 save() 인자로 직접 주입 (DRF 방식)
        serializer.save(article=article)
        return Response(serializer.data, status=status.HTTP_201_CREATED)
```

> Django의 `commit=False`와 달리, **DRF의 `save()`는 키워드 인자로 추가 데이터를 바로 전달**한다.
>
> `raise_exception=True`를 사용하면 `is_valid()` 실패 시 자동으로 **400 Bad Request**를 응답하므로 별도의 `else` 처리를 생략할 수 있다.

### POST 시 400 에러가 나는 이유

`CommentSerializer`의 `fields = '__all__'` 설정으로 인해 외래키 필드 `article`도 유효성 검사 대상에 포함된다. 그러나 `article`은 `save()` 시점에 주입되므로, **유효성 검사 목록에서 제외**해야 한다.

---

## 4. 읽기 전용 필드 (read_only_fields)

**읽기 전용 필드**란 유효성 검사에서 제외되고, 데이터 **조회 시에만 값을 제공**하는 필드를 말한다.

```
읽기 전용 필드 = 유효성 검사 제외 + 응답 시 출력
```

```python
# articles/serializers.py
class CommentSerializer(serializers.ModelSerializer):
    class Meta:
        model = Comment
        fields = '__all__'
        read_only_fields = ('article',)  # 외래키 필드를 읽기 전용으로 지정
```

**사용 목적:**
- 클라이언트가 직접 수정하면 안 되는 값 (예: 외래키, 생성 시간)
- 서버 로직에 의해 자동 생성·관리되는 값
- 입력은 받지 않지만 응답에는 포함해야 하는 값
- 추가 계산으로 만들어지는 가공 데이터 (예: 댓글 개수)

**특징:**
- 생성(POST)과 수정(PUT) 요청 **모두에서 적용** — 읽기 전용이라고 POST에서만 의미 있는 것이 아님
- 이 설정 없이 `view`에서 `save(article=article)`을 호출하면 유효성 검사 단계에서 `article` 누락으로 **400 에러** 발생

---

## 5. DELETE & PUT method — 댓글 삭제 및 수정

단일 댓글의 조회, 수정, 삭제는 모두 같은 URL(`comments/<pk>/`)을 사용하며, `request.method`로 분기한다.

```python
# articles/views.py
@api_view(['GET', 'PUT', 'DELETE'])
def comment_detail(request, comment_pk):
    comment = Comment.objects.get(pk=comment_pk)  # 공통 조회 (if 밖으로)

    if request.method == 'GET':
        serializer = CommentSerializer(comment)
        return Response(serializer.data)

    elif request.method == 'PUT':
        # 기존 인스턴스(comment)와 새 데이터를 함께 전달해야 '갱신'이 됨
        serializer = CommentSerializer(comment, data=request.data)
        if serializer.is_valid(raise_exception=True):
            serializer.save()  # 수정 시 외래키는 이미 존재하므로 재주입 불필요
            return Response(serializer.data)

    elif request.method == 'DELETE':
        comment.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

- PUT 응답: **200 OK** + 수정된 데이터
- DELETE 응답: **204 No Content** (반환할 내용 없음)

---

## 6. 응답 데이터 재구성 — 댓글 조회 시 게시글 정보 포함

기본적으로 댓글의 `article` 필드는 게시글 **번호(id)만** 반환한다. 이를 **게시글의 title**까지 포함하도록 커스텀할 수 있다.

### 접근 방식

`CommentSerializer` 내부에 `ArticleTitleSerializer`를 중첩 정의하여 `article` 필드를 **재정의**한다.

```python
# articles/serializers.py
class CommentSerializer(serializers.ModelSerializer):

    # CommentSerializer 내부에 중첩 정의 (이 클래스에서만 사용할 도구)
    class ArticleTitleSerializer(serializers.ModelSerializer):
        class Meta:
            model = Article
            fields = ('title',)

    # 기존 article 필드(외래키 id)를 ArticleTitleSerializer 결과로 재정의
    article = ArticleTitleSerializer(read_only=True)

    class Meta:
        model = Comment
        fields = '__all__'
        # read_only_fields = ('article',)  # 재정의 시 이 설정은 동작하지 않음!
```

> **응답 결과:**
> ```json
> {
>     "id": 13,
>     "article": { "title": "게시글 제목" },
>     "content": "댓글 내용",
>     ...
> }
> ```

---

## 7. 읽기 전용 필드 주의사항 — read_only_fields vs read_only 인자

| 상황 | 사용 방법 |
|------|-----------|
| 기존 외래키 필드를 그대로 응답에 사용 | `Meta.read_only_fields = ('article',)` |
| 기존 필드를 **재정의**하거나 새 필드 추가 | 필드 선언 시 `read_only=True` 인자 사용 |

메타 클래스에서 작성한 `read_only_fields`는 **필드를 재정의하는 순간 더 이상 적용되지 않는다.** 재정의된 필드에는 반드시 `read_only=True` 키워드 인자를 직접 지정해야 한다.

```python
# ❌ 잘못된 방법 — article 재정의 후 read_only_fields는 무시됨
article = ArticleTitleSerializer(read_only=True)
class Meta:
    read_only_fields = ('article',)  # 동작 안 함!

# ✅ 올바른 방법
article = ArticleTitleSerializer(read_only=True)  # 필드 선언 시 직접 지정
```

---

## 💡 한 줄 요약

> N:1 관계에서 DRF는 `read_only_fields`로 외래키를 유효성 검사에서 제외하고, `save(article=article)`로 주입하며, 중첩 Serializer로 응답 데이터를 자유롭게 커스텀할 수 있다.

## ❓ 더 찾아볼 것

- `raise_exception=True`의 내부 동작 (ValidationError 예외 처리 흐름)
- DRF `save()` 메서드의 `create()` / `update()` 내부 구분 원리
- Serializer 필드 재정의와 Meta 클래스 간의 우선순위
