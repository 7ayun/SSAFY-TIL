# [데이터 기초] Selenium의 활용

---

## 1. Selenium이란?

**Selenium**은 BeautifulSoup과 역할이 다르다. BeautifulSoup이 "HTML을 받아서 분석하는" 도구라면, Selenium은 **브라우저 자체를 조종하는 도구**다.

실제 서비스 대부분은 자바스크립트로 화면을 그리는 **동적 페이지**라서, 크롤링 관점에서 HTML만 가져오는 것으로는 한계가 있다. 이럴 때 브라우저를 직접 조작해 데이터를 가져오기 위해 Selenium을 사용한다.

- 웹 데이터 수집뿐 아니라 **반복 작업 자동화**, 웹 애플리케이션 테스트에도 활용 가능
- 주소창에 URL 입력, 뒤로 가기/새로고침, 버튼 클릭, 텍스트 입력, 화면 스크롤, 화면 캡처 등을 코드로 제어
- Chrome, Firefox(Gecko), Safari, Edge, Internet Explorer 등 여러 브라우저의 WebDriver를 지원

---

## 2. 핵심 개념

| 개념 | 설명 |
|---|---|
| **WebDriver** | 실제 브라우저와 연결해 조작하는 컨트롤러 역할의 객체 |
| **Element(요소)** | 화면의 버튼, 입력창, 텍스트 등 조작 대상이 되는 개별 요소 |
| **Locator** | 특정 요소를 찾기 위한 방식/기준 (HTML을 분석하는 게 아니라 **렌더링된 화면**에서 요소를 찾음) |
| **Action** | 찾은 요소에 대해 실제로 취하는 행동 (클릭, 입력 등) |

Selenium이 하는 일을 요약하면: 요소(Element)를 정확히 찾고, 적절한 타이밍에 행동(Action)을 주는 것이다.

### Action의 두 가지 대상

| 대상 | 설명 | 예시 |
|---|---|---|
| **Element 대상** | 찾은 요소에 대한 조작 | 클릭, 텍스트 입력, 입력값 지우기, submit(폼 전송) |
| **Browser 대상** | 브라우저 자체의 상태 변경 | URL 이동, 앞/뒤 페이지 이동, 새로고침, 브라우저(탭) 종료 |

---

## 3. 기본 조작

```python
from selenium import webdriver
import chromedriver_autoinstaller

chromedriver_autoinstaller.install()
driver = webdriver.Chrome()

# 페이지 이동 (주소창에 URL 입력하고 접속하는 것과 동일)
driver.get("https://www.naver.com")

# 새로고침 (F5와 동일한 동작)
driver.refresh()
# 현재 주소 확인
print(driver.current_url)

# 페이지 뒤로 가기
driver.back()
print(driver.current_url)
```

- `driver`는 브라우저를 조종하는 컨트롤러 역할
- `driver.get()`으로 페이지 로딩이 완료되면 다음 코드가 실행됨
- 작업이 끝나면 `driver.quit()`으로 종료 — 종료하면 더 이상 창을 띄울 수 없음

---

## 4. 대기(Wait) 처리하기

Selenium을 쓰다 보면 가장 흔히 만나는 에러가 **"요소를 못 찾았다"**는 에러다. 실제로 요소가 없어서가 아니라, 웹 페이지가 실행과 동시에 한 번에 다 뜨는 게 아니기 때문에 발생하는 경우가 많다.

### WebDriverWait vs time.sleep()

| 방식 | 동작 | 문제점 |
|---|---|---|
| `time.sleep(n)` | 무조건 n초 대기 | 조건 충족 여부와 무관하게 항상 대기 → 너무 짧으면 에러, 너무 길면 비효율 |
| `WebDriverWait` | 최대 n초까지 **조건이 만족될 때까지** 대기, 만족되면 즉시 다음 코드 실행 | 타임아웃 시 `TimeoutException` 발생 |

```python
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By

wait = WebDriverWait(driver, 10)  # 최대 10초 대기

# 예: id가 'query'인 요소가 뜰 때까지 대기
element = wait.until(
    EC.presence_of_element_located((By.ID, "query"))
)
```

