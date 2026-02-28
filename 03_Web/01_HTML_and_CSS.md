# [Web] HTML & CSS (웹 구조와 스타일링 기초)

> **핵심 키워드:** #Markup #Semantic #Selector #Specificity #REM #Box_Model #MDN

---

## 🎯 학습 목표
* 명확한 정답을 찾는 알고리즘과 달리, 다양한 기술을 조합하여 최적의 경험을 만드는 웹 개발의 특성과 공식 문서(MDN) 중심의 학습법(Why 기반 검색) 숙지
* 하이퍼텍스트(Hypertext)와 마크업(Markup)의 개념을 이해하고, HTML 문서의 기본 뼈대(`head`, `body`) 구성
* 단순한 시각적 효과를 넘어, 스크린 리더와 검색 엔진(SEO)을 위한 시맨틱(Semantic) 태그의 중요성 인지
* CSS 스타일 적용의 3가지 방식(Inline, Internal, External)을 비교하고, 유지보수를 위한 클래스(Class) 기반 외부 스타일 시트 작성법 체화
* 선택자(Selector)의 명시도(Specificity) 가중치 계산 규칙과 상속(Inheritance)의 원리 파악
* 절대 단위(`px`)와 상대 단위(`em`, `rem`)의 차이점을 분석하고, 박스 모델(Box Model)의 `border-box` 기준 레이아웃 설계 기법 숙달

---

## 💡 주요 개념 정리

### 1. 웹(Web)의 3요소와 HTML 기초
* **웹 개발의 비유:** 집을 지을 때 **HTML은 철골 프레임(구조/의미)**, **CSS는 인테리어(스타일링)**, **JS는 전기 배선(동작)** 역할을 담당함
* **HTML (HyperText Markup Language):**
    * **HyperText:** 링크(`<a>`)를 통해 다른 페이지로 즉시 접근할 수 있는 비선형적 텍스트
    * **Markup:** 태그를 이용하여 데이터의 구조를 잡는 언어. (연산이나 변수 기능이 없으므로 프로그래밍 언어가 아님)
* **기본 구조:**
    * `<!DOCTYPE html>`: HTML5 문서임을 선언
    * `<head>`: 사용자에게 보이지 않는 문서의 설정(메타 데이터, 타이틀, 인코딩 `utf-8`) 담당
    * `<body>`: 실제 웹 페이지에 표시되는 모든 내용(몸통)이 작성되는 공간

### 2. 시맨틱(Semantic) 마크업과 속성(Attribute)
* **요소(Element)의 구성:** 여는 태그, 내용(Content), 닫는 태그로 구성됨. `<img>`, `<meta>`처럼 내용이 없는 태그는 닫는 태그를 생략함(Empty Tag)
* **속성 (Attribute):** 태그의 동작을 조정하거나 추가 정보를 제공. (예: `<a href="경로">`, `<img src="경로" alt="대체텍스트">`) 공백으로 구분하며 값은 큰따옴표로 묶음
* **의미론적 태그 (Semantic Tag):**
    * 단순히 글자를 굵게 만드는 텍스트 구조 이상의 의미를 지님
    * 시각적으로는 동일한 굵은 글씨라도, `<b>`는 단순 스타일링이고 `<strong>`은 '중요함'이라는 의미를 내포하여 스크린 리더기가 강조해서 읽어줌

### 3. CSS (Cascading Style Sheets) 적용 방식
* **인라인 (Inline):** 태그 내부에 `style` 속성으로 직접 작성. 명시도가 매우 높아 구조와 스타일이 섞이므로 유지보수성이 최악임 (사용 지양)
* **내부 (Internal):** `<head>` 내부에 `<style>` 태그를 만들어 작성
* **외부 (External):** 별도의 `.css` 파일을 만들고 `<link href="style.css">`로 연결. 재사용성이 가장 높아 실무에서 1원칙으로 권장됨

### 4. CSS 선택자(Selector)와 명시도(Specificity)
* **선택자 종류:** 전체(`*`), 요소(`h1`), 클래스(`.class`), 아이디(`#id`)
    * `id`는 문서 내 유일한 요소에, `class`는 재사용 가능한 여러 요소에 사용함
* **결합자 (Combinator):** 자손(공백, 하위 모든 요소) / 자식(`>`, 직계 1단계 하위 요소)
* **명시도 (가중치 싸움):** 좁고 집약적으로 선택할수록 우선순위가 높음
    * **우선순위:** `!important` > Inline > ID > Class > 요소(Type) > 전체(*)
    * **캐스케이딩 (Cascading):** 가중치가 완벽히 동일할 경우, 코드상 가장 마지막(아래)에 작성된 선언이 덮어쓰기 형태로 적용됨

