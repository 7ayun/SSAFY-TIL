# [관통 PJT] Drag Event

---

## 1. Drag Event란?

**Drag Event**는 웹 페이지의 요소를 **끌어서 이동시키는 과정**에서 발생하는 이벤트다. 단순한 하나의 이벤트가 아닌, 끌기 시작부터 끌기 종료까지 **모든 단계에서 순서대로 발생하는 여러 개의 이벤트 그룹**이다.

"이미지 업로드할 때 드래그 앤 드롭으로 파일을 올리는 기능도 이 드래그 이벤트를 이용해 구현한다. drop 이벤트를 감지하면 파일 정보를 읽어와서 업로드하는 방식이다."

---

## 2. Drag Event 사용 조건

드래그 이벤트를 사용하려면 드래그할 HTML 요소에 **`draggable="true"`** 속성을 반드시 추가해야 한다.

```html
<!-- draggable 미설정(false) → 드래그 불가, 금지 커서 표시 -->
<div class="card">1</div>

<!-- draggable="true" → 드래그 가능 -->
<div class="card" draggable="true">1</div>
```

- 기본값은 `false`이므로, 설정하지 않으면 요소가 드래그되지 않는다.

---

## 3. Drag Event 종류

### 드래그되는 요소에서 발생하는 이벤트 (마우스로 끌고 있는 요소 기준)

| 이벤트 | 발생 시점 |
|--------|----------|
| `dragstart` | 사용자가 요소를 **끌기 시작**하는 순간 |
| `drag` | 사용자가 요소를 **끌고 다니는 동안** 계속 발생 |
| `dragend` | 사용자가 **마우스 버튼을 놓거나** 드래그가 끝나는 순간 |

### 드래그 영역에서 발생하는 이벤트 (드래그 중인 요소 아래에 위치한 영역 기준)

| 이벤트 | 발생 시점 |
|--------|----------|
| `dragenter` | 끌고 있는 요소가 대상 영역으로 **들어올 때** |
| `dragover` | 끌고 있는 요소가 대상 영역 위에 **머물고 있는 동안** 계속 발생 |
| `dragleave` | 끌고 있는 요소가 대상 영역 **밖으로 벗어날 때** |
| `drop` | 끌고 있던 요소를 대상 영역 위에 **놓는 순간** |

**이벤트 흐름 예시 (아이콘을 휴지통에 드롭하는 경우):**

1. 아이콘을 클릭 → 아이콘에서 `dragstart` 발생
2. 아이콘을 휴지통 위로 가져감 → 휴지통에서 `dragenter` 발생
3. 휴지통 위에서 계속 움직임 → 휴지통에서 `dragover` 지속 발생
4. 휴지통에 드롭 → 휴지통에서 `drop` 발생, 아이콘에서 `dragend` 발생

---

## 4. 주요 API

### `e.preventDefault()` — dragover 이벤트에서 필수

드래그 기본 동작이 드래그를 허용하지 않기 때문에, `dragover` 이벤트에서 `preventDefault()`를 호출하지 않으면 마우스 커서에 **드래그 금지 표시(🚫)** 가 나타나고 `drop` 이벤트가 동작하지 않는다.

```javascript
container.addEventListener('dragover', (e) => {
  e.preventDefault()  // 금지 커서 제거 & drop 허용
  // ...
})
```

### `e.clientX` — 현재 마우스 X 좌표

```javascript
const mouseX = e.clientX  // 드래그 중인 마우스 X좌표
```

- 현재 마우스 위치 값이 담겨 있으며, **Viewport(현재 화면) 내의 수평 좌표**를 반환한다.

> **Viewport란?** 웹 페이지 전체 중에서 현재 사용자가 실제로 보고 있는 화면 영역이다. 스크롤 위치에 따라 달라진다.

### `element.getBoundingClientRect()` — 요소의 크기 및 위치 반환

```javascript
const rect = card.getBoundingClientRect()
// rect.left, rect.width, rect.top, rect.right 등 사용 가능
```

- 요소의 크기와 Viewport에서의 **상대적인 위치 정보**를 반환한다.

### `insertBefore()` — 특정 노드 앞에 삽입

```javascript
// 기준 노드 앞에 새 노드를 추가
parentNode.insertBefore(newNode, referenceNode)
```

