# [PJT] 데이터 수집 — Git 관리 기술, 파일 시스템, 생성형 AI 활용 데이터 요약

> **핵심 키워드:** #Git #revert #reset #restore #rm #reflog #soft #mixed #hard #pathlib #Path #경로객체 #파일시스템 #인코딩 #UTF-8 #with문 #glob #iterator #generator #OpenAI #API키 #dotenv #requirements #TIL요약

---

## 학습 목표

* Git revert, reset, restore, rm 명령어의 역할과 차이를 이해하고, 필요할 때 검색하여 활용
* reset의 soft / mixed / hard 옵션별 동작 차이와 위험성 파악
* pathlib의 Path 클래스로 경로를 객체로 다루고, 디렉토리·파일 생성·읽기·탐색 수행
* 인코딩(UTF-8)의 개념과 한글 파일 처리 시 필요한 이유 이해
* OpenAI API를 코드로 호출하고, .env와 requirements.txt로 키·패키지를 관리하는 실전 워크플로우 습득

---

## 1. Git 관리 기술 — 개요

이번 파트는 **암기가 아니라 존재 여부를 아는 것**이 목표다. 커밋을 되돌리거나 취소하는 명령어들은 자주 쓰지 않기 때문에 외우려 하지 말고, 필요한 상황이 왔을 때 "이런 기능이 있었다"를 기반으로 검색해서 사용하면 된다.

> **강사님 강조**: 여기 나오는 복구·취소 기술은 최후의 수단이다. 이걸 쓸 상황을 안 만드는 게 정답이고, 커밋하기 전에 생각하는 습관이 가장 중요하다.

### 1-1. git revert — 특정 커밋을 없었던 일로

revert는 특정 커밋에서 수행한 작업을 취소하는 명령어다. 단, 취소한 사실 자체가 **새로운 커밋으로 기록**되므로 이력이 손실되지 않는다.

```bash
git revert <커밋ID>
```

실행하면 vim 에디터가 열리며 "Revert 커밋명"이라는 커밋 메시지가 자동 작성된다. `:wq`로 저장하면 해당 커밋의 작업이 제거되고, 그 제거 자체가 로그에 남는다.

핵심: 원본 커밋은 삭제되지 않고, 되돌린 행위만 새 커밋으로 추가된다. Git에서 기록 없이 무언가를 없애는 것은 불가능하다.

> **강사님 강조**: 실수하면 몰래 revert로 날려버리고 싶겠지만, 숨기다 걸리면 사건만 커진다. 실수했으면 빨리 팀장에게 말하는 게 최선이다. 책임지라고 있는 게 팀장이다.

### 1-2. git reset — 과거 시점으로 롤백

reset은 특정 커밋 시점으로 **되돌아가는** 명령어다. 되돌아간 커밋 이후의 모든 커밋이 삭제된다.

```bash
git reset [옵션] <커밋ID>
```

옵션에 따라 삭제된 커밋의 코드가 어디에 남는지가 달라진다.

| 옵션 | 삭제된 코드의 위치 | git status 색상 | 설명 |
|------|-------------------|----------------|------|
| `--soft` | Staging Area | 초록색 | 바로 다시 커밋 가능 |
| `--mixed` (기본값) | Working Directory | 빨간색 | add부터 다시 해야 함 |
| `--hard` | **완전 삭제** | 표시 없음 | 복구 불가, 가장 위험 |

옵션 없이 `git reset <커밋ID>`만 입력하면 mixed가 기본 적용된다.

> **강사님 주의**: `--hard`는 파일 시스템에서 직접 삭제하므로 복구가 불가능하다. 솔로 프로젝트에서 "코드 완전히 망했다" 싶을 때만 사용하고, 협업 환경에서는 절대 쓰지 말 것. 다른 사람이 원본을 갖고 있으면 합치는 과정에서 충돌(conflict)이 발생한다.

### 1-3. git reflog — 삭제된 기록 복구

`git log`에서는 reset으로 삭제된 커밋이 보이지 않지만, `git reflog`에는 모든 HEAD 이동 기록이 남아 있다. 여기서 복구하고자 하는 커밋 ID를 찾아 다시 reset하면 복원할 수 있다.

```bash
git reflog                         # 모든 이동 기록 확인
git reset --hard <복구할_커밋ID>    # 해당 시점으로 복원
```

