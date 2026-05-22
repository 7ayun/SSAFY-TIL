# [관통 PJT] Input Event

---

## 1. DOM 조작과 Event란?

**DOM(Document Object Model)** 은 웹 페이지의 모든 내용을 JavaScript가 이해하고 조작할 수 있도록 만든 구조화된 모델이다. 트리 형태로 구성되며, `document.querySelector()` 등을 통해 접근한다.

**DOM 조작**은 JS가 웹페이지의 뼈대(HTML)와 디자인(CSS)을 실시간으로 통제하는 핵심 기술이다.

**Event**는 웹 페이지 내에서 발생하는 모든 사건(클릭, 키 입력, 스크롤 등)을 말하며, JavaScript가 이를 감지하여 특정 동작을 실행하도록 만드는 방아쇠 역할을 한다.

> DOM 조작과 Event 처리는 웹페이지를 단순한 정적 문서가 아니라, 사용자와 끊임없이 대화하고 반응하는 **살아있는 인터페이스**로 만들어준다.

**DOM 조작과 Event 처리 활용 목적:**
- **즉각적인 피드백** : 사용자와 상호작용, 실시간 유효성 확인
- **효율적인 정보 관리** : 평소에는 숨겨두었다가 필요할 때 보여줌
- **시각적 매력과 몰입감** : 사용자 서비스 이용 경험 개선

---

## 2. Input Event 개념

**Input Event**는 `<input>`, `<textarea>` 등의 요소에서 **데이터를 입력할 때마다 발생**하는 이벤트다.

| 이벤트 | 발생 시점 | 특징 |
|--------|----------|------|
| `input` | 요소의 value(값)이 변경될 때 | 붙여넣기에도 반응 → 유효성 검사에 적합 |
| `keyup` | 키보드 키가 눌렸다가 떼어질 때 | 특정 키(예: Enter 키)를 눌렀을 때만 동작 수행 |

강사 설명: "이벤트는 MDN에 엄청 많이 있다. 프론트를 하기로 맘먹었다면 어떤 이벤트가 있는지 한 번씩 보는 것이 유의미하다."

---

## 3. CSS Animation과 @keyframes

JS를 사용하지 않고도 복잡한 움직임을 구현할 수 있게 해주는 CSS 기능이다.

**주요 animation 속성:**
- `animation-name` : 어떤 동작을 할지 (Keyframes 이름 지정)
- `animation-duration` : 몇 초 동안 진행할지
- `animation-timing-function` : 속도 변화 (느리게, 시작/끝 등)

```css
/* 애니메이션의 이름은 'pulse' */
@keyframes pulse {
  0% {                          /* 0%: 시작 지점 (초기 상태) */
    transform: scale(1);        /* 원래 크기 */
    opacity: 1;                 /* 불투명도 100% */
  }
  50% {                         /* 50%: 중간 지점 */
    transform: scale(1.2);      /* 크기 1.2배 */
    opacity: 0.5;               /* 반투명 */
  }
  100% {                        /* 100%: 종료 지점 */
    transform: scale(1);        /* 다시 원래 크기로 돌아옴 */
    opacity: 1;
  }
}
```

---

## 4. CSS transforms

요소를 이동, 회전, 크기 조절, 기울이기 등을 할 수 있게 해주는 CSS 속성이다.

- `transform: [함수]([값]);` 형태로 사용

**주요 함수:**

| 함수 | 역할 |
|------|------|
| `translate(x, y)` | x축과 y축으로 이동 |
| `rotate(각도)` | 회전 |
| `scale(x, y)` | x축과 y축 배율 조절 |
| `skew(x각도, y각도)` | x축과 y축 기준으로 기울이기 |

---

## 5. 입력 유효성 확인 실습 — 최대 글자 수 제한

**목표:** 입력창에 20자를 초과하면 배경이 붉게 변하며 좌우로 흔들리는 애니메이션 효과를 구현한다.

### HTML 구조

```html
<!-- 텍스트 입력창 -->
<input type="text" class="text-input" id="textInput" placeholder="최대 20자">
<!-- 글자 수 카운터 -->
<div class="counter" id="counter">0 / 20</div>
```

### CSS — 에러 스타일 및 shake 애니메이션

```css
body {
  margin: 20px;
  background: white;
}

.text-input {
  width: 300px;
  padding: 10px;
  border: 2px solid black;
}

.counter {
  margin-top: 10px;
}

/* 20자를 넘어갔을 때의 스타일 */
.text-input.error {
  background-color: #ffcccc;
  animation: shake 0.5s;
}

/* shake 애니메이션 정의 */
@keyframes shake {
  0%, 100% { transform: translateX(0); }
  25%       { transform: translateX(-5px); }
  75%       { transform: translateX(5px); }
}
```

### JavaScript — Input Event로 유효성 검사

```javascript
// 1. 요소 가져오기
const textInput = document.querySelector('#textInput')
const counter   = document.querySelector('#counter')
const maxLength = 20

// 2. 이벤트 리스너 등록 (핸들러 포함)
textInput.addEventListener('input', function (e) {
  const currentLength = e.target.value.length

  if (currentLength > maxLength) {
    // 글자 자르기 (붙여넣기로 한 번에 100자가 들어와도 20자까지만 남김)
    e.target.value = e.target.value.substring(0, maxLength)

    // 에러 효과 추가
    e.target.classList.add('error')

    // 500ms 후 에러 클래스 제거 → 배경색 원복
    setTimeout(() => {
      e.target.classList.remove('error')
    }, 500)

    return
  }

  // 글자 수 표시 갱신
  counter.textContent = `${currentLength} / ${maxLength}`
})
```

**핵심 포인트:**
- `input` 이벤트는 키보드 입력뿐 아니라 **붙여넣기**에도 발생하므로 유효성 검사에 유리하다.
- `substring(0, maxLength)` 로 초과 글자를 즉시 잘라낸다.
- `setTimeout()` 으로 에러 클래스를 0.5초 후 제거하여 애니메이션이 반복적으로 적용될 수 있도록 한다.
- `e.target` 은 실제로 이벤트가 발생한 요소(입력창)를 가리킨다.

---

## 💡 한 줄 요약
> Input Event와 CSS Animation을 조합하면, 사용자 입력에 즉각적으로 반응하는 실시간 유효성 검사를 시각적으로 구현할 수 있다.

## ❓ 더 찾아볼 것
- MDN에서 다양한 이벤트 종류 탐색 (`Event reference`)
- CSS `transition` vs `animation` 차이
- `e.target` vs `e.currentTarget` 차이 (버블링 관련)
- `setTimeout` vs `setInterval` 차이