- 기준이 되는 노드의 앞에 새로운 노드를 추가할 때 사용한다.

---

## 5. 카드 순서 바꾸기 실습

**목표:** 카드를 마우스로 드래그하면 다른 카드들과 위치를 실시간으로 바꾸는 드래그 앤 드롭 UI를 구현한다.

### 카드 배치 원리

마우스 포인터의 X 좌표와 각 카드의 중심 X 좌표를 비교한다.

- 마우스 포인터가 바닥 카드의 **중심보다 앞쪽(왼쪽)** → 드래그 카드를 바닥 카드 **앞**에 삽입
- 마우스 포인터가 바닥 카드의 **중심보다 뒤쪽(오른쪽)** → 드래그 카드를 바닥 카드 **뒤**에 삽입

```
[카드 중심]
    |
    ▼
마우스가 이쪽(왼쪽) → 드래그 카드가 앞으로
마우스가 이쪽(오른쪽) → 드래그 카드가 뒤로
```

### 주요 코드

```javascript
// 요소 가져오기 (버블링 활용 → 컨테이너 하나에만 리스너 연결)
const container = document.querySelector('#container')
let draggedCard = null  // 현재 드래그 중인 카드 저장

// 1. 드래그 시작: 드래그 중인 카드 저장 + 스타일 추가
container.addEventListener('dragstart', (e) => {
  draggedCard = e.target      // 실제 이벤트가 발생한 요소
  e.target.classList.add('dragging')
})

// 2. 드래그 종료: 초기화
container.addEventListener('dragend', (e) => {
  draggedCard = null
  e.target.classList.remove('dragging')
})

// 3. dragover: 금지 커서 제거 + 실시간 위치 계산
container.addEventListener('dragover', (e) => {
  e.preventDefault()

  // 드래그 중인 카드를 제외한 나머지 카드 목록
  const otherCards = document.querySelectorAll('.card:not(.dragging)')

  const mouseX = e.clientX   // 현재 마우스 X 좌표

  let closestCard = null
  let closestCardCenter = null
  let minDistance = Infinity

  // 가장 가까운 카드 찾기
  otherCards.forEach(card => {
    const rect = card.getBoundingClientRect()
    const cardCenter = rect.left + rect.width / 2      // 카드 중심 X 좌표
    const distance = Math.abs(mouseX - cardCenter)     // 마우스와 카드 중심 거리

    if (distance < minDistance) {
      minDistance = distance
      closestCard = card
      closestCardCenter = cardCenter
    }
  })

  // 드래그 중인 카드 위치 변경
  if (closestCard) {
    if (mouseX < closestCardCenter) {
      // 마우스가 카드 중심보다 왼쪽 → 해당 카드 앞에 삽입
      container.insertBefore(draggedCard, closestCard)
    } else {
      // 마우스가 카드 중심보다 오른쪽 → 해당 카드의 다음 형제 앞에 삽입
      container.insertBefore(draggedCard, closestCard.nextSibling)
    }
  }
})
```

**핵심 포인트:**

- `e.target` : 실제 이벤트가 발생한 요소 (드래그된 카드)
- `e.currentTarget` : 이벤트 리스너가 설정된 요소 (컨테이너)
- 개별 카드마다 이벤트 리스너를 등록하는 대신, **부모 컨테이너 하나**에만 등록하고 **버블링**을 활용한다.
- `dragover`가 실행될 때마다 마우스 좌표와 각 카드 중심 좌표를 비교해 가장 가까운 카드를 찾고, `insertBefore`로 위치를 실시간으로 재배치한다.

---

## 💡 한 줄 요약
> Drag Event는 dragstart/dragover/dragend 등 단계별 이벤트 그룹으로 구성되며, 마우스 좌표와 요소의 위치를 비교해 카드 순서를 실시간으로 재배치할 수 있다.

## ❓ 더 찾아볼 것
- `e.target` vs `e.currentTarget` 심화 이해
- `getBoundingClientRect()` 반환 값의 모든 속성 (`top`, `left`, `right`, `bottom`, `width`, `height`)
- `insertBefore` vs `appendChild` 차이
- HTML5 Drag and Drop API 전체 스펙 (MDN)
- `DataTransfer` 객체 활용 (파일 드롭 업로드 구현)
