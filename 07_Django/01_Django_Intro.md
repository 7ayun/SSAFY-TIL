# [Django] Django Intro — 웹 애플리케이션, 가상환경, MTV 패턴, URL·View·Template

> **핵심 키워드:** #Django #웹프레임워크 #클라이언트서버 #프론트엔드 #백엔드 #가상환경 #venv #MTV패턴 #MVC #URL라우팅 #View함수 #Template #render #HttpResponse #startproject #startapp #INSTALLED_APPS #settings

---

## 학습 목표

* 웹 애플리케이션이 클라이언트-서버 구조에서 어떻게 동작하는지 설명할 수 있다
* 프레임워크가 필요한 이유를 이해하고, Django를 선택하는 배경을 설명할 수 있다
* 가상 환경(venv)을 생성·활성화·비활성화하고 패키지 충돌 문제를 예방할 수 있다
* Django 프로젝트와 앱을 생성하고 개발 서버를 실행할 수 있다
* MTV 디자인 패턴의 요청 흐름을 이해하고 URL·View·Template을 직접 연결할 수 있다

---

## 1. 웹 애플리케이션

### 1-1. 웹 애플리케이션이란

인터넷을 통해 사용자에게 제공되는 소프트웨어 프로그램을 구축하는 과정을 웹 애플리케이션 개발이라 한다. 모바일·태블릿·PC 등 다양한 디바이스에서 크롬, 엣지, 오페라 같은 **브라우저**를 통해 접근할 수 있는 모든 서비스가 여기에 해당한다.

> **강사님 강조**: "어플리케이션이라고 해서 꼭 모바일에 들어가는 것만을 이야기하는 게 아니야. 유튜브처럼 브라우저로 접근하는 모든 것이 웹 애플리케이션이다."

### 1-2. 클라이언트-서버 구조

웹의 동작 원리는 **요청(Request)** 과 **응답(Response)** 으로 요약된다.

```
[클라이언트]                [서버]
 브라우저  --- URL 요청 --> 유튜브 서버
           <-- HTML 응답 --
```

1. 사용자가 브라우저 주소창에 URL을 입력한다 → 서버에 요청 발송
2. 서버는 해당 요청을 처리한다
3. HTML 문서를 응답으로 돌려준다 → 브라우저가 화면에 렌더링

> **강사님 강조**: "클라이언트가 요청을 보내면 서버는 응답한다. 이 구조만 먼저 완벽하게 이해하면 된다. 그 안에 보안, 도메인, 파라미터 같은 세부 사항들은 나중에 쌓아나가면 돼."

클라이언트가 보내는 요청의 종류는 다양하다.

- `GET` — 페이지 조회 요청 ("홈 페이지 보여줘")
- 로그인 ("나를 인증해 줘")
- 게시글 삭제 ("이 데이터 삭제해 줘")

이 모든 상호작용이 요청-응답 사이클 위에서 동작한다.

### 1-3. 프론트엔드와 백엔드

웹 개발은 역할에 따라 크게 두 영역으로 나뉜다.

| 구분 | 역할 | SSAFY 커리큘럼 |
|------|------|----------------|
| **프론트엔드** | 브라우저 화면 구성, 사용자 인터페이스, 데이터 흐름 시각화 | HTML/CSS → JavaScript → Vue.js |
| **백엔드** | 서버 측 로직 처리, 데이터베이스 연동, 인증/보안 | Django → Django REST Framework |

> **강사님 강조**: "Django는 오늘 배우는 시점에선 프론트엔드와 백엔드를 모두 다루지만, 나중에 DRF를 배울 때는 본격적으로 백엔드 영역만 개발한다. 오늘은 전체 흐름을 파악하는 게 목표야."

SSAFY 1학기 전체 흐름:

```
Python  →  HTML/CSS  →  알고리즘  →  AI  →  Django(백엔드)  →  JS  →  Vue(프론트엔드)
 [기반]     [클라이언트]   [로직]     [서비스]   [서버 개발]       [언어]    [프레임워크]
```

> **강사님 팁**: "1월부터 3월까지가 4월 이후를 위한 준비 단계였다. 파이썬, HTML/CSS, 알고리즘, AI — 이 모든 게 오늘부터 쌓아올릴 서비스 개발의 기초다."

---

## 2. 프레임워크와 Django

### 2-1. 프레임워크가 필요한 이유

웹 서비스를 처음부터 끝까지 직접 구현하면 어떻게 될까?

