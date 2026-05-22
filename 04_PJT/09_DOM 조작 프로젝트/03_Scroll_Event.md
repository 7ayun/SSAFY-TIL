# [관통 PJT] Scroll Event

---

## 1. Scroll Event란?

**Scroll Event**는 웹 페이지의 콘텐츠를 상하좌우로 이동(스크롤)시킬 때마다 **지속적으로 발생**하는 이벤트다.

**주요 속성:**

| 속성 | 설명 |
|------|------|
| `window.scrollY` | 브라우저 창의 **수직 스크롤 위치** (권장 표준 속성) |
| `element.scrollHeight` | 요소에 포함된 전체 콘텐츠의 높이 (스크롤되지 않은 부분까지 포함). 스크롤 막대의 끝에 도달했는지 확인할 때 사용 |

---

## 2. 스크롤 시 나타나는 요소 실습

**목표:** 처음에는 투명하게 숨겨진 카드들이 스크롤을 내려 해당 위치에 도달할 때 부드럽게 나타나도록 구현한다.

### 동작 원리

1. 모든 카드를 처음에 투명(`opacity: 0`)하게 설정
2. 스크롤 이벤트가 발생할 때마다 각 카드의 **상단 위치(top)** 를 확인
3. 카드의 상단이 화면 높이보다 200px 앞에 도달하면 `visible` 클래스 추가 → 카드가 나타남
4. `transition` 속성으로 부드럽게 등장

### 주요 코드

```javascript
// 카드 목록 가져오기
const cards = document.querySelectorAll('.card')

function checkScroll() {
  const windowHeight = window.innerHeight   // 현재 창의 높이(Viewport 높이)

  cards.forEach(card => {
    const cardTop = card.getBoundingClientRect().top  // 카드 상단의 Viewport 내 위치

    // 바로 나타나기보다 좀 더 위에서 나타날 수 있도록 200px 여유를 둠
    if (cardTop < windowHeight - 200) {
      card.classList.add('visible')
    }
  })
}

window.addEventListener('scroll', checkScroll)
```

**핵심 포인트:**
- `getBoundingClientRect().top` 은 스크롤할 때마다 변하는 **Viewport 기준의 상단 좌표**다.
- 스크롤을 내릴수록 이미 올라간 요소의 `top` 값은 음수(-가 됨)가 되어 점점 작아진다.
- `windowHeight - 200` 으로 카드가 화면 완전히 진입하기 조금 전에 미리 등장시켜 더 자연스럽게 보이도록 한다.

---

## 3. 스크롤 진행률 표시 실습

**목표:** 언론사 기사처럼 화면 상단에 스크롤 진행률을 나타내는 프로그레스 바를 구현한다.

### 스크롤 진행률 계산 원리

단순히 `현재 스크롤 높이 / 전체 높이`로 계산하면 틀린다. 처음 화면에 보이는 Viewport 높이만큼은 스크롤할 수 없는 영역이므로 제외해야 한다.

```
스크롤 진행률 = 현재 스크롤 위치 / 스크롤 이동 가능 범위

스크롤 이동 가능 범위 = 전체 높이(scrollHeight) - Viewport 높이(innerHeight)
```

> Viewport가 600px이고 전체 페이지가 2000px이라면, 실제로 스크롤할 수 있는 범위는 1400px이다.

### 주요 속성

| 속성 | 설명 |
|------|------|
| `document.documentElement.scrollHeight` | 콘텐츠의 **총 높이** (바깥으로 넘쳐서 보이지 않는 콘텐츠도 포함) |
| `window.innerHeight` | 막대 높이를 포함한 **창 내부 높이** (Viewport 높이) |

### 주요 코드

```javascript
function updateProgress(e) {
  const scrollY = window.scrollY                              // 현재 스크롤 위치

  // 스크롤 가능한 범위 = 전체 높이 - Viewport 높이
  const scrollHeight = document.documentElement.scrollHeight - window.innerHeight

  // 진행률 (0 ~ 1 사이 값)
  const progress = scrollY / scrollHeight

  // scaleX()로 프로그레스 바의 너비를 진행률에 맞게 조절
  progressFill.style.transform = `scaleX(${progress})`
}

window.addEventListener('scroll', updateProgress)
```

