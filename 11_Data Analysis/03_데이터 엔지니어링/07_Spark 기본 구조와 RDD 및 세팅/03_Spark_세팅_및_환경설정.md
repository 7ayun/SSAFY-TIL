# [데이터 엔지니어링] Spark 설치 및 실행 환경설정

---

## 1. Spark 설치 (WSL 환경)
- 실습은 WSL(Ubuntu) 환경을 기준으로 진행한다. Kafka를 설치할 때와 마찬가지로 Java(JVM)가 먼저 세팅되어 있어야 하는데, Spark 역시 JVM 기반으로 동작하는 프레임워크이기 때문이다.

```bash
wget https://dlcdn.apache.org/spark/spark-3.5.4/spark-3.5.4-bin-hadoop3.tgz
tar -xvzf spark-3.5.4-bin-hadoop3.tgz
sudo mv spark-3.5.4-bin-hadoop3 /usr/local/spark
```
- 압축을 풀면 생기는 디렉토리 안에는 `bin`, `sbin`, `conf` 등 스파크 실행·설정에 필요한 하위 디렉토리들이 기본적으로 정리되어 있다.

## 2. 환경 변수 등록 및 확인
- `SPARK_HOME`을 환경 변수로 등록해두면 스파크 관련 명령어를 어디서든 사용할 수 있다.

```bash
vi ~/.bashrc
# 아래 내용 추가
export SPARK_HOME=/home/ssafy/spark
export PATH=$SPARK_HOME/bin:$SPARK_HOME/sbin:$PATH

source ~/.bashrc
echo $SPARK_HOME   # 정상 설정 확인
```
- 환경 변수 등록 후 `spark-shell`을 실행해 정상적으로 실행되는지 확인한다.
- `spark-shell`로 진입하면 기본적으로 **Scala** 기반 셸이 뜬다. 이번 과정에서는 Scala를 다루지 않으므로 `Ctrl+C`로 빠져나오면 된다.

## 3. PySpark 설치
```bash
pip install pyspark==3.5.4
pyspark   # 정상 설치 확인
```
- Spark 전체 패키지(hadoop 포함)를 이미 설치해 `SPARK_HOME`을 잡아두었다면 `pyspark` 명령이 바로 실행되기 때문에, 사실 pip으로 별도 설치하지 않아도 동작 자체에는 문제가 없다.
- 그럼에도 pip으로 다시 설치하는 이유는 IDE에서 로컬 개발 시 코드 인식·에러 검출을 위한 인터프리터 인식 목적 때문이다.
- 가상환경(virtualenv)에서 pyspark를 설치하면, 가상환경 경로가 PATH 앞쪽에 추가되어 가상환경의 pyspark가 우선적으로 인식된다. 가상환경을 비활성화하면 다시 로컬에 다운로드해둔 Spark 경로가 잡힌다. 코드가 어떤 인터프리터에서 인식되고 있는지에 따라 세팅이 꼬일 수 있으므로 주의가 필요하다.

## 4. pyspark 셸에서 간단한 예제 실행
```python
# a=5, b=10일 때 a+b, a*b 값 출력
>>> from pyspark.sql import SparkSession
>>> spark = SparkSession.builder.appName("App").getOrCreate()
>>> a = 5
>>> b = 10
>>> print(a + b)
15
>>> print(a * b)
50
>>> spark.stop()
```
- `pyspark` 셸에 진입하면 SparkSession이 자동으로 준비된 상태(`spark` 변수로 바로 사용 가능)로 시작된다.
- 이 상태에서는 기본적인 파이썬 인터프리터처럼 동작하면서, 동시에 Spark 기능도 함께 사용할 수 있는 게 특징이다.

## 5. PySpark 실행 원리
- 겉보기에는 파이썬에서 실행되는 것처럼 보이지만, 실제로는 **Py4J**를 통해 파이썬 드라이버와 JVM 기반 Spark 객체가 서로 통신한다.

```
Python 코드 (PySpark API)
      ↓ Py4J
JVM: SparkSession / SparkContext (Driver)
      ↓
Cluster Manager → Executors
      ↓
분산 데이터 처리
```

