# [데이터 엔지니어링] DAG Scheduling 및 Trigger

---

## 1. DAG 내 Task 간 의존성(Dependency)이란?

DAG(Directed Acyclic Graph)라는 이름 그대로, **방향성이 있고 순환이 없는 그래프**다. 즉 어떤 Task가 먼저 실행되고 어떤 Task가 그 뒤를 따를지를 명확하게 선언해줘야 한다. "Task A가 끝나야만 Task B가 실행된다"는 관계를 정의하는 것이 의존성(Dependency) 설정이다.

- Airflow에서는 DAG 내에서 Task 간 실행 순서(의존성)를 정의해야 함
- Task 간 의존성을 설정하면 특정 Task가 완료된 후 다음 Task가 실행됨
- 연산자 `>>` 또는 `<<`를 활용하여 Task 간 의존성을 설정

```python
task_1 >> task_2  # task_1이 완료된 후 task_2 실행
```

`task_1 >> task_2`와 `task_2 << task_1`은 표현 방식만 다를 뿐 해석은 동일하다.

---

## 2. Task 연결 원리

Task 간 실행 관계를 명확하게 지정해야 DAG가 올바르게 동작한다. 연결 방식은 크게 **순차 실행(Sequential Execution)**과 **병렬 실행(Parallel Execution)**으로 나눌 수 있다.

| 종류 | 설명 |
|---|---|
| Upstream Task | 현재 Task 이전에 실행되는 Task |
| Downstream Task | 현재 Task 이후에 실행되는 Task |
| Linear Dependency | 순차적으로 Task를 실행 (Task A → Task B → Task C) |
| Branching | 특정 조건에 따라 Task 실행 흐름을 분기 |
| Parallel Execution | 여러 Task를 병렬로 실행 |

예를 들어 A → B로 연결되어 있다면, B 입장에서 A는 Upstream Task, A 입장에서 B는 Downstream Task다.

---

## 3. 기본 연결(순차 실행)

```python
from airflow import DAG
from airflow.operators.empty import EmptyOperator  # Airflow 2.x 기준
from airflow.operators.bash import BashOperator
import pendulum

with DAG(
    dag_id="basic_dag_example",
    start_date=pendulum.datetime(2024, 3, 1, tz="Asia/Seoul"),
    catchup=False,
) as dag:
    # 시작 Task (EmptyOperator)
    start = EmptyOperator(task_id="start")

    # BashOperator 실행 Task 1
    task_1 = BashOperator(task_id="task_1", bash_command="echo 'Task 1 실행'")

    # BashOperator 실행 Task 2
    task_2 = BashOperator(task_id="task_2", bash_command="echo 'Task 2 실행'")

    # 종료 Task (EmptyOperator)
    end = EmptyOperator(task_id="end")

    # DAG 연결 설정
    start >> task_1 >> task_2 >> end
```

`start → task_1 → task_2 → end` 순서로 하나씩 순차적으로 실행되는 가장 기본적인 흐름이다.

---

## 4. 다중 Task 연결(병렬 실행)

```python
# task_1 >> [task_2, task_3]  # task_1 실행 후 task_2와 task_3 병렬 실행

# DAG 연결 설정(다중 종속 관계 설정)
start >> [task_1, task_2]        # start 이후 task_1, task_2 병렬 실행
[task_1, task_2] >> task_3       # task_1과 task_2가 완료되어야 task_3 실행
task_3 >> [task_4, task_5]       # task_3이 완료된 후 task_4, task_5 병렬 실행
[task_4, task_5] >> end          # task_4, task_5가 완료된 후 end Task 실행
```

리스트(`[ ]`)로 여러 Task를 묶으면 그 Task들이 병렬로 실행된다. 관계가 복잡해질수록 한 줄로 표현하기보다 이런 식으로 나눠서 표현하는 편이 가독성이 좋다. 이렇게 "병렬 실행 → 합류 → 다시 병렬 실행"하는 구조는 실제 워크플로우에서 자주 쓰인다.

---

## 5. 다중 Task 종속 단계 (병렬 후 합류)

```python
# DAG 연결 설정(start → 병렬 실행 → end)
start >> [task_1, task_2, task_3] >> end
```

여러 Task를 동시에 병렬로 실행한 뒤, 모든 작업이 끝나면 하나의 최종 작업(집계 등)으로 합류시키는 구조다. 예를 들어 여러 데이터 전처리 작업을 동시에 진행한 뒤, 그 결과를 하나의 집계 작업으로 모으는 경우에 이런 패턴을 사용한다.

---

## 6. Trigger Rule(트리거 규칙)

단순히 Task를 순서대로 실행하는 것뿐 아니라, **Task가 어떤 조건에서 실행될지**도 중요하다. 이 조건을 설정하는 것이 Trigger Rule이다.

