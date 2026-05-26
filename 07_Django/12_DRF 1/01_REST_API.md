# [Django] REST API

---

## 1. API란?

**API (Application Programming Interface)**는 두 소프트웨어가 서로 통신할 수 있게 해주는 메커니즘이다.

> 냉장고에 전기를 공급할 때 직접 배선하지 않고, 플러그를 콘센트에 꽂기만 하면 된다.  
> API도 마찬가지로, 복잡한 코드를 추상화해 단순한 구문으로 제공한다.

- 서버 쪽에서 이미 복잡한 코드를 구현해두고, 클라이언트는 정해진 방식으로 요청만 보낸다
- API를 사용할 땐 반드시 **API 문서(Documentation)**를 확인해야 함
  - 어떻게 요청을 보내야 하는지, 응답이 어떻게 오는지 정의되어 있기 때문

### Web API와 Open API

- **Web API**: 웹 서버 또는 웹 브라우저를 위한 API
- 현대 개발에서는 0부터 직접 개발하기보다 **Open API**를 활용
- **Third Party**: 직접 개발하지 않은 외부의 서비스 (생산자-소비자 생태계 밖에 존재)
  - 예: Youtube API, Google Map API, Naver Papago API, Kakao Map API

---

## 2. REST와 RESTful API

### REST (Representational State Transfer)

> API Server를 개발하기 위한 일종의 **소프트웨어 설계 방법론**  
> 엄격한 규칙이 아니라 "이렇게 해보는 게 어때?" 수준의 약속

- API마다 구조가 제각각이면 사용자가 쓸 때마다 완전히 다른 구조를 이해해야 함
- REST는 그 구조를 어느 정도 표준화해서 **누구나 예측 가능한 방식으로 통신**하도록 제안한 것

```
집을 지을 때 기초-골조-내장-마감 순서가 있듯이,
API도 구조를 어느 정도 맞춰보자!
```

- REST 원리를 따르는 시스템을 **RESTful 하다**고 표현
- 안 지켜도 API는 만들 수 있지만, 업계 표준처럼 자리 잡았기 때문에 통상적으로 지킴

### RESTful API

REST 설계 방법론을 지켜 구현한 API. "**자원을 정의**하고 **자원에 대한 주소를 지정**하는 전반적인 방법"

REST에서 데이터는 **자원(Resource)**이라고 부르며, 자원을 다루는 방법이 크게 3가지다.

| 구성 요소 | 방법 | 예시 |
|---|---|---|
| 자원의 식별 | URI (URL) | `/api/v1/articles/` |
| 자원의 행위 | HTTP Methods | `GET`, `POST`, `PUT`, `DELETE` |
| 자원의 표현 | JSON 데이터 | `{"id": 1, "title": "..."}` |

---

## 3. 자원의 식별 — URI와 URL

### URI vs URL

- **URI (Uniform Resource Identifier)**: 인터넷에서 자원을 식별하는 문자열의 **모든 방법**
- **URL (Uniform Resource Locator)**: URI의 하위 개념으로, 가장 일반적인 형태인 **웹 주소**

### URL 구조

```
http:// www.example.com : 80 /path/to/myfile.html ?key1=value1 #SomeAnchor
  ↑           ↑           ↑         ↑                 ↑            ↑
Scheme   Domain Name    Port      Path            Parameters     Anchor
```

| 구성 요소 | 설명 |
|---|---|
| **Scheme** | 브라우저가 자원을 요청하는 데 사용하는 규약. 웹은 `http(s)`, 메일은 `mailto:`, 파일전송은 `ftp:` |
| **Domain Name** | 요청 중인 웹 서버. IP 주소를 기억하기 어려우므로 도메인 이름을 사용 (예: `google.com` = `142.251.42.142`) |
| **Port** | 서버의 자원에 접근하는 기술적인 문(Gate). HTTP=80, HTTPS=443. 표준 포트는 생략 가능. 장고는 8000번 포트 사용 |
| **Path** | 웹 서버의 자원 경로. 예전엔 실제 파일 위치였지만, 오늘날은 **추상화된 구조**를 표현 (`/articles/create/`가 실제 폴더 구조가 아님) |
| **Parameters** | 웹 서버에 제공하는 추가 데이터. `?`로 시작하며 `key=value` 쌍을 `&`로 구분 |
| **Anchor** | 페이지의 특정 지점(북마크) 역할. `#` 이후 부분은 **서버로 전송되지 않고** 브라우저가 해당 지점으로 이동하는 데 사용 |

