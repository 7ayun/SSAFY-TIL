# [Web] HTML & CSS — 구조, 스타일링, 레이아웃

> **핵심 키워드:** #HTML #CSS #웹 #Markup #HyperText #Tag #Selector #Specificity #BoxModel #Margin #Padding #Border #Display #Block #Inline #InlineBlock #Position #Relative #Absolute #Fixed #Sticky #Flexbox #FlexContainer #FlexItem #MainAxis #CrossAxis #반응형

---

## 학습 목표

* 웹의 개념과 웹페이지의 구성 요소(HTML, CSS, JavaScript)를 이해한다
* HTML의 기본 구조와 주요 태그를 활용하여 문서를 구조화할 수 있다
* CSS 선택자의 종류와 명시도(Specificity)의 우선순위를 이해한다
* CSS Box Model의 구성 요소(margin, border, padding, content)를 파악하고 box-sizing을 적용할 수 있다
* Position 속성(static, relative, absolute, fixed, sticky)의 차이를 구분하고 적절히 활용할 수 있다
* Flexbox를 사용하여 요소를 주축·교차축 기준으로 배치하고 정렬할 수 있다

---

## 1. 웹 개요

### 1-1. 웹이란

웹(Web)은 World Wide Web의 줄임말로, 인터넷으로 연결된 컴퓨터들이 정보를 공유하는 거대한 정보 공간이다. 웹사이트는 여러 웹페이지가 모인 것이고, 웹페이지는 HTML이나 CSS 같은 기술로 만들어진 웹사이트의 가장 작은 구성 단위이다.

### 1-2. 웹페이지의 구성 요소

웹페이지는 집을 짓는 것에 비유할 수 있다. 철골 프레임(구조)은 HTML, 인테리어(스타일)는 CSS, 전기 배선(동작)은 JavaScript가 담당한다.

| 역할 | 언어 | 설명 |
|------|------|------|
| 구조 | HTML | 웹페이지의 의미와 구조를 정의 |
| 스타일 | CSS | 디자인과 레이아웃을 구성 |
| 동작 | JavaScript | 사용자와의 상호작용 (프론트엔드 파트에서 학습) |

### 1-3. 웹 학습의 방향성

알고리즘은 명확한 정답을 찾아가는 과정이지만, 웹은 다양한 기술을 조합하여 최적의 경험을 만들어가는 과정이다. 세세한 것을 외우기보다 하나하나 페이지를 만들어 나가는 즐거움이 중요하다.

> **강사님 팁**: 검색할 때 "How to"보다 "Why"로 검색하라. AI에게 물어볼 때도 바로 답을 요구하기보다 원리와 힌트를 요청하는 방식이 좋다. 웹 기술은 빠르게 변하므로 5년 이상 된 블로그 글은 현재 표준과 다를 수 있다.

---

## 2. HTML — 웹 구조화

### 2-1. HTML이란

HTML(HyperText Markup Language)은 웹페이지의 의미와 구조를 정의하는 언어이다. 프로그래밍 언어가 아니라 마크업 언어로, 태그를 이용해서 데이터의 구조를 잡는다. 마크다운(Markdown)도 마크업 언어의 하나이며, 마크다운의 `#` 문법은 HTML의 `<h1>` 태그에서 차용한 것이다.

HyperText는 다른 페이지를 참조하는 링크를 통해 비선형적으로 이동할 수 있는 텍스트를 의미한다.

### 2-2. HTML 문서의 기본 구조

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>My Page</title>
  </head>
  <body>
    <p>웹페이지입니다</p>
  </body>
