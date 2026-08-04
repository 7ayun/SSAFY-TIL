# [데이터 엔지니어링] Airflow Connections 및 Hooks

---

## 1. Connection과 Hook의 역할

Airflow에서 외부 시스템(DB, API, 클라우드 서비스 등)에 연결하려면 호스트, 포트, 데이터베이스 이름, 유저 아이디, 비밀번호 같은 접속 정보가 필요하다. 이런 정보를 DAG 코드 안에 직접 작성하면 유지보수와 보안 측면에서 좋지 않다. Airflow는 이런 접속 정보를 **Connection**이라는 개념으로 별도 관리할 수 있게 해준다.

- **Connection**: Airflow가 외부 시스템과 연결할 수 있도록 설정하는 접속 정보. Web UI 또는 환경 변수를 통해 관리한다.
- **Hook**: Connection에 등록된 정보를 이용해 실제로 외부 시스템과 통신하는 파이썬 객체. Operator 내부에서 Hook을 호출해 작업을 수행한다.

즉 Connection은 "어디에, 어떻게 접속할 것인가"에 대한 정보이고, Hook은 그 정보를 바탕으로 "실제로 통신하는 도구"다. Connection을 등록하면 Airflow가 그에 대응하는 Hook도 함께 제공한다.

| 연결 대상 | Connection Type | 사용 Hook |
|---|---|---|
| MySQL | MySQL | MySqlHook |
| PostgreSQL | Postgres | PostgresHook |
| REST API | HTTP | HttpHook |
| AWS S3 | AWS | S3Hook |

DAG 코드 안에서 DB 주소나 비밀번호, 유저 아이디를 직접 쓰는 대신, Connection ID만 참조하도록 만들면 접속 정보가 코드와 분리되어 관리된다.

## 2. Web UI에서 Connection 등록하기

Airflow Web UI의 **Admin → Connections**에서 Connection을 등록할 수 있다. `+` 버튼을 눌러 Connection ID, Connection Type 등을 입력하면 된다.

PostgreSQL 실습 예시(도커 컨테이너로 별도로 띄운 PostgreSQL 기준):
- Connection Id: `my_postgres_conn`
- Connection Type: `Postgres`
- Host: `postgres-db` (docker-compose에서 띄운 컨테이너명)
- Database: `sapi_db`
- Login: `sapi_user`
- Password: `sapi`
- Port: `5432`

로컬 PC에서 직접 접속할 때는 포트를 `5442`처럼 다르게 매핑해 쓸 수 있지만, Connection에 입력하는 포트는 컨테이너 간 통신 기준이므로 PostgreSQL이 원래 사용하는 포트인 `5432`를 그대로 사용해야 한다. Airflow 컨테이너와 PostgreSQL 컨테이너가 같은 Docker 네트워크 안에서 통신하기 때문이다.

## 3. PostgresHook으로 연결 확인하기

Connection을 등록한 뒤에는 Hook을 통해 실제 연결 여부를 확인할 수 있다.

```python
from airflow.hooks.postgres_hook import PostgresHook
from airflow.operators.python import PythonOperator
from airflow import DAG
import pendulum

# Postgres 연결 체크 함수
def fetch_postgres_data():
    hook = PostgresHook(postgres_conn_id="my_postgres_conn")
    conn = hook.get_connection("my_postgres_conn")
    print(f"Connecting to host={conn.host}, port={conn.port}, db={conn.schema}, user={conn.login}")

    with hook.get_conn() as raw_conn:
        with raw_conn.cursor() as cur:
            cur.execute("SELECT 1;")
            one = cur.fetchone()[0]
            cur.execute("SELECT version();")
            version = cur.fetchone()[0]

    if one == 1:
        print("Postgres 연결 OK")
        print(f"Postgres version: {version}")
    else:
        raise Exception("Postgres 연결 실패")

with DAG(
    dag_id="postgres_hook_python_operator",
    start_date=pendulum.datetime(2025, 8, 18, tz="Asia/Seoul"),
    schedule_interval="@daily",
    catchup=False,
    tags=["postgres", "hook"],
) as dag:

    fetch_data_task = PythonOperator(
        task_id="fetch_postgres_data",
        python_callable=fetch_postgres_data,
    )
```

`PostgresHook`에 등록된 Connection ID(`my_postgres_conn`)를 넘겨주면, Hook이 그 정보를 바탕으로 실제 DB에 연결한다. `get_conn()`으로 얻은 커넥션 객체의 커서(cursor)를 통해 SQL을 실행하고, 결과가 정상적으로 반환되면 연결이 성공한 것으로 판단할 수 있다. 이 DAG를 실행하면 로그에서 "Postgres 연결 OK"와 버전 정보가 출력되는 것을 확인할 수 있으며, 이를 통해 Connection·Hook 설정이 올바르게 되었는지 검증할 수 있다.

## 💡 한 줄 요약
> Connection은 외부 시스템 접속 정보를 코드와 분리해 관리하는 설정이고, Hook은 그 정보를 이용해 실제로 통신을 수행하는 도구다.

## ❓ 더 찾아볼 것
- Airflow Connection을 환경 변수(`AIRFLOW_CONN_*`)로 관리하는 방법
- MySqlHook, S3Hook 등 다른 Hook의 사용법
- Connection 정보 암호화(Fernet Key)의 동작 원리
