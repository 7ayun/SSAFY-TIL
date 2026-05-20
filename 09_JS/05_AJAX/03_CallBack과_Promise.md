# [JS] CallBack과 Promise

---

## 1. 비동기 처리의 핵심 문제

비동기 처리의 핵심은 작업이 **시작되는 순서가 아니라 완료되는 순서**에 따라 처리된다는 것이다. 예를 들어 A(15초)와 B(5초)가 있을 때, A가 먼저 시작됐지만 B가 먼저 끝난다.

개발자 입장에서 **코드의 실행 순서가 불명확**하다는 단점이 존재하고, 실행 결과를 정확히 예측하며 코드를 작성하기 어렵다.

이를 극복하는 두 가지 방법:

| 방법 | 설명 |
|------|------|
| **비동기 콜백** | 비동기 작업이 완료된 후 실행될 함수를 미리 정의해 전달 (고전적 방식) |
| **Promise** | 비동기 작업의 최종 완료 또는 실패를 나타내는 객체 (현대적 방식) |

---

## 2. 비동기 콜백 (Asynchronous Callback)

**비동기 콜백**이란 비동기적으로 처리되는 작업이 완료되었을 때 실행되는 함수다. 연쇄적으로 발생하는 비동기 작업을 순차적으로 동작할 수 있게 해준다.

```js
const asyncTask = function (callback) {
  setTimeout(function () {
    console.log('비동기 작업 완료')
    callback() // 작업 완료 후 콜백 호출
  }, 2000)
}

asyncTask(function () {
  console.log('작업 완료 후 콜백 실행')
})
// 출력: (2초 후) 비동기 작업 완료 → 작업 완료 후 콜백 실행
```

---

## 3. 콜백 지옥 (Callback Hell)

비동기 작업 A가 끝나야 B가 실행되고, B가 끝나야 C가 실행되는 식으로 순서를 제어하다 보면 함수가 점점 중첩된다.

```js
// 콜백 지옥 — "Pyramid of doom(파멸의 피라미드)"이라고도 함
function hell(win) {
  return function () {
    loadLink(win, REMOTE_SRC, function () {
      loadLink(win, REMOTE_SRC, function () {
        loadLink(win, REMOTE_SRC, function () {
          loadLink(win, REMOTE_SRC, function () {
            // ... 점점 깊어지는 들여쓰기
          })
        })
      })
    })
  }
}
```

- 코드의 depth가 깊어져 **가독성**이 나빠짐
- **유지보수**가 어려워짐
- 에러가 발생한 지점을 찾기 힘들어짐

이 문제를 해결하기 위해 등장한 것이 **Promise 객체**

---

## 4. Promise

**Promise**는 JavaScript에서 비동기 작업의 결과를 나타내는 객체다.

- 비동기 작업의 **최종 완료(또는 실패)**와 그 결과값을 나타냄
- "작업이 끝나면 실행시켜 줄게"라는 **약속**
- Axios도 내부적으로 Promise를 반환해 주도록 구현되어 있음

### 비동기 콜백 vs Promise 비교

```js
// ❌ 비동기 콜백 — 함수를 중첩시켜 depth가 깊어짐
work1(function () {
  work2(result1, function (result2) {
    work3(result2, function (result3) {
      console.log('최종 결과 :' + result3)
    })
  })
})

// ✅ Promise — then 체이닝으로 순차적 연결, 가독성 향상
work1()
  .then((result1) => {
    return result2
  })
  .then((result2) => {
    return result3
  })
  .catch((error) => {
    // error handling
  })
```

---

## 5. then & catch 체이닝 (Chaining)

Axios로 처리한 비동기 로직은 항상 **Promise 객체**를 반환하고, `then`과 `catch`는 모두 Promise 객체를 반환한다. 따라서 계속해서 **체이닝**이 가능하다.

> **chaining**: 함수 반환값에 꼬리를 물고 다음 함수를 바로 호출하는 것

```js
axios({})
  .then(성공하면 수행할 1번 콜백)
  .then(1번 완료 후 수행할 2번 콜백)  // 1번의 반환값이 2번의 인자로 전달됨
  .then(2번 완료 후 수행할 3번 콜백)
  .catch(실패하면 수행할 콜백)
```

### Cat API 코드 then 체이닝으로 개선

```js
// 개선 전 — then 하나에 모든 로직
.then((response) => {
  imgUrl = response.data[0].url
  imgElem = document.createElement('img')
  imgElem.setAttribute('src', imgUrl)
  document.body.appendChild(imgElem)
})

// 개선 후 — then 체이닝으로 역할 분리
// 첫 번째 then의 반환값이 두 번째 then의 인자로 전달됨
.then((response) => {
  imgUrl = response.data[0].url
  return imgUrl
})
.then((imgData) => {
  imgElem = document.createElement('img')
  imgElem.setAttribute('src', imgData)
  document.body.appendChild(imgElem)
})
```

### then 체이닝의 장점

| 장점 | 설명 |
|------|------|
| **가독성** | 비동기 작업의 순서와 의존 관계를 명확히 표현. depth가 깊어지지 않음 |
| **에러 처리** | `.catch()` 하나로 체인 전체의 에러를 처리 |
| **유연성** | 각 단계마다 데이터를 가공하거나 다른 작업 수행 가능 |
| **코드 관리** | 비동기 작업을 분리해 구성하면 관리 용이 |

