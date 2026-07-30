# [데이터 엔지니어링] Spark 최적화

---

## 1. Spark Application Web UI로 실행 상태 확인하기

Spark Application의 성능이나 실행 상태는 로그로도 확인할 수 있지만, Web UI(기본 포트 `4040`)에 접속하면 훨씬 편하게 확인할 수 있다. 코드만 봐서는 실제로 몇 개의 Job으로 실행됐는지, Stage가 어떻게 나뉘었는지, Task가 몇 개 만들어졌는지 알기 어렵기 때문에 이 UI를 활용한다.

- Web UI는 하나의 Application(SparkSession) 단위로 동작한다.
- 주로 확인하게 되는 탭: **Jobs**, **Stages**, **SQL/DataFrame**
  - Jobs: Job이 어떻게 쪼개졌는지
  - Stages: Stage 내부 구성
  - SQL/DataFrame: SQL 기반 처리 시 실행 계획
- Application이 종료되면 `localhost:4040` 접속도 함께 끊긴다. (실행 중일 때만 확인 가능)

## 2. Job / Stage / Task 관계

이 세 가지 개념의 관계를 정확히 이해해야 Web UI를 제대로 읽을 수 있다.

- **Job**: `show()`, `collect()`, `count()` 같은 Action이 실행될 때, 그 Action을 처리하기 위한 전체 실행 흐름 하나가 Job이 된다. 일반적으로 **Action 함수 하나 = Job 하나**로 생각하면 된다.
- **Stage**: 하나의 Job은 여러 개의 Stage로 나뉠 수 있다. 나뉘는 기준은 **Shuffle 발생 여부**다.
  - `filter()`처럼 각 파티션 안에서 끝나는 연산 → Shuffle 없음 → 같은 Stage
  - `groupBy()`처럼 같은 키를 모으기 위해 파티션 간 데이터 이동이 필요한 연산 → Shuffle 발생 → 새로운 Stage 경계
- **Task**: 파티션 단위로 실제 연산을 수행하는 가장 작은 실행 단위. 보통 **파티션 1개 = Task 1개**로 매칭되며, Executor의 Core가 이 Task들을 병렬로 처리한다.

```
Driver ─ action → Job1
       ─ action → Job2 → Stage1 ─(shuffle)→ Stage2 ─(shuffle)→ Stage3
       ─ action → Job3                       └─ Task, Task, Task ...
```

파티션(=Task) 수가 너무 적으면 병렬성이 낮아지고, 너무 많으면 스케줄링에 필요한 오버헤드가 커진다. 따라서 파티션 수는 **처리할 데이터 크기**와 **클러스터가 가진 Core 수**를 함께 고려해서 정해야 한다.

## 3. Wide Transform vs Narrow Transform

- **Shuffle**: Executor 간에 파티션(데이터프레임) 데이터가 이동하거나 재분배되는 작업.
- **Wide Transformation**: Shuffle이 발생하는 연산 (`groupBy`, `distinct`, `orderBy`, `join` 등)
- **Narrow Transformation**: Shuffle 없이 현재 파티션 안에서 처리되는 연산 (`filter`, `withColumn` 등)
- 실행 계획(`explain()`)이나 Web UI에서 **Exchange**라는 표시가 보이면 그 지점에서 Shuffle이 발생했다는 신호다.

## 4. Wide Transform은 왜 고비용인가

Shuffle에는 다음 세 단계가 모두 포함되기 때문에 상대적으로 비용이 크다.

1. **데이터 직렬화**: 메모리 객체 → Byte 형태로 변환 (네트워크 전송이 가능하도록)
2. **네트워크 전송**: 임시 파일 기록 등 Disk I/O를 포함해 Executor 간 데이터 전달
3. **데이터 역직렬화**: 전달받은 Byte 데이터를 다시 메모리 객체로 복원

또한 Spark는 **앞 Stage가 모두 끝나야 다음 Stage를 시작**할 수 있다. 예를 들어 `filter()` 뒤에 `groupBy()`가 온다면, filter가 포함된 앞단 Stage의 모든 Task가 끝나야 groupBy Stage가 시작된다. 이때 특정 파티션에 데이터가 몰려(Skew) 유난히 오래 걸리는 Task가 하나라도 있으면, 나머지 Task가 다 끝났어도 그 Task를 기다려야 하므로 전체 처리 지연이 늘어날 수 있다.

