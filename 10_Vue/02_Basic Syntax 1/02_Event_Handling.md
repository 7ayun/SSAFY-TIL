# [Vue] Event Handling

---

## 1. v-on이란?

`v-on`은 DOM 요소에 이벤트 리스너를 연결 및 수신하는 directive다.  
버튼 클릭, 키보드 입력 등 사용자의 이벤트를 감지하고, 지정된 코드를 실행시킨다.

> 기억법: `@` 기호가 귀(👂)처럼 생겼다. 이벤트를 **듣는(listen)** 역할이라 직관적!

```html
<!-- 전체 문법 -->
<button v-on:click="handler">Button</button>

<!-- shorthand: '@' (실무에서 주로 사용) -->
<button @click="handler">Button</button>
```

### handler 종류

| 종류 | 설명 |
|------|------|
| **Inline handlers** | 이벤트가 트리거 될 때 실행될 JavaScript 코드 |
| **Method handlers** | 컴포넌트에 정의된 메서드 이름 |

---

## 2. Inline Handlers

주로 **간단한 로직**에 사용한다.

```js
const count = ref(0)
```
```html
<button @click="count++">Add 1</button>
<p>Count: {{ count }}</p>
```

**단점**:
- 복잡한 표현식이 들어가면 템플릿에서 코드를 이해하기 어려워짐
- 재사용이 불가능해 유지보수가 어려움
- HTML에 로직이 분산되면 휴먼 에러 발생 가능성이 높아짐

---

## 3. Method Handlers

`setup`에 정의된 메서드를 호출하는 방식으로, 로직이 복잡할 경우 Method를 분리하면 템플릿이 간결해지고 코드를 재사용하기 좋다. **강력하게 권장하는 방식**이다.

```js
const count = ref(0)
const increase = function () {
  count.value += 1
}
```
```html
<button @click="increase">Hello</button>
<p>Count: {{ count }}</p>
```

**괄호 없이 메서드 이름만 연결하면**, 핸들러의 첫 번째 인자로 DOM의 **event 객체가 자동으로 전달**된다.

```js
const name = ref('Alice')
const myFunc = function (event) {
  console.log(event)                        // PointerEvent 객체
  console.log(event.currentTarget)          // 이벤트가 발생한 DOM 요소 (<button>Hello</button>)
  console.log(`Hello ${name.value}!`)       // Hello Alice!
}
```
```html
<button @click="myFunc">Hello</button>
```

> **event 객체**: 이벤트 발생 시, 이벤트 상세 정보를 담아 전달되는 객체. 매개변수명은 `event`가 관례이지만 다른 이름도 무방하다.

---

## 4. 사용자 지정 인자 전달

특정 값을 인자로 전달하면 이벤트 객체 대신 해당 값이 매개변수에 들어간다.

```js
const greeting = function (message) {
  console.log(message)
}
```
```html
<button @click="greeting('hello')">Say hello</button>
<button @click="greeting('bye')">Say bye</button>
```

### Inline Handlers에서 event 객체도 함께 받기

`$event` 변수를 사용하여 명시적으로 전달한다. **전달하는 위치는 어디든 상관없다.**

```js
const warning = function (message, event) {
  console.log(message)
  console.log(event)
}

const danger = function (msg1, event, msg2) {
  console.log(msg1)    // '위험'
  console.log(event)   // PointerEvent 객체
  console.log(msg2)    // '합니다'
}
```
```html
<button @click="warning('경고입니다', $event)">Warning</button>
<!-- $event는 위치와 무관하게 이벤트 객체가 전달됨 -->
<button @click="danger('위험', $event, '합니다')">Danger</button>
```

> **`$`(달러 사인)이 붙은 변수**는 Vue 인스턴스 내에서 내부적으로 사용하는 공용 프로퍼티다. `$event`도 이 중 하나다.

---

## 5. Event Modifiers

Vue는 Event Modifiers를 제공하여, `event.preventDefault()`와 같은 코드를 메서드 안에 직접 작성할 필요가 없도록 한다. 메서드 로직을 순수하게 데이터 관련 처리에만 집중시키기 위함이다.

```html
<form @submit.prevent="onSubmit">...</form>
<a @click.stop.prevent="onLink">...</a>
```

> **Modifiers**: 디렉티브 뒤에 점(`.`)으로 붙여, 특별한 동작을 추가하는 기능

**주요 Event Modifiers**:

| Modifier | 대응 메서드 | 설명 |
|----------|-------------|------|
| `.stop` | `event.stopPropagation()` | 버블링(이벤트 전파) 중단 |
| `.prevent` | `event.preventDefault()` | 기본 동작 취소 (가장 많이 사용) |
| `.capture` | - | 캡처 모드로 이벤트 리스너 추가 |
| `.self` | - | 이벤트가 해당 요소에서 직접 발생한 경우에만 핸들러 실행 |
| `.once` | - | 핸들러를 최대 한 번만 실행 |
| `.passive` | - | `{ passive: true }` DOM 이벤트 첨부 |

> **Modifiers는 chained 되게 작성할 수 있으며, 이때는 작성된 순서로 실행된다** → 작성 순서에 유의!

### 버블링(Bubbling)이란?

한 요소의 이벤트가 부모, 조상 요소로 퍼져나가는 현상

**예시: `.prevent` vs `.stop.prevent` 차이**

```html
<div v-on:click="detectBubble">
  <!-- 1번: 기본 a 태그 → 클릭 시 onLink + 버블링으로 detectBubble 호출 + 구글 이동 -->
  <a href="https://www.google.com/">onLink</a><br>

  <!-- 2번: .prevent → 구글 이동 막음, but 버블링은 발생 (detectBubble 호출됨) -->
  <a @click.prevent="onLink" href="https://www.google.com/">onLink</a><br>

  <!-- 3번: .stop.prevent → 버블링 차단 + 이동 차단 → onLink만 실행됨 -->
  <a @click.stop.prevent="onLink" href="https://www.google.com/">onLink</a>
</div>
```

> 실무에서는 `.prevent` 하나만으로도 대부분 충분하다. `.stop`은 버블링을 막아야 하는 특수한 상황에서 사용한다.

---

## 6. Key Modifiers

키보드 이벤트를 수신할 때 특정 키에 관한 별도 modifiers를 사용할 수 있다.

```html
<!-- Enter 키가 입력되었을 때만 onSubmit 이벤트를 호출 -->
<input @keyup.enter="onSubmit">

<!-- Ctrl + Enter로 댓글 등록 -->
<textarea @keydown.ctrl.enter="submitComment"></textarea>
```

**keyup vs keydown 차이**:
- `keyup`: 키를 **뗐을 때** 이벤트 발생
- `keydown`: 키를 **누르는 순간** 이벤트 발생 (누르고 있는 상태 감지에 유용)

> 키보드 키코드 명칭은 MDN 문서에서 확인 가능. 이를 모디파이어로 붙여주면 해당 키가 입력될 때만 이벤트가 발생한다.

참고: https://vuejs.org/api/built-in-directives.html#v-on

---

## 💡 한 줄 요약

> `v-on`(`@`)으로 DOM 이벤트를 수신하고, Inline보다 Method handler를 선호하며, `.prevent` 등 Modifier로 이벤트의 기본 동작과 버블링을 제어한다.

## ❓ 더 찾아볼 것

- `event.stopPropagation()` vs `event.preventDefault()` 차이
- Vue의 시스템 수식어 키 (`.ctrl`, `.alt`, `.shift`, `.meta`)
- MDN 키보드 이벤트 키코드 목록
- Vue 공식 v-on API 문서: https://vuejs.org/api/built-in-directives.html#v-on
