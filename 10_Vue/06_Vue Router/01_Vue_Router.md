# [Vue] Vue Router

---

## 1. Routing이란

**Routing**은 네트워크에서 경로를 선택하는 프로세스로, 사용자가 접속한 URL 주소에 따라 적절한 페이지(컴포넌트)를 보여주는 기능이다.

장고에서 `urls.py`가 URL과 View 함수를 연결했던 것처럼, Vue에서는 클라이언트 측에서 라우팅을 처리한다.

### SSR vs CSR에서의 Routing

| 구분 | 방식 | 특징 |
|------|------|------|
| **SSR** (Django) | 서버 측에서 수행 | URL 요청 → 서버가 HTML 완성 후 전송, 링크 클릭 시 전체 페이지 다시 로드 |
| **CSR** (Vue) | 클라이언트(브라우저) 측에서 수행 | 최초 요청 시 HTML 하나만 받고, 이후 Ajax로 JSON 데이터만 요청 |

### SPA에서 Routing이 없다면 발생하는 문제

- 유저가 URL을 통해 페이지 변화를 감지할 수 없음
- 현재 무엇을 렌더링 중인지 상태를 알 수 없음
  - URL이 1개이므로 새로 고침 시 처음 페이지로 돌아감
  - 링크 공유 시 첫 페이지만 공유 가능
- 브라우저 **뒤로 가기** 기능 사용 불가

> 페이지는 1개이지만, **주소에 따라 여러 컴포넌트를 새로 렌더링**하여 마치 여러 페이지를 사용하는 것처럼 보이도록 해야 함

---

## 2. Vue Router 개요

**Vue Router**는 Vue.js의 공식 라우팅 라이브러리(The official Router for Vue.js)다.

Vue로 만든 SPA에서 페이지 이동 기능을 구현할 때 사용하며, 두 가지 핵심 컴포넌트를 제공한다.

| 태그 | 역할 |
|------|------|
| `<RouterLink>` | 페이지를 다시 로드하지 않고 URL을 변경하는 링크. HTML의 `<a>` 태그를 렌더링 |
| `<RouterView>` | 현재 URL에 매칭된 컴포넌트가 렌더링될 위치 |

강사: "라우터 링크는 정말로 링크예요. A 태그에 대응되는 친구고요. 라우터 뷰는 URL에 해당하는 컴포넌트를 요 위치에 보여주겠다는 거예요."

### 설치 (Vite 프로젝트 생성 시)

```bash
npm create vue@latest
# 프로젝트 생성 옵션 중 Router (SPA development) 선택 (스페이스바로 체크)
```

```bash
cd [프로젝트명]
npm install
npm run dev
```

### Vue 프로젝트 구조 변화

Router를 추가하면 `src` 폴더 내에 두 개의 폴더가 새로 생성된다.

```
src/
├── assets/
├── components/       # 화면을 구성하는 조각(재사용 컴포넌트)
├── router/
│   └── index.js      # 라우팅 설정 파일 (Django의 urls.py 역할)
├── views/            # 기능별 페이지 컴포넌트
│   ├── HomeView.vue
│   └── AboutView.vue
├── App.vue           # RouterLink, RouterView 태그가 추가됨
└── main.js
```

> `views` 폴더는 기능별 페이지 컴포넌트를 담으며, `components` 폴더의 조각 컴포넌트들과 구분하기 위해 파일명 끝에 `View`를 붙이는 것을 권장한다.

---

## 3. Basic Routing — 기본 동작 순서

1. `router/index.js`에 라우터 관련 설정 작성 (주소, 이름, 컴포넌트)
2. `<RouterLink>`의 `to` 속성에 index.js에서 정의한 주소 값 작성
3. `<RouterLink>` 클릭 시 경로와 일치하는 컴포넌트가 `<RouterView>`에서 렌더링

### router/index.js 설정 예시

```js
// router/index.js
import HomeView from '../views/HomeView.vue'

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    {
      path: '/',
      name: 'home',
      component: HomeView,
    },
    {
      path: '/about',
      name: 'about',
      component: () => import('../views/AboutView.vue'),  // Lazy Loading
    },
  ]
})
```

### App.vue 설정 예시

