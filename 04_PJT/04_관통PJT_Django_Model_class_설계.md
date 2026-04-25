# [관통 PJT] Django Model class 설계 — Git Branch & GitFlow 전략

> 📌 핵심 키워드: #Branch #Merge #MergeConflict #GitFlow #PullRequest #Fork #Stash #HEAD

---

## 학습 목표

* Git 브랜치의 개념과 사용 이유를 설명할 수 있다
* `git branch`, `git switch`, `git stash` 명령어를 실습으로 익힌다
* Fast Forward Merge와 3-way Merge의 차이를 이해하고 Merge Conflict를 해결할 수 있다
* GitFlow 전략(master, dev, feature, hotfix, release)에 따라 브랜치를 구성하고 협업 워크플로우를 적용할 수 있다
* Pull Request와 Fork를 활용하여 코드 리뷰 및 오픈소스 기여 방법을 이해한다

---

## 1. Git Branch 개념

### 1-1. 브랜치란

브랜치(Branch)는 나뭇가지처럼 여러 갈래로 **작업 공간을 나누어 독립적으로 작업**할 수 있도록 도와주는 도구다. 마치 프로젝트 폴더를 `컨트롤 C, V`로 복사해놓고 그 안에서 자유롭게 개발하는 것과 같다. 원본(master/main)은 건드리지 않고, 복사본에서 마음껏 망가뜨려도 원본은 멀쩡하다.

### 1-2. 브랜치를 써야 하는 이유

실무에서 브랜치가 반드시 필요한 상황들이 있다.

- 팀원이 같은 파일의 같은 부분을 동시에 수정할 때 코드가 엉키는 상황
- 로그인 기능 개발 도중 갑자기 버그 수정 요청이 들어오는 상황 (작업 중인 코드를 어떻게 처리할 것인가)
- 버그를 고쳤는데 그 수정이 또 다른 버그를 낳을 수 있는 상황

### 1-3. master / main 브랜치

Git을 처음 초기화하면 자동으로 생성되는 기본 브랜치다. **프로젝트의 가장 최신 배포 버전** 혹은 현재 운영 중인 코드를 의미한다. `master`라는 단어가 slave와 연관된 부정적인 뉘앙스가 있어 최신 Git 버전에서는 `main`으로 변경되었다.

---

## 2. Git Branch 명령어

### 2-1. git branch

브랜치를 조회하거나 생성하거나 삭제하는 명령어다.

```bash
# 로컬 브랜치 목록 조회
git branch

# 원격 저장소 브랜치 목록 조회
git branch -r

# 브랜치 생성
git branch <브랜치명>

# 브랜치 삭제 (병합된 브랜치만 가능)
git branch -d <브랜치명>
```

`git branch`는 로컬, `git branch -r`은 GitHub(원격 저장소)의 브랜치를 보여준다. 로컬에서 브랜치를 10개 만들어도 원격에 push하지 않으면 다른 팀원들은 볼 수 없다.

브랜치를 지우지 않고 방치하면 나중에 목록이 많아져 어떤 브랜치가 최신인지 헷갈리고, 작업이 완료된 브랜치인지 파악하기 어려워진다. 팀의 그라운드 룰에 따라 병합 완료 후 삭제 여부를 결정한다.

### 2-2. HEAD의 개념

HEAD는 **현재 이 프로젝트가 어느 커밋을 가리키고 있는지**를 나타내는 포인터다. 커밋 자체는 전체 코드가 아니라 변경된 내용(diff)만 저장하므로, HEAD가 어느 커밋을 가리키느냐에 따라 프로젝트의 현재 상태가 결정된다.

### 2-3. git switch

현재 브랜치에서 다른 브랜치로 이동하는 명령어다. 브랜치를 이동하면 HEAD가 해당 브랜치의 최신 커밋을 가리키도록 변경되며, 워킹 디렉토리의 파일도 그 커밋 상태로 바뀐다.

```bash
# 브랜치 이동
git switch <브랜치명>

# 브랜치 생성 후 즉시 이동 (-c: create)
git switch -c <브랜치명>
```

**⚠️ 브랜치 전환 전 반드시 확인해야 할 것:** 워킹 디렉토리의 파일이 모두 버전 관리되고 있는지 확인한다. 즉, 작업 중인 파일을 `git add`하여 스테이징에 올리거나 커밋까지 해야 전환이 가능하다.

