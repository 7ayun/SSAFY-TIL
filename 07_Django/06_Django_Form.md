# [Django] Django Form — Form & ModelForm

> 📌 핵심 키워드: #Form #ModelForm #유효성검사 #is_valid #Widget #Meta클래스 #request.method #뷰함수통합

---

## 학습 목표

* Django Form 클래스를 정의하고 템플릿에서 렌더링하는 방법을 이해한다
* Form과 ModelForm의 차이와 각각의 사용 시점을 구분한다
* Meta 클래스를 사용해 ModelForm을 정의하고 fields / exclude를 제어할 수 있다
* is_valid()를 활용한 유효성 검사와 에러 메시지 처리 방법을 익힌다
* request.method 조건 분기로 new+create, edit+update 뷰 함수를 통합할 수 있다

---

## 1. 수업 개요

* **Django Form** — HTML 입력 양식을 클래스로 정의해 재사용
* **Django ModelForm** — 모델과 연결된 폼을 자동 생성
* **뷰 함수 통합** — request.method 조건 분기로 new+create, edit+update를 하나로 합치기

---

## 2. HTML 폼을 직접 작성할 때의 문제점

### 2-1. 반복 작성 문제

게시글 생성(new.html)과 수정(edit.html)의 폼 태그 구조가 완전히 동일하다. 필드가 늘어날수록 똑같은 HTML을 여러 번 작성해야 한다.

### 2-2. 유효성 검사 불가 문제

HTML 폼만으로는 악의적이거나 비정상적인 요청을 필터링할 수 없다. 예를 들어 `max_length=100`으로 모델을 정의해도, ORM으로 직접 저장할 경우 100글자를 초과한 값도 그냥 저장된다. 유효한 데이터인지 확인하는 유효성 검사(Validation)가 별도로 필요하다.

이러한 문제들을 Django가 제공하는 **Form 클래스**를 사용해 쉽고 빠르게 해결할 수 있다.

---

## 3. Django Form

### 3-1. forms.py 생성

`urls.py`를 앱 폴더에 만들었던 것처럼, 폼을 관리하기 위해 `articles` 폴더 안에 `forms.py` 파일을 새로 만든다.

```
articles/
├── forms.py     ← 새로 생성
├── models.py
├── urls.py
└── views.py
```

### 3-2. Form 클래스 정의

```python
# articles/forms.py
from django import forms

class ArticleForm(forms.Form):
    title = forms.CharField(max_length=10)
    content = forms.CharField()
```

`forms.Form` 클래스를 상속받아 작성한다. 모델에서 필드를 정의했던 것처럼, 사용자가 입력해야 하는 필드들을 클래스 변수로 정의한다.

**Form의 필드 타입 vs 모델의 필드 타입 비교:**

| 구분 | 모델 (models.py) | 폼 (forms.py) |
|------|-----------------|--------------|
| 짧은 문자열 | `CharField(max_length=100)` | `forms.CharField(max_length=10)` |
| 긴 문자열 | `TextField()` | `forms.CharField()` |

Form에는 `TextField`가 없다. Form 필드는 HTML 인풋 태그의 종류를 나타내는 것이고, 인풋 태그는 텍스트냐 숫자냐 체크박스냐 등으로 구분될 뿐 긴 문자열/짧은 문자열로 나뉘지 않는다. 내용(content)도 `forms.CharField()`로 사용한다.

### 3-3. View에서 Form 사용

```python
# articles/views.py
from .forms import ArticleForm

def new(request):
    form = ArticleForm()
    context = {
        'form': form,
    }
    return render(request, 'articles/new.html', context)
```

`ArticleForm` 클래스의 인스턴스를 생성해 context에 담아 넘긴다. 모델에서 `Article` 클래스를 import해서 쓰는 방법과 동일하다.

### 3-4. 템플릿에서 Form 렌더링

```django
<!-- articles/templates/articles/new.html -->
{% extends 'base.html' %}

{% block content %}
  <h1>New Page</h1>
  <form action="{% url 'articles:create' %}" method="POST">
    {% csrf_token %}
    {{ form.as_p }}
    <input type="submit" value="작성">
  </form>
{% endblock content %}
```

`{{ form }}`으로 바로 출력하면 필드들이 한 줄에 이어서 나온다. `{{ form.as_p }}`를 쓰면 각 필드를 `<p>` 태그로 감싸 줄바꿈이 된다.

| 렌더링 방식 | 결과 |
|------------|------|
| `{{ form }}` | 필드들이 한 줄로 출력 |
| `{{ form.as_p }}` | 각 필드를 `<p>` 태그로 묶어 출력 |
| `{{ form.as_div }}` | 각 필드를 `<div>` 태그로 묶어 출력 |
| `{{ form.as_table }}` | `<table>` 형태로 출력 |
| `{{ form.as_ul }}` | `<ul>` 목록 형태로 출력 |

