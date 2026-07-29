# [데이터 엔지니어링] DataFrame과 Spark SQL

---

## 1. RDD의 한계 - 스키마를 표현할 수 없다
RDD(Resilient Distributed Dataset)는 데이터를 병렬로 처리하는 핵심적인 역할을 수행해 스파크가 빠르고 안정적으로 동작할 수 있게 해주는 가장 기본적인 데이터 처리 단위다.

하지만 RDD는 **데이터 값 자체는 표현할 수 있어도, 그 값이 어떤 컬럼이고 어떤 타입인지에 대한 메타데이터(스키마)를 표현할 방법이 없다.** 예를 들어 RDD 안에 `("홍석진", 20)` 같은 레코드가 있어도, RDD만 봐서는 이게 이름과 나이인지, 어떤 의미인지 알 수 없다.

이 때문에 RDD API를 직접 다루면 다음과 같은 문제가 생긴다.

- 스파크가 연산이나 표현식을 검사하지 못해 **내부적으로 최적화할 방법이 없음**
- join, filter, group by 등을 적용해도 스파크 입장에서는 단순한 함수 호출로만 보여서, 내부에서 어떤 표현식이 쓰이는지 알 수 없음
- 특히 PySpark에서는 연산 함수의 Iterator 데이터 타입을 스파크가 제대로 인식하지 못하고 단순 파이썬 기본 객체로만 인식함 (일종의 블랙박스처럼 동작)
- 컬럼의 데이터 타입을 명확히 알 수 없어 데이터 압축 같은 최적화 기법을 적용하지 못하고, 결국 바이트 형태로 그대로 직렬화해서 사용할 수밖에 없음
- 결과적으로 연산 순서를 재정렬해서 더 효율적인 실행(질의) 계획으로 바꾸는 것이 불가능함

## 2. DataFrame - 스키마를 가진 분산 데이터 컬렉션
이런 RDD의 한계를 보완하기 위해 등장한 것이 **DataFrame**이다.

> DataFrame = RDD + Schema + DSL

- **스키마(schema)를 가진 분산 데이터 컬렉션**이며, 데이터를 행(row)과 열(column)로 구성된 표 형태로 관리한다.
- 각 열은 명확한 데이터 타입과 메타데이터(schema)를 가지고 있다.
- 구조, 포맷 등 여러 측면에서 Pandas의 DataFrame에서 많은 영향을 받았다. 이름 있는 컬럼과 스키마를 가진 **분산 인메모리 테이블**처럼 동작한다고 이해하면 된다.

```
RDD (partition 단위)                    →        DataFrame (스키마가 있는 표)
partition#1                                      name   city      age  smoker height birthdate
John  Seattle 60 True 1.70 1960-01-01            John   Seattle    60   True   1.70   1960-01-01
partition#2                                      Tony   Cupertino  30   False  1.80   1990-01-01
Tony  Cupertino 30 False 1.80 1990-01-01         Mike   New York   40   True   1.65   1980-01-01
partition#3
Mike  New York 40 True 1.65 1980-01-01
```

RDD가 단순히 값들이 나열된 튜플/레코드 형태였다면, DataFrame은 각 열에 이름과 타입이 명시된 테이블 형태이기 때문에 사람이 보기에도 훨씬 익숙하고, 스파크 입장에서도 데이터를 더 잘 이해해서 내부적으로 최적화하기가 훨씬 용이해진다. (물론 내부적으로는 결국 RDD 위에서 동작하지만, 그 전에 상위 레이어에서 스키마 기반으로 최적화를 거친다고 이해하면 된다.)

### DataFrame의 데이터 타입
| 구분 | 타입 |
|---|---|
| 기본 타입 | Byte, Short, Integer, Long, Float, Double, String, Boolean, Decimal |
| 정형화 타입 | Binary, Timestamp, Date, Array, Map, Struct, StructField |

스키마를 직접 정의할 때는 `StructType` 안에 `StructField` 리스트 형태로 컬럼명·타입·nullable 여부를 선언한다.

```python
from pyspark.sql.types import StructType, StructField, StringType, ArrayType, IntegerType

schema = StructType([
    StructField("name", StringType(), True),
    StructField("scores", ArrayType(IntegerType()), True)
])
```