#### 브랜치 전환 시 untracked 파일 주의사항

한 번도 `git add`(추적 등록)를 하지 않은 untracked 파일은 브랜치를 옮겨도 그대로 따라온다. Git 입장에서 해당 파일을 버전 관리 대상으로 보지 않기 때문이다. 반대로, 이미 추적 중인 파일을 수정한 상태에서 전환을 시도하면 에러가 발생한다.

```
error: Your local changes to the following files would be overwritten by checkout
```

이 경우 두 가지 선택지가 있다. 첫째는 변경 내용을 `git add` + `git commit`으로 커밋하고 이동하는 것이다. 둘째는 `git stash`를 사용하여 작업 중인 내용을 임시 저장하고 이동하는 것이다.

### 2-4. git checkout (레거시)

과거에 브랜치 이동, 스테이징 취소, 특정 커밋 이동 등 다양한 기능을 하나의 명령어로 처리하던 명령어다. 하나의 명령어가 너무 많은 역할을 하는 것이 좋지 않아 Linus Torvalds가 `git switch`(브랜치 이동)와 `git restore`(파일 복원)로 분리했다. 공식 문서나 레거시 코드에서 여전히 자주 등장하므로 기억해두어야 한다.

```bash
git checkout <브랜치명>  # git switch <브랜치명> 과 동일
```

---

## 3. git stash

### 3-1. stash 개념

stash는 영어로 '창고'를 의미한다. **작업 중인 코드를 임시로 창고에 보내두는** 기능이다. 브랜치를 이동해야 하는데 작업 중인 내용이 있어 커밋하기 어려울 때 사용한다.

### 3-2. stash 기본 명령어

```bash
# 작업 중인 내용을 창고에 저장
git stash

# untracked 파일까지 함께 stash
git stash -u

# stash 목록 확인
git stash list

# 가장 최근 stash 꺼내기 (스택 구조 - 가장 마지막이 먼저 나옴)
git stash pop

# 특정 stash 꺼내기
git stash pop stash@{0}
```

기본 `git stash`는 한 번이라도 커밋된 적 있는 파일의 변경사항만 stash한다. 한 번도 커밋된 적 없는 untracked 파일까지 stash하려면 `-u` 옵션을 함께 사용한다.

stash한 상태에서 같은 파일을 작업하고 나중에 pop하면 충돌(conflict)이 발생할 수 있다. stash는 로컬 창고에만 저장되며 원격 저장소에 올라가지 않는다.

---

## 4. git merge

### 4-1. merge 개념 및 수신 브랜치

merge는 두 브랜치를 하나로 합치는 작업이다. 브랜치를 따서 작업하는 것 자체는 어렵지 않다. **합칠 때 문제가 생긴다.**

```bash
git merge <합칠 브랜치명>
```

**수신 브랜치(현재 위치)를 반드시 확인해야 한다.** 내가 현재 있는 브랜치가 코드를 받아들이는 쪽(수신 브랜치)이 된다. 예를 들어 `master` 브랜치에서 `git merge login`을 실행하면 login 브랜치의 코드가 master로 들어온다. 반대로 login 브랜치에서 `git merge master`를 실행하면 master의 코드가 login으로 들어온다.

| 구분 | 설명 |
|------|------|
| 수신 브랜치 | 현재 내가 위치한 브랜치 (코드를 받는 쪽) |
| 병합 대상 브랜치 | `git merge` 명령어 뒤에 입력하는 브랜치 (코드를 주는 쪽) |

### 4-2. Fast Forward Merge

한쪽 브랜치에서만 커밋이 진행되었을 때 발생하는 방식이다. 실제로 병합하는 게 아니라 마스터 브랜치의 커밋 위치를 앞으로 당겨오는(빨리 감기) 방식이다.

예를 들어 master가 C2에 머물러 있고 hotfix가 C2 → C3 → C4로 진행되었을 때, master에서 `git merge hotfix`를 실행하면 master의 HEAD가 C4로 이동한다. 별도의 병합 커밋이 생기지 않는다.

### 4-3. 3-way Merge

