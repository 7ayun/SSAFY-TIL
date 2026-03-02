# [Python] 버전 관리 — Git, GitHub, 원격 저장소

> **핵심 키워드:** #Git #GitHub #버전관리 #commit #push #pull #clone #gitignore #분산시스템

---

## 학습 목표

* Git의 역할(버전 관리 + 분산 협업)과 GitHub과의 차이 이해
* Working Directory → Staging Area → Repository 흐름 파악
* git init, add, commit, status, log 명령어로 로컬 버전 관리 수행
* 원격 저장소 연결(remote) 후 push/pull/clone으로 협업 환경 구축
* .gitignore로 추적 제외 파일 관리

---

## 1. 버전 관리란

버전 관리는 파일의 변화를 기록하고 추적하는 행위다. 우리는 이미 `보고서_최종.pptx`, `보고서_진짜최종.pptx`, `보고서_찐찐최종.pptx` 같은 방식으로 버전 관리를 해왔다.

이 방식의 문제점은 파일 전체를 매번 복사하므로 버전이 늘어날수록 용량이 기하급수적으로 커진다는 것이다. Git은 이를 해결하기 위해 **변경 사항(delta)만 저장**하고 최종본 하나만 유지하는 구조를 사용한다.

```
# 기존 방식: 파일 전체를 매번 복사
보고서_v1.pptx (10MB)
보고서_v2.pptx (10MB)  ← 1줄만 바꿔도 10MB 추가
보고서_v3.pptx (10MB)

# Git 방식: 변경 사항만 기록 + 최종본 1개 유지
v1 → v2: "3페이지 제목 변경" (수 KB)
v2 → v3: "5페이지 이미지 추가" (수 KB)
최종본: 보고서.pptx (10MB)
```

---

## 2. Git vs GitHub

| 구분 | Git | GitHub |
|------|-----|--------|
| 정체 | 분산 버전 관리 **시스템** | Git을 활용한 원격 저장소 **서비스** |
| 위치 | 내 컴퓨터(로컬) | 인터넷(원격 서버) |
| 역할 | 코드 변경 이력 기록, 버전 관리 | 코드 공유, 협업, 백업 |
| 비유 | 카메라 (사진 찍는 도구) | 클라우드 앨범 (사진 저장/공유) |

Git은 버전 관리 기능이고, GitHub은 그 기능을 활용하는 웹서비스 중 하나다. GitHub 외에 GitLab, Bitbucket 등도 있다.

---

## 3. 중앙 집중식 vs 분산식

중앙 집중식은 모든 코드가 하나의 메인 서버에서만 관리되므로 서버가 다운되면 작업이 불가능하고, 인터넷 없이는 버전 관리를 할 수 없다.

분산식(Git)은 각 개발자의 로컬 컴퓨터에 전체 버전 이력을 복제하여 저장한다. 인터넷 없이도 커밋(버전 저장)이 가능하고, 나중에 원격 서버와 동기화하면 된다.

---

## 4. Git의 세 가지 영역

Git은 파일 상태를 세 영역으로 구분하여 관리한다.

```
Working Directory  →  Staging Area  →  Repository
  (작업 공간)         (대기 공간)       (저장소)
    git add            git commit
```

**Working Directory**는 실제 파일을 수정하는 작업 공간이다. `git status`에서 빨간색으로 표시된다.

**Staging Area**는 다음 버전(커밋)에 포함시킬 파일들을 선택적으로 모아두는 중간 준비 영역이다. `git status`에서 초록색으로 표시된다. A.txt와 B.txt를 모두 수정했더라도 A.txt만 이번 버전에 포함시키고, B.txt는 다음 버전에 포함시키는 것이 가능하다.

**Repository**는 커밋된 버전들이 영구적으로 저장되는 영역이다. 모든 버전과 변경 이력이 `.git` 폴더 안에 기록된다.

> **Tip:** 커밋(commit)은 변경 사항들을 하나의 버전으로 묶어 저장하는 행위다. Git에서 "버전"과 "커밋"은 사실상 같은 의미로 쓰인다.

---

## 5. 로컬 버전 관리 명령어

### 5-1. git init — 저장소 초기화

특정 폴더에서 Git 버전 관리를 시작하겠다고 선언하는 명령어다. 실행하면 `.git` 숨김 폴더가 생성되고, 터미널에 `(master)` 브랜치 표시가 나타난다.

```bash
cd ~/Desktop/my-project
git init
# → Initialized empty Git repository in .../my-project/.git/
```

버전 관리를 해제하려면 `.git` 폴더를 삭제하면 된다. 단, 로컬에만 있는 커밋 이력이 전부 사라지므로 주의해야 한다.

