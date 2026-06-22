# [관통 PJT] 지난 관통 프로젝트 Summary

---

## 1. Git 핵심 명령어 복습

Git은 **분산 버전 관리 시스템**으로, 클라우드(GitHub 등)에 코드를 올려 팀원끼리 공유하고 커밋 단위로 변경 이력을 관리한다.

### Git 상태 영역

| 영역 | 설명 |
|------|------|
| Working Directory | `git status`에서 빨간색으로 표시되는, 아직 추적되지 않은 파일 |
| Staging Area | `git add`로 버전 관리 대상으로 등록된 파일 (초록색) |
| Repository | `git commit`으로 저장된 버전 이력 |

### 커밋 되돌리기

```bash
git revert <commit>   # 커밋 취소 (취소 커밋을 새로 생성)
git reset --soft      # 스테이징 영역으로 되돌림
git reset --mixed     # 워킹 디렉토리로 되돌림 (기본값)
git reset --hard      # 커밋 내용 완전 삭제
git restore           # 스테이징 / 워킹 디렉토리의 변경사항 되돌림
```

### pathlib — Python 파일 경로 다루기

1차시에 함께 배운 내용으로, OS에 상관없이 안전하게 파일 경로를 다룰 수 있게 해주는 표준 라이브러리다.

```python
from pathlib import Path

base = Path("images")
files = list(base.glob("*.png"))   # 모든 PNG 파일 목록
new_path = base / "renamed.png"    # 경로 합치기
```

---

## 2. API와 JSON

### API란?

서로 다른 인터페이스(서버, 언어) 간에 통신할 수 있도록 해주는 **공통 약속**.
1차시에 OpenAI API를 처음 사용했고, 2차시에는 **알라딘 API**를 호출해 도서 데이터를 받아와 가공하는 실습을 진행했다.

### 왜 JSON인가?

- API 통신 시 **텍스트만 주고받을 수 있음**
- 각 서버의 자료구조를 **JSON(키-밸류 형태)** 으로 직렬화해서 교환
- JSON은 사실상 API 통신의 표준

```python
import json

data = {"title": "파이썬 입문", "author": "홍길동"}
json_str = json.dumps(data)   # dict → JSON 문자열
back = json.loads(json_str)   # JSON 문자열 → dict
```

> 예: Python 서버(딕셔너리) ↔ Java 서버(오브젝트) → 언어가 달라 직접 교환 불가 → JSON 텍스트로 중간 변환 후 통신

---

## 3. MCP 활용 vs MCP 제작

관통 과정에서 MCP는 두 차례에 걸쳐 서로 다른 방식으로 다뤘다.

### ① 2차시 — MCP 직접 활용 (AI 에이전트 실습)

폴더 안의 이미지를 읽고, AI가 이미지를 해석한 뒤 파일명을 내용에 맞게 자동 변경하는 실습을 진행했다. VS Code Copilot이 에이전트 방식으로 MCP를 선택해 작업을 수행했다.

```
이미지 폴더 → MCP 에이전트 실행 → 이미지 내용 분석 → 파일명 자동 변경
```

### ② 마지막(13차시) — MCP 서버 직접 제작 + PyPI 배포

날씨 정보 수집 MCP 서버, 뉴스 크롤링 MCP 서버를 **직접 만들고** PyPI에 배포하는 것까지 진행했다. 더 나아가 AI 에이전트가 이 MCP 서버를 활용해 뉴스 보고서를 자동 작성하는 것도 실습했다.

```bash
# PyPI 배포
python -m build
twine upload dist/*
```

---

## 4. GitFlow 전략

실무와 팀 프로젝트 모두에서 통용되는 브랜치 관리 전략이다.

```
master  ←── hotfix
  ↑
release
  ↑
dev  ←── feature/기능명
```

| 브랜치 | 역할 |
|--------|------|
| `master` | 실제 배포 중인 코드. 직접 작업 금지 |
| `dev` | 개발 코드가 통합되는 공간 (배포 환경과 동일하게 구성) |
| `release` | 배포 직전 코드 보관 장소 (dev를 잠그지 않기 위해 분리) |
| `feature/xxx` | 각 개발자가 기능을 개발하는 브랜치. 자유롭게 수정 가능 |
| `hotfix` | 배포(master)에서 버그 발생 시 긴급 수정 후 master로 바로 합침 |

**개발 흐름**

