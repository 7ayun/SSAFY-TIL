# [Web] CSS 배치 기술 및 Flexbox

> **핵심 키워드:** #CSS_Layout #Display #Position #Z_Index #Flexbox #Margin_Collapse

---

## 🎯 학습 목표
* CSS Display(`block`, `inline`, `inline-block`, `none`)의 렌더링 특성 완벽 이해
* Position 및 Z-index를 활용한 노멀 플로우(Normal Flow) 해제 및 3D 겹침 레이아웃 제어
* Flexbox의 1차원 레이아웃 설계 원리 및 수직/수평 중앙 정렬 마스터
* 마진 상쇄(Margin Collapse) 현상의 원리 파악 및 요소별 가운데 정렬 기법 적용

---

## 💡 주요 개념 정리

### 1. 웹 페이지 레이아웃 설계의 비유 (인테리어)
* **배치의 핵심:** 방의 인테리어를 하듯 가구와 소품의 크기 및 여백을 정하고, 그룹으로 묶어 정렬하며, 특정 요소를 특별한 위치에 고정하는 일련의 과정
* **노멀 플로우 (Normal Flow):** CSS로 레이아웃을 인위적으로 변경하지 않았을 때 요소가 자연스럽게 배치되는 기본 흐름 (좌에서 우로, 위에서 아래로)

### 2. Display 속성 (박스의 렌더링 타입)
* **Block (`display: block`):** 책의 문단과 같은 역할
  * 항상 새로운 줄에서 시작하며 가로 너비(Width)를 기본적으로 100% 차지함
  * 너비, 높이, 마진, 패딩 모두 자유롭게 지정 가능하며 주변 요소를 밀어냄
  * 대표 태그: `div`, `h1~h6`, `p`, `ul`, `li` 등 (네이버 메인 레이아웃의 뼈대도 모두 `div` 블록으로 구성됨)
* **Inline (`display: inline`):** 문장 속 단어에 형광펜을 칠하는 것과 같은 역할
  * 줄바꿈이 일어나지 않고 텍스트 흐름에 자연스럽게 배치됨
  * 콘텐츠의 딱 크기만큼만 영역을 차지하므로 `width`, `height` 지정 불가
  * 상하(수직) 패딩/마진은 시각적으로 적용되나 다른 요소를 밀어내지 못함 (좌우 밀어내기만 가능)
  * 대표 태그: `span`, `a`, `img`, `strong` 등
* **Inline-Block (`display: inline-block`):** 블록과 인라인의 혼합형
  * 줄바꿈 없이 나란히 배치되는 인라인의 특성과, 크기(`width`/`height`) 및 여백 밀어내기가 가능한 블록의 특성을 모두 가짐
  * 수직 형태의 `li` 태그들을 수평 네비게이션 바 형태로 정렬할 때 유용함
* **None (`display: none`):** 축구팀의 후보 선수와 같은 역할
  * HTML 코드 상에는 존재하지만 화면에 렌더링되지 않으며 차지하는 공간(영역)조차 완전히 제거됨
  * 향후 JavaScript를 활용한 사용자와의 상호작용(클릭 시 숨김/표시 등)에 핵심적으로 사용됨

### 3. Position 속성 (노멀 플로우 해제 및 배치)
* **Static (기본값):** 일반적인 노멀 플로우 상태. `top`, `left`, `right`, `bottom` 속성 적용 불가
* **Relative (상대 위치):** 자신의 원래 `static` 위치를 기준으로 이동
  * 이동하더라도 원래 본인이 차지하던 Static 영역의 공간은 그대로 유지됨 (다른 요소가 치고 올라오지 못하는 욕심쟁이 특성)
  * `top: 100px`은 "위로 100px 이동"이 아니라 "위쪽 영역에 100px을 주어 아래로 이동시킴"을 의미
* **Absolute (절대 위치):** 노멀 플로우에서 완전히 뜯겨져 나옴
  * 본인이 차지하던 기존 공간이 완전히 사라지므로 아래쪽 레이아웃이 위로 치고 올라와 깨질 수 있음 주의
  * 가장 가까운 `position: relative` (혹은 static이 아닌) 부모를 도화지(기준점)로 삼아 이동함 (없으면 `body` 기준)
  * 쇼핑몰 상품 이미지 좌측 상단의 'BEST' 배지 등을 겹쳐서 띄울 때 주로 사용됨
* **Fixed (고정 위치):** 사용자가 보는 화면(Viewport)을 기준으로 고정됨
  * 스크롤을 내려도 항상 지정된 화면 위치에 떠 있음
  * 챗봇 아이콘, 우측 하단의 최상단 이동 플로팅 버튼, 웹툰 스크롤 바 등에 사용됨