## 3. 스키마(Schema)를 왜 미리 정의하는가
스키마는 DataFrame을 위해 컬럼 이름과 그에 연관된 데이터 타입을 정의한 구조로, 외부 데이터 소스에서 구조화된 데이터를 읽어 들일 때 사용된다.

데이터를 **읽을 때 스키마를 추론(스키마 온 리드)** 하는 방식과 달리, **미리 스키마를 정의**해두면 다음과 같은 이점이 있다.

- 스파크가 데이터 타입을 추측해야 하는 책임을 덜어준다 (추측하려면 전체/일부 파일을 스캔하는 별도의 Job이 필요해짐 → 그 과정을 생략 가능)
- 스키마 확정을 위한 별도의 Job(스테이지·태스크 생성)을 만들지 않아도 된다
- 데이터가 스키마와 맞지 않는 경우, 조기에 문제를 발견해 대응하기 쉽다

즉 스키마를 미리 정의하는 것 자체가 **성능과 안정성 두 측면 모두**에 이점을 가져다준다.

## 4. DSL(Domain Specific Language)과 두 가지 처리 방식
DataFrame은 DSL(도메인 특화 언어) 형태와 SQL 형태, 두 가지 방식으로 모두 처리할 수 있다.

```python
# DSL 수행
sciDocs = data.filter(col("label") == 1)
sciDocs.show()

# SQL 수행 (동일한 결과)
data.createOrReplaceTempView("data")
data_scaled = spark.sql("SELECT * FROM data WHERE label = 1")
data_scaled.show()
```

두 방식 모두 실행 결과는 완전히 동일하다. 여기서 "SQL"이라는 건 Spark SQL **모듈 자체**를 의미하는 것이 아니라, DataFrame을 SQL 문법으로 다룰 수 있게 해주는 **API**를 의미한다는 점에 주의하자.

### RDD vs DataFrame 비교
| 구분 | RDD | DataFrame |
|---|---|---|
| 데이터 표현 방식 | 값만 표현 가능, 스키마 표현 불가능 | 명확한 스키마(컬럼, 데이터 타입)를 가진 구조적 데이터 |
| 최적화 및 성능 | 최적화가 어려움, 직접적 연산 필요 (로우 레벨) | Catalyst Optimizer를 통한 자동 최적화 및 빠른 처리 가능 |
| 사용 편의성 | 낮음 (저수준 API) | 높음 (고수준 API, SQL 활용 가능) |

### RDD를 사용하는 경우
1. 저수준의 Transformation과 Action을 직접 세밀하게 제어해야 할 때
2. 스트림 데이터(미디어, 텍스트 스트림 등)가 구조화되지 않아 스키마를 적용할 수 없을 때 → RDD가 더 유연함
3. 함수형 프로그래밍(Scala의 체인 연산 등) 패러다임을 직접 활용하고 싶을 때
4. 스키마 변환 자체가 필요 없는 단순 값 집계·리스트 연산일 때 (RDD가 오히려 더 간단)
5. DataFrame/Dataset에서 처리할 수 없는 저수준 성능 최적화가 필요할 때

### DataFrame을 사용하는 경우
그 외 대부분의 경우 DataFrame을 쓰는 것이 유리하다.
1. 고수준의 추상화와 도메인 기반 API가 필요할 때
2. filter, map, agg, avg, sum, columnar access 등 고수준 표현이나 반구조적 데이터에 대한 람다식 처리가 필요할 때
3. 타입 안정성과 최적화를 위해 컴파일 시 타입 안전성을 보장하고 싶을 때
4. Catalyst 최적화 및 Tungsten(효율적인 코드 생성을 담당하는 실행 엔진)을 활용하고 싶을 때
5. Spark API의 일관성과 간결함을 원할 때 (DSL과 SQL을 섞어 쓸 수 있음)

## 5. SparkSQL이란?
Spark SQL은 구조화된 데이터를 SQL처럼 처리할 수 있게 해주는 스파크 모듈이다.

- 내부적으로 DataFrame/Dataset API와 **동일한 엔진(Catalyst)**을 사용해서 처리한다
- DataFrame과 Dataset을 SQL처럼 다룰 수 있게 해주는 분산 SQL 쿼리 엔진
- RDD보다 훨씬 높은 수준의 추상화와 자동 최적화를 제공