</html>
```

`<!DOCTYPE html>`은 HTML 문서임을 선언하는 코드이다. `<html>` 태그가 전체 페이지를 포함하며, 그 안에 `<head>`(설정, 메타데이터)와 `<body>`(사용자에게 보이는 내용)가 나뉜다. 하나의 HTML 문서에는 하나의 body 태그만 존재해야 한다. VS Code에서 `!` + Tab/Enter를 입력하면 기본 템플릿이 자동 생성된다.

### 2-3. HTML 요소와 속성

하나의 요소(Element)는 여는 태그, 내용, 닫는 태그로 구성된다. 닫는 태그에는 슬래시(`/`)가 붙는다. 닫는 태그가 없는 태그는 내용이 없는 태그로, `<img>`, `<meta>`, `<br>` 등이 있다.

속성(Attribute)은 태그 이름 뒤에 공백으로 구분하여 작성하며, `이름="값"` 형태이다. 여러 속성이 있을 때는 콤마가 아닌 공백으로 구분한다.

### 2-4. 주요 태그

**텍스트 구조 태그:**

| 태그 | 설명 |
|------|------|
| `<h1>` ~ `<h6>` | 제목 (h1이 가장 크고, 문서당 h1은 하나만 권장) |
| `<p>` | 문단 (다음 요소가 자동 줄바꿈) |
| `<em>` | 기울임 (의미적 강조, 스크린리더가 강조하여 읽음) |
| `<strong>` | 굵게 (의미적 강조, `<b>`는 단순 시각적 굵게) |
| `<br>` | 줄바꿈 (닫는 태그 없음) |

**목록 태그:**

`<ol>`(ordered list)은 넘버링 목록, `<ul>`(unordered list)은 기호 목록이다. 내부에 `<li>` 태그로 항목을 작성한다.

**링크와 이미지:**

`<a href="URL">텍스트</a>`는 하이퍼링크이며, `href` 속성이 필수이다. `<img src="경로" alt="대체텍스트">`는 이미지를 삽입하며, 닫는 태그가 없다. `alt`는 이미지 로드 실패 시 또는 스크린리더가 읽을 대체 텍스트이다.

### 2-5. HTML에서의 의미(Semantic)

HTML 태그는 단순한 시각적 효과가 아니라 의미를 담는다. `<h1>`은 텍스트를 크게 만드는 것이 아니라 "최상위 제목"이라는 의미를 부여하는 것이다. 텍스트를 크게 만들고 싶으면 CSS로 font-size를 키워야 한다. 이 의미는 검색 엔진 최적화와 스크린리더 접근성에 중요하다.

### 2-6. HTML 스타일 가이드

태그는 소문자로 작성하고, 속성값은 큰따옴표를 사용한다. 들여쓰기는 2칸이 권장된다(4칸은 HTML에서 코드가 오른쪽으로 쏠림). HTML은 에러 출력이 없으므로 개발자 도구를 항상 켜두고 디버깅해야 한다.

---

## 3. CSS — 웹 스타일링

### 3-1. CSS란

CSS(Cascading Style Sheet)는 웹페이지의 디자인과 레이아웃을 구성하는 언어이다. "Cascading"은 위에서 아래로 단계적으로 떨어진다는 뜻으로, 스타일 적용의 우선순위와 관련된다.

### 3-2. CSS 적용 방법 3가지

| 방식 | 위치 | 우선순위 | 권장도 |
|------|------|----------|--------|
| 인라인(Inline) | HTML 태그의 `style` 속성 안 | 가장 높음 | 비권장 (유지보수 어려움) |
| 내부(Internal) | `<head>` 안의 `<style>` 태그 | 중간 | 수업·단일 파일에 적합 |
| 외부(External) | 별도 `.css` 파일을 `<link>`로 연결 | 가장 낮음 | 가장 권장 (공통 스타일 재사용) |

실무에서는 외부 방식을 기본으로 하고 내부 방식을 보조적으로 사용한다. 인라인 방식은 코드의 가독성과 유지보수성을 크게 떨어뜨리므로 지양한다.

### 3-3. CSS 문법

```css
h1 {
  color: blue;
  font-size: 30px;
}
```

`h1`이 **선택자**(누구를 꾸밀지), 중괄호 안이 **선언**(무엇을 할지)이다. 선언은 속성(property)과 값(value) 쌍으로 이루어지며, 반드시 세미콜론으로 끝난다.

### 3-4. 선택자(Selector)

| 선택자 | 표기법 | 설명 |
|--------|--------|------|
| 전체 선택자 | `*` | 문서의 모든 태그 선택 |
| 요소 선택자 | `h1`, `p` | 특정 태그 선택 (다중 선택: `h3, h4`) |
| 클래스 선택자 | `.green` | 해당 클래스가 부여된 모든 요소 선택 |
| ID 선택자 | `#purple` | 해당 ID가 부여된 요소 선택 (문서당 하나 권장) |

**결합자(Combinator):**

자손 결합자(`공백`)는 한 단계 이하의 모든 하위 요소를 선택하고, 자식 결합자(`>`)는 한 단계 바로 아래 자식만 선택한다.

```css
.green li { color: brown; }   /* 자손 결합자 — green 클래스 안 모든 li */
.green > span { font-size: 50px; }  /* 자식 결합자 — green 바로 아래 span만 */
```

