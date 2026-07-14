# [데이터 기초] Beautiful Soup의 활용

---

## 1. BeautifulSoup이란?

**BeautifulSoup**은 HTML이나 XML 문서를 구문 분석(parsing)하기 위한 파이썬 패키지다. 웹 브라우저가 HTML을 해석하는 것과 비슷하게, HTML 소스를 **트리 구조**로 해석해 원하는 요소에 직접 접근할 수 있게 해주는 **파서(Parser)** 역할을 한다.

---

## 2. 기본 사용법

서울대학교 식단 페이지를 예시로 데이터를 가져오는 흐름은 다음과 같다.

```python
import requests
from bs4 import BeautifulSoup

url = "https://example.com/menu"

# User-Agent를 선언하지 않으면 크롤러로 인식되어 차단될 수 있음
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) ..."
}

response = requests.get(url, headers=headers)
bs = BeautifulSoup(response.text, "html.parser")
```

**User-Agent 헤더가 필요한 이유**: 사이트에서 자동 수집 도구(로봇/크롤러)로 판단되면 접근 자체를 차단하는 경우가 있다. 헤더에 일반 브라우저처럼 보이는 User-Agent 값을 넣어주면, 실제 사람이 접속한 것처럼 보이게 해 차단을 피할 수 있다.

- `requests.get(url)`로 응답 데이터를 받아온 뒤, 텍스트로 변환해 BeautifulSoup 객체로 선언하면 파싱할 준비가 끝난 것이다.

---

## 3. 데이터 추출하기

### 타이틀 바로 가져오기

```python
print(bs.title)
# <title>식단 - 서울대학교 생활협동조합</title>
```

겉보기엔 별다른 변화가 없어 보여도, 이미 `bs` 객체 안에는 HTML 정보가 BeautifulSoup을 통해 해석되어 있어 `.title`처럼 바로 접근할 수 있다.

### 요소 검사(F12)로 위치 파악하기

원하는 데이터(예: 학생회관식당 점심 메뉴)가 HTML 코드 상 어디에 있는지 알기 위해 **개발자 도구(F12) → 요소 검사**를 사용한다. 이렇게 확인해보면 점심 메뉴가 `<td class="lunch">` 태그 내부의 텍스트로 구성되어 있고, 메뉴명·가격·운영시간이 별도 태그로 분리되어 있지 않다는 걸 알 수 있다.

### select() / select_one() — CSS 셀렉터 기반 탐색

```python
# 기본형: 태그 찾기
bs.select("td.lunch")

# 첫 번째 항목만 가져오기
bs.select_one("td.lunch")
```

- `select()`는 조건에 맞는 **모든 요소를 리스트로 반환**
- `select_one()`은 **첫 번째 요소 하나만 반환**
- `td.lunch` → `<td>` 태그 중 class가 `lunch`인 것을 CSS 셀렉터 문법으로 탐색

실제로 실행하면 다음과 같이 여러 `<td class="lunch">` 요소가 리스트로 반환된다.

```python
bs.select('td.lunch')
# [<td class="lunch">점심</td>,
#  <td class="lunch">매콤어묵김밥 : 3,000원<br/><br/>※ 운영시간 : 11:00~14:00</td>,
#  <td class="lunch">한우사골떡만두국 : 13,000원<br/>...</td>, ...]
```

메뉴명 외에 운영시간 등 부가 정보까지 같이 딸려오기 때문에, 이후 `.text`로 텍스트만 뽑아내고 `strip()`, `split()` 등으로 후처리해 원하는 형태로 다듬는다.

---

## 4. select 셀렉터 문법 정리

| 문법 | 의미 |
|---|---|
| `.select('태그명')` | 해당 태그 전체 선택 |
| `.select('태그명.class')` | 특정 태그 + 특정 class 조합 선택 |
| `.select('태그명.class.subclass')` | 태그 + 여러 class 조합 선택 |
| `.select('상위태그 > 하위태그')` | 자식 선택자 (직계 자식만) |
| `.select('상위태그.class > 하위태그.class')` | 클래스 조건이 걸린 자식 선택자 |
| `.select('.class')` | class만으로 선택 |
| `.select('#id')` | id로 선택 (문서 내 유일) |
| `.select('태그명[속성=값]')` | 특정 속성값을 가진 태그 선택 |

**핵심은 결국 HTML 구조를 보고, 원하는 요소를 구분하는 기준이 태그인지·클래스인지·아이디인지 판단한 다음 그에 맞는 선택자를 쓰는 것**이다. 예를 들어 문서 내 테이블이 하나뿐이라면 `id`만으로 충분히 특정할 수 있고, 여러 개 중 특정 조건에 맞는 것들만 골라야 한다면 `class`나 `tag.class` 조합이 유용하다.

---

## 5. 텍스트 후처리

```python
lunch_texts = bs.select('td.lunch')
first_menu = lunch_texts[0].text  # 요소의 텍스트만 추출

# 후처리 예시
first_menu = first_menu.strip()          # 공백 제거
parts = first_menu.split('\n')           # 줄바꿈 기준 분리
```

BeautifulSoup으로 가져온 원본 텍스트는 운영시간처럼 불필요한 정보가 섞여 있는 경우가 많다. 이때는 `.text` 속성으로 텍스트만 뽑아낸 뒤, 파이썬 문자열 메서드(`strip()`, `split()` 등)로 원하는 형태까지 다듬는 후처리 과정을 거친다.

---

## 💡 한 줄 요약
> BeautifulSoup은 requests로 받아온 HTML을 트리 구조로 파싱해, CSS 셀렉터 기반의 select/select_one으로 원하는 요소를 찾고 텍스트를 후처리해 원하는 데이터를 추출하는 도구다.

## ❓ 더 찾아볼 것
- `find()` / `find_all()`과 `select()` / `select_one()`의 차이
- html.parser 외 lxml, html5lib 등 다른 파서의 차이
- 정규표현식(re)을 활용한 텍스트 후처리 패턴
