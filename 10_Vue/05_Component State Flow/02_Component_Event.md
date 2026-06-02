# [Vue] Component Event

---

## 1. Emit이란

**Emit**은 자식 컴포넌트가 부모 컴포넌트에게 특정 이벤트가 발생했음을 알리고 데이터를 전달하는 기능이다.

```
$emit() = 자식 컴포넌트가 이벤트를 발생시켜 부모 컴포넌트로 데이터를 전달하는 메서드
```

Props가 **위→아래**로 내려가는 데이터 흐름이라면,  
Emit은 **아래→위**로 올라가는 이벤트를 만들어 컴포넌트 간의 완전한 상호작용을 가능하게 한다.

> **비유:** 옆에 동료를 쿡 찌르면 "왜?"라는 이벤트가 발생하는 것처럼,  
> 자식 컴포넌트가 "아부지, 데이터 받아유~" 하고 부모를 부르는 커스텀 이벤트를 발생시키면  
> 부모 컴포넌트는 그 이벤트를 듣고 이벤트 처리 함수를 실행해 데이터를 받는다.

---

## 2. `$emit()` 메서드

```
$emit(event, ...args)
```

| 매개변수 | 설명 |
|---------|------|
| `event` | 커스텀 이벤트 이름 (자유롭게 지정 가능) |
| `...args` | 부모에게 전달할 추가 데이터 (선택) |

- `$`는 Vue 인스턴스의 내부 변수들을 가리키는 표기로, Life cycle hooks, 인스턴스 메서드 등 내부 특정 속성에 접근할 때 사용한다.
- 주로 HTML 템플릿에서 Vue 내부에 접근해야 할 때 `$` 표기를 붙여 작성한다.

---

## 3. 이벤트 발신 및 수신 (Emitting and Listening to Events)

### 발신 — 자식 컴포넌트에서

`$emit`을 사용하여 템플릿 표현식에서 직접 커스텀 이벤트를 발신한다.

```vue
<!-- 자식 컴포넌트 -->
<button @click="$emit('someEvent')">클릭</button>
```

### 수신 — 부모 컴포넌트에서

부모 컴포넌트는 `v-on`(또는 `@`)을 사용하여 자식이 발신한 이벤트를 수신하고, 콜백 함수를 호출한다.

```vue
<!-- 부모 컴포넌트 -->
<ParentChild @some-event="someCallback" />

<script setup>
const someCallback = function () {
  console.log('ParentChild가 발신한 이벤트를 수신했어요.')
}
</script>
```

---

## 4. emit 이벤트 선언 (`defineEmits`)

`defineEmits()`를 사용하여 발신할 이벤트를 명시적으로 선언한다.  
`<script setup>` 내에서는 `$emit`에 직접 접근할 수 없으므로, `defineEmits()`가 반환하는 `emit` 함수를 사용한다.

props와 마찬가지로 **배열** 방식과 **객체** 방식(권장)으로 선언할 수 있다.

### 배열을 사용한 선언

```vue
<!-- ParentChild.vue -->
<script setup>
// emit 이벤트 선언 (배열 방식)
const emit = defineEmits(['someEvent'])

const buttonClick = function () {
  emit('someEvent')
}
</script>

<template>
  <div>
    <button @click="buttonClick">클릭</button>
  </div>
</template>
```

```vue
<!-- Parent.vue -->
<template>
  <div>
    <ParentChild
      @some-event="someCallback"
      my-msg="message"
      :dynamic-props="name"
    />
  </div>
</template>

<script setup>
const someCallback = function () {
  console.log('ParentChild가 발신한 이벤트를 수신했어요.')
}
</script>
```

---

## 5. 이벤트 전달 — Event Arguments

`$emit`의 두 번째 인자부터 부모로 전달할 추가 데이터를 작성한다.  
부모의 콜백 함수에서 `...args`(rest parameter) 형태로 받을 수 있다.

```vue
<!-- ParentChild.vue (자식) -->
<template>
  <div>
    <button @click="emitArgs">추가 인자 전달</button>
  </div>
</template>

<script setup>
const emit = defineEmits(['emitArgs'])

const emitArgs = function () {
  emit('emitArgs', 1, 2, 3)
}
</script>
```

```vue
<!-- Parent.vue (부모) -->
<template>
  <div>
    <ParentChild @emit-args="getNumbers" />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import ParentChild from '@/components/ParentChild.vue'

const getNumbers = function (...args) {
  console.log(args)
  console.log(`ParentChild가 전달한 추가인자 ${args}를 수신했어요.`)
}
</script>
```

