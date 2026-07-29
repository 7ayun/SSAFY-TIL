# [데이터 엔지니어링] DSL과 SQL을 활용한 데이터 처리

---

## 1. SQL 쿼리 기본 문법 복습
DataFrame은 DSL(메서드 체이닝) 방식과 SQL(문자열 쿼리) 방식을 모두 지원한다. 본격적으로 예시를 보기 전에, 표준 SQL 문법을 간단히 정리하고 넘어간다.

| 목적 | 문법 |
|---|---|
| 데이터 조회 | `SELECT`, `WHERE` |
| 정렬 | `ORDER BY` |
| 중복 제거 | `DISTINCT` |
| 데이터 집계 | `GROUP BY`, `HAVING`, 집계 함수(`COUNT`, `AVG`, `SUM`) |
| 데이터 결합 | `JOIN` (INNER / LEFT / OUTER 등) |

## 2. DataFrame 생성 (DSL vs SQL)
DataFrame을 만드는 기본 흐름은 항상 **① SparkSession 생성 → ② Schema 정의 → ③ DataFrame 생성 → ④ 결과 확인** 4단계로 진행된다.

```python
# DSL 코드
spark = SparkSession.builder.appName("ExampleApp").getOrCreate()

schema = StructType([
    StructField("name", StringType(), True),
    StructField("age", IntegerType(), True)
])

# parallelize는 RDD API. RDD 값 + 스키마를 넣어 DataFrame으로 변환한다.
parts = spark.sparkContext.parallelize([("Mine", "28"), ("Filip", "29"), ("Jonathan", "30")])
people = parts.map(lambda p: Row(name=p[0], age=int(p[1].strip())))
df = spark.createDataFrame(people, schema)

df.show()
```

SQL로 조회하려면 먼저 DataFrame을 **TempView로 등록**해야 한다. `spark`는 앞서 선언한 SparkSession 객체를 가리키며, `spark.sql(...)` 안에 SQL문을 문자열로 작성해주면 된다.

```python
# DataFrame을 SQL에서 사용할 수 있도록 TempView 등록
df.createOrReplaceTempView("people")

result = spark.sql("""
    SELECT name, age
    FROM people
""")
result.show()
```

파일로부터 DataFrame을 만들 때도 마찬가지 흐름이다.

```python
# 파일에서 직접 DataFrame 생성 (DSL)
people_df = (spark.read
    .option("header", "false")
    .option("inferSchema", "true")
    .csv("people.txt")
    .toDF("name", "age"))
```
- `inferSchema=true` → 데이터 타입을 스파크가 자동으로 추론하게 함 (스키마를 따로 선언하지 않았을 때)
- `.toDF("name", "age")` → 컬럼 이름을 명시적으로 지정

```python
# SQL 방식
people_df.createOrReplaceTempView("people")
result = spark.sql("""SELECT name, age FROM people""")
result.show()
```

그 외 JSON, Parquet, TXT 등 다양한 Spark 데이터 소스도 동일한 방식으로 읽을 수 있다.

```python
# JSON
df = spark.read.json("filename.json")
df = spark.read.load("filename.json", format="json")

# Parquet
df = spark.read.load("filename.parquet")

# TXT
df = spark.read.txt("filename.txt")
```

## 3. 데이터 탐색 (Inspect Data)
DataFrame을 다루기 전, 구조와 내용을 확인하는 메서드들이다.

```python
df.dtypes          # 열 이름과 데이터 유형 반환
df.show()           # 내용 표시
df.head()           # 처음 n개의 행 반환 (Row 리스트 반환)
df.first()          # 첫 번째 행 하나만 반환 (단일 Row 객체)
df.take(n)          # 처음 n개의 행 반환 (액션 연산 - 분산 데이터를 실제로 수집)
df.schema           # DataFrame의 스키마를 구조(StructType) 객체로 반환
df.printSchema()    # 스키마를 트리 형태로 출력
df.describe().show()# 요약 통계 계산
df.columns          # 열 이름만 리스트로 반환
df.count()          # 행 개수 계산 (액션 연산)
df.distinct().count() # 고유 행 개수 계산
df.explain()        # 논리적/물리적 실행 계획 출력 → Catalyst Optimizer가 어떻게 쿼리를 처리할지 미리 확인 가능
df = df.dropDuplicates() # 중복 행 제거
```

