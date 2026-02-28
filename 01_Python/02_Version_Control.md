# [Git] 버전 관리 및 원격 저장소 활용 (Version Control & GitHub)

> **핵심 키워드:** #Git #GitHub #DVCS #Commit #Push #Pull #Clone #Gitignore

---

## 🎯 학습 목표
* 분산 버전 관리 시스템(DVCS)의 원리 및 필요성 이해
* Git의 3가지 영역(Working Directory, Staging Area, Repository) 구조 파악
* 로컬 저장소와 원격 저장소(GitHub) 간 데이터 흐름 및 명령어 숙달
* `.gitignore` 활용 효율적인 프로젝트 관리 기법 습득

---

## 💡 주요 개념 정리

### 1. 버전 관리의 본질
* **효율적 저장 방식:** 원본 전체가 아닌 **변경 사항(Snapshot)** 중심의 기록을 통한 용량 최적화
* **롤백(Rollback):** 최종본 기반 변경 사항 역산 및 특정 시점의 완벽한 복구 기능
* **분산 관리:** 팀원별 개별 로컬 저장소 소유를 통한 데이터 보존 및 오프라인 작업 환경 확보

### 2. Git의 3대 영역

* **Working Directory:** 실제 파일 수정 및 코드 작성이 이루어지는 로컬 작업 영역
* **Staging Area:** 커밋 생성 전 변경 사항을 선별적으로 모아두는 준비 공간
* **Repository:** 로컬 또는 원격에 최종 커밋 내역이 영구 저장되는 영역

### 3. Git vs GitHub 구분
* **Git:** 소스 코드 버전 관리를 수행하는 로컬 시스템(도구)
* **GitHub:** Git 기반 협업 기능 및 원격 저장소 호스팅을 제공하는 웹 서비스

---

## 💻 기능 구현 및 명령어 실습

### 1. 로컬 저장소 기본 설정 및 커밋
로컬 저장소 초기화 및 변경 사항 기록 수행 과정

```bash
# 저장소 초기 설정
git init # 현재 폴더에 .git 폴더 생성 및 관리 시작

# 사용자 정보 등록 (blame 및 추적용)
git config --global user.email "minkyu@example.com"
git config --global user.name "Patrick"

# 변경 사항 기록 루틴
git status # 파일 추적 상태(Working Directory/Staging Area) 상시 확인
git add sample.txt # 특정 파일 스테이징
git add . # 모든 변경 사항 일괄 스테이징
git commit -m "feat: add sample file" # 스테이징 영역 파일을 버전(Commit)으로 저장
git log --oneline # 커밋 히스토리 한 줄 요약 조회
```

### 2. 원격 저장소(GitHub) 연동 및 협업
로컬-원격 저장소 동기화 및 코드 공유/백업 기법

```bash
# 원격 저장소 연결
git remote add origin [GitHub URL] # 'origin'이라는 별칭으로 원격 주소 등록
git remote -v # 연결된 원격 저장소 목록 상세 확인

# 코드 전송 및 수신
git push origin master # 로컬 기록을 원격 저장소로 업로드
git pull origin master # 원격의 최신 내역을 가져와 로컬에 자동 병합
git clone [URL] # 원격 저장소 전체 내용을 새로운 로컬 폴더로 복제
```

### 3. 파일 추적 제외 관리 (.gitignore)
보안 민감 정보(API 키) 및 불필요한 설정 파일 업로드 방지 설정

```bash
# .gitignore 파일 생성
touch .gitignore 

# .gitignore 내부 설정 예시
# c.txt           <- 특정 파일 추적 제외
# *.env           <- 모든 환경변수 파일 무시 (보안 유지)
# /venv/          <- 가상환경 폴더 전체 제외
```

---

## 🚀 복습 및 AI 인사이트
* **단축 명령어(Alias) 설정:** `git config --global alias.st status` 등록을 통한 작업 속도 향상
* **AI 활용 주의 사항:** AI 생성 코드의 비판적 검토 및 주석을 이용한 코드 의도 명시 습관화
* **본질적 역량:** 도구(Git) 활용법보다 중요한 논리적 사고 및 문제 해결 능력 중심의 학습 지향