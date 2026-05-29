# [Vue] Watchers

---

## 1. watch()

`watch()`는 **하나 이상의 반응형 데이터를 감시하고, 감시하는 데이터가 변경되면 콜백 함수를 호출**하는 함수다.

> watch = "보다, 감시하다" → CCTV처럼 반응형 변수를 24시간 감시하는 **감시자** 역할

- CCTV가 침입자를 감지하면 112에 신고하는 것처럼, 반응형 변수가 변경되면 지정된 행동(콜백 함수)을 실행한다.
- 새로운 값을 계산하는 `computed`와 달리, `watch`는 데이터가 바뀔 때 **특정 행동(Side Effect)**을 수행하기 위해 사용한다.
- **콜백 함수에 `return`은 필요 없다.** 행동이 목적이지, 계산이 목적이 아니다.

### watch 구조

```javascript
watch(source, (newValue, oldValue) => {
  // do something
})
```

| 인자 | 설명 |
|------|------|
| `source` (첫 번째) | watch가 감시하는 대상 (반응형 변수, 값을 반환하는 함수 등) |
| callback (두 번째) | source가 변경될 때 호출되는 콜백 함수 |
| `newValue` | 감시하는 대상이 변화된 값 |
| `oldValue` (optional) | 감시하는 대상의 기존(변화 전) 값 |

### watch 기본 동작

```javascript
const count = ref(0)

watch(count, (newValue, oldValue) => {
  console.log(`newValue: ${newValue}, oldValue: ${oldValue}`)
})
```

```html
<button @click="count++">Add 1</button>
<p>Count: {{ count }}</p>
```

버튼을 누를 때마다 count 값이 바뀌고, watch는 그 변화를 감지하여 즉시 콜백 함수를 실행한다.

```
newValue: 1, oldValue: 0
newValue: 2, oldValue: 1
newValue: 3, oldValue: 2
```

### watch 활용 예시 — 입력값 길이 추적

감시하는 변수에 변화가 생겼을 때 연관 데이터를 업데이트하는 패턴

```javascript
const message = ref('')
const messageLength = ref(0)

watch(message, (newValue) => {
  messageLength.value = newValue.length
  // return 필요 없음! 행동이 목적이므로
})
```

```html
<input v-model="message">
<p>Message length: {{ messageLength }}</p>
```

### 여러 source를 감시할 수 있는 watch

배열을 활용하여 여러 대상을 한 번에 감시할 수 있다.

```javascript
watch([foo, bar], ([newFoo, newBar], [prevFoo, prevBar]) => {
  /* ... */
})
```

> **TIP**: 여러 소스를 감시할 때, 콜백의 인자(새 값, 이전 값)도 같은 순서의 '배열'로 전달된다.  
> 배열 속 ref(객체)의 내부까지 감시하려면, `{ deep: true }` 옵션을 추가로 설정해야 한다.

---

## 2. computed vs. watch 비교

> **computed**: 값이 변경되면 → 그 값을 **계산해서 어딘가에 저장** (반드시 return 필요)  
> **watch**: 값이 변경되면 → **특정 행동을 실행** (return 불필요, 트리거 역할)

`computed`와 `watch` 모두 의존(감시)하는 원본 데이터를 **직접 변경하지 않는다**.

| 구분 | Computed | Watchers |
|------|----------|----------|
| 공통점 | 데이터의 변화를 감지하고 처리 | 데이터의 변화를 감지하고 처리 |
| 동작 | 의존하는 데이터 속성의 계산된 값을 **반환** | 특정 데이터 속성의 변화를 감지하고 **작업을 수행** (side-effects) |
| 사용 목적 | 계산한 값을 **캐싱**하여 재사용, 중복 계산 방지 | 데이터 변화에 따른 **특정 작업**을 수행 |
| 사용 예시 | 연산된 길이, 필터링된 목록 계산 등 | DOM 변경, 다른 비동기 작업 수행, 외부 API와 연동 등 |
| return | **필수** | 불필요 |

> 만약 watch 안에서 어떤 값을 계산한 다음 어딘가에 저장하고 싶다? → 그건 **computed**를 쓰면 된다.

---

## 💡 한 줄 요약
> `watch`는 CCTV처럼 반응형 데이터의 변화를 감지하여 Side Effect(API 호출, DOM 조작 등)를 수행하는 트리거이며, 값 계산이 목적이라면 `computed`를 사용한다.

## ❓ 더 찾아볼 것
- `watchEffect`와 `watch`의 차이 (자동 의존성 추적 vs. 명시적 소스 지정)
- `{ immediate: true }` 옵션 — 컴포넌트 마운트 시 즉시 실행
- `{ deep: true }` 옵션 — 중첩 객체 내부까지 감시
- `watch`에서 반환하는 stop 함수로 감시 중단하기
