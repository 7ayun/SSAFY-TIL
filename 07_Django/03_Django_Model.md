# [Django] 모델 — Model 정의와 마이그레이션

> 📌 핵심 키워드: #Model #Migration #ORM #MTV #Admin #Field #makemigrations #migrate

---

## 학습 목표

* Django 모델이 무엇인지, 왜 클래스로 정의하는지 설명할 수 있다
* `models.Model`을 상속받아 Article 모델을 직접 정의할 수 있다
* `makemigrations`와 `migrate`의 차이를 이해하고 마이그레이션 루틴을 실행할 수 있다
* `__str__` 매직 메서드의 역할과 마이그레이션과의 관계를 구분할 수 있다
* Admin 페이지에서 모델을 등록하고 데이터를 확인할 수 있다

---

## 1. 오늘 수업 개요

### 1-1. MTV 패턴에서 Model의 위치

Django는 MTV(Model-Template-View) 디자인 패턴으로 구성된다.

| 구성 요소 | 역할 | 학습 시점 |
|-----------|------|-----------|
| Template (T) | HTML 렌더링 | 어제 |
| **Model (M)** | **데이터 구조 설계 및 DB 연동** | **오늘** |
| View (V) | 데이터 조작 로직 (CRUD) | 내일 |


### 1-2. 모델이란

모델은 데이터베이스에 테이블을 정의하고, 데이터를 조작할 수 있는 기능들을 제공하는 **테이블 구조의 청사진(Blueprint)** 이다.

* 파이썬 클래스 형태로 작성한다
* 이 클래스를 기반으로 DB 테이블이 자동 생성된다
* OOP에서 배운 클래스 개념을 그대로 활용한다

---

## 2. 프로젝트 환경 설정

### 2-1. 기본 설정 명령어 순서

```bash
# 1. 가상환경 생성 및 활성화
python -m venv venv
source venv/bin/activate

# 2. Django 설치
pip install django

# 3. 설치 확인
python -m django --version

# 4. 패키지 목록 저장
pip freeze > requirements.txt

# 5. 프로젝트 생성 (현재 위치에)
django-admin startproject third_pjt .

# 6. 앱 생성
python manage.py startapp articles
```

### 2-2. 앱 생성 명령어 두 가지 비교

앱을 생성하는 방법은 두 가지가 있다.

| 명령어 | 설명 | 사용 시점 |
|--------|------|-----------|
| `python manage.py startapp articles` | 현재 프로젝트의 매니저를 통해 앱 생성 | 이미 프로젝트가 존재할 때 (일반적) |
| `django-admin startapp articles` | Django에게 직접 앱 생성 요청 | 프로젝트 없이 앱 기능만 먼저 개발할 때 |

두 방법 모두 기능적 차이는 없다. `manage.py`를 통해 만들면 해당 프로젝트에서 앱이 정상 작동하는지 함께 검증할 수 있다는 실용적 이점이 있다.

### 2-3. settings.py에 앱 등록

앱을 생성한 것만으로는 프로젝트에서 사용할 수 없다. 반드시 `settings.py`의 `INSTALLED_APPS`에 등록해야 한다.

```python
# settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    ...
    'articles',   # ← 반드시 추가
]
```

> ⚠️ 앱 등록을 누락하면 `makemigrations` 실행 시 `No changes detected` 오류가 발생한다. 모델을 수정했는데 변경사항이 감지되지 않는다면 앱 등록 여부와 `models.py` 저장 여부를 먼저 확인한다.

---

## 3. 모델(Model) 정의

### 3-1. 클래스를 사용하는 이유

게시글이라는 데이터는 공통된 구조를 가진다. 1번 게시글이든 1234번 게시글이든 모두 제목, 내용, 작성일, 수정일이라는 동일한 틀을 가진다. OOP에서 공통된 특징을 가진 대상들을 클래스로 정의했던 것처럼, 모델도 같은 방식으로 작성한다.


### 3-2. models.Model 상속

게시글을 저장·조회·수정·삭제하기 위한 기능을 처음부터 직접 구현하면 매우 방대한 작업이 된다. Django는 이미 이러한 기능들을 `models.Model` 클래스 안에 구현해두었다. 이것을 상속받아 사용한다.

