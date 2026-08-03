# [데이터 엔지니어링] Xcom과 Variable

---

## 1. 왜 XCom이 필요한가?

Airflow에서 Task는 기본적으로 **독립적으로 실행**된다. `task1 → task2 → task3`처럼 하나의 DAG 안에 여러 Task가 있어도, 한 Task가 처리한 결과를 다른 Task가 자동으로 참조할 수는 없다. 이 문제(Task 간 데이터 전달)를 해결하기 위해 등장한 것이 **XCom**이다.

### XCom이란?
- **Cross-Communication**의 약자. Airflow에서 Task 간 데이터를 주고받기 위한 기능
- 각 Task가 독립적으로 실행되기 때문에, Task 간 데이터 공유를 위해 XCom을 활용
- **DAG Run 내에서만 존재**하며, 다른 DAG Run과는 공유되지 않음 (같은 DAG라도 실행 회차가 다르면 공유 X)
- 메타데이터베이스에 `dag_id`, `task_id`, `run_id` 등을 기준으로 저장되어 실행별로 구분됨
- **대용량 데이터는 지원하지 않음** — DataFrame 같은 대용량 데이터를 XCom으로 전달하는 것은 기술적으로 불가능하진 않지만 매우 비효율적. 문자열, 숫자 등 작은 크기의 데이터 공유에 적합
- PythonOperator를 사용할 경우, 함수의 return 값이 **자동으로 XCom에 등록**됨 (PythonOperator뿐 아니라 다른 Operator 다수도 실행 결과를 return value 형태로 등록하는 경우가 많음)

> XCom은 어디까지나 "같은 DAG Run 안에서 Task 사이의 작은 데이터를 전달"하는 용도이지, 외부 시스템과 데이터를 공유하는 용도는 아니다.

---

## 2. XCom 저장(push)과 조회(pull)

### 2-1. 데이터 저장 - `xcom_push`

Task 실행 중 데이터를 저장할 때 사용한다. Task Instance(`ti`)의 `xcom_push(key, value)` 메서드로 원하는 Key-Value를 저장한다.

```python
def push_xcom_value(**kwargs):
    kwargs['ti'].xcom_push(key='message', value='Hello from push_task')
```

- Key-Value 형식으로 저장됨
- 해당 DAG의 실행(Run) 내에서만 사용 가능
- 함수 인자의 `**kwargs`로 넘어오는 `ti`가 바로 Task Instance 객체다. 이 외에도 `ds`(date string) 같은 컨텍스트 변수들이 존재한다.

### 2-2. 데이터 조회 - `xcom_pull`

Task 실행 시 이전 Task가 저장한 데이터를 가져올 때 사용한다. `xcom_pull(task_ids, key)`로 특정 Task가 저장한 특정 키의 값을 가져온다.

```python
def pull_xcom_value(**kwargs):
    message = kwargs['ti'].xcom_pull(task_ids='push_task', key='message')
    print("XCom에서 받은 값:", message)
```

---

## 3. XCom 사용 방법 4가지

| 방식 | 설명 |
|---|---|
| PythonOperator Return 값 | `return`만 하면 XCom에 자동 등록 |
| Push-Pull 직접 호출 | `xcom_push`/`xcom_pull`을 명시적으로 호출. task_id, key를 세밀하게 지정 가능하지만 코드가 다소 복잡해짐 |
| Jinja Template | 템플릿 엔진 구문으로 Task 파라미터 값을 동적으로 할당 (직접 값을 꺼내 계산할 필요 없이 자동으로 채워짐) |
| `@task` 데코레이터 | 데코레이터 기반 Task에서 함수가 return하면 자동으로 XCom에 기록됨 |

### 3-1. PythonOperator with XCom (push → pull)

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
import pendulum

# XCom에 데이터 저장 (Push)
def push_xcom_value(**kwargs):
    kwargs['ti'].xcom_push(key='message', value='Hello from push_task')

# XCom에서 데이터 가져오기 (Pull)
def pull_xcom_value(**kwargs):
    message = kwargs['ti'].xcom_pull(task_ids='push_task', key='message')
    print("XCom에서 받은 값:", message)

with DAG(
    dag_id="xcom_example",
    start_date=pendulum.datetime(2024, 3, 1, tz="Asia/Seoul"),
    catchup=False,
) as dag:
    push_task = PythonOperator(
        task_id="push_task",
        python_callable=push_xcom_value,
    )
    pull_task = PythonOperator(
        task_id="pull_task",
        python_callable=pull_xcom_value,
    )

    push_task >> pull_task