> **강사님 강조**: 실무에서는 99% 클래스 선택자만 쓴다. 여러 선택자를 섞으면 가중치 계산이 복잡해져 생산성이 떨어지기 때문이다. 클래스 하나만 쓰면 가중치가 동일하므로 서로 다른 스타일만 잘 만들어두면 된다.

### 3-5. 명시도(Specificity)

하나의 요소에 여러 스타일이 충돌할 때, 가중치가 높은 쪽이 적용된다.

**우선순위 (높은 순):**

1. `!important` — 무조건 최우선 (계단식 구조를 무시하므로 사용 자제)
2. 인라인 스타일 (`style` 속성)
3. ID 선택자 (`#`)
4. 클래스 선택자 (`.`)
5. 요소 선택자 (`h1`, `p`)
6. 전체 선택자 (`*`)
7. 소스코드 선언 순서 (동일 가중치일 때 마지막에 선언된 것이 적용)

핵심 원칙: **좁게(집약적으로) 선택할수록 강하고, 넓게 선택할수록 약하다.**

> **주의**: 클래스 여러 개를 태그에 작성하는 순서(`class="green orange"`)는 우선순위에 영향이 없다. 중요한 것은 CSS 코드에서의 선언 순서이다.

### 3-6. CSS 값과 단위

**절대 단위**: `px`(픽셀)이 대표적이며, 고정 크기로 직관적이지만 다양한 디바이스 대응에 한계가 있다.

**상대 단위:**

| 단위 | 기준 | 특징 |
|------|------|------|
| `em` | 부모 요소의 font-size | 중첩 시 연쇄적으로 변할 수 있음 |
| `rem` | HTML 최상위 요소(기본 16px) | 중첩과 무관하게 예측 가능 |
| `%` | 부모 요소의 해당 속성 | 이미지·너비 등에 활용 |

실제 개발에서는 `px`보다 `em`, `rem`을 훨씬 많이 사용한다. 값이 0일 때는 단위를 생략하는 것이 권장된다.

### 3-7. CSS 상속

텍스트 관련 속성(color, font-size, text-align 등)은 부모에서 자식으로 상속된다. 반면 box model 관련 속성(width, height, border, margin, padding)과 position은 상속되지 않는다. 개발자 도구에서 "Inherited from ..."으로 상속 여부를 확인할 수 있다.

### 3-8. MDN 문서

CSS 속성을 검색할 때는 MDN(Mozilla Developer Network) 키워드를 함께 붙이는 것이 좋다. 예를 들어 `background-color MDN`으로 검색하면 가장 정확하고 최신화된 공식 기술 문서를 볼 수 있다.

---

## 4. CSS Box Model

### 4-1. Box Model이란

CSS는 모든 HTML 요소를 사각형 상자(Box)로 본다. 원처럼 보이는 것도 실제로는 네모 박스를 깎은 것이다. Box Model은 요소의 크기, 배치, 간격을 결정하는 규칙이다.

### 4-2. Box Model 구성 요소

바깥쪽부터 안쪽 순서로 네 가지 영역이 있다.

| 영역 | 설명 | 방향별 속성 |
|------|------|-------------|
| Margin | 박스와 다른 박스 간의 외부 여백 | margin-top/right/bottom/left |
| Border | 테두리 | border-width, border-style, border-color |
| Padding | 콘텐츠와 테두리 사이의 내부 여백 | padding-top/right/bottom/left |
| Content | 실제 내용물 | width, height |

> **주의**: `margin-left: 25px`은 "왼쪽으로 이동"이 아니라 "왼쪽 마진에 25px을 부여"하는 것이므로, 실제 이동은 오른쪽이다. `margin-top: 50px`도 마찬가지로 아래쪽으로 이동한다.

### 4-3. 단축 속성(Shorthand)

`border`는 너비, 스타일, 색상을 한 번에 지정할 수 있다(순서 무관).

```css
border: 2px solid black;
```

`margin`, `padding`은 값 개수에 따라 방향이 결정된다.

| 값 개수 | 적용 방향 |
|---------|-----------|
| 4개 | 상 → 우 → 하 → 좌 (시계 방향) |
| 3개 | 상 → 좌우 → 하 |
| 2개 | 상하 → 좌우 |
| 1개 | 네 방향 모두 |

### 4-4. box-sizing