- 파이썬 드라이버에서 실행 계획을 만들면, Cluster Manager가 필요한 자원을 할당해주고 그 위에서 실제 분산 처리가 이루어진다.
- `pyspark` 셸을 통해 실행할 때는 내부적으로 `spark-submit` 명령이 자동으로 호출되는 것처럼 동작한다.

## 6. 실행 방식: python 명령 vs spark-submit
스크립트를 실행하는 방법은 크게 두 가지다.

| 방식 | 설명 |
|---|---|
| `python example.py` | 별도의 제출(submit) 명령 없이 파이썬 인터프리터로 직접 실행. 실습·간단한 테스트에 적합 |
| `spark-submit example.py` | 스파크의 정식 제출 도구를 사용해 실행. 세부 실행 옵션을 지정할 수 있어 실제 운영·배포 시 일반적으로 사용하는 방식 |

- 실습 단계에서는 두 방식의 차이를 크게 느끼기 어렵지만, 실제로 운영·배포하는 상황이라면 옵션을 세밀하게 지정할 수 있는 `spark-submit` 형태로 잡을 제출하는 것이 일반적이다.

## 7. sc.textFile()을 활용한 파일 읽기
```bash
# 1. 터미널에서 테스트 파일 생성
echo -e "Hello Spark\nApache Spark is powerful\nBig Data Processing" > test.txt
```
```python
# 2. Spark Shell / PySpark에서 실행
>>> rdd = sc.textFile("test.txt")
>>> rdd.foreach(print)
Hello Spark
Apache Spark is powerful
Big Data Processing

# 3. 현재 파티션 개수 확인
>>> rdd.getNumPartitions()
>>> print("현재 파티션 개수:", rdd.getNumPartitions())
현재 파티션 개수: 2

# 4. Spark에서 데이터를 하나의 파티션에서 처리 (repartition)
>>> rdd_repartitioned = rdd.repartition(1)
>>> print("변경된 파티션 수:", rdd_repartitioned.getNumPartitions())
변경된 파티션 수: 1
```
- `sc.textFile()`은 텍스트 파일을 읽어 RDD로 변환하는 기본적인 함수다. 로컬 파일뿐 아니라 HDFS, S3 같은 분산 스토리지 경로도 그대로 지정해서 읽어올 수 있다.
- `repartition(N)`은 파티션 수를 N개로 재조정하는 연산으로, 여러 파티션에 있던 데이터를 다시 나누거나 한 곳으로 모아야 하기 때문에 내부적으로 **셔플**이 발생한다. 데이터 양·분포에 따라 파티션마다 데이터가 균등하게 들어간다는 보장이 없고 셔플 비용이 크기 때문에, 습관적으로 사용하는 것은 권장되지 않는다.

## 8. sc.textFile() 활용 예제 (Action / Transformation)
```python
# 1. 데이터 변환 후 결과 확인 (collect — Action)
>>> rdd = sc.textFile("file:///usr/local/spark/data/test.txt")
>>> result = rdd.collect()
>>> print(result)
['Hello Spark', 'Apache Spark is powerful', 'Big Data Processing']

# 2. 줄 개수 세기 (count — Action)
>>> line_count = rdd.count()
>>> print("줄 개수:", line_count)
줄 개수: 3

# 3. 특정 단어가 포함된 줄 필터링 (filter — Transformation)
>>> filtered_rdd = rdd.filter(lambda line: "Spark" in line)
>>> filtered_rdd.foreach(print)
Hello Spark
Apache Spark is powerful

# 4. 모든 단어를 소문자로 변환하기 (map — Transformation)
>>> lower_rdd = rdd.map(lambda line: line.lower())
>>> lower_rdd.foreach(print)
hello spark
apache spark is powerful
big data processing
```
- `collect()`는 RDD의 모든 데이터를 드라이버 한 곳으로 모아 배열 형태로 반환하는 Action이다. 결과 확인용으로는 유용하지만, 큰 데이터셋에서 사용하면 드라이버에 과부하가 걸릴 수 있어 사용을 지양해야 한다.
- `count()`는 RDD 전체 요소(줄) 개수를 세는 Action으로, 연산 결과를 즉시 확인할 수 있다.
- `filter()`는 조건을 만족하는 요소만 남기는 Transformation, `map()`은 각 요소에 함수를 적용해 새로운 RDD를 만드는 Transformation이다.