### 1-4. git restore — 파일 변경 사항 되돌리기

Working Directory나 Staging Area에 있는 파일의 변경 사항을 이전 커밋 상태로 되돌린다.

```bash
git restore <파일명>                # Working Directory → 최신 커밋 상태로 복원
git restore --staged <파일명>       # Staging Area → Working Directory로 내림 (unstage)
```

`restore`는 이전 버전으로 덮어쓰므로 수정한 코드가 영구적으로 사라진다. 사용 전에 반드시 확인할 것.

> **강사님 팁**: `git status`를 치면 Git이 친절하게 restore 사용법을 안내해준다. 명령어를 외울 필요 없이 status 메시지를 읽으면 된다.

### 1-5. git rm --cached — 추적 중단 (파일은 유지)

이미 커밋된 파일은 나중에 `.gitignore`에 추가해도 추적이 해제되지 않는다. Git이 이미 해당 파일을 "보고 있는" 상태이기 때문이다. 이때 `git rm --cached`로 추적만 중단하고 파일은 남길 수 있다.

```bash
git rm --cached <파일명>    # Git 추적 중단, 파일은 Working Directory에 유지
git rm <파일명>             # Git 추적 중단 + 파일 자체도 삭제 (주의)
```

추적을 중단한 후 다시 add → commit하면, 이후부터는 `.gitignore`가 정상 적용된다.

> **강사님 강조**: `git add .`으로 전체를 올린 뒤 뒤늦게 `.gitignore`에 추가하는 실수는 누구나 한다. 그때 `git rm --cached`를 떠올리면 된다. GitHub에 한번 올라간 기록은 커밋을 뒤져보면 남아 있으므로, 키값 같은 민감 정보는 애초에 올리지 않는 것이 최선이다.

---

## 2. 파일 시스템 — pathlib

### 2-1. pathlib을 사용하는 이유

파이썬에서 파일 경로는 기본적으로 문자열(`str`)이다. 문자열 상태에서 파일명·확장자·상위 디렉토리 등을 추출하려면 `split`, `join` 같은 문자열 처리를 직접 해야 하므로 번거롭다.

`pathlib`은 **경로 자체를 객체로 다루는** 파이썬 표준 라이브러리다. 경로 객체에 내장된 메서드와 속성을 사용하면 파일 생성·읽기·탐색 같은 작업을 간결하게 수행할 수 있다.

```python
from pathlib import Path

# 현재 작업 디렉토리
current = Path.cwd()

# 홈 디렉토리
home = Path.home()

# 임의 경로를 객체로 생성
p = Path("/home/user/documents/file.txt")
```

### 2-2. 경로 정보 추출

경로 객체에서 파일명, 확장자, 이름(확장자 제외)을 바로 꺼낼 수 있다.

```python
p = Path("/home/user/documents/file.txt")

p.name      # 'file.txt'  — 파일명 + 확장자
p.stem      # 'file'      — 이름만 (줄기)
p.suffix    # '.txt'      — 확장자만
```

경로 결합은 `/` 연산자로 간단히 처리한다.

```python
base = Path("documents")
full = base / "subfolder" / "file.txt"
# PosixPath('documents/subfolder/file.txt')
```

### 2-3. 디렉토리·파일 생성

```python
# 디렉토리 생성 (exist_ok=True → 이미 있어도 에러 없음)
new_dir = Path("new_directory")
new_dir.mkdir(exist_ok=True)

# 파일 생성 및 내용 작성
new_file = new_dir / "new.md"
new_file.write_text("새로 만들기", encoding="utf-8")
```

`exist_ok=False`(기본값)이면 이미 존재하는 디렉토리에 mkdir을 호출할 때 에러가 발생한다.

### 2-4. 인코딩 (UTF-8)

인코딩은 사람이 사용하는 문자를 컴퓨터가 이해할 수 있는 바이트로 변환하는 규칙이다. UTF-8은 전 세계 모든 문자를 4바이트 이내로 표현할 수 있는 유니코드 표준이다.

한글을 파일에 쓸 때 `encoding="utf-8"`을 지정하지 않으면 운영체제 기본 인코딩이 적용되어 한글이 깨질 수 있다. 파이썬에서 한글 파일을 다룰 때는 반드시 UTF-8을 명시한다.