Form 인스턴스는 파이썬 객체이므로 `dir(form)`으로 가지고 있는 속성들을 전부 출력해서 확인할 수 있다.

Form을 사용하면 `max_length=10`으로 설정한 경우 렌더링 시 `input` 태그에 `maxlength="10"` 속성이 자동으로 붙어 10글자 이상 입력 자체가 막힌다.

### 3-5. Widget으로 커스터마이징

위젯(Widget)은 인풋 요소의 타입이나 HTML 속성을 변경하는 도구다.

```python
# articles/forms.py
from django import forms

class ArticleForm(forms.Form):
    title = forms.CharField(max_length=10)
    content = forms.CharField(widget=forms.Textarea)
```

`forms.CharField()`는 기본적으로 `<input type="text">`로 렌더링된다. `widget=forms.Textarea`를 지정하면 `<textarea>`로 바뀐다.

위젯으로 제어할 수 있는 것들:

* 인풋 태그 타입 변경 (텍스트, 체크박스, 텍스트 에어리어 등)
* `placeholder`, `class`, `id`, `maxlength` 등 HTML 속성 추가
* 유효성 검사 실패 시 표시할 에러 메시지 커스터마이징
* 텍스트 에어리어의 `rows`, `cols` 등 사이즈 조절

더 많은 옵션은 Django 위젯 공식 문서에서 확인할 수 있다.

### 3-6. Form 클래스는 언제 쓰나?

Form 클래스는 **데이터베이스에 저장하지 않는** 사용자 입력 데이터를 다룰 때 사용한다.

| 사용 예 | 이유 |
|--------|------|
| 로그인 폼 (username, password) | 로그인할 때마다 DB에 저장하지 않음 |
| 검색 폼 | 검색어를 DB에 저장하지 않음 |
| 데이터 분석 입력 폼 | DB 저장 없이 처리만 함 |

---

## 4. Django ModelForm

### 4-1. Form vs ModelForm

사용자가 입력해야 하는 필드들을 `models.py`에 이미 정의해두었는데, `forms.py`에서 같은 내용을 또 적는 것은 비효율적이다. 이미 정의된 모델 정보를 그대로 폼으로 가져오는 것이 **ModelForm**이다.

| 구분 | Form | ModelForm |
|------|------|-----------|
| 상속 | `forms.Form` | `forms.ModelForm` |
| 사용 시점 | DB에 저장하지 않는 입력 | DB에 저장하는 입력 (모델과 연결) |
| 예시 | 로그인, 검색 | 게시글 생성/수정, 회원가입 |

### 4-2. ModelForm 정의

```python
# articles/forms.py
from django import forms
from .models import Article

class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = '__all__'
```

`forms.ModelForm`을 상속받고, 안에 **Meta 클래스**를 정의한다. Meta 클래스는 이 폼을 정의하기 위해 필요한 메타데이터(model, fields 등)를 담는 내부 클래스다.

### 4-3. Meta 클래스 속성

```python
class Meta:
    model = Article      # 어떤 모델을 기반으로 할 것인가
    fields = '__all__'   # 모델의 모든 필드를 폼에 포함
```

`fields = '__all__'`을 사용하면 `auto_now_add`, `auto_now`처럼 사용자가 입력하지 않고 자동으로 채워지는 필드(`created_at`, `updated_at`)는 자동으로 제외되고, 실제 사용자가 입력해야 하는 `title`, `content`만 폼에 포함된다.

**필드 제어 방법:**

```python
# 방법 1: 포함할 필드를 직접 지정
class Meta:
    model = Article
    fields = ['title', 'content']

# 방법 2: 제외할 필드만 지정 (나머지 전부 포함)
class Meta:
    model = Article
    exclude = ['title']
```

사용할 필드가 적을 때는 `fields`에 리스트로 직접 지정하고, 제외할 필드가 적을 때는 `exclude`를 사용하면 된다.

### 4-4. ModelForm에서 Widget 사용

ModelForm도 Form과 동일하게 위젯을 사용할 수 있다. 클래스 변수로 필드를 재정의하면 된다.

```python
class ArticleForm(forms.ModelForm):
    title = forms.CharField(
        label='제목',
        widget=forms.TextInput(attrs={
            'placeholder': '제목을 입력하세요',
            'class': 'my-input',
        })
    )
    content = forms.CharField(
        widget=forms.Textarea(attrs={
            'rows': 5,
            'cols': 50,
        })
    )

    class Meta:
        model = Article
        fields = '__all__'
```

ModelForm은 내부적으로 Form 클래스를 상속받고 있기 때문에, Form에서 위젯을 사용하는 방법과 동일하다.