- Task가 실행되기 위한 조건을 설정하는 기능
- 기본적으로 **모든 Upstream Task가 성공해야 실행됨** (`all_success`가 기본값)
- 특정 Task의 실행 결과(성공/실패/스킵)에 따라 실행 조건을 다르게 설정할 수 있음

| Trigger Rule | 설명 |
|---|---|
| `all_success` (기본값) | 모든 Upstream Task가 성공(Success) 시 실행 |
| `all_failed` | 모든 Upstream Task가 실패(Fail) 시 실행 |
| `all_done` | 모든 Upstream Task가 성공, 실패, 스킵 여부와 관계없이 실행 |
| `one_failed` | 최소 1개의 Upstream Task가 실패하면 실행 |
| `one_success` | 최소 1개의 Upstream Task가 성공하면 실행 |
| `none_failed` | Upstream Task 중 실패가 없는 경우 실행 (성공 또는 스킵) |
| `none_failed_or_skipped` | Upstream Task 중 실패와 스킵이 없는 경우 실행 (모두 성공) |
| `none_skipped` | Upstream Task가 스킵되지 않았다면 실행 |

> `none_failed`는 "성공이어야 한다"가 아니라 **"실패가 아니어야 한다"**는 조건이라는 점에 유의. 스킵된 Task가 있어도 실패만 없으면 실행된다.

### Trigger Rule 예시

```python
with DAG(
    dag_id="trigger_rule_example",
    start_date=pendulum.datetime(2024, 3, 1, tz="Asia/Seoul"),
    catchup=False,
) as dag:
    # 시작 Task
    start = EmptyOperator(task_id="start")

    # 정상 실행되는 Task
    task_1 = BashOperator(task_id="task_1", bash_command="echo 'Task 1 실행'")

    # 강제로 실패시키는 Task
    task_2 = BashOperator(task_id="task_2", bash_command="exit 1")  # 강제 실패

    # Trigger Rule: 모든 Upstream Task가 성공해야 실행됨 (실패한 Task가 있으므로 실행되지 않음)
    task_3 = BashOperator(
        task_id="task_3",
        bash_command="echo 'Task 3 실행'",
        trigger_rule="all_success",
    )

    # Trigger Rule: 최소 1개 Task가 실패하면 실행됨 (task_2가 실패했으므로 실행됨)
    task_4 = BashOperator(
        task_id="task_4",
        bash_command="echo 'Task 4 실행 (one_failed)'",
        trigger_rule="one_failed",
    )

    # Trigger Rule: 모든 Upstream Task가 끝나면 실행됨 (성공, 실패 여부 관계없이 실행됨)
    task_5 = BashOperator(
        task_id="task_5",
        bash_command="echo 'Task 5 실행 (all_done)'",
        trigger_rule="all_done",
    )

    # 종료 Task
    end = EmptyOperator(task_id="end")

    # DAG 실행 흐름 설정
    start >> [task_1, task_2]        # task_1, task_2 병렬 실행
    [task_1, task_2] >> task_3       # 모든 Task 성공 시 실행 (실행되지 않음)
    [task_1, task_2] >> task_4       # 최소 1개 실패 시 실행 (실행됨)
    [task_1, task_2] >> task_5       # 모든 Task 완료 후 실행 (실행됨)
    [task_3, task_4, task_5] >> end  # 모든 Task 종료 후 end 실행
```

`task_2`를 `exit 1`로 일부러 실패시켜 놓은 예시다. 실행 결과를 보면:

- `task_3`(`all_success`) → `task_2`가 실패했으므로 조건을 만족하지 못해 `upstream_failed` 상태가 되어 **실행되지 않음**
- `task_4`(`one_failed`) → Upstream 중 하나(`task_2`)가 실패했으므로 조건 충족 → **실행됨**
- `task_5`(`all_done`) → 성공/실패 여부와 상관없이 Upstream이 모두 끝났으므로 **실행됨**
- `end`는 별도 Trigger Rule을 지정하지 않았으므로 기본값인 `all_success` 기준으로 동작

이처럼 Trigger Rule을 활용하면 Task의 성공/실패/스킵 여부에 따라 DAG의 흐름을 유연하게 조정할 수 있다.

---

## 💡 한 줄 요약
> DAG 내 Task는 `>>`/`<<` 연산자와 리스트(`[ ]`)로 순차·병렬 실행 흐름을 구성하며, Trigger Rule(`all_success`, `one_failed`, `all_done` 등)을 통해 Upstream Task의 성공·실패·스킵 상태에 따라 다음 Task의 실행 여부를 세밀하게 제어할 수 있다.

## ❓ 더 찾아볼 것
- `trigger_rule`을 활용한 실패 알림/롤백 패턴 설계
- TaskGroup과 병렬 실행을 함께 사용할 때의 그래프 구조
- Airflow의 DAG 시각화(Graph View)에서 각 상태(success/failed/skipped/upstream_failed)별 색상 의미
