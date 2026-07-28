# [데이터 엔지니어링] Spark 개요 및 실행 구조

---

## 1. Spark란?
- Apache Spark는 대규모 데이터를 분산 환경에서 빠르게 처리할 수 있도록 만들어주는 오픈소스 통합 분석 엔진이다.
- 원래는 배치 처리를 위해 설계되었지만, 마이크로 배치(micro-batch) 방식을 통해 스트리밍 처리도 어느 정도 지원한다. 완전한 실시간까지는 아니지만 거의 실시간에 가깝게 처리할 수 있다.
- SQL 기반 처리, 스트리밍, 머신러닝, 그래프 분석 등 다양한 워크로드를 하나의 실행 엔진 위에서 처리할 수 있도록 만들어졌다.
- UC Berkeley AMPLab에서 개발되어 2010년 오픈소스로 공개되었고, 2014년 아파치 재단의 정식 프로젝트로 채택되었다. 개발진이 창업한 회사가 데이터브릭스(Databricks)다.

## 2. Spark가 필요해진 배경
- 2000년대 초반~2010년대 초반까지는 Hadoop MapReduce 기반의 배치 처리가 대용량 데이터 처리의 근간이었다.
- 이 방식의 문제점
  - **디스크 기반 처리로 인한 속도 저하**: 중간 결과를 디스크에 쓰고 다시 읽는 과정을 반복해서, 메모리보다 훨씬 느린 디스크 I/O로 인한 병목이 자주 발생
  - **반복 작업의 비효율성**: 머신러닝, 스트리밍 분석처럼 반복적이고 실시간성이 필요한 작업에 구조적으로 적절하지 못함
  - **단일 서버의 확장성 부족**: 기존엔 서버 성능을 높이는 스케일 업(수직 확장) 위주였는데, 데이터가 증가하면 하드웨어의 물리적 한계에 부딪힘 → 여러 서버를 묶는 스케일 아웃(수평 확장)의 필요성 증가
  - SQL 분석, 그래프 분석, 머신러닝, 실시간 처리 등 특정 목적에 맞는 시스템들을 하둡과 개별적으로 연동해서 사용하는 "하둡 에코시스템(Hadoop ecosystem)" 구조 자체가 지나치게 복잡함
- 이런 문제들을 해결하기 위해, 다양한 워크로드를 하나의 엔진에서 처리할 수 있는 통합 플랫폼으로 Spark가 등장했다.

## 3. Spark의 설계 철학
| 항목 | 내용 |
|---|---|
| 속도 | 메모리 기반 연산(In-Memory Computation)으로 디스크 I/O 최소화, DAG 기반 스케줄링으로 실행 계획 최적화, Tungsten 엔진을 통한 코드 생성 최적화 |
| 사용 편리성 | 단일 PC와 클러스터 간 코드 차이가 최소화된 추상화 구조, RDD → DataFrame → Dataset의 계층적 API 제공, Scala/Python/Java/R 등 다양한 언어 지원 |
| 모듈성 | Spark SQL, Streaming, MLlib, GraphX 등 다양한 워크로드를 하나의 엔진(Spark Core) 위에서 처리 |
| 확장성 | 다양한 데이터 소스(HDFS, Cassandra, MongoDB, RDBMS, S3 등)와 연동, csv·parquet·json 등 여러 파일 포맷 지원, 수많은 서드파티 패키지와 통합 가능 |

- **속도**: 반복 사용하는 데이터는 캐싱(caching)이나 퍼시스트(persist)로 등록해 재사용할 수 있게 만들어, 반복 연산에서 강점을 가진다. 또한 DAG(방향은 있지만 순환하지 않는 그래프) 기반으로 Transformation·Action의 실행 계획을 표현하고 스케줄링한다. Catalyst Optimizer, Tungsten 엔진 등을 통해 메모리 관리·CPU 효율 측면에서도 저수준 최적화를 지원한다(구체적인 동작 방식은 이후 데이터프레임 학습 시 다룸).
- **사용 편리성**: PySpark에서는 Dataset을 사용할 수 없는데, 이는 Python이 동적 타입 언어라 컴파일 시점 타입 검사가 필요한 Dataset의 타입 안정성이 적용될 수 없기 때문이다.
- **모듈성**: 서로 다른 영역의 라이브러리(SQL, Streaming, MLlib, GraphX)가 하나의 통합된 엔진 위에서 작동하기 때문에, 하둡 에코시스템처럼 개별 시스템을 따로 연동할 필요가 없다.

