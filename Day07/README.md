# Day07 — Git 복구 전략 · 파일 시스템(pathlib) · 파일 입출력 · 생성형 AI 활용

> **학습 목표**
> - Git에서 되돌리기 전략(revert / reset)의 차이를 이해한다.
> - `pathlib`으로 파일 시스템을 객체처럼 다룬다.
> - 파일 생성·읽기·쓰기·탐색을 안전하게 수행한다.
> - 파일 처리 결과를 생성형 AI(API)와 연결하는 전체 흐름을 이해한다.

---

## 1. 오늘의 핵심 요약
- `git revert`는 **기록을 남기며 취소**, `git reset`은 **과거로 이동**한다.
- `reset`은 옵션(soft/mixed/hard)에 따라 코드 보존 수준이 달라진다.
- `pathlib`은 **경로를 문자열이 아닌 객체**로 다룬다.
- `with open()`은 파일을 **안전하게 열고 닫는 표준 패턴**이다.
- 파일 처리 결과를 모아 **OpenAI API로 요약 자동화**가 가능하다.

---

## 2. 개념 구조 정리

```
개발 도구 & 파일 시스템
├─ Git 관리 기술
│   ├─ revert (기록 남김)
│   └─ reset
│       ├─ soft
│       ├─ mixed (기본)
│       └─ hard
│
├─ 파일 시스템 (Python)
│   ├─ pathlib.Path
│   ├─ 경로 결합 / 탐색
│   ├─ 파일/폴더 생성
│   └─ 파일 읽기/쓰기
│
└─ 응용
    ├─ 파일 요약 파이프라인
    └─ OpenAI API 연동
```

---

## 3. Git 복구 전략 핵심

### 3-1. git revert

> **특정 커밋의 작업을 없던 일로 만들되, 새로운 커밋으로 기록을 남김**

```bash
git revert <commit-id>
```

- 협업 환경에서 **권장**
- 히스토리 보존

---

### 3-2. git reset

> **과거 커밋 시점으로 HEAD 이동**

```bash
git reset <옵션> <commit-id>
```

| 옵션 | 코드 상태 |
|------|-----------|
| soft | 스테이징에 남김 |
| mixed | 워킹 디렉토리에 남김 |
| hard | 완전 삭제 |

⚠️ `hard`는 복구 불가 → 단독 작업에서만 사용

---

## 4. pathlib 개요 (왜 필요한가)

### 4-1. 문제점
- 경로를 문자열로 다루면 `split`, `join` 난무

### 4-2. 해결

```python
from pathlib import Path

p = Path.cwd()        # 현재 경로
home = Path.home()    # 홈 디렉토리
```

➡ 경로 자체를 **객체**로 관리

---

## 5. 경로 객체 주요 속성

```python
file = Path('docs/file.txt')
```

| 속성 | 의미 |
|------|------|
| file.name | 파일명 + 확장자 |
| file.stem | 파일명 |
| file.suffix | 확장자 |

---

## 6. 파일 · 폴더 생성

```python
new_dir = Path('new_directory')
new_dir.mkdir(exist_ok=True)
```

```python
(new_dir / 'new.txt').write_text('내용', encoding='utf-8')
```

- 상대 경로 기준
- `exist_ok=True` → 에러 방지

---

## 7. 파일 읽기 & with 문

```python
with open('new.txt', 'a', encoding='utf-8') as f:
    f.write('추가 내용')
```

- `with` → 자동 close
- `a` : append / `r` : read / `w` : write

---

## 8. 파일 탐색 (iterdir / glob)

```python
for item in Path.cwd().iterdir():
    print(item)
```

```python
Path.cwd().glob('*.py')      # 현재 폴더
Path.cwd().rglob('*.py')     # 하위 폴더 포함
```

- 반환값은 **이터레이터** (지연 평가)

---

## 9. 인코딩 (UTF-8)

- 한글 깨짐 방지 필수
- UTF-8 = 국제 표준 유니코드

```python
open('file.txt', encoding='utf-8')
```

---

## 10. 응용: 생성형 AI 요약 파이프라인

1. 파일 경로 탐색
2. 파일 내용 읽기
3. 텍스트 결합
4. OpenAI API 요청
5. 요약 결과를 md 파일로 저장

➡ **Day07은 전체 파이프라인 사고 훈련용**

---

## 11. 실수 & 시험 감점 포인트

| 실수 | 결과 |
|------|------|
| revert vs reset 혼동 | 히스토리 파괴 |
| hard 무분별 사용 | 복구 불가 |
| 인코딩 누락 | 한글 깨짐 |
| iterdir 바로 출력 | 제너레이터 출력 |

---

## 12. 5분 요약

- revert = 기록 남김 취소
- reset = 과거 이동
- pathlib = 경로 객체화
- with = 안전한 파일 처리
- 파일 → AI → 자동 요약

---

### 이동
👉 [복습 노트](./review.md)
👉 [메인 README](../README.md)

