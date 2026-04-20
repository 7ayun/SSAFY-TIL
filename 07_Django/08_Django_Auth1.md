# [Django] 인증 시스템 — 쿠키·세션·로그인

> 📌 핵심 키워드: #Cookie #Session #Authentication #AuthenticationForm #AbstractUser #AUTH_USER_MODEL

---

## 학습 목표

* HTTP의 비연결성·무상태성이 인증 문제와 어떻게 연결되는지 이해한다
* 쿠키와 세션의 개념과 동작 원리를 설명할 수 있다
* 커스텀 유저 모델(AbstractUser)을 만들고 AUTH_USER_MODEL로 대체할 수 있다
* AuthenticationForm과 auth_login을 활용해 로그인 기능을 구현할 수 있다
* 템플릿에서 `request.user`와 `user.is_authenticated`를 활용할 수 있다

---

## 1. HTTP의 특성과 인증 문제

### 1-1. 비연결성 (Connectionless)

HTTP는 클라이언트가 요청을 보내고 서버가 응답하면 연결이 끊긴다. 물리적으로 랜선이 연결되어 있는 것과는 별개로, **요청-응답 한 사이클이 끝나면 서버는 해당 연결을 유지하지 않는다.**

실시간 채팅 같은 특수한 경우(WebSocket)는 연결을 유지하지만, 일반적인 웹 서비스에서는 연결을 유지할 이유가 없다. 수억 명의 사용자 상태를 실시간으로 서버에서 계속 유지한다면 리소스 낭비가 극심해지기 때문이다.

### 1-2. 무상태성 (Stateless)

HTTP는 클라이언트와 서버 간의 통신이 끝나면 **상태 정보를 유지하지 않는다.** 즉, 이전 요청에서 로그인을 했더라도 다음 요청에서 서버는 그 사실을 기억하지 못한다.

이 특성으로 인해:

* 로그인 상태를 유지할 수 없다
* 장바구니 정보가 페이지를 넘어가도 유지되지 않는다
* 매 요청마다 아이디·비밀번호를 다시 보내야 하는 불편이 생긴다

이를 해결하기 위해 등장한 것이 **쿠키(Cookie)와 세션(Session)**이다.

---

## 2. 쿠키 (Cookie)

### 2-1. 쿠키란?

쿠키는 **서버가 클라이언트(브라우저)에 전달하는 아주 작은 데이터 조각**이다. 서버가 HTML 문서를 응답할 때 쿠키를 함께 붙여 보내면, 브라우저는 그 쿠키를 클라이언트 로컬(크롬 설치 폴더 내)에 저장한다.

이후 클라이언트가 해당 도메인으로 요청을 보낼 때마다, **쿠키는 요청에 자동으로 포함되어 서버로 전송된다.** 사용자가 별도로 처리하지 않아도 된다.

### 2-2. 쿠키의 동작 방식

서버는 HTTP 응답 헤더의 **`Set-Cookie`** 필드를 통해 클라이언트에게 쿠키를 전달한다. 클라이언트(브라우저)는 이 쿠키를 로컬 파일로 저장해두고, 이후 동일한 서버로 요청을 보낼 때 HTTP 요청 헤더의 **`Cookie`** 필드에 자동으로 포함시켜 전송한다.

쿠키 데이터는 **key-value 형태**로 저장되며, 다음과 같은 속성도 함께 저장된다.

| 속성 | 설명 |
|------|------|
| 데이터 | key-value 형태의 실제 저장 값 (예: `product_id: 13785`) |
| `Expires` | 쿠키 만료 날짜. 이 날짜가 지나면 쿠키가 자동으로 삭제됨 |
| `Domain` | 이 쿠키를 어느 도메인에서 사용하는지 명시. 해당 도메인으로 요청 시에만 자동 전송됨 |

### 2-3. 쿠키에 저장되는 데이터

| 종류 | 설명 |
|------|------|
| 인증 정보 | 암호화된 세션 ID (아이디·비번 자체는 저장 X) |
| 상태 유지 데이터 | 장바구니 목록(예: `product_id: 13785`) |
| 개인화 데이터 | 볼륨 설정, 다크모드 여부 등 |

### 2-3. 쿠키 vs 로컬 스토리지

