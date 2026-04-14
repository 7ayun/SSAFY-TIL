# [Django] ORM with View — CRUD 기능 구현

> 📌 핵심 키워드: #ORM #CRUD #MTV #GET #POST #CSRF #redirect #QuerysetAPI

---

## 학습 목표

* ORM을 Shell이 아닌 View 함수에서 직접 사용하는 방법을 이해한다
* GET과 POST 요청의 차이와 각각의 적절한 사용 시점을 구분한다
* CSRF 토큰의 개념과 사용 방법을 이해한다
* redirect 함수의 역할과 사용 방법을 익힌다
* URL → View → Template 패턴으로 게시글 조회·생성·삭제 기능을 구현한다

---

## 1. 수업 개요

지난주 목요일에 ORM을 Shell 창에서만 다뤘었는데, 오늘은 그 ORM을 Django의 MTV 디자인 패턴 중 View에서 직접 사용한다. 데이터베이스를 다룰 때는 CRUD, 즉 생성(Create)·조회(Read)·수정(Update)·삭제(Delete)라는 4가지 핵심 요소가 있다. 오늘은 그 중 생성과 조회를 웹 브라우저에서 실제로 동작하는 서비스로 만들어보고, 삭제까지 다룬다. 수정은 실습 과제로 남긴다.

프로젝트명은 `crud`, 앱명은 `articles`로 진행한다.

---

## 2. 프로젝트 초기 설정

### 2-1. 가상환경 생성 및 활성화

```bash
python -m venv venv
source venv/Scripts/activate   # Windows
source venv/bin/activate        # Mac / Linux
```

### 2-2. Django 설치 및 패키지 관리

```bash
pip install django
pip freeze > requirements.txt
```

오늘은 Shell Plus를 사용하지 않으므로 `ipython`과 `django-extensions` 설치는 불필요하다. Shell 창 복습이 필요하다면 추가로 설치하면 된다.

### 2-3. 프로젝트 및 앱 생성

```bash
django-admin startproject crud .
python manage.py startapp articles
```

`startproject` 명령 뒤에 `.`을 붙이면 현재 폴더에 프로젝트가 생성된다. `.`을 생략하면 프로젝트명과 동일한 폴더가 하나 더 생긴다는 점을 기억해야 한다.

### 2-4. 앱 등록

`settings.py`의 `INSTALLED_APPS`에 `articles` 앱을 등록한다.

```python
# crud/settings.py
INSTALLED_APPS = [
    ...
    'articles',
]
```

---

## 3. 모델 정의 및 마이그레이션

### 3-1. Article 모델 정의

```python
# articles/models.py
from django.db import models

class Article(models.Model):
    title = models.CharField(max_length=100)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

| 필드 | 타입 | 설명 |
|------|------|------|
| `title` | `CharField(max_length=100)` | 최대 100자 제한 텍스트 |
| `content` | `TextField()` | 글자 수 제한 없는 텍스트 |
| `created_at` | `DateTimeField(auto_now_add=True)` | 생성 시 자동 저장 |
| `updated_at` | `DateTimeField(auto_now=True)` | 수정 시마다 자동 갱신 |

`id`(Primary Key)는 별도로 정의하지 않아도 Django가 자동으로 생성해준다.

### 3-2. 마이그레이션

```bash
python manage.py makemigrations
python manage.py migrate
```

`makemigrations`는 설계도(마이그레이션 파일)를 생성하고, `migrate`는 그 설계도를 실제 데이터베이스에 반영한다.

---

## 4. URL 설정

### 4-1. 프로젝트 urls.py

```python
# crud/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('articles/', include('articles.urls')),
]
```

`include('articles.urls')`에서 `'articles.urls'`는 문자열 형태로 경로를 넘긴다. 이는 즉시 모듈을 불러오는 것이 아니라, 사용자가 실제로 `articles/` 경로로 요청을 보낼 때 해당 모듈을 로드하는 지연 로딩 방식이다.

### 4-2. 앱 urls.py 생성

`articles` 폴더 안에 `urls.py`를 새로 생성한다.

```python
# articles/urls.py
from django.urls import path
from . import views

app_name = 'articles'