---

## 6. 이벤트 세부사항 — Event Name Casing

Props의 이름 컨벤션과 동일한 원칙이 적용된다.

| 위치 | 표기법 | 예시 |
|------|--------|------|
| 자식: 이벤트 선언 및 발신 (JavaScript) | camelCase | `someEvent` |
| 부모: 이벤트 수신 (HTML 속성) | kebab-case | `@some-event` |

```vue
<!-- 자식: camelCase로 발신 -->
<script setup>
const emit = defineEmits(['someEvent'])
emit('someEvent')
</script>

<!-- 부모: kebab-case로 수신 -->
<ParentChild @some-event="..." />
```

---

## 7. emit 이벤트 활용 실습

**목표:** 최하단 컴포넌트 `ParentGrandChild`에서 버튼을 클릭하면 `Parent` 컴포넌트의 `name` 변수를 변경하기

컴포넌트 구조: `App > Parent > ParentChild > ParentGrandChild`

emit은 **직접 조부모로 올라가지 못하며**, 단계별로 연달아 호출해 올려야 한다.

```
ParentGrandChild --emit--> ParentChild --emit--> Parent
```

**Step 1 — ParentGrandChild에서 이벤트 발신**

```vue
<!-- ParentGrandChild.vue -->
<template>
  <div>
    <button @click="updateName">이름 변경</button>
  </div>
</template>

<script setup>
const emit = defineEmits(['updateName'])

const updateName = function () {
  emit('updateName')
}
</script>
```

**Step 2 — ParentChild에서 이벤트 수신 후 재발신**

```vue
<!-- ParentChild.vue -->
<template>
  <div>
    <p>{{ myMsg }}</p>
    <ParentGrandChild
      :my-msg="myMsg"
      @update-name="updateName"
    />
  </div>
</template>

<script setup>
import ParentGrandChild from '@/components/ParentGrandChild.vue'

const emit = defineEmits(['someEvent', 'emitArgs', 'updateName'])

defineProps({ myMsg: String })

const updateName = function () {
  emit('updateName')
}
</script>
```

**Step 3 — Parent에서 최종 이벤트 수신 후 데이터 변경**

```vue
<!-- Parent.vue -->
<template>
  <div>
    <ParentChild
      @update-name="updateName"
    />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import ParentChild from '@/components/ParentChild.vue'

const name = ref('Alice')

const updateName = function () {
  name.value = 'Bella'
}
</script>
```

버튼 클릭 시 name 값이 'Alice' → 'Bella'로 변경되고, name을 props로 받는 모든 컴포넌트에서 자동 업데이트된다.

---

## 8. 참고 — Props & Emit 객체 선언 문법

### Props 객체 선언 문법을 권장하는 이유

객체 선언 방식을 사용하면 컴포넌트의 의도를 명확히 하여 가독성을 높이고, 잘못된 타입의 데이터가 전달됐을 때 콘솔에 경고를 출력해 실수를 방지할 수 있다.

```js
defineProps({
  // 여러 타입 허용
  propB: [String, Number],
  // 문자열 필수
  propC: {
    type: String,
    required: true
  },
  // 기본값을 가지는 숫자형
  propD: {
    type: Number,
    default: 10
  },
})
```

### Emit 객체 선언 문법

emit도 객체 구문으로 선언할 수 있으며, 이벤트 발신 시 **유효성 검사 함수**를 실행할 수 있다.

```js
const emit = defineEmits({
  // 유효성 검사 없음
  click: null,
  // submit 이벤트 유효성 검사
  submit: ({ email, password }) => {
    if (email && password) {
      return true
    } else {
      console.warn('submit 이벤트가 올지 않음')
      return false
    }
  }
})

const submitForm = function (email, password) {
  emit('submit', { email, password })
}
```

> **주의:** `return false`가 되어도 이벤트 자체는 전달된다. 다만 콘솔에 **warning**이 출력되므로, 이를 통해 해당 부분을 확인하고 수정하면 된다.

---

## 💡 한 줄 요약

> 자식에서 부모로 데이터를 전달할 때는 **`emit`으로 커스텀 이벤트를 발신**하고, 부모는 **`v-on(@)`으로 수신**하여 처리한다.

---

## ❓ 더 찾아볼 것

- `v-model`을 컴포넌트에 적용하는 방법 (Props + Emit 조합)
- Pinia를 이용한 전역 상태 관리 (Prop Drilling 없이 컴포넌트 간 데이터 공유)
- `defineExpose`로 자식의 데이터/메서드를 부모에서 직접 접근하는 방법
