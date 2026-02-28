# [Web] CSS Layout (배치와 정렬)

> **핵심 키워드:** #Display #Normal_Flow #Position #Flexbox #Margin_Collapse

---

## 🎯 학습 목표
* CSS 박스 모델의 기본 흐름인 노멀 플로우(Normal Flow)를 이해하고, `display` 속성에 따른 블록(Block)과 인라인(Inline) 타입의 특징적 차이 분석
* 노멀 플로우에서 요소를 강제로 뜯어내어 자유롭게 배치하는 `position` 속성(`relative`, `absolute`, `fixed`, `sticky`)의 동작 원리와 활용처(배지, 네비게이션 바 등) 숙지
* 현대 웹 레이아웃의 핵심인 `Flexbox`의 개념을 이해하고, 주축(Main Axis)과 교차축(Cross Axis)을 기준으로 한 1차원 정렬 및 공간 배분 기법 체화
* 반응형 카드 디자인 구현을 위해 `flex-wrap`과 `flex-grow`, `flex-basis`를 혼합 적용하는 실전 테크닉 습득
* 인접한 블록 요소 간의 상하 여백이 겹칠 때 발생하는 '마진 상쇄(Margin Collapse)' 현상의 설계 의도와 일관된 여백 관리 원칙 파악

---

## 💡 주요 개념 정리

### 1. 디스플레이 타입과 노멀 플로우 (Normal Flow)
* **노멀 플로우 (Normal Flow):** CSS 레이아웃을 변경하지 않았을 때, 웹 페이지 요소들이 HTML에 작성된 순서대로 자연스럽게 배치되는 기본 흐름
* **블록 (Block) 박스:**
    * 항상 새로운 줄에서 시작하며, 부모 요소의 가로 너비(Width)를 100% 차지함
    * Width, Height, Margin, Padding 조정을 통해 주변 요소를 밀어낼 수 있음 (`div`, `p`, `h1`, `ul` 등)
* **인라인 (Inline) 박스:**
    * 줄바꿈 없이 텍스트의 흐름을 따라 배치되며, 콘텐츠의 크기만큼만 영역을 차지함 (`span`, `a`, `img` 등)
    * Width와 Height를 강제로 지정할 수 없으며, 상하(Top/Bottom) Margin과 Padding은 적용되나 주변 요소를 밀어내지 않음 (좌우는 밀어냄)
* **인라인-블록 (Inline-block):**
    * 줄바꿈이 일어나지 않는 인라인의 특징과, Width/Height/Margin/Padding으로 주변을 밀어낼 수 있는 블록의 특징을 결합한 하이브리드 타입

### 2. 포지션 (Position) 속성
* 요소를 노멀 플로우에서 제거하여 원하는 위치에 강제 배치하거나, 요소끼리 겹치게 만드는 기술
* **`static`:** 기본값. 노멀 플로우를 따르며 `top`, `left` 등의 이동 속성이 무시됨
* **`relative`:** 자신이 `static`일 때 원래 있어야 할 위치를 '기준점'으로 삼아 이동함. 이동하더라도 원래 차지하던 공간은 그대로 보존됨
* **`absolute`:** 노멀 플로우에서 완전히 뜯겨져 나와 원래 공간이 사라짐. 가장 가까운 `position: relative`인 부모 요소를 기준점으로 삼아 이동함 (부모가 없으면 `body` 기준)
* **`fixed`:** 스크롤과 무관하게 사용자가 보는 화면(Viewport)을 기준으로 고정됨 (예: 챗봇 버튼, 플로팅 탑 버튼)
* **`sticky`:** 평소에는 `relative`처럼 동작하다가, 스크롤이 특정 임계점(예: `top: 0`)에 도달하면 `fixed`처럼 고정됨 (예: 스크롤 반응형 네비게이션 바)
* **`z-index`:** 요소가 겹칠 때 화면 앞뒤(Z축)로 쌓이는 순서를 결정하며, 숫자가 클수록 앞에 배치됨 (`static` 요소에는 적용 불가)

### 3. 플렉스박스 (Flexbox) 1차원 레이아웃
* 복잡한 레이아웃(수직/수평 정렬, 균등 배분 등)을 쉽게 구현하기 위해 도입된 이너 디스플레이(Inner Display) 기술
* 부모인 **Flex Container**에 `display: flex;`를 선언하면, 1차 직계 자식들이 **Flex Item**으로 변환되어 통제를 받음
* **주축 (Main Axis) 정렬 (`justify-content`):**
    * 아이템이 흐르는 기본 방향 정렬. (`flex-start`, `center`, `flex-end`, `space-between` 등)
