# [Web] Bootstrap & 반응형 웹 — 프레임워크, Grid System, 반응형 레이아웃

> **핵심 키워드:** #Bootstrap #CSS프레임워크 #CDN #ResetCSS #Normalize #Reboot #Spacing #Typography #Component #Carousel #Modal #Navbar #Card #SemanticWeb #반응형웹 #ResponsiveDesign #UX #UI #GridSystem #Container #Row #Column #Breakpoint #Offset #Gutter #MediaQuery #Emmet

---

## 학습 목표

* Bootstrap의 개념(CSS 프레임워크)과 CDN 방식의 적용 원리를 이해한다
* Reset CSS(Normalize CSS)의 필요성과 브라우저 간 스타일 차이를 파악한다
* Bootstrap의 Spacing, Typography, Color, Component 체계를 활용할 수 있다
* Bootstrap Grid System의 12칸 구조(Container → Row → Column)를 이해하고 레이아웃을 구성할 수 있다
* Breakpoint를 활용하여 화면 크기별 칸 수를 재배분하는 반응형 웹을 구현할 수 있다
* Grid System, Flexbox, Position의 역할 차이를 구분하고 상황에 맞게 조합할 수 있다

---

## 1. Bootstrap 기초

### 1-1. Bootstrap이란

Bootstrap은 CSS 프론트엔드 프레임워크(툴킷)이다. 프레임워크란 소프트웨어 개발 시 반복적인 작업과 설정에 필요한 공통 기능·구조를 미리 만들어 제공하는 개발 환경으로, 개발을 시작할 때 활용할 수 있는 기본 토대이다.

Bootstrap은 원래 Twitter 내부에서 공통 UI/UX를 디자인·개발하기 위해 만들어졌고, 이후 오픈소스로 공개되면서 누구나 활용할 수 있게 되었다. GitHub Star 랭킹 20위권에 위치하며, 현재 최신 버전은 5.3이다.

> **강사님 주의**: Bootstrap 한글 번역 사이트는 누락이 많으므로 반드시 영문 공식 문서를 보라. 또한 과거 버전(3.x, 4.x) 문서에 들어가지 않도록 주의하라. 사이트 색상과 버전 번호를 꼭 확인하라.

### 1-2. CDN을 이용한 적용

Bootstrap을 웹페이지에 적용하려면 `<head>`에 CSS 파일, `</body>` 직전에 JavaScript 파일을 CDN으로 가져온다.

```html
<!-- head 안에 CSS -->
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.x/dist/css/bootstrap.min.css" rel="stylesheet">

<!-- body 닫히기 직전에 JS -->
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.x/dist/js/bootstrap.bundle.min.js"></script>
```

CDN(Content Delivery Network)은 전 세계 곳곳에 분산된 엣지 서버를 통해 콘텐츠를 사용자와 가장 가까운 곳에서 전달하는 기술이다. 미국에 있는 원본(Origin) 서버까지 가지 않아도 되므로 로딩 속도가 빠르고, 서버 부하를 분산시키며, 특정 엣지 서버에 장애가 생겨도 다른 서버로 대체되어 안정성이 높다. Bootstrap은 jsdelivr라는 CDN 업체를 사용한다.

### 1-3. Bootstrap의 내부 구조

CDN으로 가져오는 `bootstrap.min.css`는 약 12,000줄짜리 CSS 코드를 한 줄로 압축(Minified)한 파일이다. JavaScript도 약 5,000줄 규모의 `bootstrap.bundle.min.js`로 제공된다. 파일을 직접 다운로드하여 사용할 수도 있으며, 기능별로 분리된 파일(Grid, Reboot, Utilities 등)도 제공된다.

이 12,000줄 안에 수많은 클래스 선택자가 미리 정의되어 있으며, id 선택자는 사용되지 않는다. 우리는 공식 문서에서 이 클래스 이름들을 확인하면서 활용한다.

> **강사님 강조**: Bootstrap은 이미 잘 만들어진 도구 모음이다. 외우려고 보는 게 아니라 구조를 익히는 것이 중요하다. 공식 문서를 항상 옆에 띄워두고 진행하라.

---

## 2. Reset CSS

### 2-1. User Agent Stylesheet의 문제

