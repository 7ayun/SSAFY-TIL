# [데이터 엔지니어링] SparkSubmitOperator

---

## 1. SparkSubmitOperator란

SparkSubmitOperator는 Apache Airflow에서 Spark 애플리케이션을 실행하기 위한 전용 Operator다. 터미널에서 `spark-submit` 명령어로 `.py`/`.jar`/`.scala` 파일을 Spark 클러스터(Standalone, YARN, Kubernetes 등)에 제출하는 것과 동일한 과정을, DAG 안에서 하나의 Task로 정의할 수 있게 해준다.

중요한 점은 **Airflow가 직접 Spark 연산을 수행하는 것이 아니라는 것**이다. Airflow(정확히는 SparkSubmitOperator)는 Spark 작업을 제출하는 역할만 담당하고, 실제 데이터 처리는 Spark 클러스터(Master/Worker)가 수행한다. Airflow는 그 작업이 성공했는지, 실패했는지, 재시도해야 하는지를 추적·관리하는 오케스트레이션 계층 역할을 한다.

**주요 파라미터**
- `application`: 실행할 Spark 애플리케이션 경로 (`.py`, `.jar` 등). Airflow가 접근 가능한 경로여야 하며, Spark 실행 환경도 같은 경로에서 파일을 읽을 수 있어야 한다.
- `conn_id`: Spark 클러스터 연결 정보 (예: `spark_default`) — **필수**
- `conf`: Spark 설정 (예: `"spark.executor.memory": "2g"`)
- `executor_memory`, `driver_memory`: 리소스 지정
- `application_args`: 애플리케이션에 전달할 인자 목록
- `name`: Spark 작업 이름

이 중 반드시 필요한 것은 **`conn_id`(연결 정보)**와 **`application`(실행 파일 경로)** 두 가지이며, 나머지는 기본값으로 세팅될 수 있다.

## 2. Docker 환경 구성하기

Airflow에서 Spark 작업을 제출하려면 Airflow 컨테이너가 Spark 클라이언트를 사용할 수 있는 상태여야 한다. 즉 Airflow용 Dockerfile에 Spark 다운로드·설치 과정이 포함되어 있어야 `spark-submit` 명령을 실행할 수 있다. (Airflow에 Spark를 직접 설치하지 않고 별도 클러스터 매니저를 통해 제출하는 방식도 있지만, 실습에서는 직관적인 이해를 위해 Airflow 컨테이너에서 바로 제출하는 구성을 사용했다.)

`docker-compose.yaml`에 Spark Master/Worker 서비스를 추가한다.

```yaml
spark-master:
  build:
    context: .
    dockerfile: Dockerfile.spark
  container_name: spark-master
  environment:
    - SPARK_MODE=master
    - SPARK_MASTER_HOST=spark-master
    - SPARK_RPC_AUTHENTICATION_ENABLED=no
    - SPARK_RPC_ENCRYPTION_ENABLED=no
    - SPARK_LOCAL_STORAGE_ENCRYPTION_ENABLED=no
    - SPARK_SSL_ENABLED=no
  ports:
    - "8083:8080"
    - "7077:7077"
  networks:
    - airflow
  command: /bin/bash -c "/opt/spark/sbin/start-master.sh && tail -f /dev/null"
  volumes:
    - ${AIRFLOW_PROJ_DIR:-.}/dags/scripts:/opt/airflow/dags/scripts
    - ${AIRFLOW_PROJ_DIR:-.}/output:/opt/airflow/output
    - ${AIRFLOW_PROJ_DIR:-.}/data:/opt/airflow/data

spark-worker:
  build:
    context: .
    dockerfile: Dockerfile.spark
  container_name: spark-worker
  environment:
    - SPARK_MODE=worker
    - SPARK_MASTER_URL=spark://spark-master:7077
  depends_on:
    - spark-master
  ports:
    - "8084:8081"
  networks:
    - airflow
  command: /bin/bash -c "sleep 5; /opt/spark/sbin/start-worker.sh ${SPARK_MASTER_URL} && tail -f /dev/null"
  volumes:
    - ${AIRFLOW_PROJ_DIR:-.}/dags/scripts:/opt/airflow/dags/scripts
    - ${AIRFLOW_PROJ_DIR:-.}/output:/opt/airflow/output
    - ${AIRFLOW_PROJ_DIR:-.}/data:/opt/airflow/data
```

- `spark-master`, `spark-worker` 컨테이너명을 명확히 지정해, Worker의 `SPARK_MASTER_URL`이 Master 컨테이너를 DNS 이름으로 바라보게 한다. 컨테이너의 IP는 재기동할 때마다 바뀔 수 있으므로 DNS 이름(컨테이너명) 기반 통신이 안전하다.
- `dags/scripts` 볼륨을 마운트해, Airflow가 제출하는 PySpark 파일을 Spark 실행 환경(Master)도 같은 경로에서 읽을 수 있게 한다. 이 경로가 맞지 않으면 Airflow는 작업을 제출했지만 Spark 쪽에서 파일을 찾지 못하는 문제가 발생한다.
- **네트워크**: Airflow와 Spark를 각각 다른 Docker Compose 파일로 띄우더라도 컨테이너 간 통신이 가능하도록, 모든 서비스를 동일한 네트워크(예: `airflow` 혹은 `sharednet`)에 연결해야 한다.