* **교차축 (Cross Axis) 정렬 (`align-`):**
    * `align-items`: 한 줄의 아이템들을 교차축 기준으로 정렬 (기본값 `stretch`)
    * `align-content`: `flex-wrap: wrap`으로 인해 여러 줄이 생겼을 때, 줄 간의 공간 배분
    * `align-self`: 부모의 통제를 벗어나 특정 아이템 하나만 개별적으로 교차축 정렬
* **유연성 제어 속성 (Flex Items에 직접 부여):**
    * `flex-basis`: Flex Item의 초기 너비(Main Axis 크기) 지정. `width`보다 우선순위가 높음
    * `flex-grow`: 컨테이너 내 남는 여백을 아이템들이 나눠 가질 때의 비율(등분 개수)
    * `flex-wrap: wrap`: 아이템의 총 너비가 컨테이너를 초과할 경우, 컨텐츠가 찌그러지지 않게 다음 줄로 떨어뜨림 (반응형의 핵심)

### 4. 마진 상쇄 (Margin Collapse) 현상
* 인접한 블록 박스들이 상하(Top/Bottom)로 마진을 밀어낼 때, 두 마진 값이 합쳐지지 않고 **더 큰 마진 값 하나로 병합(상쇄)**되는 CSS의 의도된 설계
* 목적: 여러 블록 요소가 쌓일 때, 위에서 밀고 아래서 미는 주체가 혼재되어 여백 일관성이 깨지는 것을 방지함. (실무에서는 위에서 아래로(`margin-bottom`)만 여백을 미는 규칙을 권장함)

---

## 💻 기능 구현 및 코드 실습

### [코드] 요소를 겹치게 만드는 Position Absolute (배지 UI)
```css
/* 부모 요소 (기준점 역할) */
.card-container {
  position: relative; /* 자식 absolute 요소의 도화지가 됨 */
  width: 300px;
  height: 400px;
}

/* 자식 요소 (우측 상단에 겹쳐서 고정된 배지) */
.badge {
  position: absolute; /* 노멀 플로우에서 뜯어냄 */
  top: 10px;          /* 부모 컨테이너 기준 위에서 10px */
  right: 10px;        /* 부모 컨테이너 기준 오른쪽에서 10px */
  z-index: 10;        /* 다른 요소들보다 항상 위에 표시 */
}
```

### [코드] 완벽한 수직/수평 중앙 정렬 (Flexbox)
```css
.flex-container {
  display: flex;             /* 플렉스 박스 활성화 */
  height: 100vh;             /* 화면 전체 높이 지정 */
  
  /* 주축(수평) 가운데 정렬 */
  justify-content: center;   
  /* 교차축(수직) 가운데 정렬 */
  align-items: center;       
}
```

### [코드] 반응형 카드 레이아웃 (Wrap, Basis, Grow 혼합)
```css
.card-wrapper {
  display: flex;
  flex-wrap: wrap; /* 너비가 부족하면 요소를 아래로 떨어뜨림 (반응형 핵심) */
}

.card-image {
  flex-basis: 700px; /* 초기 너비 700px 보장 */
  /* 이미지는 고정 크기 유지, 남는 여백을 흡수하지 않음 */
}

.card-text {
  flex-basis: 350px; /* 초기 너비 350px 보장 */
  flex-grow: 1;      /* 컨테이너에 남는 우측 여백이 생기면 이 요소가 쭉 늘어나서 채움 */
}
```

---

## 🚀 복습 및 AI 인사이트
* **헷갈렸던 점:**
    * `position: absolute;` 부여 시 화면 레이아웃이 붕괴되는 현상. Absolute는 노멀 플로우에서 완전히 제거되어 기존 자리를 버리기 때문에, 아래에 있던 요소들이 위로 치고 올라오는 현상이 정상적인 작동 원리임을 명확히 이해함
    * `justify-*`는 `justify-content` 하나만 존재하고 `justify-items`가 없는 이유. 주축(좌우) 정렬의 개별 제어는 `margin-left: auto;` 등의 방식으로 이미 충분히 해결 가능하게 설계되었기 때문임
* **AI 활용 팁:**
    * Flexbox의 속성(`flex-direction`, `justify-content`, `align-items`) 조합에 따라 주축과 교차축이 어떻게 변하는지 헷갈릴 때, AI에게 "Flexbox 레이아웃 코드를 CSS로 짜고, 브라우저 없이도 머릿속에 그려지도록 ASCII 아트나 텍스트 표로 시각화해 줘"라고 요청하여 직관적인 이해도 향상 유도
    * 레이아웃이 화면 크기에 따라 예기치 않게 찌그러지거나 넘칠 때, AI에 HTML/CSS 코드를 붙여넣고 "현재 Flex 구조에서 창 크기가 줄어들 때 줄바꿈(`flex-wrap`)이 부드럽게 일어나도록 `flex-basis`와 `flex-grow` 설정값을 교정해 줘"라고 프롬프팅 수행