## 4. Spark의 주요 컴포넌트
모든 컴포넌트는 Spark Core 위에서 실행되며, 공통 실행 엔진과 스케줄러를 공유한다.

| 컴포넌트 | 설명 |
|---|---|
| Spark Core | 핵심 실행 플랫폼 |
| Spark SQL | 구조적 데이터 처리 및 SQL 기반 쿼리 실행 |
| Spark Streaming | 실시간 데이터 분석을 위한 스트리밍 처리(마이크로 배치 방식) |
| MLlib | 머신러닝 알고리즘 라이브러리(분류, 회귀, 군집 등) |
| GraphX | 그래프 기반 데이터 처리와 분석 지원(PageRank 등) |

- 전통적인 Spark Streaming은 마이크로 배치 방식으로 동작하며, 이를 업그레이드한 Structured Streaming은 DataFrame 기반으로 구조화된 스트리밍 처리를 지원한다. DataFrame 기반 사용법을 익혀두면 배치·실시간 처리를 모두 활용할 수 있다.

## 5. Spark의 활용 시 주의점
- 마이크로 배치 기반이기 때문에 엄밀한 실시간 처리는 완벽하게 이루어지기 어렵다. 어느 정도까지 지연을 허용할 수 있는지 고려가 필요하다.
- 빅데이터 처리 플랫폼으로 설계되어 있어, 오히려 작은 데이터를 처리할 때는 비효율적일 수 있다.
- 자체 파일 관리(저장) 시스템이 없기 때문에 HDFS나 S3 같은 외부 스토리지를 함께 사용해야 한다.
- 메모리 기반 처리 방식이라 메모리 비용이 높고, 대규모 처리 시 Out of Memory 문제가 발생할 수 있어 리소스 계획과 튜닝이 필요하다.

## 6. Spark의 실행 구조
Spark는 **Driver**, **Cluster Manager**, **Worker Node**(각각 Executor 보유)로 구성된 분산 데이터 처리 시스템이다.

- **Driver**: 스파크 애플리케이션에서 중앙 제어 역할을 담당한다. 사용자가 작성한 코드를 분석해 실행 계획을 만들고, 전체 작업을 Job → Stage → Task 단위로 분할한다. 분할된 Task를 Executor에 전달하고, 처리 결과를 받아 취합한다. 하나의 스파크 애플리케이션에는 기본적으로 1개의 드라이버만 존재할 수 있다.
  - Driver 프로세스가 어디에 있는지에 따라 **클러스터 모드**(드라이버가 클러스터 내부에 위치)와 **클라이언트 모드**(드라이버가 클러스터 외부에 위치)로 나뉜다. `spark-submit` 시 배포 옵션(Deploy Mode)으로 설정할 수 있으며, 로컬에서 프로세스를 실행하면 보통 클라이언트 방식으로 처리된다.
- **Cluster Manager**: 클러스터 전체의 리소스를 관리한다. Driver가 Executor를 요청하면 이를 받아들여 실행 중인 프로세스를 중지·재시작시키고, Executor가 사용할 CPU 코어 등의 자원을 배분한다. 종류로는 Standalone, Hadoop YARN, Kubernetes가 있다. 별도 클러스터 없이 단일 머신에서 Spark 프로세스 하나만 띄워 실습·확인하는 형태를 **로컬 모드**라고 한다.
- **Executor**: 워커 노드에서 동작하는 JVM 기반 프로세스로, Driver로부터 전달받은 Task를 실제로 실행한다. 자신에게 할당된 파티션 데이터에 대한 변환을 처리하고 결과를 Driver에 반환하며, 필요한 데이터를 메모리에 캐싱하거나 셔플 중간 데이터를 읽고 쓰는 역할도 한다. PySpark를 쓰더라도 Spark 엔진과 Executor 자체는 JVM 위에서 동작하며, Python 함수(UDF 등)가 Executor에서 실행되어야 할 때는 별도의 Python worker 프로세스가 추가로 뜬다(권장되는 방식은 아님).
- **Spark Session**: Spark Core 기능과 상호작용할 수 있는 진입점(entry point)이다. 과거 버전에서는 SparkContext, SQLContext 등으로 진입점이 나뉘어 있었지만, 지금은 SparkSession 하나로 통합되어 있다. 필요하면 SparkSession 객체에서 SparkContext만 따로 꺼내 쓸 수도 있다.

