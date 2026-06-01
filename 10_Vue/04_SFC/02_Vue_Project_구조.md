# [Vue] Vue Project 구조

---

## 1. Vue 프로젝트 생성

Vite 기반의 Vue 프로젝트는 아래 명령어로 생성한다.

```bash
npm create vue@latest
```

`$`로 시작하는 명령어는 터미널(Terminal) 또는 명령 프롬프트(Command Prompt)에 입력하는 명령어임을 의미한다. `latest`는 가장 최신 안정화 버전(stable version)을 사용하겠다는 의미다.

### 프로젝트 생성 설정 옵션

```
Project name: vue-project          ← Tab: 기본값, Enter: 확정
Use TypeScript? No                 ← 아직 학습 전이므로 No
Select features:
  □ JSX Support                    ← JS로 HTML 태그를 직접 그리는 문법, 미사용
  □ Router (SPA development)       ← 추후 학습 예정
  □ Pinia (state management)       ← 추후 학습 예정
  □ Vitest (unit testing)
  □ End-to-End Testing
  □ Linter (error prevention)
  □ Prettier (code formatting)
Skip example code? No              ← 오늘은 첫 체험이므로 No (이후엔 Yes 권장)
```

### 프로젝트 생성 후 실행

```bash
cd vue-project          # 프로젝트 폴더로 이동
npm install             # 패키지 설치 (반드시 package.json이 있는 루트에서 실행)
npm run dev             # 개발 서버 실행
```

**⚠️ 주의:** `npm install`은 반드시 `package.json`이 있는 프로젝트 루트 디렉토리에서 실행해야 한다. 다른 위치에서 실행하면 `Cannot read package.json` 에러가 발생한다.

서버 실행 후 `http://localhost:5173`으로 접속하면 "You did it!" 웰컴 페이지를 확인할 수 있다.

---

## 2. 모듈과 번들러

### Module

모듈은 프로그램을 구성하는 **독립적인 코드 블록** (`.js` 파일 하나 = 하나의 모듈)이다. 애플리케이션이 복잡해질수록 모듈 수가 급증하고, 모듈 간 **의존성(Dependency)** 이 깊어진다.

### Bundler

번들러는 이렇게 깊게 얽힌 여러 모듈 파일을 **하나(혹은 소수)의 번들로 묶고 최적화**하는 도구다.

- **의존성 관리**: 복잡한 모듈 간 의존 관계를 자동으로 처리
- **코드 최적화**: 공백 제거, 변수명 단축, 불필요 코드 제거
- **리소스 관리**: JS뿐 아니라 CSS, 이미지 등 모든 정적 자원 처리

Vite는 내부적으로 **Rollup** 번들러를 사용하며, 개발자가 별도 설정 없이도 번들링 환경을 제공한다.

---

## 3. Vue Project 기본 구조

```
vue-project/
├── node_modules/          # 설치된 패키지들의 실제 파일 저장소
├── public/                # 번들링되지 않는 정적 파일
├── src/                   # 핵심 소스 코드 폴더
│   ├── assets/            # 이미지, 폰트, 스타일시트 등 정적 자원
│   ├── components/        # .vue 컴포넌트 파일들
│   ├── App.vue            # 최상위 Root 컴포넌트
│   └── main.js            # 애플리케이션 초기화 및 마운트 시작점
├── .gitignore
├── index.html             # Vue 앱의 기본 HTML 파일 (SPA의 유일한 HTML)
├── jsconfig.json          # 컴파일 옵션 및 모듈 시스템 설정
├── package-lock.json
├── package.json
├── README.md
└── vite.config.js         # Vite 프로젝트 설정 파일
```

### `public/`

번들러가 **최적화하지 않는** 폴더. 소스 코드에서 직접 참조되지 않거나 `import`할 필요 없는 파일을 위치시킨다.

- 파비콘 (`favicon.ico`)
- `robots.txt` (크롤러 접근 허용/차단 정보)
- `manifest.json` (브라우저에 페이지 정보를 전달)
- 항상 루트 절대 경로로 참조: `public/icon.png` → `/icon.png`

### `src/`

**가장 핵심 폴더**. 실제 작업할 소스 코드가 위치한다.

| 경로 | 역할 |
|------|------|
| `src/assets/` | 컴포넌트에서 참조하는 이미지, 폰트, CSS 등 (번들링 대상) |
| `src/components/` | 재사용 가능한 `.vue` 컴포넌트 파일들이 위치하는 곳 |
| `src/App.vue` | **Root 컴포넌트** — 모든 하위 컴포넌트를 포함하는 최상위 파일 |
| `src/main.js` | Vue 앱을 초기화하고 `App.vue`를 DOM에 마운트하는 시작점 |

### `index.html`

SPA(Single Page Application)의 유일한 HTML 파일. `main.js`가 실행되면서 `App.vue`를 `<div id="app">` 위치에 마운트한다.

```html
<!-- index.html -->
<div id="app"></div>
<script type="module" src="/src/main.js"></script>
```

- Bootstrap CDN, Axios CDN 등 외부 리소스를 여기에 추가하면 모든 컴포넌트에서 사용 가능하다.

### 설정 파일

