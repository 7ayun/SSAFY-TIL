# [Web] Bootstrap 프레임워크 및 컴포넌트

> **핵심 키워드:** #Bootstrap #Framework #CDN #Reset_CSS #Component #Responsive_Web

---

## 🎯 학습 목표
* 프론트엔드 프레임워크(Bootstrap)의 개념 및 도입 필요성 이해
* CDN(Content Delivery Network) 방식의 원리 및 파일 연동 방법 마스터
* Reset CSS(Normalize.css)의 작동 원리 및 브라우저 간 렌더링 차이 해결
* Bootstrap 클래스 네이밍 규칙(스페이싱, 컬러 등) 파악 및 적용
* 반응형 및 동적 컴포넌트(Card, Navbar, Carousel, Modal 등) 활용 능력 배양

---

## 💡 주요 개념 정리

### 1. 프레임워크(Framework)와 Bootstrap
* **프레임워크 개념:** 소프트웨어 개발 시 공통적으로 필요한 기능, 구조, 뼈대 등을 미리 만들어두고 제공하는 개발 환경 (일종의 도구 모음이자 토대)
* **도입 배경:** 모바일, 태블릿, 데스크탑 등 다양한 기기에 대응하는 반응형 웹을 일일이 밑바닥부터 개발하는 것은 비효율적이므로, 이미 완성된 디자인 기술과 도구를 활용하기 위함
* **Bootstrap 특징:** 트위터 내부에서 UI/UX 개발을 위해 만들어진 후 오픈소스로 공개됨. 현재 5.3 버전이 최신이며 전 세계적으로 널리 쓰이는 프론트엔드 프레임워크

### 2. CDN (Content Delivery Network) 연동
* **개념:** 컨텐츠를 사용자와 물리적으로 가장 가까운 '엣지 서버'에서 빠르게 제공하는 네트워크 기술
* **작동 원리:** 미국 등 먼 곳에 있는 '오리진 서버'까지 갈 필요 없이, 지역별로 분산된 엣지 서버(예: jsdeliver)에서 복사본 데이터를 가져옴
* **장점:** 물리적 거리를 줄여 로딩 속도 향상, 중앙 서버 과부하 방지, 엣지 서버 장애 시 다른 서버로 우회 가능하여 안정성 확보
* **Bootstrap 파일 구조:**
  * `bootstrap.min.css`: 12,000줄짜리 CSS 원본 코드를 공백 없이 한 줄로 압축(minified)하여 로딩 속도를 최적화한 파일
  * `bootstrap.bundle.min.js`: 드롭다운, 모달, 캐러셀 등 동적 상호작용을 담당하는 JavaScript 파일

### 3. Reset CSS와 Normalize.css
* **발생 원인 (User Agent Stylesheet):** 브라우저(크롬, 사파리, 엣지 등)마다 자체적으로 부여하는 기본 스타일이 모두 달라, 똑같은 HTML 코드라도 브라우저에 따라 여백과 폰트 등이 다르게(깨져서) 보이는 문제 발생
* **해결책 (Reset CSS):** 모든 브라우저의 기본 스타일을 지워버리고(초기화) 동일한 출발선에서 개발을 시작하도록 강제하는 방법론
* **Normalize.css:** Bootstrap이 채택한 초기화 방식. 모든 걸 지우기보다 웹 표준을 기준으로 맞추고, 비표준 브라우저(구 IE 등)의 차이를 보정해 주는 약간 '너그러운' 초기화 방식 (`reboot.css`라는 약 600줄짜리 파일로 내장되어 있음)

### 4. Bootstrap 스타일링 규칙 (클래스 기반)
* **방식:** CSS 파일을 직접 작성하지 않고, Bootstrap이 이미 만들어둔 클래스 이름을 HTML 태그에 조합하여 디자인함
* **Spacing (여백) 규칙:** `[속성][방향]-[사이즈]` 포맷 사용
  * 속성: `m` (margin), `p` (padding)
  * 방향: `t` (top), `b` (bottom), `s` (start/left), `e` (end/right), `x` (좌우), `y` (상하), 미기입 시 4방향 전체
  * 사이즈: `0` ~ `5` (5는 3rem, 약 48px), 그리고 `auto`
  * 예: `mt-5` (위쪽 마진 5사이즈), `px-3` (좌우 패딩 3사이즈)
* **Color (색상) 규칙:** 의미론적 이름 사용
  * `primary`(파랑, 기본), `success`(초록, 성공), `danger`(빨강, 위험/에러), `warning`(노랑, 경고), `info`(하늘색) 등
  * 텍스트 색상 예: `text-danger`
  * 배경 색상 예: `bg-info`