- 로그인/로그아웃 — 어떻게 서버가 '나'를 기억하게 할까?
- 데이터베이스 — 회원 정보를 어디에, 어떻게 저장할까?
- 보안 — 비밀번호를 평문으로 저장하면 안 된다. 어떻게 암호화하지?
- URL 라우팅 — `/login`으로 접근하면 어떤 코드가 실행돼야 하지?

이 모든 것을 매번 바닥부터 만들면 비효율적이다. **프레임워크**는 이러한 반복적인 구조와 규칙을 미리 정의해 두어 빠르고 안전하게 개발할 수 있도록 돕는다.

> **강사님 강조**: "파이토치가 없었다면 CNN, 퍼셉트론을 직접 구현해야 했겠지? 웹 개발도 마찬가지다. 프레임워크는 우리가 핵심 로직에만 집중할 수 있게 해준다."

### 2-2. Django를 선택하는 이유

Django는 Python 기반의 웹 프레임워크다. 같은 계열의 Flask, FastAPI와 비교하면:

| 프레임워크 | 특징 | 비유 |
|-----------|------|------|
| **Flask** | 매우 가벼움, 자유도 높음 | 조립식 키트 |
| **Django** | 배터리 포함, 균형 잡힘 | 완성형 도구 |
| **FastAPI** | 비동기·고속, 현대적 | 레이싱카 |

Django를 선택하는 현실적인 이유:

- **실사용 사례** — Spotify, Instagram, Dropbox, Delivery Hero가 Django로 만들어졌다
- **쉽다** — 세 프레임워크 중 입문 장벽이 가장 낮다
- **배터리 포함** — 인증, 관리자 페이지, ORM 등이 기본 내장
- **보안** — CSRF, SQL Injection 방어가 기본 제공

> **강사님 강조**: "스포티파이, 인스타그램. 이 한 장이면 설명 끝이다. 그리고 뭣보다 장고가 쉬워. 먼저 쉬운 걸로 서버 개발이 어떤 흐름인지 익히는 게 목표다."

> **강사님 팁**: "프레임워크 이름에 집착하지 마. Django로 배운 '서버 구현 방법'의 흐름을 이해하면 어떤 프레임워크든 적용할 수 있어. 회사마다 자체 프레임워크도 있거든."

---

## 3. 가상 환경 (Virtual Environment)

### 3-1. 가상 환경이 필요한 이유

하나의 컴퓨터에서 여러 프로젝트를 동시에 개발하면 문제가 생긴다.

**시나리오:** 프로젝트 A는 `requests==1.0`, 프로젝트 B는 `requests==2.0`이 필요한 경우

```
글로벌 환경에 패키지 설치
  ├── requests 1.0 설치 (프로젝트 A용)
  └── requests 2.0 설치 (덮어쓰기!)
          ↓
    프로젝트 A가 망가진다
```

패키지 설치 = 파이썬 파일을 내 컴퓨터의 특정 폴더에 다운로드하는 것이다. 같은 이름의 패키지를 두 번 설치하면 **덮어쓰기**가 발생한다.

> **강사님 강조**: "패키지 설치란 결국 파이썬 코드가 담긴 파일을 내려받는 거야. 대단한 마법이 아니야. 같은 이름의 파일이 두 개 있을 수 없으니까 덮어써지는 거고."

더 심각한 경우: 패키지 간 **의존성 충돌**. 서로 다른 패키지가 상충할 경우 코드 자체가 실행되지 않는다.

**해결책:** 프로젝트마다 독립된 Python 환경(방)을 만들어서 사용한다.

```
글로벌 환경 (아무것도 설치 안 함)
  ├── 프로젝트_A/
  │   └── .venv/  ← requests 1.0
  └── 프로젝트_B/
      └── .venv/  ← requests 2.0
```

> **강사님 팁**: "집 안에 독립된 방을 여러 개 만들어서 방마다 물건을 따로 보관한다고 생각해. 방끼리는 서로 영향 안 미치잖아."

### 3-2. venv 생성·활성화·비활성화

```bash
# 1. 프로젝트 폴더 생성 & 이동
mkdir my_project
cd my_project

# 2. 가상 환경 생성 (venv 폴더 이름은 관례상 .venv 또는 venv)
python -m venv venv

# 3. 가상 환경 활성화
# Windows (Git Bash / bash)
source venv/Scripts/activate

# macOS / Linux
source venv/bin/activate

# 4. 활성화 확인 (터미널 왼쪽에 (venv) 표시됨)
(venv) $ pip list        # 설치된 패키지 목록 (거의 비어 있어야 정상)

# 5. 비활성화
deactivate
```