모든 브라우저에는 HTML 요소에 기본 스타일을 부여하는 User Agent Stylesheet가 존재한다. 예를 들어 `<h1>` 태그는 아무 스타일링을 하지 않아도 큰 폰트, 볼드, 위아래 마진이 적용된다.

문제는 이 기본 스타일이 브라우저마다 다르다는 것이다. 같은 HTML 파일이라도 Chrome과 Firefox에서 폰트, 여백, 크기가 다르게 보인다. 개발자가 모든 브라우저에 맞춰 따로따로 개발하는 것은 불가능하므로, 모든 브라우저의 시작점을 통일하는 작업이 필요하다.

### 2-2. Normalize CSS와 Bootstrap Reboot

Reset CSS는 브라우저 기본 스타일을 일관된 기준으로 재설정하는 방법론이다. 다양한 방식이 있는데, 가장 대중적인 것이 Normalize CSS이다. 이 방식은 웹 표준을 기준으로 맞추되, 표준을 따르지 못하는 브라우저(대표적으로 Internet Explorer)에 다른 브라우저들이 맞춰주는 "너그러운 방식"이다.

Bootstrap은 Normalize CSS를 채택하여 자체적으로 커스텀한 `reboot.css`(약 600줄)를 사용한다. 이 코드에는 전체 선택자에 `box-sizing: border-box`, `<body>`에 `margin: 0`, 헤딩 태그에 `margin-top: 0` 등의 초기화가 포함되어 있다. 이 Reboot 코드는 `bootstrap.min.css`에 이미 포함되어 있으므로 별도로 추가할 필요 없다.

> **강사님 팁**: Bootstrap 적용 후 기존과 스타일이 달라졌다면 초기화된 상태로 진행하고 있기 때문이다. 당황하지 말라.

---

## 3. Bootstrap 활용

### 3-1. Spacing (여백)

Bootstrap의 여백 클래스는 `{속성}{방향}-{크기}` 패턴으로 구성된다.

| 구분 | 값 | 설명 |
|------|-----|------|
| 속성 | `m` / `p` | margin / padding |
| 방향 | `t` `b` `s` `e` `x` `y` (생략) | top / bottom / start(좌) / end(우) / 좌우 / 상하 / 4방향 |
| 크기 | 0 ~ 5, auto | 0=0 / 1=0.25rem / 2=0.5rem / 3=1rem / 4=1.5rem / 5=3rem |

예시: `mt-5`는 margin-top 3rem(48px), `px-3`은 padding 좌우 1rem, `m-0`은 margin 4방향 0이다. rem 단위를 사용하며, 6 이상의 크기가 필요하면 직접 CSS를 작성해야 한다.

> **강사님 강조**: 이 숫자는 직접적인 픽셀·rem 값이 아니라 Bootstrap 팀이 정해둔 넘버링이다. 5로 갈수록 커지지만, 5가 곧 5rem은 아니다.

### 3-2. Typography

Bootstrap은 기본 HTML 태그의 스타일을 리셋한 뒤, 자체 Typography 클래스를 제공한다.

Display Heading은 기존 `<h1>`~`<h6>`보다 더 크고 강렬한 제목이 필요할 때 사용하며, `class="display-1"` ~ `display-6`으로 적용한다. `list-unstyled` 클래스를 사용하면 리스트의 불릿 포인트를 제거할 수 있다.

### 3-3. Color System

Bootstrap은 색상을 의미 기반으로 명명한다. `blue` 대신 `primary`, `red` 대신 `danger`, `green`은 `success`, `yellow`는 `warning`, 하늘색은 `info`로 사용한다.

텍스트 색상은 `text-{색상}`, 배경색은 `bg-{색상}` 패턴이다. 예를 들어 `text-danger`는 빨간 텍스트, `bg-info`는 하늘색 배경이다. Bootstrap 색상을 꼭 써야 하는 것은 아니며, 원하는 CSS 색상을 자유롭게 사용할 수 있다.

### 3-4. Component

Component는 재사용 가능한 독립적인 부품으로, HTML·CSS·JavaScript로 구현된 완성된 도구이다. 레고 블록처럼 조합하여 복잡한 결과물을 만들 수 있으며, 각 컴포넌트는 서로에게 영향을 주지 않고 독립적으로 동작한다.