두 브랜치가 각자 독립적으로 커밋을 진행한 경우, 공통 조상 커밋을 기준으로 병합하는 방식이다. 이 경우 새로운 병합 커밋이 자동으로 생성된다.

```bash
# 마스터 브랜치에서 실행
git merge hotfix
```

서로 다른 파일을 수정했다면 자동으로 병합된다. 같은 파일의 같은 부분을 수정한 경우 충돌(conflict)이 발생한다.

병합 전에는 항상 원격 저장소를 최신화(`git pull`)해야 한다. 내 로컬 master가 1.0인데 원격 master는 1.1로 올라가 있는 상태에서 병합하면 의도하지 않은 결과가 생긴다.

### 4-4. Merge Conflict 해결

동일한 파일의 동일한 부분을 두 브랜치가 각자 수정한 경우 Git은 어떤 코드가 옳은지 판단할 수 없어 사용자에게 판단을 넘긴다.

```
<<<<<<< HEAD (Current Change)
마스터에서 수정한 내용
=======
핫픽스에서 수정한 내용
>>>>>>> hotfix (Incoming Change)
```

충돌 발생 시 처리 방법은 다음과 같다.

| 선택지 | 설명 |
|--------|------|
| Accept Current Change | 현재 브랜치(수신 브랜치)의 코드만 사용 |
| Accept Incoming Change | 병합 대상 브랜치의 코드만 사용 |
| Accept Both Changes | 두 코드 모두 유지 |
| Compare Changes | 두 코드의 차이 비교 |

충돌을 직접 수정할 때는 `<<<`, `===`, `>>>` 기호를 모두 제거하고 원하는 코드만 남긴다. 수정이 완료되면 `git add` → `git commit`으로 병합 커밋을 생성한다. Vim 에디터가 열리면 `:wq`로 저장하고 닫는다.

> ⚠️ 충돌 해결은 반드시 당사자를 불러서 함께 진행한다. 팀장이라도 상대방 코드를 마음대로 지우면 안 된다.

**충돌 예방이 최선이다.** 애초에 담당자별로 파일과 함수를 명확히 분리하는 것이 가장 좋은 방법이다. 또한 충돌의 책임은 보통 늦게 개발한 사람(merge를 나중에 하는 사람)에게 넘어가므로 빠르게 개발하고 빠르게 merge하는 것이 유리하다.

---

## 5. 원격 저장소와 브랜치

### 5-1. origin의 의미

`origin`은 원격 저장소 URL의 **별명(alias)**이다. URL을 매번 타이핑할 수 없으므로 짧은 이름으로 등록해두는 것이다.

```bash
git remote add origin https://github.com/...
# 이후 origin만 입력하면 저 URL을 가리킴
```

### 5-2. 브랜치를 원격 저장소에 push

로컬에서 만든 브랜치는 push하지 않으면 원격 저장소에 없다. 팀원들은 로컬에만 있는 브랜치를 볼 수 없다.

```bash
# 브랜치를 원격에 push
git push origin <브랜치명>

# 예시: hotfix 브랜치 push
git push origin hotfix
```

---

## 6. Pull Request (PR)

### 6-1. PR의 목적

Pull Request는 내가 별도 브랜치에서 작업한 내용을 master(또는 dev)에 병합해달라고 공식적으로 요청하는 것이다. **핵심 목적은 코드 리뷰**다.

신입 개발자가 검토도 받지 않고 배포 중인 마스터 브랜치에 직접 merge하면 심각한 문제가 생길 수 있다. PR을 통해 총 책임자가 코드 변경사항을 파악하고 피드백을 제공하며 코드 품질을 유지한다.

### 6-2. PR 워크플로우

```
1. 로컬에서 feature 브랜치 생성
   git switch -c feature/login

2. 기능 개발 후 commit
   git add .
   git commit -m "feat: 로그인 기능 구현"

3. 로컬 브랜치를 원격에 push
   git push origin feature/login

4. GitHub에서 PR 생성
   - base: main(또는 dev) ← compare: feature/login

5. 코드 리뷰 → 승인(Approve) 또는 반려(Request changes)

6. 승인 후 Merge Pull Request 클릭

7. 병합된 브랜치 삭제 (팀 그라운드 룰에 따름)

8. 나머지 팀원들은 git pull로 최신화
```

### 6-3. GitHub Branch Rules 설정

