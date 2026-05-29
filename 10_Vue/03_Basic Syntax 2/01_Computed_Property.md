# [Vue] Computed Property

---

## 1. computed()란?

`computed()`는 **"계산된 속성"을 정의하는 함수**다.

> compute(계산하다) + ed(된) → **계산된 속성**

미리 계산된 속성을 만들어 템플릿의 표현식을 단순하게 하고, 불필요한 반복 연산을 줄여준다.  
한 번 계산된 값은 **캐싱(임시 저장)**되어, 의존하는 데이터가 바뀌기 전까지는 다시 계산하지 않으므로 성능에 매우 유리하다.

### 캐시(Cache)가 중요한 이유

예를 들어 실행에 1.5초가 걸리는 함수 `myFunk()`를 3번 출력해야 할 때:

| 방법 | 소요 시간 |
|------|-----------|
| `myFunk()` 3번 호출 | 1.5초 × 3 = **4.5초** |
| 결과를 변수에 저장 후 3번 출력 | **1.5초** (1번만 호출) |

2번 방법이 훨씬 빠르지만, 문제는 `value1`이 수정되었을 때 수동으로 `myFunk()`를 다시 호출해야 한다. 이를 깜빡하면 **휴먼 에러**가 발생한다.

→ `computed`는 이 두 가지 장점을 모두 제공한다: **자동으로 재계산하면서 캐시도 활용**.

### computed가 없는 경우

```html
<h2>남은 할 일</h2>
<p>{{ todos.length > 0 ? '아직 남았다' : '퇴근!' }}</p>
```

- 템플릿이 복잡해지며, 같은 계산을 여러 곳에서 사용하면 매번 연산이 발생한다.

### computed를 사용하는 경우

```javascript
const { createApp, ref, computed } = Vue

const restOfTodos = computed(() => {
  return todos.value.length > 0 ? '아직 남았다' : '퇴근!'
})
```

```html
<p>{{ restOfTodos }}</p>
```

- `restOfTodos`는 `todos`에 의존하고 있어, `todos`가 변경될 때만 자동으로 재계산 후 업데이트된다.
- **콜백 함수 내부에 반드시 `return`이 필요하다.** return이 없으면 `undefined`가 저장된다.

### computed 특징

- 반환값은 **computed ref**이며, 일반 ref처럼 `.value`로 참조 가능 (템플릿에서는 `.value` 생략)
- 의존된 반응형 데이터를 **자동으로 추적**
- 의존하는 반응형 데이터가 **변경될 때만 재평가**

---

## 2. computed vs. Methods

`computed` 속성과 동일한 로직을 `method`로도 정의할 수 있다.

```javascript
// computed 예시
const restOfTodos = computed(() => {
  return todos.value.length > 0 ? '아직 남았다' : '퇴근!'
})
// 템플릿: {{ restOfTodos }}

// method 예시
const getRestOfTodos = function () {
  return todos.value.length > 0 ? '아직 남았다' : '퇴근!'
}
// 템플릿: {{ getRestOfTodos() }}
```

### 핵심 차이: 캐시(Cache)

| 구분 | computed | method |
|------|----------|--------|
| 재실행 조건 | 의존 데이터가 변경될 때만 | 렌더링이 발생할 때마다 항상 |
| 결과 저장 | 캐시됨 | 캐시 안 됨 |
| 템플릿 호출 | 괄호 없이 (`{{ value }}`) | 괄호 붙여서 (`{{ getValue() }}`) |
| return 필요 여부 | **필수** | 있을 수도, 없을 수도 |

> 캐시는 자주 꺼내 먹는 식재료를 마트(원본 데이터) 대신 냉장고(임시 저장소)에서 바로 꺼내 쓰는 것과 같다.

### 적절한 사용처

- **computed**: 의존하는 데이터에 따라 결과가 바뀌는 계산된 속성, 동일 의존성을 가진 여러 곳에서 사용할 때
- **method**: 단순히 특정 **동작**을 수행하는 함수, **계산에 인자(외부 값)가 필요한 경우**

> **TIP**: `random()`처럼 의존하는 ref가 없으면 computed를 써도 최초 1번만 실행되고 이후엔 캐시된 값이 반환된다. 실행할 때마다 바뀌어야 하는 경우라면 method를 써야 한다.

---

## 3. computed 주의사항

### 1) 반환 값은 변경하지 말 것

computed의 반환 값은 의존하는 데이터의 **파생된 값(snapshot)**이다.  
계산된 값은 **읽기 전용**으로 취급해야 하며 직접 변경하면 안 된다.  
새 값을 얻으려면 의존하는 원본 데이터를 업데이트해야 한다.

> computed 값에 직접 값을 할당하면, 기본적으로 경고가 발생하며 값이 변경되지 않는다.

### 2) computed 사용 시 원본 배열 변경하지 말 것

`reverse()`나 `sort()`처럼 원본 배열을 변경하는 메서드를 사용할 때는 반드시 원본 배열의 **복사본**을 만들어 처리해야 한다. (JavaScript의 스프레드 연산자 활용)

```javascript
// ❌ 옳지 않은 예 — 원본 배열 변경
return numbers.reverse()

// ✅ 옳은 예 — 복사본을 만들어 처리
return [...numbers].reverse()
```

---

## 💡 한 줄 요약
> `computed`는 반응형 데이터가 바뀔 때만 자동 재계산되는 캐싱 속성으로, 값 계산이 목적일 때 사용하고 반드시 `return`이 필요하다.

## ❓ 더 찾아볼 것
- computed의 getter/setter 활용법 (writable computed)
- `watchEffect`와 computed의 차이
- Vue3 Composition API에서 `computed`와 `reactive`의 관계
- 배열 헬퍼 메서드: `filter`, `map`(새 배열 반환) vs `sort`, `reverse`, `push`, `pop`(원본 변경)