> 비동기 콜백 vs Promise는 **속도 차이가 없다**. 철저하게 유지보수와 가독성을 위해 Promise를 쓰는 것이다.

---

## 6. async / await — 실무 표준

`async`/`await`는 then/catch를 더 간단하고 직관적으로 작성하는 문법이다. **실무에서는 거의 무조건 사용한다.**

- `await`: Promise 객체를 반환하는 함수 앞에 붙임. Promise가 완료될 때까지 해당 줄에서 코드를 기다림
- `async`: `await`를 사용하는 함수에 반드시 붙여야 함. 세트로 다님

```js
// then/catch 방식
axios({ method: 'get', url: URL })
  .then((response) => {
    const imgUrl = response.data[0].url
    return imgUrl
  })
  .then((imgData) => {
    document.body.appendChild(...)
  })
  .catch((error) => { console.error(error) })

// async/await 방식 — 훨씬 간결하고 동기 코드처럼 읽힘
const getCats = async function () {
  try {
    const response = await axios({ method: 'get', url: URL }) // 응답 올 때까지 대기
    const imgUrl = response.data[0].url
    // 응답이 온 뒤 아래 코드 실행
    const imgElem = document.createElement('img')
    imgElem.setAttribute('src', imgUrl)
    document.body.appendChild(imgElem)
  } catch (error) {
    console.error(error)
  }
}
```

성능은 동일하다. `async`/`await`는 가독성을 위한 문법 설탕(Syntactic Sugar)이다.

---

## 7. 비동기 처리 시 주의사항 — 워터폴 현상과 트레이드오프

### 워터폴 현상 (await 남용)

서로 독립적인 작업에 모두 `await`를 쓰면, 병렬로 처리해도 될 것을 순차적으로 기다리는 낭비가 발생한다.

```js
// ❌ 워터폴 — 1초 + 1.5초 + 2초 = 총 4.5초 소요
const burger = await getBurger()   // 1초 기다리고
const fries  = await getFries()    // 1.5초 기다리고
const coke   = await getCoke()     // 2초 기다리고
```

세 작업은 서로 독립적이므로 병렬로 처리해도 된다.

### Promise.all — 병렬 처리

서로 영향이 없는 작업을 동시에 실행하고, 모두 완료될 때까지 기다린다. 총 소요 시간 = 가장 오래 걸리는 작업 시간.

```js
// ✅ Promise.all — 가장 오래 걸리는 2초만 소요
const [burger, fries, coke] = await Promise.all([
  getBurger(),  // 1초 (독립 실행)
  getFries(),   // 1.5초 (독립 실행)
  getCoke(),    // 2초 (독립 실행)
])
// 단점: 하나라도 실패하면 전체 에러 발생
```

### Promise.allSettled — 실패해도 각각 결과 반환

```js
const results = await Promise.allSettled([getBurger(), getFries(), getCokeError()])
// [{ status: 'fulfilled', value: '버거' },
//  { status: 'fulfilled', value: '감자튀김' },
//  { status: 'rejected', reason: '콜라 기계 고장' }]

// 성공한 것만 필터링 가능
const successItems = results
  .filter(r => r.status === 'fulfilled')
  .map(r => r.value)
```

### 비동기를 쓰면 안 되는 경우

비동기가 항상 좋은 건 아니다. 결제, DB 트랜잭션 등 **순서와 결과가 반드시 보장돼야 하는** 중요한 작업은 비동기로 처리하면 안 된다. 결제가 완료됐는데 5분 뒤에 실패 알림이 뜨면 큰 문제가 된다.

---

## 8. 비동기 처리와 UX

- **로딩 스피너**: 요청을 보내는 동안 뒤에서 처리 중임을 사용자에게 알림
- **에러 시 재시도 안내**: `.catch()` 분기에서 "다시 시도" 버튼 표시
- **무한 스크롤**: 스크롤할 때 비동기로 새 콘텐츠 요청
- **자동완성**: 키 입력마다 비동기로 검색 결과 요청
- **대시보드**: 여러 데이터를 병렬 로드해 먼저 준비된 것부터 표시

동기식으로 처리하면 데이터 전체가 로딩될 때까지 앱 실행이 차단돼 사용자가 "앱이 멈췄나?"라고 느낄 수 있다.

---

## 💡 한 줄 요약
> 비동기 콜백의 콜백 지옥을 해결하기 위해 Promise가 등장했고, async/await는 그 Promise를 동기 코드처럼 읽기 좋게 만든 실무 표준 문법이다.

## ❓ 더 찾아볼 것
- `Promise.race()` — 가장 먼저 완료된 Promise 결과를 반환 (타임아웃 구현 등)
- `Promise.all` vs `Promise.allSettled` 상황별 선택 기준
- `async`/`await` 에러 처리 패턴 (`try`/`catch`)
- Axios interceptors — 요청/응답을 가로채서 공통 처리 (토큰 자동 첨부 등)
- 워터폴 현상 방지 패턴