GitHub 저장소의 Settings → Rules → Rulesets에서 브랜치 보호 규칙을 설정할 수 있다.

| 규칙 | 설명 |
|------|------|
| 리뷰어 n명 이상 승인 필요 | n명이 approve해야만 merge 가능 |
| 로컬에서 직접 merge 차단 | PR을 통해서만 병합 허용 |

---

## 7. GitFlow 전략

### 7-1. GitFlow란

GitFlow는 프로젝트 운영 시 브랜치를 어떻게 관리할 것인지에 대한 **협업 전략이자 관리 전략**이다. 정답이 아니며 회사마다 유연하게 변형하여 사용한다. 그러나 큰 틀은 아래를 벗어나지 않는다.

### 7-2. 5가지 브랜치 역할

#### master 브랜치

실제로 배포되어 **운영 중인 코드**가 들어있는 브랜치다. 누구도 이 브랜치에서 직접 코드를 수정하면 안 된다. 다른 브랜치에서 충분한 테스트와 병합 과정을 거친 후 최종 결과물만 master에 넘긴다.

> ⚠️ master 브랜치에서 직접 코드 작업은 팀장도 절대 하지 않는다.

#### dev(develop) 브랜치

master와 코드 구성이 동일하지만 **개발 테스트 환경** 역할을 하는 브랜치다. 네트워크 설정, 환경 변수 등 배포 환경과 관련된 테스트를 수행하는 공간이다. feature 브랜치들은 dev 브랜치로부터 따고, 작업 완료 후 dev 브랜치로 합친다.

- 개발은 master가 아닌 dev에서 파생
- dev와 master는 항상 버전이 동일하게 유지
- hotfix가 master에 반영되면 반드시 dev에도 반영

#### feature 브랜치

각 개발자가 **기능을 개발하는 브랜치**다. 브랜치명 앞에 `feature/` 또는 `f/`를 붙이는 것이 관례다(회사마다 상이).

```bash
git switch -c feature/login   # 또는 f/login
```

- dev 브랜치에서 파생
- 개발 완료 후 dev 브랜치로 병합
- 병합 전에 반드시 dev 최신화 후 내 feature 브랜치에서 dev를 먼저 병합하여 충돌 해결

**feature 브랜치 병합 순서:**

```
1. feature 브랜치에서 dev 최신화
   git switch feature/login
   git merge dev  # 내 브랜치가 망가져도 여기서 해결

2. 충돌 해결 후 commit

3. PR 올리거나 dev 브랜치로 merge
   git switch dev
   git merge feature/login
```

#### release 브랜치

**배포 대기 공간** 역할을 하는 브랜치다. 개발이 완료된 코드를 실제 배포 날짜 전까지 임시로 보관한다. 개발 완료 후 배포까지 2주가 남았을 때 그 2주 동안 dev 브랜치를 멈출 수 없으므로 release 브랜치를 따로 파두는 것이다. 사용하지 않는 팀도 있다.

#### hotfix 브랜치

**운영 중인 서버에서 버그가 났을 때** 즉시 수정하기 위한 브랜치다. 브랜치명 앞에 `hotfix/` 또는 `h/`를 붙이는 것이 관례다.

- master 브랜치에서 파생 (운영 코드가 거기 있으므로)
- 버그 수정 후 master에 즉시 병합
- master에 반영된 내용을 dev에도 반드시 병합 (빠뜨리면 나중에 버그 재발)

### 7-3. GitFlow 전체 흐름 요약

| 브랜치 | 역할 | 파생 출처 | 병합 대상 |
|--------|------|-----------|-----------|
| master | 운영 배포 코드 | — | — |
| dev | 개발 테스트 환경 | master | master |
| feature/xxx | 기능 개발 | dev | dev |
| release | 배포 대기 | dev | master, dev |
| hotfix/xxx | 긴급 버그 수정 | master | master, dev |

---

## 8. Fork & 오픈소스 기여

### 8-1. Fork란

Fork는 소유권이 없는 원격 저장소를 **내 계정으로 복제**해오는 것이다. Collaborator로 등록된 것과 달리, 원본 저장소에 직접 push할 권한이 없는 상태에서 기여할 때 사용한다.

### 8-2. 오픈소스 기여 과정

