# [JS] Ajax

---

## 1. Ajax란?

**Ajax (Asynchronous JavaScript and XML)**
- 비동기적인 웹 애플리케이션 개발을 위한 **기술들의 집합 (개념이자 접근 방식)**
- 특정 라이브러리나 기술 하나를 지칭하는 것이 아님

> Ajax는 웹 페이지 전체를 새로고침하지 않고, 백그라운드에서 서버와 데이터를 주고받는 비동기 통신 기술이다.

이름의 'X'는 원래 XML을 의미했지만, 현재는 더 가볍고 JS에서 다루기 쉬운 **JSON** 형식을 주로 사용한다.

---

## 2. 기존 방식(장고)과 Ajax 방식 비교

### 기존 방식 — 전체 페이지 새로고침

장고에서 HTML 렌더링 방식처럼, 매번 서버가 완성된 HTML 페이지 전체를 응답으로 전달한다.

```
Client ──── 요청(form submit) ────→ Server
Client ←────── HTML 전체 ──────── Server
```

- 모든 요청에 새로운 페이지가 응답되므로 계속 새로고침 발생
- 기존 페이지와 유사한 내용도 중복된 코드를 다시 전송 → **대역폭 낭비**
- 좋아요를 한 번 누를 때마다 유저 목록, 게시글 등 전체를 다시 조회하면 서버 부하 폭발

### Ajax 방식 — 부분 업데이트

```
Client ── XHR 객체 생성 및 요청 ──→ Server
Client ←─────── JSON 데이터 ──────── Server
```

- 서버는 새로운 HTML 페이지가 아닌 **필요한 데이터만** 응답 (JSON 등)
- 페이지 일부분만 동적으로 갱신 → 새로고침 없음
- 데이터 처리의 일부가 클라이언트 쪽으로 이동 → 교환 데이터량 감소

---

## 3. Ajax의 3가지 목적

| 목적 | 설명 |
|------|------|
| **비동기 통신** | 페이지 전체 새로고침 없이 서버와 데이터 주고받기. 좋아요 누를 때마다 새로고침 불필요 |
| **부분 업데이트** | HTML 페이지 일부 DOM만 업데이트. 사용자 경험(UX) 향상 |
| **서버 부하 감소** | 필요한 데이터만 요청하므로 불필요한 중복 데이터 전송 없음 |

---

## 4. Ajax와 Axios의 관계

| 구분 | Ajax | Axios |
|------|------|-------|
| 성격 | 비동기 웹 개발 기술들의 집합 **(개념)** | HTTP 요청을 처리하는 JS 라이브러리 **(도구)** |
| 비유 | 이론, 접근 방식 | 구체적인 구현체 |

**Axios**: 브라우저와 Node.js 환경 모두에서 사용할 수 있는, **Promise 기반의 HTTP 클라이언트 라이브러리**

- Ajax 통신을 쉽게 할 수 있도록 도와주는 JS 라이브러리
- XHR 객체 생성을 추상화해 간편한 API 제공
- Promise 기반으로 비동기 요청 처리 → `then`/`catch`로 성공·실패 처리
- 데이터 변환, 에러 처리가 편리해 실무에서 가장 많이 사용

### CDN으로 사용하기

```html
<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
```

---

## 5. Axios 기본 구조 — Promise와 then/catch

`axios()`는 비동기 HTTP 요청을 보내고 **Promise 객체**를 반환한다.

```js
axios({
  method: 'get',
  url: 'https://api.example.com/data',
})
  .then((response) => {
    // 성공 시 실행: 이전 작업의 성공 결과를 response로 받음
    console.log(response)       // Response 객체 전체
    console.log(response.data)  // 실제 응답 데이터
  })
  .catch((error) => {
    // 실패 시 실행: 네트워크 오류, 4xx, 5xx 등
    console.error(error)
  })
```

| 메서드 | 역할 |
|--------|------|
| `.then(callback)` | 작업 **성공** 시 실행. callback은 성공 결과를 인자로 전달받음 |
| `.catch(callback)` | 작업 **실패** 시 실행. then()이 하나라도 실패하면 실행 (남은 then은 중단) |

---

## 6. Axios 활용 실습 — Cat API로 고양이 이미지 가져오기

**The Cat API** (`https://api.thecatapi.com/v1/images/search`)에 GET 요청을 보내면 고양이 이미지 정보를 JSON으로 응답해준다.

```json
[{ "id": "d6n", "url": "https://cdn2.thecatapi.com/images/d6n.jpg", "width": 333, "height": 500 }]
```

### 전체 구현 코드 (이벤트 복습 포함)

```html
<!-- cat-api.html -->
<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
<button>냥냥펀치</button>

<script>
  const URL = 'https://api.thecatapi.com/v1/images/search'

  // 1단계: 요소 가져오기
  const btn = document.querySelector('button')

  // 2단계: 이벤트 핸들러 정의
  const getCats = function () {
    axios({
      method: 'get',
      url: URL,
    })
      .then((response) => {
        // response.data는 배열 → [0]으로 첫 번째 항목 접근
        const imgUrl = response.data[0].url

        // img 요소 생성 후 src 속성 설정
        const imgElem = document.createElement('img')
        imgElem.setAttribute('src', imgUrl)

        // body에 이미지 추가
        document.body.appendChild(imgElem)
      })
      .catch((error) => {
        console.log(error)
        console.log('실패했다옹')
      })

    // 이 코드가 먼저 출력됨 → 비동기 처리 확인!
    console.log('야옹야옹')
  }

  // 3단계: 요소와 이벤트 핸들러 연결
  btn.addEventListener('click', getCats)
</script>
```

> 버튼 클릭 시 `"야옹야옹"`이 먼저 출력되고, 이후 응답이 오면 이미지가 화면에 추가된다. HTTP 요청이 비동기임을 직접 확인할 수 있다.

**Axios 활용 흐름**: XHR 객체 생성 및 요청 → 응답 데이터 생성 → JSON 데이터 응답 → Promise 객체를 활용해 DOM 조작 (웹 페이지 일부분만 다시 로딩)

---

## 7. 앞으로의 활용

오늘 배운 Axios는 이후 **Vue 프론트엔드**에서 장고 DRF로 만든 API 서버에 요청을 보내고, 받아온 데이터를 비동기적으로 처리해 화면을 그리는 방식으로 핵심적으로 활용된다.

---

## 💡 한 줄 요약
> Ajax는 페이지 전체 새로고침 없이 서버와 데이터를 주고받는 개념이고, Axios는 그 개념을 Promise 기반으로 쉽게 구현해주는 도구다.

## ❓ 더 찾아볼 것
- `fetch API` — 브라우저 내장 비동기 HTTP 요청 API (Axios와 비교)
- HTTP 메서드 종류 (GET, POST, PUT, DELETE, PATCH)
- JSON 구조 및 `JSON.parse()`, `JSON.stringify()`
- CORS (Cross-Origin Resource Sharing)
- 직렬화(Serialization) — 장고 DRF에서 데이터를 JSON으로 반환하는 방식
