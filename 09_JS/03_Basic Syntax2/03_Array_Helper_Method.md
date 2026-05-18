# [JS] Basic Syntax2 - Array Helper Method

---

## 1. 배열의 진화: Array Helper Method 등장 배경

**과거 방식 (명령형)**: `for` 문으로 "어떻게(How) 순회할지"를 일일이 코딩

```javascript
for (let i = 0; i < arr.length; i++) {
  // 조건, 증감 모두 직접 관리
}
```

**Array Helper Method 방식 (선언형)**: "무엇을(What) 할지"만 콜백 함수로 전달

```javascript
arr.forEach((item) => { /* 로직만 작성 */ })
```

**Array Helper Method**: 배열을 순회하며 특정 로직을 수행하는 고차 함수 모음으로, 메서드의 인자로 콜백 함수를 받는다.

---

## 2. 순회: `forEach`

"배열의 모든 요소를 하나씩 훑으면서, 내가 전달한 함수를 실행해줘!"

```javascript
// 구문
array.forEach(function (item [, index [, array]]) {
  // do something
})
```

**매개변수:**
- `item` (필수): 현재 처리 중인 배열 요소
- `index` (선택): 현재 요소의 인덱스
- `array` (선택): 메서드를 호출한 배열 본체

```javascript
const names = ['Alice', 'Bella', 'Cathy']

// 일반 함수
names.forEach(function (name) {
  console.log(name)
})

// 화살표 함수 (권장: 더 간결하고 this 처리도 안전)
names.forEach((name) => {
  console.log(name)
})
// Alice / Bella / Cathy 순서로 출력

// index, array 활용
names.forEach((name, index, array) => {
  console.log(`${name} | ${index} | ${array}`)
})
// Alice | 0 | Alice,Bella,Cathy
// Bella | 1 | Alice,Bella,Cathy
// Cathy | 2 | Alice,Bella,Cathy
```

### 주의사항

| 특징 | 설명 |
|------|------|
| 반환값 없음 | `forEach`를 변수에 할당해도 `undefined`만 저장됨 |
| 멈출 수 없음 | `break`, `continue` 사용 불가. 멈춰야 하면 `for...of` 사용 |

---

## 3. 변형: `map`

"배열의 모든 요소를 하나씩 꺼내서, 변형(가공)시킨 뒤 새로운 배열로 만들어줘!"

- 원본 배열의 요소를 1:1로 매핑(Mapping)하여 **새로운 배열 반환**
- **결과 배열의 길이 = 원본 배열의 길이 (항상 같음)**
- `return`이 반드시 필요

```javascript
// forEach vs map 비교
const arr = [1, 2, 3]

const result1 = arr.forEach((item) => {
  return item * 2  // 반환값이 없음
})
console.log(result1)  // undefined

const result2 = arr.map((item) => {
  return item * 2  // 이 값이 새로운 배열의 요소가 됨
})
console.log(result2)  // [2, 4, 6]
```

### 실무 활용 예시: 특정 속성만 추출

```javascript
const persons = [
  { name: 'Alice', age: 20 },
  { name: 'Bella', age: 21 }
]

// for...of 방식 (명령형)
let result1 = []
for (const person of persons) {
  result1.push(person.name)
}

// map 방식 (선언형) → "persons를 name만 뽑은 배열"임이 한눈에 보임
const result2 = persons.map((person) => {
  return person.name
})

// 화살표 함수 한 줄 축약
const lengths = names.map(name => name.length)
console.log(lengths)  // [5, 5, 5]
```

### map의 핵심 특성
- 원본 배열을 **절대 변경하지 않음** → 새로운 배열 반환
- `push`로 원본을 수정하는 것보다 `map`으로 새 배열을 만드는 패턴을 선호
- `map`이 배열을 반환하므로 **메서드 체이닝** 가능

```javascript
// 메서드 체이닝
arr.map(...).filter(...).forEach(...)
```

### Python `map`과의 비교

