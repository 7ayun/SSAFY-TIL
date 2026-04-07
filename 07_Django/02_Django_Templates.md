# [Django] Templates — DTL · 정적 파일 · 템플릿 상속

> **"어려운 게 아니고 낯선 거다. 많이 하다 보면 충분히 할 수 있다."** — 강사님

#Django #Templates #DTL #StaticFiles #TemplateInheritance #SSAFY

---

## 학습 목표

* Django 템플릿 시스템(DTL)의 4가지 구성 요소를 설명할 수 있다
* 변수·필터·태그를 템플릿에서 올바르게 사용할 수 있다
* View에서 context(딕셔너리)를 템플릿으로 전달하는 흐름을 이해한다
* 정적 파일(static)을 앱에 연결하고 CSS·이미지를 로드할 수 있다
* 템플릿 상속(`extends` / `block` / `include`)으로 공통 레이아웃을 구성할 수 있다

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

> **"venv는 200~300MB라 GitHub에 올리지 않는다. requirements.txt로 패키지 목록만 공유하는 것이 정석이다."** — 강사님

### 1-2. 요청·응답 3단계 흐름

```
① URL 등록   urls.py      path('index/', views.index)
② View 작성  views.py     def index(request): return render(request, '...')
③ Template   .html 파일   실제 화면 출력
```

> **"URL은 유저의 진입점(Entry Point). 주소 → 함수 → 템플릿 이 순서를 항상 기억하라."** — 강사님

---

## 2. Django 템플릿 시스템 (DTL)

**DTL(Django Template Language)** — Django가 HTML에 파이썬 로직을 녹여낼 수 있도록 제공하는 전용 문법.

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

> **"context라는 이름은 관례다. 중요한 건 딕셔너리 형태로 만드는 것."** — 강사님

### 3-2. 딕셔너리·객체 속성 접근

DTL에서는 `.`(점)으로 딕셔너리 키 / 리스트 인덱스 / 객체 속성에 모두 접근한다.

```html
{{ my_dict.key }}       <!-- 딕셔너리 값 -->
{{ my_list.0 }}         <!-- 리스트 첫 번째 요소 -->
{{ my_obj.name }}       <!-- 객체 속성 -->
```

---

## 4. 필터 (Filter)

변수 뒤에 `|`(파이프)를 붙여 적용하며, 공식 문서에서 Built-in Filter Reference를 참고한다.

```html
{{ name | lower }}              <!-- 소문자 변환 -->
{{ name | upper }}              <!-- 대문자 변환 -->
{{ content | truncatewords:2 }} <!-- 단어 2개까지만 출력 후 ... -->
{{ number | add:10 }}           <!-- 숫자 더하기 -->
{{ today | date:"Y년 m월 d일" }} <!-- 날짜 포맷 -->
```

> **"필터는 가급적 쓰지 않는 게 좋다. 숫자 더하기나 글자 자르기 같은 로직 처리는 백엔드(View) 레벨에서 이미 처리해서 넘겨주는 게 정석이다. 계산을 사용자 브라우저에게 넘기지 말고, 서버가 처리해서 던지는 것이 기본 원칙."** — 강사님

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

---

## 6. GET 요청과 URL 쿼리 스트링

### 6-1. HTML form 작성

```html
<!-- articles/templates/articles/search.html -->
<form action="https://search.naver.com/search.naver" method="GET">
    <label for="message">검색어</label>
    <input type="text" name="query" id="message">
    <input type="submit" value="검색하기">
</form>
```

```
검색어 '안녕하세요' 제출 시 URL:
https://search.naver.com/search.naver?query=안녕하세요
                                       ↑ name 값이 키가 됨
```

> **"name 속성이 바로 URL 쿼리 스트링의 키가 된다. 네이버도 동일한 방식이다."** — 강사님

### 6-2. 장고 서버로 GET 요청 받기

```python
# views.py
def search(request):
    query = request.GET.get('query')  # URL ?query=... 에서 값 추출
    context = {'query': query}
    return render(request, 'articles/search.html', context)
```

---

## 7. 정적 파일 (Static Files)

**Static file** — CSS, JS, 이미지처럼 서버 로직 없이 그대로 제공되는 파일.

### 7-1. 앱 내 정적 파일 경로 구조

```
articles/
  static/
    articles/        ← 앱 이름 폴더 (네임스페이스 역할)
      style.css
      images/
        logo.png
```

### 7-2. 템플릿에서 정적 파일 로드

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

> **"`{% load static %}`을 선언하지 않으면 `{% static %}` 태그를 인식 못 해서 에러가 난다. 꼭 최상단에 써야 한다."** — 강사님

### 7-3. settings.py 정적 파일 설정

```python
# settings.py (기본값 확인)
STATIC_URL = 'static/'
# STATICFILES_DIRS = [BASE_DIR / 'static']  # 프로젝트 공통 static 폴더 사용 시
```

---

## 8. 템플릿 상속 (Template Inheritance)

반복되는 HTML 구조(navbar, footer 등)를 **base 템플릿** 하나로 관리하고, 자식 템플릿에서 재사용하는 방법.

### 8-1. 상속 구조

```
templates/
  base.html          ← 부모 (공통 레이아웃)
articles/
  templates/
    articles/
      index.html     ← 자식 (실제 내용)
```

### 8-2. base.html (부모 템플릿)

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

### 8-3. 자식 템플릿 (index.html)

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

> **"`extends`는 반드시 파일의 맨 첫 번째 줄에 와야 한다. 그렇지 않으면 에러가 발생한다."** — 강사님

### 8-4. settings.py — 프로젝트 공통 templates 경로 등록

```python
TEMPLATES = [
    {
        'DIRS': [BASE_DIR / 'templates'],  # 프로젝트 루트 templates 폴더 등록
        ...
    }
]
```

---

## 9. include 태그

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

> **"include는 단순한 HTML 조각을 가져오는 것. extends는 레이아웃 구조를 물려받는 것. 두 가지를 혼동하지 말 것."** — 강사님

---

## 10. templates 폴더 위치 & 네임스페이스

### 10-1. 왜 앱 이름 폴더를 한 번 더 만드는가?

```
articles/templates/articles/index.html   ← 권장
articles/templates/index.html            ← 비권장
```

두 앱에 동일한 파일명이 있을 때 Django는 `INSTALLED_APPS` 순서대로 검색한다.
앱 이름 폴더를 만들지 않으면 **다른 앱의 index.html**이 실수로 로드될 수 있다.

```python
# views.py
return render(request, 'articles/index.html', context)
#                       ↑ 앱 이름/파일명 형태로 명시
```

---

## 정리

| 개념 | 문법 | 핵심 |
|------|------|------|
| 변수 | `{{ var }}` | context 딕셔너리의 키 |
| 필터 | `{{ var \| filter }}` | 가급적 View에서 처리 후 전달 |
| if 태그 | `{% if %} ... {% endif %}` | 닫는 태그 필수 |
| for 태그 | `{% for %} ... {% endfor %}` | `{% empty %}` 빈 리스트 처리 |
| 정적 파일 | `{% load static %}` + `{% static '...' %}` | load는 최상단 |
| 상속(부모) | `{% block content %}{% endblock %}` | 자식이 채울 영역 정의 |
| 상속(자식) | `{% extends 'base.html' %}` | 파일 첫 줄에 위치 |
| include | `{% include '파일명' %}` | 재사용 가능한 HTML 조각 |