> 💡 `take()`처럼 액션 연산에 해당하는 메서드는 분산되어 있는 데이터를 실제로 모아오는 작업이기 때문에, 데이터가 아주 많은 상태에서 습관적으로 호출하면 성능 리스크가 있다. 단순히 결과를 눈으로 확인하려는 목적이라면 특히 주의가 필요하다.

## 4. SELECT (컬럼 선택)
```python
# DSL
df = spark.createDataFrame(data)
df.select("column1", "column2").show()
```
```sql
-- SQL (TempView 등록 후)
SELECT column1, column2
FROM my_table
```
결과는 완전히 동일하다.

### 표현식과 별칭(alias)을 활용한 SELECT
```python
# DSL: column1과 column2+1 결과 출력
df.select(
    col("column1"),
    (col("column2") + 1).alias("column2_plus1")
).show()

# column1이 "A"보다 큰 값만 필터링
df.filter(col("column1") > "A").show()
```
```sql
-- SQL
SELECT column1, column2 + 1 AS column2_plus1
FROM my_table

SELECT *
FROM my_table
WHERE column1 > 'A'
```
DSL에는 `filter`라는 별도 메서드가 있지만, SQL에는 그런 키워드가 없으므로 SQL에서는 **`WHERE` 절**로 동일한 조건을 표현한다.

## 5. CASE WHEN / IN 조건 처리
```python
# DSL
from pyspark.sql.functions import when

df.select(
    "column1",
    when(df.column2 > 100, 1).otherwise(0).alias("flag")
).show()

df.filter(df.column1.isin("A", "B")).show()
```
```sql
-- SQL
SELECT column1,
    CASE WHEN column2 > 100 THEN 1 ELSE 0 END AS flag
FROM my_table

SELECT *
FROM my_table
WHERE column1 IN ('A', 'B')
```
DSL의 `when ... otherwise`는 SQL의 `CASE WHEN ... THEN ... ELSE ... END`에 대응하고, DSL의 `isin`은 SQL의 `IN` 조건절에 대응한다. 다중 필터링이나 조건부 로직을 유연하게 처리할 수 있으며, 어떤 방식을 쓰든 Catalyst Optimizer가 동일하게 최적화해주기 때문에 성능 차이는 거의 없다. 실무에서는 회사(팀)에서 주로 쓰는 방식에 맞추면 된다.

## 6. 문자열 조건 처리 (LIKE, STARTSWITH, ENDSWITH)
```python
# DSL
df.select(
    col("column1"), col("column1").startswith("A").alias("starts_with_A")
).show()

df.select(
    col("column2"),
    col("column2").cast("string").endswith("00").alias("ends_with_00")
).show()

df.select(
    col("column1"), col("column1").like("A").alias("is_A")
).show()
```
```sql
-- SQL
SELECT column1, column1 LIKE 'A%' AS starts_with_A
FROM my_table

SELECT column2, CAST(column2 AS STRING) LIKE '%00' AS ends_with_00
FROM my_table

SELECT column1, column1 = 'A' AS is_A
FROM my_table
```
- 모든 조건은 문자열 비교로 True/False 결과를 반환한다. **숫자 컬럼은 문자열로 캐스팅**해야 시작/끝 문자열 비교가 의미를 가진다.
- 정확히 일치하는지 확인할 때, DSL에서는 `like("A")`를 쓰지만 SQL에서는 `= 'A'`처럼 일반 equal 조건으로 표현한다.

