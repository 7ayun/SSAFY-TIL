# [Vue] Lifecycle Hooks

---

## 1. Lifecycle Hooks란?

Vue 컴포넌트가 **생성되고, DOM에 마운트되고, 업데이트되고, 소멸되는** 각 생애 주기 단계에서 실행되도록 제공되는 함수다.

> Hook(갈고리) → 특정 시점에 걸어서 내가 원하는 로직을 실행시킨다

사람으로 따지면, 초등학교 입학 때 입학 축하 메시지를 보내고, 중학교 입학 때 새 가방을 사는 것처럼, **특정 시점에만 할 수 있는 행동**이 있다. Vue도 컴포넌트의 각 시점에 이런 훅을 사용할 수 있다.

### 왜 시점이 중요한가?

```javascript
const app = createApp({ ... })
app.mount('#app')
```

- `createApp()` 실행 시점: Vue 인스턴스가 생성되지만, **아직 HTML에 접근 불가**
- `app.mount('#app')` 실행 후: 비로소 Vue와 HTML이 연결되어 **DOM 조작 가능**

> 마운트 전에 새로고침하면 `{{ count }}`처럼 원본 문법이 잠깐 보이는 **깜빡임 현상**이 생기는데, 이는 마운트 전 시점이 보이다가 마운트 후 Vue가 반영하기 때문이다.

---

## 2. 주요 Lifecycle Hooks

가장 일반적으로 사용되는 것은 `onMounted`, `onUpdated`, `onUnmounted`다.

모든 Lifecycle Hook은 컴포넌트의 `setup()` 단계 중 **동기적**으로 호출되어야 한다.

참고: https://vuejs.org/api/composition-api-lifecycle.html

### Lifecycle Hooks Diagram

```
Renderer encounters component
         ↓
    setup() [Composition API]
         ↓
    beforeCreate → created
         ↓
  Has pre-compiled template?
    YES → beforeMount
     NO → Compile template → beforeMount
         ↓
  initial render, create & insert DOM nodes
         ↓
      onMounted  ← 여기서 주로 초기 데이터 요청!
         ↓
  when data changes → beforeUpdate → re-render → onUpdated
         ↓
  when component is unmounted
         ↓
    beforeUnmount → onUnmounted
```

### onMounted

Vue 컴포넌트 인스턴스가 **초기 렌더링 및 DOM 요소 생성이 완료된 후** 특정 로직을 수행할 때 사용한다.

```javascript
const { createApp, ref, onMounted } = Vue

setup() {
  onMounted(() => {
    console.log('mounted')
    // HTML과 연결된 이후이므로 안전하게 DOM 조작 가능
  })
}
```

### onUpdated

반응형 데이터의 변경으로 인해 컴포넌트의 **DOM이 업데이트된 후** 특정 로직을 수행할 때 사용한다.

```javascript
const { createApp, ref, onMounted, onUpdated } = Vue

const count = ref(0)
const message = ref(null)

onUpdated(() => {
  message.value = 'updated!'
})
```

---

## 3. Lifecycle Hooks 활용 — Cat API 예시

`onMounted`를 활용해 컴포넌트가 마운트될 때 외부 API에 요청을 보내는 패턴  
→ **"마운트 되고 나서 데이터를 받아온다"**가 가장 일반적인 사용법이다.

```javascript
const API_URL = 'https://api.thecatapi.com/v1/images/search'

const app = createApp({
  setup() {
    const imgUrl = ref(null)

    const getCatImage = function () {
      axios({ method: 'get', url: API_URL })
        .then((response) => {
          imgUrl.value = response.data[0].url
        })
        .catch((error) => {
          console.log('실패했다옹', error)
        })
    }

    onMounted(() => {
      getCatImage()  // ✅ 마운트 후 안정적으로 실행
    })

    // ❌ 아래처럼 onMounted 외부에서 바로 호출하면 불안정
    // getCatImage()  // 될 때도, 안 될 때도 있음

    return { imgUrl, getCatImage }
  }
})
```

```html
<div id="app">
  <button @click="getCatImage">냥냥펀치</button>
  <!-- imgUrl 값이 있을 때만 img 태그 렌더링 (v-if 활용) -->
  <div v-if="imgUrl">
    <img :src="imgUrl" alt="랜덤 고양이 이미지">
  </div>
</div>
```

> **왜 onMounted 내부에서 호출해야 하나?**  
> onMounted 이전에 getCatImage를 호출하면, HTML과 연결되기 전이라 이미지 URL을 받아도 DOM을 업데이트하지 못할 수 있다. 안정적인 서비스를 위해 **확실한 시점(마운트 후)**에 실행해야 한다.

---

## 4. Lifecycle Hooks 주의사항

### 반드시 동기적으로 작성해야 함

Lifecycle Hooks는 반드시 **동기적**으로 작성해야 한다.  
Vue는 컴포넌트가 초기화될 때 모든 Hooks를 한 번에 스캔하고 준비하기 때문이다.

생의 주기는 자연스럽게 흘러가는 것이지, 우리가 억지로 미룰 수 없다.

```javascript
// ❌ 비동기로 작성 — 실행되지 않음
setTimeout(() => {
  onMounted(() => {
    console.log('이 코드는 실행되지 않습니다!')
  })
}, 100)
```

비동기로 작성하면 Vue가 해당 훅을 인식하지 못해, 원래 의도한 타이밍에 실행되지 않는다.

---

## 5. Vue Style Guide

Vue의 스타일 가이드 규칙은 우선순위에 따라 4가지 범주로 나뉜다.

| 우선순위 | 분류 | 설명 |
|---------|------|------|
| A | 필수 (Essential) | 오류를 방지하므로 어떤 경우에도 준수 |
| B | 적극 권장 (Strongly Recommended) | 가독성 및 개발자 경험 향상, 정당한 사유가 있어야 위반 가능 |
| C | 권장 (Recommended) | 일관성을 위한 임의 선택 가능 |
| D | 주의 필요 (Use with Caution) | 잠재적 위험 특성 고려 |

오늘 학습한 내용 중 **우선순위 A(필수)** 규칙:
1. `v-for`에 **key** 작성하기
2. 동일 요소에 `v-if`와 `v-for` **함께 사용하지 않기**

> 앞으로 v-for와 key는 항상 같이 쓰고, v-if와 v-for는 동일 선상에 작성하지 않는다.

참고: https://vuejs.org/style-guide/

---

## 💡 한 줄 요약
> Lifecycle Hooks는 컴포넌트의 특정 시점(마운트, 업데이트 등)에 코드를 실행하는 함수로, `onMounted`에서 초기 API 데이터를 불러오는 패턴이 가장 대표적이다.

## ❓ 더 찾아볼 것
- `onBeforeMount`, `onBeforeUpdate`, `onBeforeUnmount` 활용 사례
- `onErrorCaptured` — 자식 컴포넌트 오류 처리
- Options API의 `mounted`, `updated`와 Composition API의 `onMounted`, `onUpdated` 비교
- 컴포넌트 간 라이프사이클 실행 순서 (부모-자식 관계)
