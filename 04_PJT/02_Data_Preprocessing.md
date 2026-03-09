# [PJT] 데이터 전처리 — API, JSON, requests, MCP

> **핵심 키워드:** #API #ApplicationProgrammingInterface #엔드포인트 #쿼리파라미터 #JSON #딕셔너리 #문자열변환 #json_load #json_dump #requests #OAuth #소셜로그인 #GTTS #TextToSpeech #MCP #ModelContextProtocol #프로토콜 #표준화 #LLM #엔트로픽

---

## 학습 목표

* API의 개념(서로 다른 소프트웨어 간 통신 인터페이스)을 이해하고 실생활 예시로 설명
* JSON의 본질(키-값 구조의 텍스트)을 파악하고, 파이썬 딕셔너리와의 변환 원리 이해
* requests 라이브러리로 외부 API를 호출하고 응답 데이터를 활용
* MCP(Model Context Protocol)의 등장 배경과 표준화 의의를 파악

---

## 1. API란

### 1-1. 개념

API(Application Programming Interface)는 서로 다른 소프트웨어가 **가이드에 맞춰서 요청하고 응답**할 수 있도록 만든 통신 규격이다.

TV와 사람 사이에 리모컨이 필요하듯, 서로 다른 시스템 사이에도 소통을 위한 인터페이스가 필요하다. 음식점에 비유하면 API 문서는 메뉴판이고, 고객의 주문은 요청, 요리사의 음식은 응답이다. 메뉴판에 없는 음식을 주문하면 응답할 수 없다.

API 문서를 만드는 주체는 **데이터를 제공하는 쪽**(기상청, 구글, 알라딘 등)이며, 사용하는 쪽은 그 문서의 규격에 맞춰 데이터를 요청한다.

### 1-2. 소셜 로그인 (OAuth)

구글이나 카카오 계정으로 다른 사이트에 로그인하는 기능을 OAuth(Open Authentication)라고 한다. 동작 원리를 ChatGPT + 구글 로그인으로 보면 다음과 같다.

사용자가 "구글로 계속하기"를 누르면 ChatGPT가 구글 API(리모컨)를 사용자에게 건넨다. 사용자가 구글 API를 통해 아이디·비밀번호를 입력하면, 그 정보는 ChatGPT가 아닌 구글로 전달된다. 구글이 인증에 성공하면 사용자 정보를 응답으로 돌려주고, ChatGPT는 그 응답을 받아 회원가입·로그인을 자동 처리한다.

### 1-3. API 키

공개 API는 악용을 방지하기 위해 **키(key)** 를 발급하여 사용자를 인증한다. 키 없이는 데이터를 받을 수 없으며, 보통 분당 요청 횟수 제한이 함께 적용된다.

> **강사님 강조**: API 키는 유출되면 요금이 부과되거나 서비스가 차단될 수 있다. .env 파일에 보관하고 .gitignore에 추가하는 것이 기본이다.

---

## 2. JSON

### 2-1. JSON이란

JSON(JavaScript Object Notation)은 키-값 쌍 구조를 가지는 **텍스트 형식**의 데이터 포맷이다. 파이썬 딕셔너리와 생김새가 같지만, JSON은 엄연히 문자열이라는 점이 핵심이다.

서로 다른 프로그래밍 언어(파이썬, 자바스크립트, 자바 등)는 각자 키-값 자료형의 이름과 구현이 다르다. 따라서 언어에 상관없이 주고받을 수 있는 공통 형식이 필요한데, 그것이 JSON(텍스트)이다.

```
파이썬 딕셔너리 (객체)  ←→  JSON (텍스트)  ←→  다른 언어의 자료형
```

### 2-2. JSON → 파이썬 딕셔너리 (loads / load)

```python
import json

# 문자열 형태의 JSON → 딕셔너리로 변환
json_text = '{"name": "Alice", "age": 25}'
data = json.loads(json_text)       # loads: 문자열(string)을 변환
print(data["name"])                # Alice

# 파일에서 직접 읽어서 변환
with open("data.json", "r", encoding="utf-8") as f:
    data = json.load(f)            # load: 파일 객체를 변환
```

