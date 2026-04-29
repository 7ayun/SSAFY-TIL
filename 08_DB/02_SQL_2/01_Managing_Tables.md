# [DB] Managing Tables - 2026.04.28

---

## 1. CREATE TABLE - 테이블 생성

```sql
CREATE TABLE table_name (
  column_1 data_type constraints,
  column_2 data_type constraints,
  ...
);
```

- 필드명 / 데이터 타입 / 제약조건(constraints) 순서로 작성
- 지난 시간까지 이미 구현된 테이블을 조회했다면, 이제는 테이블을 직접 만드는 것 (DDL)

### SQLite 데이터 타입

| 타입 | 설명 |
|---|---|
| `INTEGER` | 정수 |
| `TEXT` | 문자열 |
| `REAL` | 실수 |
| `BLOB` | 바이너리 데이터 |
| `NUMERIC` | 날짜, Boolean 등 |

### 제약조건 (Constraints)

| 제약조건 | 설명 |
|---|---|
| `NOT NULL` | NULL 값 허용 안 함 |
| `DEFAULT 값` | 값 미입력 시 기본값 설정 |
| `PRIMARY KEY` | 기본 키 지정 |
| `AUTOINCREMENT` | 자동 증가 (INTEGER PRIMARY KEY와 함께 사용) |

### 예시

```sql
CREATE TABLE examples (
  ExamId INTEGER PRIMARY KEY AUTOINCREMENT,
  LastName VARCHAR(50) NOT NULL,
  FirstName VARCHAR(50) NOT NULL
);
```

---

## 2. ALTER TABLE - 테이블 수정

### 필드 추가
```sql
ALTER TABLE examples
ADD COLUMN Country VARCHAR(100) NOT NULL DEFAULT 'default value';
```
> NOT NULL 필드 추가 시 반드시 DEFAULT 값을 함께 지정해야 함 (기존 레코드에 값이 없으므로)

### 필드명 변경
```sql
ALTER TABLE examples
RENAME COLUMN Address TO PostCode;
```

### 테이블명 변경
```sql
ALTER TABLE examples RENAME TO new_examples;
```

### 필드 삭제
```sql
ALTER TABLE examples DROP COLUMN Age;
```

### ⚠️ ALTER TABLE 주의사항
- 실무에서는 **매우 위험한 명령어** 중 하나
- 수백만 건의 데이터가 있는 테이블에 컬럼 추가 시 **table lock** 발생 → 서비스 중단 가능
- 실무에서는 ALTER TABLE 직접 실행 대신 **마이그레이션 도구** 사용 (Django의 migration 등)
- **SQLite 한계**: 컬럼 자체의 데이터 타입/제약조건 변경 불가 → 새 테이블 생성 후 데이터 이전해야 함

---

## 3. DROP TABLE - 테이블 삭제

```sql
DROP TABLE new_examples;
```

- 삭제 후 **복구 불가** (백업 없으면 영구 손실)
- 실무에서는 일반 개발자에게 DROP TABLE 권한을 주지 않음
- 자동화 백업 + DBMS에서 확인 단계 추가로 실수 방지

---

## 💡 한 줄 요약
> CREATE TABLE로 테이블을 생성하고, ALTER TABLE로 구조를 수정하며, DROP TABLE로 삭제한다. 단, ALTER/DROP은 실무에서 신중하게 사용해야 한다

## ❓ 더 찾아볼 것
- Django migration이 내부적으로 ALTER TABLE을 어떻게 처리하는지
- 마이그레이션 도구 종류 (Alembic, Flyway 등)
