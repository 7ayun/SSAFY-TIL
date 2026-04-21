# [Django] 인증 시스템 — 로그아웃·회원가입·회원탈퇴·접근 제어

> 📌 핵심 키워드: #logout #signup #delete #UserCreationForm #get_user_model #is_authenticated #login_required

---

## 학습 목표

* 로그아웃·회원가입·회원탈퇴 기능을 CRUD 관점에서 이해하고 구현할 수 있다
* CustomUserCreationForm을 만들고 get_user_model()을 사용해야 하는 이유를 설명할 수 있다
* is_authenticated로 템플릿과 뷰 함수에서 인증 분기를 처리할 수 있다
* login_required 데코레이터로 뷰 함수에 인증 접근 제어를 적용할 수 있다
* 회원가입 후 자동 로그인, 회원 탈퇴 전 로그아웃 등 추가 개선 사항을 구현할 수 있다

---

## 1. Auth1 복습 — 로그인

### 1-1. 로그인의 핵심 개념

로그인은 **세션을 CREATE하는 과정**이다. HTTP는 비연결성·무상태성을 가지기 때문에, 세션을 생성하고 그 세션 ID를 쿠키로 클라이언트에 보내는 방식으로 인증 상태를 유지한다.

* GET 요청 → 로그인 폼 화면 반환
* POST 요청 → 세션 생성 처리

### 1-2. 로그인 구현 코드 요약

```python
# accounts/views.py
from django.contrib.auth.forms import AuthenticationForm
from django.contrib.auth import login as auth_login

def login(request):
    if request.method == 'POST':
        form = AuthenticationForm(request, request.POST)
        if form.is_valid():
            auth_login(request, form.get_user())
            return redirect('articles:index')
    else:
        form = AuthenticationForm()
    context = {'form': form}
    return render(request, 'accounts/login.html', context)
```

**핵심 포인트:**
* `AuthenticationForm`은 `forms.Form`을 상속하는 폼이다 (ModelForm 아님)
* 로그인 후 DB에 저장하는 게 아니라 세션을 생성하는 것이므로 `save()` 호출 없음
* Django 내장 `login` 함수와 이름 충돌을 피하기 위해 `auth_login`으로 별칭(alias) 사용

---

## 2. 로그아웃 구현

### 2-1. 로그아웃의 개념

로그아웃은 **세션을 DELETE하는 과정**이다. 로그인의 반대 행위로, 서버 DB의 세션 데이터를 삭제하여 인증 상태를 해제한다.

* 게시글 삭제와 달리 URL에 pk를 받을 필요 없다 — 누구의 세션을 삭제할지는 요청에 포함된 쿠키의 세션 ID로 자동으로 알 수 있기 때문이다.
* 삭제 행위이므로 **POST 방식**으로 요청을 보낸다.

### 2-2. URL 설정

```python
# accounts/urls.py
urlpatterns = [
    path('login/', views.login, name='login'),
    path('logout/', views.logout, name='logout'),
]
```

### 2-3. 뷰 함수

```python
# accounts/views.py
from django.contrib.auth import logout as auth_logout

def logout(request):
    if request.method == 'POST':
        auth_logout(request)
    return redirect('accounts:login')
```

Django 내장 `logout` 함수와 이름이 같으므로 `auth_logout`으로 별칭 처리한다. `auth_logout(request)`가 세션 삭제를 모두 처리해 주므로 ORM으로 직접 처리할 필요 없다.

### 2-4. base.html에 로그아웃 버튼 추가

로그아웃은 모든 페이지에서 접근할 수 있어야 하므로 `base.html`에 추가한다. POST 요청이므로 form 태그와 CSRF 토큰이 반드시 필요하다.

```django
<form action="{% url 'accounts:logout' %}" method="POST">
  {% csrf_token %}
  <input type="submit" value="로그아웃">
</form>
```

**CSRF 토큰이 필요한 이유:** DB에 변화를 일으키는 POST 요청은 정상적인 페이지에서 발생한 요청인지 검증해야 한다. CSRF 토큰 없이 요청을 보내면 403 Forbidden이 반환된다.

---

## 3. AbstractUser 심화

### 3-1. AbstractUser의 역할

AbstractUser는 **공통 정보를 여러 유저 모델에 재사용하기 위한 추상 기본 클래스**다. 직접 DB 테이블을 만들지 않고 상속 받는 클래스에 필드를 제공한다.

기본 제공 필드: `username`, `first_name`, `last_name`, `email`, `is_staff`, `is_active`, `date_joined`

