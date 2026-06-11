# [Vue] User Customize

---

## 1. User Customize란

`dj-rest-auth`를 활용한 회원가입 시, Django 기본 User 모델에 **커스텀 필드를 추가**하고
회원가입 Serializer도 이에 맞춰 수정하는 작업이다.

**왜 그냥 필드만 추가하면 안 되나?**
회원가입 로직을 직접 정의하지 않고 `dj-rest-auth` 라이브러리에 위임했기 때문에, 새롭게 추가된 필드는 기본 Serializer가 처리하지 못한다. 그래서 Serializer도 같이 커스터마이징해야 한다.

> **사전 준비**: `db.sqlite3` 삭제 + `accounts/migrations/` 파일 삭제 후 Migration 재진행

---

## 2. User Model에 필드 추가

Django 기본 User 모델을 `AbstractUser`로 상속하여 커스텀 필드(예: `age`)를 추가한다.

> **PositiveIntegerField**: 음수가 될 수 없는 숫자가 저장되는 필드

```python
# accounts/models.py
from django.db import models
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    age = models.PositiveIntegerField(null=True, blank=True)
```

Migration 진행:

```bash
$ python manage.py makemigrations
$ python manage.py migrate
```

DB `accounts_user` 테이블에 `age integer unsigned` 컬럼이 추가된 것을 확인할 수 있다.

---

## 3. Vue 회원가입 form에 age 필드 추가

`SignUpView`에 age 입력 필드와 반응형 변수를 추가하고, payload에도 포함시킨다.
숫자 입력 필드는 공백이 입력되지 않으므로 `v-model.trim` 없이 `v-model`만 써도 된다.

```html
<!-- views/SignUpView.vue -->
<label for="age">age: </label>
<input type="number" id="age" v-model="age" /> <br>
```

```js
const age = ref(null)

const signUp = function () {
  const payload = {
    username: username.value,
    password1: password1.value,
    password2: password2.value,
    age: age.value        // age 추가
  }
  accountStore.signUp(payload)
}
```

```js
// stores/accounts.js
const signUp = function (payload) {
  const { username, password1, password2 } = payload
  const age = payload.age

  axios({
    method: 'post',
    url: `${API_URL}/accounts/signup/`,
    data: { username, password1, password2, age }
  })
}
```

이 상태로 회원가입을 하면 성공은 하지만, **age 정보가 NULL로 저장**된다.
`dj-rest-auth`의 기본 `RegisterSerializer`가 `age` 필드를 모르기 때문이다.

---

## 4. RegisterSerializer의 한계 파악

`dj-rest-auth`의 `RegisterSerializer` 소스코드를 직접 확인하는 방법:
- GitHub에서 `dj-rest-auth` → `registration` → `serializers.py` 검색

```python
class RegisterSerializer(serializers.Serializer):
    username = serializers.CharField(...)
    email = serializers.EmailField(required=...)
    password1 = serializers.CharField(write_only=True)
    password2 = serializers.CharField(write_only=True)
    # age 없음!
```

`get_cleaned_data` 메서드도 `username`, `password1`, `email`만 반환하므로
저장 시에도 age가 포함되지 않는다.

---

## 5. CustomRegisterSerializer 작성

`RegisterSerializer`를 상속받아 age 필드를 추가하고, 유효성 검사와 저장 로직을 확장한다.

```python
# accounts/serializers.py
from dj_rest_auth.registration.serializers import RegisterSerializer
from rest_framework import serializers

class CustomRegisterSerializer(RegisterSerializer):
    age = serializers.IntegerField()

    def get_cleaned_data(self):
        # super()로 부모의 유효성 검사 결과를 먼저 가져온다
        data = super().get_cleaned_data()
        # age도 유효성 검사 후 데이터에 추가
        data['age'] = self.validated_data.get('age')
        return data

    def save(self, request):
        # super().save()로 기본 유저 저장 먼저 진행
        user = super().save(request)
        # 저장된 user 객체에 age 직접 할당 후 다시 저장
        user.age = self.validated_data.get('age')
        user.save()
        return user
```

| 메서드 | 역할 |
|---|---|
| `get_cleaned_data` | 유효성 검사 결과를 딕셔너리로 반환, 추가 필드 포함 |
| `save` | `super().save()`로 기본 저장 후, 추가 필드를 user 객체에 직접 저장 |

> **super() 활용**: 부모 요소에서 정의된 기능을 그대로 실행하고, 거기에 age만 덧붙이는 방식. 어려워 보이지만 별거 아닌 코드다.

---

## 6. settings.py에 커스텀 Serializer 등록

⚠️ `REST_AUTH`와 `REST_FRAMEWORK`는 완전히 다른 설정이다. 혼동하지 않도록 주의.

```python
# settings.py 최하단에 추가

# REST_FRAMEWORK = { ... }  ← 이게 아님!

REST_AUTH = {
    'REGISTER_SERIALIZER': 'accounts.serializers.CustomRegisterSerializer',
}
```

이제 회원가입 시 `age` 정보가 DB에 정상 저장된다.

---

## 💡 한 줄 요약

> `AbstractUser`로 커스텀 필드를 추가하고, `RegisterSerializer`를 상속해 `get_cleaned_data`와 `save`를 오버라이드한 `CustomRegisterSerializer`를 만들어 `settings.py`의 `REST_AUTH`에 등록하면 회원가입 시 추가 필드가 정상 저장된다.

## ❓ 더 찾아볼 것

- `AbstractUser` vs `AbstractBaseUser` 차이
- `validated_data`와 `cleaned_data`의 관계
- `dj-rest-auth` 소스코드: `RegisterSerializer.save()` 전체 흐름
- `null=True`와 `blank=True`의 차이 (DB null vs form validation)