```python
# 인코딩 지정 안 하면 한글이 깨질 수 있음
new_file.write_text("안녕하세요", encoding="utf-8")
```

### 2-5. with문을 이용한 파일 읽기·쓰기

`with`는 파일을 안전하게 열고 닫는 것을 보장하는 파이썬 구문이다. `with` 블록이 끝나면 자동으로 파일이 닫히므로, `close()`를 별도로 호출할 필요가 없다.

```python
# 파일에 여러 줄 추가 (append 모드)
with open(new_file, "a", encoding="utf-8") as file:
    file.write("first line\n")
    file.write("second line\n")
    file.write("third line\n")
```

`open()`의 두 번째 인자는 모드를 나타낸다. `"a"`는 append(추가), `"r"`은 read(읽기), `"w"`는 write(덮어쓰기)다. `as file`에서 `file`은 열린 파일 객체의 별명(alias)이다.

### 2-6. 파일 목록 탐색과 glob

```python
current_path = Path(".")

# 현재 디렉토리의 모든 항목 순회
for item in current_path.iterdir():
    print(item)

# 특정 확장자만 탐색 (glob)
for py_file in current_path.glob("*.py"):
    print(py_file)

# 하위 디렉토리까지 재귀 탐색 (rglob)
for md_file in current_path.rglob("*.md"):
    print(md_file)
```

`glob`의 `*`(에스터리스크)는 와일드카드로, 어떤 문자든 상관없이 매칭된다. `rglob`은 recursive glob으로 하위 폴더까지 탐색한다.

`iterdir()`과 `glob()`은 **제너레이터(generator)** 를 반환한다. 제너레이터는 데이터를 한꺼번에 메모리에 올리지 않고, for문이나 `list()`로 소비할 때 하나씩 불러오는 방식(lazy evaluation)이다. `map` 객체도 같은 원리다.

### 2-7. 파일 읽기

```python
file_path = Path("example.txt")

# 전체 내용을 한 번에 읽기
content = file_path.read_text(encoding="utf-8")

# with문으로 한 줄씩 읽기
with open(file_path, "r", encoding="utf-8") as f:
    line = f.readline()       # 한 줄 읽기
    lines = f.readlines()     # 모든 줄을 리스트로 읽기
```

> **강사님 팁**: pathlib은 암기하는 영역이 아니다. "파이썬에서 경로를 객체로 관리하는 표준 라이브러리가 있었다"는 정도만 기억하고, 필요할 때 검색해서 활용하면 충분하다.

---

## 3. 생성형 AI를 활용한 데이터 요약

### 3-1. OpenAI API 초기화

OpenAI API는 ChatGPT를 코드로 사용할 수 있는 통로다. 외부 패키지이므로 pip으로 설치해야 한다.

```bash
pip install openai
```

API를 사용하려면 OpenAI에서 발급받은 **API 키**가 필요하다. 키는 유료이며, 유출되면 키 소유자에게 요금이 부과되므로 반드시 안전하게 관리해야 한다.

> **강사님 강조**: 공식 문서(platform.openai.com)를 직접 보는 습관을 잡아야 한다. GPT에게 "OpenAI API 사용법 알려줘"라고 하면 옛날 레거시 코드가 나온다. AI 기술은 빠르게 업데이트되므로 최신 코드는 공식 문서에서만 확인할 수 있다.

### 3-2. .env로 API 키 관리

API 키를 코드에 직접 작성하면 GitHub에 올릴 때 키가 노출된다. GitHub은 키값이 포함된 push를 자동 차단하기도 한다.

안전한 관리 방법은 `.env` 파일에 키를 저장하고, `.gitignore`에 `.env`를 추가하는 것이다.

```bash
# .env 파일 내용
OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxx
```

```python
import os
from dotenv import load_dotenv
from openai import OpenAI

# .env 파일에서 환경 변수 로드
load_dotenv()

# 키 가져오기 (.env의 키 이름과 정확히 일치해야 함)
api_key = os.getenv("OPENAI_API_KEY")

# 클라이언트 초기화
client = OpenAI(api_key=api_key)
```

`dotenv` 패키지가 `.env` 파일을 읽어서 환경 변수로 등록하면, `os.getenv()`로 값을 꺼낼 수 있다.

### 3-3. requirements.txt — 패키지 의존성 공유

