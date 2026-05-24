# [DB] Article & User

---

## 1. Article - User N:1 모델 관계 설정

User 한 명은 0개 이상의 게시글을 작성할 수 있다. 즉 **User : Article = 1 : N** 관계이고, 외래 키는 N쪽인 `Article` 모델에 작성한다.

### User 모델을 직접 참조하지 않는 이유

`models.py`에서 `User` 클래스를 직접 `import`해서 사용하면 안 된다. Django 구동 순서상 `models.py`는 아주 이른 시점에 실행되는데, 이때 `User` 클래스가 아직 로드되지 않았을 수 있기 때문이다. 이를 **지연 평가(lazy evaluation)**로 해결한다.

| 참조 방법 | 반환값 | 사용 위치 |
|---|---|---|
| `settings.AUTH_USER_MODEL` | 문자열 (`'accounts.User'`) | `models.py` 내부 |
| `get_user_model()` | User 객체 (클래스) | `models.py` 제외한 모든 곳 |

- `settings.AUTH_USER_MODEL`은 문자열 형태로 전달되기 때문에, Django가 내부적으로 모든 모델이 완전히 로딩된 후 실제 클래스를 찾아 연결한다. 이것이 지연 평가의 핵심이다.
- `models.py` 이후에 실행되는 `forms.py`, `views.py` 등에서는 `get_user_model()`을 사용한다. 이 시점에는 모델 클래스가 이미 생성 완료된 상태이기 때문이다.

> Django ORM의 `Article.objects.all()` 역시 지연 평가를 사용한다. 이 코드를 작성한 시점에는 DB에 요청을 보내지 않고, 실제로 for문으로 순회하거나 list()로 변환하는 등 데이터가 필요해지는 순간에야 요청이 전송된다.

### models.py 코드

```python
# articles/models.py
from django.conf import settings  # User 클래스 직접 import 금지!

class Article(models.Model):
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    title = models.CharField(max_length=10)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

---

## 2. Migration 처리

외래 키 필드를 기존 테이블에 추가하면 `NOT NULL` 제약 조건 때문에 기본값 설정이 필요하다.

```bash
$ python manage.py makemigrations
# 아래 메시지가 출력됨:
# It is impossible to add a non-nullable field 'user' to article
# without specifying a default.
# Select an option:
#   1) Provide a one-off default now
#   2) Quit and manually define a default value in models.py

# → 1 입력 후 Enter
# → 기존 게시글이 없거나, 있다면 임시로 1번 유저 PK를 기본값으로 설정

$ python manage.py migrate
```

Migration 후 `articles_article` 테이블에 `user_id` 컬럼이 생성된 것을 확인할 수 있다.

> 주의: 이 기본값은 "필드를 추가하기 위한 임시 조건"이지, 실제 모델의 `default` 속성이 아니다. 이후 게시글 작성 시 `user_id`가 없으면 `NOT NULL` 오류가 발생한다.

---

## 3. 게시글 CREATE - commit=False 활용

외래 키 필드가 추가되면 `ArticleForm`에 `User` 선택 드롭다운이 자동으로 생긴다. 이것은 사용자가 직접 입력해야 할 필드가 아니라 시스템이 자동으로 채워야 한다.

**step 1.** `ArticleForm`에서 `user` 필드 제외

```python
# articles/forms.py
class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = ('title', 'content',)
```

**step 2.** `views.py`에서 `commit=False`로 인스턴스만 받아 `user` 정보 추가 후 저장

```python
# articles/views.py
@login_required
def create(request):
    if request.method == 'POST':
        form = ArticleForm(request.POST)
        if form.is_valid():
            article = form.save(commit=False)  # DB 저장 일시 중단
            article.user = request.user         # 현재 로그인한 유저 할당
            article.save()                      # 최종 저장
            return redirect('articles:detail', article.pk)
    else:
        ...
```

> `commit=False`: 폼 데이터로 모델 인스턴스를 생성하되, DB에 저장하는 요청은 보류한다. 이 사이에 누락된 외래 키 값을 직접 할당할 수 있다.

---

## 4. 게시글 READ - 작성자 출력

`article.user`로 작성자 User 객체를 참조할 수 있다. User 모델의 `AbstractUser`에는 `__str__` 매직 메서드가 `username`을 반환하도록 정의되어 있기 때문에, `{{ article.user }}`와 `{{ article.user.username }}`은 동일한 결과를 출력한다.

```html
<!-- articles/index.html -->
{% for article in articles %}
  <p>작성자 : {{ article.user }}</p>
  <p>글 번호: {{ article.pk }}</p>
  <a href="{% url 'articles:detail' article.pk %}">
    <p>글 제목: {{ article.title }}</p>
  </a>
  <p>글 내용: {{ article.content }}</p>
  <hr>
{% endfor %}

<!-- articles/detail.html -->
<p>작성자 : {{ article.user.username }}</p>
<p>제목: {{ article.title }}</p>
<p>내용: {{ article.content }}</p>
```

---

## 5. 게시글 UPDATE - 본인 글만 수정

`request.user`(수정 요청자)와 `article.user`(게시글 작성자)를 비교해 본인 글만 수정 가능하도록 제한한다.

```python
# articles/views.py
@login_required
def update(request, pk):
    article = Article.objects.get(pk=pk)
    if request.user == article.user:        # 작성자 본인 확인
        if request.method == 'POST':
            form = ArticleForm(request.POST, instance=article)
            if form.is_valid():
                form.save()
                return redirect('articles:detail', article.pk)
        else:
            form = ArticleForm(instance=article)
    else:
        return redirect('articles:index')   # 타인이면 메인으로 강제 이동
    context = {'article': article, 'form': form}
    return render(request, 'articles/update.html', context)
```

템플릿에서도 본인 게시글에만 수정/삭제 버튼을 표시한다.

```html
<!-- articles/detail.html -->
{% if request.user == article.user %}
  <a href="{% url 'articles:update' article.pk %}">UPDATE</a><br>
  <form action="{% url 'articles:delete' article.pk %}" method="POST">
    {% csrf_token %}
    <input type="submit" value="DELETE">
  </form>
{% endif %}
```

> 주의: 버튼을 숨기는 것만으로는 부족하다. Postman이나 `requests` 라이브러리 등으로 브라우저를 거치지 않고도 POST 요청을 보낼 수 있기 때문에, **반드시 View 함수 내부에서도 사용자를 구분해야 한다.**

---

## 6. 게시글 DELETE - 본인 글만 삭제

```python
# articles/views.py
@login_required
def delete(request, pk):
    article = Article.objects.get(pk=pk)
    if request.user == article.user:    # 작성자 본인 확인
        article.delete()
    return redirect('articles:index')
```

---

## 💡 한 줄 요약
> `settings.AUTH_USER_MODEL`로 User를 안전하게 참조하고, `commit=False`와 `request.user` 비교로 작성자 기반 CRUD 권한 제어를 구현한다.

## ❓ 더 찾아볼 것
- Django 구동 순서와 앱 레지스트리(App Registry)
- 지연 평가(Lazy Evaluation)와 QuerySet의 평가 시점
- `on_delete` 옵션 종류 (`CASCADE`, `PROTECT`, `SET_NULL` 등)
- `get_object_or_404()` 활용법
