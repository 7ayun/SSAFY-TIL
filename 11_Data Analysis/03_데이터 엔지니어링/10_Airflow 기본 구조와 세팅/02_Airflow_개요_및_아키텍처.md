# [데이터 엔지니어링] Airflow 개요 및 아키텍처

---

## 1. Airflow의 장점

- **Python 기반** → 개발자에게 익숙한 언어로 DAG(작업 흐름)를 코드로 작성할 수 있어 접근이 쉽다.
- **강력한 Web UI 제공** → 작업 실행 상태를 시각적으로 직관적으로 모니터링할 수 있다.
- **Task 간 의존성 관리 용이** → DAG 구조를 활용해서 "이 Task가 끝나야 저 Task로 넘어간다"와 같은 순서를 명확하게 처리할 수 있다.
- **확장성이 높음** → 다양한 Executor(작업 실행 방식)를 지원한다. Local Executor, Celery Executor, Kubernetes Executor 등이 있으며, 이번 실습에서는 Docker 기반으로 Celery Executor를 사용해 돌아가는 형태를 다룬다.
- **장애 복구 기능 제공** → 재시도(Retry)나 알림 설정이 크게 어렵지 않다.

## 2. Airflow의 단점

- **초기 설정 및 학습 곡선이 가파름** → 구성 요소(스케줄러, 워커, 웹서버 등)가 많고, 익숙하지 않은 개념이 많아 처음 접할 때 복잡하게 느껴질 수 있다. 설정 파일 방식도 처음 접하는 사람에게는 낯설 수 있다.
- **실시간 데이터 처리에는 적합하지 않음** → 실시간으로 수집된 데이터를 일정 주기로 모아 배치 처리하는 데는 적합하지만, 이벤트가 발생할 때마다 즉시 하나씩 처리하는 실시간 스트리밍 구조를 다루는 도구는 아니다. (이벤트 발생 시 DAG 자체를 트리거하는 것은 가능하지만, 실시간 데이터를 하나하나 읽어 처리하는 구조는 아니다.)
- **복잡한 DAG의 경우 성능 튜닝 필요** → 하나의 Airflow에서 수백 개의 DAG, 각 DAG 안의 여러 Task가 함께 운영되면 스케줄링 속도가 느려지는 등 성능을 높여야 하는 지점이 생길 수 있다.

> Airflow는 학습 곡선이 가파른 단점을 보완하기 위해 **공식 Docker Compose 파일**을 제공한다. 참고로 Airflow는 현재 3.x 버전이 나와 있지만, 아직 레거시로 2.x 버전을 쓰는 곳이 훨씬 많고 2→3 마이그레이션 자체는 어렵지 않기 때문에, 이번 실습에서는 **Airflow 2.10.5** 버전을 기준으로 진행한다.

## 3. Airflow 설치 및 환경세팅 (Docker Compose 기반)