## 5. Wide Transform 대응 전략

SQL의 `GROUP BY`, `DISTINCT`, `ORDER BY`, `JOIN` 등은 대부분 Shuffle을 유발하지만 분석에 꼭 필요한 연산이기도 하다. 무조건 피하기보다는 **필요한 경우에만 사용하고 성능 부담을 줄이는 방향**으로 설계하는 것이 중요하다.

**① 적절한 직렬화 라이브러리 선택**
Spark 기본 직렬화 방식(Java Serialization)은 성능이 상대적으로 좋지 않다. `KryoSerializer` 사용을 권장하며, 공식 문서 기준 기본 방식보다 최대 10배 정도 빠를 수 있다.

```python
spark.conf.set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
```

**② Shuffle 이후 파티션 개수 조정**
`spark.sql.shuffle.partitions` 값이 Wide Transform 이후 생성되는 파티션 수를 결정하며 기본값은 200이다. 모든 상황에 200이 적절한 것은 아니므로, 전체 Core 수와 처리할 데이터 크기를 기준으로 조정해야 한다.

> 예: 전체 Core 12개, join/groupBy/dropDuplicates 이후 데이터 크기 약 2GB로 예상되는 경우
> - 12개 파티션 → 파티션당 약 170MB
> - 24개 파티션 → 파티션당 약 85MB

**③ 필요한 경우 Persist 사용**
대용량 데이터프레임에 Wide Transform을 적용한 결과를 여러 번 재사용한다면 `persist()`로 재계산 비용을 줄일 수 있다. 특히 `groupBy`, `dropDuplicates`처럼 데이터 양을 줄이는 연산 이후 만들어진 데이터프레임을 재사용할 때 효과적이다.

- `cache()`/`persist()`는 호출 즉시 메모리에 저장되는 것이 아니라, **첫 번째 Action이 호출될 때** 계산 결과가 캐시에 저장된다 (Lazy Evaluation과 연관).
- 한 번만 사용할 데이터를 캐싱하면 계산 비용은 줄어들지 않고 메모리만 차지하게 되므로 무조건 캐싱하는 것은 좋지 않다.
- 데이터 양이 많으면 Executor 메모리 압박이 생길 수 있으므로, 다 쓴 뒤에는 `unpersist()`로 해제하는 습관이 중요하다.

```python
df = df.filter(...).groupBy(...).agg(...)
df.persist()
cnt = df.count()        # 여기서 실제로 계산 + 캐시 저장
df2 = df.filter(...)    # 캐시된 결과 재사용
```

**④ Broadcast Join 힌트 활용**
Join 대상 중 한쪽 데이터프레임이 충분히 작다면, 그 작은 테이블을 전체 Executor에 복사해두고(Broadcast) 큰 데이터프레임은 이동시키지 않은 채 각 Executor 내부에서 Join하는 방식으로 Shuffle 비용을 줄일 수 있다. 특히 **Streaming 데이터처럼 한 번에 들어오는 데이터 양이 크지 않고, 상대적으로 작은 마스터(Dimension) 테이블과 Join하는 경우**에 효과적이다.

```python
from pyspark.sql.functions import broadcast

# 큰 데이터프레임: 주문/거래 로그(order_df) / 작은 데이터프레임: 고객 등급 마스터(customer_grade_df)
join_df = order_df.join(
    broadcast(customer_grade_df),
    on="customer_id",
    how="left"
)
```

## 6. Spark Plan (실행 계획)

Spark는 연산을 바로 실행하지 않고, DataFrame의 Transform 연산을 어떻게 처리할지 먼저 분석하고 최적화한 뒤 실행한다. 이 실행 계획을 **Spark Plan**이라고 한다.

