# [Django] 인증 시스템 — Auth3 (회원정보 수정 · 비밀번호 변경 · 암호화)

> 📌 핵심 키워드: #UserChangeForm #PasswordChangeForm #update_session_auth_hash #해시 #솔트 #키스트레칭

---

## 학습 목표

* 회원정보 수정(Update) 기능을 `CustomUserChangeForm`으로 직접 구현할 수 있다
* `PasswordChangeForm`을 활용해 비밀번호 변경 기능을 구현할 수 있다
* `update_session_auth_hash`를 사용해 비밀번호 변경 후 세션 무효화를 방지할 수 있다
* 해시 함수의 특징과 솔트·키 스트레칭의 역할을 설명할 수 있다
* Django의 비밀번호 저장 방식과 `SECRET_KEY` 관리의 중요성을 이해한다

---

## 1. 오늘 수업 개요

Auth1(로그인), Auth2(회원가입·로그아웃·회원탈퇴)에 이어 Auth3에서는 인증 시스템의 마지막 파트인 **회원정보 수정(Update)** 과 **비밀번호 변경**을 다룬다. 지금까지 CR과 D를 구현했으므로 오늘은 U만 추가하면 회원에 대한 전체 CRUD가 완성된다.

---

## 2. 회원정보 수정 (User Update)

### 2-1. URL 설정

`accounts/urls.py`에 수정 경로를 추가한다. 게시글 수정과 달리 **pk값이 필요 없다**. 수정 대상이 "현재 로그인한 사용자"이고, 그 정보는 이미 `request.user`에 들어 있기 때문이다.

```python
# accounts/urls.py
from django.urls import path
from . import views

app_name = 'accounts'
urlpatterns = [
    # ... 기존 경로
    path('update/', views.update, name='update'),
]
```

### 2-2. View 함수 작성

수정 기능은 반드시 로그인 상태에서만 접근할 수 있어야 하므로 `login_required` 데코레이터를 적용한다.

```python
# accounts/views.py
from django.contrib.auth.decorators import login_required
from .forms import CustomUserChangeForm

@login_required
def update(request):
    if request.method == 'POST':
        form = CustomUserChangeForm(request.POST, instance=request.user)
        if form.is_valid():
            form.save()
            return redirect('articles:index')
    else:
        form = CustomUserChangeForm(instance=request.user)
    context = {'form': form}
    return render(request, 'accounts/update.html', context)
```

**`instance=request.user` 누락 시 발생하는 오류**: `instance`를 넣지 않으면 Django는 기존 데이터를 수정하는 것이 아니라 새 유저 객체를 **생성**하려 한다. 그 결과 `UNIQUE constraint failed: account_user.username` — 즉 `IntegrityError`가 발생한다.

> ⚠️ **`is_valid()`의 한계**: `is_valid()`는 **form 필드의 제약사항**을 기반으로 유효성을 검사한다. form에 `username` 필드가 없으므로 `is_valid()`는 통과하지만, DB 레벨의 UNIQUE 제약에 걸려 `IntegrityError`가 발생한다. 즉, `is_valid()` 통과 ≠ DB 저장 성공을 의미하지 않는다.

**GET에서도 `instance=request.user`가 필요한 이유**: POST 처리에만 `instance=`가 필요하다고 오해할 수 있지만, GET 요청에서도 `instance=request.user`를 넣어야 한다. GET 시 `instance=`가 있어야 폼이 현재 유저의 **기존 데이터로 미리 채워진다(prefill)**. 회원가입 시 `first_name`, `last_name`, `email`을 입력하지 않았다면 비어 있는 것이 정상이며, 수정 후 다시 페이지를 열면 수정된 값이 표시되어야 한다.

### 2-3. CustomUserChangeForm 정의

Django가 기본 제공하는 `UserChangeForm`을 그대로 사용하면 `last_login`, `is_superuser`, `user_permissions`, `groups` 등 일반 사용자가 수정해서는 안 되는 필드까지 노출된다.

