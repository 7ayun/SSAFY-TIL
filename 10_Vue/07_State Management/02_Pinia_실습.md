# [Vue] Pinia 실습

---

## 1. 실습 목표 및 컴포넌트 구조

Pinia를 활용한 **Todo CRUD 프로젝트** 구현.
- Todo 목록 조회(Read), 생성(Create), 삭제(Delete), 수정(Update)
- 완료된 Todo 개수 계산 (getters 활용)
- 새로고침 후에도 데이터 유지 (Local Storage 연동)

컴포넌트 트리 구조는 다음과 같다.

```
App
├── TodoList
│   └── TodoListItem  (v-for로 반복)
└── TodoForm
```

---

## 2. 사전 준비

```
1. App.vue를 제외한 초기 생성 컴포넌트 전부 삭제
2. src/assets 내부 파일 모두 삭제
3. main.js에서 import './assets/main.css' 삭제
```

**main.js** – Pinia가 이미 등록되어 있는지 확인한다.

```js
// main.js
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'

const app = createApp(App)

app.use(createPinia())

app.mount('#app')
```

**컴포넌트 파일 생성 및 등록**

```vue
<!-- TodoListItem.vue -->
<template>
  <div>TodoListItem</div>
</template>
```

```vue
<!-- TodoList.vue -->
<template>
  <div>
    <TodoListItem />
  </div>
</template>

<script setup>
import TodoListItem from '@/components/TodoListItem.vue'
</script>
```

```vue
<!-- TodoForm.vue -->
<template>
  <div>TodoForm</div>
</template>
```

```vue
<!-- App.vue -->
<template>
  <div>
    <h1>Todo Project</h1>
    <TodoList />
    <TodoForm />
  </div>
</template>

<script setup>
import TodoForm from '@/components/TodoForm.vue'
import TodoList from '@/components/TodoList.vue'
</script>
```

---

## 3. Todo 조회 (Read)

**store에 임시 todos 목록 state 정의**

```js
// stores/counter.js
import { ref, computed } from 'vue'
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', () => {
  let id = 0  // ref 불필요 — 화면에 노출되지 않는 단순 카운터

  const todos = ref([
    { id: id++, text: '할 일 1', isDone: false },
    { id: id++, text: '할 일 2', isDone: false }
  ])

  return { todos }
})
```

> `let id`는 화면에 보여줄 필요가 없으므로 ref로 만들지 않는다. 새 항목이 추가될 때마다 자동으로 증가하는 단순 변수다.

**TodoList에서 store의 todos를 v-for로 출력, 각 todo를 props로 전달**

```vue
<!-- TodoList.vue -->
<template>
  <div>
    <TodoListItem
      v-for="todo in store.todos"
      :key="todo.id"
      :todo="todo"
    />
  </div>
</template>

<script setup>
import TodoListItem from '@/components/TodoListItem.vue'
import { useCounterStore } from '@/stores/counter'

const store = useCounterStore()
</script>
```

> "TodoList에서 TodoListItem으로 todo를 하나씩 내려주는 건 1단계 부모-자식 전달이므로 props를 써도 된다."

**TodoListItem에서 props를 받아 화면에 출력**

```vue
<!-- TodoListItem.vue -->
<template>
  <div>
    <input type="checkbox" name="todo-text">
    <label for="todo-text">{{ todo.text }}</label>
    <button>삭제</button>
  </div>
</template>

<script setup>
defineProps({
  todo: Object
})
</script>
```

---

## 4. Todo 생성 (Create)

**store에 addTodo action 정의**

```js
// stores/counter.js
const addTodo = function (todoText) {
  todos.value.push({
    id: id++,
    text: todoText,
    isDone: false
  })
}

return { todos, addTodo }
```

**TodoForm에서 입력값을 v-model로 바인딩하고, submit 시 store의 addTodo 호출**