현재는 `is_staff`, `is_superuser` 등이 단순 BooleanField(True/False)로 처리되어 있지만, 실제 프로젝트에서는 다음과 같이 다양한 유저를 별도 클래스로 관리할 수 있다.

```python
class GeneralUser(AbstractUser): pass
class DeactivatedUser(AbstractUser): pass  # 비활성화 유저
```

### 3-2. 회원 탈퇴를 DELETE로 처리하면 안 되는 이유

요즘 서비스에서 회원 탈퇴 기능은 실제로 DB에서 즉시 삭제하지 않고 `is_active = False`로 비활성화 처리하는 경우가 많다. 유저 데이터는 서비스 운영에 중요한 자산이기 때문이다. 완전 삭제는 일정 기간(30일, 1년 등) 이후에 처리한다.

---

## 4. 회원가입 구현

### 4-1. 회원가입의 개념

회원가입은 **User 객체를 CREATE하는 과정**이다. DB에 저장이 필요하므로 ModelForm을 사용한다. 로그인의 AuthenticationForm과 달리 `form.save()`를 호출한다.

### 4-2. URL 설정

```python
# accounts/urls.py
urlpatterns = [
    path('login/', views.login, name='login'),
    path('logout/', views.logout, name='logout'),
    path('signup/', views.signup, name='signup'),
]
```

### 4-3. UserCreationForm 사용 — 에러 발생

Django가 제공하는 `UserCreationForm`을 그대로 사용하면 다음 에러가 발생한다.

```
Manager isn't available. auth.User has been swapped for accounts.User
```

이유: `UserCreationForm`은 Django 기본 `auth.User` 모델 기준으로 만들어진 폼이다. 우리가 `AUTH_USER_MODEL = 'accounts.User'`로 변경했지만, 폼 코드 자체는 자동으로 변경되지 않는다.

### 4-4. CustomUserCreationForm 작성

`accounts/forms.py`를 생성하고 `UserCreationForm`을 상속받되, Meta 클래스의 model만 우리 모델로 교체한다.

```python
# accounts/forms.py
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth import get_user_model

class CustomUserCreationForm(UserCreationForm):
    class Meta(UserCreationForm.Meta):
        model = get_user_model()
```

**포인트 1: `UserCreationForm.Meta` 상속**
`UserCreationForm`의 Meta 클래스에는 이미 `password1`, `password2` 필드와 비밀번호 암호화·확인 로직이 포함되어 있다. Meta 클래스 전체를 상속받고 model 하나만 교체하면 나머지 기능을 그대로 활용할 수 있다.

**포인트 2: `get_user_model()` 사용 이유**
```python
# 하드코딩 방식 (권장 X)
from .models import User

# 권장 방식
from django.contrib.auth import get_user_model
User = get_user_model()
```

`get_user_model()`은 **현재 프로젝트에서 활성화된 유저 모델을 반환하는 함수**다. `AUTH_USER_MODEL` 설정값을 기준으로 동적으로 반환하므로, 유저 모델이 변경되더라도 코드를 수정할 필요가 없다. 직접 import하는 방식은 하드코딩이 되어 유지보수가 어려워진다.

### 4-5. 회원가입 뷰 함수

```python
# accounts/views.py
from .forms import CustomUserCreationForm

def signup(request):
    if request.method == 'POST':
        form = CustomUserCreationForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('accounts:login')
    else:
        form = CustomUserCreationForm()
    context = {'form': form}
    return render(request, 'accounts/signup.html', context)
```

### 4-6. signup.html 템플릿

```
accounts/templates/accounts/signup.html
```

```django
{% extends 'base.html' %}

{% block content %}
  <h1>회원가입 페이지</h1>
  <form method="POST">
    {% csrf_token %}
    {{ form.as_p }}
    <input type="submit" value="회원가입">
  </form>
{% endblock content %}
```

`UserCreationForm`의 기본 필드는 `username`, `password1`, `password2` 세 가지다.

* `username`: 150자 이하, 영문자·숫자·특수문자 조합
* `password1`: 8자 이상, 일반적인 단어 지양, 숫자만으로 구성 지양
* `password2`: password1과 동일하게 입력 (오타 확인용)

---

## 5. 회원 탈퇴 구현

### 5-1. 회원 탈퇴의 개념

회원 탈퇴는 **DB에 저장된 User 객체를 삭제하는 과정**이다. 게시글 삭제(`article.delete()`)와 동일한 방식으로 처리하면 된다. `request.user`에 유저 객체가 담겨있으므로 별도로 pk를 URL에 받을 필요 없다.

