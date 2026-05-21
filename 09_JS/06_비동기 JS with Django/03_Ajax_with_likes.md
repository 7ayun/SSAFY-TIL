# [JS] Ajax with Likes

---

## 1. 팔로우와 다른 점 — 좋아요의 특수성

팔로우는 특정 프로필 페이지 안에 버튼이 **딱 하나**였지만, 좋아요 버튼은 게시글 목록 페이지에 **여러 개**가 존재한다.

| 구분 | 팔로우 | 좋아요 |
|---|---|---|
| 버튼 수 | 1개 | 게시글 수만큼 (N개) |
| 단순 접근법 | form에 리스너 1개 | form마다 리스너 N개 → 비효율 |
| 해결책 | — | **이벤트 버블링(이벤트 위임)** 활용 |

## 2. 이벤트 버블링 (Bubbling) 개념

한 요소에 이벤트가 발생하면, 이 요소에 할당된 핸들러가 동작하고, 이어서 **부모 요소의 핸들러가 동작**하는 현상이다.

- 가장 최상단의 조상 요소(document)를 만날 때까지 이 과정이 반복됨
- 이름의 유래: 이벤트가 가장 깊은 요소에서 시작해 부모로 거슬러 올라가는 모습이 물속 거품(bubble)과 닮았기 때문

```
<p>를 클릭하면 → p 이벤트 → div 이벤트 → form 이벤트 순서로 모두 발생
```

## 3. 이벤트 위임 (Event Delegation)

버블링을 활용해 **공통 조상 요소 하나에만 이벤트 리스너를 등록**하면, 자식 요소들의 이벤트를 한꺼번에 처리할 수 있다.

```html
<!-- 각 버튼에 리스너를 달지 않고, 부모 div 하나에만 달면 된다 -->
<div>
  <button></button>
  <button></button>
  <button></button>
</div>
```

```javascript
const divTag = document.querySelector('div')
divTag.addEventListener('click', function (event) {
  // event.target: 실제로 클릭된 버튼을 알 수 있음
  console.log(event.target)
})
```

## 4. `event.target` vs `event.currentTarget`

| 속성 | 설명 |
|---|---|
| `event.currentTarget` | 이벤트 **핸들러가 연결된 요소**. 항상 고정. `this`와 동일 |
| `event.target` | 이벤트가 **실제로 발생한 가장 안쪽의 요소**. 버블링이 진행되어도 변하지 않음 |

- 좋아요 구현에서 `event.target`을 사용하면 실제 어떤 게시글의 폼에서 이벤트가 발생했는지 알 수 있다.

## 5. 비동기 좋아요 구현 전체 코드

### (1) HTML — 컨테이너 태그와 data-* 속성 추가

```html
<!-- articles/index.html -->

<!-- 버블링을 위한 공통 조상 컨테이너 -->
<article class="article-container">
  {% for article in articles %}
    ...
    <form data-article-id="{{ article.pk }}">
      {% csrf_token %}
      {% if request.user in article.like_users.all %}
        <input type="submit" value="좋아요 취소" id="like-{{ article.pk }}">
      {% else %}
        <input type="submit" value="좋아요" id="like-{{ article.pk }}">
      {% endif %}
    </form>
    <hr>
  {% endfor %}
</article>
```

> **주의:** `id` 속성 값은 **숫자로 시작할 수 없다.** 따라서 `like-{{ article.pk }}`처럼 문자열 접두사를 붙인다. (`id="1"` 형태는 불가)

### (2) JS — 이벤트 위임으로 좋아요 처리

```javascript
// articles/index.html <script> 내부

const csrftoken = document.querySelector('[name=csrfmiddlewaretoken]').value

// ① 공통 조상 컨테이너 선택
const articleContainer = document.querySelector('.article-container')

// ② 컨테이너에 submit 리스너 하나만 등록 (버블링 활용)
articleContainer.addEventListener('submit', function (event) {
  event.preventDefault()

  // ③ 실제 이벤트가 발생한 form의 article pk 읽기 (event.target 사용)
  const articleId = event.target.dataset.articleId

  axios({
    method: 'post',
    url: `/articles/${articleId}/likes/`,
    headers: { 'X-CSRFToken': csrftoken },
  })
    .then((response) => {
      const isLiked = response.data.is_liked

      // ④ 동적 id 선택자로 해당 게시글의 버튼만 정확히 선택
      const likeBtn = document.querySelector(`#like-${articleId}`)

      if (isLiked === true) {
        likeBtn.value = '좋아요 취소'
      } else {
        likeBtn.value = '좋아요'
      }
    })
    .catch((error) => {
      console.log(error)
    })
})
```

### (3) Django view — 좋아요 처리 후 JsonResponse 반환

```python
# articles/views.py
from django.http import JsonResponse

@login_required
def likes(request, article_pk):
    article = Article.objects.get(pk=article_pk)

    if request.user in article.like_users.all():
        article.like_users.remove(request.user)
        is_liked = False
    else:
        article.like_users.add(request.user)
        is_liked = True

    context = {
        'is_liked': is_liked,
    }
    return JsonResponse(context)
```

## 6. 버블링을 사용하지 않는 대안 방법 (비교)

```javascript
// 버블링 미사용: querySelectorAll + forEach로 모든 버튼에 직접 리스너 등록
const formTags = document.querySelectorAll('.like-forms')
const csrftoken = document.querySelector('[name=csrfmiddlewaretoken]').value

formTags.forEach((formTag) => {
  formTag.addEventListener('submit', function (event) {
    event.preventDefault()
    const articleId = formTag.dataset.articleId
    axios({
      method: 'post',
      url: `/articles/${articleId}/likes/`,
      headers: { 'X-CSRFToken': csrftoken },
    })
      .then((response) => {
        const isLiked = response.data.is_liked
        const likeBtn = document.querySelector(`#like-${articleId}`)
        if (isLiked === true) {
          likeBtn.value = '좋아요 취소'
        } else {
          likeBtn.value = '좋아요'
        }
      })
  })
})
```

> 이 방식은 동작은 하지만, 버튼 100개면 리스너도 100개가 필요하다. **버블링(이벤트 위임) 방식이 리스너 1개로 해결하므로 훨씬 효율적**이다.

## 💡 한 줄 요약
> 한 페이지에 좋아요 버튼이 여러 개일 때는 공통 조상에 리스너 하나만 달고(이벤트 위임), `event.target`과 `data-article-id`로 어느 게시글인지 식별한 뒤 해당 버튼만 DOM에서 찾아 업데이트한다.

## ❓ 더 찾아볼 것
- 이벤트 위임(Event Delegation) 패턴 — 동적으로 추가되는 DOM 요소를 처리할 때도 유용한 이유
- `event.stopPropagation()` — 버블링을 의도적으로 중단하는 방법
- `querySelectorAll` 반환값 NodeList vs Array — forEach 사용 가능 여부 차이
- WebSocket — 실시간 좋아요 반영 (다른 탭에서도 즉시 반영되게 하려면)