* **Sticky (임계점 고정):** `relative`와 `fixed`의 결합형
  * 평소에는 `relative`처럼 스크롤되다가 설정한 임계점(`top: 0` 등)에 도달하는 순간 화면에 `fixed` 됨
  * 스크롤 시 상단에 달라붙는 네비게이션 바 구현에 사용되며, 다음 Sticky 요소가 나타나면 자연스럽게 교대됨

### 4. Z-Index (Z축 쌓임 순서)
* **비유:** 연극 무대에서 배경(1)은 뒤에, 배우(10)는 앞에 서야 하는 것과 같음
* 화면을 바라보는 사용자 방향(Z축)의 겹침 순서를 정수값으로 제어 (값이 클수록 앞에 배치됨)
* `static`이 아닌 요소(relative, absolute, fixed 등)에만 적용 가능함
* 항상 부모의 Z-index 한계를 벗어날 수 없으며, 값이 같으면 HTML 코드 작성 순서(나중에 작성된 것)에 따라 위로 쌓임
* 실무에서는 최상단 고정이 필요한 `fixed` 요소에 `9999`와 같은 압도적인 큰 값을 주어 겹침 문제를 방지함

### 5. Flexbox (1차원 내부 레이아웃 설계의 끝판왕)
* **도입 배경:** 기존 `display`와 `position`만으로는 요소들을 일정한 비율로 쪼개거나 수직/수평 정렬하는 거시적 배치가 매우 고통스러움
* **비유:** 책장에 책을 배치하듯 1차원(행 또는 열) 방향으로 아이템을 나열하고 정렬함
* **구성 요소:** 배치 공간을 제공하는 부모 `Flex Container`와 그 안에서 정렬되는 1차 자식 요소 `Flex Item`
* **Flex Container 속성 (부모에 적용):**
  * `flex-direction`: 주축(Main Axis) 방향 결정. `row`(가로, 기본값), `column`(세로), `row-reverse`, `column-reverse`
  * `flex-wrap`: 공간 부족 시 아이템의 줄바꿈 여부 결정. `nowrap`(기본값, 억지로 우겨넣음), `wrap`(반응형으로 넘치면 아래로 떨어뜨림)
  * `justify-content`: 주축(Main Axis) 공간 배분 및 정렬. `flex-start`, `center`(가운데 정렬 핵심), `flex-end`, `space-between` 등
  * `align-content`: 교차축(Cross Axis) 기준 '여러 줄(Wrap 상태)'일 때의 줄 간 공간 배분
  * `align-items`: 교차축(Cross Axis) 기준 '한 줄'일 때의 아이템 수직 정렬. `stretch`(기본값, 꽉 채움), `center`(수직 중앙 정렬 핵심)
* **Flex Item 속성 (자식에 직접 적용):**
  * `align-self`: 부모의 `align-items`를 무시하고 아이템 1개만 개별적으로 교차축 정렬
  * `flex-grow`: 남는 여백을 아이템들이 나눠 갖는 비율. (주의: 너비 배수가 아님)
  * `flex-basis`: 아이템의 초기 최소 너비. `width` 속성과 동시 적용 시 `flex-basis`가 우선순위를 가짐

### 6. 마진 상쇄 (Margin Collapse)와 정렬 팁
* **마진 상쇄 현상:** 위아래로 배치된 블록 요소 간에 `margin-bottom: 10px`과 `margin-top: 30px`이 만나면 40px이 아닌 더 큰 값인 30px로 상쇄(병합)됨
* **CSS의 의도된 설계:** 복잡한 레이아웃에서 위아래 여백을 여러 요소가 중구난방으로 밀어내면 일관성이 깨지므로, 수직 배치 시 "한쪽 방향에서만 밀어내라"는 규칙을 유도하기 위함 (좌우 마진은 상쇄되지 않음)
* **상황별 가운데 정렬 총정리:**
  * Block 박스: `margin: 0 auto` 적용 (수평만 가능, 수직 불가)
  * Inline 박스: 부모 요소에 `text-align: center` 적용
  * Flexbox: 부모 요소에 `justify-content: center` & `align-items: center` 적용 (수직/수평 완벽 중앙 정렬)

---

## 💻 기능 구현 및 코드 실습

### 1. Display 속성 비교 및 전환 (Inline-block 활용)
```css
/* 기본 상태: span은 inline이므로 width 적용 불가, 상하 여백으로 주변 요소를 밀어내지 못함 */
.inline-span {
  display: inline;
  width: 300px; /* 적용 안 됨 */
  margin-top: 50px; /* 시각적으론 보이나 위쪽 요소를 밀어내지 못하고 겹침 */
}

/* 해결책: inline-block으로 전환하여 줄바꿈 없이 크기와 여백 제어 권한 획득 */
.nav-item {
  display: inline-block;
  width: 100px; /* 정상 적용됨 */
  margin-top: 20px; /* 정상적으로 다른 요소를 밀어냄 */
  text-align: center; /* 텍스트 가운데 정렬 */
}

/* 동적 제어: 요소를 레이아웃에서 완전히 제거 (공간 반납) */
.hidden-box {
  display: none;
}
```

