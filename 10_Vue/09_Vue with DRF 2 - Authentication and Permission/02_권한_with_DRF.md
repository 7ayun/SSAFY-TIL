# [Vue] 권한 with DRF

---

## 1. 권한(Permission)의 개념

**인증(Authentication)** 이 "당신이 누구인지"를 확인하는 과정이라면, **권한(Permission)** 은 "당신이 이 요청을 수행할 자격이 있는지"를 결정하는 과정이다.

```
인증 완료 → 권한 확인 → 요청 허용 or 거부
```

- 인증 완료 후, 해당 자격 증명을 사용하여 권한 및 제한 정책을 확인하고 요청 허용 여부를 결정한다.
- `request.user`가 존재하고 인증된 상태인지를 확인한다 (`request.user.is_authenticated`와 동일한 개념).

---

## 2. 권한 정책 설정

인증 정책과 마찬가지로 **두 가지 방식**으로 설정할 수 있다.

### ① 전역 설정 (settings.py)

프로젝트 전체에 적용되는 기본 권한 방식을 `DEFAULT_PERMISSION_CLASSES`로 정의한다.

```python
# my_api/settings.py

REST_FRAMEWORK = {
    # Authentication
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
    ],
    # Permission
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.AllowAny',   # 기본값: 모두 허용
        # 'rest_framework.permissions.IsAuthenticated',  # 인증된 사용자만
    ],
}
```

> **기본값**: `DEFAULT_PERMISSION_CLASSES`를 명시하지 않으면 `AllowAny` (모두 허용)가 적용된다.

### ② View 함수별 설정 (데코레이터)

`@permission_classes` 데코레이터를 사용해 특정 view에만 다른 권한을 적용한다.

```python
from rest_framework.decorators import permission_classes
from rest_framework.permissions import IsAuthenticated

@api_view(['GET', 'POST'])
@permission_classes([IsAuthenticated])
def article_list(request):
    pass
```

---

## 3. DRF가 제공하는 주요 권한 클래스

| 권한 클래스 | 개념 | 특징 |
|-------------|------|------|
| **IsAuthenticated** | 인증된(로그인한) 사용자만 접근 허용 | `request.user`가 존재하고 인증된 상태 확인. 비인증 요청은 모두 거부 |
| **IsAdminUser** | `is_staff=True`인 관리자만 접근 허용 | `request.user.is_staff` 값이 True인지 확인. 일반 사용자와 비인증 사용자 거부 |
| **IsAuthenticatedOrReadOnly** | 비인증 사용자는 읽기만, 인증 사용자는 모든 요청 허용 | GET, HEAD, OPTIONS는 무조건 허용. POST, PUT, DELETE 등은 인증 확인 |
| **AllowAny** | 모든 요청 무조건 허용 | 권한 검사 자체를 수행하지 않음. 완전 공개 API에 사용 |

### 권한 클래스 활용 예시

- **IsAuthenticated**: 회원 전용 페이지, 결제, 프로필 수정 등 로그인 필수 기능
- **IsAdminUser**: 회원 목록 조회, 데이터 통계 등 관리자에게만 노출되어야 하는 민감한 API
- **IsAuthenticatedOrReadOnly**: 게시글 목록처럼 누구나 볼 수 있지만 작성/수정은 회원만 가능한 경우
- **AllowAny**: 회원가입, 로그인, 공개 게시글 조회 등

---

## 4. IsAuthenticated 설정 실습

### 전역을 AllowAny, 특정 View만 IsAuthenticated 적용

```python
# articles/views.py

from rest_framework.permissions import IsAuthenticated

@api_view(['GET', 'POST'])
@permission_classes([IsAuthenticated])  # 이 view는 인증된 사용자만 접근 가능
def article_list(request):
    ...
```

settings.py에서 `DEFAULT_PERMISSION_CLASSES`는 주석 처리(AllowAny)로 두고, View 함수별로 IsAuthenticated를 설정한다. 전역으로 설정하면 하나의 권한만 적용되지만, View별 설정을 통해 각 기능마다 다른 권한을 부여할 수 있다.

### 403 / 401 응답 확인

**IsAdminUser로 임시 변경 후 테스트:**

```python
# articles/views.py

from rest_framework.permissions import IsAuthenticated, IsAdminUser

@api_view(['GET', 'POST'])
@permission_classes([IsAuthenticated])  # 일반 사용자 Token으로 요청
def article_list(request):
    pass
```

Postman에서 일반 사용자 Token을 가지고 `GET /api/v1/articles/` 요청 시:

- Token 있음 + 권한 없음 → **403 Forbidden** (`"You do not have permission to perform this action."`)
- Token 없음 (비인증) → **401 Unauthorized** (`"Authentication credentials were not provided."`)

### IsAuthenticated로 복구 후 Vue 연동 영향 확인

`@permission_classes([IsAuthenticated])` 적용 후, Vue 화면에서 게시글 조회 요청 시:

```
Failed to load resource: the server responded with a status of 401 (Unauthorized)
```

게시글 조회 요청 시 토큰을 함께 보내지 않기 때문에 401이 발생한다. 이후 Vue에서 Pinia에 토큰을 저장하고, 요청마다 토큰을 Authorization 헤더에 포함시키는 방식으로 해결한다.

---

## 5. 핵심 키워드 정리

| 개념 | 설명 | 예시 |
|------|------|------|
| 인증 (Authentication) | 요청 사용자의 자격 증명을 식별 | 401: 인증 실패 (자격 증명 없음) |
| 권한 (Permission) | 인증된 사용자의 요청 허용/거부 결정 | 403: 권한 없음 (접근 거부) |
| 토큰 인증 | 발급된 토큰으로 사용자를 인증 | `rest_framework.authtoken` |
| dj-rest-auth | DRF 인증 관련 기능 제공 라이브러리 | 로그인, 회원가입 API 엔드포인트 제공 |
| Authorization 헤더 | 인증 토큰을 담아 서버에 전송 | `Authorization: Token <key>` |
| @permission_classes | View 함수에 특정 권한을 설정 | `@permission_classes([IsAuthenticated])` |
| IsAuthenticated | 인증된 사용자만 접근을 허용 | 비인증 사용자 요청은 401 반환 |

---

## 💡 한 줄 요약

> 권한(Permission)은 인증 완료 후 해당 요청의 허용 여부를 결정하며, `@permission_classes` 데코레이터로 View 함수마다 세밀하게 제어할 수 있다.

---

## ❓ 더 찾아볼 것

- `IsAuthenticatedOrReadOnly`를 게시글 API에 적용하는 방법
- DRF의 `Custom Permission` 구현 방법 (특정 사용자만 자신의 글 수정 가능하게 하기)
- `DjangoModelPermissions` — Django의 모델 권한 시스템과 연동
- Vue(Pinia)에서 Token을 저장하고 axios 요청에 Authorization 헤더를 자동으로 포함하는 방법