**Alert** — 알림 메시지 용도로, `alert alert-{색상}` 클래스로 적용한다.

**Badge** — 소규모 개수 표시나 라벨링에 사용한다. 단독보다는 버튼 안이나 다른 요소 위에 붙여서 쓰인다. 알림 아이콘 위의 빨간 숫자 배지는 `position: absolute`로 구현된다.

**Button** — `btn btn-{색상}` 클래스로 적용하며, 마우스 오버 시 색상 변화와 커서 변경까지 구현되어 있다.

**Card** — 가장 많이 쓰이는 컴포넌트 중 하나이다. 카드 형태가 아닌 레이아웃에서도 이 구조를 활용한다. 기본 구조는 `.card` 안에 `.card-img-top`(이미지)과 `.card-body`(본문)가 있고, 본문 안에 `.card-title`, `.card-text` 등이 들어간다. 수평 카드, 색상 변경 등 확장성이 높다.

**Navbar** — 사이트 최상단 내비게이션 바로, 반응형 디자인이 기본 내장되어 있다. 화면이 작아지면 햄버거 버튼으로 축소되고, 커지면 메뉴가 펼쳐진다. `data-bs-theme="dark"` 속성으로 다크 테마를 적용할 수 있다.

**Carousel** — 제한된 공간에서 이미지나 콘텐츠를 슬라이드 형태로 보여주는 컴포넌트이다. 커머스 사이트의 광고 배너에 많이 사용된다.

**Modal** — 버튼 클릭 시 배경이 흐려지며 팝업 창이 뜨는 컴포넌트이다.

### 3-5. 상호작용 컴포넌트 주의사항 — data-bs-target

Carousel, Modal처럼 사용자와 상호작용하는 컴포넌트에는 `data-bs-target` 속성이 존재한다. 이 속성은 `#id값`으로 어떤 요소를 조작할지 지정한다.

같은 페이지에 Carousel을 2개 넣을 때, 둘 다 `id="carouselExample"`이면 두 번째 Carousel의 버튼이 첫 번째 것을 조작하게 된다. 반드시 id를 다르게 설정하고, 버튼의 `data-bs-target`도 해당 id에 맞춰야 한다.

```html
<!-- 두 번째 Carousel: id와 data-bs-target을 변경 -->
<div id="carouselExample-second" class="carousel slide">
  ...
  <button data-bs-target="#carouselExample-second" ...>
</div>
```

> **강사님 주의**: 컴포넌트가 동작하지 않으면 높은 확률로 JavaScript CDN을 누락했거나 data-bs-target의 id가 불일치한 것이다.

**Modal 배치 규칙:**

1. Modal과 버튼 코드가 반드시 함께 다닐 필요는 없다 — 어차피 버튼을 누르기 전에는 Modal이 보이지 않는다
2. Modal 코드를 다른 요소 안에 중첩시키면, 활성화 시 검은 배경 뒤로 깔려 닫을 수 없게 될 위험이 있다
3. Modal 코드는 `</body>` 직전에 모아두는 것이 일반적이다

---

## 4. Semantic Web

Semantic Web(의미론적 웹)은 HTML 요소에 의미를 부여하여 웹페이지의 구조를 더 명확하게 표현하는 개념이다. `<div>` 대신 `<header>`, `<nav>`, `<main>`, `<section>`, `<article>`, `<footer>` 등의 시맨틱 태그를 사용하면 코드의 가독성이 높아지고 접근성과 SEO에도 도움이 된다. 기능적으로는 `<div>`와 동일하지만, 문서의 구조적 의미를 전달한다는 차이가 있다.

---

## 5. 반응형 웹과 UX/UI

### 5-1. 반응형 웹 디자인

반응형 웹 디자인(Responsive Web Design)은 다양한 디바이스(데스크탑, 태블릿, 스마트폰)의 화면 크기에 상관없이 일관적인 레이아웃과 사용자 경험을 제공하는 기술이다. 큰 화면에서 가로로 배치된 상품이 작은 화면에서는 세로로 재배치되거나 일부가 숨겨지는 것이 대표적인 예이다.

Bootstrap 외에도 Tailwind CSS 등 다양한 도구가 있으며, 실무에서는 상황에 맞게 여러 도구를 혼합하여 사용한다.