### 5. CSS 단위(Units)와 박스 모델(Box Model)
* **상대 단위 비교 (`em` vs `rem`):**
    * `em`: **부모 요소**의 폰트 사이즈를 기준으로 배수 계산. 중첩될 경우 사이즈 예측이 꼬일 위험이 있음
    * `rem`: **최상위 루트(`html`, 기본 16px)** 폰트 사이즈를 기준으로 배수 계산. 구조가 중첩되어도 일관된 크기를 유지하여 실무에서 권장됨
* **박스 모델 (Box Model):** 모든 HTML 요소는 사각형 박스로 취급됨
    * `Margin` (테두리 외부 여백) / `Border` (테두리) / `Padding` (테두리 내부 여백) / `Content` (실제 내용)
* **`box-sizing: border-box`:** CSS의 기본 너비(`width`)는 Content 기준이므로, Padding과 Border를 추가하면 박스 전체 크기가 의도치 않게 커짐. 이를 방지하기 위해 테두리를 기준으로 크기를 고정하는 `border-box` 선언이 필수적임

---

## 💻 기능 구현 및 코드 실습

### [코드] HTML 문서 기본 뼈대 및 시맨틱 태그
```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>문서 제목 (브라우저 탭에 표시)</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <h1>가장 큰 대제목 (문서당 1개 권장)</h1>
  <p>이것은 문단(Paragraph)입니다.</p>
  
  <p>
    검색 엔진을 위한 <strong>강조(Bold)</strong> 처리.
    <a href="[https://www.google.com](https://www.google.com)">구글로 이동(Hyperlink)</a>
  </p>

  <img src="images/sample.png" alt="대체 텍스트(경로 오류 시 출력)">

  <ul> <li>항목 1</li>
    <li>항목 2</li>
  </ul>
</body>
</html>
```

### [코드] CSS 선택자와 명시도 (Specificity)
```css
/* 1. 전체 선택자 (가중치 최하) */
* {
  color: red;
}

/* 2. 요소(Type) 선택자 */
h2 {
  color: orange;
}

/* 3. 클래스(Class) 선택자 (재사용성 목적) */
.text-green {
  color: green;
}

/* 4. 아이디(ID) 선택자 (문서 내 유일성 목적, 가중치 높음) */
#main-title {
  color: purple;
}

/* 5. 자식 결합자(>)와 자손 결합자(공백) */
.box > span { /* box 클래스의 직계 자식인 span만 선택 */
  font-size: 20px;
}
ul li { /* ul 하위의 모든 li(자손) 선택 */
  color: brown;
}

/* [주의] 명시도 충돌 시 캐스케이딩(Cascading) 규칙 */
.text-green { color: green; }
.text-orange { color: orange; } 
/* HTML에서 class="text-orange text-green" 순서로 적어도,
   CSS 파일에서 나중에 선언된 .text-orange 스타일이 덮어씀 */
```

### [코드] 박스 모델과 상대 단위(REM) 설정
```css
/* 박스 크기를 테두리(Border) 기준으로 고정하는 필수 초기화 설정 */
* {
  box-sizing: border-box;
}

.box-model-example {
  /* 단축 속성 (Top, Right, Bottom, Left 시계방향) */
  margin: 50px 0 0 25px; /* Top 50px, Left 25px */
  padding: 10px 20px;    /* 상하 10px, 좌우 20px */
  border: 2px solid black;
  
  /* 상대 단위 적용 (최상위 루트 기본값 16px 기준) */
  font-size: 1.5rem; /* 16px * 1.5 = 24px 크기로 렌더링 */
  width: 100px; /* border-box 선언 시, Content 크기가 100px에서 Padding/Border만큼 줄어들어 전체 너비가 100px로 유지됨 */
}
```

---

## 🚀 복습 및 AI 인사이트
* **헷갈렸던 점:**
    * `margin-top: 50px;`는 박스 자체를 위로 50px 올리는 것이 아니라, 박스의 **위쪽 바깥 여백을 50px 밀어내어** 실제로는 박스가 아래로 내려가는 것임을 명확히 인지
    * 클래스 다중 적용 시 `class="green orange"`라고 HTML에 작성한 순서가 우선순위를 결정하는 것이 아니라, **CSS 파일 내에서 더 아래(마지막)에 선언된 코드가 캐스케이딩 원칙에 의해 최종 적용**됨을 디버거 취소선을 통해 확인
* **AI 활용 팁:**
    * 모르는 CSS 속성이나 태그가 발생했을 때, 구글이나 AI에게 질문 시 반드시 `MDN` 키워드를 함께 붙여 (예: `css background-color mdn`) 공식 표준 문서 기반의 정확한 예시를 습득하는 습관 들이기
    * 레이아웃이 깨지거나 박스 크기가 의도와 다를 때, 크롬 개발자 도구(F12)의 '검사(Select)' 아이콘을 활성화하여 마우스를 올리고 Box Model 다이어그램(주황, 노랑, 초록색 영역)을 눈으로 직접 확인하며 디버깅 수행