> **강사님 강조**: "앞으로 글로벌 환경에는 아무것도 설치하지 마. 프로젝트 하나당 가상 환경 하나. 이 원칙만 지키면 충돌 문제는 없어."

**venv 폴더 내부 구조:**

```
venv/
├── Include/
├── Lib/          ← 여기에 패키지들이 설치됨
├── Scripts/      ← python.exe, pip.exe, activate 스크립트
└── pyvenv.cfg
```

이 구조는 스타트 캠프 때 설치한 파이썬 폴더(`C:/Users/.../Python311/`)와 동일하다. 가상 환경이란 **또 다른 파이썬 환경**을 프로젝트 폴더 안에 만드는 것이다.

### 3-3. .gitignore 설정

venv 폴더는 용량이 크고 OS마다 다르게 생성된다. 반드시 Git에서 제외한다.

```gitignore
# .gitignore
venv/
.venv/
__pycache__/
*.pyc
db.sqlite3
```

---

## 4. Django 프로젝트 시작

### 4-1. Django 설치

가상 환경이 활성화된 상태에서 설치한다.

```bash
(venv) $ pip install django

# 설치 확인
(venv) $ python -m django --version
# 4.x.x 또는 5.x.x 출력
```

### 4-2. 프로젝트 생성

```bash
# django-admin startproject 프로젝트명 .
# 끝에 .을 붙이면 현재 폴더를 프로젝트 루트로 사용 (폴더 중복 생성 방지)
(venv) $ django-admin startproject my_project .
```

생성된 폴더 구조:

```
my_project/          ← 작업 루트
├── manage.py        ← Django 명령어 실행 도구
├── my_project/      ← 프로젝트 설정 패키지
│   ├── __init__.py
│   ├── settings.py  ← 전체 설정 (DB, 앱, 언어, 시간대 등)
│   ├── urls.py      ← 최상위 URL 라우터
│   ├── asgi.py
│   └── wsgi.py
└── venv/
```

> **강사님 팁**: "`startproject 이름 .` 뒤에 점(.)을 꼭 붙여야 해. 안 붙이면 폴더가 이중으로 생겨서 경로가 복잡해져."

### 4-3. 개발 서버 실행

```bash
(venv) $ python manage.py runserver
```

브라우저에서 `http://127.0.0.1:8000/` 접속 → Django 로켓 화면 확인.

```
Django version 4.x.x, using settings 'my_project.settings'
Starting development server at http://127.0.0.1:8000/
```

---

## 5. MTV 디자인 패턴

### 5-1. 디자인 패턴이란

**디자인(Design)** 이란 예쁘게 만드는 것이 아니다. 소프트웨어에서 디자인 패턴은 **반복되는 문제를 해결하기 위한 검증된 구조**를 의미한다.

> **강사님 강조**: "디자인이라고 하면 예쁘게 만드는 거라고 생각하는 경우가 많은데, 여기서 말하는 디자인 패턴은 '어떻게 만들 것인가'의 설계 방식이야."

Django는 **MTV(Model-Template-View)** 패턴을 따른다. 이는 소프트웨어 공학에서 널리 쓰이는 MVC 패턴에서 유래했다.

| MVC | Django MTV | 역할 |
|-----|-----------|------|
| Model | **Model** | 데이터 구조 정의, DB 연동 |
| View | **Template** | 사용자에게 보여지는 화면 (HTML) |
| Controller | **View** | 요청 처리 로직, 응답 반환 |

> **강사님 팁**: "MVC에서 View가 화면, Django MTV에서 Template이 화면이야. 이름이 헷갈리니까 'Django에서 View는 Controller다'라고 기억해."

### 5-2. 요청-응답 흐름

```
사용자 (브라우저)
    │
    │ HTTP 요청 (예: GET /index/)
    ▼
urls.py  ← URL 패턴 매칭
    │
    │ 해당 View 함수 호출
    ▼
views.py ← 비즈니스 로직 처리
    │         (필요시 Model에서 데이터 조회)
    │
    │ render() → Template에 데이터 전달
    ▼
templates/ ← HTML 생성
    │
    │ HTTP 응답 (완성된 HTML)
    ▼
사용자 (브라우저 화면에 출력)
```