```python
# articles/models.py
from django.db import models

class Article(models.Model):
    pass  # 이후 필드 추가 예정
```

`CharField`, `TextField` 등 각종 필드 클래스 역시 `models` 모듈 안에 있는 클래스들이다. 파스칼케이스로 작성되어 있으며 모두 클래스다.

### 3-3. 필드(Field) 종류

필드란 데이터베이스 테이블의 각 컬럼을 어떤 데이터 타입과 제약 조건으로 만들 것인지 정의하는 것이다. `models.` 뒤에 `.`을 찍으면 VS Code에서 사용 가능한 필드 목록이 자동완성으로 표시된다.

| 필드 클래스 | 설명 | 주요 옵션 |
|-------------|------|-----------|
| `CharField` | 짧은 문자열 | `max_length` 필수 |
| `TextField` | 긴 텍스트 | - |
| `IntegerField` | 정수 | - |
| `FloatField` | 부동소수점 | - |
| `BooleanField` | True/False | - |
| `DateField` | 날짜 | `auto_now`, `auto_now_add` |
| `TimeField` | 시간 | - |
| `DateTimeField` | 날짜+시간 | `auto_now`, `auto_now_add` |
| `FileField` | 파일 | - |
| `ImageField` | 이미지 | - |

필드 종류가 많으므로 전부 외울 필요 없다. 이름만 보면 용도를 충분히 유추할 수 있고, 필요할 때 Django 공식 문서의 **Model field reference** 페이지를 참고한다.

### 3-4. 주요 필드 옵션

```python
# 예시
title = models.CharField(max_length=30)
```

| 옵션 | 설명 |
|------|------|
| `max_length` | `CharField`에 필수. 최대 글자 수 제한 |
| `null=True` | DB에 NULL 값 허용 (값 자체가 없는 것) |
| `blank=True` | 빈 문자열(`""`) 허용 (폼 유효성 검사에서 빈 값 허용) |
| `default` | 값이 제공되지 않을 때 사용할 기본값 |

`null`과 `blank`의 차이를 반드시 구분해야 한다. `null`은 파이썬의 `None`처럼 값 자체가 존재하지 않는 것이고, `blank`는 빈 문자열(`""`)처럼 값이 비어있는 것이다. 데이터 분석에서 결측치와 빈 값을 다르게 처리했던 것과 같은 맥락이다.

> ⚠️ `max_length`와 같이 필드에 제약 조건을 설정해도, 저장 전 **유효성 검사(validation)**를 거치지 않으면 제약 조건이 적용되지 않고 그냥 저장된다. 유효성 검사는 내일 View에서 다룬다.

### 3-5. 날짜/시간 자동 저장 옵션

`DateTimeField`에서 날짜와 시간을 자동으로 저장하려면 `auto_now_add` 또는 `auto_now` 옵션을 사용한다.

| 옵션 | 동작 | 사용 용도 |
|------|------|-----------|
| `auto_now_add=True` | 레코드가 **최초 생성될 때** 자동으로 현재 시간 저장 | 생성일 (`created_at`) |
| `auto_now=True` | 레코드가 **저장될 때마다(수정 포함)** 자동으로 현재 시간 갱신 | 수정일 (`updated_at`) |

> ⚠️ 이 두 옵션은 매우 헷갈리기 쉽다. `auto_now_add`는 최초 생성 시점에만 저장, `auto_now`는 저장(수정)할 때마다 갱신된다는 차이를 반드시 기억한다.

일반 사용자가 게시글 작성 시 날짜를 직접 입력하는 경우는 없다. 이 두 옵션을 사용하면 Django가 날짜·시간을 자동으로 처리해준다.

### 3-6. Article 모델 최종 코드

```python
# articles/models.py
from django.db import models

class Article(models.Model):
    title = models.CharField(max_length=30)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return self.title
```

---

## 4. 마이그레이션 (Migration)

### 4-1. 마이그레이션이란

파이썬으로 작성한 모델(클래스)을 데이터베이스에 반영하기 위한 **이주(Migration) 작업**이다. 직접 SQL을 쓰는 대신 Django가 자동으로 처리한다.

마이그레이션은 두 단계로 이루어진다.