```
feature 브랜치에서 개발
→ dev 가져온 뒤 버그 테스트
→ 문제 없으면 dev로 병합
→ 전체 QA
→ release 브랜치로 올림
→ master 배포
```

> 코드 작업은 반드시 **feature** 또는 **hotfix** 브랜치를 따서 진행한다.  
> master, dev, release 브랜치에서 직접 코드를 작성하면 안 된다.

---

## 5. UI/UX 핵심 개념

### CSS 커스텀 폰트 적용

```css
@font-face {
  font-family: 'MyFont';
  src: url('./fonts/MyFont.woff2') format('woff2');
}

body {
  font-family: 'MyFont', sans-serif;
}
```

### UI 컴포넌트 용어

| 용어 | 설명 |
|------|------|
| Carousel | 이미지/카드를 슬라이드로 보여주는 컴포넌트 |
| Modal | 현재 화면 위에 팝업으로 뜨는 대화상자 |
| Navigation Bar | 페이지 상단의 메뉴 바 |
| Toast | 잠깐 나타났다 사라지는 알림 메시지 |
| Pagination | 긴 목록을 여러 페이지로 나누는 UI |

### UX 설계 원칙

- **실시간 피드백 제공**: 로딩, 완료, 오류 상태를 사용자에게 즉시 알려야 한다
- **정보 과잉 금지**: 한 화면에 너무 많은 정보를 담으면 UX를 해친다
- **엄지존 고려**: 모바일에서 사용자가 엄지로 편하게 닿는 영역을 고려한 버튼 배치

---

## 6. OpenAI Structured Output

AI는 확률 기반으로 응답하기 때문에 출력 형태가 불규칙할 수 있다. Structured Output을 활용하면 **정해진 JSON 스키마 형식**으로 응답을 강제할 수 있다.

```python
from openai import OpenAI
from pydantic import BaseModel

class Book(BaseModel):
    title: str
    author: str
    genre: str

client = OpenAI()
response = client.beta.chat.completions.parse(
    model="gpt-4o",
    messages=[{"role": "user", "content": "파이썬 입문서 추천해줘"}],
    response_format=Book,
)
book = response.choices[0].message.parsed
```

---

## 7. 리팩토링

**내부 동작을 개선**하는 작업. 외부 기능(결과)은 바뀌지 않고 코드 품질만 향상시킨다.

### 주요 리팩토링 기법

| 기법 | 설명 |
|------|------|
| 변수명 개선 | 의미 불명확한 변수명을 명확하게 변경 |
| 매직 넘버 → 상수 | `if x > 100:` 대신 `MAX_LIMIT = 100` 사용 |
| 함수 추출 | 긴 함수를 작은 함수 단위로 분리 |
| 조건문 분해 | 복잡한 조건식을 의미 있는 함수로 대체 |
| 매개변수 객체화 | 인자가 많으면 객체(딕셔너리)로 묶기 |
| 주석 정리 | 의미 없는 주석 제거, 필요한 주석은 명확하게 |

> 돌아가는 코드가 잘 짜인 코드는 아니다. 하지만 정상 작동하는 코드를 리팩토링하기엔 용기가 필요하다.  
> **TDD와 함께 사용하면 두려움이 줄어든다.**

---

## 8. 프로젝트 기획 및 설계 프로세스

AI 시대에 가장 중요성이 높아진 영역. 기획만 잘 되어 있으면 코드는 AI가 생성해줄 수 있다.

```
문제 정의 → 아이디어 도출 → 요구사항 수집 → MVP 설정
→ 명세서 작성 → 정보 구조도(IA) → 화면 설계(Figma)
→ 프로토타입 → 기술 스택 선정 → DB 설계 → 일정 산출
```

### 일정 관리 도구

- **WBS(Work Breakdown Structure)**: 작업을 최소 단위로 분리
- **간트 차트**: 업무별 일정을 시각화
- **그라운드 룰**: 팀 내 협업 규칙 (예: API 명세는 백엔드가 주도)

> 기술 스택을 선정할 때는 "배웠으니까 쓴다"가 아니라, **이 프로젝트에 Django를 써야 하는 이유**를 설명할 수 있어야 한다.

---

## 9. 데이터베이스 정규화

데이터의 일관성과 이상 현상(삽입/갱신/삭제 이상)을 방지하기 위한 설계 원칙.

