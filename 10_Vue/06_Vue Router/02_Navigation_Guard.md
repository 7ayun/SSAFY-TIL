# [Vue] Navigation Guard

---

## 1. Navigation Guard란

**Navigation Guard**는 Vue Router를 통해 특정 URL에 접근할 때, 다른 URL로 redirect하거나 접근을 취소하여 **내비게이션을 보호**하는 기능이다.

라우트 전환 전/후에 자동으로 실행되며, 사용자의 로그인 상태나 권한을 확인하여 내비게이션을 허용하거나 취소하거나, 다른 페이지로 리다이렉트시킬 수 있다.

**주요 사용 예시:**
- 로그인하지 않은 사용자가 로그인이 필요한 페이지에 접근하려 할 때 → 로그인 페이지로 리다이렉트
- 이미 로그인한 사용자가 로그인 페이지에 접근하려 할 때 → 홈으로 리다이렉트
- 글 작성 중 페이지를 떠날 때 → 확인 팝업 표시

---

## 2. Navigation Guard의 종류

Navigation Guard는 **적용 범위**에 따라 3가지로 구분된다.

| 종류 | 적용 범위 | 작성 위치 |
|------|----------|----------|
| **Globally Guard** (전역 가드) | 애플리케이션 전체 모든 라우트 전환 | `router/index.js` |
| **Per-route Guard** (라우터 가드) | 특정 라우트에만 적용 | `router/index.js`의 각 route 객체 |
| **In-component Guard** (컴포넌트 가드) | 특정 컴포넌트 내에서만 적용 | 각 컴포넌트의 `<script>` 내부 |

---

## 3. Globally Guard — 전역 가드

애플리케이션 전역에서 동작하는 가드. `router/index.js`에서 `router` 인스턴스에 직접 붙인다.

### 종류

- `beforeEach()` — 다른 URL로 **이동하기 직전**에 실행 (가장 많이 사용)
- `beforeResolve()` — beforeEach와 모든 컴포넌트 단위 가드가 실행된 후, 내비게이션이 확정되기 직전에 호출
- `afterEach()` — 내비게이션이 완전히 확정된 후 실행 (이동을 중단하거나 변경 불가)

### beforeEach() 구조

```js
// router/index.js
router.beforeEach((to, from) => {
  // to: 이동할 URL 정보가 담긴 Route 객체 (목적지)
  // from: 현재 URL 정보가 담긴 Route 객체 (출발지)

  // return 값에 따라 이동 여부가 결정됨
  return false                    // 현재 내비게이션 취소 (이동 불가)
  return { name: 'About' }        // 해당 경로로 redirect
  // return이 없으면 자동으로 'to' URL Route 객체로 이동 (이동 허용)
})

export default router
```

### beforeEach() 활용 예시 — 로그인 체크

로그인하지 않은 사용자의 페이지 진입을 막고, 로그인 페이지로 이동시키기

```js
// router/index.js
const isAuthenticated = false  // 추후 서버로부터 받아온 유저 정보로 대체

router.beforeEach((to, from) => {
  if (!isAuthenticated && to.name !== 'login') {
    console.log('로그인이 필요합니다.')
    return { name: 'login' }  // 로그인 페이지로 redirect
  }
})
```

### afterEach() 주요 사용 예시

- 페이지 이동 기록 로깅(logging)
- 이동한 페이지에 맞춰 문서 제목(`document.title`) 변경 등 후처리 작업

### beforeResolve() 주요 사용 예시

- 페이지에 진입하기 전에 사용자의 권한과 관련된 데이터를 미리 가져오는 작업
- 비교적 `beforeEach`보다 사용 빈도가 낮음

---

## 4. Per-route Guard — 라우터 가드

특정 라우트(경로)에 진입할 때만 실행되도록 라우트 설정 객체에 직접 정의하는 가드.

주로 `beforeEnter` 가드를 많이 사용하며, `router/index.js`의 각 route 객체 안에 작성한다.

### beforeEnter() 구조

```js
// router/index.js
routes: [
  {
    path: '/user/:id',
    name: 'user',
    component: UserView,
    beforeEnter: (to, from) => {
      // ...
      return false
    }
  }, ...
]
```

> 단순히 URL의 매개변수나 쿼리 값이 변경될 때는 실행되지 않고, **다른 URL에서 탐색해 올 때만** 실행된다.

### beforeEnter() 활용 예시 — 로그인 상태에서 로그인 페이지 진입 차단

이미 로그인한 상태라면 로그인 페이지로의 진입을 막고 홈으로 리다이렉트