이 문제가 발생하는 **근본 원인**은 `UserChangeForm` 내부의 `Meta` 클래스에 `fields = '__all__'`이 설정되어 있기 때문이다. User 모델이 가진 모든 필드를 그대로 폼에 표시하므로, 아무 유저나 프로필 수정 페이지에서 `is_superuser` 체크박스를 조작해 관리자 권한을 스스로 부여할 수 있는 심각한 보안 허점이 생긴다.

따라서 `UserChangeForm`을 상속받되 `fields`만 안전한 항목으로 덮어쓰는 커스텀 폼을 만든다.

```python
# accounts/forms.py
from django.contrib.auth.forms import UserCreationForm, UserChangeForm
from django.contrib.auth import get_user_model

class CustomUserChangeForm(UserChangeForm):
    class Meta:
        model = get_user_model()
        fields = ('first_name', 'last_name', 'email',)
```

* `username`은 고유 식별자이므로 사용자가 임의로 변경하지 못하도록 제외한다.
* `password`는 별도 과정(비밀번호 변경)을 통해 처리하므로 여기서 제외한다.
* `email`은 현재 모델에서 `blank=True`로 설정되어 있어 선택 사항으로 수정 가능하다.

`views.py`에서는 처음에 `UserChangeForm`을 직접 import해서 사용했지만, `CustomUserChangeForm`을 만든 뒤에는 더 이상 필요 없다. **사용하지 않는 import는 삭제**하고 커스텀 폼으로 교체한다.

```python
# ❌ 수정 전 (UserChangeForm을 직접 import해서 사용하던 상태)
from django.contrib.auth.forms import UserChangeForm
# form = UserChangeForm(...)

# ✅ 수정 후 (CustomUserChangeForm으로 교체)
from .forms import CustomUserChangeForm
# form = CustomUserChangeForm(...)
```

즉, `views.py` 상단 import 순서는 다음과 같이 정리된다.

```python
# accounts/views.py 최종 import 정리
from django.contrib.auth.decorators import login_required
from django.contrib.auth.forms import PasswordChangeForm   # 비밀번호 변경용
from django.contrib.auth import update_session_auth_hash
from .forms import CustomUserChangeForm                    # 회원정보 수정용 (UserChangeForm 직접 import 제거)
```

### 2-4. 템플릿 작성 및 링크 연결

```html
<!-- accounts/templates/accounts/update.html -->
{% extends 'base.html' %}

{% block content %}
  <h1>프로필 업데이트</h1>
  <form method="POST">
    {% csrf_token %}
    {{ form.as_p }}
    <input type="submit" value="수정">
  </form>
{% endblock content %}
```

`base.html`의 로그인 상태 블록에 링크를 추가한다.

```html
<!-- base.html (로그인 상태 블록 내) -->
<a href="{% url 'accounts:update' %}">회원정보 수정</a>
```

---

## 3. 비밀번호 변경 (Password Change)

### 3-1. 비밀번호를 별도로 처리하는 이유

회원정보 수정 페이지에서 `CustomUserChangeForm`을 완성하면, 상속받은 `UserChangeForm`으로 인해 하단에 **"비밀번호 변경"** 링크(`../password`)가 자동으로 표시된다. 이는 Django가 비밀번호 처리를 별도 폼(`PasswordChangeForm`)으로 분리해두었기 때문이며, 비밀번호 변경에는 세션 재발급 등 추가 처리가 필요하기 때문이다.

### 3-2. URL 설정

현재 버전 기준으로 `UserChangeForm`이 생성하는 비밀번호 변경 링크는 `../password` — **상대 경로** 형태다. `accounts/update/` 페이지에서 클릭하면 `..`이 `update`의 상위인 `accounts/`를 가리키므로, 최종 경로는 `accounts/password/`가 된다.

Django 버전별 비밀번호 변경 URL 경로 변화 이력:

