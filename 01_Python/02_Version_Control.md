# [Git] 버전 관리 시스템 & 원격 저장소 (GitHub)
> **핵심 키워드:** #Git #GitHub #VersionControl #CLI #분산버전관리

---

## 🎯 학습 목표
* Git이 버전 관리를 하는 원리(변경 이력 저장 방식) 이해
* 중앙집중식 vs 분산 버전 관리 시스템 차이 파악
* Git 3영역(Working Directory → Staging Area → Repository) 흐름 숙달
* `init` / `add` / `commit` / `status` / `log` 핵심 명령어 실습
* 원격 저장소(GitHub) 연결 및 `push` / `pull` / `clone` 워크플로우 습득
* `.gitignore`로 추적 제외 파일 관리

---

## 💡 주요 개념 정리

### 1. Git이란?
* **분산 버전 관리 시스템(Distributed Version Control System)**
* Git ≠ GitHub: Git은 버전 관리 **기술(시스템)**, GitHub·GitLab은 Git을 활용한 **서비스(웹사이트)**
* SSAFY: 실습 저장소로 **GitLab** 사용, 개인 포트폴리오 등은 **GitHub** 사용

### 2. 버전 관리가 필요한 이유
* 흔히 쓰는 방식 → `발표자료_최종`, `발표자료_찐최종`, `발표자료_찐찐최종…` → **관리 불가**
* 날짜/시간 부여 방식 개선 → 어떤 내용이 바뀌었는지 **기억 불가**
* 변경사항 기록 파일 별도 관리 → 파일이 1GB면 복사본 1,000개 = **1,000GB 낭비**

**Git의 해결책**
* 원본(최종본) **하나**만 유지
* 매 버전마다 **변경된 부분(변경 이력)만** 저장
* 롤백 시: 최종본에서 변경사항만 역순으로 **제거** → 과거 시점 복원

> 💡 **핵심:** 변경 이력만 남기고, 최종 파일 하나만 보유 → 용량 최소화 + 자유로운 롤백

### 3. 중앙집중식 vs 분산식

| 구분 | 중앙집중식 | **분산식 (Git)** |
|---|---|---|
| 코드 위치 | 서버 1곳 | 각 개발자 로컬 컴퓨터 + 중앙 서버 |
| 서버 장애 시 | **코드 전체 손실** | 각자 로컬에 복사본 보유 → **안전** |
| 인터넷 없이 작업 | 불가 | **가능** (나중에 동기화) |
| 동시 편집 충돌 | 실시간 충돌 위험 | 각자 로컬에서 작업 → **충돌 최소화** |

* **로컬(Local):** 지금 내가 앉아 있는 이 컴퓨터 ("로컬 맛집" = 그 지역 맛집)
* 변경 이력/코드 → 내 컴퓨터에 저장 → 나중에 GitHub과 **동기화**

### 4. Git 3영역 (핵심 동작 흐름)
```
[Working Directory] → (git add) → [Staging Area] → (git commit) → [Repository]
    작업 중인 공간          버전 저장 전 대기 공간          버전(커밋)이 영구 저장되는 곳
```

* **Working Directory:** 실제 파일을 수정하는 작업 공간 → `git status` 시 **빨간색** 표시
* **Staging Area:** 다음 버전에 포함시킬 파일들을 선택적으로 모아두는 **중간 준비 영역** → `git status` 시 **초록색** 표시
* **Repository:** 커밋(버전)과 모든 변경 이력이 영구 기록되는 공간 (`.git` 폴더 내부)

> 💡 "파일 저장(Ctrl+S)" ≠ "커밋": Ctrl+S는 VS Code 저장, 커밋은 **Git 버전 생성**

### 5. 커밋(Commit)이란?
* **버전** 과 동일한 개념 — Git에서는 `커밋` 용어 사용
* Staging Area에 올라온 파일들을 하나의 **버전으로 묶어 저장**하는 행위
* 각 커밋에는 **작성자(이메일·이름), 날짜, 변경 내용** 기록
* `git blame`: 코드 줄마다 누가 언제 수정했는지 **추적** → 협업 시 책임 소재 파악

---

## 💻 기능 구현 및 코드 실습

