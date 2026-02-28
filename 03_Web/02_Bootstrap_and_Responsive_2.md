# [Web] 반응형 웹 (Responsive Web)

## 1. 반응형 웹 (Responsive Web)과 UX/UI
다양한 기기(스마트폰, 태블릿, 모니터 등)의 화면 크기에 맞춰 일관된 레이아웃과 최적의 경험을 제공하는 기술입니다.

* **UX (User Experience, 사용자 경험):** 사용자가 서비스를 이용하며 느끼는 전체적인 만족감과 경험. 사용자 니즈 분석, 페르소나 설정, 프로토타입 작성 등을 통해 설계됩니다. (예: 러쉬 매장의 향기, 검색 오타 보정 기능)
* **UI (User Interface, 사용자 인터페이스):** 사용자가 서비스와 직접 만나는 시각적 디자인과 요소의 배치. 정보의 위계와 일관성을 고려하여 개발됩니다. (예: 리모컨 버튼의 배치, 로그인 버튼의 위치와 색상)

결국 **잘 설계된 UI를 통해 최적의 UX를 제공**하는 것이 반응형 웹의 궁극적인 목표입니다.

---

## 2. 부트스트랩 그리드 시스템 (Bootstrap Grid System) 핵심 구조
화면의 레이아웃을 잡을 때 **하나의 행(Row)을 12개의 칸(Column)으로 나누어 배치**하는 시스템입니다. (12를 사용하는 이유는 1, 2, 3, 4, 6, 12 등 약수가 많아 화면 분할에 매우 유리하기 때문입니다.)

### 기본 문법과 구조
* `.container`: 그리드 시스템을 감싸는 가장 큰 도화지 (양쪽 여백을 주어 시선을 모아줌)
* `.row`: 12개의 컬럼을 포함하는 하나의 행
* `.col`: 실제 콘텐츠가 들어가는 칸 (12칸 중 몇 칸을 차지할지 숫자로 지정)

```html
<div class="container">
  <div class="row">
    <div class="col">자동 배분 1</div>
    <div class="col">자동 배분 2</div>
    <div class="col">자동 배분 3</div>
  </div>

  <div class="row">
    <div class="col-4">4칸 차지</div>
    <div class="col-4">4칸 차지</div>
    <div class="col-4">4칸 차지</div>
  </div>

  <div class="row">
    <div class="col-2">2칸 차지</div>
    <div class="col-8">8칸 차지 (메인 콘텐츠)</div>
    <div class="col-2">2칸 차지</div>
  </div>
</div>
```

---

## 3. 중첩 (Nesting)
컬럼(`.col`) 안에 또 다른 행(`.row`)을 넣어 그 안에서 새로운 12칸의 그리드 시스템을 시작할 수 있습니다.

```html
<div class="container">
  <div class="row">
    <div class="col-4">좌측 사이드바 (4칸)</div>
    
    <div class="col-8">
      <div class="row">
        <div class="col-6">중첩된 박스 1 (절반)</div>
        <div class="col-6">중첩된 박스 2 (절반)</div>
      </div>
    </div>
  </div>
</div>
```

---

## 4. 오프셋 (Offset)
콘텐츠를 화면의 특정 위치로 밀어내기 위해 **빈 칸(여백)을 상쇄(건너뛰기)**하는 기능입니다. 적용된 요소의 '왼쪽'에 빈 공간을 만듭니다.

```html
<div class="container">
  <div class="row">
    <div class="col-4 offset-4">가운데 4칸만 차지하는 콘텐츠</div>
  </div>
</div>
```

---

## 5. 거터 (Gutters)
컬럼과 컬럼 사이의 간격(여백)을 의미합니다. 컨트롤은 개별 `.col`이 아닌 부모인 `.row`에서 수행합니다. (`g-0`부터 `g-5`까지 설정 가능)

* **좌우 거터 (`gx-*`):** Padding으로 조절됩니다. 12칸의 전체 Container 너비를 깨지 않기 위해, Margin으로 밀어내지 않고 내부 콘텐츠의 크기를 줄이는(Padding) 방식을 씁니다.
* **상하 거터 (`gy-*`):** Margin으로 조절됩니다. 상하는 제약이 없으므로 실제로 요소를 밀어냅니다.

```html
<div class="container">
  <div class="row gx-0">
    <div class="col-6">좌측 콘텐츠</div>
    <div class="col-6">우측 콘텐츠</div>
  </div>

  <div class="row gy-5">
    <div class="col-6">위쪽 콘텐츠</div>
    <div class="col-6">위쪽 콘텐츠</div>
    <div class="col-6">아래쪽 콘텐츠</div>
    <div class="col-6">아래쪽 콘텐츠</div>
  </div>
</div>
```

---

## 6. 반응형 브레이크 포인트 (Breakpoints)
화면 크기에 따라 컬럼이 차지하는 칸 수를 다르게 배정하는 반응형 웹의 핵심 기능입니다. 
* **키워드:** `sm`(≥576px), `md`(≥768px), `lg`(≥992px), `xl`(≥1200px), `xxl`(≥1400px)
* **동작 원리:** 기준점 **"이상으로"** 계속 적용됩니다. (예: `sm`을 주면 `md`, `lg` 화면에서도 덮어쓰지 않는 한 동일하게 적용됨)

```html
<div class="container">
  <div class="row">
    <div class="col-12 col-sm-6 col-md-3">반응형 박스 1</div>
    <div class="col-12 col-sm-6 col-md-3">반응형 박스 2</div>
    <div class="col-12 col-sm-6 col-md-3">반응형 박스 3</div>
    <div class="col-12 col-sm-6 col-md-3">반응형 박스 4</div>
  </div>
</div>
```
> **주의사항 (오프셋 결합 시):** `offset-sm-4`를 주면 브레이크 포인트 특성상 `md` 화면에서도 계속 4칸이 비워집니다. 큰 화면에서 오프셋을 없애려면 `offset-md-0`을 명시적으로 추가해야 합니다.

---

## 7. CSS 레이아웃 기술 요약 (역할 분담)
각 기술은 고유한 목적이 있으며, 실무에서는 이를 혼합하여 사용합니다.
1. **Grid System (그리드):** 화면 전체의 거대한 레이아웃과 구역(방 크기)을 나눌 때 사용.
2. **Flexbox (플렉스박스):** 그리드로 나눠진 구역 내부에서 텍스트나 버튼 등의 요소를 정렬(`justify-content`, `align-items`)할 때 사용.
3. **Position (포지션):** 좋아요 하트 아이콘, 스크롤을 따라다니는 내비게이션 바 등 화면 특정 위치에 요소를 띄우거나 고정할 때 사용 (`absolute`, `fixed`, `sticky`).

---

## 8. 부록: 에밋(Emmet) 활용법
HTML과 CSS를 빠르게 작성할 수 있는 자동 완성 기능입니다. 선택자(태그, `.클래스`, `#아이디`)와 결합자(`>`, `+`, `*`)를 활용합니다.

```html
<section class="container">
  <div class="card">
    <div class="tag"></div>
    <div class="title"></div>
    <div class="btn"></div>
  </div>
  <div class="card">
    <div class="tag"></div>
    <div class="title"></div>
    <div class="btn"></div>
  </div>
  <div class="card">
    <div class="tag"></div>
    <div class="title"></div>
    <div class="btn"></div>
  </div>
</section>
```