- **Logical Plan**을 먼저 만든 뒤, 이를 바탕으로 여러 **Physical Plan** 후보를 생성한다.
- Catalog의 메타데이터·통계 정보를 참고해 Logical Plan을 최적화하고, 그중 가장 효율적인 Physical Plan을 최종 선택해 실제로 실행한다.

```
Spark 프로그램 → Logical Plan → (Catalog 통계 참고, 최적화) → 여러 Physical Plan 후보 → 최종 Physical Plan 선택 → 실행
```

- `df.explain()`을 호출하면 기본적으로 Physical Plan이 출력된다.
- `df.explain(extended=True)`를 사용하면 Physical Plan뿐 아니라 **Parsed Logical → Analyzed Logical → Optimized Logical → Physical** 순서로 전체 단계를 모두 확인할 수 있다.
- Web UI의 SQL/DataFrame 탭에서 실행된 연산 단계를 선택하고 Details를 열면 텍스트 형태의 Physical Plan을 동일하게 확인할 수 있다.

## 7. Spark UDF

**UDF(User Defined Function)**는 DataFrame API나 SQL 함수만으로 처리하기 어려운 로직이 있을 때 Python 함수를 직접 정의해 컬럼 연산에 사용하는 방법이다.

```python
def grade(score):
    if score >= 90:
        return "A"
    elif score >= 70:
        return "B"
    else:
        return "C"

grade_udf = udf(grade, StringType())
result_df = df.withColumn("grade", grade_udf(col("score")))
```

**문제점**
- Executor 내부에서는 JVM 기반 Spark 연산과 Python Worker가 별도 프로세스로 동작하는데, Python UDF를 쓰면 이 둘 사이에 데이터를 주고받아야 한다. 이 과정에서 직렬화·역직렬화가 반복되고 파티션 단위로 변환 비용이 발생한다.
- Python으로 작성한 로직은 Catalyst Optimizer가 내용을 이해할 수 없어 **실행 계획을 최적화하지 못한다.** (Physical Plan에 `BatchEvalPython` 형태로 나타나며, 처리할 요소가 거의 안 보일 정도로 단순하게 찍힌다 — 그만큼 최적화 여지가 없다는 뜻)
- 반면 내장 함수(예: 산술 표현식)를 쓰면 Physical Plan에 표현식이 그대로 들어가 코드 생성·최적화가 쉽게 적용된다.

**해결 방안**
1. 가능하면 DataFrame API/SQL 내장 함수로 대체한다. (Catalyst Optimizer가 최적화할 수 있어 성능상 유리) — 예: 출생연도로 나이를 계산하는 단순 로직은 굳이 UDF를 만들 필요 없이 컬럼 연산으로 처리 가능.
2. UDF가 꼭 필요하다면 **Pandas UDF**(Spark 3.0+)를 고려한다. 행 단위로 처리하는 일반 Python UDF와 달리, 여러 행을 배치로 묶어 **Apache Arrow**(컬럼 기반 포맷, Java·Python 양쪽에서 효율적으로 접근 가능) 기반으로 처리하므로 데이터 변환 비용을 줄일 수 있다.

```python
@pandas_udf("int")
def calculate_age(birth_year: pd.Series) -> pd.Series:
    current_year = datetime.now().year
    return current_year - birth_year

df = df.withColumn("age_pandas_udf", calculate_age(col("birth_year")))
```

## 8. Spark Join 전략

Join을 수행할 때 데이터 크기, Join 조건(등가/비등가), Shuffle 필요 여부 등을 기준으로 여러 전략 중 하나가 선택된다. (등가조인 여부 → 힌트 존재 여부 → Broadcast 임계값보다 작은지 → SortMergeJoin 선호 여부 등을 순서대로 판단)

**기본 전략 (등가 조인에서 사용)**

| | Broadcast Hash Join (BHJ) | Sort Merge Join (SMJ) |
|---|---|---|
| 방식 | 작은 데이터프레임을 각 Executor에 Broadcast → Hash Table로 만들어 큰 데이터프레임과 Join | 양쪽 데이터프레임을 Join Key 기준으로 Shuffle → 정렬(Sort) → 병합(Merge) |
| Shuffle | 발생하지 않음 | 발생함 |
| 메모리 사용량 | 상대적으로 높음 (Broadcast 데이터를 메모리에 올려야 함) | 상대적으로 낮음 |
| 특징 | 조건이 맞으면 가장 빠름 | 대용량 데이터 간 Join에서 기본적으로 자주 선택됨 |

