# [Vue] Passing Props

---

## 1. 컴포넌트와 데이터 흐름

Vue의 컴포넌트는 각각 **싱글 파일(.vue)**로 데이터를 개별 관리한다.  
즉, `A.vue`의 데이터를 `B.vue`에서 별도 설정 없이 직접 참조하는 것은 불가능하다.

컴포넌트 간 데이터를 주고받는 두 가지 흐름:

| 방향 | 방법 |
|------|------|
| 부모 → 자식 | **Props** 전달 |
| 자식 → 부모 | **Emit** (커스텀 이벤트) |

> 예: Todo List를 만들 때, 전체 목록은 부모가 관리하고 각 항목은 자식이 렌더링한다.  
> 부모는 자식에게 '오늘 할 일'을 **props**로 알려주고, 자식은 삭제 버튼이 눌리면 **emit**으로 부모에게 알린다.

---

## 2. Props란

**Props(Properties)**는 부모 컴포넌트가 자식 컴포넌트에게 데이터를 전달할 때 사용하는 **사용자 지정 특성(attribute)**이다.

```
Props = 부모 컴포넌트로부터 자식 컴포넌트로 데이터를 전달하는 사용자 지정 특성
```

- 데이터는 **부모 → 자식** 방향으로만 흐른다 (읽기 전용)
- 자식 컴포넌트는 전달받은 props를 **직접 수정해서는 안 된다**
- 부모 컴포넌트가 업데이트될 때마다 해당 자식의 모든 props도 최신 값으로 자동 갱신된다

> **TIP:** 자식이 부모의 데이터를 변경하고 싶다면, props를 직접 수정하지 말고 이후에 배울 **emit**을 통해 부모에게 알려야 한다.  
> 객체/배열 props는 자식에서 내부 값을 바꾸면 부모의 원본도 바뀌므로 주의해야 한다.

---

## 3. One-Way Data Flow (단방향 데이터 흐름)

모든 props는 자식 속성과 부모 속성 사이에 **하향식 단방향 바인딩(one-way-down binding)**을 형성한다.

단방향인 이유:
- 하위 컴포넌트가 실수로 상위 컴포넌트의 상태를 변경하여 앱의 데이터 흐름을 이해하기 어렵게 만드는 것을 방지하기 위함 (무한 루프, 디버깅 난이도 상승 등)
- 데이터 흐름의 **일관성** 및 **예측 가능성**을 높이는 것이 목표

---

## 4. Props 선언 (`defineProps`)

부모에서 내려보낸 props를 자식 컴포넌트에서 사용하려면 **명시적인 선언**이 필요하다.  
`defineProps()`를 사용하며, 인자의 형태에 따라 두 가지 방식으로 선언한다.

### 사전 준비 — 컴포넌트 구조

```
App > Parent > ParentChild
```

```vue
<!-- App.vue -->
<template>
  <div>
    <Parent />
  </div>
</template>

<script setup>
import Parent from '@/components/Parent.vue'
</script>
```

```vue
<!-- Parent.vue -->
<template>
  <div>
    <h1>Parent</h1>
    <ParentChild />
  </div>
</template>

<script setup>
import ParentChild from '@/components/ParentChild.vue'
</script>
```

### 방식 1 — 문자열 배열을 사용한 선언

배열의 문자열 요소가 전달받을 props의 이름이 된다.

```vue
<!-- ParentChild.vue -->
<script setup>
defineProps(['myMsg'])
</script>
```

### 방식 2 — 객체를 사용한 선언 (권장)

각 객체 속성의 키가 props 이름, 값은 해당 데이터의 타입(생성자 함수)이 된다.

```vue
<!-- ParentChild.vue -->
<script setup>
defineProps({
  myMsg: String
})
</script>
```

> **TIP:** 가급적 **객체 선언 방식**을 권장한다. 각 prop에 대해 상세한 규칙(유효성 검사)을 설정하여 컴포넌트의 안정성을 높이고, 코드 자체가 컴포넌트의 설명서 역할을 한다.

---

## 5. Props 작성 및 데이터 사용

### 부모에서 props 작성

부모 컴포넌트에서 자식 컴포넌트 태그에 `props이름="값"` 형태로 작성한다.

```vue
<!-- Parent.vue -->
<template>
  <div>
    <h1>Parent</h1>
    <ParentChild my-msg="message" />
  </div>
</template>
```

### 자식에서 props 데이터 사용

선언 후 **템플릿**에서는 반응형 변수처럼, **JS**에서는 `defineProps()`의 반환 객체로 접근한다.

```vue
<!-- ParentChild.vue -->
<template>
  <div>
    <h2>ParentChild</h2>
    <!-- 템플릿에서 활용 -->
    <p>{{ myMsg }}</p>
  </div>
</template>

<script setup>
// JS에서 props 데이터를 활용해야 하는 경우
const props = defineProps({ myMsg: String })

console.log(props)
console.log(props.myMsg)
</script>
```

---

