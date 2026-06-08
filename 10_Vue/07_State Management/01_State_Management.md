# [Vue] State Management

---

## 1. Props/Emit 방식의 한계

Vue 컴포넌트는 **State(상태) → View(뷰) → Actions(기능)** 의 단방향 데이터 흐름을 따른다.

- **State**: 앱 구동에 필요한 기본 데이터
- **View**: 상태를 선언적으로 매핑하여 시각화한 템플릿
- **Actions**: 뷰에서 사용자 입력에 대해 반응적으로 상태를 변경하는 함수

이 흐름은 컴포넌트 수가 적을 때는 잘 동작한다. 그러나 **컴포넌트 depth가 5단계 이상** 깊어질 경우 두 가지 문제가 생긴다.

**문제 1 – Props Drilling**
여러 뷰가 동일한 상태에 종속될 때, 공유 데이터를 공통 조상 컴포넌트로 끌어올린 뒤 props로 내려보내야 한다. 계층이 깊어질수록 `prop → prop → prop →` 연쇄가 길어지고, 중간 컴포넌트를 수정하다 props를 빠뜨리면 휴먼 에러가 발생한다.

**문제 2 – Emit Chain**
서로 다른 뷰의 기능이 동일한 상태를 변경해야 할 때, `emit → emit → emit →` 연쇄로 데이터를 올려보내야 한다. 어느 컴포넌트에서 상태가 변경되는지 역추적하기 어려워 유지보수가 힘들어진다.

---

## 2. 해결책 – 중앙 저장소(Store)

각 컴포넌트에서 개별로 데이터를 관리하는 대신, **모든 컴포넌트가 공유하는 중앙 데이터 저장소**를 두는 방식이다.

- 컴포넌트가 데이터를 소유하지 않고, **store에 저장**한다
- 어떤 컴포넌트든 계층 구조와 무관하게 **store에 직접 접근**해서 읽거나 쓴다
- props/emit 없이도 멀리 떨어진 컴포넌트끼리 상태를 공유할 수 있다

이 역할을 하는 Vue 공식 라이브러리가 바로 **Pinia**다.

---

## 3. Pinia란?

**Pinia** = Vue의 공식 상태 관리 라이브러리

> "Pinia는 여러 컴포넌트가 함께 사용해야 하는 공통 데이터를 중앙 저장소에서 통합 관리해주는 Vue의 공식 상태 관리 라이브러리입니다. Props나 emit으로 복잡하게 데이터를 전달할 필요 없이, 어떤 컴포넌트든 이 중앙 저장소에 직접 접근하여 데이터를 읽거나 수정할 수 있습니다."

**설치 방법** – Vite 프로젝트 생성 시 기능 선택 단계에서 체크하면 된다.

```bash
$ npm create vue@latest
```

설치 후 `src/stores/` 폴더가 자동 생성되고, 여기에 스토어 파일을 관리한다.

---

## 4. Pinia 구성 요소

Pinia의 store는 `defineStore()`로 정의하며, 6가지 구성 요소를 갖는다.

```js
// stores/counter.js
import { ref, computed } from 'vue'
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', () => {
  // ① state
  const count = ref(0)

  // ② getters
  const doubleCount = computed(() => count.value * 2)

  // ③ actions
  function increment() {
    count.value++
  }

  // ④ 반환 값
  return { count, doubleCount, increment }
})
```

### ① store
공통 데이터를 관리하는 중앙 저장소. 모든 컴포넌트가 공유하는 state와 기능이 작성된다.

- `defineStore()`의 반환 값을 담는 변수명은 **`use...Store` 패턴** 사용을 권장한다. (예: `useCounterStore`, `useUserStore`)
- 첫 번째 인자는 애플리케이션 전체에 걸쳐 사용하는 **store의 고유 ID** 문자열이다.

> "스토어의 성격에 따라 분리해서 사용해도 되고, 내용이 작으면 하나로 관리해도 된다. 어카운트 관련 정보는 어카운트 스토어를 생성하는 식으로."

### ② state
중앙 저장소에 저장되는 **반응형 상태(데이터)**.

- 컴포넌트의 `ref()`와 같은 역할을 한다.
- 값이 변경되면 이 데이터를 사용하는 모든 컴포넌트의 화면이 자동으로 업데이트된다.

### ③ getters
state를 기반으로 파생된 계산 값.

- 컴포넌트의 `computed()`와 똑같은 역할을 한다.
- 원본 데이터가 바뀔 때만 다시 계산한다.

### ④ actions
state를 변경하는 역할의 함수.

- 컴포넌트의 `methods`와 같은 역할을 한다.
- getters와 달리 **비동기 처리, API 호출** 등의 로직도 실행할 수 있다.

### ⑤ 반환 값 (return)
- Pinia store에서 상태들을 사용하려면 **반드시 return해야 한다.**
- store는 private한 상태 속성을 갖지 않는다 — 반환하지 않은 값은 외부에서 접근 불가.

### ⑥ plugin
애플리케이션 상태 관리에 필요한 추가 기능을 제공하는 도구나 모듈.

- 패키지 매니저로 설치 후 별도 설정을 통해 추가한다.
- 대표적인 예: **모든 스토어 상태를 자동으로 localStorage에 저장/복원**하는 플러그인
  - 등록 시 state 값이 변경될 때마다 브라우저의 localStorage에 자동 저장된다.
  - 페이지를 새로고침해도 값이 초기화되지 않고 유지된다.

---

## 5. Pinia 구성 요소 활용 (컴포넌트에서 사용)

컴포넌트에서 store를 사용하려면 **반드시 두 줄**이 필요하다.

```js
// App.vue (script setup)
import { useCounterStore } from '@/stores/counter'

const store = useCounterStore()
```

### state 접근

컴포넌트 깊이에 관계없이 store 인스턴스로 state에 직접 읽고 쓸 수 있다. store에 정의되지 않은 state는 컴포넌트에서 새로 추가할 수 없다.

```js
// script
console.log(store.count)
const newNumber = store.count + 1
```

```html
<!-- template -->
<p>state : {{ store.count }}</p>
```

### getters 접근

state처럼 직접 접근 가능하다. `computed()`이므로 함수처럼 호출하지 않아도 된다 — 이미 그 자체가 값이기 때문이다.

```js
console.log(store.doubleCount)
```

```html
<p>getters : {{ store.doubleCount }}</p>
```

### actions 호출

함수이므로 직접 호출하거나 이벤트와 연결해서 사용한다.

```js
// 직접 호출
store.increment()
```

```html
<!-- 이벤트 바인딩 -->
<button @click="store.increment()">+++</button>
```

### Vue DevTools로 확인

Vue DevTools의 **Pinia 탭**에서 각 store의 state와 getters 값을 실시간으로 확인할 수 있다.

---

## 💡 한 줄 요약
> 컴포넌트 계층이 깊어질수록 props/emit 전달이 복잡해지므로, Pinia라는 중앙 저장소를 두고 모든 컴포넌트가 직접 접근해 상태(state)를 공유한다.

## ❓ 더 찾아볼 것
- Vuex vs Pinia 차이점 (Vue 2 → Vue 3 마이그레이션 관점)
- Pinia Setup Store vs Options Store 형태 비교
- Pinia 공식 문서: https://pinia.vuejs.org/