### 2. Position 5대 속성과 Z-index 제어
```css
/* 도화지(기준점) 역할 세팅 */
.container {
  position: relative;
  width: 500px;
  height: 500px;
}

/* 1. Relative: 자신의 원래 자리 기준 이동 (원래 빈자리 유지됨) */
.box-relative {
  position: relative;
  top: 100px; /* 윗통수에 100px 여백을 주어 아래로 이동 */
  left: 100px; /* 왼쪽에서 100px 밀어냄 */
}

/* 2. Absolute: 부모(.container)를 기준으로 절대 배치 (기존 자리 소멸) */
.box-absolute {
  position: absolute;
  top: 0;
  left: 0; /* 좌측 상단 뱃지처럼 딱 붙음 */
}

/* 3. Fixed: 화면(Viewport) 기준 고정 (스크롤 무시) */
.chatbot-btn {
  position: fixed;
  bottom: 20px;
  right: 20px; /* 우측 하단에 영구 고정 */
  z-index: 9999; /* 무조건 최상단에 보이도록 압도적인 값 부여 */
}

/* 4. Sticky: 스크롤하다가 최상단에 닿으면 고정 */
.nav-bar {
  position: sticky;
  top: 0; /* 임계점 설정 */
}
```

### 3. Flexbox - 부모 컨테이너(Container) 정렬 속성
```css
.flex-container {
  display: flex; /* 1. 자식들을 Flex Item으로 변환 */
  flex-direction: row; /* 2. 주축 방향 설정 (왼쪽->오른쪽 흐름, 기본값) */
  flex-wrap: wrap; /* 3. 자식 너비가 부모를 넘치면 아래로 줄바꿈 허용 */
  
  /* 4. 주축(가로) 공간 배분: 수평 정렬 */
  justify-content: center; /* 중앙 정렬 (이 외에 space-between 등 활용 가능) */
  
  /* 5. 교차축(세로) 단일 줄 정렬: 수직 정렬 */
  align-items: center; /* 수평/수직 완벽한 정가운데 배치 완성 */
  
  /* 참고: 교차축 여러 줄(Wrap 상태) 공간 배분 */
  align-content: flex-start; 
}
```

### 4. Flexbox - 자식 아이템(Item) 반응형 속성
```css
/* 반응형 카드 레이아웃 구현 예시 */
.card-item-image {
  /* width 대신 Flex 초기 최소 너비 부여 (width보다 우선순위 높음) */
  flex-basis: 700px; 
}

.card-item-text {
  flex-basis: 350px; /* 최소 너비 350px 보장 */
  
  /* 남는 여백 100% 흡수. 형제 요소가 flex-grow: 1이면 1:1 비율로 쪼개어 추가 너비 획득 */
  flex-grow: 1; 
  
  /* 개별 수직 정렬 제어 (부모의 align-items 속성 무시) */
  align-self: flex-end; /* 이 텍스트 박스만 바닥으로 내림 */
}
```

---

## 🚀 복습 및 AI 인사이트
* **헷갈렸던 점 1 (Flex-grow 연산의 함정):** 여백이 200px 남았을 때 아이템에 `flex-grow`를 `1, 1, 0, 2`로 주면 각각의 너비가 2배수가 되는 것이 절대 아님. 남은 여백 200px을 총합인 '4'로 4등분(50px)한 뒤, 각각 50px, 50px, 0px, 100px씩 기존 너비에 '추가'해 주는 개념임
* **헷갈렸던 점 2 (Justify-items가 없는 이유):** 교차축 정렬(align)과 달리 주축 정렬(justify)에는 `justify-self`나 `justify-items` 속성이 없음. 수평(가로)의 개별 정렬은 기존의 `margin-left: auto` 등으로 완벽하게 제어할 수 있어 CSS 설계 단계에서 굳이 만들지 않았기 때문임
* **AI 활용 팁:** 복잡한 Flexbox 래핑(Wrap)이나 Margin Collapse(마진 상쇄)로 레이아웃이 붕괴될 때, AI에게 "현재 박스 모델의 Margin 상쇄 현상 여부와 Flex Container의 축(Axis) 설정 오류를 MDN 기반으로 진단해 줘"라고 요청하면 구조적 원인을 빠르게 파악 가능함