## 6. Props 세부사항

### Props Name Casing (이름 컨벤션)

| 위치 | 표기법 | 예시 |
|------|--------|------|
| 부모 템플릿 (HTML 속성) | kebab-case | `my-msg="message"` |
| 자식 스크립트 (JavaScript) | camelCase | `myMsg: String` |

```vue
<!-- 부모: HTML 속성이므로 kebab-case -->
<ParentChild my-msg="message" />

<!-- 자식: JavaScript이므로 camelCase -->
<script setup>
defineProps({ myMsg: String })
</script>

<template>
  <p>{{ myMsg }}</p>
</template>
```

### Static Props vs Dynamic Props

지금까지 작성한 것은 **Static(정적) props**로, 값이 고정된 문자열로 전달된다.  
`v-bind(:)`를 사용하면 **동적으로 할당된 값**을 props로 전달할 수 있다.

> **TIP — 동적 할당이란?** 고정된 값을 전달하는 것이 아닌, 변하는 데이터를 연결하는 것이다.  
> 이는 부모의 데이터와 자식의 속성을 실시간으로 연결하는 것을 의미한다.

```vue
<!-- Parent.vue -->
<template>
  <div>
    <ParentChild
      my-msg="message"
      :dynamic-props="name"
    />
  </div>
</template>

<script setup>
import { ref } from 'vue'
const name = ref('Alice')
</script>
```

```vue
<!-- ParentChild.vue -->
<template>
  <div>
    <p>{{ myMsg }}</p>
    <p>{{ dynamicProps }}</p>
  </div>
</template>

<script setup>
defineProps({
  myMsg: String,
  dynamicProps: String,
})
</script>
```

> **주의사항 — 정적 vs 동적 props 타입:**  
> - **정적(바인딩 없음):** 전달되는 값은 무조건 **String** 타입  
>   `<SomeComponent num-props="1" />` → 문자열 `"1"` 전달  
> - **동적(v-bind 사용):** 변수에 저장된 실제 데이터 타입 유지  
>   `<SomeComponent :num-props="1" />` → 숫자 `1` 전달

---

## 7. 한 단계 더 props 내려 보내기

props는 계층 구조에서 단계적으로 전달할 수 있다.

```
App > Parent > ParentChild > ParentGrandChild
```

ParentChild가 Parent로부터 받은 `myMsg`를 ParentGrandChild에게 다시 전달하는 경우:

```vue
<!-- ParentChild.vue -->
<template>
  <div>
    <h2>ParentChild</h2>
    <p>{{ myMsg }}</p>
    <!-- v-bind로 동적 전달 -->
    <ParentGrandChild :my-msg="myMsg" />
  </div>
</template>

<script setup>
import ParentGrandChild from '@/components/ParentGrandChild.vue'
defineProps({ myMsg: String })
</script>
```

```vue
<!-- ParentGrandChild.vue -->
<template>
  <div>
    <p>{{ myMsg }}</p>
  </div>
</template>

<script setup>
defineProps({ myMsg: String })
</script>
```

Parent가 props를 변경하면 해당 props를 전달받고 있는 ParentChild, ParentGrandChild 모두 자동 업데이트된다.

---

## 8. Props 활용 — v-for와 함께 반복 요소 전달하기

배열 데이터를 `v-for`로 반복하면서 각 요소를 자식 컴포넌트의 props로 전달하는 패턴이다.  
실무에서는 **Axios로 서버에서 목록 데이터를 받아온 후**, 자식 컴포넌트에게 하나씩 넘겨 출력하는 방식으로 활용된다.

```vue
<!-- Parent.vue -->
<template>
  <div>
    <ParentItem
      v-for="item in items"
      :key="item.id"
      :my-prop="item"
    />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import ParentItem from '@/components/ParentItem.vue'

const items = ref([
  { id: 1, name: '사과' },
  { id: 2, name: '바나나' },
  { id: 3, name: '딸기' }
])
</script>
```

```vue
<!-- ParentItem.vue -->
<template>
  <div>
    <p>{{ myProp.id }}</p>
    <p>{{ myProp.name }}</p>
  </div>
</template>

<script setup>
defineProps({
  myProp: Object
})
</script>
```

- `v-for`와 `:key`는 한 쌍으로 사용한다. `:key`에는 각 요소를 구분할 수 있는 고유값(id 등)을 바인딩한다.
- 반복이 돌 때마다 해당 `item`이 자식 컴포넌트에게 props로 전달된다.

---

## 💡 한 줄 요약

> 부모에서 자식으로 데이터를 전달할 때는 **Props**를 사용하며, 데이터는 항상 **한 방향(위→아래)**으로만 흐른다.

---

## ❓ 더 찾아볼 것

- Props 유효성 검사 상세 옵션 (`required`, `default`, `validator`)
- `v-bind` 없이 객체 전체를 props로 전달하는 방법 (`v-bind="obj"`)
- Prop Drilling 문제와 해결책 (Pinia, Provide/Inject)