### 5-2. URL 설정

```python
# accounts/urls.py
path('delete/', views.delete, name='delete'),
```

### 5-3. 뷰 함수

```python
# accounts/views.py
def delete(request):
    if request.method == 'POST':
        request.user.delete()
        return redirect('accounts:login')
    return redirect('articles:index')
```

`request.user`는 현재 요청을 보낸 유저 객체이므로 `.delete()`를 바로 호출할 수 있다.

### 5-4. base.html에 회원 탈퇴 버튼 추가

```django
<form action="{% url 'accounts:delete' %}" method="POST">
  {% csrf_token %}
  <input type="submit" value="회원 탈퇴">
</form>
```

### 5-5. AnonymousUser

`request.user`에는 항상 유저 객체가 존재한다. 로그인하지 않은 사용자도 `AnonymousUser` 객체로 표현된다. AnonymousUser는 DB에 없는 객체이므로 `.delete()`를 호출하면 오류가 발생한다.

---

## 6. is_authenticated를 활용한 템플릿 접근 제어

### 6-1. 템플릿 분기

로그인 여부에 따라 다른 UI를 보여준다. `user.is_authenticated`는 인증된 유저면 `True`, AnonymousUser면 `False`를 반환한다.

```django
{# base.html #}
{% if user.is_authenticated %}
  <p>Hello, {{ user.username }}</p>
  <form action="{% url 'accounts:logout' %}" method="POST">
    {% csrf_token %}
    <input type="submit" value="로그아웃">
  </form>
  <form action="{% url 'accounts:delete' %}" method="POST">
    {% csrf_token %}
    <input type="submit" value="회원 탈퇴">
  </form>
  <a href="{% url 'articles:create' %}">게시글 생성</a>
{% else %}
  <a href="{% url 'accounts:login' %}">로그인</a>
  <a href="{% url 'accounts:signup' %}">회원가입</a>
{% endif %}
```

### 6-2. 뷰 함수에서 분기

이미 로그인된 사용자가 다시 로그인·회원가입 페이지에 접근하는 것을 막는다.

```python
def login(request):
    if request.user.is_authenticated:
        return redirect('articles:index')
    # 이하 로그인 로직
    ...

def signup(request):
    if request.user.is_authenticated:
        return redirect('articles:index')
    # 이하 회원가입 로직
    ...
```

**이유:** 이미 로그인된 상태에서 로그인 페이지에 접근하여 다른 계정으로 로그인하면 세션이 중복 생성될 수 있다. 또한 사용자 경험 측면에서도 이미 로그인됐는데 로그인 페이지가 보이는 것은 어색하다.

---

## 7. login_required 데코레이터

### 7-1. login_required란?

`login_required`는 **로그인되지 않은 사용자가 특정 뷰에 접근하는 것을 차단하는 데코레이터**다. 인증되지 않은 사용자가 해당 뷰에 접근하면 자동으로 로그인 페이지로 redirect 시킨다.

```python
from django.contrib.auth.decorators import login_required
```

### 7-2. is_authenticated와의 차이

| 구분 | is_authenticated | login_required |
|------|-----------------|----------------|
| 형태 | 속성 (True/False) | 데코레이터 |
| 용도 | 조건 분기 처리 | 함수 자체 접근 제어 |
| 미인증 처리 | 직접 코드 작성 | 자동으로 로그인 페이지 redirect |
| 사용 위치 | 템플릿, 뷰 함수 내부 | 뷰 함수 선언 위 |

### 7-3. accounts 뷰에 적용

```python
# accounts/views.py
from django.contrib.auth.decorators import login_required

@login_required
def logout(request):
    if request.method == 'POST':
        auth_logout(request)
    return redirect('accounts:login')

@login_required
def delete(request):
    if request.method == 'POST':
        request.user.delete()
        return redirect('accounts:login')
    return redirect('articles:index')
```

### 7-4. articles 뷰에도 적용 가능

`login_required`는 어디서든 import하여 사용할 수 있다.

```python
# articles/views.py
from django.contrib.auth.decorators import login_required

@login_required
def create(request): ...

@login_required
def delete(request, pk): ...

@login_required
def update(request, pk): ...
```

### 7-5. next 파라미터

`login_required`가 로그인 페이지로 redirect할 때, URL에 `next` 파라미터를 함께 전달한다.

```
/accounts/login/?next=/admin/auth/group/
```