```html
<!-- App.vue -->
<template>
  <header>
    <nav>
      <RouterLink to="/">Home</RouterLink>
      <RouterLink to="/about">About</RouterLink>
    </nav>
  </header>

  <RouterView />
</template>
```

---

## 4. Named Routes — 이름으로 경로 이동

경로(path)를 직접 사용하는 방식은 URL이 변경될 때 해당 경로를 사용하는 모든 파일을 수정해야 하는 단점이 있다(유지보수 난이도 증가).

**Named Routes**는 경로에 `name` 속성으로 이름을 지정하고, `v-bind`를 사용해 `to` props를 객체로 전달하는 방식이다.

```html
<!-- App.vue -->
<!-- ❌ 경로 하드코딩: 유지보수 어려움 -->
<RouterLink to="/">Home</RouterLink>

<!-- ✅ Named Routes 사용: 경로 변경 시 index.js만 수정하면 됨 -->
<RouterLink :to="{ name: 'home' }">Home</RouterLink>
<RouterLink :to="{ name: 'about' }">About</RouterLink>
```

---

## 5. Dynamic Route Matching — 동적 경로 매칭

URL의 일부를 변수로 사용하여 경로를 동적으로 매칭하는 기능이다. Django의 **Variable Routing** (`<int:article_pk>`)과 동일한 개념이며, Vue에서는 **콜론(`:`)** 으로 매개변수를 표기한다.

예: `/user/1`, `/user/2`, `/user/100` → 동일한 `UserView` 컴포넌트 하나로 처리

### index.js 설정

```js
// index.js
import UserView from '../views/UserView.vue'

const router = createRouter({
  routes: [
    {
      path: '/user/:id',   // :id가 동적 매개변수
      name: 'user',
      component: UserView
    },
  ]
})
```

### RouterLink에서 params 전달

```html
<!-- App.vue -->
<script setup>
import { ref } from 'vue'
const userId = ref(1)
</script>

<template>
  <!-- params 속성으로 매개변수 전달 (key는 index.js의 변수명과 일치해야 함) -->
  <RouterLink :to="{ name: 'user', params: { id: userId } }">User</RouterLink>
</template>
```

### 컴포넌트에서 매개변수 접근

**방법 1: 템플릿에서 `$route.params` 직접 사용**

```html
<!-- UserView.vue -->
<template>
  <div>
    <h2>{{ $route.params }}번 User 페이지</h2>
    <h2>{{ $route.params.id }}번 User 페이지</h2>
  </div>
</template>
```

**방법 2: `useRoute()` 함수 사용 (스크립트 내에서 활용 시 권장)**

```html
<!-- UserView.vue -->
<script setup>
import { ref } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()
const userId = ref(route.params.id)
</script>

<template>
  <div>
    <h2>{{ userId }}번 User 페이지</h2>
  </div>
</template>
```

> `$route.params`로 템플릿에서 바로 출력하는 것보다, `useRoute()`를 통해 스크립트 내에서 반응형 변수에 할당 후 템플릿에 출력하는 방식을 권장한다.

---

## 6. Nested Routes — 중첩 라우팅

애플리케이션의 UI가 여러 레벨로 중첩된 컴포넌트로 구성될 때 사용하는 라우팅 방식이다. 특정 페이지(부모)의 레이아웃은 유지한 채, 그 안의 일부 영역만 URL에 따라 다르게 보여준다.

```
/user/:id/profile  →  User 페이지 안에 UserProfile 컴포넌트 렌더링
/user/:id/posts    →  User 페이지 안에 UserPosts 컴포넌트 렌더링
```

### index.js — children 속성 사용

```js
// index.js
{
  path: '/user/:id',
  // name: 'user',  ← 중첩 라우트에서는 보통 하위 경로에만 이름 지정
  component: UserView,
  children: [
    {
      path: '',           // /user/:id 접속 시 기본으로 보여줄 컴포넌트
      name: 'user',
      component: UserProfile
    },
    {
      path: 'profile',   // /user/:id/profile  ← 슬래시(/) 없이 작성!
      name: 'userProfile',
      component: UserProfile
    },
    {
      path: 'posts',     // /user/:id/posts
      name: 'userPosts',
      component: UserPosts
    }
  ],
},
```

### UserView.vue — 자식 RouterLink와 RouterView 추가

