# [Django] Templates — DTL · 정적 파일 · 폼 · URL 심화

> 📌 핵심 키워드: #DTL #템플릿상속 #context #MTV #VariableRouting #URLNamespace

---

## 학습 목표

* Django 템플릿 시스템(DTL)의 4가지 구성 요소(변수·필터·태그·주석)를 설명하고 사용할 수 있다
* View에서 context(딕셔너리)를 템플릿으로 전달하고, for·if 태그로 렌더링할 수 있다
* HTML form 태그로 데이터를 서버에 전송하고, `request.GET`으로 수신하는 흐름을 구현할 수 있다
* Variable Routing(`<int:num>`)으로 URL에 변수를 포함하고 View에서 활용할 수 있다
* URL 분리·별명(name)·앱 네임스페이스(app_name)로 URL을 체계적으로 관리할 수 있다

---

## 1. 복습 — 어제 배운 흐름

### 1-1. Django 프로젝트 기본 세팅 순서

```bash
# 1. 가상환경 생성 & 활성화
python -m venv venv
source venv/Scripts/activate          # Windows
# source venv/bin/activate            # macOS/Linux

# 2. Django 설치
pip install django

# 3. 패키지 목록 저장 (협업 필수 습관)
pip freeze > requirements.txt
# 팀원이 받을 때: pip install -r requirements.txt

# 4. .gitignore 설정 (gitignore.io → django 검색 후 붙여넣기)

# 5. 프로젝트 생성
django-admin startproject startpjt .   # . : 현재 폴더에 생성

# 6. 앱 생성
python manage.py startapp articles

# 7. settings.py → INSTALLED_APPS 에 앱 등록
#    ※ 등록 안 하면 프로젝트가 앱을 인식 못 함 → 자주 하는 실수!

# 8. 서버 실행
python manage.py runserver             # 기본 포트 8000
python manage.py runserver 7777        # 포트 직접 지정 가능
```

> 💬 "venv는 200~300MB라 GitHub에 올리지 않는다. requirements.txt로 패키지 목록만 공유하는 것이 정석이다."

### 1-2. 요청·응답 3단계 흐름

```
① URL 등록   urls.py      path('index/', views.index)
② View 작성  views.py     def index(request): return render(request, '...')
③ Template   .html 파일   실제 화면 출력
```

> 💬 "URL은 유저의 진입점(Entry Point). 주소 → 함수 → 템플릿 이 순서를 항상 기억하라."

### 1-3. 127.0.0.1과 포트 번호

`127.0.0.1`은 내 로컬 컴퓨터를 의미하는 주소다. 포트 번호(`8000`, `8888` 등)는 그 컴퓨터 안에서 돌아가는 서버를 구분하는 번호다. 로컬에 서버가 여러 개 동시에 떠 있을 수 있다.

---

## 2. Django 템플릿 시스템 (DTL)

**DTL(Django Template Language)** — Django가 HTML에 파이썬 로직을 녹여낼 수 있도록 제공하는 전용 문법. 파이썬 문법과 유사하지만 별개의 시스템이다.

| 구성요소 | 문법 | 설명 |
|---------|------|------|
| **변수(Variable)** | `{{ variable }}` | View에서 넘긴 context 값 출력 |
| **필터(Filter)** | `{{ variable \| filter }}` | 변수에 가공 함수 적용 |
| **태그(Tag)** | `{% tag %}` | 제어 흐름 (반복·조건 등) |
| **주석(Comment)** | `{# 주석 #}` | 렌더링 시 제외 |

---

## 3. 변수 (Variable)

### 3-1. context 전달 방법

View 함수에서 **딕셔너리**를 `render()`의 세 번째 인자로 넘긴다.

```python
# articles/views.py
from django.shortcuts import render

def index(request):
    context = {
        'name': 'Jaden',       # 키: 템플릿에서 사용할 이름
        'numbers': [1, 2, 3],  # 리스트도 전달 가능
    }
    return render(request, 'articles/index.html', context)
```

