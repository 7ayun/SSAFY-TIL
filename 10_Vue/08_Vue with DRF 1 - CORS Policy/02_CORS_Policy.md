# [Vue] CORS Policy

---

## 1. 브라우저의 동일 출처 정책 (SOP)

웹 브라우저는 기본적으로 **같은 출처(Origin)에서 온 요청만 허용**한다. 다른 출처의 요청은 보안을 이유로 차단하는데, 이 정책을 **SOP(Same-Origin Policy, 동일 출처 정책)**라고 한다.

> **SOP가 필요한 이유 — 악의적 스크립트 시나리오**
>
> 1. 사용자 A가 `bank.com`에 로그인 → 브라우저에 로그인 쿠키 저장
> 2. 같은 브라우저로 불법 사이트 접속
> 3. 불법 사이트의 악의적 JavaScript가 `bank.com`으로 Axios 요청 전송 (브라우저에 저장된 쿠키를 포함)
> 4. 은행은 정상 요청으로 판단하고 계좌 정보 응답
>
> → SOP가 없었다면 해커가 계좌 정보를 탈취할 수 있다. 브라우저가 출처를 확인하고 다른 출처의 응답을 차단함으로써 이를 방지한다.

---

## 2. Origin(출처)란?

출처(Origin)는 URL의 **Protocol + Host + Port** 세 가지가 모두 일치해야 같은 출처로 인정한다.

```
http://localhost:3000/posts/3
│───┘  │─────────┘│───┘│────┘
Protocol  Host    Port  Path
└─────────────────────┘
       이 세 가지가 Origin
```

기준 URL `http://localhost:3000/articles/3/`와의 비교:

| URL | 결과 | 이유 |
|-----|------|------|
| `http://localhost:3000/articles/` | 성공 | Path만 다름 |
| `http://localhost:3000/comments/3/` | 성공 | Path만 다름 |
| `https://localhost:3000/articles/3/` | 실패 | Protocol 다름 |
| `http://localhost:80/articles/3/` | 실패 | Port 다름 |
| `http://yahuua:3000/articles/3/` | 실패 | Host 다름 |

> Vue는 포트 5173, DRF는 포트 8000을 사용하므로 **Port가 달라 다른 출처**로 분류된다. 포스트맨이나 단순 파이썬 스크립트에서는 CORS 에러가 없는 이유는 **브라우저가 아니기 때문**이다. CORS는 브라우저의 보안 정책이다.

---

## 3. CORS(Cross-Origin Resource Sharing)

현대 웹은 CDN, 외부 API, 프론트엔드-백엔드 분리 등으로 인해 다른 출처의 리소스를 빈번하게 요청한다. 이를 허용하기 위한 메커니즘이 **CORS(교차 출처 리소스 공유)**다.

**작동 원리**: 서버가 HTTP 응답 헤더에 `Access-Control-Allow-Origin: 허용할 출처`를 포함시키면, 브라우저는 해당 요청을 허용한다.

```
Browser (도메인 A)  ──── 요청 ────>  Server (도메인 B)
                   <─── 응답 ──────
                         + "Access-Control-Allow-Origin: 도메인 A" 헤더 포함
                         
브라우저: "서버가 허용한다고 했으니 OK"
```

핵심 포인트: **CORS 설정은 클라이언트(Vue)가 아닌 서버(DRF)에서 한다.**

---

## 4. Django에서 CORS Headers 설정

Django에서는 `django-cors-headers` 라이브러리로 손쉽게 CORS Header를 응답에 추가할 수 있다.

### 설치

```bash
$ pip install django-cors-headers
# (스켈레톤 코드에는 requirements.txt로 이미 설치되어 있음)
```

### settings.py 설정 — 주석 해제

```python
# settings.py

INSTALLED_APPS = [
    ...
    'corsheaders',       # 주석 해제
    ...
]

MIDDLEWARE = [
    ...
    'corsheaders.middleware.CorsMiddleware',    # 주석 해제 (CommonMiddleware 위에 위치)
    'django.middleware.common.CommonMiddleware',
    ...
]

# 허용할 Vue 프로젝트의 출처 등록 — 주석 해제
CORS_ALLOWED_ORIGINS = [
    'http://127.0.0.1:5173',
    'http://localhost:5173',
]
```

> **미들웨어 순서 주의**: `CorsMiddleware`는 반드시 `CommonMiddleware`보다 앞에 위치해야 한다.

### 설정 후 동작 확인

DRF 서버 재실행 후, 브라우저 개발자도구 → Network 탭 → Fetch/XHR에서 응답 헤더를 확인한다.

```
Response Headers:
  Access-Control-Allow-Origin: http://localhost:5173
```

이 헤더가 포함되었다면 브라우저가 요청을 허용하고, Vue 콘솔에서 DRF 데이터를 정상적으로 확인할 수 있다.

---

## 5. store에 응답 데이터 저장 및 화면 출력

CORS 설정 완료 후, 콘솔에서 DRF 응답 데이터가 확인되면 `articles.value`에 저장한다.

```javascript
// store/articles.js
const getArticles = function () {
  axios({
    method: 'get',
    url: `${API_URL}/api/v1/articles/`
  })
    .then(res => {
      articles.value = res.data    // 응답 데이터를 store에 저장
    })
    .catch(err => console.log(err))
}
```

`pinia-plugin-persistedstate` 덕분에 `articles` 데이터가 브라우저의 **localStorage에도 자동 저장**된다. 개발자도구 → Application → Local Storage에서 확인 가능하다.

---

## 💡 한 줄 요약
> CORS는 브라우저의 SOP를 안전하게 우회하는 메커니즘으로, DRF 서버의 settings.py에서 django-cors-headers로 허용 출처를 등록하면 해결된다.

## ❓ 더 찾아볼 것
- CORS Preflight 요청 (OPTIONS 메서드)
- `CORS_ALLOW_ALL_ORIGINS = True` — 개발 편의용, 프로덕션 환경에서는 위험
- `CORS_ALLOW_HEADERS`, `CORS_ALLOW_METHODS` 세부 설정
- 참고: https://developer.mozilla.org/ko/docs/Web/HTTP/CORS