```html
<!-- UserView.vue -->
<script setup>
import { useRoute, RouterLink, RouterView } from 'vue-router'
const route = useRoute()
const userId = ref(route.params.id)
</script>

<template>
  <div>
    <RouterLink :to="{ name: 'userProfile' }">Profile</RouterLink>
    <RouterLink :to="{ name: 'userPosts' }">Posts</RouterLink>
    <h1>UserView</h1>
    <h2>{{ userId }}번 User 페이지</h2>
    <hr>
    <RouterView />  <!-- 자식 컴포넌트가 렌더링되는 위치 -->
  </div>
</template>
```

### 중첩 라우팅 주의사항

- 컴포넌트 간 부모-자식 관계가 아닌, **URL에서의 중첩 관계**로 바라봐야 함
- 자식 라우트의 `path`는 `/` 없이 작성 (앞에 `/`를 붙이면 루트 경로로 인식됨)
- 부모 라우트의 매개변수(`:id`)는 자식 컴포넌트에서도 바로 접근해 사용 가능

---

## 7. Programmatic Navigation — 프로그래밍 방식 이동

`<RouterLink>`를 사용하는 대신, **JavaScript 코드를 사용해 페이지를 이동**시키는 방식이다. 장고에서 `redirect()`를 사용했던 것처럼, 특정 동작(예: 글 작성 완료 후) 이후 자동으로 다른 페이지로 이동시킬 때 유용하다.

### router.push() — 다른 URL로 이동

히스토리 스택에 새 항목을 push하므로 **뒤로 가기** 가능. `<RouterLink>` 클릭 시 내부적으로 호출되는 메서드와 동일하다.

```html
<!-- UserView.vue -->
<script setup>
import { useRoute, useRouter } from 'vue-router'

const route = useRoute()
const router = useRouter()

const goHome = function () {
  router.push({ name: 'home' })
}
</script>

<template>
  <button @click="goHome">홈으로!</button>
</template>
```

**router.push() 인자 활용**

```js
router.push('/users/alice')                              // 리터럴 문자열
router.push({ path: '/users/alice' })                   // 경로 객체
router.push({ name: 'user', params: { username: 'alice' } })  // Named Route + params
router.push({ path: '/register', query: { plan: 'private' } }) // 쿼리스트링
```

### router.replace() — 현재 위치 교체

히스토리 스택에 새 항목을 push하지 않고 **현재 항목을 교체**하므로, 이동 전 URL로 **뒤로 가기 불가**.

```js
router.replace({ name: 'home' })
```

| 구분 | 선언적 표현 | 프로그래밍적 표현 |
|------|------------|----------------|
| push | `<RouterLink :to="...">` | `router.push(...)` |
| replace | `<RouterLink :to="..." replace>` | `router.replace(...)` |

---

## 8. route와 router — 핵심 차이

| 구분 | `useRoute()` (route) | `useRouter()` (router) |
|------|---------------------|------------------------|
| 개념 | 현재 경로(페이지) 정보 읽기용 | 전체 라우팅 관리용 인스턴스 |
| 역할 | 현재 상태 확인 | 경로 변경, 이동 |
| 주요 사용 | `params`, `query`, `name` 등 현재 라우트 정보 확인 | `push`, `replace` 등으로 라우트 변경(내비게이션) |
| 예제 | `route.params.id` 확인 | `router.push('/home')`으로 페이지 이동 |

> 'er'이 붙은 거는 액션용(router), 그냥 'e'로 끝나는 거는 읽기용(route)이다.

```html
<script setup>
import { useRoute, useRouter } from 'vue-router'

const route = useRoute()   // 읽기용: 현재 URL 정보 접근
const router = useRouter() // 동작용: 페이지 이동 등 제어
</script>
```

---

## 💡 한 줄 요약

> Vue Router는 SPA에서 URL에 따라 다른 컴포넌트를 렌더링하는 공식 라우팅 라이브러리로, `<RouterLink>`로 링크를 만들고 `<RouterView>`에서 컴포넌트를 보여준다.

## ❓ 더 찾아볼 것

- `createWebHistory` vs `createWebHashHistory` 차이
- Vue Router 공식 문서: https://router.vuejs.org/
- Named Routes 공식 문서: https://router.vuejs.org/guide/essentials/named-routes.html
- Dynamic Route Matching 공식 문서: https://router.vuejs.org/guide/essentials/dynamic-matching.html