```html
<!-- articles/templates/articles/index.html -->
<h1>{{ name }}님 안녕하세요!</h1>  <!-- Jaden님 안녕하세요! -->
```

> 💬 "context라는 이름은 관례다. 중요한 건 딕셔너리 형태로 만드는 것."

### 3-2. 딕셔너리·객체 속성 접근

DTL에서는 `.`(점)으로 딕셔너리 키 / 리스트 인덱스 / 객체 속성에 모두 접근한다.

```html
{{ my_dict.key }}       <!-- 딕셔너리 값 -->
{{ my_list.0 }}         <!-- 리스트 첫 번째 요소 -->
{{ my_obj.name }}       <!-- 객체 속성 -->
```

---

## 4. 필터 (Filter)

변수 뒤에 `|`(파이프)를 붙여 적용하며, 공식 문서 Built-in Filter Reference를 참고한다.

```html
{{ name | lower }}              <!-- 소문자 변환 -->
{{ name | upper }}              <!-- 대문자 변환 -->
{{ content | truncatewords:2 }} <!-- 단어 2개까지만 출력 후 ... -->
{{ number | add:10 }}           <!-- 숫자 더하기 -->
{{ today | date:"Y년 m월 d일" }} <!-- 날짜 포맷 -->
```

> 💬 "필터는 가급적 쓰지 않는 게 좋다. 숫자 더하기나 글자 자르기 같은 로직 처리는 백엔드(View) 레벨에서 이미 처리해서 넘겨주는 게 정석이다. 계산을 사용자 브라우저에게 넘기지 말고, 서버가 처리해서 던지는 것이 기본 원칙."

---

## 5. 태그 (Tag)

`{% %}` 안에 제어 흐름 문법을 작성한다. **반드시 닫는 태그**(`{% end~ %}`)가 필요하다.

### 5-1. if 조건문

```html
{% if user_list %}
    <p>유저가 있습니다.</p>
{% elif room_list %}
    <p>방이 있습니다.</p>
{% else %}
    <p>아무것도 없습니다.</p>
{% endif %}
```

### 5-2. for 반복문

```html
{% for article in articles %}
    <p>{{ article.title }}</p>
{% empty %}
    <p>게시글이 없습니다.</p>  <!-- 리스트가 비어 있을 때 -->
{% endfor %}
```

### 5-3. for 반복문 내 특수 변수

| 변수 | 설명 |
|------|------|
| `forloop.counter` | 1부터 시작하는 인덱스 |
| `forloop.counter0` | 0부터 시작하는 인덱스 |
| `forloop.first` | 첫 번째 순회 여부 (True/False) |
| `forloop.last` | 마지막 순회 여부 (True/False) |

```html
{% for item in items %}
    {% if forloop.first %}
        <strong>{{ item }}</strong>   <!-- 첫 번째만 굵게 -->
    {% else %}
        <span>{{ item }}</span>
    {% endif %}
{% endfor %}
```

### 5-4. DTL 실습 예시 — 랜덤 메뉴 선택기 (dinner 페이지)

View에서 데이터 가공 후 context로 전달하고, 템플릿에서 for·if로 출력하는 전형적인 패턴이다.

```python
# articles/views.py
import random

def dinner(request):
    foods = ['국밥', '국수', '카레', '탕수육']
    picked = random.choice(foods)     # 리스트에서 랜덤하게 하나 선택
    context = {
        'foods': foods,
        'picked': picked,
    }
    return render(request, 'articles/dinner.html', context)
```

```html
<!-- articles/templates/articles/dinner.html -->
<h2>오늘의 추천 메뉴: {{ picked }}</h2>
<ul>
    {% for food in foods %}
        <li>{{ food }}</li>
    {% empty %}
        <li>메뉴가 없습니다.</li>
    {% endfor %}
</ul>
```

