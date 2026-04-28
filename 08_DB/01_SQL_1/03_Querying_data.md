# [DB] Querying data

---

## 1. SELECT 기본 구조

```sql
SELECT field1, field2
FROM table_name;
```

- `SELECT` 뒤에 조회할 필드 지정, 여러 개일 경우 쉼표로 구분
- `FROM` 뒤에 조회할 테이블 지정
- `SELECT *` : 모든 필드 조회 (`*`는 asterisk, 전체의 의미)

## 2. AS - 별칭 지정

```sql
SELECT FirstName AS '이름'
FROM employees;
```

- 조회 결과에서 필드명을 다른 이름으로 출력
- 한글이나 공백이 포함된 경우 반드시 따옴표 사용
- Python의 `import ... as ...`와 동일한 개념

## 3. 산술 연산

```sql
SELECT
  Name,
  Milliseconds / 60000 AS '재생시간(분)'
FROM tracks;
```

- SELECT 절 내에서 기본 산술 연산자(`+`, `-`, `*`, `/`) 사용 가능
- AS와 함께 사용해 결과에 의미 있는 이름 부여

---

## 💡 한 줄 요약
> SELECT는 테이블에서 원하는 필드를 조회하며, AS로 별칭을 지정하고 산술 연산도 가능하다

## ❓ 더 찾아볼 것
- SELECT 절에서 사용할 수 있는 다양한 내장 함수
