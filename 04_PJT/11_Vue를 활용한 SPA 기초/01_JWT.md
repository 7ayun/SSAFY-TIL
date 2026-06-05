# [관통 PJT] JWT

---

## 1. JWT(JSON Web Token) 개념

### JWT란?
- **JSON Web Token**의 약자 — JSON 형태로 된 웹용 토큰
- 유저가 스스로 누군지 증명하는 **디지털 출입증**
- 서버가 유저에게 발급해주는 긴 문자열로, 그 안에 유저의 정보가 인코딩되어 있음
- 실무에서 서비스 인증의 80% 이상이 이 방식을 사용

### 기존 쿠키/세션 방식의 한계
기존에는 로그인 시 서버가 **세션 ID를 생성해 DB에 저장**하고, 사용자는 쿠키에 저장하여 매 요청마다 헤더에 담아 보내는 방식이었다.

이 방식의 한계:

- **서버 부담 증가**: 100만 명 접속 시 100만 개의 세션 ID를 DB에 저장/조회해야 함
- **확장 어려움**: 서버를 여러 대 늘리면(Scale-out) 서버 간 세션 정보가 분산되어 관리 어려움
- **레디스 도입 필요**: 세션 중앙 관리를 위해 Redis 같은 인메모리 DB가 필요하나, 이 또한 단일 장애점(SPOF) 문제 발생

### JWT가 이를 극복하는 방식
- 서버가 세션을 저장하지 않음 **(Stateless)**
- 토큰 자체에 유저 정보가 인코딩되어 있어, **DB 조회 없이 토큰만으로 검증** 가능
- 로컬 CPU 연산만으로 빠르게 처리 가능 → 속도 빠름, 인프라 단순

---

## 2. JWT 구조

JWT는 `.`으로 구분된 3부분으로 구성된다.

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIn0
.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

| 파트 | 설명 |
|------|------|
| **Header** | 어떤 알고리즘(예: HS256)으로 암호화했는지 명시 — 봉투 역할 |
| **Payload** | 실제 유저 정보 (user_id, token_type, exp 등) — 누구나 열어볼 수 있음 |
| **Signature** | 헤더+페이로드를 서버 비밀키로 해싱한 값 — 위조 방지 도장 |

> **인코딩(Base64) ≠ 암호화**: JWT는 누구나 디코딩 가능하므로, Payload에 비밀번호 등 민감 정보를 절대 저장하면 안 된다.

**서명 검증 방식**: 서버는 사용자가 보낸 Header+Payload와 서버가 보유한 비밀키를 합쳐 해싱한 값이, Signature와 일치하는지 비교하여 위조 여부를 판단한다.