---

## 4. 자원의 행위 — HTTP Methods

HTTP Request Methods는 리소스에 대해 **수행하고자 하는 동작**을 정의한다.

> 주소로 행위를 표현하는 게 아니라, **메서드로 행위를 구분**하는 것이 REST 방식

과거 장고에서는 GET(조회)과 POST(나머지 전부)만 사용했으나, REST 방법론에서는 각 역할을 명확히 분리한다.

| Method | CRUD | 설명 |
|---|---|---|
| **GET** | Read | 서버에 자원의 표현을 요청. 데이터만 검색 |
| **POST** | Create | 데이터를 지정된 리소스에 제출. 서버 상태 변경 |
| **PUT** | Update | 요청한 주소의 자원을 전체 수정 |
| **DELETE** | Delete | 지정된 리소스를 삭제 |

> PATCH도 있음. PUT은 전체 수정, PATCH는 일부(부분) 수정

### HTTP Response Status Codes

서버가 요청 처리 결과를 숫자 코드로 알려줌.

| 범위 | 분류 | 의미 |
|---|---|---|
| 100-199 | Informational | 요청을 계속 진행 중 |
| 200-299 | Successful | 요청이 정상 처리됨 |
| 300-399 | Redirection | 리소스가 다른 위치로 이동 (장고 redirect 시 반환) |
| 400-499 | Client Error | **클라이언트 잘못**으로 인한 오류 |
| 500-599 | Server Error | **서버 잘못**으로 인한 오류 |

---

## 5. 자원의 표현 — JSON 응답

### Django 서버 역할의 변화

| 과거 (MTV 풀스택) | 현재 (Back-end API 서버) |
|---|---|
| 클라이언트 요청 → 서버가 HTML 렌더링 후 응답 | 클라이언트 요청 → 서버가 JSON 데이터만 응답 |
| 템플릿(Template) 존재 | 템플릿 없음 |
| 좋아요/팔로우 클릭 시 전체 페이지 새로고침 | 필요한 데이터만 받아 일부 업데이트 가능 |

- JSON은 **데이터만을 전달하기 위한 최소한의 형식**
- 어떤 클라이언트(언어, 플랫폼)와도 독립적으로 통신 가능
- HTML 대신 JSON만 전달하므로 **응답 용량 ↓, 처리 속도 ↑**
- 화면 구성은 **Vue 같은 Front-end 프레임워크**가 담당

### JSON 데이터 응답 실습 (99번 프로젝트)

```bash
# 가상 환경 생성 및 활성화
$ python -m venv venv
$ source venv/Scripts/activate

# 패키지 설치 및 마이그레이트
$ pip install -r requirements.txt
$ python manage.py migrate

# 초기 데이터 로드
$ python manage.py loaddata articles.json
# Installed 20 object(s) from 1 fixture(s)

# 서버 실행
$ python manage.py runserver
```

```python
# python-request-sample.py 예시
import requests
from pprint import pprint

response = requests.get('http://127.0.0.1:8000/api/v1/articles/')

# JSON을 Python 타입으로 변환
result = response.json()  # list 타입 반환

print(type(result))   # <class 'list'>
pprint(result)        # 전체 게시글 목록
pprint(result[0])     # 첫 번째 게시글 (dict 타입)
pprint(result[0].get('title'))  # 첫 번째 게시글 제목
```

> JSON 덩어리(문자열)를 `response.json()`으로 파이썬 타입(list 안의 dict)으로 형변환해서 활용

### Postman

**API 개발 및 테스트를 위한 도구**

- 설치: https://www.postman.com/downloads/
- 구성:
  - 좌측 상단 드롭다운: **HTTP 메서드 선택** (GET, POST, PUT, DELETE 등)
  - 상단 입력창: **요청 URL 작성**
  - 중간 탭: **요청 시 필요한 데이터 작성** (Params, Body 등)
  - 하단: **응답 결과 확인**

---

## 💡 한 줄 요약

> REST API는 URI(주소)로 자원을 식별하고, HTTP 메서드로 행위를 정의하며, JSON으로 자원을 표현하는 API 설계 방법론이다.

## ❓ 더 찾아볼 것

- PATCH vs PUT의 실제 사용 기준
- REST 6가지 제약 조건 (Stateless, Uniform Interface 등)
- HTTP Status Code 세부 목록 (201, 204, 401, 403, 404 등)
- JSON vs XML 비교
