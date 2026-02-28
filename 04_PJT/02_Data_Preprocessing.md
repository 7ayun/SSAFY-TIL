# [PJT] 데이터 전처리 및 외부 서비스 연동 (Data Preprocessing & API Integration)

> **핵심 키워드:** #API #JSON #MCP #HTTP_Request #Data_Parsing #Environment_Variable

---

## 🎯 학습 목표
* API(Application Programming Interface)의 개념 이해 및 소프트웨어 간 통신 원리 습득
* JSON 형식의 데이터 구조 파악 및 파이썬 데이터 타입과의 상호 변환 능력 배양
* `requests` 라이브러리를 활용한 외부 서버 데이터 요청 및 응답 처리 숙달
* MCP(Model Context Protocol) 표준화를 통한 LLM과 외부 도구 간 효율적 연결 구조 파악

---

## 💡 주요 개념 정리

### 1. API 및 통신 규격
* **인터페이스(Interface):** 서로 다른 시스템 사이에서 정보나 신호를 주고받는 접점
* **엔드포인트(Endpoint):** 데이터 요청을 위해 서버에서 지정한 특정 주소
* **쿼리 파라미터(Query Parameter):** 엔드포인트 주소 뒤 물음표(?) 이후에 작성하는 세부 질문(조건) 내용

### 2. 데이터 교환 표준 (JSON)
* **정의:** 키(Key)-값(Value) 쌍으로 이루어진 텍스트 기반 데이터 포맷
* **특징:** 언어와 무관하게 사용 가능한 범용 텍스트 형식이므로 통신 시 반드시 문자열로 전달
* **변환:** `loads` (문자열 → 딕셔너리), `dumps` (딕셔너리 → 문자열) 메서드 활용

### 3. 차세대 연결 표준 (MCP)
* **프로토콜 표준화:** 각 서비스마다 다른 API 규격을 LLM 중심의 공통 약속으로 통합하는 규약
* **구성 요소:** 기능을 제공하는 **Server**, 사용자 의도를 해석하는 **Host**, 통신을 담당하는 **Client**
* **확장성:** 개별 API 개발 없이 표준 규약 준수만으로 수많은 외부 도구(Notion, GitHub 등)와의 즉각적인 연동 가능

---

## 💻 기능 구현 및 코드 실습

### 1. JSON 데이터 파싱 및 가공
외부에서 유입된 텍스트 데이터를 파이썬 객체로 변환하여 조작하는 기법

```python
import json

# 1. JSON 문자열을 파이썬 딕셔너리로 변환 (Deserialization)
json_text = '{"title": "Python Guide", "version": 3.12}'
data = json.loads(json_text)
print(data.get("title")) # 'Python Guide' 접근 가능

# 2. 파이썬 데이터를 JSON 문자열로 변환 (Serialization)
new_data = {"status": "success", "code": 200}
json_string = json.dumps(new_data)
```

### 2. Requests 라이브러리 활용 API 호출
엔드포인트와 파라미터를 조합하여 외부 데이터를 수집하는 프로세스

```python
import requests

# 외부 API 엔드포인트 및 요청 조건 설정
url = "https://dog.ceo/api/breeds/image/random"
params = {"breed": "poodle"} # 예시 파라미터

# 데이터 요청 및 JSON 응답 파싱
response = requests.get(url, params=params).json()

# 가공된 데이터 추출
image_url = response.get("message")
print(f"수집된 이미지 주소: {image_url}")
```

### 3. 보안 관리 (Environment Variables)
API 키와 같은 민감 정보를 코드에서 분리하여 안전하게 관리하는 기법

```python
import os
from dotenv import load_dotenv

# .env 파일 내 설정값 로드
load_dotenv()
api_key = os.getenv("OPENAI_API_KEY")

# API 호출 시 키 포함 전달
# client = OpenAI(api_key=api_key)
```

---

## 🚀 복습 및 AI 인사이트
* **데이터 무결성:** 서로 다른 언어 간 통신 시 데이터 타입 불일치를 방지하기 위한 JSON 변환 과정의 필수성
* **공식 문서 활용:** API 규격은 수시로 갱신되므로 AI에 의존하기보다 서비스 제공처의 공식 Document를 직접 확인하는 습관 강조
* **최신 트렌드 대응:** 단일 API 연동을 넘어 MCP와 같은 표준 프로토콜을 통해 LLM 에이전트가 직접 파일 시스템이나 DB를 제어하는 자동화 환경 이해
* **보안 준수:** API 키 유출 방지를 위한 `.env` 활용 및 `.gitignore` 설정의 철저한 수행 권장