# [Vue] Form Input Bindings

---

## 1. Form 입력 바인딩이란?

form을 처리할 때 사용자가 input에 입력하는 값을 실시간으로 JavaScript 상태에 동기화해야 하는 경우 → **양방향 바인딩(Two-way Binding)** 이 필요하다.

양방향 바인딩 방법 2가지:
1. `v-bind`와 `v-on`을 함께 사용
2. `v-model` 사용

---

## 2. v-bind와 v-on을 함께 사용

1. `v-bind`로 input 요소의 `value` 속성을 반응형 변수에 연결 (단방향)
2. `v-on`으로 `input` 이벤트가 발생할 때마다, 입력값을 반응형 변수에 저장

```js
const inputText1 = ref('')
const onInput = function (event) {
  inputText1.value = event.currentTarget.value
}
```
```html
<p>{{ inputText1 }}</p>
<input :value="inputText1" @input="onInput">
```

이 방식은 코드가 길어지고 함수를 별도로 작성해야 한다는 단점이 있다.

---

## 3. v-model

`v-model`은 `v-bind` + `v-on` 조합보다 훨씬 간결하게 양방향 바인딩을 구현하는 directive다.

- 정의: **form input 요소 또는 컴포넌트에서 양방향 바인딩을 만듦**
- 사용자의 입력이 즉시 데이터에 반영되고, 데이터의 변경이 즉시 화면에 반영됨

```js
const inputText2 = ref('')
```
```html
<p>{{ inputText2 }}</p>
<input v-model="inputText2">
```

**`v-bind` vs `v-model` 차이**:

| | v-bind | v-model |
|--|--------|---------|
| 방향 | 단방향 (변수 → DOM) | 양방향 (변수 ↔ DOM) |
| 사용자 입력 반영 | 별도 이벤트 핸들러 필요 | 자동 반영 |
| 사용 대상 | 모든 HTML 속성 | 사용자 입력 요소 (`input`, `textarea`, `select`) |

> 내가 입력하면 변수에도 적용되고, 변수의 값을 바꾸면 입력창에도 적용된다 = **양방향**

> **IME 주의**: IME가 필요한 언어(한국어, 중국어, 일본어 등)는 조합형 입력 방식 때문에 `v-model`이 **한 박자 늦게 반응**한다. 예를 들어 '하'를 입력 중일 때는 변수에 반영되지 않고, 다음 글자로 넘어가야 반영된다.  
> 이런 딜레이 없이 즉시 반영되길 원하면 `v-bind` + `v-on` 방식을 사용하면 된다.

---

## 4. v-model 활용

`v-model`은 단순 Text input 뿐만 아니라 다양한 타입의 사용자 입력 방식과 함께 사용 가능하다.

### Checkbox 활용

**1. 단일 체크박스와 boolean 값 활용**

```js
const checked = ref(false)
```
```html
<input type="checkbox" id="checkbox" v-model="checked">
<label for="checkbox">{{ checked }}</label>
```

→ 체크 시 `true`, 해제 시 `false`

**2. 여러 체크박스와 배열 활용**

초기 반응형 변수를 배열로 초기화하면 해당 배열에 현재 선택된 체크박스의 값이 포함된다.

```js
const checkedNames = ref([])
```
```html
<div>Checked names: {{ checkedNames }}</div>
<input type="checkbox" id="alice" value="Alice" v-model="checkedNames">
<label for="alice">Alice</label>
<input type="checkbox" id="bella" value="Bella" v-model="checkedNames">
<label for="bella">Bella</label>
```

→ Alice, Bella 모두 선택 시: `Checked names: [ "Alice", "Bella" ]`

### Select 활용

`v-model` 표현식의 초기 값이 어떤 option과도 일치하지 않는 경우 `select` 요소는 '선택되지 않은(unselected)' 상태로 렌더링된다.

```js
const selected = ref('')
```
```html
<div>Selected: {{ selected }}</div>
<select v-model="selected">
  <option disabled value="">Please select one</option>
  <option>Alice</option>
  <option>Bella</option>
  <option>Cathy</option>
</select>
```

참고: https://vuejs.org/api/built-in-directives.html#v-model

---

## 5. 참고: 접두어 `$`

`$` 접두어가 붙은 변수는 Vue 인스턴스 내에서 사용할 수 있도록 **Vue가 제공하는 공용 프로퍼티**다.

- 사용자가 지정한 반응형 변수나 메서드와 구분하기 위함
- 주로 Vue 인스턴스 내부 상태를 다룰 때 사용

```
예시: $event, $refs, $router, $store ...
```

> **주의사항**:
> - 내가 만드는 데이터와 메서드 이름에 `$`나 `_`(언더바) 접두사를 사용하지 않는 것이 좋다.
> - Vue가 내부적으로 사용하는 이름과 겹쳐서 덮어씌워지면 내부 동작이 제대로 되지 않을 수 있다.
> - `_`로 시작하는 속성은 내부용이므로 직접 사용하면 안 되고, 예고 없이 변경될 수 있다.

---

## 6. 참고: IME (Input Method Editor)

IME는 사용자가 입력 장치에서 기본적으로 사용할 수 없는 문자(비영어권 언어)를 입력할 수 있도록 하는 운영 체제 구성 프로그램이다.

- 한국어, 중국어, 일본어 같은 **조합형 언어**에서 사용
- IME가 활성화된 상태(예: 한글 조합 중)에서 `v-model`의 업데이트 방식이 충돌하여 **한 박자 늦게 반영**될 수 있음

| 방법 | 한글 실시간 반영 | 코드 간결성 |
|------|----------------|-------------|
| `v-model` | ❌ (한 박자 늦음) | ✅ 간결 |
| `v-bind` + `v-on` | ✅ 즉시 반영 | ❌ 코드 길어짐 |
| `v-model.lazy` | ✅ (포커스 잃을 때 반영) | ✅ 간결 (단, 실시간 아님) |

---

## 💡 한 줄 요약

> `v-model`은 `v-bind`(단방향)와 `v-on`의 조합을 단순화한 양방향 바인딩 directive로, 폼 요소의 사용자 입력을 Vue 데이터와 실시간으로 동기화한다.

## ❓ 더 찾아볼 것

- `v-model` modifiers: `.lazy`, `.number`, `.trim`
- Radio 버튼에서의 `v-model` 활용
- Vue 공식 v-model API 문서: https://vuejs.org/api/built-in-directives.html#v-model
- `compositionstart` / `compositionend` 이벤트와 IME 처리
