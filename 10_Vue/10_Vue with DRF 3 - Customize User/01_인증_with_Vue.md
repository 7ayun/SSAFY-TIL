# [Vue] 인증 with Vue

---

## 1. 배경: 왜 인증이 필요한가

DRF에 권한 설정을 추가하자 게시글 조회가 **401 Unauthorized** 오류와 함께 막혔다.
Vue에서 로그인을 요청해 DRF로부터 토큰을 받은 뒤, 그 토큰을 Pinia store에 저장하고
모든 인증이 필요한 요청마다 토큰을 함께 보내면 문제가 해결된다.

> 오늘의 목표: 회원가입 → 로그인 → 토큰 저장 → 토큰을 헤더에 담아 요청 → 인증 여부에 따른 페이지 접근 제어

---

## 2. 사전 준비

- `db.sqlite3` 삭제 후 Migration 재진행
- 관리자 계정 생성 후 게시글 1개 이상 작성
  - 기존 fixtures 데이터는 user 정보가 없으므로 사용 불가

---

## 3. accounts.js 스토어 별도 생성

기존에는 `stores/articles.js` 하나만 존재했다.
유저 정보와 게시글 정보는 **성격이 다르기 때문에** `stores/accounts.js`를 새로 만들어 분리해서 관리한다.

```js
// stores/accounts.js
import { defineStore } from 'pinia'
import { ref } from 'vue'
import axios from 'axios'

export const useAccountStore = defineStore('account', () => {
  const API_URL = 'http://127.0.0.1:8000'
  const token = ref(null)

  // ... (함수들)

  return { token }
}, { persist: true })
```

> `persist: true` 옵션 덕분에 토큰이 브라우저 **localStorage**에 자동 저장되어 새로고침 후에도 유지된다.

---

## 4. 회원가입 구현

### 라우터 등록 및 네비게이션 링크

```js
// router/index.js
import SignUpView from '@/views/SignUpView.vue'

const router = createRouter({
  routes: [
    {
      path: '/signup',
      name: 'SignUpView',
      component: SignUpView
    }
  ]
})
```

```html
<!-- App.vue -->
<RouterLink :to="{ name: 'SignUpView'}">SignUpPage</RouterLink>
```

### SignUpView 컴포넌트 form + v-model 변수

```html
<!-- views/SignUpView.vue -->
<template>
  <div>
    <h1>Sign Up Page</h1>
    <form @submit.prevent="signUp">
      <label for="username">username: </label>
      <input type="text" id="username" v-model.trim="username" /> <br>
      <label for="password1">password: </label>
      <input type="password" id="password1" v-model.trim="password1" /> <br>
      <label for="password2">password confirmation: </label>
      <input type="password" id="password2" v-model.trim="password2" /> <br>
      <input type="submit" value="signup">
    </form>
  </div>
</template>
```

```js
import { ref } from 'vue'
const username = ref(null)
const password1 = ref(null)
const password2 = ref(null)
```

> **v-model.trim**: 앞뒤 불필요한 공백을 자동 제거한다.

### signUp 함수 작성 — payload 개념

**payload**: 전달할 데이터들을 통칭하는 변수명. 가독성을 위해 Django의 `context` 변수처럼 묶어서 전달한다.

```js
// views/SignUpView.vue
import { useAccountStore } from '@/stores/accounts'
const accountStore = useAccountStore()

const signUp = function () {
  const payload = {
    username: username.value,   // ref이므로 반드시 .value로 실제 값 접근
    password1: password1.value,
    password2: password2.value,
  }
  accountStore.signUp(payload)
}
```

> ref는 객체이므로 그대로 전달하면 오브젝트가 넘어간다. 반드시 `.value`로 실제 값을 꺼내야 한다.

### store의 signUp 함수

```js
// stores/accounts.js
const signUp = function (payload) {
  const { username, password1, password2 } = payload  // 구조분해 할당

  axios({
    method: 'post',
    url: `${API_URL}/accounts/signup/`,
    data: { username, password1, password2 }
  })
    .then(res => {
      console.log('회원가입이 완료되었습니다.')
    })
    .catch(err => console.log(err))
}
return { signUp }
```

---

## 5. 로그인 구현

### 라우터 및 LogInView

```js
// router/index.js
{ path: '/login', name: 'LogInView', component: LogInView }
```

```html
<!-- App.vue -->
<RouterLink :to="{ name:'LogInView' }">LogInPage</RouterLink>
```

로그인 form은 회원가입과 달리 `password` 하나만 사용한다.
(Django는 `username`과 `password`로 받기 때문에 `password1`로 보내면 로그인이 안 된다.)

```js
// views/LogInView.vue
const username = ref(null)
const password = ref(null)  // password1이 아니라 password

const logIn = function () {
  const payload = { username: username.value, password: password.value }
  accountStore.logIn(payload)
}
```

### store의 logIn 함수 + 토큰 저장 + 로그인 후 이동