> 💬 "어떤 데이터를 가공하는 영역은 반드시 View에 작성한다. 템플릿은 출력만 담당한다."

---

## 6. 정적 파일 (Static Files)

**Static file** — CSS, JS, 이미지처럼 서버 로직 없이 그대로 제공되는 파일.

### 6-1. 앱 내 정적 파일 경로 구조

```
articles/
  static/
    articles/        ← 앱 이름 폴더 (네임스페이스 역할)
      style.css
      images/
        logo.png
```

### 6-2. 템플릿에서 정적 파일 로드

```html
{% load static %}  <!-- 반드시 파일 최상단에 선언 -->
<!DOCTYPE html>
<html>
<head>
    <link rel="stylesheet" href="{% static 'articles/style.css' %}">
</head>
<body>
    <img src="{% static 'articles/images/logo.png' %}" alt="로고">
</body>
</html>
```

> 💬 "`{% load static %}`을 선언하지 않으면 `{% static %}` 태그를 인식 못 해서 에러가 난다. 꼭 최상단에 써야 한다."

### 6-3. settings.py 정적 파일 설정

```python
# settings.py (기본값 확인)
STATIC_URL = 'static/'
# STATICFILES_DIRS = [BASE_DIR / 'static']  # 프로젝트 공통 static 폴더 사용 시
```

---

## 7. 템플릿 상속 (Template Inheritance)

반복되는 HTML 구조(navbar, footer 등)를 **base 템플릿** 하나로 관리하고, 자식 템플릿에서 재사용하는 방법.

### 7-1. 상속 구조

```
templates/
  base.html          ← 부모 (공통 레이아웃)
articles/
  templates/
    articles/
      index.html     ← 자식 (실제 내용)
```

### 7-2. base.html (부모 템플릿)

```html
{% load static %}
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>My Site</title>
    <link rel="stylesheet" href="{% static 'articles/style.css' %}">
</head>
<body>
    <nav>
        <a href="/">홈</a>
    </nav>

    {% block content %}
    <!-- 자식 템플릿이 이 영역을 채운다 -->
    {% endblock content %}

    <footer>공통 푸터</footer>
</body>
</html>
```

### 7-3. 자식 템플릿 (index.html)

```html
{% extends 'base.html' %}   <!-- 반드시 파일 첫 번째 줄 -->

{% block content %}
    <h1>Articles 목록</h1>
    <ul>
        {% for article in articles %}
            <li>{{ article.title }}</li>
        {% endfor %}
    </ul>
{% endblock content %}
```

> 💬 "`extends`는 반드시 파일의 맨 첫 번째 줄에 와야 한다. 그렇지 않으면 에러가 발생한다."

`extends`는 하나의 부모만 가질 수 있다. `block`은 자식이 재정의할 영역을 표시하는 것이고, `block` 이름으로 위치를 구분한다.

### 7-4. settings.py — 프로젝트 공통 templates 경로 등록

프로젝트 루트에 공통 `base.html`을 두고 모든 앱에서 공유하려면 `TEMPLATES`의 `DIRS`에 경로를 추가한다.

```python
TEMPLATES = [
    {
        'DIRS': [BASE_DIR / 'templates'],  # 프로젝트 루트 templates 폴더 등록
        ...
    }
]
```

---

## 8. include 태그

반복되는 **부분 컴포넌트**(네비게이션 바, 카드 등)를 별도 파일로 분리해 재사용한다.

```html
<!-- base.html 또는 다른 템플릿 어디서든 -->
{% include 'articles/_nav.html' %}
```

```html
<!-- articles/templates/articles/_nav.html -->
<nav>
    <a href="/">홈</a>
    <a href="/articles/">게시판</a>
</nav>
```

> 💬 "include는 단순한 HTML 조각을 가져오는 것. extends는 레이아웃 구조를 물려받는 것. 두 가지를 혼동하지 말 것."

---

## 9. templates 폴더 위치 & 네임스페이스