> **강사님 강조**: "URL 만들고, View 함수 만들고, Template 만들고. 이 세 단계를 한 달 동안 계속 반복한다. 지금 당장 외우려 하지 말고 흐름만 기억해."

---

## 6. 앱 생성과 등록

### 6-1. Django 앱이란

Django 프로젝트는 여러 **앱(App)** 의 집합이다. 앱은 특정 기능 단위의 독립적인 모듈이다.

```
프로젝트 (my_project)
├── 앱1: articles  (게시글 CRUD)
├── 앱2: accounts  (회원가입/로그인)
└── 앱3: comments  (댓글)
```

### 6-2. 앱 생성

```bash
(venv) $ python manage.py startapp articles
```

생성된 구조:

```
articles/
├── migrations/      ← DB 변경 이력
│   └── __init__.py
├── __init__.py
├── admin.py         ← 관리자 페이지 등록
├── apps.py          ← 앱 설정
├── models.py        ← 데이터 모델 (Model)
├── tests.py
└── views.py         ← 뷰 함수 (View)
```

### 6-3. 앱 등록 (필수!)

생성만 하면 Django가 인식하지 못한다. `settings.py`의 `INSTALLED_APPS`에 반드시 등록해야 한다.

```python
# my_project/settings.py

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # 내가 만든 앱 등록
    'articles',
]
```

> **강사님 강조**: "앱을 만들고 나서 `INSTALLED_APPS`에 등록 안 하면 Django가 그 앱의 존재를 모른다. 앱 만들면 바로 등록하는 습관 들여."

---

## 7. URL 라우팅

### 7-1. URL 처리 흐름

클라이언트가 `/index/`로 요청을 보내면 Django는 URL 패턴을 순서대로 확인해 일치하는 뷰를 실행한다.

### 7-2. 프로젝트 urls.py — 앱 URL 위임

```python
# my_project/urls.py

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    # articles 앱의 URL을 articles/urls.py에 위임
    path('articles/', include('articles.urls')),
]
```

### 7-3. 앱 urls.py 생성

앱 폴더에 `urls.py`를 직접 생성한다 (자동 생성되지 않음).

```python
# articles/urls.py

from django.urls import path
from . import views

urlpatterns = [
    # http://127.0.0.1:8000/articles/index/ → views.index 실행
    path('index/', views.index, name='index'),
]
```

`path()` 함수 인자:

| 인자 | 설명 | 예시 |
|------|------|------|
| `route` | URL 패턴 문자열 | `'index/'` |
| `view` | 실행할 View 함수 | `views.index` |
| `name` | URL 역참조용 이름 (선택) | `'index'` |

---

## 8. View 함수

### 8-1. View 함수 기본 구조

View 함수는 **요청(request)을 받아 응답(response)을 반환**하는 함수다.

```python
# articles/views.py

from django.shortcuts import render
from django.http import HttpResponse

# 가장 단순한 형태 — 문자열 직접 반환
def index(request):
    return HttpResponse('<h1>Hello, Django!</h1>')
```

- `request` — 클라이언트의 요청 정보가 담긴 객체 (필수 매개변수)
- `HttpResponse` — 문자열·HTML을 직접 반환
- `render` — Template 파일을 렌더링해서 반환 (일반적인 방법)

### 8-2. render()로 Template 반환

```python
# articles/views.py

from django.shortcuts import render

def index(request):
    # render(request, '템플릿 경로', context딕셔너리)
    return render(request, 'articles/index.html')
```

`render()` 함수 인자:

| 인자 | 설명 |
|------|------|
| `request` | 요청 객체 (그대로 전달) |
| `template_name` | 템플릿 파일 경로 |
| `context` | 템플릿에 전달할 데이터 딕셔너리 (선택) |

---

## 9. Template

### 9-1. 템플릿 폴더 구조

Django는 기본적으로 각 앱 안의 `templates/` 폴더를 탐색한다.

```
articles/
├── templates/
│   └── articles/       ← 앱 이름과 동일한 중간 폴더 필수!
│       └── index.html
└── views.py
```

> **강사님 강조**: "`templates/articles/index.html`처럼 중간에 앱 이름 폴더를 하나 더 만드는 이유는, 여러 앱에 같은 이름의 `index.html`이 있을 때 어느 앱 것인지 구분하기 위해서야. 내일 이 경로 이야기 더 깊게 다룰 거야."

### 9-2. 기본 HTML 템플릿 작성

