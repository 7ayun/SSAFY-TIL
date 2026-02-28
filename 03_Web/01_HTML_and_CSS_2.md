# [Web] CSS 레이아웃

## 1. 박스 모델의 타입 (Display 속성)
요소들이 화면에 어떻게 흐르고(Normal Flow) 자리를 차지하는지 결정합니다.

* **`block`**: 한 줄 전체를 차지하며, 무조건 줄바꿈이 일어납니다. (width, height, margin, padding 적용 가능)
* **`inline`**: 컨텐츠의 크기만큼만 차지하며, 줄바꿈이 일어나지 않습니다. (width, height 강제 지정 불가)
* **`inline-block`**: `inline`처럼 가로로 배치되면서, `block`처럼 크기와 여백을 자유롭게 조절할 수 있습니다.
* **`none`**: 화면에서 아예 사라지며, 공간조차 차지하지 않습니다.

```css
/* Display 적용 예시 */
.box-block { display: block; width: 100%; }
.text-inline { display: inline; }
.menu-item { display: inline-block; width: 100px; height: 50px; }
.hidden-box { display: none; }
```

---

## 2. 위치 지정 (Position 속성)
요소를 기본 흐름(Normal Flow)에서 뜯어내어 원하는 곳에 자유롭게 배치합니다.

* **`static`**: 기본값. 일반적인 흐름에 따라 배치됩니다.
* **`relative`**: 본인의 `static` 위치를 기준으로 상대적으로 이동합니다. (원래 있던 빈 공간은 유지됨)
* **`absolute`**: 노멀 플로우에서 완전히 벗어나며, 가장 가까운 **`position: relative`인 부모**를 기준으로 절대 이동합니다. (원래 공간이 사라짐)
* **`fixed`**: 현재 보고 있는 **화면(Viewport)**을 기준으로 고정됩니다. 스크롤을 내려도 항상 같은 자리에 떠 있습니다.
* **`sticky`**: 평소엔 `relative`처럼 동작하다가, 스크롤이 특정 임계점(예: `top: 0`)에 닿으면 `fixed`처럼 상단 등에 고정됩니다.

```css
/* Position 적용 예시 */
.parent-box {
  position: relative; /* absolute 자식의 기준점이 됨 */
}
.absolute-child {
  position: absolute;
  top: 0;
  left: 0; /* 부모의 좌측 상단에 딱 붙음 */
}
.floating-button {
  position: fixed;
  bottom: 20px;
  right: 20px; /* 화면 우측 하단에 고정 */
}
```

---

## 3. 쌓임 순서 (Z-Index)
요소들이 겹칠 때 누가 앞으로 나올지(Z축)를 결정합니다.

* 값이 클수록 사용자에게 가깝게(앞으로) 배치됩니다.
* `static`이 아닌 요소(relative, absolute, fixed 등)에만 적용됩니다.
* 부모의 z-index보다 자식의 z-index가 우선할 수 없습니다.

```css
/* Z-Index 적용 예시 */
.background { position: absolute; z-index: 1; }
.character { position: absolute; z-index: 10; }
.always-on-top { position: fixed; z-index: 9999; }
```

---

## 4. 플렉스박스 (Flexbox) ★
1차원(행 또는 열) 방향으로 요소들을 유연하게 정렬하고 공간을 배분하는 가장 강력한 레이아웃 기술입니다.

### 4-1. 부모 (Flex Container) 속성
부모에게 속성을 주어 자식들을 통제합니다.

```css
.flex-container {
  display: flex; /* 플렉스박스 선언 */
  flex-direction: row; /* 주축 방향 설정: 가로(row), 세로(column) */
  flex-wrap: wrap; /* 자식 요소가 넘칠 때 줄바꿈 허용 (반응형에 필수) */
  
  /* 메인축 기준 정렬 (가로) */
  justify-content: center; /* flex-start, flex-end, space-between 등 */
  
  /* 교차축 기준 한 줄 정렬 (세로) */
  align-items: center; /* 수직/수평 중앙 정렬 시 핵심! */
  
  /* 교차축 기준 여러 줄의 공간 분배 (flex-wrap: wrap일 때) */
  align-content: flex-start;
}
```

### 4-2. 자식 (Flex Item) 속성
```css
.flex-item {
  align-self: flex-end; /* 나 혼자만 교차축에서 다르게 정렬하고 싶을 때 */
  flex-basis: 300px; /* 아이템의 최소/기본 시작 너비 (width보다 우선) */
  flex-grow: 1; /* 남는 여백 공간을 n등분하여 비율에 따라 나눠 가짐 */
}
```

---

## 5. 실무 레이아웃 꿀팁

* **마진 상쇄 (Margin Collapsing)**: 블록 요소가 위아래로 마진을 밀어낼 때, 더블로 합쳐지지 않고 더 큰 마진 값 하나로 흡수되는 현상입니다. (CSS의 일관된 배치를 위한 의도적 동작)
* **가운데 정렬 마스터하기**:
  1. **Block 요소**: `margin: 0 auto;` (반드시 width 지정 필요)
  2. **Inline / Inline-block 요소**: 부모에게 `text-align: center;`
  3. **Flex (가장 추천)**: 부모에게 `display: flex; justify-content: center; align-items: center;`