# [Django] Static / Media — 파일 서빙 및 이미지 업로드

> 📌 핵심 키워드: #StaticFiles #MediaFiles #ImageField #Pillow #FileUpload #STATIC_URL #MEDIA_ROOT

---

## 학습 목표

* Static 파일과 Media 파일의 개념적 차이를 설명할 수 있다
* settings.py에 STATIC_URL, STATICFILES_DIRS, MEDIA_URL, MEDIA_ROOT를 올바르게 설정할 수 있다
* 템플릿에서 `{% load static %}` 및 `{% static %}` 태그를 사용할 수 있다
* 프로젝트 urls.py에 미디어 파일 제공 경로를 추가할 수 있다
* ImageField와 Pillow를 활용해 이미지 업로드·저장·렌더링 기능을 구현할 수 있다

---

## 1. Static 파일

### 1-1. Static 파일이란?

Static(정적) 파일이란 **서버 측에서 변경되지 않고 고정적으로 제공되는 파일**을 의미한다. 파일의 확장자나 종류(이미지·동영상·문서)가 중요한 게 아니라, **어디에서 어떻게 제공되느냐**가 핵심이다.

서버를 구성하기 위해 필요한 모든 리소스가 해당된다.

- CSS 파일
- JavaScript 파일
- 이미지 파일 (로고, 배경 등)
- 음악·영상 파일

### 1-2. 웹서버에서 파일을 제공하는 원리

HTML 파일 안의 `<img>` 태그는 `src` 속성에 경로를 가지고 있고, 브라우저는 **그 경로로 별도 요청을 보내** 이미지를 받아온다.

```
클라이언트 → 서버 요청(URL) → 서버가 이미지 파일 응답
```

- 로컬 파일 경로(`C:/Users/...`)는 다른 사람의 컴퓨터에서 동작하지 않는다
- 서버에 있는 파일을 제공하려면 **해당 파일을 반환하는 URL 경로**가 필요하다
- 예) `http://127.0.0.1:8000/static/articles/sample-1.png` 로 요청하면 이미지 파일이 응답된다

### 1-3. settings.py 기본 설정

Django 프로젝트를 처음 생성하면 settings.py에 아래 항목이 이미 포함되어 있다.

```python
# settings.py

STATIC_URL = 'static/'
```

- `STATIC_URL`: 정적 파일을 제공할 URL 경로 (사용자에게 보이는 URL)
- 이 값을 바꾸면 URL만 바뀌고 실제 폴더 위치는 변경되지 않는다

### 1-4. 앱(App) 내 Static 폴더 구조

Django는 기본적으로 **앱 폴더 내의 `static` 폴더**를 자동으로 탐색한다. 템플릿이 `templates` 폴더를 사용하는 것과 같은 원리다.

```
articles/
├── static/
│   └── articles/
│       └── sample-1.png   ← 앱 이름 폴더로 한 번 더 감싸는 것이 관례
├── templates/
│   └── articles/
│       └── index.html
├── models.py
├── views.py
└── urls.py
```

> ⚠️ 파일을 나중에 추가했다면 **서버를 재시작**해야 인식된다. 서버 시작 시점에 없던 파일은 찾지 못한다.

### 1-5. 추가 Static 경로 설정 (STATICFILES_DIRS)

여러 앱에서 공통으로 사용하는 파일은 앱 밖에 별도 폴더를 만들고 `STATICFILES_DIRS`에 등록한다. `TEMPLATES`의 `DIRS` 설정과 동일한 개념이다.

```python
# settings.py

STATICFILES_DIRS = [
    BASE_DIR / 'static',
]
```

```
프로젝트 루트/
├── static/             ← 공용 static 폴더 (assets, styles 등 자유롭게 네이밍 가능)
│   └── sample-e.png
├── articles/
└── manage.py
```

| 구분 | 설명 |
|------|------|
| 앱 내 `static/` | 해당 앱에서만 사용하는 파일 |
| `STATICFILES_DIRS` | 여러 앱에서 공통으로 사용하는 파일 |

### 1-6. 템플릿에서 Static 태그 사용

Static 파일을 템플릿에서 사용하려면 반드시 `{% load static %}`을 먼저 선언해야 한다.