### 5-2. UX (User Experience)

UX는 사용자가 서비스나 제품을 사용하면서 느끼는 전체적인 경험과 만족도를 개선·최적화하는 분야이다. 감각적 경험(매장의 향기, 패키징), 기능적 경험(오타가 있어도 검색 결과를 보여주는 것), 편의적 경험 등이 모두 포함된다.

공원의 "희망보행로" 사례처럼, 아무리 예쁘게 설계해도 사용자는 더 빠르고 효율적인 길을 선택한다. 개발자는 단순히 예쁜 것을 넘어 사용자의 실제 행동 패턴을 조사하고 데이터를 분석해야 한다.

### 5-3. UI (User Interface)

UI는 서비스와 사용자 간의 직접적인 상호작용을 담당하는 영역이다. 물리적 UI(리모컨의 버튼 배치·크기·그립감), 복합적 UI(ATM 기기의 터치 화면과 물리 키패드), 디지털 UI(로그인 버튼의 색상·위치·피드백) 등이 있다.

UI 설계의 핵심은 단순히 예쁘게 만드는 것이 아니라, 정보의 위계(중요도)를 정리하고 일관성 있게 배치하여 사용자가 불필요한 경험을 느끼지 않게 하는 것이다. 결국 잘 만든 UI를 통해 좋은 UX를 경험하게 된다.

> **강사님 팁**: 프로젝트 시 Samsung One UI, Apple Human Interface Guidelines, Google Material Design 등 기업 UI 가이드라인의 데이터를 AI에 사전 설계로 넣어두면 더 좋은 프론트엔드 결과를 얻을 수 있다.

---

## 6. Bootstrap Grid System

### 6-1. 12칸 Grid의 원리

Bootstrap Grid System은 하나의 화면을 12개의 Column으로 나누고, 각 요소에게 칸 수를 배분하는 레이아웃 시스템이다. 12를 선택한 이유는 적절한 크기의 짝수 중 약수가 많기 때문이다(1, 2, 3, 4, 6, 12). 나누어 떨어지는 수가 많을수록 다양한 기기에서 유연한 칸 수 조정이 가능하다.

Grid System의 구조는 Container → Row → Column 3단계이다.

| 요소 | 클래스 | 역할 |
|------|--------|------|
| Container | `.container` | 12칸을 담는 큰 박스. 콘텐츠를 가운데로 모아주는 역할 |
| Row | `.row` | 하나의 행. 같은 Row 안의 요소는 같은 결의 콘텐츠 |
| Column | `.col-{n}` | 실제 콘텐츠 영역. n은 12칸 중 차지할 칸 수 |
| Gutter | — | Column 간의 여백 |

```html
<div class="container">
  <div class="row">
    <div class="col-4">4칸</div>
    <div class="col-4">4칸</div>
    <div class="col-4">4칸</div>
  </div>
</div>
```

숫자를 생략하고 `col`만 쓰면 자동 균등 분배되지만, 명시적이지 않으므로 칸 수를 직접 작성하는 것이 권장된다. 12칸을 초과하면 넘치는 요소가 자동으로 다음 줄로 내려간다.

> **강사님 강조**: VS Code에서 div를 접는 기능을 활용하여 큰 그림부터 작은 그림으로 구조를 확인하라. 하나씩 접어가면서 Container → Row → Column 구조가 맞는지 체크하는 습관이 중요하다.

### 6-2. Nesting (중첩)

하나의 Column 안에 다시 Row → Column 구조를 넣을 수 있다. 구글 뉴스처럼 큰 영역을 나눈 뒤, 각 영역 내부에서 다시 12칸 Grid를 독립적으로 운영하는 형태이다.

```html
<div class="row">
  <div class="col-4">왼쪽 사이드바</div>
  <div class="col-8">
    <!-- 이 안에서 새로운 12칸 시작 -->
    <div class="row">
      <div class="col-6">내부 1</div>
      <div class="col-6">내부 2</div>
      <div class="col-6">내부 3</div>
      <div class="col-6">내부 4</div>
    </div>
  </div>
</div>
```

Container는 하나만 존재하는 것이 아니며, Column이 또 다른 Container 역할을 할 수 있다.