### 9-1. 왜 앱 이름 폴더를 한 번 더 만드는가?

```
articles/templates/articles/index.html   ← 권장
articles/templates/index.html            ← 비권장
```

두 앱에 동일한 파일명이 있을 때 Django는 `INSTALLED_APPS` 순서대로 검색한다. 앱 이름 폴더를 만들지 않으면 **다른 앱의 index.html**이 실수로 로드될 수 있다.

```python
# views.py
return render(request, 'articles/index.html', context)
#                       ↑ 앱 이름/파일명 형태로 명시
```

---

## 10. 폼(Form)으로 데이터 전송

### 10-1. HTML form 태그 기본 구조

사용자가 입력한 데이터를 서버로 전송하는 가장 보편적인 방법이 HTML `<form>` 태그다. 네이버·구글 모두 동일한 방식을 사용한다.

```html
<form action="전송할_URL" method="GET">
    <label for="message">검색어</label>
    <input type="text" name="query" id="message">
    <input type="submit" value="검색하기">
</form>
```

- **action**: 폼을 제출했을 때 데이터가 전달될 URL
- **method**: 전송 방식 (GET 또는 POST)
- **name**: URL 쿼리 스트링의 키가 된다

```
검색어 '안녕하세요' 제출 시 URL:
https://search.naver.com/search.naver?query=안녕하세요
                                       ↑ name 값이 키가 됨
```

> 💬 "name 속성이 바로 URL 쿼리 스트링의 키가 된다. 네이버도 동일한 방식이다."

### 10-2. throw / catch 패턴 — 장고 내부 데이터 전달

사용자가 입력한 데이터를 Django 서버가 받아 다른 페이지에 렌더링하는 전형적인 패턴이다.

**전체 흐름:**
```
① 사용자가 through 페이지에서 데이터 입력
② 폼 제출 → action에 지정된 /catch/ URL로 GET 요청
③ catch View가 request.GET으로 데이터 수신
④ catch.html에 데이터를 건네어 렌더링
```

**① urls.py — 두 경로 등록**

```python
# articles/urls.py
urlpatterns = [
    path('through/', views.through),
    path('catch/', views.catch),
]
```

**② views.py — through 함수**

```python
def through(request):
    return render(request, 'articles/through.html')
```

**③ through.html — 입력 폼**

```html
{% extends 'base.html' %}
{% block content %}
    <form action="/catch/" method="GET">
        <label for="message">메시지</label>
        <input type="text" name="message" id="message">
        <input type="submit" value="전송하기">
    </form>
{% endblock content %}
```

**④ views.py — catch 함수**

```python
def catch(request):
    message = request.GET.get('message')   # GET 딕셔너리에서 'message' 키로 꺼냄
    # 또는: message = request.GET['message']
    context = {'message': message}
    return render(request, 'articles/catch.html', context)
```

**⑤ catch.html — 결과 출력**

```html
{% extends 'base.html' %}
{% block content %}
    <h2>사용자가 입력한 내용: {{ message }}</h2>
{% endblock content %}
```

> 💬 "request 객체에는 유저의 모든 요청 관련 데이터(GET, POST, 쿠키, 세션, 유저 정보 등)가 담겨 있다."

**GET vs POST 수신 방법 비교:**

| 전송 방식 | 수신 코드 | 용도 |
|-----------|-----------|------|
| method="GET" | `request.GET.get('key')` | 검색, 조회 (데이터가 URL에 노출) |
| method="POST" | `request.POST.get('key')` | 로그인, 회원가입 등 (추후 학습) |

---

## 11. Variable Routing (밸리어블 라우팅)

### 11-1. 개념

URL에 **변수**를 포함시켜, 동일한 URL 패턴으로 서로 다른 데이터를 처리하는 방법이다. 예를 들어 게시글 1번, 2번, 3번마다 URL을 하드코딩으로 따로 만들지 않아도 된다.