## 7. Job / Stage / Task
- **Job**: 사용자가 실행한 Action(`collect()`, `count()` 등)에 의해 생성되는 가장 큰 작업 단위다. Action이 호출되는 순간 하나의 Job이 명확하게 생성된다.
- **Stage**: Job을 셔플(데이터 이동) 기준으로 나눈 실행 단위다. 서로 의존 관계를 가지는 다수의 Task 모음으로 구성된다. 파티션 간 데이터 재배치(셔플)가 필요 없는 연산들은 하나의 Stage 안에서 계속 처리되고, 셔플이 발생하는 지점을 기준으로 새로운 Stage가 만들어진다.
- **Task**: Stage를 구성하는 실제 실행 단위다. 데이터의 개별 파티션 단위로 작업하며, 하나의 파티션이 하나의 Task로 매핑되어 Executor에서 병렬 실행된다. 예를 들어 데이터가 10개의 파티션으로 나뉘어 있다면 Task도 10개 정도 생성된다.

## 8. Transformation과 Action
스파크에서 데이터를 다룰 때 수행하는 연산은 크게 **Transformation(변환)**과 **Action(액션)** 두 가지로 나뉜다.

- **Transformation**: 원본 데이터를 수정하지 않고(immutable) 하나의 RDD·DataFrame을 새로운 RDD·DataFrame으로 변환하는 연산이다. `map()`, `filter()`, `flatMap()`, `select()`, `groupBy()`, `orderBy()` 등이 해당된다. 호출한다고 바로 실행되는 것이 아니라, 실행 계획(Lineage, 계보)으로 기록만 되었다가 Action이 호출될 때 실제로 처리된다.
  - **Narrow Transformation**: 입력 파티션 1개 → 출력 파티션 1개로 이어지는 변환으로, 파티션 간 데이터 교환이 발생하지 않는다. 예: `map()`, `filter()`, `coalesce()`, 파티셔닝이 맞아떨어지는 `join()`
  - **Wide Transformation**: 연산 시 파티션 간 데이터 교환이 발생하는 변환이다. 예: `groupBy()`, `orderBy()`, `sortByKey()`, `reduceByKey()`. `join()`은 두 RDD·DataFrame이 어떻게 파티셔닝되어 있는지에 따라 narrow일 수도, wide일 수도 있지만 보통은 wide에 해당한다.
  - 파티션 간 데이터 교환을 스파크에서는 **shuffle**이라고 부른다. 셔플은 네트워크 통신, 디스크 I/O, 정렬 등의 비용이 크기 때문에 속도가 느려지고 리소스를 많이 사용한다. 그래서 스파크는 셔플이 발생하는 지점을 기준으로 Stage를 나눈다.
- **Action**: immutable한 입력에 대해 side effect(부수 효과)를 포함하며, 결과물이 RDD·DataFrame이 아닌 연산이다. `count()` → int, `collect()` → array, `save()` → void처럼 실제 값이나 파일을 만들어낸다. Action은 스파크에게 "지금 실행해라"라고 지시하는 연산이며, 이 시점에서 실제 분산 처리가 이루어진다.

## 9. Lazy Evaluation (지연 평가)
- 모든 Transformation은 즉시 계산되지 않고, Lineage(계보)라는 형태로 실행 계획만 기록된다.
- 실제 계산은 Action이 실행되는 시점에 그동안 기록된 모든 Transformation이 한 번에 수행되면서 이루어진다.
- 지연 평가를 사용하는 이유
  - 스파크가 전체 연산 흐름을 분석해 어디를 최적화할지 파악할 시간을 벌 수 있다(즉시 실행하는 Eager Evaluation이라면 선언되는 순간 바로 수행되어 최적화 여지가 없다).
  - 장애가 발생했을 때도 Lineage를 기반으로 특정 시점의 RDD 형태를 다시 계산해 복구할 수 있다.
  - 불필요한 연산을 방지해 리소스를 절약할 수 있다.

## 💡 한 줄 요약
> Spark는 Hadoop MapReduce의 디스크 기반 처리 한계를 인메모리 연산과 DAG 기반 지연 평가로 극복한, Driver–Cluster Manager–Executor 구조의 통합 분산 처리 엔진이다.

## ❓ 더 찾아볼 것
- Tungsten 엔진과 Catalyst Optimizer의 구체적인 동작 원리
- Standalone / YARN / Kubernetes 클러스터 매니저 간 차이
- Client 모드와 Cluster 모드를 실무에서 선택하는 기준
