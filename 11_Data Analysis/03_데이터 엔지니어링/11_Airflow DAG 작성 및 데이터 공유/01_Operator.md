# [데이터 엔지니어링] Operator

---

## 1. Operator란?

Operator는 **Task가 실제로 무슨 작업을 할지 정의하는 객체**다. DAG를 구성하는 기본 단위가 Task라면, 그 Task 안에서 "무엇을 어떻게 실행할지"를 결정하는 역할을 Operator가 담당한다.

- Task는 특정 Operator를 기반으로 정의됨
- Airflow는 Python 기반이라 Operator도 Python으로 작성/확장 가능 (기본 제공 Operator 외에 커스텀 Operator도 제작 가능)
- 어떤 Task는 Python 스크립트를 실행하고, 어떤 Task는 Bash 명령을, 어떤 Task는 DB 쿼리를 날려야 하는 등 역할이 저마다 다른데, 이런 다양한 실행 방식을 제공하는 것이 Operator의 역할
- 여러 Operator를 조합하면 다양한 워크플로우를 구성할 수 있음

가장 대표적으로 많이 쓰이는 **Action Operator** 세 가지를 먼저 살펴본다.

| Operator | 역할 |
|---|---|
| BashOperator | Bash 명령어 실행 |
| PythonOperator | Python 함수 실행 |
| EmailOperator | 이메일 전송 |

Action Operator는 특정 동작을 바로 실행하게 만들어주는 Operator다. 예를 들어 파이프라인 실행이 끝났을 때 EmailOperator로 결과를 이메일로 보내는 알림 Task처럼 활용할 수 있다.

---

## 2. BashOperator - 명령어 직접 실행

`bash_command` 파라미터에 실행할 명령어를 직접 문자열로 넘겨서 사용한다.

```python
from airflow.models.dag import DAG
import datetime
import pendulum
from airflow.operators.bash import BashOperator

with DAG(
    dag_id="bash_operator_example",
    schedule="0 0 * * *",
    start_date=pendulum.datetime(2021, 1, 1, tz="Asia/Seoul"),
    catchup=False,
    dagrun_timeout=datetime.timedelta(minutes=60),
) as dag:
    bash_t1 = BashOperator(
        task_id="bahs_t1",
        bash_command='echo "Hello, Airflow!"',
    )

    bash_t2 = BashOperator(
        task_id="bahs_t2",
        bash_command="ls -al",
    )

    bash_t1 >> bash_t2
```

- `bash_t1`은 문자열을 출력하고, `bash_t2`는 현재 디렉토리의 파일 목록을 확인한다.
- 로그(Logs) 탭에서 `Running command: ...`, `Command exited with return code 0` 형태로 실제 실행된 커맨드와 결과를 확인할 수 있다.
- `dag_id`, `schedule`, `start_date`, `catchup`, `dagrun_timeout` 등은 DAG 정의 시 함께 설정하는 값이며, 이 중 스케줄링/캐치업 관련 개념은 이후 "Cron 스케줄링" 파트에서 자세히 다룬다.

---

## 3. BashOperator - 외부 쉘 스크립트 실행

실무에서는 `echo` 같은 단순 명령어보다, **쉘 스크립트 파일 자체를 실행**하는 방식이 훨씬 일반적으로 쓰인다. 하나의 스크립트 안에 여러 작업을 정의해두고 한 번에 수행시키는 형태다.

### 3-1. 쉘 스크립트 작성 및 권한 설정

```bash
# plugins/shell/select_fruit.sh
FRUIT=$1
if [ $FRUIT == APPLE ]; then
    echo "You selected Apple!"
elif [ $FRUIT == ORANGE ]; then
    echo "You selected Orange!"
elif [ $FRUIT == GRAPE ]; then
    echo "You selected Grape!"
else
    echo "You selected other Fruit!"
fi
```

스크립트를 새로 만들면 기본적으로 **실행 권한이 없기 때문에** 권한을 부여해야 한다.

```bash
chmod +x select_fruit.sh
```

> Windows에서 스크립트를 작성해 Linux 환경(컨테이너)으로 옮기면 줄바꿈 문자 차이로 오류가 날 수 있다. 이럴 때는 `dos2unix`를 설치해서 파일을 리눅스 형식으로 변환해주면 안전하다.
> ```bash
> sudo apt install dos2unix
> dos2unix select_fruit.sh
> ```

### 3-2. docker-compose 볼륨 경로 확인

BashOperator가 컨테이너 외부(호스트)의 쉘 스크립트를 인식하려면, `docker-compose.yaml`의 `volumes` 설정에 `plugins` 경로가 실제 파일 경로로 매핑되어 있어야 한다.

```yaml
volumes:
  - ${AIRFLOW_PROJ_DIR:-.}/dags:/opt/airflow/dags
  - ${AIRFLOW_PROJ_DIR:-.}/logs:/opt/airflow/logs
  - ${AIRFLOW_PROJ_DIR:-.}/config:/opt/airflow/config
  - ${AIRFLOW_PROJ_DIR:-.}/plugins:/opt/airflow/plugins
```