**JAVA_HOME 설정**: `spark-submit`을 실행하려면 Java(JVM)가 세팅되어 있어야 하는데, Airflow 기본 이미지에는 JVM이 포함되어 있지 않다. Dockerfile 또는 docker-compose의 environment에 `JAVA_HOME`을 추가해야 한다.

```yaml
environment:
  &airflow-common-env
  AIRFLOW__CORE__EXECUTOR: CeleryExecutor
  AIRFLOW__DATABASE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@postgres/airflow
  # ...
  JAVA_HOME: /usr/lib/jvm/java-17-openjdk-amd64
```

Dockerfile(이미지 빌드 시점)에 세팅할지, docker-compose(컨테이너 기동 시점)에 세팅할지는 상황에 따라 선택한다. 이미지 빌드 시점에 넣으면 그 이미지를 쓰는 순간 이미 적용되어 있고, 컨테이너 기동 시점에 넣으면 컨테이너마다 별도로 세팅해야 한다.

**폴더 권한**: 볼륨 마운트한 디렉토리의 권한 문제로 실행이 막힐 경우, 임시로 `chmod -R 777`을 output/data 디렉토리에 적용해 우선 동작 여부부터 확인하는 방법도 있다(실제 운영 환경에서는 최소 권한 원칙에 맞게 조정이 필요하다).

## 3. Spark Connection 등록 및 실행

Web UI Admin → Connections에서 Spark용 Connection을 추가한다.

- Connection Id: `spark_default`
- Connection Type: `Spark`
- Host: `spark://spark-master`
- Port: `7077`
- Deploy mode: `client`
- Spark binary: `spark-submit`

```python
from airflow import DAG
from airflow.providers.apache.spark.operators.spark_submit import SparkSubmitOperator
from datetime import datetime

with DAG(
    dag_id='spark_submit_example',
    start_date=datetime(2024, 1, 1),
    schedule_interval=None,
    catchup=False,
    tags=['spark'],
) as dag:

    submit_job = SparkSubmitOperator(
        task_id='spark_submit_task',
        application='/opt/airflow/dags/scripts/spark_wordcount.py',
        conn_id='spark_default',
        conf={'spark.master': 'spark://spark-master:7077'},
        verbose=True,
    )
```

`spark_wordcount.py`는 이름별 평균 점수를 계산하는 간단한 PySpark 코드다.

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, avg

spark = SparkSession.builder \
    .appName("DataFrameTest") \
    .master("spark://spark-master:7077") \
    .getOrCreate()

data = [
    ("Alice", "Math", 85),
    ("Bob", "Math", 90),
    ("Alice", "English", 78),
    ("Bob", "English", 83),
    ("Alice", "Science", 92),
    ("Bob", "Science", 87),
]
columns = ["name", "subject", "score"]

df = spark.createDataFrame(data, columns)

avg_scores = df.groupBy("name").agg(avg("score").alias("average_score"))
avg_scores.show()

spark.stop()
```

DAG를 실행하면 SparkSubmitOperator가 이 파일을 애플리케이션으로 제출하고, 로그에서 Spark 실행 과정과 `average_score` 결과(GroupBy한 평균 점수 DataFrame)를 확인할 수 있다. Spark 관련 로그(INFO)가 많이 출력되므로, 최종 출력물(`show()` 결과)을 기준으로 확인하면 된다.

## 4. Spark UI로 실행 결과 확인하기

`http://localhost:8083`으로 접속하면 Spark Master UI를 확인할 수 있다. 8080이 아닌 8083을 쓰는 이유는 Airflow가 이미 8080 포트를 사용 중이라 포트 충돌을 피하기 위해 변경했기 때문이다. (Worker UI는 8084)

Master UI에서 확인 가능한 정보:
- 제출된 Application 이름, ID, 상태(FINISHED 등), 제출 시각
- 사용한 Executor 코어 수, 메모리
- Worker 연결 상태, 코어/메모리 가용량

Worker UI(8084)는 컨테이너 DNS 이름 기반으로 링크가 걸려 있어 로컬 브라우저에서 직접 클릭하면 접속이 안 될 수 있다는 점도 참고할 부분이다.

## 💡 한 줄 요약
> SparkSubmitOperator는 Spark 작업 제출을 Airflow Task로 통합해 스케줄링·추적할 수 있게 해주는 Operator이며, 실제 연산은 Spark 클러스터가 수행하고 Airflow는 오케스트레이션만 담당한다.

## ❓ 더 찾아볼 것
- Airflow에 Spark를 직접 설치하지 않고 YARN/Kubernetes 클러스터 매니저로 제출하는 방식
- SparkSubmitOperator의 `application_args`, `executor_memory` 등 세부 옵션 활용
- Docker 네트워크(`sharednet`)로 여러 Compose 파일의 컨테이너를 연결하는 방법