urlpatterns = [
    path('', views.index, name='index'),
]
```

`app_name`은 URL 네임스페이스를 설정하는 변수로, 템플릿의 `{% url %}` 태그에서 `articles:index`와 같이 사용할 수 있게 해준다. `app_name`과 실제 앱 폴더명이 같을 필요는 없지만, 관례상 동일하게 사용한다.

---

## 5. 게시글 전체 조회 (Read — index)

### 5-1. templates 폴더 구조

`settings.py`의 `TEMPLATES` 설정에서 `'APP_DIRS': True`로 되어 있으면, Django는 각 앱 폴더 안의 `templates` 디렉토리를 자동으로 탐색한다.

```
articles/
└── templates/
    └── articles/
        └── index.html
```

모든 앱이 공통으로 사용하는 베이스 템플릿은 `settings.py`의 `TEMPLATES > DIRS`에 경로를 추가해 관리한다.

```python
# crud/settings.py
TEMPLATES = [
    {
        ...
        'DIRS': [BASE_DIR / 'templates'],
        ...
    }
]
```

프로젝트 루트에 `templates/` 폴더를 생성하고 그 안에 `base.html`을 만든다.

### 5-2. base.html (베이스 템플릿)

```html
<!-- templates/base.html -->
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>Django CRUD</title>
</head>
<body>
  {% block content %}
  {% endblock content %}
</body>
</html>
```

`block content`는 각 자식 템플릿이 고유한 내용을 채워 넣을 영역이다.

### 5-3. View 함수 — index

```python
# articles/views.py
from django.shortcuts import render
from .models import Article

def index(request):
    articles = Article.objects.all()
    context = {
        'articles': articles,
    }
    return render(request, 'articles/index.html', context)
```

`Article.objects.all()`은 QuerySet API를 통해 데이터베이스의 모든 게시글을 가져온다. `objects`는 Django가 자동으로 제공하는 매니저(Manager)다. `context`는 딕셔너리 형태로 템플릿에 전달하는 데이터다.

### 5-4. index.html

```django
<!-- articles/templates/articles/index.html -->
{% extends 'base.html' %}

{% block content %}
  <h1>Article Index Page</h1>
  <a href="{% url 'articles:new' %}">새 게시글 작성</a>
  <hr>
  {% for article in articles %}
    <p>{{ article.pk }}번 : {{ article.title }}</p>
    <hr>
  {% endfor %}
{% endblock content %}
```

---

## 6. 게시글 생성 (Create — new / create)

게시글 생성은 두 가지 단계로 나뉜다.

| 단계 | URL | View 함수 | HTTP Method | 역할 |
|------|-----|-----------|-------------|------|
| 1단계 | `/articles/new/` | `new` | GET | 입력 폼 렌더링 |
| 2단계 | `/articles/create/` | `create` | POST | 데이터 저장 후 redirect |

### 6-1. URL 추가

```python
# articles/urls.py
urlpatterns = [
    path('', views.index, name='index'),
    path('new/', views.new, name='new'),
    path('create/', views.create, name='create'),
]
```

### 6-2. View 함수 — new

```python
def new(request):
    return render(request, 'articles/new.html')
```

### 6-3. new.html (게시글 작성 폼)

```django
<!-- articles/templates/articles/new.html -->
{% extends 'base.html' %}

{% block content %}
  <h1>New Page</h1>
  <form action="{% url 'articles:create' %}" method="POST">
    {% csrf_token %}
    <label for="title">제목을 작성하세요.</label>
    <input type="text" id="title" name="title">
    <br>
    <label for="content">내용을 작성하세요.</label>
    <textarea id="content" name="content"></textarea>
    <br>
    <input type="submit" value="작성">
  </form>
{% endblock content %}
```

`input` 태그의 `name` 속성이 딕셔너리의 키값이 되어 View 함수에서 데이터를 꺼낼 때 사용된다.

제출 버튼은 두 가지 방식으로 작성할 수 있으며 동일한 역할을 한다.

```html
<!-- 방법 1 -->
<input type="submit" value="작성">

