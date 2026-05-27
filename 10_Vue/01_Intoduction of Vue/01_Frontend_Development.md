# [Vue] Frontend Development

---

## 1. Frontend Development란

웹사이트와 웹 애플리케이션의 사용자 인터페이스(UI)와 사용자 경험(UX)을 만들고 디자인하는 것을 **Frontend Development**라고 한다.

HTML, CSS, JavaScript 등을 활용하여 사용자가 직접 상호작용하는 부분을 개발한다.

```
Client  →  Frontend Framework (Vue, React, Angular)  →  Server (Django 등)
       ←                                              ←
```

> "장고는 Backend 서버로 동작하고, 사용자에게 보여지는 UI/UX 경험은 프론트엔드 프레임워크로 개발한다."

---

## 2. Client-side frameworks

### 정의

클라이언트 측에서 UI와 상호작용을 개발하기 위해 사용되는 **JavaScript 기반 프레임워크**다.

- 웹사이트의 UI를 효율적으로 만들기 위해 미리 짜놓은 코드의 뼈대
- 복잡한 웹 애플리케이션을 마치 레고 조립하듯 가능한 부품 단위로 쉽게 개발할 수 있게 도와줌
- 대표적인 3대장: **Vue**, **React**, **Angular**

### Client-side frameworks가 필요한 이유

**이유 1 — 단순히 읽는 곳 → 무언가를 하는 곳으로 변화**

- 사용자는 웹에서 음악 스트리밍, 영화 시청, 영상 채팅 등을 하게 됨
- 이처럼 현대적이고 복잡한 대화형 웹사이트를 **"웹 애플리케이션(web applications)"** 이라 부르기 시작
- JavaScript 기반의 Client-side frameworks가 등장하면서 매우 동적인 대화형 애플리케이션을 훨씬 더 쉽게 구축할 수 있게 됨

**이유 2 — 다루는 데이터가 많아짐**

예시: Facebook에서 닉네임을 변경한다면?

- 친구 목록, 타임라인, 스토리 등 친구 이름이 출력되는 **모든 곳**이 함께 변경되어야 함
- 애플리케이션의 기본 데이터를 안정적으로 추적하고 업데이트(렌더링, 추가, 삭제 등) 하는 도구가 필요
- 애플리케이션의 상태가 변경될 때마다 UI가 **일관성 있게** 업데이트되어야 함

> "Vanilla JS로 하면 이름이 사용되는 곳을 전부 선택해서 변경해야 하는데, 새로운 기능에서 하나라도 깜빡하면 UI 일관성이 떨어지고 UX가 굉장히 나빠진다."

### Vanilla JS만으로는 간단하지 않음

불필요한 코드 반복 예시:

```html
<!-- HTML -->
<h1>안녕 하세요 <span id="username1"></span></h1>
<div><span id="username2"></span>님의 친구 목록</div>
<div><span id="username3"></span>님의 알림 목록</div>
<div><span id="username4"></span>님의 친구 요청 목록</div>
```

```javascript
// JavaScript
const inputArea = document.querySelector('#inputArea')
const username1 = document.querySelector('#username1')
const username2 = document.querySelector('#username2')
const username3 = document.querySelector('#username3')
const username4 = document.querySelector('#username4')

inputArea.addEventListener('input', function (e) {
  username1.textContent = e.target.value
  username2.textContent = e.target.value
  username3.textContent = e.target.value
  username4.textContent = e.target.value
})
```

### Client-side frameworks의 필요성

| 필요성 | 설명 |
|--------|------|
| 동적이고 반응적인 웹 앱 개발 | 실시간 데이터 업데이트 |
| 코드 재사용성 증가 | 컴포넌트 기반 아키텍처, 모듈화된 코드 구조 |
| 개발 생산성 향상 | 강력한 개발 도구 지원 |

---

## 3. SPA (Single Page Application)

### 정의

**단일 페이지에서 동작하는 웹 애플리케이션**

하나의 HTML 파일 위에서 JavaScript가 필요한 부분만 교체하며 **'진짜' 페이지 이동 없이** 동작한다.

### SPA 작동 원리

1. 최초 로드 시, 애플리케이션에 필요한 주요 리소스를 다운로드
2. 페이지 갱신에 대해 필요한 데이터만을 비동기적으로 전달받아 화면의 필요한 부분만 동적으로 갱신
   - AJAX와 같은 기술을 사용하여 필요한 데이터만 비동기적으로 로드
   - 페이지 전체를 다시 로드할 필요 없이, 필요한 데이터만 서버로부터 가져와 화면에 표시
