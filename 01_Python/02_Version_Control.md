# [Git] 버전 관리 및 원격 저장소 활용 (Version Control & GitHub)

> **핵심 키워드:** #Git #GitHub #DVCS #Commit #Push #Pull #Clone #Gitignore

---

## 🎯 학습 목표
* 분산 버전 관리 시스템(DVCS)의 원리 및 필요성 이해
* Git의 3가지 영역(Working Directory, Staging Area, Repository) 구조 파악
* 로컬 저장소와 원격 저장소(GitHub) 간의 데이터 흐름 및 명령어 숙달
* `.gitignore`를 활용한 효율적인 프로젝트 관리 기법 습득

---

## 💡 주요 개념 정리

### 1. 버전 관리의 본질
* **효율적 저장 방식:** 원본 전체가 아닌 **변경 사항(Snapshot)**만 기록하여 용량 최적화
* **롤백(Rollback):** 최종본에서 변경 사항을 역으로 계산하여 특정 시점으로 완벽한 복구 가능
* **분산 관리:** 모든 팀원이 개별 로컬 저장소를 소유하여 중앙 서버 장애 시에도 데이터 보존 및 오프라인 작업 가능

### 2. Git의 3대 영역

* **Working Directory:** 현재 실제 파일 작업을 수행하는 로컬 폴더 영역
* **Staging Area:** 커밋(버전 생성) 전 변경 사항을 임시로 모아두는 준비 공간
* **Repository:** 최종적으로 커밋이 기록되어 영구 저장되는 로컬 또는 원격 저장소

### 3. Git vs GitHub 구분
* **Git:** 소스 코드의 버전을 관리하는 시스템(도구) 자체
* **GitHub:** Git을 활용하여 협업 기능을 제공하는 웹 기반 원격 저장소 서비스

---

## 💻 기능 구현 및 명령어 실습

### 1. 로컬 저장소 기본 설정 및 커밋
내 컴퓨터에서 버전 관리를 시작하고 변화를 기록하는 필수 과정임.

```bash
# 저장소 초기 설정
git init # 현재 폴더를 Git 저장소로 초기화 (.git 폴더 생성)

# 사용자 정보 등록 (최초 1회)
git config --global user.email "minkyu@example.com"
git config --global user.name "Patrick"

# 변경 사항 기록 루틴
git status # 현재 파일들의 상태(추적 여부) 상시 확인
git add sample.txt # 특정 파일을 Staging Area로 추가
git add . # 현재 경로의 모든 변경 사항을 한꺼번에 스테이징
git commit -m "feat: add sample file" # 스테이징된 파일을 메시지와 함께 버전으로 저장
git log --oneline # 커밋 히스토리를 한 줄씩 요약해서 확인
```

### 2. 원격 저장소(GitHub) 연동 및 협업
로컬과 원격 저장소를 동기화하여 코드를 공유하고 백업하는 기법임.

```bash
# 원격 저장소 연결
git remote add origin [GitHub URL] # origin이라는 별명으로 원격 주소 등록
git remote -v # 연결된 원격 저장소 목록 및 상태 확인

# 코드 전송 및 수신
git push origin master # 로컬 master 브랜치의 기록을 원격(origin)으로 업로드
git pull origin master # 원격의 최신 변경 사항을 로컬로 가져와서 자동 병합
git clone [URL] # 원격 저장소의 전체 내용을 새로운 로컬 폴더로 복제
```

### 3. 파일 추적 제외 관리 (.gitignore)
보안상 민감한 정보(API 키 등)나 불필요한 설정 파일의 업로드를 방지함.

```bash
# .gitignore 파일 작성 예시
touch .gitignore # 무시할 파일 목록을 담은 설정 파일 생성

# .gitignore 내부 내용 예시
# c.txt           <- 특정 파일 무시
# *.env           <- 모든 .env 확장자 파일 무시 (API 키 보호)
# /venv/          <- 가상환경 폴더 전체 무시
```

---

## 🚀 복습 및 AI 인사이트
* **단축 명령어(Alias) 설정:** `git config --global alias.st status`와 같이 자주 쓰는 명령어를 별칭으로 등록하여 작업 속도 향상 권장
* **AI 활용 주의 사항:** AI가 작성한 코드를 맹신하지 말고 비판적으로 검토하며, 주석을 통해 코드의 의도를 명확히 기록하는 습관 필요
* **본질적 역량:** 프로그래밍 언어는 문제를 해결하기 위한 '수단'일 뿐이며, 논리적 사고와 문제 해결 능력 배양에 집중할 것