| 버전 | 경로 형태 |
|---|---|
| 구버전 | `127.0.0.1:8000/password` |
| 중간 버전 | `accounts/{pk}/password` |
| 이전 버전 | `accounts/update/password` |
| **현재 버전** | **`accounts/password/`** |

`urls.py`에는 현재 버전 기준으로 다음과 같이 등록한다.

```python
# accounts/urls.py
path('password/', views.password_change, name='password_change'),
```

### 3-3. View 함수 작성

`PasswordChangeForm`은 **ModelForm이 아닌 일반 Form**이다. 따라서 `instance` 키워드 인자를 사용하지 않고, **첫 번째 인자로 `request.user`를 직접 전달**한다.

> ⚠️ **실수 포인트 — 아이러니한 실수**: `update` view를 작성할 때 `instance=request.user`를 빠뜨려 `IntegrityError`가 발생했다. 그 전철을 밟지 않으려고 `password_change` view의 GET 코드를 작성할 때 "이번엔 처음부터 `instance=` 넣자"고 의도적으로 추가했는데, 이것이 오히려 새로운 실수가 됐다. `PasswordChangeForm`은 ModelForm이 아니기 때문에 `instance=` 키워드 자체가 지원되지 않는다.

```python
# ❌ 잘못된 코드 — PasswordChangeForm은 ModelForm이 아니므로 instance= 사용 불가
form = PasswordChangeForm(request.POST, instance=request.user)

# ✅ 올바른 코드 — request.user를 첫 번째 위치 인자로 직접 전달
form = PasswordChangeForm(request.user, request.POST)
```

강의 중 수강생의 지적으로 즉시 수정하는 상황이 발생했다. `PasswordChangeForm`의 생성자 시그니처 자체가 첫 번째 인자로 유저 객체를 받도록 설계되어 있기 때문이다.

최종 view 함수는 다음과 같다.

```python
# accounts/views.py
from django.contrib.auth.forms import PasswordChangeForm
from django.contrib.auth import update_session_auth_hash

@login_required
def password_change(request):
    if request.method == 'POST':
        form = PasswordChangeForm(request.user, request.POST)
        if form.is_valid():
            user = form.save()
            update_session_auth_hash(request, user)
            return redirect('articles:index')
    else:
        form = PasswordChangeForm(request.user)
    context = {'form': form}
    return render(request, 'accounts/password_change.html', context)
```

| 폼 | 상속 | 첫 번째 인자 |
|---|---|---|
| `AuthenticationForm` | Form | `request` |
| `PasswordChangeForm` | Form | `request.user` |
| `CustomUserChangeForm` | ModelForm | — (POST 데이터와 `instance=request.user`) |

### 3-4. 템플릿 작성

```html
<!-- accounts/templates/accounts/password_change.html -->
{% extends 'base.html' %}

{% block content %}
  <h1>비밀번호 변경</h1>
  <form method="POST">
    {% csrf_token %}
    {{ form.as_p }}
    <input type="submit" value="변경">
  </form>
{% endblock content %}
```

`PasswordChangeForm`에는 현재 비밀번호 + 새 비밀번호 + 새 비밀번호 확인, 세 가지 필드가 자동으로 렌더링된다.

---

## 4. 세션 무효화 방지 (`update_session_auth_hash`)

### 4-1. 문제 상황 — 비밀번호 변경 후 자동 로그아웃

처음 작성한 `password_change` view는 `form.save()` 후 바로 리다이렉트하는 구조였다.

```python
# 처음 작성한 코드 (문제 있음)
if form.is_valid():
    form.save()
    return redirect('articles:index')  # 이동하면 로그아웃 상태가 됨
```

이 상태로 서버를 실행해서 실제로 비밀번호를 변경하면, `articles:index`로 이동한 뒤 **로그인이 풀려 있는 것을 확인할 수 있다**. 이유는 다음과 같다.

비밀번호가 변경되면 Django는 세션 해시를 새로 계산한다. 그러나 브라우저 쿠키에 남아있는 세션 ID는 변경 전 것이므로, 서버의 새 세션 해시와 불일치하여 인증이 무효화된다. `auth_login`은 로그인 시 세션을 새로 만들어 쿠키를 갱신해줬지만, `form.save()`만으로는 이 역할을 수행하지 않기 때문이다.

