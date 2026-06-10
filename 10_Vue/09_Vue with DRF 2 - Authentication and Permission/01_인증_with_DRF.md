# [Vue] 인증 with DRF

---

## 1. 사전 준비

인증 로직을 추가하기 전에 이전 실습 코드에서 주석 처리되어 있던 User 관련 코드를 활성화하고, DB를 초기화해야 한다.

### 코드 활성화

**① articles/models.py** — `user` ForeignKey 주석 해제

```python
# articles/models.py

class Article(models.Model):
    user = models.ForeignKey(
        settings.AUTH_USER_MODEL, on_delete=models.CASCADE
    )
    title = models.CharField(max_length=100)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

**② articles/serializers.py** — `read_only_fields` 주석 해제

```python
# articles/serializers.py

class ArticleSerializer(serializers.ModelSerializer):
    class Meta:
        model = Article
        fields = '__all__'
        read_only_fields = ('user',)  # user는 사용자가 직접 입력하는 게 아니라 서버가 자동으로 넣어줌
```

> `read_only_fields`란 클라이언트가 수정할 수 없는 필드를 지정한다. `user`는 요청 객체에서 자동으로 채워야 하므로 읽기 전용으로 설정한다.

**③ articles/views.py** — 게시글 저장 시 user 정보 함께 저장

```python
# articles/views.py

elif request.method == 'POST':
    serializer = ArticleSerializer(data=request.data)
    if serializer.is_valid(raise_exception=True):
        serializer.save(user=request.user)  # serializer.save() → 로그인한 유저 함께 저장
        return Response(serializer.data, status=status.HTTP_201_CREATED)
```

### DB 초기화

기존 데이터에는 `user` 정보가 없기 때문에 그대로 사용하면 충돌이 발생한다.

1. `db.sqlite3` 삭제
2. `articles/migrations/` 폴더 안의 `0001_initial.py` 파일 삭제 (폴더 자체는 삭제 금지)
3. Migration 재진행
4. `fixtures/articles.json`은 user 정보가 없으므로 `loaddata` 불가 → 직접 데이터 생성

---

## 2. 인증(Authentication)의 개념

### 인증이 필요한 이유

HTTP는 기본적으로 **무상태(stateless)** 프로토콜이다. 요청을 보내고 응답을 받으면 연결이 끊기며, 서버는 클라이언트가 누구인지 기억하지 못한다.

기존 Django(순수 장고)에서는 **쿠키(Cookie)와 세션(Session)** 으로 이 문제를 해결했다. 로그인 시 서버가 Session ID를 발급하고, 브라우저는 이를 쿠키에 담아 매 요청마다 자동으로 전송한다.

그러나 Vue + DRF처럼 프론트엔드와 백엔드가 **완전히 분리된 구조**에서는 다른 인증 방식이 필요하다.

### DRF에서의 인증

```
인증은 항상 view 함수가 실행되기 전, 다른 코드의 진행이 허용되기 전에 먼저 실행된다.
```

- 수신된 요청을 해당 사용자(또는 서명된 토큰 등 자격 증명 자료)와 연결한다.
- 인증 완료 후 → 권한(Permission) 및 제한 정책을 확인하고 요청 처리 여부를 결정한다.

> **⚠️ 중요:** 인증 자체로는 요청을 허용하거나 거부할 수 없다. 인증은 단순히 요청에 사용된 자격 증명을 **식별**할 뿐이며, 접근 허용/거부는 이후 **권한(Permission)** 단계에서 결정된다.

### 프론트엔드 vs 백엔드의 역할

| 역할 | 담당 |
|------|------|
| 사용자 정보 검증, 토큰 발급, 권한 규칙 설정 | **DRF (백엔드)** |
| 로그인 폼 제공, 토큰 저장, 매 요청마다 토큰 전송 | **Vue (프론트엔드)** |

---

## 3. HTTP 인증 실패 응답 코드

인증되지 않은 요청이 권한을 거부당할 경우, DRF는 두 가지 오류 코드로 응답한다.

| 코드 | 이름 | 의미 |
|------|------|------|
| **401** | Unauthorized | "누구세요?" — 유효한 인증 자격 증명이 없어 **사용자를 식별할 수 없음** |
| **403** | Forbidden (Permission Denied) | "권한이 없어요" — 누구인지는 알지만, **해당 요청을 처리할 권한이 없음** |

- 401: 로그인이 안 되어 있거나 토큰이 없는 상태
- 403: 로그인은 되어 있지만 해당 리소스에 대한 권한이 없는 상태 (서버는 클라이언트가 누구인지 알고 있음)

---

## 4. 인증 정책 설정

DRF는 인증 방식을 **두 가지 범위**로 설정할 수 있다.

### ① 전역 설정 (settings.py)

프로젝트 전체에 적용되는 기본 인증 방식을 `DEFAULT_AUTHENTICATION_CLASSES`로 정의한다.

```python
# my_api/settings.py

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
    ],
}
```

- 기본값(미설정 시): `SessionAuthentication` + `BasicAuthentication`
- 인증 방식은 **하나로 통일**하는 것이 일반적

### ② View 함수별 설정 (데코레이터)

`@authentication_classes` 데코레이터를 사용해 특정 view에만 재정의한다.

```python
from rest_framework.decorators import authentication_classes
from rest_framework.authentication import TokenAuthentication, BasicAuthentication

