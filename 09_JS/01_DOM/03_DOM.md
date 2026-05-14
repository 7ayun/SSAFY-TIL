# [JS] DOM

---

## 1. 브라우저에서 JavaScript를 사용하는 방법

JavaScript를 웹 브라우저에서 실행하는 방법은 세 가지다.

```html
<!-- 방법 1: HTML script 태그 내부에 직접 작성 -->
<body>
  <script>
    console.log('hello')
  </script>
</body>
```

```html
<!-- 방법 2: .js 파일로 분리하여 불러오기 (실무에서 가장 많이 사용) -->
<!-- hello.js -->
console.log('hello')

<!-- HTML에서 불러오기 -->
<body>
  <script src="hello.js"></script>
</body>
```

```
방법 3: 브라우저 개발자 도구 Console 탭에서 직접 입력 (F12)
```

> 실무에서는 **방법 2(.js 파일 분리)**를 주로 사용한다. 오늘 수업에서는 브라우저 Console에서 직접 실행하며 변화를 눈으로 확인한다.

---

## 2. DOM(Document Object Model)이란?

### 문서 구조

HTML 문서는 여러 상자가 중첩된 구조를 갖는다.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Document</title>
  </head>
  <body>
    <h1>Heading</h1>
    <p>Hello,
      <a href="https://www.google.com/">google</a>
    </p>
  </body>
</html>
```

이 구조는 오른쪽과 같이 **트리(Tree) 형태**로 표현된다.

```
html
├── head
│   └── title
└── body
    ├── h1
    └── p
        └── a
```

각 태그 하나하나가 **객체(Object)**이며, 이 객체들이 서로 상호작용하면서 "어떤 태그인지, 어떤 내용이 담겼는지"를 의미한다.

이렇게 HTML 문서를 **구조화된 객체의 모델(Document Object Model)**로 표현한 것이 **DOM**이다.

### DOM의 역할

- 웹 페이지(Document)를 구조화된 객체로 제공
- JavaScript가 이 객체에 접근하여 **문서 구조, 스타일, 내용을 변경**할 수 있도록 해주는 API

### DOM API

JavaScript(JS)와 HTML(DOM)은 서로 다른 시스템이다. 서로 다른 시스템이 의사소통하려면 **API(Application Programming Interface)**가 필요하다. DOM API는 JS가 DOM에 접근하고 조작할 수 있도록 제공하는 메서드 모음이다.

> JavaScript의 목적 = **DOM을 조작해 웹 페이지를 동적으로 만드는 것**

---

## 3. document 객체

`document` 객체는 **DOM 트리의 최상위 객체**이자, HTML 문서에 접근하는 **진입점(Entry Point)**이다.

DOM에서 모든 요소, 속성, 텍스트는 하나의 객체이며, 모두 `document` 객체의 하위 객체로 구성된다.

```js
// 브라우저 Console에서 실행
document          // 현재 페이지 전체 DOM 반환
document.title    // 현재 탭의 타이틀 조회

document.title = 'Hello :)'  // 탭 제목을 'Hello :)'로 변경
```

> ⚠️ **주의:** 이렇게 바꾼 내용은 새로고침하면 모두 사라진다. 실제 서버 데이터를 바꾼 것이 아니라 브라우저가 렌더링한 화면만 바꾼 것이기 때문이다.

---

## 4. DOM Tree와 Node

DOM이 트리 구조이므로 각 구성 요소를 **Node(노드)**라고 부른다.

| 노드 종류 | 설명 | 예시 |
|-----------|------|------|
| **Document Node** | HTML 문서 전체를 나타내는 최상위 노드 | `document` |
| **Element Node** | HTML 요소를 나타내는 노드 | `<p>`, `<div>`, `<span>` |
| **Text Node** | Element 내의 텍스트 컨텐츠 | `<p>` 안의 텍스트 |
| **Attribute Node** | HTML 요소의 속성 | `href`, `class`, `id` |

> Element는 Node의 하위 유형이다. 모든 Element는 Node이지만, 모든 Node가 Element인 것은 아니다.

---

## 5. DOM 선택 (querySelector)

DOM을 조작하려면 먼저 **조작할 요소를 선택**해야 한다.

```
조작 순서: 요소를 선택 → 선택된 요소의 속성/내용을 조작
```

### `document.querySelector(selector)`

- 선택자와 일치하는 **첫 번째 요소 하나**를 반환
- 없으면 `null` 반환

```js
document.querySelector('.heading')   // class가 'heading'인 첫 번째 요소
document.querySelector('#title')     // id가 'title'인 요소
document.querySelector('a')          // 첫 번째 a 태그
```

### `document.querySelectorAll(selector)`

- 선택자와 일치하는 **모든 요소**를 `NodeList` 형태로 반환

```js
document.querySelectorAll('.content')     // class가 'content'인 모든 요소
document.querySelectorAll('ul > li')      // ul 바로 아래 모든 li 요소
```

### NodeList

`querySelectorAll()`이 반환하는 배열 유사 객체.
- 인덱스로 각 항목 접근 가능 (`nodeList[0]`, `nodeList[1]`)
- ⚠️ 나중에 DOM이 변경돼도 **NodeList 값은 바뀌지 않는다** (초기 선택 당시 값 유지)

```js
// 선택자 예시
const h1Tag = document.querySelector('.heading')
const items = document.querySelectorAll('.content')