3. JavaScript를 사용하여 클라이언트 측에서 동적으로 콘텐츠를 생성하고 업데이트 ➡️ **CSR 방식**

---

## 4. CSR (Client Side Rendering)

### 정의

**클라이언트에서 콘텐츠를 렌더링 하는 방식**

> 비유: "완제품을 공장에서 만들어서 받아보는 것(SSR) vs 부품이 날라와서 직접 조립하는 것(CSR)"

- **SSR** (Server-side Rendering): 서버에서 완성된 HTML을 만들어 전달 → Django의 `render()` 함수
- **CSR** (Client-side Rendering): 빈 HTML + JS를 받고, 클라이언트(브라우저)가 JavaScript로 직접 조립

### CSR 작동 원리

1. 사용자가 웹사이트에 요청을 보냄
2. 서버는 최소한의 HTML과 JavaScript 파일을 클라이언트로 전송
3. 클라이언트는 HTML과 JavaScript를 다운로드 받음
4. 브라우저가 JavaScript를 실행하여 동적으로 페이지 콘텐츠를 생성
5. 필요한 데이터는 API를 통해 서버로부터 **비동기적**으로 가져옴

```
Client  → (최초 요청)               → Server
        ← HTML(빈 페이지) + JS       ←
        
Client  → (Ajax: IT 카테고리 데이터 요청) → Server
        ← JSON 응답                      ←
(클라이언트가 JS로 직접 DOM 업데이트)
```

### CSR과 SPA의 장점

| 장점 | 설명 |
|------|------|
| 빠른 페이지 전환 | 첫 로드 후 필요한 데이터만 가져오면 되므로, 페이지의 일부만 다시 렌더링 가능 / 서버로 전송되는 데이터 양 최소화(서버 부하 방지) |
| 사용자 경험 | 새로고침이 발생하지 않아 네이티브 앱과 유사한 사용자 경험 제공 |
| Frontend와 Backend의 명확한 분리 | Frontend는 UI 렌더링 및 사용자 상호작용 담당, Backend는 데이터 및 API 제공 담당 / 대규모 애플리케이션을 더 쉽게 개발하고 유지관리 가능 |

### CSR과 SPA의 단점

**1. 느린 초기 로드 속도**
- 전체 페이지를 보기 전에 약간의 지연을 느낄 수 있음
- JavaScript가 다운로드, 구문 분석 및 실행될 때까지 페이지가 완전히 렌더링 되지 않기 때문

**2. SEO(검색 엔진 최적화) 문제**
- 페이지를 나중에 그려 나가기 때문에 검색에 잘 노출되지 않을 수 있음
- 검색엔진 입장에서 HTML을 읽어서 분석해야 하는데 아직 콘텐츠가 모두 존재하지 않기 때문

> TIP: 초기 로딩 속도가 느리지만, 초기 로딩만 끝나고 나면 훌륭한 UX 경험을 주는 장점이 단점을 상쇄한다. 위에서 언급한 단점은 **Next.js, Nuxt.js** 같은 하이브리드 프레임워크를 통해 대부분 해결할 수 있게 되었다.

### SPA vs MPA / CSR vs SSR 비교

| 구분 | 설명 |
|------|------|
| **MPA** (Multi Page Application) | 여러 개의 HTML 파일이 서버로부터 각각 로드됨 / 사용자가 다른 페이지로 이동할 때마다 새로운 HTML 파일이 로드됨 (Django의 방식) |
| **SSR** (Server-side Rendering) | 서버에서 화면을 렌더링 하는 방식 / 모든 데이터가 담긴 HTML을 서버에서 완성 후 클라이언트에게 전달 |

---

## 💡 한 줄 요약

> Client-side frameworks는 Vanilla JS의 반복적인 DOM 조작 한계를 극복하기 위해 등장했으며, SPA와 CSR 방식을 통해 빠른 페이지 전환과 부드러운 UX를 제공한다.

## ❓ 더 찾아볼 것

- SEO 점수 측정 사이트 및 SEO 최적화 규칙
- Next.js (React), Nuxt.js (Vue) — SPA의 SEO 문제를 해결하는 하이브리드 프레임워크
- PWA (Progressive Web App) — 오프라인에서도 동작하는 웹 앱