```django
{% load static %}

<!DOCTYPE html>
<html>
<body>
  <!-- 앱 내 static 파일 -->
  <img src="{% static 'articles/sample-1.png' %}" alt="sample image">

  <!-- 공용 static 파일 (STATICFILES_DIRS 기반) -->
  <img src="{% static 'sample-e.png' %}" alt="sample-e">
</body>
</html>
```

렌더된 결과:
```html
<img src="/static/articles/sample-1.png" alt="sample image">
```

- `{% static '경로' %}`: 실제 파일 위치(path)를 사용자에게 제공할 URL로 변환해준다
- `{% load static %}`은 **각 템플릿 파일마다** 직접 선언해야 한다

### 1-7. load static은 베이스 템플릿에서 상속되지 않는다

> ⚠️ `base.html`에 `{% load static %}`을 작성해도, 이를 `extends`하는 자식 템플릿에서 `{% static %}` 태그를 사용할 수 없다.

**이유:** `{% extends 'base.html' %}`는 HTML 문서 구조를 가져와 `{% block content %}` 영역을 채우는 방식이다. `{% load static %}`은 DTL 모듈 로드이므로, 자식 템플릿이 렌더링될 때 별도로 불러와야 한다.

또한 `base.html`에 `load static`을 추가하면 static을 사용하지 않는 다른 템플릿에서도 불필요하게 모듈이 로드되어 비효율적이다.

---

## 2. Media 파일

### 2-1. Media 파일이란?

Media 파일은 **사용자가 웹에서 업로드한 파일**을 의미한다. 서버가 동작 중에도 새로운 파일이 추가될 수 있다는 점에서 static 파일과 다르다.

- 게시글 첨부 이미지, 프로필 사진, 문서 파일 등
- 일단 업로드되면 해당 파일 자체는 변경되지 않는 정적인 파일이다

| 구분 | Static 파일 | Media 파일 |
|------|------------|-----------|
| 주체 | 개발자(서버 제공) | 사용자(업로드) |
| 추가 시점 | 서버 시작 전 | 서버 운영 중 |
| 설정 위치 | `STATICFILES_DIRS` | `MEDIA_ROOT` |
| URL 설정 | `STATIC_URL` (자동) | `MEDIA_URL` (수동 등록 필요) |

### 2-2. settings.py Media 설정

```python
# settings.py

MEDIA_URL = 'media/'         # 업로드된 파일을 제공할 URL
MEDIA_ROOT = BASE_DIR / 'media'   # 업로드된 파일이 저장될 실제 경로(path)
```

- `MEDIA_URL`: 사용자에게 제공할 URL (`STATIC_URL`과 같은 역할)
- `MEDIA_ROOT`: 파일이 실제 저장될 서버 디렉토리 경로 (`STATICFILES_DIRS`와 유사한 역할)

### 2-3. 프로젝트 urls.py에 미디어 URL 추가

Static과 달리 Media URL은 자동으로 등록되지 않는다. 프로젝트의 `urls.py`에 직접 추가해야 한다.

```python
# 프로젝트/urls.py

from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('articles/', include('articles.urls')),
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

**코드 해설:**

- `from django.conf import settings`: `settings.py`에 정의한 변수를 **안전하게** 불러오는 방법
- `from django.conf.urls.static import static`: Django 내장 `static()` 함수 import
- `static(prefix, document_root)`: URL 패턴 리스트를 반환하는 함수
  - 첫 번째 인자(`prefix`): `settings.MEDIA_URL`
  - `document_root` 키워드 인자: `settings.MEDIA_ROOT`
- 반환값이 리스트이므로 `urlpatterns`에 `+` 연산으로 합친다

**`from django.conf import settings`를 사용하는 이유**

Django가 설치되면 내부에 `django.conf.global_settings`라는 기본 설정 파일이 존재한다. 우리가 작성하는 `settings.py`는 이 기본값을 **덮어쓰기(override)** 하는 구조다. 따라서 `settings.py`를 직접 import하면 기본값이 아닌 우리가 커스터마이징한 최신 설정이 보장되지 않을 수 있다.

`from django.conf import settings`를 사용하면 서버가 실제로 실행될 때 적용된 최종 설정을 **레이지(lazy)하게** 가져온다. 즉, 필요한 시점에 덮어쓰기까지 완료된 설정값을 정확히 참조할 수 있다.

```python
# 이렇게 직접 import 하지 않고
from myproject.settings import MEDIA_ROOT  # ❌ 권장하지 않음

