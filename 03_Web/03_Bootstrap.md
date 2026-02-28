# [Web] Bootstrap (CSS 프레임워크와 컴포넌트 활용)

> **핵심 키워드:** #Bootstrap #Framework #CDN #Reset_CSS #Reboot #Component #Spacing_System

---

## 🎯 학습 목표
* 반복적인 디자인 작업을 줄이고 개발 토대를 제공하는 프레임워크(Framework)의 개념과 컴포넌트 기반 개발 방식 이해
* 물리적 거리를 극복하여 콘텐츠를 빠르게 전달하는 CDN(Content Delivery Network)의 원리와 장점 파악
* 브라우저마다 다른 기본 스타일(User Agent Stylesheet)을 통일하는 리셋 CSS(Reset CSS)와 부트스트랩의 'Reboot' 메커니즘 분석
* 부트스트랩만의 명명 규칙(Spacing, Color System)을 익히고 공식 문서를 활용한 유틸리티 클래스 적용 능력 배양
* 자바스크립트가 포함된 복합 컴포넌트(Carousel, Modal, Navbar)의 동작 원리와 데이터 속성(`data-bs-*`)을 이용한 연결 로직 숙달

---

## 💡 주요 개념 정리

### 1. 부트스트랩(Bootstrap)과 프레임워크
* **정의:** 트위터에서 개발된 오픈소스 프론트엔드 프레임워크로, 웹 개발에 필요한 공통 구조와 디자인 도구(CSS/JS)를 미리 만들어 제공함
* **특징:** 미리 정의된 클래스 선택자만으로 레이아웃과 디자인을 완성할 수 있으며, 반응형 웹 디자인이 기본적으로 내장되어 있음

### 2. 콘텐츠 전송 네트워크 (CDN)
* **원리:** 원본 서버(Origin)가 아닌, 전 세계 거점에 분산된 엣지 서버(Edge) 중 사용자와 가장 가까운 곳에서 파일을 전달받는 기술
* **장점:** 1. 지리적 한계를 줄여 로딩 속도 향상 2. 중앙 서버 과부하 방지 3. 특정 서버 장애 시 타 서버로 대체되는 안정성 확보
* **구현:** `<link>`와 `<script>` 태그를 통해 외부 주소의 압축된 파일(`min.css`, `min.js`)을 참조함

### 3. 리셋 CSS와 Reboot
* **배경 (User Agent Stylesheet):** 브라우저(크롬, 사파리, 엣지 등)마다 태그에 부여하는 기본 스타일이 제각각이라 동일한 코드가 다르게 보이는 문제 발생
* **해결책 (Normalization):** 모든 브라우저의 스타일을 웹 표준에 맞춰 정규화(Reset)하여 동일한 시작점에서 개발하도록 유도함
* **부트스트랩 Reboot:** `reboot.css`를 통해 `box-sizing: border-box` 설정, `body` 마진 제거, 헤딩 태그 간격 조정 등을 일괄 처리함

### 4. 부트스트랩 시스템 체계
* **Spacing (여백):** `[속성][방향]-[사이즈]` 규칙 준수
    * **속성:** `m`(margin), `p`(padding)
    * **방향:** `t`(top), `b`(bottom), `s`(start/left), `e`(end/right), `x`(좌우), `y`(상하)
    * **사이즈:** 0~5 (단위는 `rem` 기반)
* **Color (색상):** 단순 색상명이 아닌 '의미' 중심의 네이밍 활용
    * `primary`(기본), `secondary`(보조), `success`(성공), `danger`(위험), `warning`(경고), `info`(정보)
    * 적용 방식: `text-primary`, `bg-success`, `btn-danger` 등

### 5. 주요 컴포넌트(Component) 활용 팁
* **Navbar:** 반응형 디자인이 내장되어 있어 너비가 줄어들면 자동으로 '햄버거 버튼'으로 압축됨
* **Carousel (슬라이드):** 한 페이지에 여러 개를 쓸 경우, 각 캐러셀의 `id`와 내부 버튼의 `data-bs-target` 값이 일치해야 꼬이지 않음
* **Modal (팝업):** 레이아웃 구조를 깨뜨리지 않고 배경 뒤로 가려지는 현상을 방지하기 위해, 보통 `<body>` 태그 닫히기 직전 맨 아래에 독립적으로 배치함

---

## 💻 기능 구현 및 코드 실습

### [코드] 부트스트랩 적용 (CDN 방식)
```html
<head>
  <link href="[https://cdn.jsdelivr.net/.../bootstrap.min.css](https://cdn.jsdelivr.net/.../bootstrap.min.css)" rel="stylesheet">
</head>
<body>
  <h1 class="mt-5 text-primary">Hello Bootstrap!</h1>

  <script src="[https://cdn.jsdelivr.net/.../bootstrap.bundle.min.js](https://cdn.jsdelivr.net/.../bootstrap.bundle.min.js)"></script>
</body>
```

### [코드] 박스 모델 및 유틸리티 클래스 조합
```html
<div class="box border border-dark border-2 bg-info p-3 m-4">
  <p class="text-white fw-bold">부트스트랩 클래스 조합 실습</p>
  <button class="btn btn-primary shadow">확인</button>
  <button class="btn btn-outline-danger">취소</button>
</div>
```

### [코드] 모달(Modal) 연결 메커니즘
```html
<button type="button" class="btn btn-primary" data-bs-toggle="modal" data-bs-target="#myUniqueModal">
  모달 열기
</button>

<div class="modal fade" id="myUniqueModal" tabindex="-1">
  <div class="modal-dialog">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title">알림</h5>
        <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
      </div>
      <div class="modal-body">작업이 완료되었습니다.</div>
    </div>
  </div>
</div>
```

---

## 🚀 복습 및 AI 인사이트
* **헷갈렸던 점:**
    * 부트스트랩 적용 시 기존에 직접 짠 스타일이 무너지거나 정렬이 바뀌는 이유. 이는 `reboot.css`가 브라우저 기본 마진과 폰트 설정을 리셋하기 때문이며, 이를 감안하여 초기 레이아웃을 설계해야 함을 인지함
    * 캐러셀(Carousel)이 작동하지 않았던 문제. 여러 개를 복사해 사용할 때 고유 ID를 부여하지 않아 첫 번째 캐러셀만 제어되는 현상을 `id`와 `target` 매칭을 통해 해결함
* **AI 활용 팁:**
    * 특정 디자인(예: "카드 상단에 배지가 붙은 형태")이 필요할 때, 부트스트랩 공식 문서를 다 읽기보다 AI에게 "Bootstrap 5로 카드 컴포넌트 위에 배지를 Absolute로 고정한 코드 예시 보여줘"라고 요청하여 효율적으로 코드 조각(Snippet)을 확보
    * 컴포넌트의 자바스크립트 동작이 멈췄을 때, AI에 HTML 구조를 붙여넣고 "CDN 연결 순서나 data-bs 속성 중 잘못된 부분이 있는지 체크해 줘"라고 질문하여 디버깅 시간 단축