```js
// router/index.js
const isAuthenticated = true  // 로그인 상태

{
  path: '/login',
  name: 'login',
  component: LoginView,
  beforeEnter: (to, from) => {
    if (isAuthenticated === true) {
      console.log('이미 로그인 상태입니다.')
      return { name: 'home' }
    }
  }
}
```

---

## 5. In-component Guard — 컴포넌트 가드

특정 컴포넌트 내에서만 동작하는 가드. 각 컴포넌트의 `<script>` 내부에 작성한다.

### 종류

| 가드 | 실행 시점 | 주요 용도 |
|------|----------|----------|
| `onBeforeRouteLeave` | 현재 라우트에서 다른 라우트로 이동하기 전 | 페이지를 떠날 때 확인 팝업 (미저장 데이터 경고 등) |
| `onBeforeRouteUpdate` | 이미 렌더링된 컴포넌트가 같은 라우트 내에서 업데이트되기 전 | 같은 컴포넌트에서 params가 변경될 때 데이터 재요청 |

### onBeforeRouteLeave 활용 예시

사용자가 UserView를 떠날 시 팝업 창 출력

```html
<!-- UserView.vue -->
<script setup>
import { onBeforeRouteLeave } from 'vue-router'

onBeforeRouteLeave((to, from) => {
  const answer = window.confirm('정말 떠나실 건가요?')
  if (answer === false) {
    return false  // 확인 취소 → 이동 취소
  }
})
</script>
```

### onBeforeRouteUpdate 활용 예시

같은 컴포넌트에서 params만 변경될 때 (예: `/user/1` → `/user/100`) 데이터가 자동으로 갱신되지 않는 경우 처리

```html
<!-- UserView.vue -->
<script setup>
import { ref } from 'vue'
import { useRoute, useRouter, onBeforeRouteLeave, onBeforeRouteUpdate } from 'vue-router'

const route = useRoute()
const router = useRouter()
const userId = ref(route.params.id)

// 같은 라우트 내에서 업데이트 될 때
onBeforeRouteUpdate((to, from) => {
  userId.value = to.params.id  // 목적지의 params.id로 userId를 갱신
})

const routeUpdate = function () {
  router.push({ name: 'user', params: { id: 100 } })
}
</script>

<template>
  <div>
    <h1>UserView</h1>
    <h2>{{ userId }}번 User 페이지</h2>
    <button @click="routeUpdate">100번 유저 페이지</button>
  </div>
</template>
```

> **주의** `onBeforeRouteUpdate`를 사용하지 않으면: URL의 params는 변경되지만, `useRoute()`로 초기에 설정한 `userId` 변수는 갱신되지 않아 화면에 이전 값이 그대로 남는다.

---

## 6. Navigation Guard 정리

```
범위: 전역(Globally) > 라우터별(Per-route) > 컴포넌트별(In-component)
```

| 종류 | 가드 | 작성 위치 |
|------|------|----------|
| **Globally** | `beforeEach`, `beforeResolve`, `afterEach` | `router/index.js` |
| **Per-route** | `beforeEnter` | `router/index.js` 각 routes |
| **In-component** | `onBeforeRouteLeave`, `onBeforeRouteUpdate` | 각 컴포넌트의 `<script>` |

---

## 7. 참고 — Lazy Loading Routes

Vue 애플리케이션 첫 빌드 시 해당 컴포넌트를 로드하지 않고, **해당 경로를 처음으로 방문할 때 컴포넌트를 로드**하는 것.

빌드 시 처음부터 모든 컴포넌트를 준비하면, 컴포넌트 크기에 따라 페이지 로드 시간이 길어질 수 있기 때문이다.

```js
// index.js

// ❌ 즉시 로딩: 앱 로드 시 AboutView도 함께 로드됨
import AboutView from '../views/AboutView.vue'
{ component: AboutView }

// ✅ Lazy Loading: /about 경로에 처음 방문할 때만 로드됨
{
  path: '/about',
  name: 'about',
  component: () => import('../views/AboutView.vue')
}
```

> 자주 방문하지 않을 페이지들은 Lazy Loading 형태로 작성하면 초기 로딩 속도를 향상시킬 수 있다.

---

## 💡 한 줄 요약

> Navigation Guard는 라우트 전환 시 자동 실행되는 수문장으로, 전역/라우터별/컴포넌트별 세 가지 범위로 로그인 체크, 접근 제어, 데이터 업데이트 등에 활용한다.

## ❓ 더 찾아볼 것

- Navigation Guard 공식 문서: https://router.vuejs.org/guide/advanced/navigation-guards.html
- Pinia와 Navigation Guard 연동 (로그인 상태 전역 관리)
- `meta` 필드를 활용한 라우터 권한 관리 패턴
