# [Vue] Vue tutorial

---

## 1. Vue를 사용하는 2가지 방법

| 방법 | 설명 |
|------|------|
| **'CDN' 방식** | HTML 파일에 CDN 링크만 추가하여 사용. 이번 주 진행 (Vue 문법 자체에 집중) |
| **'NPM' 설치 방식** | Node.js를 통해 설치 후 사용. 다음 주 진행 (본격적인 프로젝트 구조) |

---

## 2. Vue Application 생성하기

### 1단계 — vue 사용을 위한 CDN 작성

```html
<!-- vue_instance.html -->

<div id="app"></div>

<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>

<script>
  const { createApp } = Vue

  const app = createApp({
    setup() {}
  })

  app.mount('#app')
</script>
```

### 2단계 — Application instance (createApp)

- CDN에서 Vue를 사용하는 경우 **전역 Vue 객체**를 불러오게 됨
- **구조분해할당** 문법으로 Vue 객체의 `createApp` 함수를 할당

```javascript
const { createApp } = Vue
```

### 3단계 — createApp으로 인스턴스 생성

- 모든 Vue 애플리케이션은 `createApp` 함수로 새 **Application instance**를 생성하는 것으로 시작함

```javascript
const app = createApp({
  setup() {}
})
```

### 4단계 — Root Component

- `createApp` 함수에는 **객체(컴포넌트)** 가 전달됨
- 모든 App에는 다른 컴포넌트들을 하위 컴포넌트로 포함할 수 있는 **Root(최상위) 컴포넌트**가 필요
- 현재는 단일 컴포넌트

### 5단계 — Mounting the App (앱 연결)

- HTML 요소에 Vue Application instance를 **탑재(연결)**
- 각 앱 인스턴스에 대해 `mount()`는 **한 번만** 호출할 수 있음

```javascript
app.mount('#app')
```

- `#app`: id가 `app`인 요소에 Vue 인스턴스를 연결
- `mount()` 된 요소 **내부에서만** Vue를 활용할 수 있음 (외부 요소에는 적용 안 됨)

### 6단계 — setup() 함수

- 컴포넌트가 동작하기 전에 미리 준비하는 **"시작점"**, "초기 설정용 함수"
- 이 함수 안에서 데이터를 정의하거나, 화면에 표시할 값을 계산하거나, 각종 로직(함수)을 준비할 수 있음
- `setup`에서 준비한 값들은 이후 **템플릿이나 컴포넌트의 다른 부분**에서 바로 사용 가능
- ⚠️ 반드시 **객체를 반환(return)** 해야 함

```javascript
const app = createApp({
  setup() {
    // 여기에 데이터, 함수, 로직 작성
    return {
      // 반환한 값들이 템플릿에서 사용 가능
    }
  }
})
```

---

## 3. 반응형 상태 (ref)

### ref() 함수

**반응형 상태(데이터)를 선언하는 함수** (Declaring Reactive State)

- 일반 JavaScript 변수를 Vue가 **변화를 감지할 수 있는 반응형 객체**로 만들어줌
- 컴포넌트 내에서 변하는 값(예: 숫자, 문자열, input 값 등)의 상태를 추적하고 관리하기 위해 사용

```javascript
const { createApp, ref } = Vue  // ref도 구조분해할당으로 가져옴

const app = createApp({
  setup() {
    const message = ref('Hello vue!')
    console.log(message)        // ref 객체 (RefImpl)
    console.log(message.value)  // Hello vue!
    
    return {
      message  // return에 포함해야 템플릿에서 사용 가능
    }
  }
})
```

### ref 함수의 특징

- `.value` 속성이 있는 ref 객체로 **래핑(wrapping)** 하여 반환하는 함수
- ref로 선언된 변수의 값이 변경되면, 해당 값을 사용하는 **템플릿에서 자동으로 업데이트**
- 인자는 어떠한 타입도 가능

### 🔑 핵심: JS와 템플릿에서의 접근 방식

| 위치 | 접근 방법 | 이유 |
|------|-----------|------|
| JavaScript 내부 | **반드시 `.value`** 사용 | ref는 객체이므로, 실제 값은 `.value` 키에 저장됨 |
| 템플릿 (콧수염 구문) | `.value` **생략 가능** | Vue가 자동으로 언래핑(unwrapping) 처리 |

```javascript
// JavaScript 내부 - .value 필수
const count = ref(0)
count.value++  // ⭕ 올바른 방법
count++        // ❌ 화면이 업데이트되지 않음
```

```html
<!-- 템플릿 내부 - .value 생략 -->
<h1>{{ message }}</h1>       <!-- ⭕ 자동 언래핑 -->
<h1>{{ message.value }}</h1> <!-- ⭕ 이것도 동작함 -->
```

---

## 4. Vue 기본 구조

### setup()의 return 규칙

`createApp()`에 전달되는 객체는 Vue 컴포넌트이며, **컴포넌트의 상태는 setup() 함수 내에서 선언되어야 하며 반드시 객체를 반환해야 함**

```javascript
const app = createApp({
  setup() {
    const message = ref('Hello vue!')
    
    return {  // ← 반드시 return 해야 템플릿에서 사용 가능!
      message
    }
  }
})
```

### 템플릿 렌더링 (Mustache Syntax)

- 반환된 객체의 속성은 **템플릿에서 사용**할 수 있음
- **Mustache syntax(콧수염 구문, `{{ }}`)** 를 사용하여 메시지 값을 기반으로 **동적 텍스트**를 렌더링

```html
<div id="app">
  <h1>{{ message }}</h1>
</div>
```

- 콘텐츠는 식별자나 경로에만 국한되지 않으며 **유효한 JavaScript 표현식**을 사용할 수 있음

```html
<!-- JS 표현식 사용 예시 -->
<h1>{{ message.split('').reverse().join('') }}</h1>
<p>{{ number + 1 }}</p>
<p>{{ ok ? 'YES' : 'NO' }}</p>
```

### Event Listeners in Vue (v-on)

- **`v-on`** 디렉티브를 사용하여 DOM 이벤트를 수신할 수 있음
- 함수 내에서 반응형 변수를 변경하여 구성 요소 상태를 업데이트

```html
<!-- HTML -->
<div id="app">
  <button v-on:click="increment">버튼</button>
  <p>{{ number }}</p>
</div>
```

```javascript
// JavaScript
const { createApp, ref } = Vue

const app = createApp({
  setup() {
    const number = ref(0)
    const increment = function () {
      number.value++  // JS 내부이므로 .value 필수!
    }
    return {
      number,
      increment  // 함수도 return에 포함해야 함
    }
  }
})
```

---

## 💡 한 줄 요약

> Vue 앱은 `createApp` → `setup()` → `return` → `mount()` 순으로 구성되며, 화면에 자동 업데이트되길 원하는 상태는 반드시 `ref()`로 선언하고 JS 내에서는 `.value`로 접근한다.

## ❓ 더 찾아볼 것

- Vue 디렉티브 전체 목록 (v-on, v-bind, v-if, v-for, v-model 등)
- `reactive()` vs `ref()` — Vue의 다른 반응형 API
- Vue Composition API의 생명주기 훅 (onMounted, onUpdated 등)