기본값인 `content-box`는 width/height가 콘텐츠 영역만을 기준으로 한다. 따라서 width를 100px로 지정해도 padding과 border가 추가되면 실제 박스 너비는 더 커진다.

`border-box`로 설정하면 width/height가 테두리를 포함한 전체 박스 기준이 되어 직관적이다.

```css
* {
  box-sizing: border-box;  /* 대부분의 개발에서 이렇게 선언하고 시작 */
}
```

예시: width 100px, padding 10px, border 2px일 때 `content-box`에서는 실제 너비가 124px이 되지만, `border-box`에서는 내부 콘텐츠가 76px로 줄어들어 전체 너비가 정확히 100px이 된다.

---

## 5. Display 속성과 Normal Flow

### 5-1. Normal Flow

레이아웃을 강제로 변경하지 않았을 때 요소가 자연스럽게 배치되는 방식이다. Block 요소는 위에서 아래로, Inline 요소는 왼쪽에서 오른쪽으로 흐른다.

### 5-2. Block 타입

한 줄 전체를 차지하며 자동으로 줄바꿈이 일어난다. width를 지정하지 않으면 100% 너비를 가진다. width, height, margin, padding 모두 자유롭게 조정 가능하다.

대표 태그: `<div>`, `<h1>`~`<h6>`, `<p>`, `<ul>`, `<li>`

`<div>`는 의미 없이 섹션을 구조화하는 블록 태그로, 실제 웹사이트에서 가장 많이 보이는 태그이다.

### 5-3. Inline 타입

줄바꿈 없이 콘텐츠 크기만큼만 영역을 차지한다. width, height를 지정할 수 없고, 수직 방향(위아래)의 padding/margin/border는 다른 요소를 밀어내지 못한다. 좌우로는 밀어낼 수 있다.

대표 태그: `<a>`, `<img>`, `<span>`, `<strong>`, `<em>`

`<span>`은 의미 없이 텍스트 일부를 선택하는 인라인 태그이다.

### 5-4. Inline-Block 타입

Inline의 특징(줄바꿈 없음)과 Block의 특징(width/height/상하 margin 등 자유 조정)을 결합한 타입이다. 내비게이션 바처럼 리스트를 수평 배치하거나, 여러 박스를 가운데 정렬할 때 활용한다.

### 5-5. None 타입

`display: none`은 요소를 화면에서 완전히 제거한다. 공간도 차지하지 않으며, 자바스크립트와 함께 사용자 상호작용(버튼 클릭 시 요소 숨기기/보여주기)에 활용된다.

---

## 6. CSS Position

### 6-1. Position이란

Normal Flow에서 요소를 뜯어내어 다른 위치로 배치하는 속성이다. 이동 방향은 top, right, bottom, left 4가지이며, Z축(z-index)으로 쌓임 순서도 제어할 수 있다.

### 6-2. Position 유형

| 유형 | Normal Flow | 기준점 | 원래 영역 유지 |
|------|-------------|--------|----------------|
| `static` | 유지 | — (기본값, 방향 속성 불가) | O |
| `relative` | 유지 | 본인의 static 위치 | O (차지한 상태로 이동) |
| `absolute` | 제거 | 가장 가까운 relative 부모 (없으면 body) | X (영역 사라짐) |
| `fixed` | 제거 | 뷰포트(화면) | X (스크롤해도 고정) |
| `sticky` | 조건부 | 임계점 전 relative, 도달 후 fixed | 상황에 따라 |

### 6-3. Relative

본인의 원래 위치(static일 때)를 기준으로 top, left 등으로 이동한다. 이동해도 원래 자리는 그대로 차지하고 있으므로 다른 요소가 빈 공간을 채우지 않는다.

### 6-4. Absolute

Normal Flow에서 완전히 빠져나와 가장 가까운 `position: relative`인 부모를 기준으로 배치된다. relative 부모가 없으면 body를 기준으로 한다. 원래 영역이 사라지므로 아래 요소들이 위로 올라온다.

활용 예시: 이미지 위에 배지를 올려놓기(`top: 0; left: 0;`으로 좌측 상단 고정)

> **주의**: absolute를 쓰기 전에 "누구를 기준으로 움직일지" 부모에 relative를 설정해야 한다.

### 6-5. Fixed

뷰포트(브라우저 화면)를 기준으로 배치된다. 스크롤해도 항상 같은 위치에 고정된다. 챗봇 아이콘, 웹툰의 상하 이동 버튼 등에 활용된다.

