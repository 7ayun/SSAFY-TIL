# [Vue] Dynamically data binding

---

## 1. Template Syntax란?

Vue는 DOM을 컴포넌트 인스턴스의 데이터에 **선언적으로 바인딩**할 수 있는, HTML 기반 **템플릿 구문**을 사용한다.

- **선언적 바인딩**: JavaScript 데이터(상태)가 바뀌면 DOM(화면)이 알아서 업데이트되는 것
- **템플릿 구문**: HTML에 Vue만의 특별한 문법을 추가해서 사용하는 것

### Template Syntax 종류

| 종류 | 설명 |
|------|------|
| Text Interpolation | 이중 중괄호(`{{ }}`)로 텍스트 데이터 삽입 |
| Raw HTML | `v-html`로 실제 HTML 출력 |
| Attribute Bindings | `v-bind`로 HTML 속성에 데이터 바인딩 |
| JavaScript Expressions | 템플릿 내에서 JS 표현식 사용 |

---

## 2. Text Interpolation

데이터 바인딩의 가장 기본 형태로, **이중 중괄호(콧수염 구문)** 을 사용한다.

```html
<p>Message: {{ msg }}</p>
```

- 콧수염 구문은 해당 컴포넌트 인스턴스의 `msg` 속성 값으로 대체된다
- `msg` 속성이 변경될 때마다 자동으로 업데이트된다

> **ref가 필요한 경우**: 값이 보이는 것 자체는 `ref`와 상관없다. 단순히 출력만 하는 변수라면 `ref` 없이도 된다. **값이 변경되고, 그 변경이 화면에 반영되어야 할 때** `ref`로 감싸준다.

---

## 3. Raw HTML

콧수염 구문은 데이터를 일반 텍스트로 해석하기 때문에, 실제 HTML을 출력하려면 `v-html`을 사용해야 한다.

```html
<!-- ❌ 콧수염으로 쓰면 텍스트 그대로 출력됨 -->
<div>{{ rawHtml }}</div>

<!-- ✅ HTML 태그가 적용된 형태로 출력 -->
<div v-html="rawHtml"></div>
```

```js
const rawHtml = ref('<span style="color:red">This should be red.</span>')
```

---

## 4. JavaScript Expressions

Vue는 모든 데이터 바인딩 내에서 JavaScript 표현식의 모든 기능을 지원한다.

사용 가능한 위치 두 가지:
1. 콧수염 구문 내부
2. 모든 디렉티브의 속성 값 (`v-`로 시작하는 특수 속성)

```html
{{ number + 1 }}
{{ ok ? 'YES' : 'NO' }}
{{ message.split('').reverse().join('') }}

<div v-bind:id="`list-${id}`"></div>
```

> **주의**: 각 바인딩에는 **하나의 단일 표현식**만 포함될 수 있다. 표현식은 값으로 평가할 수 있는 코드 조각(`return` 뒤에 사용할 수 있는 코드)이어야 한다.

```html
<!-- ❌ 작동하지 않는 경우 -->
{{ const number = 1 }}           <!-- 선언식은 불가 -->
{{ if (ok) { return message } }} <!-- 제어문은 삼항 표현식으로 사용해야 함 -->
```

---

## 5. Directive (디렉티브)

`v-` 접두사를 가진 특수 속성을 **디렉티브**라고 한다. DOM 요소에 특정 반응형 동작을 적용하는 명령어다.

```
v-on:submit.prevent="onSubmit"
  ↑      ↑       ↑        ↑
 Name  Argument Modifiers Value
```

| 구성 요소 | 설명 |
|-----------|------|
| **Name** | Directive의 핵심 이름. `v-`로 시작하며 shorthand 사용 시 생략 가능 |
| **Argument** | Directive가 '무엇에 대해' 동작할지 알려주는 대상. 콜론(`:`)으로 표시 |
| **Modifiers** | 점(`.`)으로 표시되는 특별한 접미사. Directive의 기본 동작을 수정 |
| **Value** | Directive에 연결될 JavaScript 표현식 |