```vue
<!-- TodoForm.vue -->
<template>
  <div>
    <form @submit.prevent="createTodo(todoText)" ref="formElem">
      <input type="text" v-model="todoText">
      <input type="submit">
    </form>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useCounterStore } from '@/stores/counter'

const store = useCounterStore()
const todoText = ref('')
const formElem = ref(null)

const createTodo = function (todoText) {
  store.addTodo(todoText)
  formElem.value.reset()  // 입력창 초기화
}
</script>
```

> store의 addTodo를 직접 호출하지 않고 createTodo 함수를 별도로 만드는 이유: **addTodo 호출 전후로 추가 로직(여기서는 폼 초기화)을 작성할 수 있기 때문**이다.

---

## 5. Todo 삭제 (Delete)

**store에 deleteTodo action 정의 (2가지 방법)**

```js
// stores/counter.js

// 방법 1: findIndex + splice (in-place 삭제)
const deleteTodo = function (selectedId) {
  const index = todos.value.findIndex((todo) => todo.id === selectedId)
  todos.value.splice(index, 1)
}

// 방법 2: filter (전체 배열 재생성)
const deleteTodo = function (selectedId) {
  todos.value = todos.value.filter(todo => todo.id !== selectedId)
}

return { todos, addTodo, deleteTodo }
```

**TodoListItem에서 삭제 버튼 클릭 시 store의 deleteTodo 호출**

```vue
<!-- TodoListItem.vue -->
<template>
  <div>
    <input type="checkbox" name="todo-text">
    <label for="todo-text">{{ todo.text }}</label>
    <button @click="deleteTodo(todo.id)">삭제</button>
  </div>
</template>

<script setup>
import { useCounterStore } from '@/stores/counter'

defineProps({ todo: Object })

const store = useCounterStore()

const deleteTodo = function (selectedId) {
  store.deleteTodo(selectedId)
}
</script>
```

---

## 6. Todo 수정 (Update)

목표: 체크박스 클릭 시 해당 todo의 `isDone` 속성을 토글한다.

**흐름**: 체크박스 클릭 → `isDone` ref 변경 → `watch`가 감지 → `store.updateTodo` 호출

**store에 updateTodo action 정의 (2가지 방법)**

```js
// stores/counter.js

// 방법 1: forEach (in-place 수정)
const updateTodo = function (selectedId) {
  todos.value.forEach((todo) => {
    if (todo.id === selectedId) {
      todo.isDone = !todo.isDone
    }
  })
}

// 방법 2: map (전체 배열 재생성)
const updateTodo = function (selectedId) {
  todos.value = todos.value.map((todo) => {
    if (todo.id === selectedId) {
      todo.isDone = !todo.isDone
    }
    return todo
  })
}

return { todos, addTodo, deleteTodo, updateTodo }
```

**TodoListItem에서 체크박스에 v-model 바인딩 후 watch로 변화 감지**

```vue
<!-- TodoListItem.vue -->
<template>
  <div>
    <input type="checkbox" name="todo-text" v-model="isDone">
    <label for="todo-text" :class="{ 'is-done': todo.isDone }">{{ todo.text }}</label>
    <button @click="deleteTodo(todo.id)">삭제</button>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'
import { useCounterStore } from '@/stores/counter'

const props = defineProps({ todo: Object })
const store = useCounterStore()

const isDone = ref(props.todo.isDone)

watch(isDone, () => {
  store.updateTodo(props.todo.id)
})

const deleteTodo = function (selectedId) {
  store.deleteTodo(selectedId)
}
</script>

<style scoped>
.is-done {
  text-decoration: line-through;
}
</style>
```

---

## 7. 수정과 삭제 구현 방식 비교

| 방식 | 설명 | 메서드 예시 |
|------|------|------------|
| **In-place** | 배열 전체 재생성 없이 필요한 항목만 바로 수정/제거 | `forEach` + 직접 수정, `splice` |
| **전체 배열 재생성** | 조건에 맞는 항목만 남기거나, 변경 사항을 반영한 새 배열을 만든 뒤 기존 배열에 재할당 | `filter`, `map` |