ModelForm을 사용하면 `content`가 모델에서 `TextField`로 정의되어 있으므로, 자동으로 `<textarea>`로 렌더링된다. 별도로 위젯을 지정하지 않아도 된다.

---

## 5. 유효성 검사 (is_valid)

### 5-1. is_valid() 메서드

```python
form = ArticleForm(request.POST)
if form.is_valid():
    # 유효성 검사 통과 → 저장
    article = form.save()
    return redirect('articles:index')
```

`is_valid()`는 폼에 담긴 데이터가 유효한지 검사하고 `True` 또는 `False`를 반환한다. 모델에 정의된 제약사항(max_length, blank 등)을 기준으로 판별한다.

Django 모델 필드는 기본적으로 `blank=False`이므로, 빈 문자열은 유효하지 않다.

### 5-2. 유효성 검사 실패 시 처리

유효성 검사에 실패했을 때 `redirect`를 사용하면 사용자가 입력했던 내용이 모두 사라지고 빈 폼으로 이동한다는 문제가 있다. 올바른 처리는 **에러 메시지가 포함된 폼으로 같은 페이지를 다시 렌더링**하는 것이다.

```python
def create(request):
    form = ArticleForm(request.POST)
    if form.is_valid():
        article = form.save()
        return redirect('articles:index')
    # is_valid() 실패 시 → 에러가 담긴 form으로 new.html 다시 렌더링
    context = {'form': form}
    return render(request, 'articles/new.html', context)
```

유효성 검사에 실패하면 `form` 인스턴스 안에 사용자가 입력했던 데이터와 에러 메시지가 함께 포함된다. 이 `form`을 그대로 context에 담아 렌더링하면, 사용자가 입력한 내용은 유지되면서 어떤 필드에서 에러가 발생했는지 알려주는 메시지(예: `This field is required.`)가 화면에 표시된다.

에러 메시지 언어를 한국어로 바꾸고 싶다면 `settings.py`의 `LANGUAGE_CODE`를 `'ko-kr'`로 변경하면 된다.

---

## 6. form.save() — 생성과 수정 구분

`form.save()`는 폼 인스턴스에 담긴 데이터를 데이터베이스에 저장하는 메서드다. 생성(`INSERT`)인지 수정(`UPDATE`)인지는 폼 인스턴스를 만들 때 `instance` 인자 여부로 자동 판별한다.

| 상황 | 코드 | 동작 |
|------|------|------|
| **생성** | `ArticleForm(request.POST)` | DB에 새 레코드 INSERT |
| **수정** | `ArticleForm(request.POST, instance=article)` | 기존 레코드 UPDATE |

수정 시에는 반드시 `instance=article`을 키워드 인자로 넘겨줘야 한다. `instance`가 포함된 폼으로 저장하면 Django가 해당 객체의 PK를 기준으로 UPDATE를 수행한다.

```python
# 수정(update) 뷰 함수 예시
def update(request, article_pk):
    article = Article.objects.get(pk=article_pk)
    form = ArticleForm(request.POST, instance=article)
    if form.is_valid():
        article = form.save()
        return redirect('articles:detail', article.pk)
    ...
```

---

## 7. 뷰 함수 통합 — request.method 조건 분기

### 7-1. new + create 통합

기존에는 `new`(폼 보여주기)와 `create`(데이터 저장) 두 함수가 분리되어 있었다. 유효성 검사 실패 처리를 추가하다 보면, `create` 함수가 `new` 함수와 거의 동일한 코드를 갖게 된다. 이를 **하나의 함수**로 통합하고 `request.method`로 분기한다.

```python
# articles/urls.py
urlpatterns = [
    path('', views.index, name='index'),
    path('create/', views.create, name='create'),  # new 경로 제거, create로 통합
    ...
]
```

```python
# articles/views.py
def create(request):
    if request.method == 'POST':
        # 사용자가 데이터를 제출한 경우
        form = ArticleForm(request.POST)
        if form.is_valid():
            article = form.save()
            return redirect('articles:index')
    else:
        # GET 요청: 빈 폼 페이지를 보여주는 경우
        form = ArticleForm()
    context = {'form': form}
    return render(request, 'articles/create.html', context)
```

### 7-2. POST 기준으로 분기하는 이유

`if request.method == 'GET'`으로 분기하는 것과 `POST`로 분기하는 것 중 어느 쪽이 좋을까?

> 💬 "데이터의 변화를 발생시키는 요청인 것을 기준으로 조건 분기를 하는 쪽이 조금 더 단단한 형태의 웹 서비스를 구성할 수가 있을 것 같아요."

