# [데이터 엔지니어링] Spark Streaming

---

## 1. Batch Processing vs Streaming Processing

- **Batch Processing**: 정해진 큰 데이터셋에 대해 한 번에 연산을 수행하는 형태.
- **Streaming Processing**: 끊임없이 들어오는 데이터의 흐름을 연속적으로 처리하는 형태.
- 실무에서는 Batch와 Streaming을 함께 사용해 서로의 약점을 보완하는 경우가 많다. (예: 배치 레이어와 실시간 레이어를 분리해서 운영하는 Lambda Architecture)

Spark는 API 수준(Level)에 따라 아래처럼 구분된다.

| | Batch | Streaming |
|---|---|---|
| **Structured API** (High Level) | SQL, Dataset, DataFrame | Structured Streaming |
| **Low Level API** | RDD | DStream |

## 2. DStream vs Structured Streaming

- **DStream API (Low Level)**: Spark RDD를 기반으로 처리하며, Input도 RDD, 처리되는 데이터도 RDD 형태가 된다. Micro Batch 모드로 동작하며, 일정 시간 동안 새롭게 들어온 데이터를 모아서 한 번에 처리한다. RDD 기반이기 때문에 Catalyst 최적화나 구조화된 스키마 처리 같은 이점을 누릴 수 없다는 한계가 있다.
- **Structured Streaming (High Level)**: DataFrame, Dataset API를 기반으로 하며 RDD를 직접 다루지 않는 만큼 더 많은 종류의 스트리밍 최적화 기술을 사용할 수 있다.
  - 데이터의 Stream을 **무한히 계속 추가되는 데이터의 테이블(Unbounded Table)** 개념으로 간주한다.
  - 새로운 데이터가 들어올 때마다 Input Table에 새로운 row가 append(추가)되고, 그 변화가 기존 상태(state)에 누적되어 Result Table이 계속 갱신되는 구조다.
  - 기본은 Micro-batch 처리지만 Continuous 모드도 지원한다. 다만 기능 제약이 있어 대부분 Micro-batch 모드를 사용한다.

## 3. Spark Streaming(Structured Streaming) 기본 구조

기본 작성 흐름은 일반 Spark Application과 비슷하지만, 정적 데이터의 `read`/`write` 대신 **`readStream`/`writeStream`**을 사용한다.

```
정적 데이터:   SparkSession → read → DataFrame → write → HDFS 등
스트리밍 데이터: SparkSession → readStream → DataFrame → writeStream → StreamingQuery → Sink(HDFS 등)
```

- `writeStream` 실행 결과로 **StreamingQuery** 객체가 생성된다.
- `awaitTermination()`을 호출해야 프로그램이 바로 종료되지 않고, 스트리밍 작업이 계속 실행되도록 대기한다. 실행 중에는 Spark가 백그라운드에서 소스(예: Kafka)에 새로운 데이터가 들어왔는지 지속적으로 확인하며, 새로운 데이터가 감지되면 `readStream`을 통해 읽고 정해진 처리 흐름에 따라 새로운 Micro-batch를 수행한다.
- **Streaming DataFrame**은 일반 DataFrame과 성격이 다르다. `count()`, `show()`, `collect()` 같은 Action 함수를 직접 호출할 수 없고, 반드시 `writeStream`을 통해 외부(Sink)로 내보내야 한다.

## 4. Micro-batch 처리 방식

Spark Streaming은 들어오는 실시간 데이터를 레코드 하나하나 즉시 처리하는 것이 아니라, 일정 시간 단위로 모아 작은 배치처럼 처리한다. 이렇게 생성되는 작은 처리 단위를 **Micro-batch**라고 부른다. 데이터 유입량이 많을수록 하나의 Micro-batch에서 처리하는 row 수와 처리 시간이 늘어난다.

- **maxOffsetsPerTrigger**: 스트리밍 소스(Kafka 등)에 데이터가 한꺼번에 많이 쌓이면 하나의 Micro-batch에서 처리할 데이터가 과도하게 많아져 처리 지연이나 Executor 메모리 사용량 증가로 이어질 수 있다. 이 옵션으로 한 번의 Trigger에서 읽어올 offset 수의 상한을 지정한다.
  - 이 값은 **파티션 하나의 상한이 아니라 전체 파티션에 대한 전체 읽기 상한**이다. Kafka Topic에 여러 Partition이 있으면, Spark가 각 Partition의 데이터 양을 고려해 읽을 offset 범위를 나누어 결정한다. (Checkpoint에 기록된 offset을 보면 Partition별 처리 건수가 다르게 나타날 수 있다)

## 5. Window와 Watermark

집계 결과가 시간이 지나도 계속 하나로만 쌓이지 않도록, Structured Streaming에서는 **Window**와 **Watermark** 개념을 함께 사용한다.