## 9. Docker Compose를 활용한 Spark 클러스터 구성
- Spark 버전에 맞는 이미지를 Dockerfile로 준비하고, docker-compose.yaml로 마스터·워커 컨테이너를 구성해 클러스터 모드로 실행할 수 있다.

```dockerfile
FROM apache/spark:3.5.4-java17-python
USER root

# 파이썬 심볼릭 링크 설정
RUN ln -sf $(which python3) /usr/bin/python

# 필요 패키지 설치
RUN python -m pip install --no-cache-dir --upgrade pip && \
    python -m pip install --no-cache-dir pandas==2.2.1

USER spark
```
```yaml
# docker-compose.yaml (일부)
services:
  spark-master:
    build:
      context: .
      dockerfile: dockerfile.spark
    container_name: spark-master
    volumes:
      - ./jobs:/opt/spark/jobs
      - ./data:/opt/spark/data
      - ./output:/opt/spark/output
    networks:
      dz_net:
        driver: bridge
```
- Dockerfile에서 잠시 `root` 권한으로 전환하는 이유는 이미지 내 기본 사용자에게 설치 권한이 없기 때문이다. 파이썬 심볼릭 링크를 잡아주고, 필요하다면 pandas 같은 패키지를 설치한 뒤 다시 원래 사용자(`spark`)로 전환한다.
- 볼륨으로 연결한 `jobs`, `data`, `output` 디렉토리는 `docker compose up` 실행 전에 **미리 만들어 두는 것이 안전**하다. 디렉토리가 없으면 Docker가 root 권한으로 자동 생성하는데, 이때 호스트 사용자(UID)와 컨테이너 내부 사용자(UID)가 일치하지 않으면 Permission Denied 오류가 발생할 수 있기 때문이다.
- 컨테이너는 마스터용 1개, 워커용 1개로 구성하며, 워커를 여러 개로 늘리고 싶다면 worker1, worker2, worker3처럼 서비스를 추가하면 된다.

## 10. Docker 기반 Spark 작업 실행
```bash
# 컨테이너 외부에서 spark-submit 실행 (docker exec)
docker exec spark-master \
  /opt/spark/bin/spark-submit \
  --master spark://spark-master:7077 \
  /opt/spark/jobs/example.py
```
- Docker 컨테이너 내부에서는 `spark://spark-master:7077`처럼 서비스명을 DNS 네임으로 사용해 컨테이너끼리 통신할 수 있다.
- `docker exec` 명령으로 컨테이너 외부에서 곧바로 `spark-submit` 명령을 실행할 수도 있고, 컨테이너 내부로 직접 접속(`docker exec -it spark-master bash`)한 뒤 동일한 명령을 실행할 수도 있다. 어떤 방식이든 결과는 동일하다.
- 예시 스크립트(`example.py`)는 SparkSession을 생성하고, 간단한 DataFrame을 만들어 출력한 뒤 세션을 종료하는 형태로 구성되어 있다.

```python
from pyspark.sql import SparkSession

# 스파크 세션 생성
spark = SparkSession.builder.appName("Example").getOrCreate()

# 샘플 데이터
data = [("Alice", 20), ("Bob", 30), ("Cathy", 40)]
columns = ["name", "age"]

# 데이터프레임 생성
df = spark.createDataFrame(data, columns)

# 결과 출력
df.show()

# 스파크 종료
spark.stop()
```

## 💡 한 줄 요약
> Spark는 WSL 로컬 환경뿐 아니라 Docker Compose 기반의 마스터-워커 클러스터로도 손쉽게 구성할 수 있으며, `python` 실행과 `spark-submit` 실행의 차이, 볼륨·사용자 권한(UID) 이슈를 이해하는 것이 환경설정의 핵심이다.

## ❓ 더 찾아볼 것
- `spark-submit`의 다양한 실행 옵션 (`--executor-memory`, `--num-executors` 등)
- Docker 컨테이너 간 사용자 권한(UID) 불일치 문제의 근본적인 해결 방법
- Spark Standalone 클러스터에서 워커 노드를 여러 개로 확장하는 방법
