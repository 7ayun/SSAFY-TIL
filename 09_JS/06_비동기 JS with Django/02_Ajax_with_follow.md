# [JS] Ajax with Follow

---

## 1. 사전 준비

M:N 관계 모델링까지 완성된 Django 스켈레톤 코드를 내려받아 환경을 구성한다.

```bash
# 가상 환경 생성 및 활성화
python -m venv venv
source venv/Scripts/activate  # Windows

# 패키지 설치
pip install -r requirements.txt

# 서버 실행
python manage.py runserver
```

> 코드 짜는 것보다 환경 구성이 더 어려울 때가 많다. 버전 충돌 문제는 항상 골치 아프지만 가상환경 관리 습관이 중요하다.

팔로우 테스트를 위해 계정을 **2개** 생성해두자. 자기 자신은 팔로우할 수 없으므로 서로 다른 계정에서 테스트해야 한다.

## 2. 비동기 팔로우 구현 전체 흐름

```
① profile.html에 Axios CDN 추가
② form에 id, data-user-id 추가 / action·method 제거
③ JS에서 form 선택 → submit 이벤트 가로채기 (preventDefault)
④ data-* 속성으로 user pk 읽기
⑤ CSRF 토큰을 JS에서 읽어 axios 헤더에 포함
⑥ Django view에서 팔로우 처리 후 JsonResponse 반환
⑦ .then()에서 응답 데이터로 버튼 텍스트 및 카운트 DOM 업데이트
```

## 3. Axios CDN 위치와 이유

```html
<!-- accounts/profile.html -->

  ...
  <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
  <script>
    // JS 코드 작성
  </script>
</body>
</html>
```

> **왜 `</body>` 직전에 작성하는가?**  
> HTML은 위에서 아래로 순서대로 로드된다. script 태그가 무겁다면 페이지 렌더링 자체가 늦어질 수 있다. 따라서 화면을 먼저 그린 뒤 JS를 마지막에 로드하는 것이 일반적이다.

## 4. `data-*` 속성 — HTML에서 JS로 데이터 전달하기

HTML 요소에 직접 데이터를 심어두고 JS에서 읽어오는 방법이다.

```html
<!-- HTML: data-속성명="값" -->
<div data-my-id="my-data"></div>

<!-- JS: element.dataset.속성명(카멜케이스) -->
<script>
  const myId = event.target.dataset.myId
</script>
```

**주의사항:**
- `xml`로 시작하는 이름 불가
- 세미콜론 포함 불가
- 대문자 포함 불가 (HTML 속성명은 소문자, JS에서는 카멜케이스로 자동 변환)

## 5. form 수정 및 JS 이벤트 처리

### (1) form에 id와 data-* 속성 추가, action/method 제거

```html
<!-- accounts/profile.html -->
<form id="follow-form" data-user-id="{{ person.pk }}">
  {% csrf_token %}
  {% if request.user in person.followers.all %}
    <input type="submit" value="Unfollow">
  {% else %}
    <input type="submit" value="Follow">
  {% endif %}
</form>
```

- `action`과 `method` 속성은 **삭제** → 요청은 axios로 대체
- `data-user-id`로 user pk를 HTML에 삽입해 JS에서 읽을 수 있게 함

### (2) JS에서 form 선택 및 submit 이벤트 가로채기

```javascript
// accounts/profile.html <script> 내부

// querySelector: CSS 선택자로 첫 번째 매칭 HTML 요소를 반환
const formTag = document.querySelector('#follow-form')

formTag.addEventListener('submit', function (event) {
  event.preventDefault()  // 기본 동작(페이지 새로고침) 취소
})
```

## 6. CSRF 토큰을 axios 헤더에 포함하기

form을 기본 방식으로 submit하면 CSRF 토큰이 자동으로 전송되지만, **axios는 직접 헤더에 넣어줘야 한다.**

개발자도구 Elements 탭에서 `<input type="hidden" name="csrfmiddlewaretoken" value="...">` 가 실제로 존재하는 것을 확인할 수 있다. 이 값을 JS에서 읽어온다.