### 6-6. Sticky

평소에는 relative처럼 동작하다가, 스크롤이 임계점(예: `top: 0`)에 도달하면 fixed처럼 고정된다. 내비게이션 바가 스크롤 시 상단에 고정되는 사이트에서 사용된다. 다음 sticky 요소가 나타나면 이전 것을 대체한다.

### 6-7. z-index

요소의 쌓임 순서를 정수값으로 정의한다. 값이 클수록 앞(사용자 쪽)에 배치된다. static이 아닌 요소에만 적용되며, 부모의 z-index가 낮으면 자식이 아무리 높은 값을 줘도 부모보다 위로 올라갈 수 없다. 값이 같으면 HTML 작성 순서대로 쌓인다.

보통 fixed 요소에 `z-index: 9999` 같은 큰 값을 지정하여 다른 요소보다 항상 위에 표시되도록 한다.

---

## 7. CSS Flexbox

### 7-1. Flexbox란

요소를 행(row)과 열(column) 형태로 배치하는 1차원 레이아웃 방식이다. display와 position만으로는 큰 그림의 공간 배치가 어려웠던 문제를 해결한다. 부모 컨테이너에 `display: flex`를 선언하면 1차 자식 요소들이 flex item이 된다.

### 7-2. 핵심 구성 요소

| 구성 요소 | 설명 |
|-----------|------|
| Flex Container | `display: flex`가 선언된 부모 요소 (배치의 주체) |
| Flex Item | 컨테이너의 1차 자식 요소들 (자손은 해당 안 됨) |
| 주축(Main Axis) | 아이템이 배치되는 기본 축 (기본값: 가로, 왼→오) |
| 교차축(Cross Axis) | 주축의 수직 방향 (기본값: 세로, 위→아래) |

주축만 기억하면 교차축은 자동으로 결정된다.

### 7-3. Flex Container 속성 — 방향과 줄바꿈

**flex-direction:** 아이템이 나열되는 방향을 결정한다.

| 값 | 방향 |
|----|------|
| `row` (기본값) | 왼쪽 → 오른쪽 |
| `row-reverse` | 오른쪽 → 왼쪽 |
| `column` | 위 → 아래 |
| `column-reverse` | 아래 → 위 |

**flex-wrap:** 아이템이 한 행에 들어가지 못할 때의 동작을 결정한다.

| 값 | 동작 |
|----|------|
| `nowrap` (기본값) | 줄바꿈 없이 축소됨 |
| `wrap` | 넘치는 아이템을 다음 행으로 줄바꿈 |
| `wrap-reverse` | 줄바꿈 방향을 반대로 |

### 7-4. 주축 정렬 — justify-content

주축 방향으로 아이템 전체를 정렬한다. 부모 컨테이너에 설정한다.

| 값 | 동작 |
|----|------|
| `flex-start` (기본값) | 시작점 정렬 |
| `center` | 가운데 정렬 |
| `flex-end` | 끝점 정렬 |
| `space-between` | 양 끝 배치, 나머지 균등 분배 |
| `space-around` | 각 아이템 양쪽 동일 여백 |
| `space-evenly` | 모든 간격 동일 |

### 7-5. 교차축 정렬

**align-content**: 여러 줄(flex-wrap 필요)의 공간 배분. 한 줄일 때는 의미 없다.

**align-items**: 교차축 방향으로 아이템 전체를 정렬한다. 기본값은 `stretch`(교차축 방향으로 늘려서 채움)이며, `flex-start`, `center`, `flex-end` 등을 사용할 수 있다.

**align-self**: 개별 아이템 하나만 교차축으로 이동한다. 부모가 아닌 해당 아이템에 직접 설정한다.

**수직·수평 중앙 정렬 공식:**

```css
.container {
  display: flex;
  justify-content: center;   /* 주축 가운데 */
  align-items: center;        /* 교차축 가운데 */
}
```

> **왜 justify-items나 justify-self는 없는가?**: 주축(좌우) 방향은 margin auto로 개별 조정이 가능하기 때문에 CSS에서 별도로 만들지 않았다. 반면 교차축(위아래)은 margin auto로 정렬이 되지 않으므로 align 속성이 세 가지(content, items, self)나 존재한다.

### 7-6. Flex Item 속성

**flex-grow**: 남는 여백을 비율에 따라 분배한다. 부모가 아닌 개별 아이템에 설정한다.

