# [데이터 엔지니어링] DAG 실행

---

## 1. Airflow Web UI 개요

Airflow Web UI는 **DAG 및 Task의 실행 상태를 시각적으로 모니터링하고, DAG 실행을 제어**할 수 있는 웹 기반 인터페이스다.

- DAG 및 Task의 실행 상태 확인 (색상으로 구분: 초록색 = 성공, 빨간색 = 실패, 회색 = 대기 중 등)
- DAG의 수동 실행 및 중지 (Trigger 버튼)
- Task의 재시도 및 강제 실행
- 실행 로그 확인 및 실패 원인 분석
- DAG 및 Task의 의존성 시각화

학습·실습 단계에서는 스케줄이 돌아갈 때까지 기다리기 어려운 경우가 많기 때문에, 실제로는 **수동 트리거(Trigger DAG 버튼)**를 눌러 즉시 실행시키며 확인하는 경우가 많다.

## 2. Web UI 주요 메뉴

### DAGs View

- 등록된 모든 DAG을 조회할 수 있는 기본 화면이다.
- DAG 실행 상태(활성, 비활성, 일시정지)를 확인할 수 있고, 특정 DAG을 클릭하면 세부 정보를 확인할 수 있다.
- **"Trigger DAG" 버튼**을 눌러 DAG을 수동으로 즉시 실행할 수 있다.

### Graph View (그래프 뷰)

- DAG 내부 Task 간의 관계를 시각적 그래프로 표현한다.
- Task들의 의존성(Upstream/Downstream) 구조를 한눈에 파악할 수 있다.
- 하나의 DAG Run 단위로 볼 때는 각 Task의 실행 상태가 색상으로 구분되어(성공=초록, 실패=빨강, 대기=회색 등) 복잡한 DAG도 한눈에 파악할 수 있다. 반대로 특정 Run이 아니라 DAG 구조 자체만 클릭해서 보면 색상 없이 구조만 표시된다.

### Tree View (트리 뷰)

- DAG의 실행 이력을 날짜별 트리 구조로 제공한다.
- 하나의 DAG Run에 포함된 여러 Task Instance들이 각각 성공/실패 등 어떤 상태였는지 기록을 확인할 수 있다.
- 특정 실행 날짜의 DAG 상태를 쉽게 추적할 수 있다.

### Task Instance Details (태스크 실행 상세 정보)

- 특정 Task Instance를 클릭하면 실행 정보(Median Total Duration 등 소요 시간 통계 포함)를 확인할 수 있다.
- 평소 자주 들여다볼 필요는 없지만, 시간 지연이나 병목이 의심될 때 상세히 확인하는 용도로 유용하다.
- 실패한 Task를 다시 실행하거나 강제로 종료할 수 있다.

### Gantt Chart (간트 차트)

- DAG 실행 시 각 Task의 실행 시간을 시각화한다.
- Task 실행 시간 비교 및 병렬 실행 분석이 가능해, 실행 시간이 오래 걸리는 Task를 찾아 최적화할 지점을 판단하는 데 활용할 수 있다.

### Code View (코드 뷰)

- DAG의 소스 코드를 UI에서 직접 확인할 수 있다. (단, 이 화면에서 직접 수정은 불가능)
- 어디서 병목이나 문제가 발생했는지 코드 레벨에서 확인하고 싶을 때 사용한다.

### Log View (로그 뷰)

- 실행된 Task의 로그를 확인하여 오류 분석 및 디버깅에 활용한다.
- 실패한 Task의 원인을 파악하고 재시도 여부를 결정하는 데 사용한다. (예: EmptyOperator처럼 실제 동작이 없는 Task는 로그가 거의 남지 않지만, 실제 명령을 수행하는 Task는 그 실행 결과가 로그에 출력됨)

## 3. Task LifeCycle (태스크 생명 주기)

Task는 DAG에 정의는 되어 있지만 아직 실행되지 않은 **none** 상태에서 시작해 다음과 같은 흐름을 거친다.

```
none → (Scheduler) → scheduled → (Executor) → queued → (Worker) → running → success
```

- **none**: DAG에 정의되어 있지만 아직 실행되지 않은 상태
- **scheduled**: Scheduler를 거쳐 실행 대상으로 판단된 상태
- **queued**: Executor로 넘어가 실행 시스템(Queue)에 전달된 상태
- **running**: Worker가 배정되어 실제로 실행 중인 상태. 이때부터 로그가 기록되고 UI에서 실행 중 상태를 확인할 수 있음
- **success**: 정상적으로 종료된 상태

실행 중 오류가 발생하면 재시도 횟수가 남아있는지에 따라 분기된다.

- 재시도 횟수가 남아있으면 **up_for_retry** → **restarting** → 다시 스케줄링되어 재실행
- 재시도 횟수를 모두 소진하면 **failed**

### 주요 Task 상태값 정리