# 이렇게 django.conf를 통해 불러온다
from django.conf import settings
settings.MEDIA_ROOT  # ✅ 항상 최종 적용된 값을 참조
```

---

## 3. ImageField와 파일 업로드

### 3-1. models.py — ImageField 추가

```python
# articles/models.py

from django.db import models

class Article(models.Model):
    title = models.CharField(max_length=100)
    content = models.TextField()
    image = models.ImageField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

- `ImageField`: 이미지 업로드에 사용되는 모델 필드
- `blank=True`: 이미지를 업로드하지 않아도 게시글 작성 가능 (필수 입력 아님)
- 이미지 파일 외 다른 파일 타입은 `FileField` 사용 가능

> ⚠️ **DB에 저장되는 것은 파일 자체가 아닌 파일의 경로(문자열)**이다. 실제 파일은 `MEDIA_ROOT` 경로에 저장된다.

**DB에 파일 자체를 저장하지 않는 이유**

파일을 DB에 직접 저장하는 것이 불가능하지는 않지만, 웹서버 환경에서는 권장하지 않는다.

- **손실 위험**: DB 장애 시 파일도 함께 손실될 수 있다
- **관리 어려움**: DB와 파일 시스템을 분리하면 각각 독립적으로 관리·백업할 수 있다
- **성능 저하**: 이미지를 텍스트(base64 등)로 변환하면 원본보다 용량이 크게 늘어나고, 조회 시 쿼리 속도가 저하된다
- **비효율**: 파일 서빙은 이미 static 파일 방식으로 URL 경로를 통해 효율적으로 처리할 수 있다

그래서 DB의 이미지 컬럼에는 **파일이 저장된 경로(문자열)만** 기록하고, 실제 파일은 서버 디렉토리에 저장한다. 대규모 서비스에서는 파일 전용 스토리지 서버(예: AWS S3)를 별도로 두는 방식을 사용한다.

### 3-2. Pillow 설치

`ImageField`를 사용하려면 Python 이미지 처리 라이브러리인 **Pillow**가 필요하다. Django가 makemigrations 시 직접 안내해준다.

```bash
pip install Pillow
pip freeze > requirements.txt
```

```bash
python manage.py makemigrations
python manage.py migrate
```

### 3-3. forms.py — ArticleForm 작성

```python
# articles/forms.py

from django import forms
from .models import Article

class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = '__all__'
```

- `ImageField`가 포함된 모델을 사용해도 `ArticleForm`은 동일하게 작성한다
- Django가 `ImageField`를 `<input type="file">`로 자동 렌더링해준다

### 3-4. views.py — Create 기능 구현

```python
# articles/views.py

from django.shortcuts import render, redirect
from .models import Article
from .forms import ArticleForm

def create(request):
    if request.method == 'POST':
        form = ArticleForm(request.POST, request.FILES)  # ← request.FILES 추가
        if form.is_valid():
            article = form.save()
            return redirect('articles:detail', article.pk)
    else:
        form = ArticleForm()
    context = {'form': form}
    return render(request, 'articles/create.html', context)
```

핵심 변경점:

- `ArticleForm(request.POST, request.FILES)` — 두 번째 인자로 `request.FILES` 전달
- `request.POST`: 텍스트 데이터 (제목, 내용 등)
- `request.FILES`: 파일 데이터 (이미지 등)

### 3-5. 단계별 디버깅 순서

코드를 작성할 때 GET 렌더링과 POST 처리를 한꺼번에 작성하지 말고, **반드시 단계를 나눠 확인**해야 한다. 검증 없이 코드를 계속 쌓으면 어느 지점에서 오류가 발생했는지 찾을 수 없다.

```
1단계: views.py에 GET 처리(렌더)만 먼저 작성
        ↓
2단계: 서버 실행 → create 페이지 접속 → 폼이 정상 렌더링되는지 확인
        ↓
3단계: create.html에 enctype 추가, 파일 선택 input이 보이는지 확인
        ↓
4단계: 이상 없으면 views.py에 POST 처리 코드 추가
        ↓
5단계: 실제 이미지 첨부 후 게시글 작성 → DB 및 media 폴더 확인
```

### 3-6. create.html — enctype 속성 추가