공식 문서에서 제공하는 `docker-compose.yaml`을 그대로 쓸 수도 있지만, 실습 편의를 위해 일부를 수정한 파일을 사용한다. (공식 문서: https://airflow.apache.org/docs/apache-airflow/stable/howto/docker-compose/index.html)

### 원본 대비 주요 변경 사항

| 변경 항목 | 설명 |
|---|---|
| PostgreSQL 포트 (5432 → 5433) | 로컬/프로젝트에서 이미 5432 포트로 PostgreSQL을 띄워둔 경우가 많아 포트 충돌을 피하기 위해 변경. 호스트에서 직접 접근할 때는 5433, 컨테이너끼리 내부 통신할 때는 원래 포트(5432)를 사용 |
| 타임존(TZ) 설정 | 기본적으로 UTC 기준이라 별도 설정이 없으면 "매일 00시 실행"이 한국 시간 기준 오전 9시에 실행되는 문제가 생김. 이를 방지하기 위해 한국 시간(Asia/Seoul) 기준으로 설정 |
| 예제 DAG 제거 (`AIRFLOW__CORE__LOAD_EXAMPLES: 'false'`) | 기본값을 `true`로 두면 약 70개의 예제 DAG가 함께 로드되는데, 운영 환경뿐 아니라 실습 시에도 원하는 DAG를 찾기 불편하므로 비활성화 |
| 네트워크 통일 | 추후 Spark 등 다른 Compose 파일과 통신할 수 있도록 공용 네트워크(shared network)로 구성 |

### 설치 순서

```bash
# 1. docker-compose.yaml 다운로드 (공식 문서 기준 예시, 실습에서는 변형된 파일 제공)
$ curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.10.5/docker-compose.yaml'

# 2. Airflow에서 사용할 디렉토리 생성
$ mkdir -p ./dags ./logs ./plugins ./config

# 3. 로컬 유저 ID를 Airflow 컨테이너의 유저 ID로 맞춰서 권한(Permission Denied) 이슈 방지
$ echo -e "AIRFLOW_UID=$(id -u)" > .env

# 4. Spark 등 다른 Compose 파일과 통신 가능하도록 공용 네트워크 생성
$ docker network create shared_net

# 5. 메타데이터 초기화
$ docker compose up airflow-init

# 6. Airflow 실행
$ docker compose up
```

- `AIRFLOW_UID`는 로컬 유저 계정의 UID를 그대로 Airflow 컨테이너 내부 유저 ID로 사용하기 위한 설정이다. `id -u`로 확인한 값을 `.env` 파일에 기록해두면 Airflow가 이를 읽어 사용한다.
- `docker network create shared_net`은 Airflow 컨테이너와 별도의 Compose로 띄운 Spark 등 다른 컨테이너들이 서로 DNS 기반으로 통신할 수 있도록 네트워크를 미리 만들어두는 과정이다.
- `docker compose up airflow-init`은 메타데이터 DB 초기화 등 Airflow가 처음 뜨기 위한 세팅을 진행하는 초기화 전용 실행이며, 이후 `docker compose up`으로 실제 서비스(Webserver, Scheduler, Worker 등)를 띄운다.

### 실행 상태 확인

```bash
$ docker ps
```

정상적으로 실행되면 `airflow-triggerer`, `airflow-worker`, `airflow-webserver`, `airflow-scheduler`, `postgres`, `redis` 컨테이너가 모두 `Up`(healthy) 상태로 뜨는 것을 확인할 수 있다.

### Web UI 접속

- 접속 주소: `http://localhost:8080`
- 로그인 계정: Username `airflow` / Password `airflow`

로그인 후 DAGs 목록 화면이 뜨는데, `AIRFLOW__CORE__LOAD_EXAMPLES`를 `false`로 설정했다면 예제 DAG 없이 깔끔한 상태로 시작된다.

## 4. 개발 환경용 Airflow 라이브러리 설치

실제 Airflow 자체는 Docker 컨테이너 안에서 돌아가지만, 로컬 개발 환경(IDE)에서도 Airflow 라이브러리를 설치해두면 좋다. 이는 Airflow를 로컬에서 직접 실행하기 위한 목적이 아니라, **IDE가 DAG 관련 코드를 제대로 인식**하도록(자동완성, import 오류 방지 등) 하기 위한 목적이다. 필수는 아니지만 있으면 편리한 요소다.

```bash
# 가상환경에서 실행 (Python 버전에 맞는 constraints 파일 사용)
pip install "apache-airflow[celery]==2.10.5" \
  --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-2.10.5/constraints-3.10.txt"
```

- 단순히 `pip install apache-airflow`로 설치하지 않고, 위처럼 `constraint` 옵션과 함께 설치하는 이유는 Airflow가 의존성이 매우 많은 패키지이기 때문이다. Celery Executor 구성에 맞는 의존성까지 함께 버전 고정(constraint)해서 설치해야 충돌 없이 안전하게 설치할 수 있다.
- `[celery]`가 붙는 이유는 이번 Docker Compose 환경이 기본적으로 **Celery Executor**로 세팅되어 있기 때문이다.

## 💡 한 줄 요약
> Airflow는 Python 기반 접근성과 Web UI, 확장성이 장점이지만 초기 학습 곡선이 가파른 도구로, 공식 제공 Docker Compose 파일(포트·타임존·예제 DAG 설정을 실습 환경에 맞게 조정)을 이용해 `airflow-init` → `docker compose up` 순서로 손쉽게 띄울 수 있다.

## ❓ 더 찾아볼 것
- Airflow 2.x와 3.x의 주요 차이 및 마이그레이션 포인트
- Celery Executor의 내부 동작(Broker로 Redis를 사용하는 구조)
- constraints 파일이 관리하는 의존성 버전 고정 방식