```python
# 하드코딩 방식 (비권장)
path('articles/1/', views.detail),
path('articles/2/', views.detail),
path('articles/3/', views.detail),
# ...100개, 1000개가 되면 불가능

# Variable Routing 방식 (권장)
path('articles/<int:num>/', views.detail),
#              ↑ 꺽쇠 안에 타입:변수명
```

### 11-2. path converter 타입

URL에 들어올 수 있는 변수 타입은 다섯 가지다.

| 타입 | 설명 | 예시 |
|------|------|------|
| `str` | 슬래시를 제외한 모든 문자열 (기본값) | `articles/hello/` |
| `int` | 정수 | `articles/1/` |
| `slug` | 하이픈·언더스코어 포함 문자열 | `articles/hello-world/` |
| `uuid` | UUID 형식의 고유값 | - |
| `path` | 슬래시 포함 경로 | - |

일반적으로 게시글 번호 등에는 `int`를 가장 많이 쓴다.

### 11-3. View 함수에서 변수 받기

URL에서 넘어온 변수는 View 함수의 매개변수로 받는다. **변수명은 URL 패턴에 지정한 이름과 동일해야 한다.**

```python
# urls.py
path('articles/<int:num>/', views.detail),

# views.py
def detail(request, num):        # ← URL의 <int:num>과 이름 일치
    context = {'num': num}
    return render(request, 'articles/detail.html', context)
```

```html
<!-- articles/templates/articles/detail.html -->
<h1>{{ num }}번 게시글</h1>
```

> 💬 "variable routing이라는 건 URL에 변수를 포함시키는 거다. URL에 입력한 값이 URL에 들어가는 거다."

---

## 12. URL 분리 (include)

### 12-1. 왜 분리하는가?

프로젝트에 앱이 여러 개일 때 모든 URL을 프로젝트의 `urls.py` 한 파일에 몰아 넣으면 유지보수가 어렵다. URL이 100개, 1000개가 되면 사실상 관리가 불가능하다.

> 💬 "articles로 시작하면 articles 앱의 URL이겠거니 하고 articles에 있는 URL로 건네버리는 겁니다."

### 12-2. 구현 방법

**① 앱 내부에 urls.py 생성**

```python
# articles/urls.py (새로 생성)
from django.urls import path
from . import views

urlpatterns = [
    path('index/', views.index),
    path('dinner/', views.dinner),
    path('through/', views.through),
    path('catch/', views.catch),
    path('<int:num>/', views.detail),
]
```

**② 프로젝트 urls.py에서 include로 연결**

```python
# startpjt/urls.py
from django.urls import path, include

urlpatterns = [
    path('articles/', include('articles.urls')),
    #     ↑ 앞부분       ↑ 앱의 urls.py로 위임
]
```

### 12-3. URL 접근 주소 변화

include로 분리하면 URL 앞에 앱 경로가 붙는다. 기존에 `/index/`로 접근하던 것이 `/articles/index/`가 된다.

```
변경 전: http://127.0.0.1:8000/index/
변경 후: http://127.0.0.1:8000/articles/index/
```

`include()`는 일치하는 앞부분을 잘라내고, 나머지를 앱의 `urls.py`에 넘겨 다시 매칭한다.

---

## 13. URL name — URL에 별명 짓기

### 13-1. 개념

URL 분리 후 전체 경로(`/articles/catch/`)를 외워 하드코딩하는 것은 실수의 원인이 된다. URL이 바뀔 때마다 모든 템플릿을 수정해야 하는 문제도 있다. `path()`의 세 번째 인자 `name`으로 별명을 지어두면 별명만으로 URL을 불러올 수 있다.

### 13-2. urls.py에 name 추가

```python
# articles/urls.py
urlpatterns = [
    path('index/', views.index, name='index'),
    path('dinner/', views.dinner, name='dinner'),
    path('through/', views.through, name='through'),
    path('catch/', views.catch, name='catch'),
    path('<int:num>/', views.detail, name='detail'),
]
```