console.log(h1Tag)   // <h1 class="heading">...</h1>
console.log(items)   // NodeList(3) [p.content, p.content, p.content]
console.log(items[1]) // 두 번째 .content 요소
```

> **CSS 선택자 복습:** 클래스는 `.클래스명`, ID는 `#아이디명`, 자식 요소는 `부모 > 자식`

---

## 6. DOM 조작

선택한 요소를 실제로 변경한다. 조작 방식은 크게 4가지다.

### 6-1. 클래스(Class) 속성 조작 — `classList`

스타일 변경 시 직접 CSS를 건드리는 것이 아니라 **클래스를 추가/제거**하는 방식을 사용한다. (실무에서 가장 많이 사용)

`classList` 프로퍼티는 요소의 클래스 목록을 `DOMTokenList` 형태로 반환하며, 다음 메서드를 제공한다.

| 메서드 | 설명 |
|--------|------|
| `element.classList.add('클래스명')` | 지정한 클래스를 추가 |
| `element.classList.remove('클래스명')` | 지정한 클래스를 제거 |
| `element.classList.toggle('클래스명')` | 있으면 제거, 없으면 추가 |

```js
const h1Tag = document.querySelector('.heading')

h1Tag.classList.add('red')     // 'red' 클래스 추가 → CSS 스타일 적용됨
h1Tag.classList.remove('red')  // 'red' 클래스 제거
h1Tag.classList.toggle('red')  // 있으면 제거, 없으면 추가
```

> **실제 활용:** 다크모드 버튼을 누를 때 `body.classList.toggle('dark-mode')` 한 줄로 전체 테마를 전환할 수 있다.

### 6-2. 일반 속성(Attribute) 조작

`class` 외에 `id`, `href` 등 HTML 속성 값을 직접 설정·조회·제거한다.

| 메서드 | 설명 |
|--------|------|
| `Element.getAttribute('속성명')` | 해당 속성의 **초기 HTML 값** 반환 |
| `Element.setAttribute('속성명', '값')` | 해당 속성 값을 설정 (없으면 새로 추가) |
| `Element.removeAttribute('속성명')` | 해당 속성을 제거 |

```js
const aTag = document.querySelector('a')

console.log(aTag.getAttribute('href'))          // 'https://www.google.com/'
aTag.setAttribute('href', 'https://www.naver.com/')  // href를 네이버로 변경
aTag.removeAttribute('href')                    // href 속성 완전 제거 → 링크 사라짐
```

> ⚠️ `getAttribute()`는 HTML 초기 값을 반환한다. JS로 값을 변경한 후 `.value` 같은 프로퍼티는 현재 상태를 반환하지만, `getAttribute()`는 여전히 초기 값을 반환하니 주의.

### 6-3. HTML 콘텐츠 조작 — `textContent`

요소 내부의 **순수한 텍스트 내용**을 읽거나 변경한다. HTML 태그는 완전히 제거하고 텍스트 데이터만 다룬다.

```js
const h1Tag = document.querySelector('.heading')

console.log(h1Tag.textContent)       // 'DOM 조작' (현재 텍스트)
h1Tag.textContent = '내용 수정'      // 텍스트를 '내용 수정'으로 변경
```

### 6-4. DOM 요소 자체 조작 (추가 / 삭제)

기존 요소의 속성을 바꾸는 것이 아니라, **DOM 트리에 새 요소를 추가하거나 기존 요소를 삭제**한다.

| 메서드 | 설명 |
|--------|------|
| `document.createElement('tagName')` | 지정한 태그의 HTML 요소를 **메모리에** 생성 |
| `Node.appendChild(노드)` | 특정 부모 노드의 **마지막 자식**으로 삽입 |
| `Node.removeChild(노드)` | DOM에서 자식 노드를 제거 |

```js
// 생성 → 내용 채우기 → DOM에 삽입
const h1Tag = document.createElement('h1')   // 빈 h1 태그 메모리에 생성
h1Tag.textContent = '제목'                   // 내용 채우기

const divTag = document.querySelector('div')
divTag.appendChild(h1Tag)                    // div의 자식으로 삽입 → 화면에 표시

// 삭제
const pTag = document.querySelector('p')
divTag.removeChild(pTag)                     // p 태그를 div에서 제거
```