> 두 방식 모두 성능과 가독성 면에서 큰 차이가 없는 경우가 많다. 중요한 것은 **무엇을 의도하는지(단일 항목 수정/삭제 vs. 전체 재생성)를 명확히 알고 선택하는 것**이다. 프로젝트나 팀 컨벤션에 따라 달라질 수 있다.

---

## 8. Todo 카운팅 (getters 활용)

완료된 Todo 개수를 getters로 계산한다.

```js
// stores/counter.js
const doneTodosCount = computed(() => {
  const doneTodos = todos.value.filter((todo) => todo.isDone)
  return doneTodos.length
})

return { todos, addTodo, deleteTodo, updateTodo, doneTodosCount }
```

**App.vue에서 getter를 참조해 화면에 표시**

```vue
<!-- App.vue -->
<template>
  <div>
    <h1>Todo Project</h1>
    <h2>완료된 Todo 개수 : {{ store.doneTodosCount }}</h2>
    <TodoList />
    <TodoForm />
  </div>
</template>

<script setup>
import { useCounterStore } from '@/stores/counter'
import TodoForm from '@/components/TodoForm.vue'
import TodoList from '@/components/TodoList.vue'

const store = useCounterStore()
</script>
```

---

## 9. Local Storage 연동 (pinia-plugin-persistedstate)

새로고침 후에도 todos 상태가 유지되도록 `pinia-plugin-persistedstate`를 사용한다.

**Local Storage란?** 브라우저 내에 key-value 쌍을 저장하는 웹 스토리지 객체.
- 페이지를 새로고침하거나 브라우저를 재실행해도 데이터가 유지된다.
- 쿠키와 달리 네트워크 요청 시 서버로 전송되지 않는다.
- 여러 탭이나 창 간에 데이터를 공유할 수 있다.

**설치 및 등록**

```bash
$ npm i pinia-plugin-persistedstate
```

```js
// main.js
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'
import App from './App.vue'

const app = createApp(App)
const pinia = createPinia()

pinia.use(piniaPluginPersistedstate)

// app.use(createPinia()) → 변경
app.use(pinia)

app.mount('#app')
```

**store에 persist 옵션 추가 (`defineStore` 3번째 인자)**

```js
// stores/counter.js
export const useCounterStore = defineStore('counter', () => {
  // ... state, getters, actions ...
  return { todos, addTodo, deleteTodo, updateTodo, doneTodosCount }
}, { persist: true })  // ← 3번째 인자
```

적용 후 개발자도구 → Application → Local Storage에서 `counter` 키로 todos 상태가 저장되는 것을 확인할 수 있다.

---

## 10. Pinia 활용 시점

> "Pinia를 사용한다고 모든 데이터를 state에 넣어야 하는 것은 아니다. 컴포넌트 내부에서만 사용하는 데이터까지 Pinia로 관리하면 코드가 불필요하게 복잡해진다."

| 상황 | 권장 방식 |
|------|-----------|
| 단순한 부모-자식 간 1단계 데이터 전달 | props / emit |
| 여러 컴포넌트가 공유하거나, depth가 깊어 전달이 복잡한 경우 | Pinia |
| SPA 규모가 커질수록 | Pinia 선택이 자연스러워짐 |

---

## 💡 한 줄 요약
> Pinia store에 todos 상태와 CRUD actions을 정의하면, 컴포넌트 계층 어디서든 직접 접근해 데이터를 조작할 수 있으며, `pinia-plugin-persistedstate`로 새로고침 후에도 상태를 유지할 수 있다.

## ❓ 더 찾아볼 것
- `watch` vs `watchEffect` 차이점
- Pinia store 여러 개 사용 시 스토어 간 의존성 처리 방법
- `pinia-plugin-persistedstate` 공식 문서: https://prazdevs.github.io/pinia-plugin-persistedstate/
- DRF(Django REST Framework)와 Vue Pinia 연동 방법 (axios 사용)