- **Window**: 데이터를 하나의 집계 단위로 묶을 특정 시간 구간. 예를 들어 1분 단위 Window라면 `00분00초~00분59초`, `01분00초~01분59초` 식으로 구간을 나누고, 타임스탬프가 각 구간에 속하는 데이터끼리 하나의 그룹으로 묶어 집계한다.
- **Watermark**: 늦게 도착하는 데이터를 얼마나 기다려줄지 정하는 기준. 예를 들어 어떤 데이터가 1분이 지난 뒤에야 도착했더라도, Watermark를 1분으로 설정해두면 최대 1분까지 늦게 도착한 데이터도 해당 Window 집계에 반영해준다. 동시에 너무 오래된 상태(state)를 언제 정리할지 결정하는 기준이 되기도 한다.

```python
result_df = (parsed_df
    .withWatermark("timestamp", "1 minute")
    .groupBy(window("timestamp", "1 minute"))
    .count()
)
```

## 6. Kafka 데이터를 Spark Structured Streaming으로 읽기

```python
kafka_source_df = spark.readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "kafka:9092") \
    .option("subscribe", "spark-streaming-test") \
    .option("startingOffsets", "latest") \
    .load()

# Kafka 메시지의 key, value를 문자열로 변환
message_df = kafka_source_df.selectExpr(
    "CAST(key AS STRING) AS key",
    "CAST(value AS STRING) AS value"
)
```

**startingOffsets**
- Spark Streaming Job이 처음 시작될 때 Kafka Topic의 어느 offset부터 읽을지 지정하는 옵션. 기본값은 `latest`이므로, 별도 설정이 없으면 Job 시작 이후 새로 들어오는 레코드부터 처리한다.
- JSON 형식으로 Topic/Partition별 offset을 세밀하게 지정할 수도 있다.

```python
# topicA의 0번 파티션 → offset 5부터, 1번 파티션 → latest부터
# topicB의 0번 파티션 → earliest부터
kafka_source_df = spark.readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "kafka:9092") \
    .option("subscribe", "topicA, topicB") \
    .option(
        "startingOffsets",
        """{"topicA":{"0":5,"1":-1},"topicB":{"0":-2}}"""
    ) \
    .load()
```

## 7. writeStream 함수와 Sink

`writeStream`을 호출하면 `DataStreamWriter` 객체를 사용할 수 있다. 이 객체는 스트리밍 데이터를 **어디에, 어떤 방식으로, 어떤 주기로** 출력할지 설정하는 역할을 하며, 마지막에 `start()`를 호출해야 실제로 스트리밍 쿼리가 실행된다.

| 함수 | 설명 |
|---|---|
| `format()` | 출력 대상 Sink 지정 |
| `option()` | Sink별 세부 옵션 지정 |
| `outputMode()` | 결과를 어떤 방식으로 출력할지 지정 |
| `trigger()` | 마이크로 배치 실행 주기 설정 |
| `foreachBatch()` | 마이크로 배치 단위로 사용자 정의 처리 |
| `foreach()` | Row 단위로 사용자 정의 처리 |
| `toTable()` | 결과를 테이블에 저장 |
| `start()` | 스트리밍 쿼리 실행 시작 |

```python
query = (
    message_df.writeStream
    .format("console")
    .option("checkpointLocation", "/tmp/spark-checkpoint/kafka-console")
    .option("truncate", "false")
    .outputMode("append")
    .trigger(processingTime="5 seconds")
    .start()
)
```

**Sink**: `readStream`으로 읽은 Streaming DataFrame을 `writeStream`을 통해 외부 대상으로 내보내는 출력 지점. 유형으로는 `console`, `kafka`, `file`, `memory`, `foreachBatch` 등이 있다.

```python
query = (
    message_df.writeStream
    .format("kafka")
    .option("kafka.bootstrap.servers", "kafka:9092")
    .option("topic", "spark-streaming-test2")
    .option("checkpointLocation", "/tmp/spark-checkpoint/kafka-sink-output")
    .outputMode("append")
    .start()
)
```

## 8. ForeachBatch

`foreachBatch`는 Streaming DataFrame의 각 Micro-batch를 사용자가 정의한 함수로 전달해 처리하는 방식이다. 전달된 함수 안에서 각 batch에 대한 처리·출력·저장 로직을 직접 작성할 수 있으며, 함수로 전달되는 `batch_df`는 해당 Micro-batch 시점의 **정적(static) DataFrame**이므로 일반 DataFrame처럼 `count()`, `show()` 등을 자유롭게 사용할 수 있다.