파일을 전송하려면 폼 태그의 인코딩 타입을 변경해야 한다. 기본값은 문자열 전송이라 파일을 보낼 수 없다.

```django
{% load static %}
{% extends 'base.html' %}

{% block content %}
  <h1>게시글 작성</h1>
  <form action="{% url 'articles:create' %}" method="POST" enctype="multipart/form-data">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">작성</button>
  </form>
{% endblock content %}
```

- `enctype="multipart/form-data"`: 파일(이진 데이터) 전송을 위한 인코딩 타입
- `<input type="file">`이 포함된 경우 반드시 추가해야 한다

### 3-7. detail.html — 업로드 이미지 표시

```django
{% block content %}
  <h2>{{ article.title }}</h2>
  <p>{{ article.content }}</p>

  {% if article.image %}
    <img src="{{ article.image.url }}" alt="{{ article.image }}">
  {% endif %}
{% endblock content %}
```

- `article.image.url`: settings에 설정된 `MEDIA_URL`을 기반으로 만들어진 URL
- `article.image.name`(또는 `{{ article.image }}`): 파일 경로 문자열 출력
- `{% if article.image %}`: `blank=True`로 설정했으므로 이미지가 없는 경우 조건 분기 필요
- 이미지가 없을 때 default 이미지를 보여주는 방법도 고려할 수 있다

---

## 4. 참고사항

### 4-1. ImageField의 upload_to 옵션

`upload_to` 속성을 지정하면 `MEDIA_ROOT` 아래 하위 폴더에 파일을 분류 저장할 수 있다.

```python
# 문자열로 고정 폴더 지정
image = models.ImageField(blank=True, upload_to='images/')
# → MEDIA_ROOT/images/ 에 저장

# 날짜별 폴더 자동 생성
image = models.ImageField(blank=True, upload_to='images/%Y/%m/%d/')
# → MEDIA_ROOT/images/2026/04/16/ 에 저장

# 함수로 동적 경로 지정 (예: 유저별 폴더)
def user_directory_path(instance, filename):
    return f'images/{instance.user.username}/{filename}'

image = models.ImageField(blank=True, upload_to=user_directory_path)
```

### 4-2. 이미지 수정 구현 시 주의사항

업데이트(update) 기능에서도 이미지를 다루려면 `request.FILES`를 두 번째 인자로 전달해야 한다. create와 큰 차이는 없으나 인스턴스를 `instance` 키워드 인자로 전달하는 점이 다르다.

```python
# views.py - update 예시

def update(request, pk):
    article = Article.objects.get(pk=pk)
    if request.method == 'POST':
        form = ArticleForm(request.POST, request.FILES, instance=article)
        ...
```

---

## 📋 핵심 개념 정리

| 개념 | 설명 | 예시/명령어 |
|------|------|-------------|
| Static 파일 | 서버 구성에 필요한 고정 파일 | CSS, JS, 로고 이미지 등 |
| Media 파일 | 사용자가 업로드한 파일 | 게시글 첨부 이미지 등 |
| `STATIC_URL` | static 파일 제공 URL prefix | `STATIC_URL = 'static/'` |
| `STATICFILES_DIRS` | 추가 static 탐색 경로 리스트 | `[BASE_DIR / 'static']` |
| `MEDIA_URL` | media 파일 제공 URL prefix | `MEDIA_URL = 'media/'` |
| `MEDIA_ROOT` | media 파일 저장 경로 | `BASE_DIR / 'media'` |
| `{% load static %}` | DTL static 모듈 import | 각 템플릿마다 직접 선언 |
| `{% static '경로' %}` | 파일 path → URL 변환 태그 | `{% static 'articles/img.png' %}` |
| `ImageField` | 이미지 저장 모델 필드 | `models.ImageField(blank=True)` |
| `Pillow` | Python 이미지 처리 라이브러리 | `pip install Pillow` |
| `request.FILES` | 요청에 포함된 파일 데이터 | `ArticleForm(request.POST, request.FILES)` |
| `enctype` | 파일 전송을 위한 폼 인코딩 타입 | `enctype="multipart/form-data"` |
| `article.image.url` | 이미지 접근 URL 속성 | `<img src="{{ article.image.url }}">` |
| `upload_to` | 파일 저장 하위 경로 설정 | `upload_to='images/%Y/%m/%d/'` |
