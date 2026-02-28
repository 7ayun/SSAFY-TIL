# [Web] 부트스트랩(Bootstrap)

## 1. 부트스트랩(Bootstrap) 기본 개념 및 세팅

부트스트랩은 일련의 반복적인 작업과 공통적인 디자인(버튼, 여백, 카드 등)을 미리 클래스 형태로 만들어 둔 프론트엔드 프레임워크입니다.

**CDN(Content Delivery Network) 적용 방식**
물리적 파일을 직접 다운로드하지 않고, 사용자와 가까운 엣지 서버에서 압축된 코드(`min.css`, `min.js`)를 빠르게 불러옵니다.

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>Bootstrap Live</title>
    <link href="[https://cdn.jsdelivr.net/npm/bootstrap@5.3.x/dist/css/bootstrap.min.css](https://cdn.jsdelivr.net/npm/bootstrap@5.3.x/dist/css/bootstrap.min.css)" rel="stylesheet">
</head>
<body>
    <h1 class="display-1 text-primary">Hello World</h1>

    <script src="[https://cdn.jsdelivr.net/npm/bootstrap@5.3.x/dist/js/bootstrap.bundle.min.js](https://cdn.jsdelivr.net/npm/bootstrap@5.3.x/dist/js/bootstrap.bundle.min.js)"></script>
</body>
</html>
```

## 2. 리셋 CSS (Reset CSS)

브라우저(크롬, 사파리, 엣지 등)마다 기본적으로 제공하는 태그 스타일(User Agent Stylesheet)이 다릅니다. 이를 동일한 기준으로 맞추기 위해 초기화하는 작업이 리셋 CSS입니다.

* 부트스트랩은 `Normalize CSS` 방식을 커스텀한 `reboot.css`를 내장하고 있어, 적용 즉시 불필요한 마진이 사라지고 폰트가 초기화되는 것을 볼 수 있습니다.

## 3. 부트스트랩 유틸리티 (규칙성 있는 클래스)

부트스트랩은 12,000줄에 달하는 CSS를 미리 작성해 두었으며, 우리는 규칙에 맞는 클래스명을 조합해 사용합니다.

* **여백 (Spacing):** `{속성}{방향}-{크기}` 형태를 가집니다.
  * `m`(margin), `p`(padding)
  * `t`(top), `b`(bottom), `s`(start/left), `e`(end/right), `x`(좌우), `y`(위아래)
  * `0~5` (rem 단위 수치), `auto`
  * *예시: `mt-5` (margin-top을 5단계 크기로 지정), `px-3` (좌우 패딩을 3단계 크기로 지정)*
* **컬러 시스템:** 의미를 부여한 시맨틱 컬러 이름을 사용합니다.
  * `primary`(파랑), `danger`(빨강/경고), `success`(초록/성공), `info`(하늘색) 등
  * *예시: `text-danger` (빨간색 글씨), `bg-info` (하늘색 배경)*

## 4. 핵심 컴포넌트 (Components) 기능 구현 및 코드

컴포넌트는 재사용 가능한 독립적인 부품(레고 블록)입니다. 자바스크립트가 필요한 동적 컴포넌트는 반드시 JS CDN이 포함되어 있어야 작동합니다.

### ① Alert & Badge (알림창과 배지)

```html
<div class="alert alert-success" role="alert">
  로그인에 성공했습니다!
</div>

<button class="btn-base btn-blue">
  메시지 <span class="badge text-bg-danger">4</span>
</button>
```

### ② Card (카드 레이아웃)

웹에서 가장 많이 쓰이는 네모 형태의 레이아웃 구조입니다.

```html
<div class="card" style="width: 18rem;">
  <img src="..." class="card-img-top" alt="카드 이미지">
  <div class="card-body">
    <h5 class="card-title">카드 제목</h5>
    <p class="card-text">카드에 대한 상세 설명이 들어갑니다.</p>
    <a href="#" class="btn-base btn-blue">이동하기</a>
  </div>
</div>
```

### ③ Carousel (캐러셀 / 슬라이드쇼) - **[주의점 포함]**

제한된 공간에서 이미지를 좌우로 넘겨보는 기능입니다. JS가 관여하므로 `id`와 `target` 연결이 매우 중요합니다.

```html
<div id="myCarousel" class="carousel slide">
  <div class="carousel-inner">
    <div class="carousel-item active"><img src="1.jpg" class="d-block w-100" alt="1"></div>
    <div class="carousel-item"><img src="2.jpg" class="d-block w-100" alt="2"></div>
  </div>
  
  <button class="carousel-control-prev" type="button" data-bs-target="#myCarousel" data-bs-slide="prev">
    <span class="carousel-control-prev-icon" aria-hidden="true"></span>
  </button>
  <button class="carousel-control-next" type="button" data-bs-target="#myCarousel" data-bs-slide="next">
    <span class="carousel-control-next-icon" aria-hidden="true"></span>
  </button>