> **주의:** 이미 `git init`한 폴더 내부에서 또 `git init`을 하면 안 된다. 가장 흔한 실수는 바탕화면에서 `git init`을 하고, 그 안의 프로젝트 폴더에서 또 `git init`을 하는 것이다. 버전 관리가 충돌하여 꼬인다. `(master)`가 이미 보이는 경로에서는 절대 다시 `git init`하지 말 것.

### 5-2. git add — Staging Area로 이동

변경된 파일을 Staging Area에 올리는 명령어다.

```bash
git add a.txt          # 특정 파일만 올리기
git add .              # 현재 폴더의 모든 변경 파일 올리기
git add -A             # 프로젝트 전체(위치 무관) 모든 변경 파일 올리기
```

`git add .`은 현재 터미널 위치(pwd) 기준의 파일만 올린다. 하위 폴더에서 실행하면 상위 폴더의 변경 파일은 포함되지 않는다. 위치에 관계없이 전체를 올리려면 `git add -A`를 사용한다.

### 5-3. git commit — 버전 저장

Staging Area에 있는 파일들을 하나의 버전으로 묶어 저장한다.

```bash
git commit -m "커밋 메시지"     # 한 줄 메시지로 커밋
git commit                      # vim 에디터가 열려 긴 메시지 작성 가능
```

> **Tip:** vim 에디터가 열렸을 때: `i` → 입력 모드 → 메시지 작성 → `ESC` → `:wq` → 저장 후 종료. 익숙하지 않으면 `-m` 옵션을 사용하는 것이 편하다.

첫 커밋 시 사용자 정보가 없으면 에러가 발생한다. 아래 명령어로 한 번만 설정하면 이후 모든 커밋에 적용된다.

```bash
git config --global user.email "your@email.com"
git config --global user.name "YourName"
```

### 5-4. git status — 파일 상태 확인

현재 파일들이 어느 영역에 있는지 확인하는 명령어다. Git 작업에서 가장 자주 사용해야 하는 습관적 명령어다.

```bash
git status
# 빨간색: Working Directory (아직 add 안 함)
# 초록색: Staging Area (commit 대기 중)
# nothing to commit: 모든 변경이 커밋 완료됨
```

> **Tip:** `git config --global alias.st status`를 설정하면 `git st`만으로 status를 확인할 수 있다. 가장 자주 치는 명령어이므로 별칭 설정을 권장한다.

### 5-5. git log — 커밋 이력 확인

저장된 커밋 목록을 확인한다.

```bash
git log            # 상세 이력 (작성자, 날짜, 메시지, 해시)
git log --oneline  # 한 줄 요약 (해시 축약 + 메시지)
```

### 5-6. git restore — 변경 사항 되돌리기

Working Directory에서 수정한 내용을 가장 최근 커밋 상태로 되돌린다. 해당 파일의 변경 사항만 사라지고 다른 파일에는 영향이 없다.

```bash
git restore sample.txt  # sample.txt의 변경 사항을 최신 커밋 기준으로 롤백
```

> **주의:** `restore`를 실행하면 수정한 코드가 복구 불가능하게 사라진다. "지금까지 작업한 게 완전히 필요 없을 때"만 사용할 것.

---

## 6. 전체 워크플로우 (로컬)

하나의 사이클을 정리하면 다음과 같다.

```bash
# 1. 파일 생성/수정 (Working Directory에서 작업)
touch a.txt

# 2. 상태 확인 (습관!)
git status              # → 빨간색 a.txt

# 3. Staging Area로 이동
git add a.txt

# 4. 상태 확인 (습관!)
git status              # → 초록색 a.txt

# 5. 커밋 (버전 저장)
git commit -m "a.txt 파일 추가"

# 6. 상태 확인 (습관!)
git status              # → nothing to commit, working tree clean

# 7. 이력 확인
git log --oneline
```

이 사이클을 반복하는 것이 Git의 전부다.

---

## 7. 원격 저장소 (Remote Repository)

### 7-1. GitHub 레포지토리 생성

GitHub에서 `New Repository`를 클릭하여 원격 저장소를 만든다. 생성 후 표시되는 HTTPS URL이 원격 저장소의 주소다.

### 7-2. git remote — 원격 저장소 연결

로컬 저장소와 원격 저장소를 연결한다.

```bash
git remote add origin https://github.com/username/repo.git
```

`origin`은 원격 저장소 URL의 별명이다. 긴 URL을 매번 입력할 수 없으므로 별명으로 등록하는 것이다. 관례적으로 `origin`을 사용하지만 다른 이름도 가능하다.

```bash
git remote -v    # 연결된 원격 저장소 확인
```

### 7-3. git push — 원격에 업로드

로컬에서 커밋한 버전들을 원격 저장소에 업로드한다.

```bash
git push origin master
```

