# [데이터 기초] Open API

---

## 1. Open API란?

**Open API**(Open Application Programming Interface)는 누구나 사용할 수 있도록 공개된 API를 말한다. 웹 사이트가 가진 기능을 외부에서도 이용할 수 있도록 공개한 프로그래밍 인터페이스로, 데이터 수집 관점에서 보면 **데이터를 수집할 수 있도록 제공하는 형태로 열어놓은 것**이라 볼 수 있다.

**대표 예시**: 네이버 지도, 구글 맵, 공공데이터 포털 등

### 동작 방식

대부분의 Open API는 HTTP 프로토콜의 GET, POST 등 메서드를 사용해 자원이나 서비스를 요청한다.

- 일반적인 웹 페이지에 GET 요청 → HTML 문서로 응답
- Open API에 GET 요청 → 정해진 형식(주로 **XML** 또는 **JSON**)의 텍스트로 응답

```
[사용자] → API Key로 요청 → [공공데이터 포털] → 오픈 API 호출 → [제공기관]
[사용자] ← API 결과 응답  ← [공공데이터 포털] ← API 회신        ← [제공기관]
```

공공데이터 포털처럼 데이터를 가진 서버(제공기관)와 사용자 사이에 **플랫폼**이 중개하는 이중 구조로 되어 있는 경우가 많다. 카카오 로그인, 네이버 로그인 같은 서비스도 이와 동일한 구조를 거친다.

---

## 2. 공공 데이터 포털 활용 절차

1. 포털 사이트에서 필요한 키워드로 검색 (예: "코로나19 확진자 성별 연령별 현황")
2. 원하는 데이터를 제공하는 Open API가 있는지 확인
3. 활용 목적을 작성하고 **활용신청** (검토 단계 없이 바로 신청 가능한 경우가 많음)
4. 마이페이지 > 오픈API > 인증키발급현황에서 **API Key** 확인

> **주의**: 동일한 인증키로 과도한 요청을 보내면 해당 키의 권한이 일시정지될 수 있다.

---

## 3. 요청 변수(Request Parameter)

API 상세페이지에는 요청 시 필요한 변수가 명시되어 있다. 필수 항목은 반드시 포함해야 하고, 옵션 항목은 생략 가능하다.

| 항목명(국문) | 항목명(영문) | 항목구분 | 샘플데이터 |
|---|---|---|---|
| 서비스키 | ServiceKey | 필수 | (발급받은 키) |
| 페이지 번호 | pageNo | 옵션 | 1 |
| 한 페이지 결과 수 | numOfRows | 옵션 | 10 |
| 데이터 생성일 시작범위 | startCreateDt | 옵션 | 20200310 |
| 데이터 생성일 종료범위 | endCreateDt | 옵션 | 20200414 |

**쿼리 파라미터 형식**: URL 뒤에 `?`로 시작해 파라미터를 붙이고, 여러 조건은 `&`로 연결한다.

```
http://openapi.data.go.kr/openapi/service/rest/Covid19/getCovid19GenAgeCaseInfJson
    ?ServiceKey={키}&pageNo=1&numOfRows=10&startCreateDt=20200310&endCreateDt=20200414
```

> API 키는 코드에 직접 선언하기보다 `.env` 파일 등으로 별도 관리하는 것이 좋다 (키 노출 방지).

---

## 4. XML 응답 파싱하기

Open API는 결과를 보통 XML 형태로 반환하며, XML도 HTML과 비슷한 문법(태그 기반 구조)으로 해석할 수 있다.

```xml
<response>
  <header>
    <resultCode>00</resultCode>
    <resultMsg>NORMAL SERVICE</resultMsg>
  </header>
  <body>
    <items>
      <item>
        <confCase>53179</confCase>
        <confCaseRate>8.05</confCaseRate>
        <death>3</death>
        <gubun>0-9</gubun>
        <createDt>2022-01-08</createDt>
      </item>
      <item>
        <confCase>98569</confCase>
        ...
      </item>
    </items>
  </body>
</response>
```

- 매 `<item>` 태그마다 하나의 통계 레코드가 들어있는 구조 (HTML의 `<tr>`과 비슷한 개념)
- `confCase`(확진자 수), `confCaseRate`(확진률), `gubun`(성/연령 구분) 등 API 설명 페이지의 변수명을 참고해 원하는 항목만 추출

### BeautifulSoup으로 XML 파싱

```python
import requests
from bs4 import BeautifulSoup

url = "http://openapi.data.go.kr/openapi/service/rest/Covid19/getCovid19GenAgeCaseInfJson"
params = {
    "ServiceKey": SERVICE_KEY,
    "pageNo": 1,
    "numOfRows": 10,
    "startCreateDt": "20220108",
    "endCreateDt": "20220108",
}

response = requests.get(url, params=params)
bs = BeautifulSoup(response.text, "xml")

items = bs.select("item")
for item in items:
    conf_case = item.select_one("confCase").text
    gubun = item.select_one("gubun").text
    print(gubun, conf_case)
```

- HTML을 다룰 때와 마찬가지로 `select()`를 사용해 `item` 태그를 모두 가져온 뒤, 각 항목에서 원하는 하위 태그를 다시 `select_one()`으로 추출
- 여러 날짜에 걸친 데이터가 필요하면 날짜 범위를 바꿔가며 요청을 반복(루프)해 여러 데이터를 순차적으로 수집
- 수집한 데이터는 이후 pandas 등으로 결측치·중복 확인, 원하는 형태로 가공해 분석·시각화·자동화 서비스 등에 활용

---

## 💡 한 줄 요약
> Open API는 기관이 HTTP 요청 방식으로 공개한 정형화된 데이터를 API Key 인증을 거쳐 요청하고, XML/JSON 형태의 응답을 파싱해 활용하는 데이터 수집 방법이다.

## ❓ 더 찾아볼 것
- JSON 형식 응답을 다룰 때 `response.json()`으로 바로 파싱하는 방법
- REST API의 GET/POST/PUT/DELETE 메서드별 용도 차이
- API 요청 실패(4xx, 5xx) 시 에러 핸들링과 재시도(retry) 전략