### 4-2. 세션 동작 원리 복습

```
[로그인] → 서버: 세션 생성 → 세션 ID를 쿠키로 사용자에게 전달
[요청]   → 사용자: 쿠키의 세션 ID를 서버로 전송
[서버]   → DB의 세션 ID와 비교 → 일치하면 인증된 사용자로 처리
```

비밀번호가 변경되면 세션 해시값이 바뀐다. 기존 쿠키는 구 세션 ID를 갖고 있으므로 인증이 무효화된다.

### 4-3. 해결: `update_session_auth_hash`

위 문제를 해결하기 위해 `form.save()` 직후에 `update_session_auth_hash`를 추가한다. 이 함수는 `auth_login`이 로그인 시 해줬던 "세션 생성 → 쿠키 갱신" 역할을 비밀번호 변경 상황에서도 수행해준다.

```python
from django.contrib.auth import update_session_auth_hash

# form.save()의 반환값 = 변경이 반영된 새 유저 객체
user = form.save()
update_session_auth_hash(request, user)  # 새 유저 객체로 세션 갱신
```

이 함수는 비밀번호가 변경된 유저 객체를 기반으로 **새 세션을 생성하여 쿠키를 업데이트**한다. 결과적으로 사용자는 비밀번호를 바꾼 뒤에도 로그인 상태가 유지된다.

**보안적 의의**: 쿠키가 탈취당했더라도 비밀번호를 변경하면 기존 세션 ID가 무효화된다. 모든 디바이스에서 로그아웃하는 서비스 기능의 원리가 바로 이것이다.

| `form.save()` 반환값에 주의 | 설명 |
|---|---|
| `update_session_auth_hash(request, request.user)` | ❌ 비밀번호 변경 전 유저 객체 |
| `update_session_auth_hash(request, form.save())` | ✅ 비밀번호가 변경된 새 유저 객체 |

---

## 5. 비밀번호 암호화

### 5-1. 평문 저장의 위험성

사용자가 입력한 비밀번호를 그대로 데이터베이스에 저장하면, 데이터베이스가 탈취되었을 때 해당 비밀번호가 그대로 노출된다. 외부 해커뿐 아니라 내부 직원에 의한 유출 위험도 존재한다.

단순 인코딩(예: UTF-8 → CP-949)은 디코딩으로 원문을 복구할 수 있으므로 의미가 없다. 카이사르 암호처럼 가역(可逆) 변환이기 때문이다.

### 5-2. 해시 함수

해시 함수의 핵심 특징:

| 특징 | 설명 |
|---|---|
| 단방향성 | 해시값으로 원문을 복원할 수 없다 |
| 결정론성 | 같은 입력 → 항상 같은 해시값 |
| 고정 출력 길이 | 입력 크기와 무관하게 출력 길이는 항상 고정 |
| 눈사태 효과 | 입력값이 한 글자만 바뀌어도 해시값이 완전히 달라진다 |

Django는 **SHA-256** 해시 함수를 사용한다. 256비트 길이의 16진수 난수로 변환하며, 현재 알려진 해시 알고리즘 중 복호화가 가장 어렵다.

**로그인 시 비밀번호 검증 원리**:

```
[회원가입] 비밀번호 "A" → SHA-256 → "1638FE" → DB 저장
[로그인]   입력값 "A" → SHA-256 → "1638FE" → DB의 "1638FE"와 비교 → 일치 → 인증 성공
```

### 5-3. 브루트포스 공격과 레인보우 테이블

해시 함수는 단방향이므로 복호화는 불가능하다. 그러나 해커는 두 가지 방법을 사용한다.