| 구분 | 쿠키 | 로컬 스토리지 |
|------|------|--------------|
| 서버 전송 | 요청 시 자동 포함 | 자동 전송 X |
| 만료 기한 | 설정 가능 | 별도 설정 없으면 영구 보관 |
| 주요 용도 | 인증·세션 관리 | 비민감 클라이언트 데이터 |

**⚠️ 주의 (발표 때 자주 발생하는 이슈)**

여러 팀이 한 대의 PC에서 발표를 진행할 경우, 앞 팀이 로컬 스토리지에 남긴 인증 정보가 다음 팀 발표에 영향을 줄 수 있다. 로컬 스토리지에는 보안에 민감하지 않은 데이터만 저장하고, 발표 전 초기화 여부를 확인해야 한다.

### 2-4. 쿠키의 주요 용도

* **세션 관리**: 두 요청이 동일한 브라우저에서 들어왔는지 판단
* **개인화**: 사용자 맞춤 서비스 제공 (로그인 상태 유지)
* **사용자 행동 분석**: 어떤 경로로 어느 기능을 사용했는지 추적

---

## 3. 세션 (Session)

### 3-1. 세션이란?

세션은 **서버 측에서 클라이언트-서버 간의 상태를 저장하기 위한 데이터 저장 방식**이다. 쿠키가 클라이언트에 저장되는 것과 달리, 세션은 서버의 데이터베이스에 저장된다.

### 3-2. 세션 동작 흐름

```
1. 클라이언트 → 서버: 로그인 요청 (username, password 전송)
2. 서버: DB에서 계정 확인 후 → 세션 데이터 생성 (세션 ID 발급)
3. 서버 → 클라이언트: 세션 ID를 쿠키에 담아 응답
4. 클라이언트: 쿠키에 세션 ID 저장
5. 이후 모든 요청: 쿠키에 세션 ID 자동 포함하여 전송
6. 서버: 세션 ID로 DB 조회 → 유저 인증 확인
```

### 3-3. 세션 만료와 보안

은행 앱에서 30분 후 자동 로그아웃이 되는 원리가 바로 세션 만료다.

* 로그인 시 서버 DB에 세션 데이터 생성 (만료 시간 30분 설정)
* 사용자가 요청을 보낼 때마다 세션이 갱신되어 30분이 초기화된다
* 30분이 지나면 서버 DB에서 해당 세션 ID가 삭제된다
* 이후 클라이언트가 기존 세션 ID를 보내도 서버는 찾지 못해 인증 거부 → 로그아웃 상태

**세션 ID 탈취 시나리오:**

누군가가 내 쿠키에 있는 세션 ID를 탈취했더라도, 만료 시간(30분)이 지난 뒤에는 서버 DB에서 해당 세션이 삭제되어 탈취된 세션 ID는 더 이상 유효하지 않다.

이처럼 세션 방식은 민감 정보를 주고받지 않고, 만료 시간 설정으로 탈취된 세션 ID의 위험을 시간적으로 제한할 수 있다.

---

## 4. Django 인증 시스템 설정

### 4-1. accounts 앱 생성

인증 기능은 articles 앱에 넣는 것이 아니라 별도 앱으로 분리한다. 앱 이름은 `accounts`를 사용하는 것을 권장한다. Django 내장 인증 기능이 `accounts` 경로로 연결되어 있어 관리가 편하기 때문이다.

```bash
python manage.py startapp accounts
```

생성 후 `settings.py`의 `INSTALLED_APPS`에 등록한다.

```python
INSTALLED_APPS = [
    ...
    'accounts',
]
```

### 4-2. 프로젝트 urls.py 연결

```python
# 프로젝트/urls.py
urlpatterns = [
    ...
    path('accounts/', include('accounts.urls')),
]
```

### 4-3. accounts/urls.py 생성

```python
from django.urls import path
from . import views

app_name = 'accounts'
urlpatterns = [
    # 추후 기능 추가
]
```

---

## 5. 커스텀 유저 모델

### 5-1. 왜 커스텀 유저 모델이 필요한가?

Django는 `django.contrib.auth`에 기본 User 모델을 제공한다. 하지만 기본 User 모델은 필드가 고정되어 있어 이메일 로그인, 전화번호 추가 등 커스터마이징이 어렵다.