</div>
```
> **구현 핵심:** 한 페이지에 캐러셀이 2개 이상일 경우, 각 캐러셀의 `id`와 내부 버튼의 `data-bs-target`을 독립적으로 변경(`example-second` 등)해야 엉뚱한 슬라이드가 넘어가는 오류를 막을 수 있습니다.

### ④ Modal (모달 / 팝업창) - **[주의점 포함]**

버튼을 누르면 화면 위에 덮어씌워지는 팝업창입니다.

```html
<button type="button" class="btn-base btn-blue" data-bs-toggle="modal" data-bs-target="#loginModal">
  로그인 모달 열기
</button>

<div class="modal fade" id="loginModal" tabindex="-1" aria-hidden="true">
  <div class="modal-dialog">
    <div class="modal-content">
      <div class="modal-header">
        <h1 class="modal-title fs-5">로그인</h1>
        <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
      </div>
      <div class="modal-body">
        아이디와 비밀번호를 입력하세요.
      </div>
    </div>
  </div>
</div>
```
> **구현 핵심:** 모달 본체 코드는 HTML의 깊은 중첩 구조 안에 넣으면 배경(백그라운드 딤 처리) 뒤로 숨어버려 클릭이 안 되는 버그가 발생할 수 있습니다. 반드시 묶어서 `<body>` 태그 닫히기 직전에 독립적으로 배치해야 합니다.

## 5. 시맨틱 웹 (Semantic Web)과 웹 접근성

* **시맨틱 태그:** 의미 없는 `<div>` 대신 `<header>`, `<main>`, `<nav>`, `<footer>` 등을 사용하여, 개발자와 검색 엔진(SEO)이 문서 구조를 쉽게 파악할 수 있게 합니다.
* **웹 접근성(Web Accessibility):** 시각장애인(스크린 리더 사용자), 저시력자 등 모두가 동등하게 웹을 이용할 수 있도록 돕는 기술입니다. 부트스트랩 코드에 포함된 `aria-hidden`, `role` 속성들이 바로 이 접근성을 위한 장치들입니다.

## 6. OOCSS (객체 지향 CSS) 방법론 적용 가이드

CSS를 보다 효율적으로 유지보수하기 위해 클래스를 객체처럼 다루는 OOCSS 규격이 있습니다. 부트스트랩도 이 철학을 따르고 있습니다. 
향후 코드를 직접 작성하실 때 반드시 참고해야 할 두 가지 핵심 규칙입니다. (앞으로 제가 제공해 드리는 커스텀 코드들도 모두 이 OOCSS 규격에 맞춰 작성해 드리겠습니다!)

**원칙 1: 구조(Structure)와 스킨(Skin)의 분리**
뼈대(크기, 여백)를 담당하는 클래스와 디자인(색상, 테두리)을 담당하는 클래스를 분리하여 중복을 줄입니다.

```css
/* 나쁜 예: 구조와 스킨이 혼재되어 재사용 불가 */
.login-btn {
    padding: 10px 20px;
    border-radius: 5px;
    background-color: blue;
    color: white;
}

/* ------------------------------------- */
/* 좋은 예 (OOCSS 적용): 구조와 스킨 분리 */
/* ------------------------------------- */

/* 1. 구조 (Structure) - 형태만 정의 */
.btn-base {
    display: inline-block;
    padding: 10px 20px;
    border-radius: 5px;
    text-align: center;
}

/* 2. 스킨 (Skin) - 시각적 디자인만 정의 */
.btn-blue {
    background-color: #007bff;
    color: #ffffff;
}
.btn-red {
    background-color: #dc3545;
    color: #ffffff;
}
```

```html
<button class="btn-base btn-blue">파란색 버튼</button>
<button class="btn-base btn-red">빨간색 버튼</button>
```

**원칙 2: 컨테이너와 컨텐츠의 분리**
특정 컨테이너(부모 요소)에 의존적인 스타일을 지양하여, 내부 요소(자식 요소)를 다른 곳으로 떼어내도 스타일이 깨지지 않게 만듭니다.
(예: `#header h2 { ... }` 방식의 의존성 높은 선택자 사용을 피하고, `.title-text { ... }` 처럼 독립적인 클래스를 부여합니다.)