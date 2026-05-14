# [JS] History of Javascript

---

## 1. 웹의 탄생과 JavaScript가 필요해진 이유

HTML은 웹 페이지의 **'설계도'** 일 뿐이며, 한 번 로드되면 아무런 변화도 일어나지 않는 정적인 문서다.
JavaScript는 이 설계도에 **생명을 불어넣는 역할** 을 한다.
- 웹 페이지의 특정 요소를 찾아내고, 내용을 바꾸거나, 색깔을 입히거나, 새로운 요소를 만들어 붙이는 모든 동적인 조작을 담당
- JavaScript가 HTML을 이해하고 조작할 수 있도록 브라우저는 설계도를 **DOM(Document Object Model)** 이라는 객체 트리 구조로 변환해 제공

> **비유:** HTML은 뼈대, CSS는 디자인, JavaScript는 동작 — 세 개가 세트로 움직인다.

---

## 2. 웹의 역사 (1990 ~ 현재)

### 웹의 탄생 (1990)
- **Tim Berners-Lee 경**이 WWW(World Wide Web), URL, HTTP, HTML을 최초로 고안·개발
- 업적이 워낙 커 영국 여왕으로부터 기사 작위를 받음
- 개발 이후 특허를 내지 않고 **무료로 개방** → 웹이 폭발적으로 발전한 결정적 계기

> **참고:** 같은 시기인 1991년, **Linus Torvalds**가 Linux 운영체제를 개발하고 무료로 공개. 여기에 더해 Git도 만든 인물로, 백엔드 서버 생태계를 열었다. 프론트(Web)와 백엔드(Linux)가 거의 동시에 무료로 풀린 셈.

### 웹 브라우저의 대중화 (1993)
- **Netscape Navigator** 출시 — 최초의 상용 브라우저, 당시 시장 점유율 90% 이상
- 이미지 지원, 일반 PC에서 동작 → 인기 폭발
- 단점: 여전히 **정적인 페이지**만 가능 (버튼 눌러도 반응 없음, 새로고침해야 변경)

### JavaScript의 탄생 (1995)
- Netscape가 동적 기능을 위해 개발자 **Brendan Eich**에게 스크립트 언어 개발을 의뢰
- Eich는 단 **10일 만에** 'Mocha'라는 언어를 개발 (거의 잠을 안 자며 집중)
- Mocha → LiveScript → 당시 Java가 인기였기 때문에 **JavaScript**로 이름 변경
- ⚠️ **Java와 JavaScript는 완전히 다른 언어다** — 이름만 따온 것

> 10일 만에 만든 탓에 언어 내부에 허술한 부분이 많다.
> 예) `null`의 타입이 `object`인 버그, 비교 연산자 `===`를 써야 하는 이유 등. 
> 이미 너무 많이 쓰여 수정이 불가능한 상태가 됐다.

### JavaScript 파편화 (1996)
- Microsoft가 JavaScript를 **리버스 엔지니어링(역공학)** 해 'JScript'를 만들어 IE 3.0에 도입
- 여러 기업이 각자의 규격을 만들기 시작 → **JS 파편화** 시작
- 개발자들은 브라우저마다 다른 언어로 개발해야 하는 혼란에 빠짐 → 웹 표준의 중요성 인식

### 1차 브라우저 전쟁 (1995 ~ 2001)
- Microsoft가 IE를 Windows에 무료로 번들 → Netscape 빠르게 몰락
- IE 시장 점유율 **96%** 달성 → 사실상 독점

- Netscape의 선택: 코드를 그냥 열면 MS가 그대로 사용 → 코드 공개도, 숨기기도 아닌 **'설계 명세(표준)만 공개'**
- Netscape는 ECMA 국제 표준화 기구에 자바스크립트 표준 제정을 요청

### ECMAScript 탄생 (1997)
- **ECMAScript** = Ecma International이 정의한 "표준화된 스크립트 프로그래밍 언어 **명세**"
- 코드가 아닌 글(영어 문서)로 작성된 규칙·세부사항 설명서
- JavaScript는 이 명세를 구체적으로 **구현한** 프로그래밍 언어