**특징**:
- 속성 값은 단일 JavaScript 표현식이어야 함 (`v-for`, `v-on` 제외)
- 표현식 값이 변경될 때 DOM에 반응적으로 업데이트 적용

**동적 인자(Dynamic Argument)**: 대괄호로 감싸면 속성명 자체를 동적으로 바꿀 수 있다.

```html
<!-- key 변수 값에 따라 속성명이 바뀜 -->
<button :[key]="myValue"></button>
```

> 대괄호 안의 이름은 반드시 소문자만 가능 (브라우저가 속성 이름을 소문자로 강제 변환하기 때문)  
> 대괄호 안의 값이 `null`이면 해당 속성이나 이벤트 리스너가 아예 제거됨

**주요 Built-in Directives**: `v-text`, `v-show`, `v-if`, `v-for` 등  
참고: https://vuejs.org/api/built-in-directives.html

---

## 6. v-bind (속성 바인딩)

`v-bind`는 HTML 태그의 속성을 Vue의 데이터와 실시간으로 연결해 동적으로 제어하는 directive다.

> 뷰에서 가장 많이 사용하는 디렉티브 중 하나. 이걸 모르면 Vue를 개발할 수 없다.

- 정의: **하나 이상의 속성 또는 컴포넌트 데이터를 표현식에 동적으로 바인딩**
- **shorthand(약어)**: `v-bind:` → `:`(colon)

```html
<!-- 전체 문법 -->
<img v-bind:src="imageSrc">
<a v-bind:href="myUrl">Move to url</a>

<!-- shorthand (실무에서는 이 형태를 주로 사용) -->
<img :src="imageSrc">
<a :href="myUrl">Move to url</a>
```

> 바인딩 값이 `null`이나 `undefined`인 경우 해당 속성은 렌더링 요소에서 제거된다.

---

## 7. Attribute Bindings (속성 바인딩) 예시

```html
<img v-bind:src="imageSrc">
<a v-bind:href="myUrl">Move to url</a>
<img :src="imageSrc">
<a :href="myUrl">Move to url</a>
<p :[dynamicattr]="dynamicValue">…</p>
```

```js
const app = createApp({
  setup() {
    const imageSrc = ref('https://picsum.photos/200')
    const myUrl = ref('https://www.google.co.kr/')
    const dynamicattr = ref('title')
    const dynamicValue = ref('Hello Vue.js')
    return { imageSrc, myUrl, dynamicattr, dynamicValue }
  }
})
app.mount('#app')
```

---

## 8. Class and Style Bindings (클래스와 스타일 바인딩)

`class`와 `style`은 HTML 속성이므로 `v-bind`를 사용해 동적으로 값을 할당할 수 있다.  
Vue는 class 및 style 속성을 `v-bind`로 사용할 때 **객체 또는 배열**을 활용해 작성할 수 있도록 한다.

> **실용 예시**: 다크 모드/라이트 모드 토글 시 CSS 클래스를 미리 준비해두고, Boolean 값으로 클래스를 붙였다 뗐다 하면 편리하게 구현 가능.

### 1.1 Binding HTML Classes: Binding to Objects

객체를 `:class`에 전달하여 클래스를 동적으로 전환할 수 있다. 클래스명이 Boolean 값에 의해 추가되거나 제거된다.

```js
// 예시 1: Boolean 값으로 클래스 결정
const isActive = ref(true)
```
```html
<div :class="{ active: isActive }">Text</div>
<!-- 결과: <div class="active">Text</div> -->
```

```js
// 예시 2: 일반 클래스 속성과 함께 사용 + 여러 클래스
// ※ 'text-primary'처럼 - 가 있는 경우 따옴표로 감싸야 함 (JS에서 빼기 연산으로 인식하기 때문)
const isActive = ref(false)
const hasInfo = ref(true)
```
```html
<div class="static" :class="{ active: isActive, 'text-primary': hasInfo }">Text</div>
<!-- 결과: <div class="static text-primary">Text</div> -->
```