이 경로에 스크립트를 올려두면 Airflow(스케줄러)가 이를 인식해서 처리할 수 있다.

### 3-3. DAG에서 스크립트 실행

```python
from airflow import DAG
import pendulum
import datetime
from airflow.operators.bash import BashOperator

with DAG(
    dag_id="dags_bash_select_fruit",
    schedule="10 0 * * 6",  # 매주 토요일 0시 10분마다
    start_date=pendulum.datetime(2023, 3, 1, tz="Asia/Seoul"),
    catchup=False,
) as dag:

    t1_orange = BashOperator(
        task_id="t1_orange",
        bash_command="/opt/airflow/plugins/shell/select_fruit.sh ORANGE",
    )

    t2_avocado = BashOperator(
        task_id="t2_avocado",
        bash_command="/opt/airflow/plugins/shell/select_fruit.sh AVOCADO",
    )

    t1_orange >> t2_avocado
```

- `t1_orange`는 스크립트에 `ORANGE`를 인자로 넘겨 `You selected Orange!` 결과를 출력
- `t2_avocado`는 스크립트에 정의되지 않은 과일(`AVOCADO`)이므로 else 분기인 `You selected other Fruit!` 출력

즉 스크립트 안의 조건문에 따라 넘긴 인자 값에 맞는 결과가 로그에 그대로 반영된다.

---

## 4. PythonOperator

`python_callable`에 실행할 Python 함수를 넘겨주는 방식으로 동작한다. 핵심 파라미터는 `task_id`와 `python_callable` 두 가지다.

```python
from airflow.models.dag import DAG
import datetime
import pendulum
from airflow.operators.python import PythonOperator

def print_hello():
    return "Hello, Airflow!"

with DAG(
    dag_id="dags_python_operator",
    schedule="0 8 1 * *",
    start_date=pendulum.datetime(2021, 1, 1, tz="Asia/Seoul"),
    catchup=False,
    dagrun_timeout=datetime.timedelta(minutes=60),
) as dag:
    python_task = PythonOperator(
        task_id="print_hello_task",
        python_callable=print_hello,
    )
```

- 함수의 `return` 값은 로그에 `Done. Returned value was: Hello, Airflow!` 형태로 확인할 수 있다.
- DAG 안에서 실행하고 싶은 로직을 Python 코드로 그대로 작성해두고 실행시킬 수 있다는 점이 핵심이다.

---

## 5. EmptyOperator

**아무 작업도 수행하지 않는 Task**를 만드는 Operator다. 얼핏 쓸모없어 보이지만, DAG의 **논리적인 흐름을 구성**할 때 유용하게 쓰인다.

- 여러 Task를 동시에 시작하고 싶을 때
- 특정 Task들이 모두 끝난 뒤에 다음 Task를 실행하고 싶을 때

즉 병렬로 실행되는 여러 Task를 하나로 모으거나 갈라지게 하는 "연결점" 역할을 한다.

```python
from airflow import DAG
from airflow.operators.empty import EmptyOperator  # Airflow 2.x 이상에서 사용
from airflow.operators.bash import BashOperator
import pendulum

with DAG(
    dag_id="empty_operator_example",
    start_date=pendulum.datetime(2024, 3, 1, tz="Asia/Seoul"),
    catchup=False,
) as dag:
    start = EmptyOperator(task_id="start")

    task_1 = BashOperator(
        task_id="task_1",
        bash_command="echo 'Task 1 실행!'",
    )
    task_2 = BashOperator(
        task_id="task_2",
        bash_command="echo 'Task 2 실행!'",
    )

    end = EmptyOperator(task_id="end")

    # DAG 흐름 설정
    start >> [task_1, task_2] >> end
```

- 이 DAG는 Task가 총 4개(start, task_1, task_2, end)로 구성되어 있다.
- `start` 이후 `task_1`, `task_2`가 병렬로 실행되고, 두 Task가 **모두** 끝나야 `end`로 넘어간다.
- 리스트(`[task_1, task_2]`)를 활용해 병렬 실행 흐름을 선언할 수 있다.

---

## 💡 한 줄 요약
> Operator는 Task가 무엇을 실행할지 정의하는 객체이며, BashOperator(명령어/쉘 스크립트 실행), PythonOperator(파이썬 함수 실행), EmptyOperator(흐름 제어용 빈 Task)를 조합해 다양한 DAG 워크플로우를 구성할 수 있다.

## ❓ 더 찾아볼 것
- EmailOperator 실제 사용 예시 (SMTP 설정 포함)
- TaskFlow API(`@task` 데코레이터)와 전통적 Operator 방식의 차이
- 커스텀 Operator를 직접 만드는 방법
- TransferOperator 등 기타 Airflow 기본 제공 Operator 종류