`loads`의 `s`는 string을 의미한다. 문자열을 변환할 때는 `loads`, 파일을 직접 열어 변환할 때는 `load`를 사용한다.

### 2-3. 파이썬 딕셔너리 → JSON (dumps / dump)

```python
import json

sample = {"title": "Python", "version": 3}

# 딕셔너리 → JSON 문자열
json_str = json.dumps(sample, ensure_ascii=False)

# 딕셔너리 → JSON 파일로 저장
with open("output.json", "w", encoding="utf-8") as f:
    json.dump(sample, f, ensure_ascii=False)
```

`dumps`는 딕셔너리를 문자열로, `dump`는 딕셔너리를 파일로 저장한다. `ensure_ascii=False`를 설정하면 한글이 유니코드 이스케이프 없이 그대로 저장된다.

> **강사님 강조**: JSON 변환은 앞으로 엄청 많이 쓰인다. 왜 쓰는지를 모르고 관성으로 쓰는 사람이 많으니, "서로 다른 앱은 텍스트로만 통신하고, 받은 텍스트를 딕셔너리로 바꿔서 사용한다"는 원리를 반드시 이해할 것.

---

## 3. API 실습 — requests

### 3-1. requests 라이브러리

`requests`는 파이썬에서 HTTP 요청을 보내는 외부 라이브러리다.

```bash
pip install requests
```

### 3-2. Dog API 예제

Dog API는 랜덤 강아지 사진 URL을 JSON으로 응답해주는 공개 API다.

```python
import requests

url = "https://dog.ceo/api/breeds/image/random"
response = requests.get(url)
data = response.json()          # JSON 응답 → 딕셔너리 변환

image_url = data.get("message")
print(image_url)                # 랜덤 강아지 이미지 URL
```

`response.json()`은 응답 본문의 JSON 텍스트를 파이썬 딕셔너리로 변환하는 메서드다.

### 3-3. 엔드포인트와 쿼리 파라미터

API 요청 URL은 **엔드포인트**(서버 주소)와 **쿼리 파라미터**(질의 조건)로 구성된다.

```
https://www.aladin.co.kr/ttb/api/ItemSearch.aspx?ttbkey=xxx&Query=Python
│                        엔드포인트                │  쿼리 파라미터  │
```

물음표(`?`) 이전이 엔드포인트, 이후가 쿼리 파라미터다. 여러 조건은 `&`로 연결한다. 구글 검색도 동일한 구조로, `google.com/search?q=검색어`에서 `q`가 쿼리 파라미터다.

requests에서는 params 딕셔너리로 쿼리를 전달하면 URL이 자동 조립된다.

```python
import requests

url = "https://www.aladin.co.kr/ttb/api/ItemSearch.aspx"
params = {
    "ttbkey": "발급받은_API_키",
    "Query": "파이썬",
    "QueryType": "Keyword",
    "output": "js"
}
response = requests.get(url, params=params)
data = response.json()
```

> **강사님 주의**: 쿼리 파라미터는 URL에 노출되므로 보안성이 낮다. 로그인 아이디·비밀번호 같은 민감 정보는 URL이 아닌 요청 본문(body)에 담아야 한다.

---

## 4. 참고 — gTTS (읽을거리)

gTTS(Google Text-to-Speech)는 텍스트를 음성 파일로 변환하는 라이브러리다. 구글 번역의 음성 엔진을 기반으로 동작한다.

```bash
pip install gtts
```

```python
from gtts import gTTS

text = "안녕하세요, 오늘 학습한 내용을 요약합니다."
tts = gTTS(text=text, lang="ko", slow=False)
tts.save("output.mp3")
```

시각장애인 지원, 오디오북, 도네이션 TTS 등 다양한 서비스에 활용할 수 있다. 직접 구현하는 것보다 만들어진 라이브러리를 잘 조합해서 좋은 서비스를 만드는 역량이 점점 중요해지는 시대다.

---

## 5. MCP (Model Context Protocol)