```javascript
// CSRF 토큰 값 읽기
const csrftoken = document.querySelector('[name=csrfmiddlewaretoken]').value

formTag.addEventListener('submit', function (event) {
  event.preventDefault()

  // data-* 속성으로 user pk 읽기 (세 가지 방법 모두 동일)
  const userId = event.currentTarget.dataset.userId
  // const userId = this.dataset.userId
  // const userId = formTag.dataset.userId

  axios({
    method: 'post',
    url: `/accounts/${userId}/follow/`,
    headers: { 'X-CSRFToken': csrftoken },  // 헤더에 CSRF 토큰 포함
  })
})
```

## 7. Django view — JsonResponse로 응답

원래 `redirect`로 HTML 전체를 반환하던 방식에서, 처리 결과만 담은 **JSON 데이터**를 반환하도록 수정한다.

```python
# accounts/views.py
from django.http import JsonResponse

@login_required
def follow(request, user_pk):
    User = get_user_model()
    person = User.objects.get(pk=user_pk)
    if person != request.user:
        if person.followers.filter(pk=request.user.pk).exists():
            person.followers.remove(request.user)
            is_followed = False
        else:
            person.followers.add(request.user)
            is_followed = True
        context = {
            'is_followed': is_followed,
            'followings_count': person.followings.count(),  # 팔로잉 수
            'followers_count': person.followers.count(),    # 팔로워 수
        }
        return JsonResponse(context)
    return redirect('accounts:profile', person.username)
```

- 개발자도구 Network 탭에서 `follow/` 요청의 Type이 `xhr`로 표시되고, Content-Type이 `application/json`이면 정상
- 응답 데이터에 `is_followed`(팔로우 상태), `followings_count`, `followers_count`를 함께 담아 JS가 DOM을 업데이트할 수 있게 함

## 8. JS에서 응답 데이터로 DOM 업데이트

### (1) 팔로우 버튼 텍스트 토글

```javascript
.then((response) => {
  const isFollowed = response.data.is_followed
  const followBtn = document.querySelector('input[type=submit]')

  if (isFollowed === true) {
    followBtn.value = 'Unfollow'
  } else {
    followBtn.value = 'Follow'
  }
})
```

### (2) 팔로잉/팔로워 수 업데이트

JS에서 특정 요소를 선택하고 값을 변경하려면, 먼저 HTML에서 **span 태그로 감싸고 id를 부여**해야 한다.

```html
<!-- accounts/profile.html -->
<div>
  팔로잉 : <span id="followings-count">{{ person.followings.all|length }}</span> /
  팔로워 : <span id="followers-count">{{ person.followers.all|length }}</span>
</div>
```

```javascript
.then((response) => {
  const isFollowed = response.data.is_followed
  const followBtn = document.querySelector('input[type=submit]')
  if (isFollowed === true) {
    followBtn.value = 'Unfollow'
  } else {
    followBtn.value = 'Follow'
  }

  // 팔로잉/팔로워 수 업데이트
  const followingsCountTag = document.querySelector('#followings-count')
  const followersCountTag = document.querySelector('#followers-count')

  followingsCountTag.textContent = response.data.followings_count
  followersCountTag.textContent = response.data.followers_count
})
```

> **주의:** 이 방식은 **현재 보고 있는 페이지의 숫자만 업데이트**한다. 두 개의 탭을 띄워놓고 한쪽에서 팔로우해도 다른 탭이 자동으로 바뀌지 않는다. 두 탭 간 실시간 동기화를 원한다면 **WebSocket(소켓 통신)**을 배워야 한다.

## 💡 한 줄 요약
> axios로 CSRF 토큰을 헤더에 담아 POST 요청하고, Django의 JsonResponse에서 팔로우 상태와 카운트를 받아 JS로 버튼과 숫자를 동적으로 업데이트한다.

## ❓ 더 찾아볼 것
- `event.target` vs `event.currentTarget` — 버블링 상황에서 어떻게 다른가
- `dataset` 속성의 카멜케이스 변환 규칙 (`data-user-id` → `dataset.userId`)
- WebSocket / 소켓 통신 — 실시간 양방향 데이터 동기화
- Django `JsonResponse`의 `safe` 파라미터 (리스트 등 비딕셔너리 직렬화 시 필요)