> DataFrame이 중심이고, Spark SQL은 그것을 SQL 방식으로 접근하게 해주는 방법 중 하나다.

### Spark SQL의 역할
- SQL 같은 질의 수행
- Dataframe, Dataset이 Java, Scala, Python, R 등 여러 언어에서 정형화된 데이터 작업을 단순화할 수 있도록 추상화
- 정형화된 파일 포맷(JSON, CSV, txt, avro, parquet, orc 등)에서 스키마와 정형화 데이터를 읽고 쓰며, 데이터를 임시 테이블(View)로 변환
- 빠른 데이터 탐색을 위한 대화형 Spark SQL 셸 제공
- 표준 JDBC/ODBC 커넥터를 통해 외부 도구(DB 등)와 연결하는 중간 역할 제공
- 최종 실행을 위한 최적화된 질의 계획과 JVM 실행 코드를 생성

```
[JDBC] [Console] [User Programs (Java, Scala, Python)]
        ↓            ↓                  ↓
   ┌───────────────────────────┐        │
   │  Spark SQL  DataFrame API │ ←──────┘
   │     Catalyst Optimizer    │
   └───────────────────────────┘
              ↓
   Spark (Resilient Distributed Datasets)
```

## 6. Catalyst Optimizer - 내부 동작 원리
Spark SQL이 SQL 쿼리나 DataFrame 명령을 **가장 빠르고 효율적인 방식으로 처리**할 수 있도록 만들어주는 핵심 엔진이 **Catalyst Optimizer**다.

SQL, DataFrame, Dataset 세 가지 방식 모두 결국 이 Catalyst Optimizer를 거쳐서 처리되기 때문에, **어떤 방식으로 쓰든 성능 차이는 거의 없다.** 차이는 코드 스타일과 사용 편의성 정도다.

### 최적화 4단계
1. **SQL Parser & DataFrame API 해석 단계** — 입력(SQL Query, DataFrame)을 파싱해서 `Unresolved Logical Plan`을 만든다. 이 단계에서는 아직 컬럼 타입 등 정보가 확정되지 않은 상태다.
2. **Logical Plan 생성** — `Catalog`(스키마 정보)를 참조해서 컬럼 타입과 스키마를 검증하고, 실질적인 논리 계획(Logical Plan)을 세운다.
3. **Optimized Logical Plan 생성** — 필터의 순서를 바꾸거나, 불필요한 컬럼을 정리하거나, JOIN 방식을 최적화하는 등 카탈리스트의 핵심 최적화 규칙들이 적용된다.
4. **Physical Plan 생성** — 논리 계획을 실제로 실행 가능한 물리적 계획으로 변환한다. 이때 여러 개의 후보 Physical Plan이 나올 수 있는데, **Cost Model**로 각 계획의 비용을 계산해서 가장 좋은 하나를 `Selected Physical Plan`으로 선택한다. 이 계획이 최종적으로 RDD 연산 기반으로 변환되어 실행된다.

```
SQL Query  ─┐                                                        Code
DataFrame  ─┼→ Unresolved   → Logical  → Optimized    → Physical  → Generation → RDDs
            │  Logical Plan    Plan       Logical Plan   Plans
Catalog ────┘     (Analysis)  (Logical      (Physical Planning, Cost Model로
                              Optimization)  Selected Physical Plan 선정)
```

정리하면, Catalyst Optimizer가 쿼리 계획 자체를 더 똑똑하게 만들어서, 어떤 순서와 방식으로 연산을 실행해야 가장 빠를지를 자동으로 계산·최적화해주는 단계라고 볼 수 있다.

## 7. Dataset API (참고)
Spark 2.0에서 개발자가 한 종류의 API만 알면 되도록 DataFrame과 Dataset API를 하나로 통합했다.