```python
def write_batch(batch_df: DataFrame, batch_id: int):
    print(f"batch_id: {batch_id} start")
    cnt = batch_df.count()
    print(f"데이터 건수: {cnt}")
    batch_df.show(truncate=False)
    print(f"batch_id: {batch_id} end")

query = (
    kafka_source_df.writeStream
    .foreachBatch(write_batch)
    .option("checkpointLocation", "/tmp/spark-checkpoint/foreach-batch")
    .start()
)
```

## 9. 실습: Kafka 메시지를 Spark로 읽어 타임스탬프별 집계하기

**실습 전 체크리스트**
- Kafka 실행 중인지 확인 (topic 존재 여부 확인 겸)
  ```bash
  kafka/bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
  ```
- Spark 3.5.4 설치
- PySpark 환경 설정 완료 (`pip install pyspark==3.5.4`)
- 필요한 패키지는 실습 코드의 `spark.jars.packages` 옵션으로 자동 다운로드됨

**① spark_producer.py** — Kafka Producer로 다양한 시간대의 타임스탬프를 가진 메시지 생성해서 전송

```python
producer = KafkaProducer(
    bootstrap_servers=['localhost:9092'],
    value_serializer=lambda v: json.dumps(v).encode('utf-8')
)

base_time = datetime.now() - timedelta(minutes=5)  # 기준 시간: 현재 -5분

for i in range(30):
    # 0~4분, 0~59초 사이의 임의 시점 생성 (1분 단위 집계를 위해)
    offset_minutes = random.randint(0, 4)
    offset_seconds = random.randint(0, 59)
    msg_time = base_time + timedelta(minutes=offset_minutes, seconds=offset_seconds)

    message = {
        'id': i,
        'message': f'테스트 메시지 {i}',
        'timestamp': msg_time.strftime('%Y-%m-%d %H:%M:%S')
    }
    producer.send('test-topic', message)
    time.sleep(0.5)  # 너무 빠르게 보내지 않도록

producer.flush()
```

**② spark_kafka_process.py** — Spark로 타임스탬프별 집계 처리

```python
def process_kafka_stream():
    spark = create_spark_session()

    # Kafka 스트림 읽기
    df = (
        spark.readStream
        .format("kafka")
        .option("kafka.bootstrap.servers", "localhost:9092")
        .option("subscribe", "test-topic")
        .option("startingOffsets", "latest")
        .load()
    )

    value_df = df.selectExpr("CAST(value AS STRING)")

    schema = StructType([
        StructField("timestamp", TimestampType(), True),
        StructField("message", StringType(), True)
    ])

    # JSON 파싱
    parsed_df = (
        value_df
        .select(from_json(col("value"), schema).alias("data"))
        .select("data.*")
    )

    # 1분 윈도우 + watermark + count
    result_df = (
        parsed_df
        .withWatermark("timestamp", "1 minute")
        .groupBy(window("timestamp", "1 minute"))
        .count()
    )

    # 콘솔 출력
    query = (
        result_df.writeStream
        .outputMode("complete")
        .format("console")
        .start()
    )

    query.awaitTermination()
```

**실습 순서**
1. `spark_producer.py` 실행
2. `spark_kafka_process.py` 실행 → 출력되는 집계 결과 확인
3. 필요 시 아래 명령으로 토픽에 타임스탬프가 잘 들어오는지 직접 확인 가능
   ```bash
   bin/kafka-console-consumer.sh --topic {토픽명} --bootstrap-server localhost:9092 --from-beginning
   ```

**결과 확인**
실행 결과를 보면 Batch가 진행될수록(Batch 0 → 1 → 2 → 3) 같은 Window의 count 값이 점점 누적되어 증가한다. 즉 Structured Streaming은 이전 결과를 버리지 않고, 새로 들어오는 데이터를 기존 상태(state)에 계속 반영해 Result Table을 갱신한다.

```
Batch: 0                     Batch: 1                      Batch: 2
(초기 - 아직 집계 없음)        window | count                 window | count
                              {13:07...} | 2                 {13:07...} | 5
                              {13:04...} | 2                 {13:04...} | 2
                              {13:03...} | 2                 {13:03...} | 4
                              {13:05...} | 2                 {13:05...} | 4
                              {13:06...} | 1                 {13:06...} | 4
```

## 💡 한 줄 요약
> Spark Structured Streaming은 들어오는 데이터를 계속 append되는 무한 테이블(Unbounded Table)로 보고 Micro-batch 단위로 처리하며, Window와 Watermark를 함께 사용해 시간 구간별 집계와 지연 도착 데이터 처리를 관리한다.

## ❓ 더 찾아볼 것
- Structured Streaming의 outputMode(append / update / complete) 차이
- Watermark 설정에 따른 State Store 정리(cleanup) 동작
- Continuous Processing 모드의 구체적인 기능 제약 사항