### 5. 컴포넌트 (Component) 핵심
* **개념:** 재사용 가능한 독립적인 부품(레고 블럭). 디자인과 기능(CSS+JS)이 하나로 묶여 있어 조립만 하면 복잡한 웹페이지를 쉽게 구현 가능
* **주요 컴포넌트:**
  * **Alert / Badge:** 알림창 및 작은 라벨링 (배지는 `position: absolute`와 자주 결합됨)
  * **Button:** 호버 효과 및 마우스 커서 변경이 내장된 버튼 (`btn btn-primary`)
  * **Card:** 이미지, 제목, 본문, 버튼 등을 담는 네모난 레이아웃 컨테이너 (실무 활용도 최상)
  * **Navbar:** 최상단 네비게이션 바. 화면을 줄이면 햄버거 버튼으로 접히는 반응형 기능이 내장됨 (`data-bs-theme="dark"` 등으로 테마 변경 가능)
  * **Carousel:** 이미지를 좌우로 슬라이드하는 슬라이더. **주의:** 한 화면에 2개 이상 사용할 경우, JavaScript가 타겟을 인식할 수 있도록 HTML `id` 값과 버튼의 `data-bs-target` 속성 값을 반드시 쌍으로 고유하게 맞춰줘야 함
  * **Modal:** 클릭 시 배경이 어두워지고 튀어나오는 팝업창. **주의사항 2가지 필수 암기:**
    1. 버튼 코드와 모달 본문 코드가 HTML상에서 반드시 같이 붙어있을 필요 없음
    2. 모달 코드가 다른 컨테이너 깊숙이 중첩되어 있으면 Z-index 문제로 검은 배경 뒤로 깔려버려 클릭할 수 없는 먹통 상태가 됨. 따라서 모달 코드는 `</body>` 태그가 닫히기 직전 맨 아래에 독립적으로 모아두는 것이 실무 표준

---

## 💻 기능 구현 및 코드 실습

### 1. CDN 연동 및 Bootstrap 클래스 적용 (Color & Spacing)
```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <link href="[https://cdn.jsdelivr.net/npm/bootstrap@5.3.x/dist/css/bootstrap.min.css](https://cdn.jsdelivr.net/npm/bootstrap@5.3.x/dist/css/bootstrap.min.css)" rel="stylesheet">
</head>
<body>
  <div class="bg-info border border-dark border-2 p-3 mt-5" style="width: 300px; height: 200px;">
    <p class="text-danger fw-bold">Bootstrap Box Test</p>
  </div>

  <script src="[https://cdn.jsdelivr.net/npm/bootstrap@5.3.x/dist/js/bootstrap.bundle.min.js](https://cdn.jsdelivr.net/npm/bootstrap@5.3.x/dist/js/bootstrap.bundle.min.js)"></script>
</body>
</html>
```

### 2. 다중 캐러셀(Carousel) 충돌 방지 구조 설계
```html
<div id="carousel-first" class="carousel slide">
  <div class="carousel-inner">
    <div class="carousel-item active"><img src="img1.jpg" class="d-block w-100" alt="1"></div>
    <div class="carousel-item"><img src="img2.jpg" class="d-block w-100" alt="2"></div>
  </div>
  <button class="carousel-control-prev" type="button" data-bs-target="#carousel-first" data-bs-slide="prev">
    <span class="carousel-control-prev-icon" aria-hidden="true"></span>
  </button>
</div>

<div id="carousel-second" class="carousel slide mt-5">
  <div class="carousel-inner">
    <div class="carousel-item active"><img src="img4.jpg" class="d-block w-100" alt="4"></div>
  </div>
  <button class="carousel-control-prev" type="button" data-bs-target="#carousel-second" data-bs-slide="prev">
    <span class="carousel-control-prev-icon" aria-hidden="true"></span>
  </button>
</div>
```

### 3. 모달(Modal) 팝업창 레이아웃 분리 구조
```html
<body>
  <div class="content-area">
    <h1>메인 컨텐츠</h1>
    <button type="button" class="btn btn-primary" data-bs-toggle="modal" data-bs-target="#myModal">
      모달 띄우기
    </button>
  </div>

  <div class="modal fade" id="myModal" tabindex="-1">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">알림</h5>
          <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
        </div>
        <div class="modal-body">
          <p>독립적인 모달 팝업 내용입니다</p>
        </div>
      </div>
    </div>
  </div>

  <script src="[https://cdn.jsdelivr.net/npm/bootstrap@5.3.x/dist/js/bootstrap.bundle.min.js](https://cdn.jsdelivr.net/npm/bootstrap@5.3.x/dist/js/bootstrap.bundle.min.js)"></script>
</body>
```

---

## 🚀 복습 및 AI 인사이트
* **헷갈렸던 점 1 (CDN 적용 시 디자인 붕괴):** Bootstrap CDN 링크를 추가했더니 기존에 작성해둔 내 CSS 디자인이 이상해지거나 폰트 굵기/여백이 싹 사라졌다면 버그가 아님. Bootstrap 내부에 있는 `reboot.css`가 브라우저 편차를 맞추기 위해 모든 기본 스타일을 리셋(초기화)해 버렸기 때문임을 인지해야 함
* **헷갈렸던 점 2 (컴포넌트 먹통 현상):** 캐러셀 슬라이드가 안 넘어가거나, 모달 버튼을 눌러도 반응이 없다면 1순위로 맨 아래 `<script>` (JS CDN) 코드가 빠지지 않았는지 확인. 2순위로 여러 개를 복붙하며 `id`와 `data-bs-target` 속성값이 엇갈리지 않았는지 점검 필수
* **AI 활용 팁:** Bootstrap 컴포넌트 커스텀 시, AI에게 단순히 "이거 바꿔줘"라고 하기보다는 *"Bootstrap 5.3 버전을 기준으로, 이 캐러셀 컴포넌트의 버튼 색상과 전환 속도를 덮어씌울 수 있는(Override) 정확한 CSS 클래스나 SCSS 변수 접근법을 알려줘"* 라고 프롬프팅하면, 레거시 문법이 섞이지 않은 깔끔한 오버라이딩 코드를 얻을 수 있음