```
models.py 수정
    ↓
makemigrations (설계도 생성)
    ↓
migrate (DB에 반영)
```

### 4-2. makemigrations — 설계도 생성

```bash
python manage.py makemigrations
```

`models.py`에 정의된 클래스를 분석하여 DB 반영을 위한 **설계도 파일**을 자동 생성한다. 설계도 파일은 `migrations/` 폴더 안에 순번 형태로 저장된다.

```
articles/
└── migrations/
    ├── 0001_initial.py             ← 최초 모델 생성 설계도
    └── 0002_article_created_at_article_updated_at.py  ← 필드 추가 설계도
```

설계도 파일은 모델의 변경 히스토리를 기록하는 역할도 한다. 나중에 투입된 팀원도 `migrations/` 폴더를 역순으로 보면 DB 구조가 어떻게 변해왔는지 파악할 수 있다. 따라서 설계도 파일은 **삭제하지 않으며 Git으로 반드시 관리한다.**

### 4-3. migrate — DB에 반영

```bash
python manage.py migrate
```

설계도 파일을 실제 데이터베이스(`db.sqlite3`)에 적용한다. 처음 실행하면 Django 내부에서 사전 정의한 18개의 기본 마이그레이션도 함께 적용된다.

`python manage.py runserver` 실행 시 붉은색으로 `You have N unapplied migration(s).` 경고가 표시된다면 `migrate`를 아직 실행하지 않은 것이다.

### 4-4. id(PK) 자동 생성

`makemigrations` 결과를 보면 직접 정의하지 않은 `id` 필드가 자동 추가되어 있다.

```python
# 0001_initial.py 내용 요약
migrations.CreateModel(
    name='Article',
    fields=[
        ('id', models.BigAutoField(primary_key=True, ...)),  # ← 자동 추가됨
        ('title', models.CharField(max_length=30)),
        ('content', models.TextField()),
    ],
)
```

`id`는 각 레코드를 고유하게 식별하는 **Primary Key**다. 1번 게시글과 2번 게시글이 제목과 내용이 동일하더라도 `id`로 구분할 수 있다. Django가 기본으로 정수형 자동증가 값(`AutoField`)으로 만들어주므로 별도 정의가 불필요하다.

### 4-5. 기존 데이터가 있을 때 필드 추가 시 주의

이미 DB에 데이터가 존재하는 테이블에 새 필드를 추가하면, `makemigrations` 실행 시 기존 레코드에 어떤 값을 채울지 묻는 대화형 프롬프트가 나타난다.

```
Please select a fix:
 1) Provide a one-off default now
 2) Quit and manually define a default value
```

`1`번을 선택하면 터미널 안에서 즉시 기본값을 지정할 수 있다. 날짜 필드에 `auto_now_add`를 사용하는 경우 프롬프트에서 기본값으로 `django.utils.timezone.now`가 제안되며, 그냥 Enter를 치면 현재 시간으로 채워진다.

`2`번을 선택하면 작업을 종료하고 `models.py`에 직접 `default` 값을 명시하는 방식이다.

### 4-6. 타임존(Timezone) 주의사항

날짜·시간 데이터는 `settings.py`의 `USE_TZ` 설정에 영향을 받는다.

```python
# settings.py
TIME_ZONE = 'UTC'   # 기본값: UTC (협정 세계시)
USE_TZ = True
```

Django의 기본 타임존은 **UTC**이므로, 한국 시간(KST, UTC+9)과 **9시간 차이**가 발생한다. 오전 10시 25분에 생성한 게시글이 DB에는 오전 1시 25분으로 저장될 수 있다. 한국 시간으로 저장하려면 `TIME_ZONE = 'Asia/Seoul'`로 변경한다.

```python
# settings.py (한국 시간 설정 시)
TIME_ZONE = 'Asia/Seoul'
USE_TZ = True
```

### 4-6. 마이그레이션 루틴 정리

```
1. models.py에서 클래스(필드) 추가/수정
        ↓
2. python manage.py makemigrations   (설계도 생성)
        ↓
3. python manage.py migrate          (DB 반영)
        ↓
4. 반복 (모델 변경사항 발생 시마다)
```

### 4-7. 자주 만나는 오류