## 7. 문자열 추출과 범위 조건 (SUBSTRING, BETWEEN)
```python
# DSL: column1에서 2번째 문자부터 3개의 문자를 추출하여 "name"으로 지정
df.select(df.column1.substr(2, 3).alias("name")).show()

# column2의 값이 50~150 사이에 있으면 True 표시 (50 이상 150 이하)
df.select(
    col("column1"), col("column2"),
    col("column2").between(50, 150).alias("is_between_50_150")
).show()
```
```sql
-- SQL
SELECT SUBSTRING(column1, 2, 3) AS name FROM my_table

SELECT column1, column2,
    column2 BETWEEN 50 AND 150 AS is_between_50_150
FROM my_table
```
> ⚠️ **주의**: SQL의 `SUBSTRING`은 문자 위치가 **1부터 시작**한다 (0부터 시작하는 인덱스가 아니다). DSL의 `substr()`도 이와 동일한 기준으로 동작한다.

`BETWEEN a AND b` 문법은 구간(이상~이하) 내 포함 여부를 확인하는 데 사용한다.

## 8. 컬럼명 변경 및 삭제 (Rename & Remove Columns)
```python
# DSL
df.withColumnRenamed("column1", "alphabet") \
  .withColumnRenamed("column2", "number") \
  .show()

df.drop("column1").show()
```
```sql
-- SQL
SELECT column1 AS alphabet, column2 AS number
FROM my_table

SELECT column2 FROM my_table
```
> 💡 **핵심 차이**: DataFrame API의 `withColumnRenamed`/`drop`은 실제로 스키마(컬럼 구성) 자체를 바꾸는 것이지만, SQL에서 `AS`로 이름을 바꾸는 것은 **출력할 때만 그렇게 보이도록** alias를 붙이는 것일 뿐, 원본 테이블의 이름을 바꾸는 것은 아니다. 마찬가지로 SQL에는 별도의 `DROP` 키워드가 없어서, 특정 컬럼을 빼고 싶으면 그냥 SELECT할 때 그 컬럼을 조회하지 않는 방식으로 구현한다. 테이블 구조 자체를 조작하는 작업은 DataFrame API 쪽이 더 용이하다.

## 9. 그룹별 집계 (Group By, Count)
```python
# DSL
df.groupBy("column1").count().show()
```
```sql
-- SQL
SELECT column1, COUNT(*) as count
FROM my_table
GROUP BY column1
```

## 10. 조건 필터 (Filter)
```python
# DSL: column2 값이 200보다 큰 항목만 필터링
df.filter(df["column2"] > 200).show()
```
```sql
-- SQL
SELECT *
FROM my_table
WHERE column2 > 200
```

## 11. 정렬 (Sort, OrderBy)
```python
# DSL: 단일 컬럼 내림차순
df.sort(df["column1"].desc()).show()

# 다중 컬럼 정렬: column1 오름차순, column2 내림차순
df.orderBy(["column1", "column2"], ascending=[True, False]).show()
```
```sql
-- SQL: 단일 컬럼 내림차순
SELECT * FROM my_table ORDER BY column1 DESC

-- 다중 컬럼 정렬
SELECT * FROM my_table ORDER BY column1 ASC, column2 DESC
```
- 단일 컬럼 정렬은 DSL의 `sort()`로도 가능하지만, 다중 컬럼 정렬 시에는 `orderBy()`를 쓰는 게 더 명확하다.
- `ascending`의 기본값은 오름차순(`True`)이며, `False`를 주면 내림차순으로 정렬된다.
- DSL과 SQL 중 어느 쪽이 더 편한지는 상황에 따라 다르므로, SQL을 잘 모른다면 DataFrame API만으로도 대부분의 문제를 해결할 수 있다.

## 12. 결측값 처리 및 값 치환 (Missing & Replacing Values)
```python
# DSL
# column1의 NULL → "Unknown", column2의 NULL → 0
df.na.fill({"column1": "Unknown", "column2": 0}).show()

# 두 컬럼 중 하나라도 NULL이 있으면 해당 행 제거
df.na.drop().show()

# "A" → "Alpha", "B" → "Beta"로 문자열 값 대체
df.na.replace({"A": "Alpha", "B": "Beta"}).show()
```
```sql
-- SQL
SELECT
    COALESCE(column1, 'Unknown') AS column1,
    COALESCE(column2, 0) AS column2
FROM my_table

SELECT * FROM my_table
WHERE column1 IS NOT NULL AND column2 IS NOT NULL

SELECT CASE
    WHEN column1 = 'A' THEN 'Alpha'
    WHEN column1 = 'B' THEN 'Beta'
    ELSE column1
END AS column1, column2
FROM my_table
```
> 💡 **핵심 차이**: DataFrame API의 `na.fill` / `na.drop` / `na.replace`는 Pandas의 `fillna`/`dropna`와 비슷하게 동작하며, 결과가 DataFrame 객체 자체에 반영된다. 반면 SQL에서 `COALESCE`나 `CASE WHEN`으로 값을 대체하는 것은 **일회성(조회 시점에만 적용)**이다. 즉 저장된 값 자체를 직접 바꾸는 게 아니라, 조회 결과를 그렇게 보여주는 것뿐이다. NULL 여부 확인은 `IS NULL` / `IS NOT NULL` 조건으로 처리한다.

