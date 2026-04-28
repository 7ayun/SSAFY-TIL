# [DB] Filtering data

---

## 1. DISTINCT - 중복 제거

```sql
SELECT DISTINCT Country
FROM customers;
```

- `SELECT` 키워드 바로 뒤에 작성
- 조회 결과에서 중복된 레코드를 제거
- 예) 고객 테이블에서 어떤 국가의 고객이 있는지 중복 없이 확인할 때

## 2. WHERE - 조건 필터링

```sql
SELECT field1, field2
FROM table_name
WHERE 조건;
```

## 3. Operators (연산자)

### 비교 연산자

| 연산자 | 설명 |
|---|---|
| `=` | 같음 |
| `!=` 또는 `<>` | 다름 |
| `>`, `<`, `>=`, `<=` | 크기 비교 |

### 범위 및 목록 연산자

| 연산자 | 설명 | 예시 |
|---|---|---|
| `BETWEEN a AND b` | a 이상 b 이하 범위 | `WHERE Age BETWEEN 20 AND 30` |
| `IN (...)` | 목록 중 하나와 일치 | `WHERE Country IN ('Canada', 'Germany', 'France')` |
| `LIKE` | 패턴 매칭 | `WHERE Name LIKE 'A%'` |

> `IN`은 여러 `OR` 조건을 나열한 것과 동일하게 동작

### 논리 연산자

| 연산자 | 설명 |
|---|---|
| `AND` | 두 조건 모두 만족 |
| `OR` | 두 조건 중 하나 만족 |
| `NOT` | 조건의 부정 |

## 4. LIMIT - 결과 개수 제한

```sql
SELECT field1
FROM table_name
LIMIT 10;

-- OFFSET과 함께 (페이지네이션)
LIMIT 10 OFFSET 20;  -- 21번째부터 10개
```

---

## 💡 한 줄 요약
> WHERE로 조건을 걸어 원하는 데이터만 필터링하고, DISTINCT로 중복을 제거하며, LIMIT으로 결과 개수를 제한할 수 있다

## ❓ 더 찾아볼 것
- `LIKE` 패턴 매칭 (`%`, `_`) 상세 사용법
- `NULL` 비교 방법 (`IS NULL`, `IS NOT NULL`)
- `WHERE` vs `HAVING` 차이