### 🔧 초기 설정 (최초 1회)
```bash
# Git 사용자 정보 등록 (커밋에 작성자 정보 기록)
git config --global user.email "본인이메일@example.com"
git config --global user.name "본인이름"

# 설정 확인
git config --global -l

# git status 단축 별명 등록 (강사님 권장 습관)
git config --global alias.st status
# 이후 'git st' 만 입력해도 'git status'와 동일 동작
```

### 🔧 로컬 버전 관리 기본 흐름
```bash
# 1. 버전 관리할 폴더를 Git 로컬 저장소로 초기화
git init
# → .git 숨김 폴더 생성, 터미널에 (master) 브랜치 표시됨

# 2. 파일 생성 (CLI 방식)
touch sample.txt

# 3. 현재 상태 확인 (습관적으로 자주 실행)
git status        # 또는 git st (별명 등록 시)
# → 빨간색: Working Directory에 있는 파일 (아직 Staging 미등록)

# 4. 변경 파일을 Staging Area로 등록
git add sample.txt      # 특정 파일만 등록
git add .               # 현재 폴더의 모든 파일 등록 (CLI 점(.) = 현재 폴더)
git add -A              # 위치와 상관없이 init한 폴더 내 모든 변경사항 등록
                        # ※ -A 남발 주의: 의도치 않은 파일 포함 가능

# 5. 상태 재확인
git status
# → 초록색: Staging Area에 등록 완료 (커밋 대기 중)

# 6. 커밋 생성 (버전 저장)
git commit -m "커밋 메시지 작성"
# -m 옵션: 커밋 메시지를 한 줄로 간단하게 작성할 때 사용
# -m 없이 git commit 만 입력하면 Vim 에디터 진입
#   → i 키: 입력(INSERT) 모드 진입
#   → ESC: 입력 모드 종료
#   → :wq + Enter: 저장 후 종료 (Write + Quit)

# 7. 커밋 이력 확인
git log               # 전체 커밋 이력 (작성자, 날짜, 메시지 포함)
git log --oneline     # 한 줄 요약 목록 (간결 확인 시 유용)

# 8. 변경사항 되돌리기 (Working Directory의 수정 내용 취소)
git restore sample.txt
# ※ 주의: 마지막 커밋 기준으로 해당 파일의 변경사항 전체 삭제 (복구 불가)
#          잘못 수정했을 때보다, 작업 내용 전체가 불필요할 때 사용
```

### ⚠️ git init 중첩 금지
```bash
# 잘못된 예시 — 절대 금지!
# 바탕화면(Desktop)에서 git init → 이후 그 안의 폴더에서 또 git init
# → 버전 관리 꼬임, 충돌 발생

# 실수로 잘못된 위치에 init한 경우 → .git 폴더 삭제로 취소
rm -rf .git
# ※ 주의: 원격(GitHub)에 올리기 전이라면 이 폴더의 모든 커밋 이력 함께 삭제됨
```

### 🔧 원격 저장소(GitHub) 연동 흐름
```bash
# [사전 준비] GitHub에서 새 Repository 생성 후 URL 복사

# 1. 로컬 저장소와 원격 저장소 연결
git remote add origin https://github.com/유저명/레포명.git
# origin: 이 URL의 별명 (긴 URL을 매번 입력하지 않기 위한 별칭)
# 별명은 origin 외 다른 이름도 가능하나 관례상 origin 사용

# 2. 연결 상태 확인
git remote -v

# 3. 로컬 커밋을 원격 저장소에 업로드 (Push)
git push origin master
# origin: 연결된 원격 저장소 별명
# master: 현재 브랜치명
# ※ 푸시 전 반드시 커밋 완료 확인 — 커밋 없이 push하면 올라가지 않음
#   push 안 될 때 → 먼저 git status로 미커밋 파일 여부 확인

# 4. 원격 저장소의 변경사항 가져오기 (Pull)
git pull origin master
# 또는 단순히: git pull
# 이미 clone한 저장소에서 다른 사람의 업데이트된 커밋만 부분 반영

# 5. 원격 저장소를 처음 로컬로 복제 (Clone)
git clone https://github.com/유저명/레포명.git
# 최초 1회만 사용 — 이후에는 pull로 업데이트
# clone한 폴더는 .git이 이미 포함 → git init 추가 불필요
```