**결론: 프로젝트 처음 시작 시, 반드시 커스텀 유저 모델을 만들고 대체해두어야 한다.**

### 5-2. AbstractUser vs AbstractBaseUser

`django.contrib.auth.models`에는 상속 가능한 클래스가 두 가지 있다.

| 클래스 | 포함 내용 | 선택 기준 |
|--------|-----------|-----------|
| `AbstractUser` | username, first_name, last_name, email, is_staff, is_active, date_joined 등 기본 필드 전부 + 비밀번호 처리 로직 | 기본 제공 필드를 그대로 쓰면서 커스터마이징만 하고 싶을 때 (대부분의 경우) |
| `AbstractBaseUser` | 비밀번호 처리 로직만 포함 (last_login, password 등 최소한) | 기본 제공 필드가 불필요하고, 모든 필드를 직접 정의하고 싶을 때 |

### 5-3. AbstractUser 상속

Django의 `AbstractUser`를 상속받으면, 기존 User 모델의 기능(username 유니크 처리, 비밀번호 암호화 등)을 그대로 유지하면서 필드를 자유롭게 추가·수정할 수 있다.

```python
# accounts/models.py
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    pass
```

현재는 아무 필드도 추가하지 않았지만, 나중에 언제든지 필드를 추가할 수 있도록 구조를 갖춰두는 것이 목적이다.

### 5-3. AUTH_USER_MODEL 설정

`settings.py`에서 Django가 사용할 유저 모델을 명시한다. 이 설정이 없으면 Django는 기본 User 모델을 사용한다.

```python
# settings.py
AUTH_USER_MODEL = 'accounts.User'
```

### 5-4. admin.py 등록

```python
# accounts/admin.py
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin
from .models import User

admin.site.register(User, UserAdmin)
```

### 5-5. 마이그레이션

```bash
python manage.py makemigrations
python manage.py migrate
```

**⚠️ 주의**: 이미 migrate를 한 번 진행한 상태에서 AUTH_USER_MODEL을 변경하면 오류가 발생한다. 반드시 **프로젝트 초기에** 유저 모델을 대체해야 한다.

---

## 6. 로그인 기능 구현

### 6-1. 로그인의 HTTP 메서드

로그인은 **세션을 생성(CREATE)하는 행위**이다. 따라서 POST 요청으로 처리한다.

* `GET /accounts/login/` → 로그인 폼 화면 반환
* `POST /accounts/login/` → 세션 생성 처리 후 redirect

### 6-2. URL 설정

```python
# accounts/urls.py
from django.urls import path
from . import views

app_name = 'accounts'
urlpatterns = [
    path('login/', views.login, name='login'),
]
```

### 6-3. AuthenticationForm — ModelForm이 아닌 이유

로그인 폼은 직접 만들 필요 없이 Django가 제공하는 `AuthenticationForm`을 사용한다. username과 password 필드를 기본으로 포함하고 있으며, 유효성 검사도 내장되어 있다.

```python
from django.contrib.auth.forms import AuthenticationForm
```

**중요: AuthenticationForm은 ModelForm이 아닌 `forms.Form`을 상속받는다.**

로그인은 유저 데이터를 DB에 저장(CREATE)하는 행위가 아니라 세션을 생성하는 행위다. 따라서 유효성 검사 후 `form.save()`를 호출하지 않는다.

| 구분 | ModelForm (예: ArticleForm) | AuthenticationForm |
|------|-----------------------------|--------------------|
| 상속 | `forms.ModelForm` | `forms.Form` |
| 목적 | DB에 데이터 저장 | 세션 생성 (인증) |
| 유효성 검사 후 | `form.save()` | `auth_login()` |
| 첫 번째 인자 | 없음 (data=request.POST) | `request` |

**AuthenticationForm의 첫 번째 인자는 `request`다** (일반 Form과 다름):

```python
# 일반 ModelForm
form = ArticleForm(request.POST)

# AuthenticationForm
form = AuthenticationForm(request, request.POST)
#                         ↑ 첫 번째 인자로 request를 받는다
```

### 6-4. login 뷰 함수

Django의 내장 `login` 함수와 이름 충돌을 피하기 위해 `auth_login`이라는 별칭(alias)으로 import한다.