| 상태 | 설명 |
|---|---|
| deferred | 특정 이벤트 대기 중 (예: Sensor 대기) |
| failed | 실행이 실패한 상태 (exit 1 또는 예외 발생) |
| queued | 실행을 기다리는 대기열(Queue)에 있는 상태 |
| removed | DAG에서 삭제된 상태 (일반적으로 UI에서 볼 수 없음) |
| restarting | 재시작 중인 상태 |
| running | 현재 실행 중인 상태 |
| scheduled | 스케줄링되었지만 아직 실행되지 않은 상태 |
| shutdown | 실행이 중지된 상태 (시스템 종료 또는 강제 중지) |
| skipped | 실행되지 않고 건너뛰어진 상태 (예: 조건 분기에 의해) |
| success | 정상적으로 완료된 상태 |
| up_for_reschedule | Sensor가 일정 시간마다 재실행 대기 중인 상태 |
| up_for_retry | 실패했지만 재시도(Retry) 대기 중인 상태 |
| upstream_failed | 이전(Upstream) Task가 실패하여 실행되지 않은 상태 |
| no_status | 실행 정보가 없는 상태 (아직 실행된 적 없음) |

## 4. DAG 실행 방법

| 방식 | 설명 |
|---|---|
| **자동 실행** | DAG에 설정된 스케줄(Interval, Cron 등)에 따라 실행 |
| **수동 실행** | Web UI 또는 CLI(Command Line Interface)를 통해 즉시 실행 |
| **이벤트 기반 실행** | 특정 트리거(예: API 요청, 파일 업로드 등)로 실행. Sensor 등을 활용 |

실제 운영에서는 자동 실행을 사용하지만, 학습·실습 단계에서는 스케줄을 기다리기 어렵기 때문에 Web UI에서 직접 트리거해서 확인하는 경우가 많다.

## 5. Airflow 스케줄링

- **Schedule**: DAG를 실행하는 주기를 설정하는 방식
- **Schedule interval**: DAG이 실행되는 시간 간격을 결정하는 속성. 최신 버전에서는 `schedule` 인자로 통일해서 사용하는 추세이며 둘의 동작 차이는 없다.

```python
dag = DAG(
    dag_id="example_schedule",
    start_date=datetime(2025, 3, 1),
    schedule_interval="0 6 * * *",  # 매일 오전 6시 실행
    catchup=False
)
```

| 설정 방식 | 설명 | 예제 |
|---|---|---|
| None | DAG이 자동 실행되지 않음 (수동 실행만 가능) | `schedule_interval=None` |
| 예약어 사용 | 실행 주기를 간단히 지정 | `@daily`(매일), `@hourly`(매시간) |
| Cron 표현식 사용 | 세부적인 실행 주기 설정 가능 | `"0 9 * * *"` (매일 오전 9시 실행) |
| Timedelta 사용 | 특정 시간 간격마다 실행 | `schedule_interval=timedelta(hours=6)` |

## 6. DAG 예제 코드로 보는 기본 구조

`dags` 폴더 하위에 파이썬 파일(`dags_bash_operator.py`)을 만들어 아래와 같이 DAG을 정의할 수 있다.

```python
from airflow.models.dag import DAG
import datetime
import pendulum
from airflow.operators.bash import BashOperator

with DAG(
    dag_id="dags_bash_operator",
    schedule="0 0 * * *",
    start_date=pendulum.datetime(2021, 1, 1, tz="Asia/Seoul"),
    catchup=False,
    dagrun_timeout=datetime.timedelta(minutes=60),
) as dag:
    bash_t1 = BashOperator(
        task_id="bahs_t1",
        bash_command="echo whoami",
    )
    bash_t2 = BashOperator(
        task_id="bahs_t2",
        bash_command="echo $HOSTNAME",
    )

    bash_t1 >> bash_t2
```

- `with DAG(...) as dag:` 구문으로 DAG 객체를 하나 만들고, `dag_id`·`schedule`·`start_date` 등의 인자를 선언한다. (`start_date`와 `schedule`을 선언하지 않으면 수동 실행만 가능한 DAG이 된다.)
- `dagrun_timeout`은 DAG 실행이 지정된 시간(여기서는 60분) 내에 끝나지 않으면 실패 처리하는 설정이다.
- `BashOperator`는 Task를 정의할 때 사용하는 객체(Operator)의 한 종류로, Bash 명령어를 실행한다.
- `bash_t1 >> bash_t2`처럼 `>>` 연산자로 Task 간 실행 순서(의존성)를 표현한다. 즉 `bash_t1`이 먼저 실행되고 성공해야 `bash_t2`가 실행된다.

### DAG 디렉토리 설정 및 볼륨 마운트