| 구분 | 설명 |
|------|------|
| ECMAScript | 스크립트 언어가 준수해야 할 규칙/명세 (설계도) |
| JavaScript | ECMAScript 표준을 구현한 구체적인 언어 |

- ECMAScript 4판은 10년 동안 기업 간 합의 실패로 **폐기(무승부)**
  - Mozilla/Adobe: "강력하게 개발하자 (Java 수준으로)"
  - Microsoft/Yahoo: "가볍게 가자"
  - 결론: 타협 실패 → 4판 폐기

### 2차 브라우저 전쟁 (2004 ~ 2017)
- 독점 상태에서 IE는 웹 표준을 지키지 않고 독자 규격 유지 (ActiveX 등)
- Brendan Eich가 Netscape에서 나와 Mozilla 재단 설립 → **Firefox** 출시, 웹 표준 준수로 점유율 확대
- 2008년 Google **Chrome** 등장, 출시 3년 만에 Firefox, IE 점유율 모두 추월

### Chrome이 승리한 이유
Chrome의 성공 요인은 속도·보안·Google 생태계 통합 등 여러 가지가 있지만, 가장 결정적인 이유는 **적극적인 웹 표준 준수**였다.

- 웹 표준 준수 → 브라우저 간 일관된 동작, 다양한 플랫폼 지원
- 강력한 **개발자 도구(F12)** 제공
- Google이 자체 개발한 **V8 JavaScript 엔진** 도입으로 압도적 성능

---

## 3. V8 JavaScript 엔진

Chrome이 빠른 이유의 핵심이자, **Node.js의 엔진**이기도 한 V8의 작동 방식:

- 일반 인터프리터: 코드를 한 줄씩 해석하며 실행
- V8 **JIT(Just-In-Time) 컴파일**: 처음에는 인터프리터처럼 한 줄씩 읽다가, 자주 실행되는 코드는 미리 번역(컴파일)해 저장 → 반복 실행 시 해석 없이 바로 실행 → **속도 향상**
- **가비지 컬렉터** 내장: 사용하지 않는 메모리를 자동으로 정리

> V8 엔진 덕분에 JavaScript가 브라우저를 넘어 서버(Node.js)에서도 돌아가게 됐다.

---

## 4. ECMAScript 주요 버전

| 버전 | 연도 | 주요 내용 |
|------|------|-----------|
| ES5 | 2009 | 안정성과 생산성을 크게 높인 버전 |
| **ES6 (ES2015)** | **2015** | **가장 중요한 버전** — let, const, 화살표 함수, 클래스 등 현대 JS의 기반 |
| ES8 (ES2017) | 2017 | async/await 도입 (비동기 처리 혁신) |
| ES11 (ES2020) | 2020 | Optional chaining 등 |

> **ES6 이후의 문법**을 기준으로 JS를 배우는 것이 현재 표준. 이전 버전 문법(var 등)은 레거시로 취급.

---

## 5. JavaScript의 현재와 미래

- 현재는 Chrome, Firefox, Safari, Edge 등이 경쟁하며 모두 웹 표준을 준수
- 초기에는 브라우저 내 동적 기능 구현 목적이었으나, **Node.js(2009)** 등장으로 서버 사이드 개발에도 사용
- Discord, Slack, Notion, VS Code 모두 JavaScript(TypeScript) 기반
- **TypeScript** = JavaScript를 더 타이트하게 관리하는 상위 언어, 실무에서 사실상 필수

---

## 💡 한 줄 요약

> JavaScript는 10일 만에 만들어진 언어지만, 브라우저 전쟁과 표준화 과정을 거쳐 웹 프론트부터 서버까지 아우르는 생태계의 핵심 언어로 자리잡았다.

---

## ❓ 더 찾아볼 것

- ECMAScript 공식 명세 사이트 (https://ecma-international.org)
- V8 엔진 동작 원리 (JIT 컴파일, Hidden Class)
- TypeScript vs JavaScript 차이점
- Linus Torvalds — Linux, Git 창시자
- `null`의 타입이 `object`인 이유 (JS 역사적 버그)