| 오류 메시지 | 원인 | 해결 방법 |
|-------------|------|-----------|
| `No changes detected` | 앱이 `INSTALLED_APPS`에 등록되지 않았거나, `models.py`를 저장하지 않음 | `settings.py` 확인 후 저장 |
| `no such table` | `migrate`를 실행하지 않았거나 모델 수정 후 마이그레이션 루틴 미실행 | `python manage.py migrate` 실행 |

---

## 5. `__str__` 메서드

### 5-1. 역할

파이썬에서 객체를 `print()`하면 기본적으로 메모리 주소가 출력된다. Django 모델에서는 PK값이 출력되도록 설정되어 있다. 사람이 읽기 좋은 형태로 출력하고 싶을 때 `__str__` 매직 메서드를 정의한다.

```python
class Article(models.Model):
    title = models.CharField(max_length=30)
    content = models.TextField()

    def __str__(self):
        return self.title
```

이렇게 정의하면 admin 페이지나 터미널에서 Article 객체를 출력할 때 PK값 대신 제목이 표시된다.

### 5-2. ⚠️ makemigrations가 필요하지 않은 경우

`__str__`은 **파이썬 동작 방식만 바꾸는 것**으로, 데이터베이스 테이블 구조에는 아무런 영향을 미치지 않는다. 따라서 `__str__`을 추가하거나 수정한 후에는 `makemigrations`를 실행할 필요가 없다. 실행해도 `No changes detected`가 출력된다.

> 💬 "makemigrations로 기대하는 건 데이터베이스의 테이블이 바뀌길 바라는 거예요. `__str__`은 데이터베이스와 아무 상관없는 친구입니다."

---

## 6. Admin(관리자) 페이지

### 6-1. 관리자 페이지란

Django가 기본 제공하는 백오피스 페이지(`/admin/`)로, 모델에 등록된 데이터를 GUI로 확인·생성·수정·삭제할 수 있다. 오늘 수업에서는 내일 View 작성 전에 모델이 정상적으로 동작하는지 확인하기 위해 사용한다.

### 6-2. 슈퍼유저 생성

관리자 페이지에 로그인하려면 슈퍼유저 계정이 필요하다. manage.py를 통해 생성한다.

```bash
python manage.py createsuperuser
```

실행 후 순서대로 입력한다.

```
Username: admin
Email address: (엔터로 생략 가능)
Password: ********
Password (again): ********
```

비밀번호는 Django가 자동으로 유효성 검사를 수행한다(최소 8자, 숫자만으로 구성 불가, 너무 흔한 비밀번호 불가). 테스트 환경에서 짧은 비밀번호를 사용하려면 경고 메시지 후 `y`를 입력하여 강제 생성할 수 있다.

저장된 비밀번호를 확인하면 해싱(Hashing) 알고리즘이 적용된 암호화된 값이 들어있다. Django가 자동으로 처리해준다.

### 6-3. admin.py에 모델 등록

모델을 만들고 마이그레이션을 완료해도 admin 페이지에 자동으로 나타나지 않는다. 관리자 페이지에서 보이게 하려면 `admin.py`에 수동으로 등록해야 한다.

```python
# articles/admin.py
from django.contrib import admin
from .models import Article      # ← 상대 경로 임포트

admin.site.register(Article)
```

`from .models import Article`에서 `.`(점)의 의미는 **현재 패키지(폴더) 기준의 상대 경로**다. Django MTV 패턴에서는 앱이 여러 개 존재하고 각 앱마다 `models.py`가 있다. 어떤 앱의 `models`인지 명확히 지정하기 위해 점 표기법을 사용한다.

| 구문 | 의미 |
|------|------|
| `from .models import Article` | 현재 폴더(`articles/`)의 `models.py`에서 `Article` 가져오기 |
| `from articles.models import Article` | 절대 경로로 동일한 의미 |

등록 후 서버를 재시작하면 admin 페이지에 `Articles` 섹션이 생기고, 게시글 조회·생성·수정·삭제가 가능하다.

### 6-4. Admin 페이지 표시 커스터마이징