프로젝트에서 사용하는 패키지 목록을 `requirements.txt`로 관리하면, 다른 개발자가 한 번에 설치할 수 있다.

```bash
# 현재 환경의 패키지 목록을 파일로 저장
pip freeze > requirements.txt

# 파일에 명시된 패키지를 한 번에 설치
pip install -r requirements.txt
```

패키지를 추가할 때마다 `pip freeze > requirements.txt`로 갱신하고, 이 파일은 GitHub에 올려서 모든 개발자가 공유한다.

### 3-4. TIL 요약 파이프라인

오늘 실습의 전체 흐름은 다음과 같다.

```
MD 파일 목록 수집 (glob)
    → 각 파일 내용 읽기 (read_text)
        → 내용 합치기 (리스트에 취합)
            → OpenAI API에 요약 요청
                → 응답을 MD 파일로 저장 (write_text)
```

```python
from pathlib import Path
from openai import OpenAI

# 1. MD 파일 목록 수집
def find_md_files(directory):
    return list(Path(directory).glob("*.md"))

# 2. 파일 내용 읽기
def get_file_data(file_path):
    p = Path(file_path)
    if p.exists():
        return p.read_text(encoding="utf-8")
    return ""

# 3. 내용 취합
def merge_md_files(directory):
    md_files = find_md_files(directory)
    data_list = []
    for md in md_files:
        data = get_file_data(md)
        if data:
            data_list.append(data)
    return data_list

# 4. GPT 호출 및 저장
def summary_using_gpt(persona, user_data, client):
    text = " ".join(user_data)
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": persona},
            {"role": "user", "content": text}
        ]
    )
    return response.choices[0].message.content

# 5. 응답 저장
def save_response(content, output_path):
    Path(output_path).write_text(content, encoding="utf-8")
```

`__name__ == "__main__"` 패턴은 해당 파일을 직접 실행했을 때만 아래 코드가 동작하도록 하는 파이썬 내장 변수 활용이다. 다른 파일에서 import할 때는 실행되지 않는다.

---

## 4. 참고 — Git 관리 명령어 비교 (읽을거리)

| 명령어 | 대상 | 핵심 동작 | 기록 |
|--------|------|----------|------|
| `revert` | 특정 커밋 | 해당 커밋의 작업을 취소 | 취소 사실이 새 커밋으로 남음 |
| `reset` | HEAD 위치 | 과거 시점으로 이동, 이후 커밋 삭제 | 옵션에 따라 코드 보존 여부 다름 |
| `restore` | 파일 | 파일을 이전 버전으로 복원 | 복구 불가, 덮어쓰기 |
| `rm --cached` | 추적 상태 | Git 추적 중단 (파일 유지) | .gitignore와 함께 사용 |
| `reflog` | 이동 기록 | 모든 HEAD 이동 이력 확인 | reset으로 삭제된 커밋 복구 가능 |

---

## 정리

| 구분 | 핵심 포인트 |
|------|-------------|
| revert | 특정 커밋 작업 취소 + 취소 자체가 새 커밋으로 기록됨 |
| reset | 과거 시점으로 롤백. soft(Staging), mixed(Working), hard(완전 삭제) |
| restore | 파일 변경 사항을 이전 커밋으로 복원, 복구 불가이므로 주의 |
| rm --cached | 이미 커밋된 파일의 Git 추적만 중단, .gitignore 실수 수정에 활용 |
| pathlib | 경로를 객체로 관리하는 표준 라이브러리, 파일 생성·읽기·탐색을 간결하게 처리 |
| 인코딩 (UTF-8) | 한글 파일 처리 시 반드시 `encoding="utf-8"` 지정, 미지정 시 글자 깨짐 |
| with문 | 파일을 안전하게 열고 닫는 구문, close() 자동 처리 |
| glob / rglob | 패턴 기반 파일 탐색, rglob은 하위 디렉토리까지 재귀 탐색 |
| OpenAI API | ChatGPT를 코드로 호출, 공식 문서 기반으로 최신 코드 확인 필수 |
| .env + dotenv | API 키를 환경 변수 파일로 분리 관리, .gitignore에 추가하여 유출 방지 |
| requirements.txt | `pip freeze`로 패키지 목록 저장, `pip install -r`로 일괄 설치 |
