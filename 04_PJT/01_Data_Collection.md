# [PJT] 데이터 수집 및 관리 (Data Collection & Management)

> **핵심 키워드:** #Git_Advanced #Pathlib #File_System #Revert #Reset #Markdown_Collection

---

## 🎯 학습 목표
* Git의 고급 복구 기술(Revert, Reset, Restore, RM) 습득 및 저장소 관리 역량 강화
* Python 표준 라이브러리 `pathlib`을 활용한 객체 지향적 파일 시스템 제어 숙달
* 인코딩(UTF-8)의 개념 이해 및 다양한 운영체제 환경에서의 파일 입출력 처리 능력 배양

---

## 💡 주요 개념 정리

### 1. Git 버전 관리 심화 기술
* **리버트(Revert):** 기존 커밋 기록을 보존한 채 특정 시점의 작업 내역만 취소하는 신규 커밋 생성
* **리셋(Reset):** 저장소 상태를 과거 시점으로 완전히 되돌리는 롤백 기능 (Soft, Mixed, Hard 옵션에 따른 보존 범위 차이)
* **리스토어(Restore):** 워킹 디렉토리 또는 스테이징 에어리어의 변경 사항을 이전 상태로 복구
* **RM:** Git의 추적(Tracking) 중단 및 파일 삭제 기능 (`--cached` 옵션을 통한 `.gitignore` 미반영 파일 해결)

### 2. 파일 시스템 관리 (Pathlib)
* **객체 지향적 경로:** 문자열 기반 경로 처리의 번거로움을 해결하기 위해 경로 자체를 하나의 객체(Path)로 관리
* **주요 속성:** 파일 전체 이름(`name`), 확장자 제외 이름(`stem`), 확장자(`suffix`)의 직관적 추출
* **플랫폼 독립성:** 슬래시(/) 연산자를 활용하여 윈도우와 맥/리눅스 간 경로 구분자 차이 자동 해결

---

## 💻 기능 구현 및 코드 실습

### 1. Git 실수 복구 및 추적 제어
버전 관리 과정에서의 오류 수정 및 파일 제외 설정 기법

```bash
# 특정 커밋의 작업 내역 취소 (기록 보존)
git revert <commit_id>

# 스테이징된 파일 다시 내리기
git restore --staged <file_name>

# 이미 커밋된 파일을 추적 대상에서 제외 (.gitignore 적용 시 필수)
git rm --cached <file_name>

# 리셋으로 소실된 커밋 기록 조회 및 복구
git reflog
git reset --hard <commit_id>
```

### 2. Pathlib 활용 데이터 탐색 및 입출력
객체 기반 파일 시스템 조작을 통한 효율적인 데이터 수집 로직

```python
from pathlib import Path

# 1. 현재 작업 경로 확인 및 폴더 생성
current_path = Path.cwd()
new_dir = current_path / "data"
new_dir.mkdir(exist_ok=True)  # 폴더 존재 시 에러 방지

# 2. 와일드카드를 활용한 특정 파일 일괄 탐색 (Data Collection)
# 현재 폴더 내의 모든 Markdown 파일 수집
md_files = list(current_path.glob("*.md"))

# 3. 파일 안전하게 쓰기 (UTF-8 인코딩 준수)
file_path = new_dir / "summary.txt"
file_path.write_text("Hello Python", encoding="utf-8")
```

---

## 🚀 복습 및 AI 인사이트
* **인코딩의 중요성:** 한글 등 멀티바이트 문자 처리 시 `utf-8` 옵션을 명시하여 환경 간 문자 깨짐 현상 방지 필수
* **게으른 평가 (Lazy Evaluation):** `iterdir()` 또는 `glob()` 호출 시 결과가 제너레이터로 반환되어 실제 사용 시점까지 연산이 지연되는 효율적 메모리 관리 구조
* **안전한 파일 열기:** `with` 구문(Context Manager) 활용을 통한 리소스 자동 해제 및 파일 파손 방지 설계 지향
* **수집 전략:** 대규모 프로젝트 내 산재한 TIL 파일 등을 자동 수집하기 위해 `rglob` (재귀적 탐색) 메서드 활용 권장