```
1. GitHub에서 오픈소스 레포의 Fork 버튼 클릭
   → 내 계정으로 레포가 복제됨

2. 내 레포에서 git clone

3. 새 브랜치 파서 기능 개발 또는 버그 수정

4. 내 원격 레포(fork한 레포)에 push

5. 원본 레포에 PR 제출

6. 메인테이너가 승인하면 코드가 반영되고 컨트리뷰터 등록
```

기업에서 오픈소스 기여자를 선호하는 이유는 협업 방법을 알고, 코딩 스타일이 다른 개발자들에게 인정받았으며, 오픈소스에 적극적으로 관심을 가지고 있음을 증명하기 때문이다.

---

## 9. GitHub Issues 활용

Issues는 프로젝트와 관련된 **할 일, 버그, 기능 요청** 등을 팀원들과 공유하는 공간이다. 노션이나 메모장 대신 해당 레포에서 직접 이슈를 관리하면 PR과 연결하여 추적이 쉬워진다.

실제 회사에서 개발자들은 Issues에서 자신이 담당할 작업을 가져와서 진행하고, 팀장은 Issues를 보면서 현재 누가 무엇을 작업 중인지 파악한다. 이슈에 `bug`, `feature`, `enhancement` 같은 라벨을 붙여 분류하고, PR을 올릴 때 해당 이슈를 연결할 수 있다.

---

## 10. 관통 PJT 안내

오늘 관통 프로젝트는 **Django Model 및 ORM을 처음부터 직접 구현**하는 것이다. Django 프로젝트를 팀원끼리 협력하여 구현하면서 오늘 배운 GitFlow 전략을 직접 적용해 본다.

추천 실습 항목은 다음과 같다.

- 팀원을 Collaborator로 초대하고 GitHub Branch Rules 설정
- 터미널 직접 merge를 막고 반드시 PR을 통해서만 병합되도록 설정
- PR 승인 조건에 리뷰어 2명 이상 필요 조건 추가
- 실제로 PR 올리고 코드 리뷰 및 반려 경험해보기

---

## 📋 핵심 개념 정리

| 개념 | 설명 | 예시/명령어 |
|------|------|-------------|
| Branch | 독립된 작업 공간 | `git branch feature/login` |
| HEAD | 현재 커밋을 가리키는 포인터 | `git log --oneline` |
| git switch | 브랜치 이동 | `git switch <브랜치명>` |
| git switch -c | 브랜치 생성 + 이동 | `git switch -c <브랜치명>` |
| git stash | 작업 중 코드 임시 저장 | `git stash` / `git stash -u` |
| git stash pop | stash에서 코드 꺼내기 | `git stash pop` |
| git merge | 두 브랜치 병합 | `git merge <브랜치명>` |
| Fast Forward Merge | 한쪽만 커밋 진행 시 HEAD 이동 | conflict 없음 |
| 3-way Merge | 양쪽 모두 커밋 진행 시 병합 커밋 생성 | conflict 가능 |
| Merge Conflict | 동일 파일 동일 부분 수정 시 충돌 | 당사자와 함께 수동 해결 |
| origin | 원격 저장소 URL 별명 | `git push origin <브랜치>` |
| Pull Request (PR) | 코드 리뷰 및 병합 요청 | GitHub에서 생성 |
| GitFlow | 브랜치 관리 전략 | master / dev / feature / hotfix / release |
| Fork | 타인 레포를 내 계정으로 복제 | 오픈소스 기여 시 사용 |
| Issues | 버그/기능 요청 관리 | PR과 연결 가능 |

---

## 참고사항 (수업 후 읽기)

- Git 공식 문서 브랜치 섹션: https://git-scm.com/book/ko/v2
- GitFlow 원본 글 (Vincent Driessen): https://nvie.com/posts/a-successful-git-branching-model/
- CPython 오픈소스 기여 가이드: https://devguide.python.org
- `git stash drop` 명령어로 필요 없는 stash를 개별 삭제할 수 있다
- `git log --oneline --all --graph` 옵션으로 모든 브랜치의 커밋 히스토리를 그래프로 시각화할 수 있다
- `git checkout`은 레거시 명령어이지만 여전히 동작한다. 공식 문서에서 자주 보이므로 인지해두어야 한다
