# [DB] Comment & User

---

## 1. Comment - User N:1 모델 관계 설정

User 한 명은 0개 이상의 댓글을 작성할 수 있다. 즉 **User : Comment = 1 : N** 관계이고, 외래 키는 N쪽인 `Comment` 모델에 작성한다.

`Comment` 모델은 이미 `Article`에 대한 외래 키를 가지고 있으므로, 여기에 `User`에 대한 외래 키를 하나 더 추가한다. 결과적으로 댓글 하나가 저장되려면 **외래 키가 두 개** 모두 필요하다.

```python
# articles/models.py
class Comment(models.Model):
    article = models.ForeignKey(Article, on_delete=models.CASCADE)
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)  # 추가
    content = models.CharField(max_length=200)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

> `Article` 관계와 동일하게, `User` 참조 시 `settings.AUTH_USER_MODEL`을 사용한다.

---

## 2. Migration 처리

Article - User 관계 설정 때와 동일하게 기존 테이블에 `NOT NULL` 필드를 추가하므로 기본값 설정 과정이 필요하다.

```bash
$ python manage.py makemigrations
# → 1 입력 (one-off default 제공)
# → 1 입력 (기존 댓글이 있다면 1번 유저 PK를 기본값으로)

$ python manage.py migrate
```

Migration 후 `articles_comment` 테이블에 `user_id` 필드가 생성되며, 이제 댓글 한 행은 `article_id`와 `user_id` **두 개의 외래 키**를 모두 포함하게 된다.

---

## 3. 댓글 CREATE - 작성자 정보 추가

외래 키가 두 개이므로 `commit=False` 이후 두 가지 모두 할당해야 한다. `article` 정보는 이미 지난 시간에 처리했고, 이번에 `user` 정보를 추가로 할당한다.

```python
# articles/views.py
def comments_create(request, pk):
    article = Article.objects.get(pk=pk)
    comment_form = CommentForm(request.POST)
    if comment_form.is_valid():
        comment = comment_form.save(commit=False)   # DB 저장 일시 중단
        comment.article = article                    # 게시글 외래 키 할당
        comment.user = request.user                 # 유저 외래 키 할당 (추가)
        comment.save()
        return redirect('articles:detail', article.pk)
    context = {
        ...
    }
```

댓글 작성 시 `user_id`가 누락되면 `IntegrityError: NOT NULL constraint failed: articles_comment.user_id` 오류가 발생한다.

---

## 4. 댓글 READ - 작성자 출력

댓글 목록에 작성자 정보를 함께 표시한다.

```html
<!-- articles/detail.html -->
<h4>댓글 목록</h4>
<ul>
  {% for comment in comments %}
    <li>
      {{ comment.user }} - {{ comment.content }}
      ...
    </li>
  {% endfor %}
</ul>
```

`{{ comment.user }}`만 써도 `username`이 출력된다. `AbstractUser`의 `__str__` 메서드가 `username`을 반환하도록 정의되어 있기 때문이다.

---

## 5. 댓글 DELETE - 본인 댓글만 삭제

### 템플릿: 삭제 버튼 조건부 표시

```html
<!-- articles/detail.html -->
{% for comment in comments %}
  <li>
    {{ comment.user }} - {{ comment.content }}
    {% if request.user == comment.user %}
      <form action="{% url 'articles:comments_delete' article.pk comment.pk %}" method="POST">
        {% csrf_token %}
        <input type="submit" value="DELETE">
      </form>
    {% endif %}
  </li>
{% endfor %}
```

### View 함수: 시스템 차원에서 차단

```python
# articles/views.py
def comments_delete(request, article_pk, comment_pk):
    comment = Comment.objects.get(pk=comment_pk)
    if request.user == comment.user:    # 작성자 본인 확인
        comment.delete()
    return redirect('articles:detail', article_pk)
```

템플릿에서 버튼을 숨기더라도 외부 도구(Postman, requests 라이브러리 등)로 POST 요청을 보낼 수 있기 때문에, View 함수 내부에서의 사용자 검증이 반드시 필요하다.

---

## 💡 한 줄 요약
> Comment 모델에 User 외래 키를 추가하고, `commit=False` 이후 `comment.user = request.user`를 할당해 작성자를 저장한 뒤, `request.user == comment.user` 비교로 본인 댓글만 삭제 가능하게 제한한다.

## ❓ 더 찾아볼 것
- 외래 키가 두 개인 모델의 `select_related()` 활용 (쿼리 최적화)
- `CommentForm`에서 `user`, `article` 필드를 `exclude` 처리하는 이유
- `@login_required`를 댓글 CRUD에 적용하는 방법 (참고 파트)