<!-- 방법 2 -->
<button type="submit">작성</button>
```

### 6-4. DTL `{% url %}` 태그와 하드코딩 문제

`href`나 `action`에 경로를 직접 문자열로 적는 것을 **하드코딩**이라고 한다.

```html
<!-- 하드코딩 (지양) -->
<a href="http://127.0.0.1:8000/articles/new">새 게시글 작성</a>
```

이 방식은 URL이 바뀔 때마다 모든 파일을 직접 수정해야 하는 유지보수 문제가 생긴다. 대신 DTL의 `{% url %}` 태그를 사용하면 `urls.py`에 등록된 `app_name`과 `name`을 조합해 경로를 동적으로 생성한다.

```django
<!-- DTL url 태그 사용 (권장) -->
<a href="{% url 'articles:new' %}">새 게시글 작성</a>
```

`articles`는 `urls.py`의 `app_name` 변수에 할당한 값이고, `new`는 `path()`의 `name` 인자에 넣은 값이다. 렌더링 시 이 태그는 실제 경로 문자열(`/articles/new/`)로 변환된다. DTL은 HTML 표준 문법이 아니지만, `render()` 함수가 이를 해석해 완성된 HTML로 만들어주기 때문에 브라우저에서 정상 동작한다.

---

## 7. GET과 POST

### 7-1. GET

* URL 쿼리스트링 형태로 데이터를 전송한다 (예: `?title=hello&content=world`)
* 데이터가 URL에 노출되어 브라우저 히스토리에 남는다
* 전송 가능한 데이터 양이 제한적이다
* **사용 목적**: 데이터 조회, 페이지 요청, API 데이터 조회

### 7-2. POST

* HTTP Body에 데이터를 담아 전송한다
* URL에 데이터가 노출되지 않아 브라우저 히스토리에 남지 않는다
* 더 많은 양의 데이터를 전송할 수 있다
* 캐싱을 하지 않는다
* **사용 목적**: 생성·수정·삭제처럼 리소스를 변경하는 모든 요청

폼 태그에서 POST 방식으로 전송하려면 `method="POST"`를 명시한다. 기본값은 GET이다.

---

## 8. CSRF 토큰

### 8-1. CSRF란

CSRF(Cross-Site Request Forgery)는 사이트 간 요청 위조를 의미한다. Django가 렌더링한 우리 사이트가 아닌, 정체불명의 외부 사이트에서 보낸 것처럼 위조된 요청을 의미하는 공격 방식이다.

POST 요청으로 리소스를 변경할 수 있기 때문에, Django는 신뢰할 수 있는 사이트에서 온 요청인지 검증하기 위한 인장(토큰)을 요구한다.

### 8-2. CSRF 토큰 적용

```django
<form action="{% url 'articles:create' %}" method="POST">
  {% csrf_token %}
  ...
</form>
```

`{% csrf_token %}`을 폼 태그 안에 넣으면, Django는 렌더링 시 히든(hidden) `input` 태그를 자동으로 생성한다.

```html
<!-- Django가 자동 생성하는 코드 (예시) -->
<input type="hidden" name="csrfmiddlewaretoken" value="16진수난수...">
```

이 16진수 난수 값은 외부에서 동일하게 생성하기 불가능에 가깝기 때문에, 위조 요청을 효과적으로 차단할 수 있다. CSRF 토큰 없이 POST 요청을 보내면 **403 Forbidden** 상태 코드가 반환된다.

---

## 9. View 함수 — create

```python
# articles/views.py
def create(request):
    title = request.POST.get('title')
    content = request.POST.get('content')

    article = Article()
    article.title = title
    article.content = content
    article.save()

    return redirect('articles:index')
```

POST 방식으로 전송된 데이터는 `request.POST`(딕셔너리 형태)에 담겨있다. `.get('키명')`으로 값을 꺼낸다.

게시글 생성 후에는 `render`가 아닌 `redirect`를 사용한다.

---

## 10. redirect

### 10-1. redirect가 필요한 이유

게시글 생성(`create`) 뷰 함수의 역할은 데이터를 저장하는 것이다. 하나의 함수는 한 가지 일만 해야 하는 원칙에 따라, 저장 후에 `create.html`을 렌더링하는 것은 이상하다. 사용자는 생성이 완료된 후 목록 페이지나 상세 페이지로 이동해야 한다.

### 10-2. redirect 사용법

```python
from django.shortcuts import render, redirect