```css
.item1 { flex-grow: 1; }
.item2 { flex-grow: 2; }
.item3 { flex-grow: 3; }
/* 남는 영역을 6등분하여 1:2:3 개수로 배분 */
```

주의할 점은 flex-grow 값이 아이템 너비의 배수가 아니라는 것이다. 원래 콘텐츠 너비에 남는 공간을 등분하여 추가하는 방식이므로, grow 값이 3이라고 해서 1의 3배가 되지는 않는다.

**flex-basis**: flex item의 초기 크기를 설정한다. width와 함께 적용하면 flex-basis가 우선 적용된다.

### 7-7. Flexbox 속성 분류 정리

| 목적 | 속성 | 설명 |
|------|------|------|
| 큰 그림 배치 | flex-direction | 흐르는 방향 |
| 큰 그림 배치 | flex-wrap | 줄바꿈 여부 |
| 주축 공간 분배 | justify-content | 주축 정렬 (전체) |
| 교차축 공간 분배 | align-content | 여러 줄의 교차축 공간 배분 |
| 교차축 정렬 | align-items | 교차축 정렬 (전체) |
| 교차축 정렬 | align-self | 교차축 정렬 (개별, 아이템에 직접 설정) |
| 여백 분배 | flex-grow | 남는 여백 비율 배분 (아이템에 설정) |
| 초기 크기 | flex-basis | 아이템의 기본 너비 (아이템에 설정) |

키워드 기억법: `justify-*`는 주축, `align-*`은 교차축이다.

### 7-8. Flexbox 활용 — 반응형 카드 레이아웃

flex-wrap + flex-basis + flex-grow를 조합하면 화면 크기에 따라 카드 배치가 변하는 반응형 레이아웃을 구현할 수 있다.

```css
.card { display: flex; flex-wrap: wrap; }
.thumbnail { flex-basis: 700px; flex-grow: 1; }
.content { flex-basis: 350px; flex-grow: 1; }
```

화면이 충분히 넓으면 이미지와 텍스트가 좌우로 배치되고, 화면을 줄이면 텍스트가 아래로 떨어진다. 보다 정교한 반응형 웹은 이후 Responsive Web에서 학습한다.

---

## 8. 참고

### 8-1. 마진 상쇄(Margin Collapse)

두 블록 요소의 위아래 마진이 만나면 더 큰 값으로 합쳐진다(더해지지 않는다). 예를 들어 위 요소가 margin-bottom 20px, 아래 요소가 margin-top 20px이면 둘 사이의 거리는 40px이 아니라 20px이다.

CSS가 의도한 설계이며, 한쪽에서만 마진을 관리하도록 일관성을 유지하기 위함이다. 좌우 마진은 상쇄되지 않는다.

### 8-2. 박스 타입별 가운데 정렬

| 박스 타입 | 수평 가운데 정렬 방법 |
|-----------|----------------------|
| Block | `margin: 0 auto;` (좌우 margin을 auto) |
| Inline / Inline-Block | 부모에 `text-align: center;` |
| Flex Item | 부모에 `justify-content: center;` |

수직 가운데 정렬은 margin으로는 불가능하며, Flexbox의 `align-items: center`를 사용한다.

### 8-3. CSS 스타일 가이드

CSS도 들여쓰기 2칸, 속성과 선택자는 각각 새로운 줄, 중괄호 앞 공백, 콜론 뒤 공백, 마지막 속성에도 세미콜론을 넣는다.

---

## 정리

| 구분 | 핵심 포인트 |
|------|-------------|
| HTML | 웹페이지의 구조와 의미를 정의하는 마크업 언어 |
| CSS | 웹페이지의 디자인·레이아웃을 담당하는 스타일 언어 |
| 선택자 명시도 | !important > inline > #id > .class > 요소 > * |
| Box Model | margin → border → padding → content (4개 영역) |
| box-sizing | `border-box`로 설정하면 width가 테두리 포함 전체 기준 |
| Block vs Inline | Block은 한 줄 전체 차지, Inline은 콘텐츠만큼만 |
| Position | static(기본) / relative(자기 기준) / absolute(부모 기준) / fixed(뷰포트) / sticky(조건부) |
| Flexbox | 부모에 `display: flex` 선언, justify는 주축, align은 교차축 |
| flex-grow | 남는 여백을 총합으로 나눠 개수만큼 분배 (배수가 아님) |
| 반응형 기초 | flex-wrap + flex-basis + flex-grow 조합 |
