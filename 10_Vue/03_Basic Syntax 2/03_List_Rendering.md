# [Vue] List Rendering

---

## 1. v-for

`v-for`는 **소스 데이터를 기반으로 요소 또는 템플릿 블록을 반복 렌더링**하는 Directive다.  
장고(Django)의 DTL `{% for %}`처럼, Vue에서도 동일한 구조의 요소를 여러 번 반복해서 화면에 표시할 때 사용한다.

반복 가능한 소스: `Array`, `Object`, `Number`, `String`, `Iterable`

### v-for 구조

`v-for`는 `alias in expression` 형식의 특별한 구문을 사용한다. Python의 `for item in items`와 동일한 구조다.

```html
<div v-for="item in items">
  {{ item.text }}
</div>
```

객체는 key-value 쌍으로 이루어져 있어, 값(value), 키(key), 인덱스(index)를 조합하여 순회할 수 있다.  
JavaScript는 전달되는 인자보다 받는 매개변수 수를 마음대로 조정할 수 있어, 필요한 것만 받아서 쓸 수 있다.

```html
<!-- 가장 기본적인 구조 -->
<div v-for="(item, index) in arr"></div>

<!-- 값만 순회 -->
<div v-for="value in object"></div>

<!-- 값과 키를 순회 -->
<div v-for="(value, key) in object"></div>

<!-- 값과 키, 인덱스를 순회 -->
<div v-for="(value, key, index) in object"></div>
```

> 배열은 두 번째 요소가 **인덱스**, 객체는 두 번째 요소가 **키**, 세 번째가 인덱스다.

### v-for 예시 — 배열

```javascript
const myArr = ref([
  { name: 'Alice', age: 20 },
  { name: 'Bella', age: 21 }
])
```

```html
<div v-for="(item, index) in myArr">
  {{ index }} / {{ item.name }}
</div>
<!-- 결과: 0 / Alice, 1 / Bella -->
```

### v-for 예시 — 객체

```javascript
const myObj = ref({
  name: 'Cathy',
  age: 30
})
```

```html
<div v-for="(value, key, index) in myObj">
  {{ index }} / {{ key }} / {{ value }}
</div>
<!-- 결과: 0 / name / Cathy, 1 / age / 30 -->
```

### 여러 요소에 v-for 적용 — `<template>` 활용

```html
<ul>
  <template v-for="item in myArr">
    <li>{{ item.name }}</li>
    <li>{{ item.age }}</li>
    <hr>
  </template>
</ul>
```

### 중첩된 v-for

각 `v-for`의 하위 영역(scope)은 상위 영역에 접근할 수 있다.

```javascript
const myInfo = ref([
  { name: 'Alice', age: 20, friends: ['Bella', 'Cathy', 'Dan'] },
  { name: 'Bella', age: 21, friends: ['Alice', 'Cathy'] }
])
```

```html
<ul v-for="item in myInfo">
  <li v-for="friend in item.friends">
    {{ item.name }} - {{ friend }}
  </li>
</ul>
```

---

## 2. v-for with key

`v-for` 구문은 각 요소를 **Key**를 활용하여 고유한 값으로 식별할 수 있다.

> **"반드시 v-for와 key를 함께 사용한다"**

Vue의 내부 가상 DOM 알고리즘이 이전 목록과 새 노드 목록을 비교할 때, 각 node를 식별하는 용도로 사용된다. 강의장을 201호, 202호, 203호로 구분하듯이 각 요소에 이름표를 붙여주는 것이다.

key가 없으면, 삭제된 요소의 자리에 남아있는 DOM을 재활용하면서 스타일이나 상태가 엉뚱하게 적용되는 버그가 발생할 수 있다.

> **Vue 3 + 컴포넌트**: 컴포넌트 단위로 v-for를 사용할 때 key를 생략하면 **에러**가 발생한다. 지금은 단순 HTML이라 경고 수준이지만, 다음 주 컴포넌트 학습부터는 필수다.

```javascript
let id = 0
const items = ref([
  { id: id++, name: 'Alice' },
  { id: id++, name: 'Bella' }
])
```

```html
<div v-for="item in items" :key="item.id">
  {{ item.name }}
</div>
```

### 올바른 key 선택 기준

| 구분 | 예시 | 이유 |
|------|------|------|
| ✅ 권장 | 데이터베이스의 고유 ID (pk), UUID | 중복 없는 고유 식별자 |
| ❌ 피해야 함 | 배열 인덱스(index) | 요소 삭제 시 번호가 재배정되어 고유성 없음 |
| ❌ 피해야 함 | 객체 자체 | 숫자/문자열이 아님 |

> **배열 인덱스가 key로 부적절한 이유**: 3개 요소 중 중간이 삭제되면, 남은 인덱스는 0, 1이 된다. 삭제 전 0, 1, 2에서 인덱스 1번이 삭제되어도 남은 것의 1번이 여전히 존재하므로 동일 인덱스가 다른 데이터를 가리키게 된다.

---

## 3. v-for with v-if

동일 요소에 `v-for`와 `v-if`를 함께 사용하면 **안 된다**.

### Vue 2 vs Vue 3 우선순위 변화

| 버전 | 우선순위 |
|------|---------|
| Vue 2 | v-for 먼저 처리 → 동작함 |
| **Vue 3** | **v-if 먼저 처리 → 에러 발생** |

Vue 3에서 `v-if`가 먼저 처리되어 `v-for` 범위의 변수(todo)에 접근할 수 없으므로 에러가 발생한다.

> AI(LLM)가 Vue 2 문법 코드를 줄 수도 있으니 주의! `v-if`와 `v-for`를 같은 요소에 쓴 코드라면 걸러낼 수 있어야 한다.

```html
<!-- ❌ Vue 3에서 에러 발생 -->
<li v-for="todo in todos" v-if="!todo.isComplete" :key="todo.id">
  {{ todo.name }}
</li>
<!-- Uncaught TypeError: Cannot read properties of undefined (reading 'isComplete') -->
```

### 해결법 1: computed 활용

미리 필터링된 목록을 computed로 만들어 반복한다.

```javascript
const completeTodos = computed(() => {
  return todos.value.filter((todo) => !todo.isComplete)
})
```

```html
<ul>
  <li v-for="todo in completeTodos" :key="todo.id">
    {{ todo.name }}
  </li>
</ul>
```

### 해결법 2: v-for와 `<template>` 분리

`v-for`를 `<template>`으로 빼고, `v-if`를 내부 요소에 넣으면 가독성도 좋고 오류도 없다.

```html
<ul>
  <template v-for="todo in todos" :key="todo.id">
    <li v-if="!todo.isComplete">
      {{ todo.name }}
    </li>
  </template>
</ul>
```

---

## 💡 한 줄 요약
> `v-for`는 배열·객체를 반복 렌더링하며, 반드시 `:key`를 함께 사용해야 하고, Vue 3에서 `v-if`와는 동일 요소에 쓰지 않는다.

## ❓ 더 찾아볼 것
- 배열 변경 메서드 비교: 원본 변경(`push`, `pop`, `shift`, `unshift`, `splice`, `sort`, `reverse`) vs 새 배열 반환(`filter`, `map`, `concat`, `slice`)
- `v-for`와 배열을 활용한 필터링/정렬 패턴 (computed 활용)
- Todo 애플리케이션 구현: v-for + v-model + v-if + computed 종합 활용