def create(request):
    ...
    return redirect('articles:index')
```

`redirect`는 `django.shortcuts` 패키지에 있으며, 클라이언트에게 인자로 전달된 URL로 **GET 요청**을 다시 보내도록 지시한다. 이때 HTTP 상태 코드 **302**가 반환된다.

| 상태 코드 | 의미 |
|-----------|------|
| 200 | 요청 성공 (OK) |
| 302 | 다른 URL로 임시 리다이렉트 |
| 403 | 권한 없음 (CSRF 토큰 누락 등) |
| 404 | 페이지 없음 |

---

## 11. 게시글 삭제 (Delete)

### 11-1. URL 추가

삭제할 게시글의 pk 값을 경로에 포함시킨다.

```python
# articles/urls.py
urlpatterns = [
    path('', views.index, name='index'),
    path('new/', views.new, name='new'),
    path('create/', views.create, name='create'),
    path('<int:article_pk>/delete/', views.delete, name='delete'),
]
```

### 11-2. View 함수 — delete

```python
def delete(request, article_pk):
    article = Article.objects.get(pk=article_pk)
    article.delete()
    return redirect('articles:index')
```

경로에 담아 온 `article_pk` 값으로 게시글 하나를 조회(`get`)한 뒤 삭제(`.delete()`)하고, 인덱스 페이지로 리다이렉트한다. 삭제는 데이터를 변경하는 행위이므로 반드시 POST 방식으로 처리해야 한다.

### 11-3. index.html 삭제 버튼 추가

```django
{% for article in articles %}
  <p>{{ article.pk }}번 : {{ article.title }}</p>
  <form action="{% url 'articles:delete' article.pk %}" method="POST">
    {% csrf_token %}
    <input type="submit" value="삭제">
  </form>
  <hr>
{% endfor %}
```

삭제 요청은 POST 방식이므로 `{% csrf_token %}`이 반드시 필요하다.

---

## 12. 게시글 수정 (Update) — 실습 과제

수정 기능은 create와 구조가 동일하다. `edit` 뷰(수정 폼 렌더링)와 `update` 뷰(데이터 저장)로 구성한다.

create와의 차이점은, 기존 데이터를 조회해서 그 인스턴스의 `title`과 `content`를 새 값으로 덮어쓴 뒤 저장한다는 것이다.

```python
# 예시 로직
article = Article.objects.get(pk=article_pk)
article.title = request.POST.get('title')
article.content = request.POST.get('content')
article.save()
```

---

## 참고사항

* 수업 후 GET과 POST의 정확한 차이가 무엇인지 한 번 더 정리해보기
* 오늘 사용한 완성 코드는 강의 레포지토리의 `05_Django_ORM_with_view/answer` 폴더에서 확인 가능

---

## 📋 핵심 개념 정리

| 개념 | 설명 | 예시/명령어 |
|------|------|-------------|
| `makemigrations` | 모델 변경사항 설계도(마이그레이션 파일) 생성 | `python manage.py makemigrations` |
| `migrate` | 설계도를 실제 DB에 반영 | `python manage.py migrate` |
| `objects.all()` | 전체 데이터 조회 QuerySet 반환 | `Article.objects.all()` |
| `objects.get()` | 조건에 맞는 단일 객체 조회 | `Article.objects.get(pk=1)` |
| `instance.save()` | 인스턴스를 DB에 저장 | `article.save()` |
| `instance.delete()` | 인스턴스를 DB에서 삭제 | `article.delete()` |
| `request.POST` | POST 방식으로 전송된 데이터 딕셔너리 | `request.POST.get('title')` |
| `{% csrf_token %}` | CSRF 토큰 히든 인풋 태그 자동 생성 | 폼 태그 안에 삽입 |
| `redirect` | 클라이언트를 특정 URL로 GET 재요청 | `redirect('articles:index')` |
| `app_name` | URL 네임스페이스 설정 변수 | `app_name = 'articles'` |
| `{% url %}` | DTL URL 태그로 경로 동적 생성 | `{% url 'articles:delete' article.pk %}` |
| `302` | 리다이렉트 상태 코드 | `redirect()` 호출 시 반환 |
| `403` | CSRF 토큰 누락 등 권한 없음 상태 코드 | — |