```python
# accounts/views.py
from django.shortcuts import render, redirect
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
    context = {
        'form': form,
    }
    return render(request, 'accounts/login.html', context)
```

**핵심 포인트:**
* `AuthenticationForm(request, request.POST)` → 첫 번째 인자로 `request`를 받는다 (일반 ModelForm과 다름)
* `form.get_user()` → 폼에서 유저 객체를 반환하는 메서드
* `auth_login(request, user)` → 세션 생성 + 쿠키에 세션 ID 설정을 한 번에 처리

### 6-5. login.html 템플릿

```
accounts/
└── templates/
    └── accounts/
        └── login.html
```

```django
{% extends 'base.html' %}

{% block content %}
  <h1>로그인 페이지</h1>
  <form method="POST">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">로그인</button>
  </form>
{% endblock content %}
```

**꿀팁:** `<form>` 태그의 `action` 속성을 비워두면 현재 URL로 POST 요청이 전송된다. 로그인 페이지처럼 같은 경로에서 GET/POST를 함께 처리하는 경우 유용하다.

### 6-6. 로그인 동작 확인

회원가입 기능이 없는 상태에서는 `createsuperuser`로 테스트 계정을 생성하여 확인한다.

```bash
python manage.py createsuperuser
```

로그인 성공 후 브라우저 개발자 도구 → Application → Cookies에서 `sessionid` 값이 생성된 것을 확인할 수 있다. Django 데이터베이스의 `django_session` 테이블에도 같은 키값이 저장된다.

---

## 7. 템플릿에서 인증 데이터 활용

### 7-1. request.user

Django는 모든 요청에서 쿠키 안의 세션 ID를 자동으로 검증하여, 현재 로그인된 유저 객체를 `request.user`에 담아준다.

`render()` 함수에 `request`를 전달하면, 템플릿에서도 별도의 context 설정 없이 `user` 변수를 사용할 수 있다.

```python
# views.py
def index(request):
    articles = Article.objects.all()
    context = {
        'articles': articles,
        # user를 따로 넘기지 않아도 request 덕분에 템플릿에서 사용 가능
    }
    return render(request, 'articles/index.html', context)
```

```django
{# 템플릿에서 사용 가능한 정보 #}
{{ user.username }}
{{ user.email }}
{{ user.first_name }}
```

> 💬 "request에는 사용자가 보내온 모든 요청 정보가 다 들어있다. 요청 보낸 유저가 누군지에 대한 정보도 들어있다."

### 7-2. user.is_authenticated

로그인되지 않은 사용자에게도 `Hello`가 보이는 것은 어색하다. `is_authenticated` 속성을 활용하여 로그인 여부에 따라 화면을 분기할 수 있다.

```django
{% if user.is_authenticated %}
  <p>Hello, {{ user.username }}</p>
{% endif %}
```

* `is_authenticated` → `True`: 로그인된 사용자
* `is_authenticated` → `False`: 비로그인 사용자 (AnonymousUser)

---

## 📋 핵심 개념 정리

| 개념 | 설명 | 예시/명령어 |
|------|------|-------------|
| Connectionless | 요청-응답 후 연결 종료 | HTTP 기본 특성 |
| Stateless | 이전 요청 상태 미보존 | HTTP 기본 특성 |
| Cookie | 서버→클라이언트 저장 소형 데이터 | 세션 ID, 장바구니 정보 |
| Session | 서버 측 상태 저장 방식 | `django_session` 테이블 |
| AbstractUser | Django 기본 User 모델의 추상 클래스 | `from django.contrib.auth.models import AbstractUser` |
| AUTH_USER_MODEL | 프로젝트에서 사용할 유저 모델 지정 | `AUTH_USER_MODEL = 'accounts.User'` |
| AuthenticationForm | Django 내장 로그인 폼 | `from django.contrib.auth.forms import AuthenticationForm` |
| auth_login | 세션 생성 + 쿠키 설정 함수 | `from django.contrib.auth import login as auth_login` |
| form.get_user() | 폼에서 유저 객체 반환 | `auth_login(request, form.get_user())` |
| request.user | 현재 로그인 유저 객체 | `{{ user.username }}` |
| user.is_authenticated | 로그인 여부 (True/False) | `{% if user.is_authenticated %}` |

---