**브루트포스(Brute Force)**: 가능한 모든 문자열을 해시하여 대조한다. 컴퓨터 연산 속도가 빠르기 때문에 단순하고 짧은 비밀번호는 위험하다. 실제 해커는 A부터 Z까지 무작정 넣는 것이 아니라, 알고리즘 프루닝처럼 **자주 쓰이는 비밀번호 패턴**을 후보군으로 추려서 시도한다.

**레인보우 테이블(Rainbow Table)**: 자주 사용되는 비밀번호들의 해시값을 미리 계산해 저장해 둔 테이블이다. 탈취한 DB의 해시값을 레인보우 테이블과 대조하면 비밀번호를 빠르게 찾아낼 수 있다.

**연쇄 해킹(하나 털리면 다 털리는 이유)**: 여러 사이트에 `1234!`, `1234@`처럼 패턴이 비슷한 비밀번호를 쓰면, A사 비밀번호가 유출됐을 때 해커는 `1234!` 대입 실패 → `1234@`, `1234#`, `4321!` 등 변형을 자동으로 시도한다. 결국 B사 계정도 뚫린다. 각 사이트마다 완전히 다른 비밀번호를 써야 하는 이유가 바로 이것이다.

### 5-4. 솔트(Salt)

레인보우 테이블 공격을 막기 위해 비밀번호에 랜덤한 추가 값을 붙인 뒤 해싱한다. 이것이 **솔트(Salt)**다.

```
원본 비밀번호: "1234!@"
솔트 값:       "#random_salt"
해싱 대상:     "1234!@#random_salt" → SHA-256 → "a7f3c..."
```

솔트가 다르면 동일한 비밀번호도 완전히 다른 해시값이 되므로, 기존 레인보우 테이블이 무효화된다. Django는 솔트 값을 해시값과 함께 데이터베이스에 저장하지만, 솔트만으로는 비밀번호를 복호화할 수 없다.

### 5-5. 키 스트레칭(Key Stretching)

현대의 컴퓨터는 연산 속도가 매우 빠르므로, 브루트포스 시도 횟수 자체가 많아진다. 이를 방어하기 위해 해시 연산을 **수만~수십만 번 반복**하여 한 번의 비밀번호 검증에 드는 연산량을 의도적으로 늘린다. 이를 **키 스트레칭**이라 한다.

```
일반 해시:      비밀번호 10억 개 브루트포스 → 수십 초
키 스트레칭:    비밀번호 10억 개 브루트포스 → 사실상 불가능한 시간
```

사용자 입장에서는 단 한 개의 비밀번호만 검증하므로 체감 성능 저하가 없다. Big-O 관점에서 상수항에 불과하기 때문이다.

### 5-6. Django의 비밀번호 저장 형식

Django는 **PBKDF2** 알고리즘(키 스트레칭 방식의 하나)을 기본으로 사용하며, 비밀번호를 다음 형식으로 저장한다.

```
알고리즘$이터레이션(횟수)$솔트$해시값
예: pbkdf2_sha256$1000000$랜덤솔트$해시값
```

Django 프로젝트마다 키 스트레칭 횟수는 달라질 수 있다.

### 5-7. settings.py의 SECRET_KEY

Django 프로젝트의 `settings.py`에는 `SECRET_KEY`가 있다. 이 키는 비밀번호 암호화, CSRF 토큰 생성 등 다양한 보안 처리에 사용된다.

> ⚠️ `SECRET_KEY`는 절대로 GitHub 등 공개 저장소에 업로드해선 안 된다. `.env` 파일 등으로 분리하여 관리해야 하며, `.gitignore`에 추가해야 한다.

### 5-8. 검증된 프레임워크를 쓰는 이유

암호화 과정 전체를 직접 구현하면 실수할 가능성이 높다. Django는 SHA-256, 솔트, PBKDF2 키 스트레칭까지 모두 내장하고 있으므로, 직접 구현하기보다 **Django가 제공하는 기능을 사용하는 것이 훨씬 안전하다**.