```js
// 예시 3: 반응형 변수로 객체를 한번에 작성 (가독성 향상)
const classObj = ref({
  active: isActive,
  'text-primary': hasInfo
})
```
```html
<div class="static" :class="classObj">Text</div>
```

### 1.2 Binding HTML Classes: Binding to Arrays

`:class`를 배열에 바인딩하여 여러 클래스를 한 번에 적용할 수 있다. 변수 값을 바꾸면 클래스명도 함께 변경된다.

```js
const activeClass = ref('active')
const infoClass = ref('text-primary')
```
```html
<!-- 배열 바인딩: 여러 개 동시 적용 -->
<div :class="[activeClass, infoClass]">Text</div>
<!-- 결과: <div class="active text-primary">Text</div> -->

<!-- 배열 내 객체 구문 사용: 넣을지 말지 + 값 변경 함께 활용 -->
<div :class="[{ active: isActive }, infoClass]">Text</div>
```

### 2.1 Binding Inline Styles: Binding to Objects

`:style`은 HTML의 `style` 속성에 JavaScript 객체를 바인딩한다. CSS 속성명을 camelCase로 작성하면 **브라우저가 알아서 kebab-case로 변환**해준다.

```js
const activeColor = ref('crimson')
const fontSize = ref(50)
```
```html
<!-- camelCase 사용 (권장) - 브라우저가 font-size로 자동 변환 -->
<div :style="{ color: activeColor, fontSize: fontSize + 'px' }">Text</div>

<!-- kebab-cased 키도 지원 (따옴표로 감싸야 함) -->
<div :style="{ color: activeColor, 'font-size': fontSize + 'px' }">Text</div>
```

> **주의**: 객체 내에서 `fontSize.value + 'px'`처럼 추가 계산이 필요한 경우 `.value`를 반드시 명시해야 한다. `.value`를 빠뜨리면 ref 객체 자체가 전달되어 `"[object Object]px"` 같은 오동작이 발생한다.

반응형 변수로 객체를 한번에 작성:
```js
const styleObj = ref({
  color: activeColor,
  fontSize: fontSize.value + 'px'  // 추가 계산이 있으므로 .value 명시
})
```
```html
<div :style="styleObj">Text</div>
```

### 2.2 Binding Inline Styles: Binding to Arrays

여러 스타일 객체를 배열에 작성해서 `:style`을 바인딩할 수 있다. 객체들은 병합되어 동일한 요소에 적용된다.

```js
const styleObj2 = ref({
  color: 'blue',
  border: '1px solid black'
})
```
```html
<div :style="[styleObj, styleObj2]">Text</div>
<!-- 결과: <div style="color: blue; font-size: 50px; border: 1px solid black;"> -->
```

> **클래스 vs 스타일 바인딩 차이**: 클래스는 클래스명을 넣을지/말지(Boolean) 또는 클래스명 자체를 변경하는 방식이고, 스타일은 CSS 속성-값 형태의 객체를 전달한다는 차이가 있다. 기본적인 객체/배열 활용 방식은 동일하다.

---

## 💡 한 줄 요약

> `v-bind`(`:`)로 HTML 속성을 Vue 데이터와 실시간 연결하고, 객체/배열 문법으로 class·style을 동적으로 제어한다.

## ❓ 더 찾아볼 것

- `v-if`, `v-for`, `v-show` 등 나머지 Built-in Directives (내일 학습 예정)
- Vue DevTools 설치 및 활용법 (Chrome 확장 프로그램)
- Vue 공식 Directives API 문서: https://vuejs.org/api/built-in-directives.html#v-bind
- `v-bind` 없이 속성 이름과 바인딩 변수명이 같을 때 사용하는 shorthand (Vue 3.4+)