@api_view(['GET', 'POST'])
@authentication_classes([TokenAuthentication, BasicAuthentication])
def article_list(request):
    pass
```

> **데코레이터**란 기존 함수를 감싸서 특별한 기능을 추가하는 함수다.

---

## 5. DRF가 제공하는 인증 체계

| 방식 | 설명 |
|------|------|
| **BasicAuthentication** | 요청마다 `아이디:비밀번호`를 Base64로 인코딩하여 `Authorization` 헤더에 담아 전송. 비밀번호 노출 위험 있음 |
| **TokenAuthentication** ✅ | 로그인 시 발급받은 고유 토큰을 `Authorization` 헤더에 담아 전송. 기본 데스크톱/모바일 클라이언트에 적합 |
| **SessionAuthentication** | 장고의 기본 세션 시스템 활용. 브라우저가 보내는 `sessionid` 쿠키로 인증 |
| **RemoteUserAuthentication** | 외부 시스템이 이미 처리한 인증 결과를 신뢰하여 인증. (예: SSAFY 계정 → LAB 계정 접근 가능한 것이 이 방식) |

> 카카오/네이버 소셜 로그인은 **OAuth** 방식으로, RemoteUserAuthentication과는 다른 개념이다.

---

## 6. Token 인증 설정

`TokenAuthentication` 적용을 위해 세 단계가 필요하다.

### ① 인증 클래스 설정

```python
# my_api/settings.py

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
    ],
}
```

### ② INSTALLED_APPS에 authtoken 추가

```python
# my_api/settings.py

INSTALLED_APPS = [
    'articles',
    'accounts',
    'rest_framework',
    'rest_framework.authtoken',  # 토큰 앱 추가
    ...
]
```

### ③ Migrate 진행

```bash
$ python manage.py migrate
```

migrate 후 `authtoken_token` 테이블이 생성되며, 여기에 발급된 토큰이 저장된다.

---

## 7. Dj-Rest-Auth 라이브러리

### 개요

회원가입, 로그인/로그아웃, 비밀번호 재설정 등 **다양한 인증 관련 기능을 RESTful API 엔드포인트로 제공**하는 라이브러리.

- `django.contrib.auth`를 대체하는 게 아니라 그 위에서 기능을 **확장**한다.
- 직접 구현 없이 설치만으로 인증 API를 즉시 사용 가능.

### 설치 및 적용

**① 설치**

```bash
$ pip install dj-rest-auth
```

**② INSTALLED_APPS 설정**

```python
# my_api/settings.py

INSTALLED_APPS = [
    'articles',
    'accounts',
    'rest_framework',
    'rest_framework.authtoken',
    'dj_rest_auth',     # 추가
    ...
]
```

**③ URL 등록**

```python
# my_api/urls.py

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/v1/', include('articles.urls')),
    path('accounts/', include('dj_rest_auth.urls')),  # 로그인/로그아웃 등 URL 자동 등록
    # path('accounts/signup/', include('dj_rest_auth.registration.urls')),  # 회원가입 (별도 설정)
]
```

### Registration(회원가입) 기능 추가 설정

회원가입 기능은 별도 패키지가 필요하며, 실제 회원 로직을 담당한다.

```bash
$ pip install 'dj-rest-auth[with-social]'
```

```python
# my_api/settings.py