- `dags` 폴더를 생성하고 그 하위에 DAG 파일을 저장한다.
- Docker Compose 환경에서는 로컬 `dags` 디렉토리를 컨테이너 내부 경로(`/opt/airflow/dags`)로 **볼륨 마운트**해두어야, 로컬에 파일을 저장하는 즉시 Airflow 컨테이너 안에서도 해당 파일이 인식된다.

```yaml
volumes:
  - ${AIRFLOW_PROJ_DIR:-.}/dags:/opt/airflow/dags
  - ${AIRFLOW_PROJ_DIR:-.}/logs:/opt/airflow/logs
  - ${AIRFLOW_PROJ_DIR:-.}/config:/opt/airflow/config
  - ${AIRFLOW_PROJ_DIR:-.}/plugins:/opt/airflow/plugins
```

- 파일을 올린 뒤에는 Scheduler가 DAG Directory를 주기적으로 스캔하기 때문에, 저장하자마자 바로 UI에 뜨지 않을 수 있고 약간의 시간이 걸릴 수 있다. UI에서 DAG이 정상적으로 업로드되었는지 확인한 뒤 Trigger 버튼으로 실행해보면 된다.

## 7. Operator(오퍼레이터) 기본

**Operator**는 Airflow에서 **Task를 실행하는 역할을 수행하는 객체**다. DAG 내에서 개별 Task로 사용되며, 무엇을 실행할지(Bash 명령어인지, Python 함수인지, SQL 문인지 등)를 정의한다. Python 기반으로 직접 확장(사용자 정의 Operator 제작)도 가능하며, 기본 제공(내장) Operator도 다양하게 존재한다.

### Operator 종류

| 오퍼레이터 종류 | 설명 | 예제 |
|---|---|---|
| Action Operators (기본 실행 오퍼레이터) | 특정 동작을 수행 | PythonOperator, BashOperator, EmailOperator |
| Sensor Operators (센서 오퍼레이터) | 특정 이벤트를 감지할 때까지 대기 (예: 파일 업로드, HTTP API 응답) | FileSensor, HttpSensor, S3KeySensor |
| Transfer Operators (데이터 전송 오퍼레이터) | 한 위치에서 다른 위치로 데이터를 이동 | S3ToGCSOperator, MySQLToGCSOperator |
| Database Operators (데이터베이스 관련 오퍼레이터) | DB에서 SQL을 실행 | PostgresOperator, MySqlOperator, SnowflakeOperator |
| Big Data & ML Operators (빅데이터 & 머신러닝 오퍼레이터) | Spark, Hive, Dataproc, ML 관련 작업 (예: Spark에 Job 제출) | SparkSubmitOperator, DataflowOperator |
| Docker & Kubernetes Operators | 컨테이너 환경에서 작업 실행 (파이프라인을 격리된 환경에서 실행) | DockerOperator, KubernetesPodOperator |
| Empty Operators | 실제 작업 없이 Task 흐름(구조)을 설정하는 데 사용 | EmptyOperator (구 DummyOperator) |

> 참고로 `EmptyOperator`는 과거에는 `DummyOperator`라는 이름으로 불렸다. 아무 작업도 하지 않지만, 여러 Task를 병렬로 묶어 시작점을 만들어주는 등 DAG 구조를 표현할 때 사용된다.

## 8. Task의 주요 속성

| 속성 | 설명 |
|---|---|
| task_id | Task의 고유 식별자 (DAG 내에서 유일해야 함) |
| operator | Task가 수행할 작업을 정의 (BashOperator, PythonOperator 등) |
| depends_on_past | 이전 실행 결과에 따라 Task 실행 여부 결정 |
| retries | Task 실패 시 재시도 횟수 설정 (기본값: 0) |
| execution_timeout | Task 실행 시간 제한 설정 (지정된 시간 내 미완료 시 실패 처리) |
| start_date | DAG 실행 시작 날짜 및 시간 설정 |
| end_date | DAG 실행 종료 날짜 및 시간 설정 |
| schedule_interval | Task 실행 주기 설정 (예: `@daily`, `@hourly`, `0 12 * * *`) |
| priority_weight | Task의 실행 우선순위 설정 (높을수록 우선 실행) |
| task_concurrency | 특정 Task의 병렬 실행 가능 개수 제한 |

## 💡 한 줄 요약
> Airflow Web UI(DAGs/Graph/Tree/Gantt/Code/Log View)를 통해 DAG과 Task의 실행 상태를 시각적으로 확인·제어할 수 있고, Task는 none → scheduled → queued → running → success/failed의 생명 주기를 가지며, Operator로 Task의 실행 내용을 정의하고 `schedule`·`retries` 등의 속성으로 실행 주기와 재시도 방식을 세밀하게 제어할 수 있다.

## ❓ 더 찾아볼 것
- Sensor 기반 이벤트 트리거 DAG 구성 방법
- `catchup` 옵션의 동작 방식과 백필(Backfill) 처리
- 사용자 정의(Custom) Operator 만드는 방법