## 13. JOIN 처리
예시 테이블: `users(user_id, name)`, `orders(order_id, user_id, amount)`

### INNER JOIN / LEFT JOIN
```sql
-- INNER JOIN (공통 키만 조회 - 등가조인)
SELECT u.user_id, u.name, o.order_id, o.amount
FROM users u
INNER JOIN orders o
ON u.user_id = o.user_id

-- LEFT JOIN (users 기준, 매칭 없으면 NULL)
SELECT u.user_id, u.name, o.order_id, o.amount
FROM users u
LEFT JOIN orders o
ON u.user_id = o.user_id
```

### RIGHT JOIN / FULL OUTER JOIN
```sql
-- RIGHT JOIN (orders 기준)
SELECT u.user_id, u.name, o.order_id, o.amount
FROM users u
RIGHT JOIN orders o
ON u.user_id = o.user_id

-- FULL OUTER JOIN (양쪽 모두 포함)
SELECT u.user_id, u.name, o.order_id, o.amount
FROM users u
FULL OUTER JOIN orders o
ON u.user_id = o.user_id
```

### DSL 방식
SQL에서는 `JOIN` 키워드를 문자열 안에 작성하지만, DSL에서는 `.join()` 메서드의 **세 번째 인자**로 조인 방식을 문자열로 넘겨준다.

```python
# INNER JOIN
df_users.join(
    df_orders,
    df_users.user_id == df_orders.user_id,
    "inner"
).select(
    df_users.user_id,
    df_users.name,
    df_orders.order_id,
    df_orders.amount
).show()

# LEFT JOIN
df_users.join(
    df_orders,
    df_users.user_id == df_orders.user_id,
    "left"
).show()

# RIGHT JOIN
df_users.join(
    df_orders,
    df_users.user_id == df_orders.user_id,
    "right"
).show()

# FULL OUTER JOIN
df_users.join(
    df_orders,
    df_users.user_id == df_orders.user_id,
    "full_outer"
).select(
    df_users.user_id,
    df_users.name,
    df_orders.order_id,
    df_orders.amount
).show()
```
`"inner"` 자리를 `"left"`, `"right"`, `"full_outer"`로 바꿔주기만 하면 각각 LEFT/RIGHT/FULL OUTER JOIN이 된다. 어떤 조인을 쓸지는 두 테이블 중 어느 쪽을 "기준"으로 삼을지에 따라 결정하면 된다 (LEFT JOIN이면 왼쪽 테이블 기준, RIGHT JOIN이면 오른쪽 테이블 기준).

---

## 💡 한 줄 요약
> DataFrame의 대부분의 연산은 DSL(메서드 체이닝)과 SQL(문자열 쿼리) 두 가지 방식으로 동일하게 표현할 수 있고 성능 차이도 없으므로, 익숙한 방식이나 팀의 컨벤션에 맞춰 자유롭게 선택하되 SUBSTRING의 1-based 인덱스나 SQL 값 치환의 일회성 같은 방식 간 미묘한 차이는 구분해서 알아둘 필요가 있다.

## ❓ 더 찾아볼 것
- Spark DSL의 `Window` 함수 (그룹 내 순위, 누적 집계 등)
- `HAVING` 절을 활용한 그룹 필터링
- 대용량 조인 시 Broadcast Join 등 조인 최적화 전략
- Cost Model 기반의 JOIN 순서 최적화