프레임워크(Django, Vue 등)를 사용하는 것이 "제대로 된 개발이 아니다"는 시각은 잘못됐다. AI로 개발하는 것이 잘못됐다고 하는 것과 다르지 않다. **쓸 수 있는 도구를 잘 사용하는 것도 실력이다.** 단, 도구를 그냥 외워서 쓰는 것이 아니라 **왜(Why) 이 도구가 이렇게 동작하는지**를 이해하는 데 집중해야 한다.

---

## 6. 참고 라인

### 6-1. 비밀번호 초기화 기능

비밀번호 변경 대신 초기화 후 이메일로 임시 비밀번호를 전송하는 방식도 많이 쓰인다. Django는 이를 위한 `SetPasswordForm`을 제공한다.

**`SetPasswordForm` 인자 순서 주의**: `PasswordChangeForm`과 달리, 첫 번째 인자로 유저 객체(`user`)를 받는다.

```python
from django.contrib.auth.forms import SetPasswordForm

form = SetPasswordForm(user, request.POST)  # 첫 번째 인자: user 객체
```

### 6-2. 피싱 사이트 주의

비밀번호 암호화가 완벽하더라도, 피싱 사이트를 통한 탈취에는 무방비하다.

**암호화가 이루어지는 시점**: 비밀번호는 사용자가 폼에서 전송한 뒤, **데이터베이스 서버에 저장할 때** 암호화된다. 전송 과정(네트워크)에서 암호화되는 것이 아니다. 따라서 폼 전송 단계에서 비밀번호는 **평문(문자열) 상태**다.

피싱 사이트는 실제 서비스와 동일하게 HTML/CSS를 복제하여 만들 수 있으며, `<form>` 태그의 `action` 속성을 해커 서버로 향하도록 설정한다. 사용자가 입력한 비밀번호 **평문이 암호화되기 전에 그대로 해커에게 전달**되므로, 아무리 DB 암호화가 완벽해도 아무 소용이 없다.

> ⚠️ URL 주소창을 반드시 확인하는 습관을 기른다. `https://` 여부와 도메인 이름을 꼼꼼히 확인한다.

### 6-3. 환경변수 관리

`settings.py`의 `SECRET_KEY`, API 키 등 민감한 정보는 `.env` 파일에 분리하고 `.gitignore`에 추가한다. 단, `.env` 파일 자체를 분실하지 않도록 별도의 안전한 저장소에 보관해야 한다.

---

## 📋 핵심 개념 정리

| 개념 | 설명 | 예시/명령어 |
|---|---|---|
| UserChangeForm | Django 제공, 유저 정보 수정용 ModelForm | `from django.contrib.auth.forms import UserChangeForm` |
| CustomUserChangeForm | UserChangeForm 상속, 노출 필드 제한 | `fields = ('first_name', 'last_name', 'email',)` |
| instance=request.user | 수정 대상 지정 (누락 시 IntegrityError) | `form = CustomUserChangeForm(request.POST, instance=request.user)` |
| PasswordChangeForm | 비밀번호 변경용 Form (ModelForm 아님) | 첫 번째 인자: `request.user` |
| update_session_auth_hash | 비밀번호 변경 후 세션 유지 | `update_session_auth_hash(request, user)` |
| 해시(Hash) | 단방향 암호화, 복원 불가 | SHA-256 → 256비트 16진수 난수 |
| 솔트(Salt) | 비밀번호에 랜덤값 추가 → 레인보우 테이블 방어 | `"1234!@" + "#salt"` → 해싱 |
| 키 스트레칭(Key Stretching) | 해싱을 수십만 번 반복 → 브루트포스 방어 | PBKDF2 알고리즘 |
| Django 비밀번호 저장 형식 | 알고리즘·반복횟수·솔트·해시를 `$`로 구분 저장 | `pbkdf2_sha256$1000000$salt$hash` |
| SECRET_KEY | 암호화·CSRF 등 보안 처리의 근간 | `.env`로 분리, GitHub 업로드 금지 |
| SetPasswordForm | 비밀번호 초기화용 (인자: `user`, `request.POST`) | `SetPasswordForm(user, request.POST)` |