```

### 3-2. BashOperator with XCom

BashOperator는 `do_xcom_push=True`를 지정하면 명령어의 출력값을 XCom에 저장할 수 있고, Jinja 템플릿 문법(`{{ ti.xcom_pull(...) }}`)으로 이를 꺼내 쓸 수 있다.

```python
from airflow import DAG
from airflow.operators.bash import BashOperator
import pendulum

with DAG(
    dag_id="bash_xcom_example",
    start_date=pendulum.datetime(2024, 3, 1, tz="Asia/Seoul"),
    catchup=False,
) as dag:
    # XCom에 값 저장 (Push)
    push_task = BashOperator(
        task_id="push_task",
        bash_command="echo 'Hello from BashOperator!'",
        do_xcom_push=True,  # XCom에 출력값 저장
    )

    # XCom에서 값 가져오기 (Pull)
    pull_task = BashOperator(
        task_id="pull_task",
        bash_command="echo '{{ ti.xcom_pull(task_ids=\"push_task\") }}'",
    )

    push_task >> pull_task
```

### 3-3. PythonOperator ↔ BashOperator 혼합 XCom

Operator 종류가 달라도 XCom을 통해 데이터를 주고받을 수 있다.

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.bash import BashOperator
import pendulum

# XCom에 데이터 저장 (PythonOperator)
def push_xcom_value(**kwargs):
    kwargs['ti'].xcom_push(key='message', value='Hello from PythonOperator!')

with DAG(
    dag_id="python_bash_xcom_example",
    start_date=pendulum.datetime(2024, 3, 1, tz="Asia/Seoul"),
    catchup=False,
) as dag:
    # PythonOperator를 사용하여 XCom에 값 저장
    push_task = PythonOperator(
        task_id="push_task",
        python_callable=push_xcom_value,
    )

    # BashOperator에서 XCom 값을 가져와 출력
    pull_task = BashOperator(
        task_id="pull_task",
        bash_command="echo '{{ ti.xcom_pull(task_ids=\"push_task\", key=\"message\") }}'",
    )

    push_task >> pull_task
```

---

## 4. Airflow Decorators (`@task`)

Python 함수에 `@task` 데코레이터만 붙이면 해당 함수를 Airflow Task로 변환할 수 있다.

**주요 특징**
- `@task`: 함수 데코레이터. Python 함수를 Airflow Task로 자동 변환
- DAG 정의: `@dag` 데코레이터로 함수 전체가 속할 DAG를 정의
- Task 간 의존성: 함수 호출값(리턴값을 다음 함수의 인자로 넘기는 것)만으로 자연스럽게 설정됨 — XCom을 수동으로 다룰 필요가 없어짐

### 코드 비교 — 전통적인 방식 vs `@task` 데코레이터

**전통적인 방식** (PythonOperator + 명시적 XCom pull)

```python
def extract_data():
    print("Extracting data...")
    return "raw_data"

def transform_data(data):
    print(f"Transforming data: {data}")
    return f"transformed_{data}"

def load_data(data):
    print(f"Loading data: {data}")
    return f"loaded_{data}"

with DAG(dag_id="example_dag", schedule_interval="@daily", start_date=datetime(2025, 11, 1)) as dag:
    extract_task = PythonOperator(task_id="extract_data", python_callable=extract_data)
    transform_task = PythonOperator(
        task_id="transform_data", python_callable=transform_data,
        op_args=["{{ task_instance.xcom_pull(task_ids='extract_data') }}"]
    )
    load_task = PythonOperator(
        task_id="load_data", python_callable=load_data,
        op_args=["{{ task_instance.xcom_pull(task_ids='transform_data') }}"]
    )

    extract_task >> transform_task >> load_task
```

**`@task` 데코레이터 방식** (XCom을 신경 쓰지 않고 함수 리턴값을 그대로 다음 함수 인자로 전달)

```python
@dag(schedule_interval="@daily", start_date=datetime(2025, 11, 1))
def example_dag():

    @task
    def extract_data():
        print("Extracting data...")
        return "raw_data"

    @task
    def transform_data(data):
        print(f"Transforming data: {data}")
        return f"transformed_{data}"

    @task
    def load_data(data):
        print(f"Loading data: {data}")
        return f"loaded_{data}"

    # 태스크 간 의존성 설정
    data = extract_data()              # 첫 번째 태스크
    transformed_data = transform_data(data)  # 두 번째 태스크
    load_data(transformed_data)        # 세 번째 태스크

# DAG 실행
example_dag()
```

두 코드는 동일한 ETL 흐름을 구현하지만, 데코레이터 방식은 `op_args`에 XCom pull 구문을 직접 쓰지 않고 **함수 반환값을 그대로 다음 함수 인자로 넘기는 것만으로 XCom 전달과 의존성 설정이 자동 처리**된다. 그래서 함수형 스타일로 DAG를 작성할 때 코드가 짧아지고 직관적이다.

