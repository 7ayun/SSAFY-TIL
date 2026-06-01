# [Vue] SFC

---

## 1. Component

컴포넌트는 웹 페이지를 구성하는 **재사용 가능한 UI 조각**이다. 마치 레고 블록처럼 독립적인 기능을 가지며, 각 블록을 조립해 페이지 전체를 구성한다.

**비유:** 애플리케이션을 집 짓기에 비유하면, 컴포넌트는 각 방(침실, 거실, 주방)에 해당한다. 모든 가구를 한 방(원룸)에 넣는 것보다, 방마다 역할을 분리(투룸 이상)하는 것이 훨씬 관리하기 쉽다.

**특징:**
- **독립성**: 각 컴포넌트는 독립적으로 기능하며, 수정 시 해당 컴포넌트에만 영향을 줌
- **재사용성**: 하나의 컴포넌트를 여러 곳에서 반복 사용 가능
- **유지보수**: 같은 컴포넌트로 만들어진 모든 곳이 한 번의 수정으로 동시에 변경

**트리 구조:** 자연스럽게 중첩된 컴포넌트 트리 형태로 구성된다.

```
<Root>
├── <Header>
├── <Main>
│   ├── <Article> × 2
└── <Aside>
    └── <Item> × 3
```

---

## 2. Single File Component (SFC)

SFC는 하나의 `.vue` 파일 안에 컴포넌트의 **HTML, JavaScript, CSS를 모두 담는** Vue의 개발 방식이다.

```vue
<!-- MyComponent.vue -->
<template>
  <div class="greeting">{{ msg }}</div>
</template>

<script setup>
import { ref } from 'vue'
const msg = ref('Hello World!')
</script>

<style scoped>
.greeting {
  color: red;
}
</style>
```

- Vue SFC는 일반 HTML 파일로 직접 실행할 수 없으며, **Vite 같은 빌드 도구로 컴파일**되어야 한다.
- 연습용으로는 [Vue SFC Playground](https://play.vuejs.org)를 활용할 수 있다.

---

## 3. SFC 구성요소

각 `.vue` 파일은 세 가지 최상위 언어 블록으로 구성된다. 작성 순서는 자유이나, 일반적으로 `template → script → style` 순서를 따른다.

### `<template>` 블록

- HTML 구조를 담당하며, 화면에 렌더링될 마크업을 작성
- **파일 당 하나만** 포함 가능
- `<template>` 태그 자체는 실제 DOM에 렌더링되지 않으며, 내부 요소를 감싸는 래퍼 역할을 함

### `<script setup>` 블록

- JavaScript 로직을 담당
- `setup` 속성은 과거 `createApp` 인스턴스 내부의 `setup()` 함수를 대체한다.
- **파일 당 하나만** 포함 가능 (`<script setup>` 기준)
- `setup` 속성이 붙으면 `return` 없이도 선언한 변수·함수를 템플릿에서 바로 사용 가능 → 개발 편의성 향상

```vue
<script setup>
import { ref } from 'vue'
const msg = ref('Hello World!')
// return 없이도 템플릿에서 msg 사용 가능
</script>
```

> 일반 `<script>` 태그는 여러 개 작성할 수 있지만, 관리 측면에서 별도 파일로 분리하는 것을 권장한다.

### `<style scoped>` 블록

- CSS 스타일을 담당
- `scoped` 속성이 붙으면 해당 스타일은 **현재 컴포넌트에만 적용**됨
- `scoped`를 생략하면 전역으로 적용되어 다른 컴포넌트에도 영향을 미치므로, 웬만하면 `scoped`를 붙이는 것이 정신건강에 유리하다.

---

## 4. Vite (빌드 도구)

Vite(발음: "비트")는 Vue 프론트엔드 개발을 위한 **빌드 도구이자 개발 서버**다. Vue를 만든 Evan You가 참여해 개발했으며, Vue 개발에 최적화되어 있다.

> 배경: 과거 Vue CLI는 개발 서버 실행에 컴퓨터에 따라 5분 이상 걸리기도 했다. Vite 도입 이후 속도가 비약적으로 개선되었다.

**특징:**
- 개발 서버 시작 속도 매우 빠름 (파일을 필요할 때만 요청)
- 코드 수정 시 실시간에 가까운 반영 (HMR)
- 배포 시 **Rollup** 번들러를 사용해 최적화된 형태로 번들링

### 빌드(Build)란?

프로젝트 소스 코드를 브라우저가 이해할 수 있는 최적화된 파일로 변환하는 과정이다.

- **트리 쉐이킹(Tree Shaking)**: `import`하지 않은 코드는 최종 번들에서 자동 제거 (나무를 흔들어 낙엽 떨구기)
- **난독화**: 변수명·함수명을 짧게 줄여 기술 노출 최소화 및 파일 크기 감소
- **번들링**: 여러 모듈 파일을 하나(또는 소수)의 파일로 합쳐 HTTP 요청 수를 줄이고 로딩 속도 향상
- **브라우저 호환성**: ES6+ 문법을 구버전 브라우저도 읽을 수 있는 형태로 트랜스파일

---

## 5. NPM과 Node.js

### NPM (Node Package Manager)

NPM은 JavaScript 패키지를 모아 놓은 **거대한 저장소이자 명령어 도구**다. Python의 `pip`와 유사한 역할이다.

```bash
npm install        # package.json 기반으로 패키지 설치
npm install <패키지명>  # 특정 패키지 설치
```

- 프로젝트에 사용된 모든 패키지 목록과 버전 정보는 `package.json`에 기록된다. (`pip freeze`의 `requirements.txt`와 유사하지만 더 많은 정보 포함)

### Node.js

JavaScript는 원래 브라우저 안에서만 동작하는 언어였다. (이 때문에 지금까지 `console.log` 결과를 브라우저 개발자 도구에서 확인했다.)

Node.js는 **Chrome의 V8 JavaScript 엔진을 브라우저 밖으로 꺼내**, 서버(컴퓨터)에서도 JavaScript를 실행할 수 있게 만든 환경이다.

- **Server-Side 실행 가능**: VS Code 터미널에서 직접 JS 실행 결과 확인 가능
- **풀스택 개발**: 프론트엔드와 백엔드 모두 JavaScript 하나로 구현 가능
- **현황**: JavaScript는 현재 전 세계 프로그래밍 언어 랭킹 1위

---

## 💡 한 줄 요약
> `.vue` 파일 하나에 HTML·JS·CSS를 담은 SFC 방식으로, Vite와 NPM을 활용해 컴포넌트 단위로 체계적인 Vue 개발을 한다.

## ❓ 더 찾아볼 것
- Vue SFC Playground 직접 사용해보기
- Vite 공식 문서: https://vitejs.dev
- JavaScript 세계 랭킹 1위의 이유 (Stack Overflow Survey 등)
- ES6 → ES5 트랜스파일링 (Babel과의 비교)