### 🔧 .gitignore 설정
```bash
# .gitignore 파일 생성
touch .gitignore
```
```
# .gitignore 파일 내부 작성 예시
# 추적하지 않을 파일명 또는 패턴 입력

secret.txt          # 특정 파일 제외
.env                # API 키 등 환경변수 파일 (절대 GitHub에 업로드 금지)
__pycache__/        # Python 캐시 폴더
*.log               # 모든 .log 확장자 파일
```
```bash
# gitignore 작성 후 상태 확인
git status
# → .gitignore에 등록된 파일은 아예 목록에 나타나지 않음

# ※ 중요 함정: 이미 커밋(버전 관리)된 파일은 .gitignore에 추가해도 추적 계속됨
#   해결 방법: git rm --cached 파일명 (다음 주 학습 예정)

# .gitignore 파일 자체도 반드시 커밋 관리
git add .gitignore
git commit -m "add .gitignore"
```

> 💡 **gitignore.io 활용:** [gitignore.io](https://www.toptal.com/developers/gitignore) 에서 사용 기술(Django, Python 등) 입력
> → 해당 프레임워크에서 올리면 안 되는 파일 목록 자동 생성 → 복사 후 `.gitignore`에 붙여넣기

### 🔧 전체 워크플로우 요약
```bash
# [로컬 작업 반복 사이클]
git status          # 1. 현재 상태 확인 (습관화 필수)
git add .           # 2. 변경 파일 Staging에 등록
git status          # 3. Staging 상태 재확인 (초록색 확인)
git commit -m "작업 내용 메시지"   # 4. 버전(커밋) 생성
git push origin master            # 5. 원격 저장소에 업로드

# [다른 환경에서 작업 이어서 하기]
git pull origin master  # 최신 커밋 내려받기
```

---

## 🚀 복습 및 AI 인사이트

### ✅ 핵심 체크포인트
* **Git ≠ GitHub:** Git은 버전 관리 시스템(기술), GitHub/GitLab은 이를 활용한 서비스
* **변경 이력만 저장:** 매 버전마다 전체 복사본 X → 최종본 1개 + 변경 이력만 유지
* **빨간색 = Working Directory, 초록색 = Staging Area:** `git status` 색깔로 영역 파악
* **`git add .` vs `git add -A`:** `.`은 현재 위치 기준, `-A`는 init 폴더 전체 기준
* **커밋 없이 push 불가:** push 오류 시 `git status` 먼저 — 미커밋 파일 있는지 확인
* **clone vs pull:** clone은 최초 1회 전체 복제, pull은 이후 변경사항 부분 업데이트
* **git init 중첩 금지:** 이미 init된 폴더 내부에서 재init 절대 금지 → 버전 관리 꼬임
* **.git 폴더 = 버전 이력 저장소:** 원격 업로드 전 삭제 시 모든 커밋 이력 소멸 주의
* **.gitignore 선행 등록 원칙:** 이미 커밋된 파일은 .gitignore 추가해도 계속 추적됨

### 💬 강사님 마인드셋 메모
* **언어는 수단:** Python, Java, C++ 중 무엇이 중요한 게 아님 → 소설 작가에게 한글/영어보다 **이야기를 쓰는 능력**이 중요한 것처럼, **논리적 사고·문제 해결 능력**이 본질
* **AI 활용 원칙:** AI가 생성한 코드 그대로 복붙 금지 → 왜 동작하는지 이해 + 주석으로 정리하는 **습관** 필수
* **AI 한계 인지:** AI(Gemini, GPT 등)는 Hallucination(환각 증세) 존재 → 논리적 사고로 **비판적 검토·테스트** 능력 필요
* **현재 우선순위:** 메모리 최적화·성능 집착보다 → **직접 기능 구현·완성** 경험 우선
* **복습 습관:** 주말 최소 1시간, 점진적으로 늘려가기

### 🤖 AI 활용 팁
* **프롬프트 예시 (에러 해결):** `"git push origin master 실행 시 [에러 메시지] 발생. 원인과 해결 방법을 단계별로 설명해줘"`
* **프롬프트 예시 (개념 확인):** `"git add . 과 git add -A 의 차이를 폴더 구조 예시와 함께 설명해줘"`
* **프롬프트 예시 (.gitignore):** `"Django + Python 프로젝트에서 .gitignore에 반드시 포함해야 할 항목 목록 만들어줘"`