INSTALLED_APPS = [
    ...
    'django.contrib.sites',
    'allauth',
    'allauth.account',
    'allauth.socialaccount',
    'dj_rest_auth.registration',
    ...
]

SITE_ID = 1  # DB의 Site 테이블에서 ID=1에 매치되는 주소 사용
```

```python
# settings.py MIDDLEWARE에 추가
MIDDLEWARE = [
    ...
    'allauth.account.middleware.AccountMiddleware',
]
```

```python
# my_api/urls.py

urlpatterns = [
    ...
    path('accounts/', include('dj_rest_auth.urls')),
    path('accounts/signup/', include('dj_rest_auth.registration.urls')),  # 회원가입 URL
]
```

```bash
$ python manage.py migrate
```

> `django.contrib.sites`를 `INSTALLED_APPS`에 추가하면 `django_site` 테이블이 자동으로 생성된다.

### 라이브러리 설치 후 자동 추가되는 URL

`http://127.0.0.1:8000/accounts/`에서 아래 URL들이 자동으로 등록된 것을 확인할 수 있다.

```
accounts/ password/reset/
accounts/ password/reset/confirm/
accounts/ login/
accounts/ logout/
accounts/ user/
accounts/ password/change/
accounts/ signup/
```

---

## 8. Token 발급 및 활용

### Token 발급 과정

**회원가입** (`POST /accounts/signup/`)

`http://127.0.0.1:8000/accounts/signup/`에서 username, email, password1, password2를 입력하면 회원가입과 동시에 토큰이 발급된다.

**로그인** (`POST /accounts/login/`)

`http://127.0.0.1:8000/accounts/login/`에서 로그인하면 응답 JSON에 `"key"` 값으로 토큰이 반환된다.

```json
{
    "key": "fb0684027092db0c8a98a76b020ae2d0cb137203"
}
```

> 이 토큰을 Vue에서 별도로 저장하여 매 요청마다 함께 보내야 한다.

### Token으로 인증 받는 방법 (클라이언트 측)

1. **`Authorization` HTTP Header에 포함**
2. 키 앞에 반드시 문자열 **`Token`** 을 붙이고, **공백(whitespace)** 으로 두 문자열을 구분

```
Authorization: Token 9944b09199c62bcf9418ad846dd0e4bbdfc6ee4b
```

> `Token`의 **T는 반드시 대문자**여야 한다.

### Postman으로 확인

**Token 없이 POST 요청 시** → 에러 발생 (`cannot assign anonymous user...`)

**Headers에 Token 추가 후 요청 시** → 성공

| Key | Value |
|-----|-------|
| Authorization | Token fb0684027092db0c8a98a76b020ae2d0cb137203 |

### Token 데이터 확인

발급된 토큰은 DB의 `authtoken_token` 테이블에 저장된다.

```sql
SELECT * FROM authtoken_token LIMIT 100;
```

- `key` (varchar 40): 토큰 값
- `created` (datetime): 생성 시각
- `user` (bigint): 연결된 사용자 ID

> 발급받은 토큰은 인증이 필요한 **모든 요청마다 함께 보내야 한다.** 토큰은 로그인할 때마다 새로 발급된다.

---

## 💡 한 줄 요약

> DRF에서 인증은 view 실행 전 자격증명을 식별하는 과정이며, 로그인 시 발급받은 Token을 `Authorization: Token <key>` 형식의 헤더에 담아 매 요청마다 전송하는 방식으로 구현한다.

---

## ❓ 더 찾아볼 것

- JWT(JSON Web Token)와 TokenAuthentication의 차이점
- `dj-rest-auth`의 `allauth` 소셜 로그인 적용 방법
- DRF의 `Custom Authentication` 구현 방법
- `SessionAuthentication`과 CSRF 토큰의 관계