### 13-3. 템플릿에서 URL 태그 사용

하드코딩된 경로 대신 `{% url '별명' %}`을 사용한다. URL 구조가 바뀌어도 템플릿을 수정할 필요가 없다.

```html
<!-- 하드코딩 방식 (비권장) -->
<form action="/articles/catch/" method="GET">

<!-- URL 태그 방식 (권장) -->
<form action="{% url 'catch' %}" method="GET">
```

> 💬 "URL 태그를 쓰고 그 안에다 별명을 적으면 그 별명에 매칭된 절대 경로가 알아서 들어간다."

---

## 14. app_name — 앱 네임스페이스

### 14-1. 왜 필요한가?

앱이 두 개(`articles`, `pages`) 이상이면, 각 앱 `urls.py`에 동일한 별명(예: `index`)이 존재할 수 있다. 이때 `{% url 'index' %}`만으로는 어느 앱의 `index`인지 Django가 구분하지 못한다.

### 14-2. app_name 설정

```python
# articles/urls.py
app_name = 'articles'   # 앱 네임스페이스 지정

urlpatterns = [
    path('index/', views.index, name='index'),
    path('catch/', views.catch, name='catch'),
    ...
]
```

```python
# pages/urls.py
app_name = 'pages'

urlpatterns = [
    path('index/', views.index, name='index'),
    ...
]
```

### 14-3. 템플릿에서 앱 네임스페이스로 구분

`앱이름:별명` 형식으로 명확하게 지정한다.

```html
<!-- articles 앱의 catch URL로 이동 -->
<form action="{% url 'articles:catch' %}" method="GET">

<!-- pages 앱의 index URL로 이동 -->
<a href="{% url 'pages:index' %}">페이지</a>
```

**최종 URL 관리 요약:**

```
프로젝트 urls.py        앱 urls.py             템플릿
path('articles/',  →    path('index/',     →   {% url 'articles:index' %}
      include(...))            name='index')
                         app_name='articles'
```

> 💬 "앱 네임 적고, 프로젝트 URL을 앱별로 분리하고, 거기에는 name으로 별명 짓고, 별명도 중복될 수 있으니까 앱 이름으로 앱별로 구분하자."

---

## 📋 핵심 개념 정리

| 개념 | 문법 | 핵심 |
|------|------|------|
| 변수 | `{{ var }}` | context 딕셔너리의 키 |
| 필터 | `{{ var \| filter }}` | 가급적 View에서 처리 후 전달 |
| if 태그 | `{% if %} ... {% endif %}` | 닫는 태그 필수 |
| for 태그 | `{% for %} ... {% endfor %}` | `{% empty %}` 빈 리스트 처리 |
| 정적 파일 | `{% load static %}` + `{% static '...' %}` | load는 최상단 |
| 상속(부모) | `{% block content %}{% endblock %}` | 자식이 채울 영역 정의 |
| 상속(자식) | `{% extends 'base.html' %}` | 파일 첫 줄에 위치, 하나의 부모만 가능 |
| include | `{% include '파일명' %}` | 재사용 가능한 HTML 조각 |
| 폼 전송 | `<form action="URL" method="GET">` | action = 보낼 주소 |
| GET 수신 | `request.GET.get('key')` | 딕셔너리에서 꺼내기 |
| Variable Routing | `path('articles/<int:num>/', ...)` | URL 변수 포함 |
| 뷰 파라미터 | `def detail(request, num):` | URL 변수명과 동일하게 |
| URL 분리 | `include('articles.urls')` | 앱별 urls.py로 위임 |
| URL 별명 | `path(..., name='index')` | 세 번째 인자로 지정 |
| URL 태그 | `{% url 'index' %}` | 하드코딩 경로 대체 |
| 앱 네임스페이스 | `app_name = 'articles'` | 앱 urls.py 최상단 |
| 네임스페이스 사용 | `{% url 'articles:index' %}` | 앱이름:별명 형식 |