**Expected Conditions(EC)**로 "무엇을 기준으로 기다릴지" 조건을 지정할 수 있다 — 요소가 DOM에 존재할 때, 화면에 보일 때, 클릭 가능해질 때, 모든 요소가 뜰 때, URL에 특정 문자열이 포함될 때 등 다양한 조건 메서드가 제공된다.

---

## 5. Locator 종류

| Locator | 설명 |
|---|---|
| `By.TAG_NAME` | HTML 태그 이름으로 요소를 찾음 (예: p, div, a 전체) |
| `By.ID` | id 속성값으로 찾음 — 페이지 내 유일하므로 가장 정확 |
| `By.CLASS_NAME` | class 속성값으로 찾음 — 같은 class가 여러 개 있을 수 있어 보통 여러 개를 가져올 때 사용 |
| `By.CSS_SELECTOR` | CSS 문법으로 요소를 찾음 — id·class·태그를 자유롭게 조합 가능, **가장 많이 쓰임** |
| `By.XPATH` | HTML 구조(위치)를 기준으로 문서를 따라 내려가며 찾음 |
| `By.NAME` | name 속성값으로 찾음 — 주로 form, input 태그에서 사용 |
| `By.LINK_TEXT` | `<a>` 태그 안의 전체 텍스트로 찾음 |
| `By.PARTIAL_LINK_TEXT` | `<a>` 태그 텍스트의 일부만으로 찾음 |

실무에서는 TAG_NAME / ID / CLASS_NAME / CSS_SELECTOR 정도만 잘 알아도 대부분의 상황을 해결할 수 있다.

---

## 6. 실습 흐름 — 네이버 검색 → 뉴스 기사 수집

```python
# 검색창 요소를 찾아 대기 후 텍스트 입력
query = wait.until(EC.presence_of_element_located((By.ID, "query")))
query.send_keys("파이썬")
query.send_keys(Keys.ENTER)

# 뉴스 탭 클릭 (링크 텍스트가 "뉴스"인 요소가 클릭 가능해질 때까지 대기)
news_tab = wait.until(
    EC.element_to_be_clickable((By.LINK_TEXT, "뉴스"))
)
news_tab.click()

# 모든 <a> 태그를 가져온 뒤 조건으로 필터링
anchors = wait.until(
    EC.presence_of_all_elements_located((By.TAG_NAME, "a"))
)

articles = []
for a in anchors:
    title = (a.text or "").strip()
    link = a.get_attribute("href")

    if not title or not link:
        continue

    # 실제 뉴스 기사 URL만 허용 (광고, 메뉴, 프로모션 페이지 제외)
    if "n.news.naver.com" in link and "/article/" in link:
        articles.append((title, link))
```

- 검색창에 "파이썬" 입력 후 Enter → 뉴스 탭 클릭 → 이동한 페이지에서 `<a>` 태그를 모두 수집
- 광고나 프로모션 페이지가 섞이지 않도록 실제 뉴스 기사 URL 패턴(`n.news.naver.com`, `/article/`)으로 필터링하는 방어 로직을 추가
- 이후 첫 번째 기사로 이동해 CSS 셀렉터(`h2#title_area` 등)로 제목·본문 요소를 추출
- 데이터 수집이 끝나면 `driver.quit()`으로 브라우저 종료

---

## 💡 한 줄 요약
> Selenium은 WebDriver로 브라우저를 직접 제어해 동적 페이지의 요소를 Locator로 찾고 Action으로 상호작용하며, WebDriverWait으로 로딩 타이밍 문제를 안정적으로 처리하는 자동화 도구다.

## ❓ 더 찾아볼 것
- 암시적 대기(Implicit Wait)와 명시적 대기(Explicit Wait)의 차이
- Selenium 헤드리스(headless) 모드로 브라우저 창 없이 실행하는 방법
- BeautifulSoup과 Selenium을 함께 사용하는 경우 (driver.page_source를 BeautifulSoup으로 파싱)