### 6-3. Offset (상쇄)

12칸을 모두 채우지 않고 특정 Column 앞에 빈 칸을 두고 싶을 때 `offset-{n}`을 사용한다.

```html
<!-- 4칸 | 빈 4칸 | 4칸 -->
<div class="col-4">A</div>
<div class="col-4 offset-4">B</div>

<!-- 가운데 6칸만 사용 -->
<div class="col-6 offset-3">가운데 정렬</div>
```

Offset은 해당 Column의 왼쪽에 상쇄 영역을 만든다. "이 Column은 n칸을 상쇄시키고 등장한다"로 이해하면 된다.

> **강사님 주의**: Breakpoint와 함께 사용할 때, `offset-sm-4`는 "sm 이상에서 4칸 상쇄"이므로 md 이상에서도 계속 적용된다. md에서 상쇄를 해제하려면 `offset-md-0`을 명시해야 한다.

### 6-4. Gutters (여백)

Gutter는 Column 간의 여백으로, Row에 적용하여 해당 행의 모든 Column에 일괄 반영한다.

| 클래스 | 방향 | 방식 |
|--------|------|------|
| `gx-{0~5}` | 좌우 | Column의 **padding**으로 조정 (콘텐츠가 좁아짐) |
| `gy-{0~5}` | 상하 | Column의 **margin**으로 조정 (물리적으로 밀어냄) |
| `g-{0~5}` | 4방향 | 좌우는 padding, 상하는 margin |

좌우 Gutter가 padding인 이유는 Column을 margin으로 밀어내면 Container의 12칸 구조가 깨지기 때문이다. Column의 너비는 그대로 유지하면서 내부 콘텐츠 영역만 줄여 여백처럼 보이게 한다. 반면 상하는 12칸 제한이 없으므로 margin으로 직접 밀어낸다.

### 6-5. Breakpoints — 반응형 레이아웃

Breakpoint는 화면 크기에 따라 레이아웃이 바뀌는 분기점이다. Bootstrap은 6개의 Breakpoint를 정의해 두었다.

| Breakpoint | 키워드 | 너비 기준 |
|------------|--------|-----------|
| Extra Small | (생략) | < 576px |
| Small | `sm` | ≥ 576px |
| Medium | `md` | ≥ 768px |
| Large | `lg` | ≥ 992px |
| Extra Large | `xl` | ≥ 1200px |
| 2X Large | `xxl` | ≥ 1400px |

핵심은 "~~ 이상"이라는 것이다. `col-sm-6`은 "sm 이상일 때 6칸"이며, md·lg·xl에서도 다른 설정이 없으면 계속 6칸이 적용된다. Extra Small은 키워드 없이 `col-12`로 작성한다.

```html
<div class="col-12 col-sm-6 col-md-2 col-lg-3">
  <!-- xs: 12칸(한 줄) → sm: 6칸(반반) → md: 2칸 → lg: 3칸 -->
</div>
```

Offset도 동일하게 Breakpoint를 적용한다. `offset-sm-4`는 sm 이상에서 4칸 상쇄이며, 이후 사이즈에서 해제하려면 `offset-md-0`처럼 0으로 명시해야 한다.

내부적으로는 CSS의 Media Query로 구현되어 있으며, Bootstrap 파일 안에 각 Breakpoint별 코드가 작성되어 있다. 우리는 클래스만으로 간편하게 사용한다.

---

## 7. CSS Layout 종합 정리

에어비앤비 같은 실제 웹사이트를 분석해 보면, Grid System·Flexbox·Position이 각각 다른 역할을 수행하고 있다.

| 기술 | 역할 | 비유 |
|------|------|------|
| Grid System | 웹페이지 전체의 큰 구조·영역 배분 | 건물의 철골 구조, 방 배정 |
| Flexbox | Grid로 나눈 구역 내부의 콘텐츠 정렬·배치 | 방 안의 가구 배치 |
| Position | 특정 요소를 고정 위치에 배치 (absolute, fixed, sticky) | 벽에 붙인 액자, 천장 조명 |