```js
// stores/accounts.js
import { useRouter } from 'vue-router'

export const useAccountStore = defineStore('account', () => {
  const router = useRouter()
  const token = ref(null)

  const logIn = function (payload) {
    const { username, password } = payload

    axios({
      method: 'post',
      url: `${API_URL}/accounts/login/`,
      data: { username, password }
    })
      .then(res => {
        token.value = res.data.key        // 응답의 key 값이 토큰
        router.push({ name: 'ArticleView' })  // 로그인 성공 후 메인으로 이동
      })
      .catch(err => console.log(err))
  }

  return { signUp, logIn, token }
}, { persist: true })
```

---

## 6. 토큰을 헤더에 담아 요청하기

로그인 응답에 `{ key: 'abc123...' }` 형태로 토큰이 온다.
이제 이 토큰을 인증이 필요한 모든 요청의 `Authorization` 헤더에 포함해서 보내면 401이 해결된다.

```js
// stores/articles.js
import { useAccountStore } from './accounts'

export const useArticleStore = defineStore('article', () => {
  const accountStore = useAccountStore()

  const getArticles = function () {
    axios({
      method: 'get',
      url: `${API_URL}/api/v1/articles/`,
      headers: {
        'Authorization': `Token ${accountStore.token}`
      }
    })
  }

  const createArticle = function () {
    axios({
      method: 'post',
      url: `${API_URL}/api/v1/articles/`,
      data: { /* ... */ },
      headers: {
        'Authorization': `Token ${accountStore.token}`
      }
    })
  }
})
```

> axios의 `headers`는 객체로 전달한다 (axios 공식 문서 Request Config 참고).

---

## 7. 인증 여부 확인 및 Navigation Guard

### isLogin — computed를 사용하는 이유

token 유무로 로그인 여부를 판별한다. 일반 함수로 매번 호출하기보다 **토큰 값이 바뀔 때만 다시 계산**하도록 `computed`를 사용한다.

```js
// stores/accounts.js
import { computed } from 'vue'

const isLogin = computed(() => {
  return token.value ? true : false
})

return { signUp, logIn, token, isLogin }
```

### Navigation Guard (`router.beforeEach`)

> **beforeEach**: 모든 페이지 이동 직전에 실행되는 전역 네비게이션 가드

⚠️ `useAccountStore()`는 반드시 **beforeEach 콜백 내부**에서 호출해야 정상 동작한다. 외부에서 호출하면 에러 발생.

```js
// router/index.js
import { useAccountStore } from '@/stores/accounts'

router.beforeEach((to, from) => {
  const accountStore = useAccountStore()  // 반드시 내부에서 호출

  // 1. 미인증 사용자: 메인 페이지 접근 차단
  if (to.name === 'ArticleView' && !accountStore.isLogin) {
    window.alert('로그인이 필요합니다.')
    return { name: 'LogInView' }  // router.push 대신 return으로 이동
  }

  // 2. 인증된 사용자: 회원가입/로그인 페이지 접근 차단
  if ((to.name === 'SignUpView' || to.name === 'LogInView') && (accountStore.isLogin)) {
    window.alert('이미 로그인 되어 있습니다.')
    return { name: 'ArticleView' }
  }
})
```

| 상황 | 조건 | 동작 |
|---|---|---|
| 미인증 사용자가 메인 접근 | `!accountStore.isLogin` | 로그인 페이지로 리다이렉트 |
| 인증된 사용자가 회원가입/로그인 접근 | `accountStore.isLogin` | 메인 페이지로 리다이렉트 |
| 그 외 | — | 목적지로 정상 이동 |

> return 값이 없으면 가고자 하는 곳으로 보내준다.

---

## 8. 브라우저 인스턴스와 토큰 분리

Vue 서버는 `index.html`을 제공하고, 각각의 브라우저가 그 HTML을 받아 Vue 인스턴스를 새로 생성한다. 따라서 **브라우저마다 각자의 토큰이 별도로 저장**된다. DRF 서버는 여러 브라우저(여러 사용자)로부터 요청을 받는 것이 맞다.

---

## 💡 한 줄 요약

> Vue에서 DRF 인증 토큰을 Pinia store에 저장하고, 모든 요청에 `Authorization: Token ...` 헤더를 담아 보내면 401 오류가 해결되며, Navigation Guard로 페이지 접근까지 제어할 수 있다.

## ❓ 더 찾아볼 것

- `pinia-plugin-persistedstate` 동작 원리와 localStorage 저장 구조
- JWT(JSON Web Token)와 Knox Token의 차이점
- Vue Router Navigation Guard의 종류 (`beforeEach`, `beforeEnter`, `beforeRouteEnter`)
- axios 인터셉터(interceptor)로 헤더를 전역으로 설정하는 방법
- v-if로 isLogin 여부에 따라 로그인/로그아웃 네비게이션 링크 조건부 표시
