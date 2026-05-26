# [Django] DRF with Single Model

---

## 1. Django REST Framework(DRF)란?

**Django에서 RESTful API 서버를 쉽게 구축할 수 있도록 도와주는 오픈소스 라이브러리**

> 가구를 직접 만들려면 톱, 망치, 설계도, 재료가 모두 필요하다.  
> 하지만 조립식 가구는 드라이버 하나만 있으면 완성할 수 있다.  
> DRF는 그 드라이버 세트와 같다.

- 복잡한 API 서버 개발 과정을 **표준화하고 자동화**한 함수·클래스·변수들을 제공
- 외부 라이브러리이므로 **설치 + 앱 등록** 두 단계 모두 필요

### 설치 및 등록

```bash
$ pip install djangorestframework
# 또는 requirements.txt 기반 설치
$ pip install -r requirements.txt
```

```python
# settings.py
INSTALLED_APPS = [
    ...
    'rest_framework',  # 반드시 앱으로 등록해야 함
]
```

> 외부 라이브러리 설치 시 공식 문서를 꼼꼼히 읽어야 함.  
> `pip install`만 하고 끝나는 게 아니라 **앱 등록까지 해야 하는** 라이브러리들이 있음.

### 프로젝트 준비 (drf 프로젝트)

```bash
# 가상 환경 생성 및 활성화
$ python -m venv venv
$ source venv/Scripts/activate
$ pip install -r requirements.txt

# 마이그레이트 및 초기 데이터 로드
$ python manage.py migrate
$ python manage.py loaddata articles.json
# Installed 20 object(s) from 1 fixture(s)
```

- DRF 프로젝트에는 **Template 없음, Forms 없음** → 순수한 API 서버
- Article 모델 구조: `title`, `content`, `created_at`, `updated_at`

---

## 2. Serialization과 Serializer

### Serialization (직렬화)

**여러 시스템에서 활용하기 위해, 데이터 구조나 객체 상태를 재구성할 수 있는 포맷으로 변환하는 과정**

```
원물 데이터(QuerySet)  →  Serialization  →  Serialized Data  →  JSON, Python, Java, C++...
```

- Django의 QuerySet은 Django에서만 알아들을 수 있는 타입
- 이걸 다른 서비스나 언어에서도 활용 가능하도록 **유연한 형태로 변환**하는 과정
- 직렬화된 데이터는 다른 프로그램, 다른 언어, 다른 컴퓨터에서도 다시 원래 구조로 복원 가능
- 즉, **"어디서든 읽고 사용할 수 있게 만드는 공통 언어로의 번역"**
- 수업에서는 JSON으로의 변환에 집중

### Serializer 클래스

**Serialization을 진행하여 Serialized Data를 반환해주는 클래스**

- `serializers.py` 파일을 생성하여 작성 (파일명은 관례상 이렇게 사용)
- 기능마다 별도의 Serializer 클래스를 하나씩 만드는 방식

| 구분 | 설명 |
|---|---|
| **Serializer** | 일반 직렬화 클래스. 데이터베이스와 무관한 데이터를 직렬화 |
| **ModelSerializer** | Django 모델과 연결된 클래스. 모델 필드에 맞춰 자동으로 직렬화 |

> Django에서 Form과 ModelForm의 관계와 동일한 구조!  
> DRF팀이 장고 개발자에게 친숙한 구조로 설계해준 배려

### ModelSerializer 사용 예시

```python
# articles/serializers.py
from rest_framework import serializers
from .models import Article


class ArticleSerializer(serializers.ModelSerializer):
    class Meta:
        model = Article
        fields = '__all__'
```

- `Meta` 클래스: ModelForm의 `Meta`와 동일한 역할
- `model`: 직렬화할 Django 모델 지정
- `fields`: 포함할 필드 지정 (`'__all__'`이면 전체 필드)
- `serializers.py`의 위치나 파일명은 자유롭게 작성 가능

---

## 3. Postman 활용

**Postman**: API 개발 및 테스트를 위한 서비스

- 요청 데이터 구성, 응답 확인, 환경 설정, 자동화 테스트 등 다양한 기능 제공
- Postman 화면 구성:

```
[메서드 선택 드롭다운]  [요청 URL 입력창]        [Send 버튼]
──────────────────────────────────────────────────────
[Params / Body / Authorization / Headers ...]  ← 요청 설정
──────────────────────────────────────────────────────
[응답 결과 (Body, Status Code, 응답 시간 등)]   ← 응답 확인
```

- Postman에서 요청을 보내면 RESTful한 요소 세 가지를 모두 확인 가능
  - **자원의 식별**: URL
  - **자원의 행위**: Method (GET, POST 등)
  - **자원의 표현**: JSON 형태의 응답 데이터

> DRF가 브라우저에서 보기 편한 페이지 형태로 JSON을 뿌려주지만,  
> 실제 응답은 **JSON 데이터**이고 그 페이지는 DRF가 개발 편의를 위해 자체 제공하는 것

---

## 💡 한 줄 요약

> DRF는 Django에서 RESTful API 서버를 만들기 위한 오픈소스 라이브러리이며, 핵심은 QuerySet을 JSON으로 변환하는 Serializer 클래스다.

## ❓ 더 찾아볼 것

- DRF 공식 문서 (djangorestframework.org)
- Serializer 필드 타입 종류
- `fields`에 튜플로 특정 필드만 지정하는 방법
- `read_only_fields` 옵션