```python
# Python: list()로 형변환 필요 (이터레이터를 반환하기 때문)
numbers = [1, 2, 3]
def square(num): return num ** 2
new_numbers = list(map(square, numbers))
```

```javascript
// JavaScript: 즉시 배열 반환
const numbers = [1, 2, 3]
const callBackFunction = function (number) { return number ** 2 }
const newNumbers = numbers.map(callBackFunction)
```

---

## 4. 선별: `filter`

"배열에서 내가 원하는 조건에 맞는 요소만 걸러내줘!"

- 콜백 함수의 반환값이 `true`인 요소만 모아 **새로운 배열 반환**
- `return` 값이 반드시 `Boolean` (`true`/`false`)이어야 함
- **결과 배열의 길이 ≤ 원본 배열의 길이**

```javascript
// 구문 (forEach, map과 완전히 동일)
const newArr = array.filter(function (item, index, array) {
  // do something
})

// 짝수만 걸러내기
const numbers = [1, 2, 3, 4, 5]
const evens = numbers.filter((num) => {
  return num % 2 === 0  // true면 남기고, false면 버림
})
console.log(evens)  // [2, 4]
```

### 실무 활용 예시: 카테고리 필터링 & 삭제

```javascript
const products = [
  { id: 1, name: 'Cucumber', type: 'vegetable' },
  { id: 2, name: 'Banana',   type: 'fruit' },
  { id: 3, name: 'Carrot',   type: 'vegetable' },
  { id: 4, name: 'Apple',    type: 'fruit' },
]

// 1. 'fruit'만 필터링
const fruits = products.filter(product => product.type === 'fruit')
console.log(fruits)  // Banana, Apple 객체

// 2. id가 3인 상품 '삭제' (사실은 3번 빼고 나머지만 유지)
const deletedList = products.filter(product => product.id !== 3)
console.log(deletedList)  // Cucumber, Banana, Apple
```

> 조건을 만족하는 요소가 하나도 없으면 `null`이 아닌 **빈 배열 `[]`** 을 반환한다.

### `map` vs `filter` 비교

| 구분 | `map` (변형) | `filter` (선별) |
|------|-------------|----------------|
| 목적 | 요소를 가공해 형태 변환 | 조건에 맞는 요소만 선택 |
| 결과 길이 | 원본과 항상 같음 (1:1) | 원본보다 같거나 작음 |
| 비유 | 빵 굽기 (밀가루 → 빵) | 과일 고르기 (썩은 것 버리기) |

---

## 5. 기타 유용한 Array Helper Methods

MDN 문서를 참고하여 활용해보자.

| 메서드 | 역할 |
|--------|------|
| `find` | 콜백 반환값이 `true`인 **첫 번째 요소** 반환 |
| `some` | 하나라도 `true`면 `true` 반환 (OR 개념), 즉시 순회 중지 |
| `every` | 모두 `true`일 때만 `true` 반환 (AND 개념), `false` 발생 시 즉시 중지 |

---

## 6. 배열과 전개 구문 (Spread Syntax)

`...`은 배열의 괄호를 없애고 내용물만 꺼내므로, 배열을 합치거나 중간에 삽입할 때 유용하다.

```javascript
let parts = ['어깨', '무릎']
let lyrics = ['머리', ...parts, '발']

console.log(lyrics)  // ['머리', '어깨', '무릎', '발']
```

- 전개 구문은 항상 **새로운 배열** 생성, 원본 배열은 변경되지 않음
- 배열 안의 객체는 주소값만 복사(얕은 복사)됨

---

## 7. 배열 순회 방법 비교

| 방식 | 특징 | 사용 시점 |
|------|------|-----------|
| `for` 문 | 인덱스 직접 제어, `break`/`continue` 가능 | 복잡한 인덱스 제어가 필요할 때만 |
| `for...of` | 요소에 바로 접근, `break`/`continue` 가능 | 중간에 멈추거나 건너뛰어야 할 때 |
| `forEach()` | 간결하고 가독성 높음, `break`/`continue` 불가 | 배열을 처음부터 끝까지 순회할 때 |