| 정규형 | 핵심 규칙 |
|--------|-----------|
| **1NF** | 하나의 컬럼에는 하나의 값만 (전화번호 여러 개를 쉼표로 구분 금지) |
| **2NF** | 복합키 테이블에서 비기본키 컬럼은 **복합키 전체**에 종속되어야 함 (부분 종속 제거) |
| **3NF** | 비기본키가 다른 비기본키에 의해 결정되면 안 됨 (이행 종속 제거) |
| **BCNF** | 모든 결정자(다른 컬럼을 결정하는 컬럼)는 후보 키여야 함 |

---

## 10. TDD (Test-Driven Development)

**테스트 주도 설계**: 먼저 요구사항을 테스트 케이스로 나열하고, 그 케이스를 통과하는 코드를 작성하는 개발 방법론.

### 흐름

```
요구사항 나열 → 테스트 케이스 작성 → 테스트 통과하는 코드 구현 → 리팩토링
```

### 장단점

| 장점 | 단점 |
|------|------|
| 설계가 깔끔해짐 | 초기 개발 속도 저하 |
| 리팩토링에 대한 두려움 감소 | 테스트 케이스 작성 비용 |
| 버그 조기 발견 | 팀 전체가 익숙해지는 데 시간 필요 |

> AI 시대에 TDD의 가치는 더 높아졌다. AI가 만든 코드의 **검수자 역할**을 테스트가 대신하기 때문이다.  
> 포트폴리오에 "TDD 방식으로 개발"을 적으면 면접에서 깊이 있는 질문을 받을 수 있으므로, 경험 없이 적는 것은 피해야 한다.

---

## 11. JavaScript 이벤트 처리

| 이벤트 유형 | 예시 |
|-------------|------|
| Input 이벤트 | 입력 시 유효성 검사 (50자 초과 시 입력 차단) |
| Drag 이벤트 | 카드 드래그&드롭으로 순서 변경 |
| Scroll 이벤트 | 스크롤 위치에 따른 진행도 표시, 배경 색 전환 |

---

## 12. 문서 벡터화와 유사도 계산

자연어(텍스트)를 숫자(벡터)로 변환하는 과정. AI 추천 기능의 핵심 기반 기술이다.

### 벡터화 방법

| 방법 | 설명 |
|------|------|
| Count Vectorization | 단어의 등장 횟수로 표현 (단순하지만 의미 미반영) |
| TF-IDF | 등장 빈도와 문서 내 희귀도를 고려한 가중치 부여 |
| **Embedding (Word2Vec 등)** | 실제로 가장 많이 사용. 주변 단어를 예측하는 모델로 학습된 벡터 |

### 코사인 유사도

두 벡터 사이의 **각도**로 유사도를 측정. 각도가 좁을수록(0에 가까울수록) 두 문서가 유사하다.

```python
from sklearn.metrics.pairwise import cosine_similarity

similarity = cosine_similarity(vec_a, vec_b)
```

---

## 13. JWT (JSON Web Token)

**토큰 기반 인증 방식**. 세션 방식과 달리 서버가 상태를 저장하지 않는다(Stateless).

### 구조

```
Header.Payload.Signature
```

| 구성 요소 | 내용 |
|-----------|------|
| Header | 토큰 유형 및 암호화 알고리즘 |
| Payload | 사용자 정보 (공개 가능한 정보. 탈취 시 노출될 수 있음) |
| Signature | 위·변조 검증용 서명 |

### 인증 흐름

```
로그인 → Access Token(유효기간 짧음) + Refresh Token 발급
→ Access Token 만료 시 Refresh Token으로 재발급
→ Refresh Token도 만료 시 재로그인
```

> 2학기 프로젝트에서는 거의 **95%가 JWT 방식**을 사용하니 반드시 숙지할 것.

---

## 💡 한 줄 요약

> 1학기 관통 13차시를 통해 Git·pathlib·API/JSON·MCP 활용→제작·GitFlow·UI/UX·Structured Output·리팩토링·기획설계·DB정규화·TDD·이벤트·벡터화·JWT까지, 실무 전 주기를 종단면으로 경험했다.

---

## ❓ 더 찾아볼 것

- GitFlow vs Trunk-Based Development 비교
- pathlib vs os.path 차이점
- OpenAI Structured Output 공식 문서 (JSON Schema 활용법)
- Word2Vec vs BERT 임베딩 차이
- JWT 탈취 방지 전략 (HttpOnly Cookie, Refresh Token Rotation)
- TDD 실전 적용 사례 (pytest, Django Test Client)