```html
<!-- articles/templates/articles/index.html -->
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>Index Page</title>
</head>
<body>
    <h1>인덱스 페이지</h1>
    <p>Django MTV 패턴으로 반환된 첫 번째 페이지입니다.</p>
</body>
</html>
```

### 9-3. context 데이터 전달

View에서 Template으로 데이터를 넘길 때는 딕셔너리 형태로 전달한다.

```python
# articles/views.py

def index(request):
    name = '김구현'
    context = {
        'name': name,
    }
    return render(request, 'articles/index.html', context)
```

```html
<!-- articles/templates/articles/index.html -->
<h1>안녕하세요, {{ name }}님!</h1>
```

템플릿에서 `{{ 변수명 }}` 구문으로 context 값을 출력한다. 이것이 **DTL(Django Template Language)**의 기본이다.

---

## 10. settings.py 주요 항목

```python
# my_project/settings.py

# 개발 중에만 True. 배포 시 반드시 False로 변경
DEBUG = True

# 허용할 호스트 목록
ALLOWED_HOSTS = []

# 등록된 앱 목록
INSTALLED_APPS = [...]

# 템플릿 설정
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],          # 프로젝트 공통 템플릿 폴더 경로
        'APP_DIRS': True,    # 각 앱의 templates/ 폴더 자동 탐색
        ...
    },
]

# 데이터베이스 설정 (기본값: SQLite)
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}

# 언어 설정
LANGUAGE_CODE = 'ko-kr'

# 시간대 설정
TIME_ZONE = 'Asia/Seoul'
```

| 항목 | 기본값 | 설명 |
|------|--------|------|
| `DEBUG` | `True` | 개발용 에러 페이지 표시 |
| `LANGUAGE_CODE` | `'en-us'` | 관리자 페이지 언어 |
| `TIME_ZONE` | `'UTC'` | 서버 시간대 |
| `APP_DIRS` | `True` | 앱 내 templates/ 폴더 자동 탐색 |

---

## 11. 전체 흐름 실습 — 처음부터 끝까지

프로젝트 생성부터 화면 출력까지 순서대로:

```bash
# Step 1. 폴더 생성 및 이동
mkdir django_intro && cd django_intro

# Step 2. 가상 환경 생성 및 활성화
python -m venv venv
source venv/Scripts/activate    # Windows

# Step 3. Django 설치
pip install django

# Step 4. 프로젝트 생성
django-admin startproject my_project .

# Step 5. 앱 생성
python manage.py startapp articles

# Step 6. settings.py에 앱 등록
# INSTALLED_APPS에 'articles' 추가

# Step 7. 앱 urls.py 생성
# articles/urls.py 파일 직접 생성

# Step 8. 프로젝트 urls.py에 include 추가
# path('articles/', include('articles.urls'))

# Step 9. View 함수 작성
# articles/views.py

# Step 10. Template 작성
# articles/templates/articles/index.html

# Step 11. 개발 서버 실행
python manage.py runserver
# http://127.0.0.1:8000/articles/index/ 접속
```

요청 흐름 최종 정리:

```
브라우저 GET /articles/index/
    → my_project/urls.py: 'articles/' → articles/urls.py로 위임
    → articles/urls.py: 'index/' → views.index 실행
    → articles/views.py: render(request, 'articles/index.html') 반환
    → articles/templates/articles/index.html: HTML 생성
    → 브라우저 화면에 출력
```

> **강사님 강조**: "URL 만들고 → View 함수 만들고 → Template 만들고. 이 세 단계를 한 달 동안 계속 반복한다. 지금 당장 외울 필요 없어. 반복하면 자연스럽게 손이 기억한다."

---

## 정리

| 구분 | 핵심 포인트 |
|------|-------------|
| 웹 애플리케이션 | 브라우저(클라이언트)가 URL로 요청 → 서버가 HTML로 응답 |
| 프레임워크 | 반복 구조를 미리 정의해둔 도구. Django = Python 기반 웹 프레임워크 |
| 가상 환경 | 프로젝트마다 독립된 Python 환경. `python -m venv venv` 생성, `activate` 활성화 |
| 앱 등록 | `startapp`으로 생성 후 `settings.py`의 `INSTALLED_APPS`에 반드시 추가 |
| MTV 패턴 | Model(데이터) / Template(화면) / View(로직). MVC의 Django식 변형 |
| URL 라우팅 | `urls.py`에서 URL 패턴 → View 함수 연결. 앱마다 별도 `urls.py` 사용 |
| View → Template | `render(request, 'template_path', context)`로 HTML 반환 |