> 특정 방식을 "무조건적으로 사용"하기보다 **상황에 맞는 사용**이 권장된다.

---

## 8. (심화) 집계: `reduce`

"배열의 요소를 하나씩 줄여가며 결국 하나의 결과값으로 합쳐줘!"

배열을 숫자 하나, 문자열, 객체, 다른 배열로 **변환**할 때 사용한다.

### 핵심 파라미터

```javascript
arr.reduce((acc, cur, index, array) => {
  return nextAccValue
}, initialValue)
```

- **`acc` (Accumulator)**: 누적값. 이전 단계까지 계산된 결과, `return`한 값이 다음 단계의 `acc`가 됨
- **`cur` (Current)**: 현재 처리 중인 배열 요소
- **`initialValue`**: `acc`의 초기값 (생략 가능하지만 명시 강력 권장)

### 숫자 합계 예시

```javascript
const numbers = [1, 2, 3, 4, 5]

const sum = numbers.reduce((acc, cur) => {
  console.log(`누적: ${acc}, 현재: ${cur}`)
  return acc + cur
}, 0)

// 누적: 0, 현재: 1
// 누적: 1, 현재: 2
// 누적: 3, 현재: 3
// 누적: 6, 현재: 4
// 누적: 10, 현재: 5

console.log(sum)  // 15
```

### 객체 그루핑 예시 (실무 활용)

배열 데이터를 원하는 형태의 객체로 변환할 때 강력하다.

```javascript
const names = ['Alice', 'Bob', 'Alice', 'Charlie', 'Bob']

const nameCounts = names.reduce((acc, name) => {
  acc[name] = (acc[name] || 0) + 1
  return acc
}, {})  // 중요! 초기값으로 빈 객체 {} 설정

console.log(nameCounts)  // { Alice: 2, Bob: 2, Charlie: 1 }
```

### `map` + `filter` 동시 처리

```javascript
const nums = [1, 2, 3, 4, 5]

// filter().map()을 reduce 하나로
const result = nums.reduce((newArr, cur) => {
  if (cur % 2 === 0) {         // 1. filter 역할 (짝수만)
    newArr.push(cur * 2)       // 2. map 역할 (2배)
  }
  return newArr                // 3. 누적 배열 반환
}, [])

console.log(result)  // [4, 8]
```

> `reduce`는 강력하지만 가독성이 떨어질 수 있다. `filter().map()` 체이닝이 더 명확한 경우가 많다.

---

## 9. `forEach`의 중요성: 프레임워크와의 연결

`for...of`로도 충분하지만, 모던 웹 개발(Vue, React)에서는 배열 메서드 방식을 선호한다.

```javascript
// forEach 메서드
numbers.forEach((num) => {
  console.log(num)
})

// map 메서드 (Vue에서 데이터를 화면에 뿌릴 때 가장 많이 사용)
const newNumbers = numbers.map((num) => {
  return num * 2
})

// 추후 배울 Axios 서버 통신: forEach와 구조가 동일!
axios.get(url).then((response) => {
  console.log(response)
})
```

- `for...of` (명령형): "어떻게(How)" 순회할지 명령 중심
- `forEach` (선언형): "무엇을(What)" 할지만 집중, 데이터 처리 중심

---

## 💡 한 줄 요약
> `forEach`(순회), `map`(변형), `filter`(선별)는 콜백 함수를 인자로 받아 선언형 스타일로 배열을 다루며, Vue/React 등 프레임워크의 핵심 패턴으로 이어진다.

## ❓ 더 찾아볼 것
- 선언형(Declarative) vs 명령형(Imperative) 프로그래밍
- `Array.prototype.find()`, `findIndex()`
- `flatMap()`: map + flat 동시 처리
- Vue.js에서 배열 메서드 활용 패턴
- MDN Array 문서: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array