### 5-1. 문제 — API마다 규격이 다르다

GitHub, Notion, YouTube, Google Drive 등 각 서비스는 자체 방식으로 API를 개발한다. 서버에서 여러 서비스를 연결하려면 서비스마다 개별적으로 API 연동 코드를 작성해야 하므로, 한 번 개발한 코드를 재사용할 수 없다.

### 5-2. MCP — 표준화된 약속

MCP(Model Context Protocol)는 LLM(대형 언어 모델)에게 문맥(context)을 제공하는 방법을 **표준화한 프로토콜(약속)** 이다. 도구나 앱이 아니라 규약이며, "앞으로 기능을 제공할 때 이 형식을 따라달라"는 제안이다.

USB-C 타입에 비유하면 이해하기 쉽다. 과거에는 삼성은 C타입, 애플은 라이트닝, 모니터는 HDMI 등 각각 다른 포트를 사용했다. 결국 C타입이 표준으로 자리잡으면서 하나의 케이블로 충전·데이터 전송이 가능해졌다. MCP도 마찬가지로, 서비스마다 다른 API 규격을 하나의 표준 프로토콜로 통일하자는 것이다.

### 5-3. MCP의 등장과 확산

MCP는 2024년 11월 엔트로픽(Anthropic)이 공개했다. 초기에는 OpenAI가 시장을 지배하던 시기라 주목받지 못했지만, 2025년 2월 Cursor(AI 코딩 에디터)가 MCP를 도입하면서 급속히 확산되었다. 이후 OpenAI도 MCP를 도입하면서 사실상 업계 표준으로 자리잡았다.

### 5-4. MCP 서버 활용 예시

MCP 서버를 VS Code(Copilot)에 연결하면, 채팅 한 줄로 다양한 작업을 수행할 수 있다.

"이 경로에 무슨 파일들이 있어?"라고 입력하면 LLM이 파일 시스템 MCP 서버를 자동으로 호출하여 파일 목록을 반환한다. "각 이미지를 분석해서 파일 제목을 변경해줘"라고 입력하면 이미지 분석 MCP 서버와 파일 시스템 MCP 서버를 조합하여 자동 처리한다. 사용자가 어떤 MCP 서버를 쓰라고 지정하지 않아도, LLM이 작업 내용에 맞는 서버를 스스로 판단하여 호출하는 것이 핵심이다.

> **강사님 강조**: MCP는 시험에 안 나오고 모른다고 개발에 지장이 없지만, 매우 트렌디한 내용이다. 단순 파일명 변경에 국한하지 말고, 엑셀 파일 날짜별 분류, 데이터베이스 조회·수정, 노션 페이지 자동 생성 등으로 응용할 수 있다는 점을 이해할 것.

---

## 정리

| 구분 | 핵심 포인트 |
|------|-------------|
| API | 서로 다른 소프트웨어 간 통신 인터페이스, 데이터 제공자가 문서·규격을 만들고 사용자가 따름 |
| API 키 | 인증 수단, 악용 방지 + 요청 횟수 제한, .env로 관리하고 GitHub에 올리지 않음 |
| JSON | 키-값 구조의 텍스트, 서로 다른 언어 간 데이터 교환 형식 |
| json.loads / load | JSON 텍스트 → 파이썬 딕셔너리 변환 (s = string) |
| json.dumps / dump | 파이썬 딕셔너리 → JSON 텍스트 변환 |
| requests | `requests.get(url, params=...)` → `response.json()`으로 API 호출 및 응답 처리 |
| 엔드포인트 | API 서버 주소, 물음표(?) 이후가 쿼리 파라미터(질의 조건) |
| gTTS | 텍스트 → 음성 변환 라이브러리, 구글 번역 엔진 기반 |
| MCP | LLM에게 문맥을 제공하는 표준 프로토콜, USB-C처럼 API 규격을 통일하자는 약속 |
| MCP 활용 | LLM이 작업에 맞는 MCP 서버를 자동 선택·호출, 파일 관리·DB 조회 등 자동화 가능 |