- RDD가 저수준, DataFrame이 고수준이었다면, Dataset은 **이 두 가지의 장점을 합친 형태**다.
- RDD처럼 타입 안정성(typed)을 갖고 있으면서도, DataFrame처럼 구조화된 연산과 Catalyst Optimizer 기반의 고성능 처리를 지원한다.
- 다만 **타입 안전을 보장하는 언어(Java, Scala)에서만 사용 가능**하고, 동적 타입 언어인 Python·R에서는 사용할 수 없다 (DataFrame API만 사용 가능).
  - `DataFrame = Dataset[Row]` (Untyped API), `Dataset[T]` (Typed API)

## 8. View 등록 및 SQL 실행
PySpark에서 DataFrame API로 만든 데이터를 SQL 형태로 쿼리하려면, 먼저 DataFrame을 **View**로 등록해야 한다.

```python
# DataFrame을 뷰로 등록하기
df.createTempView("viewName")
df.createGlobalTempView("viewName")
df.createOrReplaceTempView("viewName")

# 뷰에 SQL 쿼리 실행하기
spark.sql("SELECT * FROM viewName").show()
spark.sql("SELECT * FROM global_temp.viewName").show()
```

- `createTempView` / `createOrReplaceTempView`: **현재 세션에서만** 유효한 뷰 (이미 있으면 대체)
- `createGlobalTempView`: **모든 세션에서 접근 가능**한 뷰. 다만 참조할 때는 반드시 `global_temp.뷰명` 형태로 접근해야 한다.

뷰가 등록된 후에는 `spark.sql("...")` 형태로 RDBMS에서 테이블을 조회하듯 편하게 SQL을 사용할 수 있다. (`spark`는 앞서 선언한 SparkSession 객체를 의미한다.)

## 9. DataFrame 구조 변환
DataFrame은 다른 형태로도 자유롭게 변환할 수 있다.

```python
# DataFrame → RDD (분산 처리용 RDD로 변환)
rdd1 = df.rdd

# DataFrame → JSON 문자열 → 첫 번째 항목 확인
df.toJSON().first()

# DataFrame → Pandas DataFrame (로컬 메모리 내 작업)
pandas_df = df.toPandas()
print(pandas_df)
```

## 10. SparkSession과 DataFrame 생성 API
DataFrame을 직접 만들거나 외부 데이터를 불러올 때는 항상 **SparkSession**이 필요하다. SparkSession은 Spark에서 DataFrame 작업을 시작하기 위한 기본 진입점이다.

```
Driver Program                  Cluster Manager        Worker Node
┌─────────────┐                                        ┌──────────────────┐
│ SparkContext │ ←───────────→                          │ Executor  Cache   │
└─────────────┘                                        │  Task    Task     │
                                                        └──────────────────┘
```

대표적으로 `createDataFrame()`, `read`, `readStream`을 통해 DataFrame을 생성할 수 있다.

### createDataFrame() - 코드 내부 데이터로 생성
Python 객체를 Spark DataFrame으로 변환하는 메서드로, 코드 내부의 데이터를 기반으로 DataFrame을 생성한다.

```python
from pyspark.sql import SparkSession

spark = SparkSession \
    .builder \
    .appName("create_order_dataframe_example") \
    .getOrCreate()

schema = """
    order_id STRING,
    customer_id STRING,
    order_date STRING,
    order_amount DOUBLE,
    payment_method STRING,
    category STRING
"""

data = [
    ("O0001", "C0001", "2026-01-05", 32000.0, "CARD", "book"),
    ("O0002", "C0002", "2026-01-06", 18000.0, "CASH", "food"),
    ("O0003", "C0001", "2026-01-07", 54000.0, "CARD", "electronics")
]

order_df = spark.createDataFrame(data=data, schema=schema)

order_df.show()
order_df.printSchema()
```

### read - 외부 데이터를 불러올 때
외부에 저장된 데이터를 Spark DataFrame으로 불러올 때 사용한다. 파일 경로나 저장소 위치를 지정해서 데이터를 로드하며, CSV, JSON, Parquet, JDBC 등 다양한 입력 소스를 지원한다.

```python
order_df = spark.read \
    .format("csv") \
    .option("header", True) \
    .option("delimiter", ",") \
    .schema(order_schema) \
    .load("data/input/order_history.csv")

order_df.show(10)
```