> **중요:** `createElement()`는 메모리에만 요소를 생성한다. 화면에 나타나게 하려면 반드시 `appendChild()`로 DOM에 삽입해야 한다.

---

## 7. Style 조작 — `style` property

요소의 CSS 스타일을 직접 변경할 수 있다. 단, **실무에서는 거의 사용하지 않는다.**

```js
const pTag = document.querySelector('p')

pTag.style.color = 'crimson'
pTag.style.fontSize = '2rem'
pTag.style.border = '1px solid black'
```

> 이렇게 직접 스타일을 박는 것은 좋지 않은 습관이다. 스타일 변경은 클래스를 만들어서 `classList.add()`로 적용하는 것이 올바른 방법이다.

---

## 8. DOM 속성 확인 팁

개발자 도구(F12) → **Elements** 탭 → 요소 선택 → **Properties** 패널에서 해당 요소의 모든 DOM 속성을 시각적으로 확인할 수 있다.

---

## 9. 세미콜론 (참고)

JavaScript는 문장 끝에 세미콜론(`;`)을 **선택적으로** 사용할 수 있다.

- 없으면 **ASI(Automatic Semicolon Insertion)** 규칙에 따라 자동으로 삽입됨
- JavaScript를 만든 Brendan Eich도 세미콜론을 강제하지 않는 스타일을 선호

> 그러나 자동 삽입이 의도치 않은 방향으로 동작할 수 있다. 가장 중요한 것은 **팀의 스타일 가이드를 따르는 것**이다.

**자동 삽입으로 인한 버그 예시:**

```js
// ❌ return 바로 뒤에 줄바꿈 → 자동으로 세미콜론 삽입 → undefined 반환
function getObject() {
  return    // ← 여기서 자동으로 세미콜론이 삽입됨
  {
    name: 'ssafy'
  }
}

// ✅ 올바른 작성법
function getObject() {
  return {
    name: 'ssafy'
  }
}
```

---

## 10. var와 호이스팅 (참고)

### `var`
- ES6 이전 변수 선언 키워드
- 재할당/재선언 모두 가능, 함수 스코프를 가짐
- 변수를 선언하지 않고 사용하면 자동으로 `var`로 선언됨
- **호이스팅** 문제 때문에 사용 권장하지 않음

### 호이스팅 (Hoisting)

코드가 실행되기 전, 변수 및 함수 선언문이 **스코프의 최상단으로 끌어올려지는 듯한 현상**이다.

```js
// 함수 선언문 — 호이스팅으로 선언 전에도 호출 가능
sayHello()   // 'Hello!' 출력 — 에러 없음

function sayHello() {
  console.log('Hello!')
}
```

**변수의 생성 단계:**  
선언(Declaration) → 초기화(Initialization) → 할당(Assignment)

| 키워드 | 호이스팅 | 초기화 시점 | TDZ | 비고 |
|--------|---------|------------|-----|------|
| `var` | O | 스코프 진입 시 (`undefined`) | X | 선언 전 접근 가능 — 권장 X |
| `let` / `const` | O | 실제 코드 선언 줄 도달 시 | O | 선언 전 접근 불가 (안전) |
| `function` | O | 스코프 진입 시 (함수 전체) | X | 선언 전 호출 가능 |
| `class` | O | 실제 코드 선언 줄 도달 시 | O | 선언 전 사용 불가 |

**TDZ(Temporal Dead Zone):** 스코프 시작 지점부터 실제 변수 초기화가 이루어지는 코드 줄까지의 구간. `let`/`const`/`class`는 호이스팅되지만 TDZ에 갇혀 접근 불가능하다.

```js
// var — TDZ 없음. 선언 전 접근 시 undefined 반환
console.log(name)  // undefined (에러 없음)
var name = '홍길동'

// let, const — TDZ 있음. 선언 전 접근 시 ReferenceError
console.log(age)   // ReferenceError: Cannot access 'age' before initialization
let age = 30
```

> `let`, `const`, `class`는 호이스팅이 되지만 TDZ에 갇혀 있기 때문에 선언 전에 접근할 수 없다. 이것이 개발자의 실수를 방지하는 안전장치 역할을 한다.

---

## 💡 한 줄 요약

> DOM은 HTML을 객체 트리로 관리하는 API이며, JavaScript는 `querySelector`로 요소를 선택하고 `classList`, `setAttribute`, `textContent`, `appendChild` 등으로 요소를 조작해 웹 페이지를 동적으로 만든다.

---

## ❓ 더 찾아볼 것

- `innerHTML` vs `textContent` 차이점 (XSS 보안 취약점 관련)
- classList MDN 문서 (https://developer.mozilla.org/ko/docs/Web/API/Element/classList)
- NodeList vs HTMLCollection 차이
- TDZ (Temporal Dead Zone) MDN 문서
- 이벤트(Event) — 다음 수업에서 학습 예정
- `console.log()` = 파이썬의 `print()` 역할
