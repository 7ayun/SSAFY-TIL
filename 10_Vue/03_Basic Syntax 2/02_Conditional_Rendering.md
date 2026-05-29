# [Vue] Conditional Rendering

---

## 1. v-if

`v-if`는 표현식 값의 **true/false를 기반으로 요소를 조건부로 렌더링**하는 Directive다.

> **Directive**: `v-` 접두사가 있는 특수 속성

조건이 거짓(false)이면, 해당 요소는 **DOM에서 완전히 제거**되어 보이지 않게 된다.  
주로 로그인 상태에 따라 다른 메뉴를 보여주거나, 특정 상황에만 경고 메시지를 표시하는 등 조건부 렌더링에 사용된다.

### v-if 기본 사용

```javascript
const isSeen = ref(true)
```

```html
<p v-if="isSeen">true일때 보여요</p>
```

### v-else

```html
<p v-if="isSeen">true일때 보여요</p>
<p v-else>false일때 보여요</p>
<button @click="isSeen = !isSeen">토글</button>
```

> **주의**: `v-if`와 `v-else` 사이에 `<br>` 같은 다른 요소가 들어오면 `v-else`가 동작하지 않는다!  
> `v-if` 다음에 오는 형제(sibling) 요소가 바로 `v-else`여야 한다.

```html
<!-- ❌ v-else가 동작하지 않음 -->
<p v-if="isSeen">true일때 보여요</p>
<br>  <!-- 이 요소가 끊어버린다 -->
<p v-else>false일때 보여요</p>

<!-- ✅ br을 자식 태그로 이동 -->
<p v-if="isSeen">true일때 보여요<br></p>
<p v-else>false일때 보여요</p>
```

### v-else-if

`v-else-if` directive를 사용하여 `v-if`에 대한 `else if` 블록을 나타낼 수 있다.

```javascript
const name = ref('Cathy')
```

```html
<div v-if="name === 'Alice'">Alice입니다</div>
<div v-else-if="name === 'Bella'">Bella입니다</div>
<div v-else-if="name === 'Cathy'">Cathy입니다</div>
<div v-else>아무도 아닙니다.</div>
```

> 알고 있는 if/else if/else 문 앞에 `v-`만 붙이면 된다.

### 여러 요소에 v-if 적용 — `<template>` 활용

`<template>` 요소에 `v-if`를 사용하면, 여러 요소를 하나의 조건부 블록으로 묶을 수 있다. (`v-else`, `v-else-if` 모두 적용 가능)

```html
<template v-if="name === 'Cathy'">
  <div>Cathy입니다</div>
  <div>나이는 30살입니다</div>
</template>
```

> **HTML `<template>` element**: 페이지 로드 시 렌더링되지 않지만 JavaScript로 활용 가능한 HTML을 담는 **보이지 않는 wrapper** 역할. v-if와 자주 함께 사용되며, `<template>` 태그 자체는 DOM에 나타나지 않는다.

---

## 2. v-show

`v-show`는 `v-if`와 비슷하게 특정 조건에 따라 HTML 요소를 보여주거나 숨기는 Directive다.

DOM에서 요소를 완전히 제거하는 `v-if`와 달리, `v-show`는 **CSS의 `display` 속성을 `none`으로 바꿔 화면에서만 보이지 않게 숨긴다.**

```javascript
const isShow = ref(false)
```

```html
<div v-show="isShow">v-show</div>
<!-- 실제 렌더링 결과: -->
<div style="display: none;">v-show</div>
```

- `v-show`를 사용한 요소는 조건과 관계없이 **항상 DOM에 렌더링**된다.
- 개발자 도구 Elements 탭에서 확인하면 태그가 존재하지만 `display:none` 스타일이 적용된 것을 볼 수 있다.

---

## 3. v-if vs. v-show 비교

| 구분 | v-if | v-show |
|------|------|--------|
| DOM 제거 여부 | 조건이 false면 DOM에서 **완전 제거** | 항상 DOM에 존재 (display: none) |
| 초기 렌더링 비용 | **낮음** (false면 아무 작업도 안 함) | **높음** (항상 렌더링) |
| 토글 비용 | **높음** (DOM 생성/제거 반복) | **낮음** (CSS 전환만) |
| else if 지원 | ✅ v-else-if 사용 가능 | ❌ v-show만 존재 |
| 별칭 | Cheap initial load, expensive toggle | Expensive initial load, cheap toggle |

> **콘텐츠를 매우 자주 전환해야 하는 경우 → `v-show`**  
> **조건이 여러 개이거나 실행 중 잘 바뀌지 않는 경우 → `v-if`**

---

## 💡 한 줄 요약
> `v-if`는 조건에 따라 DOM에서 요소를 완전히 제거하고, `v-show`는 CSS로만 가시성을 전환하므로, 토글 빈도에 따라 적절히 선택해야 한다.

## ❓ 더 찾아볼 것
- `v-if`와 `v-for`의 우선순위 충돌 문제 (동일 요소에 함께 사용 금지)
- `<transition>` 컴포넌트와 `v-if` 조합
- `v-show`가 `<template>`에는 사용 불가한 이유