**기타 Join 전략**
- **Shuffle Hash Join(SHJ)**: 같은 Join Key를 가진 데이터가 같은 파티션에 오도록 Shuffle한 뒤, 상대적으로 작은 쪽을 Hash Table로 만들어 Join. Sort Merge Join보다 우선 선택되려면 별도 설정이 필요해 잘 쓰이지 않는다.
- **Broadcast Nested Loop Join(BNLJ)**: 작은 테이블을 Broadcast한 뒤 큰 테이블의 각 row와 비교하며 Join. 등가조인이 아닌 조건에서도 사용 가능하지만, 비교 횟수가 많아질수록 처리 속도가 느리다.
- **Shuffle Replicate Nested Loop Join**: 한쪽 데이터를 여러 파티션으로 복제한 뒤 Nested Loop 방식으로 비교. 등가조인이 아니거나 Broadcast하기엔 데이터가 클 때 선택될 수 있는, 비용이 큰 방식.

## 9. AQE (Adaptive Query Execution)

AQE는 Spark가 Job을 실행하는 동안 수집한 **실제 실행 정보**를 바탕으로 실행 계획을 **런타임에 동적으로 조정**하는 최적화 기능이다. (Spark 3.0+ 도입, `spark.sql.adaptive.enabled` 기본값 `true`)

실행 전 계획은 통계 정보를 바탕으로 세운 예상일 뿐이라 실제 데이터와 다를 수 있는데, AQE는 이를 보정해준다. 대표적으로 다음 세 가지를 수행한다.

1. **Shuffle 이후 작은 파티션 병합**: 너무 잘게 나뉜 Shuffle Partition을 병합해 불필요하게 많은 Reduce Task가 생기는 것을 줄인다. (예: Reduce Task 5개 필요 → 병합 후 3개로 감소)
2. **Join 전략 변경**: 실행 중 실제 데이터 크기를 보고, 예상과 달리 한쪽 테이블이 충분히 작다고 판단되면 Sort Merge Join을 Broadcast Hash Join으로 전환한다. (단, AQE가 켜져 있다고 항상 Broadcast로 바뀌는 것은 아니고 가능성이 생기는 정도로 이해하면 된다)
3. **Skew Join 최적화**: 특정 Key에 데이터가 몰려(Skewed Partition) 병렬 처리에 병목이 생기면, 그 파티션을 더 작은 단위로 split하고 반대쪽 Join 대상 파티션을 복제해 병렬 처리 부담을 줄인다.

실습에서 초기 Plan과 실제 실행 후 Final Plan을 비교해보면:
- Action(`collect()` 등) 실행 전에는 AdaptiveSparkPlan의 `isFinalPlan`이 `false`
- Action 실행 후에는 실제 통계를 반영한 Final Plan으로 바뀌고, `AQEShuffleRead` 같은 표시가 붙어 AQE가 동작했음을 확인할 수 있다.
- 초기 설정한 파티션 수(예: 20개)와 실제 처리에 사용된 Task 수가 달라졌는지 비교해보면 AQE가 무엇을 조정했는지 파악할 수 있다.

일반적으로 데이터 크기가 매우 거대한 경우가 아니라면 AQE는 별도 설정 없이도 잘 동작한다.

## 💡 한 줄 요약
> Spark 최적화는 결국 Shuffle(Wide Transform) 비용을 얼마나 줄이느냐의 문제이며, 실행 계획(explain, Web UI)을 확인하면서 직렬화 방식·파티션 수·Persist·Broadcast Join·AQE를 상황에 맞게 활용하는 것이 핵심이다.

## ❓ 더 찾아볼 것
- Catalyst Optimizer의 Logical Plan 최적화 규칙
- Pandas UDF와 Apache Arrow의 내부 동작 방식
- AQE의 세부 설정값 (advisoryPartitionSizeInBytes, skewedPartitionFactor 등)