---

## 4. Parallax Scroll (패럴랙스 스크롤)

### 개념

스크롤할 때 **화면의 배경과 앞에 보이는 내용이 서로 다른 속도로 움직여**, 화면에 깊이나 3D 같은 입체감을 주는 시각적 효과다.

실제 적용 예시로는 DDD Hotel 같은 고급 웹사이트에서 배경과 텍스트가 각자 다른 속도로 움직이는 것을 볼 수 있다.

---

## 5. Parallax Scroll 실습 1 — 레이어별 속도 차이

**원리:** 각 레이어(배경, 중간, 앞)에 서로 다른 이동 속도를 설정하여, 스크롤할수록 레이어들이 다른 속도로 위로 올라가게 한다.

```javascript
// 각 레이어의 속도 설정
const speeds = {
  back:   0.2,   // 배경: 가장 느림
  middle: 0.3,   // 중간
  front:  0.4    // 앞: 가장 빠름
}

window.addEventListener('scroll', () => {
  const scrolled = window.scrollY  // 현재 스크롤 위치

  // 각 레이어를 다른 속도로 위로 이동 (음수 = 위로 올라가게)
  layerBack.style.transform   = `translateY(${-scrolled * speeds.back}px)`
  layerMiddle.style.transform = `translateY(${-scrolled * speeds.middle}px)`
  layerFront.style.transform  = `translateY(${-scrolled * speeds.front}px)`
})
```

**핵심 포인트:**
- 스크롤을 내리면 요소가 위로 올라가야 하므로 음수(`-scrolled`)를 사용한다.
- 속도 계수가 클수록 더 빠르게 움직이므로, 앞에 있는 요소가 배경보다 빠르게 이동해 입체감이 생긴다.

---

## 6. Parallax Scroll 실습 2 — 요소별 독립 동작

**원리:** 각 요소에 독립적인 이동 방향과 속도를 부여하여 스크롤 시 애니메이션처럼 동작하도록 설정한다. X/Y 좌표를 모두 활용해 대각선 이동도 가능하다.

```javascript
// 각 레이어의 속도 설정
const speeds = {
  mountain: 0.05,  // 산: 매우 느림
  cloud:    0.1,   // 구름
  branch:   0.1,   // 나뭇가지: 아래로
  bird:     0.4    // 새: 가장 빠름, 대각선 이동
}

window.addEventListener('scroll', () => {
  const scrolled = window.scrollY  // 화면이 얼마나 내려왔는지 위치 확인

  // 각 레이어를 다른 속도로 이동
  layerMountain.style.transform = `translate(${scrolled * speeds.mountain * 1.5}px, ${scrolled * speeds.mountain}px)`
  layerCloud.style.transform    = `translate(${scrolled * speeds.cloud}px)`

  // 나뭇가지는 조금씩 아래로 이동
  layerBranch.style.transform   = `translateY(${scrolled * speeds.branch}px)`

  // 새는 좌상단 대각선으로 이동 (위 + 왼쪽) → translate(x, y) 활용
  layerBird.style.transform     = `translate(${-scrolled * speeds.bird * 0.1}px, ${-scrolled * speeds.bird * 0.05}px)`
})
```

**핵심 포인트:**
- `translate(x, y)` : 첫 번째 값이 X 좌표, 두 번째 값이 Y 좌표
- 요소마다 이동 방향과 속도를 다르게 설정하면 복잡한 애니메이션 효과를 만들 수 있다.
- 배경(산)은 느리게, 새(foreground)는 빠르게 설정해 원근감을 연출한다.

---

## 💡 한 줄 요약
> Scroll Event와 각 요소의 위치·속도 계산을 조합하면, 스크롤에 반응하는 요소 등장 / 진행률 표시 / Parallax 입체 효과를 모두 구현할 수 있다.

## ❓ 더 찾아볼 것
- `window.scrollY` vs `document.documentElement.scrollTop` 차이
- `IntersectionObserver` API (스크롤 감지를 더 효율적으로 처리하는 최신 방법)
- CSS `scroll-behavior: smooth` 속성
- CSS `position: sticky` 를 활용한 스크롤 효과
- Parallax 라이브러리 (Rellax.js, ScrollMagic 등)