기본 상태의 admin 목록 페이지는 `__str__`에서 반환한 값(제목)만 표시된다. id, 제목, 날짜 등 여러 컬럼을 목록에서 한눈에 보고 싶다면 Django 공식 문서의 **Admin site** 페이지에서 `list_display` 등의 옵션을 찾아 적용한다.

---

## 7. 참고사항 (수업 후 스스로 읽기)

강사가 수업 후 실습 전에 반드시 읽어보도록 안내한 항목들이다.

### 7-1. SQLite란

Django는 기본 데이터베이스로 **SQLite**를 사용한다. SQLite는 별도 서버 설치 없이 파일(`db.sqlite3`) 하나로 동작하는 경량 DBMS다. 개발·학습 환경에서 주로 사용하며, 실제 서비스 배포 시에는 PostgreSQL, MySQL 등으로 교체한다. SQL 문법과 DBMS 심화 내용은 다다음주 DB 수업에서 다룬다.

### 7-2. DB 삭제 후 재마이그레이션

실습 중 DB가 꼬였을 때의 초기화 방법이다.

```bash
# 1. db.sqlite3 파일 삭제
rm db.sqlite3

# 2. migrations 폴더 안의 숫자로 시작하는 파일들 삭제 (예: 0001_initial.py)
#    __init__.py는 삭제하지 않는다

# 3. 마이그레이션 재생성 및 적용
python manage.py makemigrations
python manage.py migrate
```

> ⚠️ DB를 삭제하면 저장된 데이터가 모두 사라진다. 개발 초기 실습 환경에서만 사용한다.

### 7-3. 주요 migrate 관련 명령어

| 명령어 | 설명 |
|--------|------|
| `python manage.py makemigrations` | 설계도 파일 생성 |
| `python manage.py migrate` | 설계도를 DB에 반영 |
| `python manage.py showmigrations` | 마이그레이션 적용 현황 확인 |
| `python manage.py sqlmigrate articles 0001` | 특정 설계도가 실행하는 SQL 확인 |

---

## 8. 오늘 수업 흐름 정리

```
1. 프로젝트 생성 (third_pjt)
        ↓
2. 앱 생성 (articles) + settings.py 등록
        ↓
3. models.py에 Article 클래스 정의
   (CharField, TextField, DateTimeField)
        ↓
4. makemigrations → 설계도 생성 (0001_initial.py)
        ↓
5. migrate → DB에 반영
        ↓
6. 필드 추가 (created_at, updated_at)
        ↓
7. makemigrations → 새 설계도 생성 (0002_...)
        ↓
8. migrate → DB 반영
        ↓
9. __str__ 메서드 추가 (makemigrations 불필요)
        ↓
10. createsuperuser → 관리자 계정 생성
        ↓
11. admin.py에 Article 등록
        ↓
12. /admin/ 접속 후 게시글 CRUD 확인
```

---

## 📋 핵심 개념 정리

| 개념 | 설명 | 예시/명령어 |
|------|------|-------------|
| Model | DB 테이블 구조의 청사진 | `class Article(models.Model)` |
| CharField | 짧은 문자열 필드 | `models.CharField(max_length=30)` |
| TextField | 긴 텍스트 필드 | `models.TextField()` |
| DateTimeField | 날짜+시간 필드 | `models.DateTimeField()` |
| auto_now_add | 레코드 최초 생성 시 자동 저장 | `auto_now_add=True` |
| auto_now | 레코드 저장(수정)마다 자동 갱신 | `auto_now=True` |
| null | DB에 NULL 허용 여부 | `null=True` |
| blank | 빈 문자열 허용 여부 | `blank=True` |
| Primary Key (id) | 각 레코드의 고유 식별값 | Django가 자동 생성 |
| makemigrations | models.py 변경사항을 설계도 파일로 생성 | `python manage.py makemigrations` |
| migrate | 설계도를 DB에 실제 반영 | `python manage.py migrate` |
| createsuperuser | 관리자 계정 생성 | `python manage.py createsuperuser` |
| admin.site.register | 모델을 admin 페이지에 등록 | `admin.site.register(Article)` |
| 상대경로 임포트 (.) | 현재 폴더 기준 모듈 가져오기 | `from .models import Article` |
| `__str__` | 객체 출력 표현 정의 (DB 무관) | `return self.title` |
