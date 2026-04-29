# [DB] Modifying Data

---

> DML (Data Manipulation Language): 데이터 조작 (CUD)
> SQL에서 Create = `INSERT`, Update = `UPDATE`, Delete = `DELETE`

## 1. INSERT - 데이터 삽입

```sql
INSERT INTO table_name (column_1, column_2, ...)
VALUES (value_1, value_2, ...);
```

- `INSERT INTO` 다음에 테이블명과 삽입할 필드 목록 지정
- `VALUES` 다음에 필드 순서에 맞게 값 작성
- `id (PRIMARY KEY AUTOINCREMENT)` 필드는 생략해도 자동 생성됨

### 여러 행 동시 삽입

```sql
INSERT INTO articles (title, content, createdAt)
VALUES
  ('title1', 'content1', '1900-01-01'),
  ('title2', 'content2', '1800-01-01'),
  ('title3', 'content3', '1700-01-01');
```

---

## 2. UPDATE - 데이터 수정

```sql
UPDATE table_name
SET column_name = expression
[WHERE condition];
```

- `SET` 절: 수정할 필드와 새 값 지정
- `WHERE` 절: 수정할 레코드 조건 지정
- **⚠️ WHERE 절 생략 시 전체 레코드 수정됨**

### 예시

```sql
-- 특정 레코드 수정
UPDATE articles
SET title = '수정된 제목', content = '수정된 내용'
WHERE id = 1;
```

---

## 3. DELETE - 데이터 삭제

```sql
DELETE FROM table_name
[WHERE condition];
```

- **⚠️ WHERE 절 생략 시 전체 레코드 삭제됨**

### 예시

```sql
-- 특정 레코드 삭제
DELETE FROM articles
WHERE id = 1;

-- 오래된 순으로 2개 삭제 (서브쿼리 활용)
DELETE FROM articles
WHERE id IN (
  SELECT id FROM articles
  ORDER BY createdAt
  LIMIT 2
);
```

---

## 💡 한 줄 요약
> INSERT로 데이터를 삽입하고, UPDATE/DELETE는 반드시 WHERE 조건을 확인한 후 실행한다

## ❓ 더 찾아볼 것
- `RETURNING` 절 (삽입/수정/삭제 후 결과 반환)
- 트랜잭션(`BEGIN`, `COMMIT`, `ROLLBACK`)으로 실수 방지하는 방법