Grid System은 12칸을 나누고 화면 크기에 따라 칸 수를 재배분하는 "큰 그림"을 그리는 도구이다. Flexbox는 그 안에서 콘텐츠를 주축·교차축 기준으로 섬세하게 정렬한다. Position의 absolute는 특정 요소 위에 배지나 좋아요 버튼을 올리고, fixed/sticky는 내비게이션 바를 상단에 고정한다.

이 기술들은 상호보완적이며, 나중에 나온 기술이 이전 기술을 대체하는 것이 아니다. 각각 고유한 특성과 장단점이 있어 상황에 따라 적합한 도구가 달라진다.

> **강사님 강조**: 전체 숲은 Grid System으로 빠르게 짓고, 거기에 배치되는 나무들은 Flexbox로 섬세하게 다듬고, 더 섬세한 위치 이동은 Position으로 살펴보라. 이것이 다양한 CSS 레이아웃 기술을 배운 이유이다.

---

## 8. 참고

### 8-1. Grid Cards

Bootstrap은 카드 배치 전용 Breakpoint인 Grid Cards를 제공한다. 기존 Grid System은 Column에서 칸 수를 제어하지만, Grid Cards는 Row에서 `row-cols-{n}` 형태로 카드 출력 개수를 제어한다.

```html
<!-- row-cols: 숫자는 "칸 수"가 아니라 "카드 개수" -->
<div class="row row-cols-1 row-cols-sm-2 row-cols-md-3">
  <div class="col"><div class="card">...</div></div>
  ...
</div>
```

`row-cols-sm-2`는 "sm 이상일 때 한 줄에 카드 2개"라는 뜻이다. 몰라도 기존 Grid System으로 동일하게 구현 가능하지만, 카드를 다룰 때는 더 직관적이다.

### 8-2. Emmet

Emmet은 VS Code에 내장된 코드 자동완성 도구로, CSS 선택자·결합자 문법을 활용한다. `>` 는 자식 결합자, `+`는 형제 선택, `.`은 클래스, `#`은 id, `*n`은 반복이다. `div`는 가장 많이 사용되므로 생략 가능하여 `.container`만 입력해도 `<div class="container"></div>`가 생성된다.

```
section.container>div.card*3>img.thumbnail+div>h3.title+button.btn
```

위 한 줄로 section 안에 card 3개, 각각 이미지와 제목·버튼이 포함된 구조가 한 번에 생성된다. Emmet Cheat Sheet를 참고하면 더 다양한 단축 문법을 확인할 수 있다.

### 8-3. 올바른 UI 판별 게임

can't unsee(cantunsee.space)는 두 개의 UI를 비교하여 더 나은 쪽을 선택하는 게임이다. 뒤로 갈수록 디테일한 차이를 구별해야 하며, UI 감각을 기르는 데 도움이 된다.

---

## 정리

| 구분 | 핵심 포인트 |
|------|-------------|
| Bootstrap | CSS 프론트엔드 프레임워크. 12,000줄 CSS를 CDN 한 줄로 가져와 클래스로 활용 |
| CDN | 전 세계 엣지 서버를 통해 콘텐츠를 빠르게 전달하는 네트워크 |
| Reset CSS | 브라우저별 기본 스타일 차이를 통일. Bootstrap은 Normalize 기반 Reboot 사용 |
| Spacing | `{m/p}{방향}-{0~5}` 패턴. rem 단위, 5가 최대(3rem) |
| Color | 의미 기반 명명: primary(파랑), danger(빨강), success(초록), warning(노랑) |
| Component | 재사용 가능한 독립 부품. Card, Navbar, Carousel, Modal 등 |
| data-bs-target | 상호작용 컴포넌트의 id 연결. 같은 컴포넌트 여러 개 사용 시 id를 다르게 설정 |
| Grid System | Container → Row → Column. 12칸을 나눠주는 레이아웃 시스템 |
| Breakpoint | xs / sm(≥576) / md(≥768) / lg(≥992) / xl(≥1200) / xxl(≥1400). "이상"으로 적용 |
| Offset | Column 앞에 빈 칸 생성. Breakpoint와 함께 사용 시 해제(`offset-{bp}-0`) 주의 |
| Gutter | Column 간 여백. 좌우는 padding, 상하는 margin. Row에 적용 |
| Grid vs Flexbox vs Position | 큰 구조(Grid) → 내부 정렬(Flex) → 특수 고정(Position). 상호보완적 |
