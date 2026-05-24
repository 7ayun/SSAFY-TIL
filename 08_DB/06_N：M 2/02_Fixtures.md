# [DB] Fixtures

---

## 1. Fixtures가 필요한 이유

협업 시 DB 파일(`.sqlite3`)은 `.gitignore`에 의해 원격 저장소에 올라가지 않는다. A가 만든 테스트 데이터(계정, 게시글 등)는 B가 `pull`해도 전달되지 않아, B는 처음부터 더미 데이터를 다시 만들어야 하는 불편함이 생긴다.

**Fixtures**는 DB 데이터를 파일(주로 JSON)로 추출해 공유하는 방식으로 이 문제를 해결한다.

**주요 활용 목적**

- **초기 데이터 세팅**: 서비스 시작 시 필요한 기본 데이터(상품 카테고리, 권한 그룹 등)를 미리 세팅
- **테스트 샘플 데이터**: 항상 동일하고 예측 가능한 데이터 환경 구성
- **협업 시 데이터 동기화**: 팀원 모두가 동일한 초기 데이터에서 개발 시작

---

## 2. dumpdata — 데이터 추출

현재 DB의 데이터를 JSON 파일로 추출하는 명령어.

```bash
# 기본 형식
python manage.py dumpdata [앱이름.모델이름] [옵션] > 추출파일명.json

# 특정 모델 추출 (--indent 4: JSON 들여쓰기 4칸)
python manage.py dumpdata --indent 4 articles.article > articles.json

# 여러 모델 각각 추출
python manage.py dumpdata --indent 4 articles.comment > comments.json
python manage.py dumpdata --indent 4 accounts.user > users.json

# 프로젝트 전체 추출
python manage.py dumpdata --indent 4 > data.json
```

- 앱 이름만 쓰면 해당 앱의 전체 모델 데이터를 추출한다.
- 실행 후 별도 완료 메시지 없이 프로젝트 루트에 JSON 파일이 생성된다.
- JSON 파일 형식은 Django가 정한 규격이므로 직접 작성하지 않고 반드시 명령어로 생성한다.

> **인코딩 문제**: 한글이 포함된 경우 JSON 파일이 깨져 보일 수 있지만 loaddata 시에는 정상 동작한다. 처음부터 방지하려면 아래 옵션을 추가한다.
> ```bash
> python -Xutf8 manage.py dumpdata --indent 4 articles.article > articles.json
> ```

---

## 3. loaddata — 데이터 로드

dumpdata로 추출한 JSON 파일을 DB에 반영하는 명령어.

**Fixtures 파일의 기본 경로**

Django는 각 앱 폴더 안의 `fixtures/` 디렉토리를 약속된 경로로 인식한다. 파일을 이 경로에 위치시켜야 한다.

```
articles/
└── fixtures/
    ├── articles.json
    ├── comments.json
    └── users.json
```

**로드 명령어**

```bash
# 여러 파일을 한 번에 로드 (권장)
python manage.py loaddata articles.json users.json comments.json

# 별도로 순서를 지정해 로드
python manage.py loaddata users.json
python manage.py loaddata articles.json
python manage.py loaddata comments.json
```

**로드 순서의 중요성**

파일을 따로따로 로드할 때는 모델 간 관계(외래 키)를 고려해야 한다.

| 순서 | 모델 | 이유 |
|------|------|------|
| 1 | User | 다른 모델들이 참조함 |
| 2 | Article | User를 참조, Comment가 참조함 |
| 3 | Comment | Article과 User 모두 참조 |

Comment를 먼저 로드하면 참조할 Article이나 User가 없어 에러가 발생한다.

> 한 번에 여러 파일을 나열하면 Django가 자동으로 올바른 순서를 파악해 로드하므로, 가능하면 한 번에 로드하는 것을 권장한다.

**주의사항**

- loaddata 전에 해당 모델의 마이그레이션이 완료되어 있어야 한다.
- 같은 PK를 가진 데이터가 이미 존재하면 중복 에러가 발생할 수 있다.

---

## 💡 한 줄 요약
> `dumpdata`로 DB 데이터를 JSON으로 추출하고, `loaddata`로 다시 불러와 팀원 모두가 동일한 초기 데이터 환경에서 개발할 수 있다.

## ❓ 더 찾아볼 것
- Fixtures 파일을 앱별로 관리할 때의 폴더 구조 설계 방법
- JSON 외 YAML 포맷으로 추출하는 방법 (`--format yaml`)
- 대용량 데이터의 Fixtures 관리 전략 (압축, 앱 단위 분리 등)