> [jwt.io](https://jwt.io)에서 토큰을 붙여넣으면 내용 확인 및 시크릿 키로 서명 검증 가능

---

## 3. JWT 동작 흐름

```
1. 클라이언트 → 서버: 로그인 요청
2. 서버: 사용자 정보 검증 후 JWT 토큰 발급
3. 클라이언트: JWT를 브라우저(로컬스토리지 등)에 저장
4. 클라이언트 → 서버: JWT를 Authorization 헤더에 담아 API 요청
   Authorization: Bearer <access_token>
5. 서버: 별도의 DB 조회 없이 JWT에 포함된 서명을 검증 후 응답
```

---

## 4. JWT vs Token(쿠키/세션) 방식 비교

| 항목 | Token(쿠키/세션) 방식 | JWT 방식 |
|------|----------------------|----------|
| 키 안의 정보 | 없음 (서버 DB에 저장) | 유저 정보 포함 |
| 서버 부담 | 높음 (DB 조회 필요) | 낮음 (토큰 자체 검증) |
| 확장성(Scale-out) | 어려움 | 용이 |
| 키 유출 대응 | 용이 (서버에서 세션 비활성화) | 어려움 |
| 주요 사용처 | 금융권, 보안 엄격한 서비스 | SNS, 쇼핑몰, 대규모 트래픽 서비스 |

> **트레이드오프**: JWT는 만능이 아니다. 개발에서 모든 기술은 트레이드오프 관계에 있다. 보안이 중요한 금융권에서는 쿠키/세션 방식이 적합하고, 대규모 트래픽 처리가 필요한 서비스에는 JWT가 적합하다. 포트폴리오에서 "왜 JWT를 썼는가"라는 면접 질문에 이 트레이드오프를 명확히 설명할 수 있어야 한다.

---

## 5. JWT 실습 — Django 설정

### 패키지 설치

```bash
pip install djangorestframework-simplejwt
```

### settings.py 설정

**① 인증 방식 변경** — 기존 `TokenAuthentication`을 주석 처리하고 JWT 방식으로 변경

```python
REST_FRAMEWORK = {
    # Authentication
    'DEFAULT_AUTHENTICATION_CLASSES': [
        # 'rest_framework.authentication.TokenAuthentication',
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
    # permission
    ...
}
```

**② INSTALLED_APPS에 등록**

```python
INSTALLED_APPS = [
    'articles',
    ...
    'allauth.socialaccount',
    'dj_rest_auth.registration',
    'rest_framework_simplejwt',  # 추가
    ...
]
```

**③ REST_AUTH 설정** — `USE_JWT: True`로 dj-rest-auth의 로그인 URL에서 JWT 방식 사용

```python
REST_AUTH = {
    'REGISTER_SERIALIZER': 'accounts.serializers.CustomRegisterSerializer',
    'USE_JWT': True,
}
```

---

## 6. Vue에서 로그인 및 API 요청 수정

### 로그인 응답 처리 — `stores/accounts.js`

JWT 방식에서는 응답 데이터에 `access`와 `user`가 담겨 온다.

```javascript
const logIn = function ({ username, password }) {
  axios({
    method: 'post',
    url: `${API_URL}/accounts/login/`,
    data: { username, password },
  })
    .then(res => {
      console.log('로그인이 완료되었습니다.')
      token.value = res.data.access   // access token 저장
      user.value = res.data.user      // 로그인 user 정보 저장
      router.push({ name: 'ArticleView' })
    })
    .catch(err => console.log(err))
}
```

### 게시글 조회 — `stores/articles.js`

`Token` → `Bearer`로 변경 (JWT 표준 방식)

```javascript
const getArticles = function () {
  axios({
    method: 'get',
    url: `${API_URL}/api/v1/articles/`,
    headers: {
      // 'Authorization': `Token ${accountStore.token}`  // 기존 방식
      'Authorization': `Bearer ${accountStore.token}`   // JWT 방식
    }
  })
    .then(res => { articles.value = res.data })
    .catch(err => console.log(err))
}
```

### 게시글 생성 — `views/CreateView.vue`

```javascript
const createArticle = function () {
  axios({
    method: 'post',
    url: `${store.API_URL}/api/v1/articles/`,
    data: { title: title.value, content: content.value },
    headers: {
      // 'Authorization': `Token ${accountStore.token}`
      'Authorization': `Bearer ${accountStore.token}`
    }
  })
    .then(res => { router.push({ name: 'ArticleView' }) })
    .catch(err => console.log(err))
}
```

> 공개 Open API들이 대부분 `Bearer` 방식을 사용하는 이유가 바로 JWT 기반으로 운영되기 때문이다.

---

## 7. Refresh Token

### Refresh Token이란?
로그인을 다시 하지 않아도 **Access Token을 새로 받을 수 있게 하는 장기 열쇠**

### 필요성
Access Token은 유출 시 대응이 어렵기 때문에 **유효기간을 짧게** 설정한다. 하지만 너무 짧으면 자주 만료되어 사용자가 불편을 겪으므로, Refresh Token으로 자동 갱신하는 방식을 함께 사용한다.

| 토큰 | 적정 유효기간 | 역할 |
|------|-------------|------|
| Access Token | 10~15분 (금융: 5~10분) | 실제 API 요청 시 사용 |
| Refresh Token | 1~2주 또는 30일 (금융: 1~14일) | Access Token 재발급 |

### Access Token 재발급 흐름

```
1. Access Token으로 API 요청
2. Access Token 만료 → 서버에서 401 에러 발생
3. Refresh Token으로 Access Token 재발급 요청
   - 재발급 성공: 새 Access Token으로 원래 요청 재시도
   - 재발급 실패(Refresh Token도 만료): 로그인 페이지로 이동
```

---

## 8. Refresh Token 실습

### ① JWT_AUTH_HTTPONLY 설정 변경

기본값 `True`이면 Refresh Token이 발급되지 않으므로 `False`로 변경한다.

```python
# settings.py
REST_AUTH = {
    'REGISTER_SERIALIZER': 'accounts.serializers.CustomRegisterSerializer',
    'USE_JWT': True,
    'JWT_AUTH_HTTPONLY': False,    # refresh token을 받기 위한 준비 (기본이 True)
}
```

### ② 테스트용 토큰 만료 기한 설정

```python
# settings.py
from datetime import timedelta

SIMPLE_JWT = {
    # 테스트를 위해 임시로 설정 (실제 프로젝트에서는 권장 시간으로 설정)
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=1),
    'REFRESH_TOKEN_LIFETIME': timedelta(minutes=2),
}
```

### ③ Refresh Token 저장 및 재발급 함수 — `stores/accounts.js`

```javascript
const token = ref(null)
const user = ref(null)
const refresh = ref(null)   // refresh token 변수 추가

const logIn = function ({ username, password }) {
  axios({ ... })
    .then(res => {
      token.value = res.data.access
      user.value = res.data.user
      refresh.value = res.data.refresh   // refresh token 저장
      router.push({ name: 'ArticleView' })
    })
}

// Access Token 재발급 함수
const refreshAccessToken = function () {
  return axios({
    method: 'post',
    url: `${API_URL}/accounts/token/refresh/`,
    data: {
      refresh: refresh.value,
    },
  })
    .then(res => {
      // console.log(res)
      token.value = res.data.access   // access 토큰 갱신
      return true
    })
    .catch(err => {
      console.log(err)
      return false
    })
}

return { token, user, refresh, logIn, logOut, signUp, refreshAccessToken }
```

### ④ 게시글 요청 시 401 에러 처리 — `stores/articles.js`

```javascript
const getArticles = function () {
  // ...
  .catch(err => {
    console.log(err)
    if (err.response?.status === 401) {
      console.log('Access Token 재발급 진행!')
      // access token 재발급은 비동기 요청 (promise 객체)
      accountStore.refreshAccessToken()
        .then(ok => {
          // 재발급에 실패한 경우 종료
          if (!ok) {
            window.alert('다시 로그인이 필요합니다.')
            accountStore.logOut()    // 기존에 저장된 token을 제거하기 위함
            router.push({ name: 'LogInView' })
            return
          }
          // 재발급에 성공한 경우 목록 재요청 진행
          axios({
            method: 'get',
            url: `${API_URL}/api/v1/articles/`,
            headers: {
              'Authorization': `Bearer ${accountStore.token}`
            },
          })
            .then(res => { articles.value = res.data })
        })
    }
  })
}
```

- `CreateView`도 동일한 로직으로 수정한다.

---

## 💡 한 줄 요약
> JWT는 서버가 세션을 저장하지 않고 토큰 자체에 유저 정보를 담아 인증하는 Stateless 방식으로, Refresh Token과 함께 사용하면 짧은 Access Token 수명으로 보안을 유지하면서도 사용자 불편을 최소화할 수 있다.

## ❓ 더 찾아볼 것
- JWT 페이로드 인코딩(Base64) vs 암호화 차이
- Redis를 활용한 세션 관리 vs JWT 실무 선택 기준
- `CSRF 공격`과 `JWT_AUTH_HTTPONLY` 설정의 관계
- `djangorestframework-simplejwt` 공식 문서 — 토큰 블랙리스트 기능
- XSS 공격과 JWT 저장 위치(로컬스토리지 vs 쿠키) 트레이드오프