최초 push 시 GitHub 계정 인증(브라우저 로그인)이 필요하다.

> **주의:** push가 안 될 때 가장 먼저 `git status`를 확인해야 한다. 대부분 add나 commit을 빠뜨린 경우다. push는 커밋된 내용만 올리므로, 커밋이 없으면 올릴 것도 없다.

### 7-4. git clone — 원격 저장소 복제

원격 저장소의 전체 내용을 로컬에 최초로 가져올 때 사용한다.

```bash
git clone https://github.com/username/repo.git
```

clone을 받으면 이미 `git init`이 되어 있으므로 별도로 init할 필요가 없다.

### 7-5. git pull — 변경 사항 가져오기

이미 clone한 저장소에서 이후 추가된 변경 사항만 가져올 때 사용한다.

```bash
git pull origin master
# 또는 간단히
git pull
```

### 7-6. clone vs pull 정리

| 명령어 | 용도 | 시점 |
|--------|------|------|
| clone | 원격 저장소 전체를 복제 | 프로젝트에 처음 합류할 때 (1회) |
| pull | 변경 사항만 가져오기 | clone 이후 업데이트할 때 (반복) |

---

## 8. 협업 워크플로우

```
[개발자 A: 로컬]                    [GitHub: 원격]                  [개발자 B: 로컬]
                                       │
 작업 → add → commit → push ──────→   │
                                       │  ←────── clone (최초 1회)
 작업 → add → commit → push ──────→   │
                                       │  ←────── pull (이후 업데이트)
```

A가 코드를 push하면, B는 최초에 clone으로 전체를 받고 이후에는 pull로 변경분만 받는다. B가 작업한 것도 push로 올리면 A가 pull로 받을 수 있다. 이 과정을 반복하는 것이 Git 기반 협업의 핵심이다.

---

## 9. .gitignore — 추적 제외 설정

프로젝트에서 Git이 추적하지 않아야 할 파일을 지정하는 설정 파일이다. API 키, 비밀번호, 환경 변수 파일 등 외부에 공개되면 안 되는 정보가 대표적인 대상이다.

```bash
# .gitignore 파일 생성 후 제외할 파일명 입력
touch .gitignore

# .gitignore 내용 예시
.env           # 환경 변수 파일
secret.txt     # 비밀 정보 파일
*.log          # 모든 .log 파일
__pycache__/   # 파이썬 캐시 폴더
```

`.gitignore`에 등록된 파일은 `git status`에 아예 표시되지 않는다.

> **주의:** 이미 한 번이라도 커밋된 파일은 나중에 `.gitignore`에 추가해도 추적이 해제되지 않는다. 이 경우 `git rm --cached 파일명` 명령으로 Git 캐시에서 먼저 제거해야 한다.

> **Tip:** 프로젝트 종류에 따라 제외해야 할 파일 목록이 다르다. [gitignore.io](https://www.toptal.com/developers/gitignore)에서 Django, Node, Python 등을 검색하면 자동으로 `.gitignore` 내용을 생성해준다.

---

## 10. 명령어 요약

| 명령어 | 기능 |
|--------|------|
| `git init` | 현재 폴더를 Git 저장소로 초기화 |
| `git add .` | 변경 파일을 Staging Area로 이동 |
| `git commit -m "msg"` | Staging Area → 커밋(버전) 저장 |
| `git status` | 파일 상태 확인 (가장 자주 사용) |
| `git log --oneline` | 커밋 이력 한 줄 확인 |
| `git restore 파일명` | 파일 변경 사항 되돌리기 |
| `git remote add origin URL` | 원격 저장소 연결 |
| `git push origin master` | 로컬 커밋을 원격에 업로드 |
| `git pull origin master` | 원격 변경 사항을 로컬에 다운로드 |
| `git clone URL` | 원격 저장소 전체 복제 |
| `git config --global alias.st status` | status 별칭 설정 |

---

## 정리

* Git은 변경 사항만 기록하는 버전 관리 시스템이고, GitHub은 원격 저장소 서비스다
* 파일은 Working Directory → Staging Area → Repository 순서로 이동한다
* `git status`는 모든 작업 전후에 습관적으로 확인해야 한다
* 커밋은 Staging Area에 있는 파일만 버전으로 저장한다 — add를 빠뜨리면 push도 안 된다
* `git init`한 폴더 내부에서 또 `git init`하면 버전 관리가 꼬인다
* 원격 연결은 `git remote add origin URL`, 업로드는 `push`, 다운로드는 `pull`
* 최초 합류 시 `clone`, 이후 업데이트는 `pull`을 사용한다
* `.gitignore`는 커밋 전에 설정해야 효과가 있다 — 이미 커밋된 파일은 `git rm --cached`로 먼저 해제