POST 요청(데이터 변경)을 첫 번째 조건으로 잡는 이유는, 데이터 변경이 수반되는 중요한 로직을 가장 명확하게 제어할 수 있기 때문이다. GET은 상대적으로 안전하고, POST · PUT · DELETE 등 리소스를 변경하는 요청에 집중해서 로직을 작성하는 것이 더 견고한 서비스를 만드는 방법이다.

**통합된 뷰 함수의 동작 흐름:**

```
GET 요청 (페이지 접속)
  → else 분기: 빈 ArticleForm 생성
  → context에 담아 create.html 렌더링

POST 요청 (폼 제출)
  → if 분기: 사용자 데이터 포함한 ArticleForm 생성
  → is_valid() 통과 → 저장 → 상세 페이지로 redirect
  → is_valid() 실패 → 에러 담긴 form으로 create.html 재렌더링
```

### 7-3. edit + update 통합

수정도 동일한 구조로 통합한다. 생성과의 차이는 `instance=article`을 넣는 것뿐이다.

```python
# articles/urls.py
urlpatterns = [
    ...
    path('<int:article_pk>/update/', views.update, name='update'),  # edit 제거
]
```

```python
# articles/views.py
def update(request, article_pk):
    article = Article.objects.get(pk=article_pk)
    if request.method == 'POST':
        form = ArticleForm(request.POST, instance=article)  # instance 필수
        if form.is_valid():
            article = form.save()
            return redirect('articles:detail', article.pk)
    else:
        form = ArticleForm(instance=article)  # 기존 데이터 채워서 폼 렌더링
    context = {'form': form, 'article': article}
    return render(request, 'articles/update.html', context)
```

GET 요청 시 `ArticleForm(instance=article)`로 인스턴스를 넘기면, 기존 게시글의 제목과 내용이 채워진 폼이 렌더링된다.

> ⚠️ 수정 페이지 렌더링 시 `article`도 함께 context에 넘겨줘야 템플릿에서 `article.pk`로 action URL을 구성할 수 있다.

---

## 8. HTTP 요청과 실질적 경로

Django 서버 로그를 보면 요청이 이렇게 기록된다.

```
GET  /articles/5/update/  →  200
POST /articles/5/update/  →  302
```

URL이 같더라도 **HTTP 메서드(GET/POST)**에 따라 실질적으로 다른 요청으로 처리된다. 경로를 기억할 때는 URL과 HTTP 메서드를 함께 고려해야 한다.

---

## 9. 코드 구조 재사용성

오늘 작성한 뷰 함수 구조는 모델이 바뀌어도 그대로 재사용할 수 있다. `Article`을 `Book`, `Movie`, `Member` 등 어떤 모델로 바꾸더라도 달라지는 것은 클래스 이름뿐이고, 코드 작성 방법 자체는 변하지 않는다.

```python
# Article → Book으로 바꿀 경우
# ArticleForm → BookForm
# article → book
# 그 외 로직은 동일
```

---

## 참고사항

* Django 공식 위젯 문서를 참고하면 인풋 타입, 어트리뷰트, 에러 메시지 등 다양한 커스터마이징 옵션을 확인할 수 있다
* 여유가 된다면 `edit.html`과 `new.html`(또는 `create.html`)을 하나의 폼 템플릿으로 합치는 방법도 찾아보면 좋다

---

## 📋 핵심 개념 정리

| 개념 | 설명 | 예시 |
|------|------|------|
| `forms.Form` | DB 저장 없는 입력 폼 클래스 | 로그인, 검색 폼 |
| `forms.ModelForm` | 모델 기반 입력 폼 자동 생성 | 게시글 생성/수정 폼 |
| `Meta` 클래스 | ModelForm의 메타데이터 정의 내부 클래스 | `model`, `fields`, `exclude` |
| `fields = '__all__'` | 모델의 모든 사용자 입력 필드 포함 | auto_now 등 자동 필드 제외 |
| `exclude` | 특정 필드만 제외하고 나머지 포함 | `exclude = ['title']` |
| `Widget` | 인풋 요소의 타입·속성 커스터마이징 | `widget=forms.Textarea` |
| `{{ form.as_p }}` | 폼을 `<p>` 태그로 감싸 렌더링 | 줄바꿈 있는 형태 |
| `is_valid()` | 폼 데이터 유효성 검사, True/False 반환 | `if form.is_valid():` |
| `form.save()` | 폼 데이터를 DB에 저장 | 생성(instance 없음) / 수정(instance 있음) |
| `instance=article` | save() 시 기존 객체 수정으로 처리 | `ArticleForm(request.POST, instance=article)` |
| `request.method` | 사용자 요청 HTTP 메서드 확인 | `'GET'` 또는 `'POST'` |
| `blank=False` | Django 기본값, 빈 값 허용 안 함 | is_valid() 실패 조건 |