### readStream - 스트리밍 데이터를 지속적으로 읽을 때
| 함수 | 역할 | 예시 |
|---|---|---|
| `format()` | 읽어올 데이터의 형식 또는 소스 지정 | parquet, csv, json, orc, jdbc, text, table, kafka 등 |
| `option()` | 데이터 소스별 세부 읽기 옵션 지정 | ("delimiter", ","), ("header", True) 등 |
| `schema()` | 컬럼명과 데이터 타입을 직접 정의 | "order_id STRING, order_amount DOUBLE" 등 |
| `load()` | 설정한 정보를 바탕으로 실제 데이터 로드 | "data/input/order_history.csv" 등 |

`read`는 저장된 데이터를 **한 번에** 읽는 배치 입력 방식이고, `readStream`은 **계속 들어오는 데이터를 지속적으로** 읽는 스트리밍 입력 방식이다. 두 방식 모두 기본 설정 흐름은 비슷하지만, 데이터 소스(예: 카프카 토픽처럼 새 메시지가 계속 들어오는 경우, 특정 디렉터리에 새 파일이 쌓이는 경우 등)에 따라 세부 옵션과 실행 방식이 달라진다.

```python
stream_order_df = (spark.readStream
    .format("csv")
    .option("header", True)
    .schema(order_schema)
    # data/stream 디렉터리에 새 CSV 파일이 추가되면 스트리밍 입력 대상으로 처리 가능
    .load("data/stream")
)
```

## 11. 데이터 저장 - write / writeStream
데이터를 **만들거나 읽을 때는 SparkSession**을 사용하지만, **저장하거나 전송할 때는 생성된 DataFrame 객체에서 직접 write를 수행**한다는 점을 기억하자. 즉 입력 단계와 출력 단계의 호출 위치(주체)가 다르다.

- 배치 결과 저장: `write`
- 스트리밍 결과 저장: `writeStream`

```python
order_df = spark.read \
    .option("header", True) \
    .schema(order_schema) \
    .csv("data/input/order_history.csv")

card_order_df = order_df.filter(col("payment_method") == "CARD")

card_order_df.write \
    .mode("overwrite") \
    .format("csv") \
    .save("data/output")
```

```python
query = (
    stream_order_df.writeStream
    .format("console")
    .outputMode("append")
    .option("truncate", False)
    .start()
)

query.awaitTermination()
```

| 함수 | 역할 | 예시 |
|---|---|---|
| `format()` | 저장할 데이터 형식 또는 저장 대상 지정 | parquet(default), csv, json, orc, jdbc, text, kafka 등 |
| `mode()` | 기존 데이터가 있을 때의 처리 방식 지정 | `overwrite`(덮어쓰기) / `append`(뒤에 추가) / `ignore`(이미 있으면 저장 생략) / `error`(이미 있으면 오류 발생) |
| `option()` | 저장 대상에 필요한 세부 옵션 지정 | `.option("header", True)` |
| `bucketBy()` | 해시 기준으로 데이터를 버킷 단위로 나누어 저장 | `.bucketBy(10, "customer_id")` |
| `partitionBy()` | 특정 컬럼 값을 기준으로 디렉터리를 나누어 저장 | `.partitionBy("order_date")` |
| `sortBy()` | 저장 전 지정 컬럼 기준으로 정렬 | `.sortBy("order_amount")` |
| `save()` | 설정된 조건에 따라 실제 저장 실행 | `.save("data/output/orders_by_date")` |

> 참고: 실습 시에는 `python 파일명.py`로도 결과를 눈으로 확인하는 데 문제는 없지만, 실제 운영 환경에서는 `spark-submit`을 통해 실행하는 것이 권장된다.

---

## 💡 한 줄 요약
> RDD의 스키마 부재 문제를 보완한 것이 DataFrame이고, DataFrame과 SQL 모두 동일한 Catalyst Optimizer를 거쳐 최적화되기 때문에 어떤 방식을 쓰든 성능 차이 없이 자유롭게 선택해서 사용할 수 있다.

## ❓ 더 찾아볼 것
- Tungsten 실행 엔진 (Catalyst와의 역할 차이)
- spark-submit vs 일반 python 실행 방식의 차이
- Kafka 등 스트리밍 데이터 소스 연동
- Cost-based Optimization(CBO)의 구체적 비용 산정 방식