- **`jsconfig.json`**: IDE에 컴파일 옵션, 모듈 시스템, 경로 별칭(`@` = `src/`) 등을 알려주는 설정 파일
- **`vite.config.js`**: Vite의 플러그인, 빌드 옵션, 개발 서버 설정을 담당. 배포 시 추가 설정이 필요하다면 이 파일을 수정한다.

---

## 4. 패키지 관리

### `package.json` — 설계도

프로젝트의 메타데이터와 패키지 의존성을 정의하는 파일. Python의 `requirements.txt`와 유사하지만 더 많은 정보를 담는다.

```json
{
  "scripts": {
    "dev": "vite",         // npm run dev → vite 실행
    "build": "vite build", // npm run build → 배포용 빌드
    "preview": "vite preview"
  },
  "dependencies": {
    "vue": "^3.5.13"
  },
  "devDependencies": {
    "vite": "^6.0.3",
    "@vitejs/plugin-vue": "^5.2.1"
  }
}
```

> `^` (캐럿): 메이저 버전은 고정하되, 마이너·패치 버전은 호환되는 최신 버전 허용

> `npm run dev`가 동작하는 건 `scripts` 안에 `"dev": "vite"`가 정의되어 있기 때문. `start`가 없으면 `npm run start`는 에러가 발생한다. 필요하면 직접 추가하면 된다.

### `package-lock.json` — 상세 내역서

실제 설치된 패키지들의 **정확한 버전 정보**를 고정하는 파일. `npm install` 시 자동으로 생성·업데이트된다.

| 비유 | 역할 |
|------|------|
| `package.json` | "A 패키지 1.x 버전이 필요하다" (설계도) |
| `package-lock.json` | "A 패키지 1.1.0, 서브 패키지 B 2.4.1, C 3.5.2 ..." (구매 내역서) |

- 협업 시 팀원 모두가 동일한 패키지 버전으로 개발환경을 재현할 수 있다.
- Git에 함께 올려야 한다 (`package.json`과 항상 같이 공유).

### `node_modules/` — 자재 창고

`package.json`과 `package-lock.json`에 따라 실제로 설치된 모든 패키지가 저장되는 폴더.

- 용량이 매우 크므로 `.gitignore`에 포함 — **절대 Git에 올리지 않는다.**
- 삭제 후 `npm install`로 언제든 재생성 가능.

| 파일 | Git 포함 여부 |
|------|--------------|
| `package.json` | ✅ 포함 |
| `package-lock.json` | ✅ 포함 |
| `node_modules/` | ❌ 제외 (`.gitignore`) |

---

## 5. Vue Component 활용

### 컴포넌트 사용 3단계

**1단계 — 사전 준비**

초기 생성된 샘플 컴포넌트를 모두 삭제하고, `App.vue`를 초기화한다.

```vue
<!-- App.vue -->
<template>
  <h1>App.vue</h1>
</template>

<script setup>
</script>
```

**2단계 — 컴포넌트 파일 생성**

`components/` 폴더 하위에 **PascalCase**로 `.vue` 파일을 생성한다.

```vue
<!-- components/MyComponent.vue -->
<template>
  <div>
    <h2>MyComponent</h2>
  </div>
</template>

<script setup>
</script>
```

> VS Code 스니펫: `v3s` 입력 후 Enter → Vue Base3 Setup 기본 구조 자동 생성 (Vue VS Code Snippets 확장 필요)

**3단계 — 컴포넌트 등록 (import)**

부모 컴포넌트의 `<script setup>`에서 import하고 템플릿에 태그로 사용한다.

```vue
<!-- App.vue -->
<template>
  <h1>App.vue</h1>
  <MyComponent />
</template>

<script setup>
// '@'는 'src/' 경로를 가리키는 별칭 (상대경로보다 절대경로 권장)
import MyComponent from '@/components/MyComponent.vue'
</script>
```

> `@`는 `src/` 경로의 별칭. 컴포넌트를 나중에 이동시켜도 `@/components/...` 형태면 경로 수정이 최소화되어 유지보수가 편하다.

### 중첩 컴포넌트

컴포넌트 안에 또 다른 컴포넌트를 위치시킬 수 있다.

```
App.vue (Root)
└── MyComponent
    ├── MyComponentItem
    ├── MyComponentItem
    └── MyComponentItem
```

각 컴포넌트 인스턴스는 **독립적으로 동작**한다. Vue DevTools(F12 → Vue 탭)에서 컴포넌트 트리와 각 컴포넌트의 상태를 확인할 수 있다.

### 컴포넌트 이름 스타일 가이드

Vue 공식 스타일 가이드(Priority B: Strongly Recommended)에 따라 컴포넌트 파일명과 태그명은 **PascalCase**로 작성한다.

```
✅ MyComponent.vue   → <MyComponent />
❌ mycomponent.vue   → <mycomponent />
```

---

## 💡 한 줄 요약
> `npm create vue@latest`로 프로젝트를 생성하면, `src/` 안의 `.vue` 파일들이 App.vue를 루트로 트리 구조를 이루며, package.json이 패키지 관리의 설계도 역할을 한다.

## ❓ 더 찾아볼 것
- TypeScript + Vue 조합 학습
- Vue Router (SPA 라우팅)
- Pinia (상태 관리)
- Semantic Versioning (SemVer) — 메이저/마이너/패치 버전 의미
- `npm run build` 결과물 확인 (`dist/` 폴더)