---

## 5. 전역 공유 변수(Variable)

XCom이 "한 DAG Run 안에서의 Task 간 데이터 전달"이라면, **Variable**은 "DAG 실행 단위를 넘어 Airflow 전체에서 공통으로 쓰는 값 저장"이다.

### Variable이란?
- Airflow에서 여러 DAG 및 Task 간에 데이터를 공유하기 위한 변수
- **모든 DAG가 공유** 가능 (XCom과 가장 큰 차이점)
- 협업 환경에서 표준화된 DAG를 만들기 위해 상수처럼 지정해서 사용 (실행 환경 구분값, 테이블명, 공통 디렉토리 경로 등)
- Variable에 등록한 key, value는 메타데이터베이스에 저장
- Airflow UI, CLI, API를 통해 관리 가능

### 5-1. Variable 등록하기 (UI)

1. Webserver 접속 → `Admin` → `Variables` 클릭
2. `[+]` 클릭해서 새 Variable 생성
3. Key / Value 값을 입력하고 `Save` (Description은 자유롭게 입력)

### 5-2. Variable 사용하기 (코드)

`Variable` 모듈의 `get()` 함수로 값을 조회한다. 두 번째 인자로 `default_var`를 지정해두면 해당 Variable이 없을 때 기본값을 사용한다.

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.models import Variable  # Airflow Variable 모듈 불러오기
import pendulum

# Variable에서 값을 가져오는 함수
def print_variable():
    my_var = Variable.get("my_variable", default_var="default")
    print("Airflow Variable 값:", my_var)

with DAG(
    dag_id="airflow_variable_example",
    start_date=pendulum.datetime(2024, 3, 1, tz="Asia/Seoul"),
    catchup=False,
) as dag:
    print_var_task = PythonOperator(
        task_id="print_variable_task",
        python_callable=print_variable,
    )
```

- Variable에 `my_variable = Hello, Airflow!`가 등록되어 있으면 로그에 `Airflow Variable 값: Hello, Airflow!`가 출력된다.
- 만약 Variable을 삭제하고 다시 실행하면, DAG는 실패하지 않고 `default_var`로 지정한 `"default"` 값이 대신 출력된다.

### 5-3. Variable vs XCom

| 비교 항목 | XCom | Variable |
|---|---|---|
| 데이터 유지 기간 | DAG 실행(Run) 단위로 유지됨 | Airflow 전체에서 지속적으로 유지됨 |
| 데이터 저장 위치 | Airflow 메타데이터 DB | Airflow 메타데이터 DB |
| 사용 목적 | Task 간 데이터 전달 | DAG 실행 간 설정값 저장 |
| 데이터 호출 방식 | `xcom_push()`, `xcom_pull()` | `Variable.get()`, `Variable.set()` |
| 저장 가능한 데이터 유형 | JSON 직렬화 가능한 작은 데이터 (문자열, 숫자, 리스트, 딕셔너리) | 문자열 및 JSON 직렬화 가능한 데이터 |
| 범위 | DAG 실행 내에서만 사용 가능 | 모든 DAG에서 전역적으로 사용 가능 |
| 자동 저장 여부 | PythonOperator의 return 값이 자동 저장됨 | 자동 저장되지 않음, 명시적으로 설정 필요 |
| 보안 고려 사항 | Task 간 민감한 데이터 전달 시 사용하지 않음 | API 키, 비밀번호 등은 Connection을 활용하는 것이 더 안전함 |

> API 키나 비밀번호 같은 민감한 설정값은 XCom·Variable이 아니라 Airflow의 **Connection** 기능으로 관리하는 것이 더 안전하다.

---

## 💡 한 줄 요약
> XCom은 한 DAG Run 안에서 Task 간에 작은 데이터를 주고받는 용도(push/pull, 자동 return 저장, `@task` 데코레이터로 더 간결하게 사용 가능)이고, Variable은 모든 DAG가 공유하는 전역 설정값 저장용이다.

## ❓ 더 찾아볼 것
- Jinja Template 문법(`{{ }}`) 전체 정리 및 Airflow에서 자주 쓰는 매크로 변수(`ds`, `ts` 등)
- Airflow Connection으로 민감한 인증 정보(API 키, DB 비밀번호) 관리하는 방법
- XCom 백엔드를 커스텀하여 대용량 데이터를 다루는 방법(Custom XCom Backend)
- TaskFlow API(`@task`, `@dag`)의 추가 기능 (multiple_outputs 등)
