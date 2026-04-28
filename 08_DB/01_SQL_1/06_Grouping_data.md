# [DB] Grouping data
---

## 1. 집계 함수 (Aggregation Functions)

값에 대한 계산을 수행하고 **단일한 값 하나**를 반환하는 함수

| 함수 | 설명 |
|---|---|
| `SUM()` | 합계 |
| `AVG()` | 평균 |
| `MAX()` | 최댓값 |
| `MIN()` | 최솟값 |
| `COUNT()` | 개수 |

## 2. GROUP BY

```sql
SELECT Country, COUNT(*)
FROM customers
GROUP BY Country;
```

- `SELECT FROM` 뒤에 작성
- 특정 필드를 기준으로 레코드를 그룹화
- **반드시 집계 함수와 함께 사용** → 단순 그룹화만 하면 DISTINCT와 다를 게 없음
- 기준 컬럼은 여러 개 지정 가능

## 3. HAVING - 그룹 결과 필터링

```sql
SELECT Country, COUNT(*)
FROM customers
GROUP BY Country
HAVING COUNT(*) >= 5;
```

- `GROUP BY`로 그룹화된 결과에 조건을 적용
- `WHERE`는 레코드 전체에 조건 → `HAVING`은 **그룹화된 결과**에 조건

## 4. SQL 실행 순서

```
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

> **작성 순서와 실행 순서가 다름!**  
> `WHERE`로 먼저 필터링 → `GROUP BY`로 그룹화 → `HAVING`으로 그룹 필터링 → 마지막에 `SELECT`로 출력

---

## 💡 한 줄 요약
> GROUP BY는 집계 함수와 함께 데이터를 그룹화하고, HAVING으로 그룹 결과를 필터링한다

## ❓ 더 찾아볼 것
- `WHERE` vs `HAVING` 차이 심화 정리
- Django ORM이 내부적으로 생성하는 SQL 확인 방법