로그인 완료 후 `next` 값의 페이지로 자동 이동한다. 사용자가 원래 접근하려 했던 페이지로 돌아가도록 해주는 기능이다.

### 7-6. 데코레이터 동작 원리

데코레이터는 **함수를 인자로 받아 그 함수의 앞뒤에 추가 로직을 붙여주는 함수**다.

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        print("함수 실행 전")
        result = func(*args, **kwargs)
        print("함수 실행 후")
        return result
    return wrapper

@my_decorator
def say_hello(name):
    print(f"Hello {name}")
```

`login_required`도 같은 원리로 동작한다. 뷰 함수가 실행되기 전에 `is_authenticated`를 확인하고, False면 로그인 페이지로 redirect하며, True면 원래 뷰 함수를 정상 실행한다.

---

## 8. 추가 개선 사항

### 8-1. 회원가입 후 자동 로그인

회원가입 시 이미 아이디와 비밀번호를 입력했으므로, 로그인 페이지로 보내서 다시 입력하게 하는 것은 불필요하다. `form.save()` 후 바로 `auth_login`을 호출한다.

```python
def signup(request):
    if request.method == 'POST':
        form = CustomUserCreationForm(request.POST)
        if form.is_valid():
            user = form.save()
            auth_login(request, user)  # 자동 로그인
            return redirect('articles:index')
    ...
```

### 8-2. 회원 탈퇴 전 로그아웃 처리

회원을 DB에서 삭제하기 전에 세션도 함께 정리해야 한다. 순서는 **로그아웃(세션 삭제) → 유저 삭제** 순으로 처리한다.

```python
@login_required
def delete(request):
    if request.method == 'POST':
        auth_logout(request)    # 먼저 세션 삭제
        request.user.delete()   # 그 다음 유저 삭제
        return redirect('accounts:login')
    return redirect('articles:index')
```

> ⚠️ 순서에 주의: user.delete()를 먼저 하면 이후 request.user가 없어진 상태에서 logout을 시도하므로 문제가 생길 수 있다.

---

## 9. VS Code 단축키 (수업 팁)

| 단축키 | 동작 |
|--------|------|
| `Alt + Shift + ↑↓` | 현재 줄 위/아래로 복제 |
| `Alt + ↑↓` | 현재 줄 위/아래로 이동 |
| `Ctrl + Shift + ←→` | 단어 단위 드래그 선택 |
| `Ctrl + D` | 드래그된 단어와 동일한 단어 동시 선택 (멀티 커서) |
| 더블클릭 | 단어 선택 |
| 트리플클릭 | 한 줄 전체 선택 |
| `Home` / `End` | 줄 맨 앞 / 맨 뒤로 이동 |
| `Shift + Home` / `Shift + End` | 줄 맨 앞 / 맨 뒤까지 드래그 |

---

## 📋 핵심 개념 정리

| 개념 | 설명 | 예시/명령어 |
|------|------|-------------|
| auth_logout | 세션 삭제 함수 (로그아웃) | `from django.contrib.auth import logout as auth_logout` |
| UserCreationForm | Django 내장 회원가입 폼 | `from django.contrib.auth.forms import UserCreationForm` |
| CustomUserCreationForm | 커스텀 유저 모델용 회원가입 폼 | `UserCreationForm`을 상속, Meta.model만 교체 |
| get_user_model() | 현재 활성화된 유저 모델 반환 함수 | `from django.contrib.auth import get_user_model` |
| AnonymousUser | 비로그인 사용자를 표현하는 객체 | DB에 없으므로 `.delete()` 불가 |
| is_authenticated | 로그인 여부 반환 속성 | `request.user.is_authenticated` → True/False |
| login_required | 로그인 필수 데코레이터 | `from django.contrib.auth.decorators import login_required` |
| next 파라미터 | login_required 사용 시 원래 경로 보존 | `/accounts/login/?next=/articles/create/` |
| request.user.delete() | 현재 로그인 유저 삭제 | 회원 탈퇴 처리 |

---

## 참고사항 (수업 후 읽기)

* `login_required`의 `login_url` 파라미터: 기본값은 `settings.LOGIN_URL`이며 `accounts/login/`이 아닌 경우 변경 필요
* `AbstractBaseUser`를 사용한 완전 커스텀 유저 모델 작성 방법
* 세션 관리 심화: 세션 만료 시간, 세션 스토리지 변경 방법
* 회원 탈퇴를 DELETE 대신 `is_active = False`로 처리하는 소프트 딜리트 패턴

---
