# [데이터 엔지니어링] Airflow 주요 컴포넌트

---

## 1. DAG(Directed Acyclic Graph)란?

Airflow에서 워크플로우를 구성할 때 가장 중요한 개념이 **DAG**다.

- **방향성(Directed)**: 작업이 정해진 순서대로만 진행된다. (A → B → A처럼 되돌아가는 흐름은 불가능)
- **비순환(Acyclic)**: 루프 구조가 없어 무한 실행을 방지한다. 한 번 다음 단계로 넘어가면 이전 단계로 돌아갈 수 없다.

즉 DAG는 Airflow에서 하나의 작업 흐름 전체를 표현하는 단위이며, 하나의 객체라기보다는 "방향성 있고 순환하지 않는 그래프 형태"를 통칭하는 개념이다.

## 2. Task란?

- **Task**는 워크플로우를 구성하는 **개별 작업 단위**다. 하나의 DAG는 여러 개의 Task로 구성된다.
- ETL 작업 중 데이터 변환, 머신러닝 모델 실행, 파일 이동 등 실제 실행 명령이 될 수 있으며, 얼마나 작게/크게 쪼갤지는 사용 목적에 따라 달라진다.
- 보통 Python 코드 기반으로 정의한다.

DAG는 여러 개의 Task로 구성되어 하나의 흐름으로 동작하는 형태로 만들어진다고 이해하면 된다.

## 3. Airflow 내부 컴포넌트

Airflow는 하나의 프로그램처럼 보이지만, 실제로는 여러 서브 시스템이 함께 동작하는 구조다.

| 컴포넌트 | 역할 |
|---|---|
| **Scheduler** | DAG 파일을 파싱하고, Task 및 DAG를 모니터링하며 실행을 스케줄링하는 핵심 컴포넌트 |
| **Executor** | Task 실행 방식(전략)을 결정하는 컴포넌트 (Local, Celery, Kubernetes 등) |
| **Worker** | 실제로 Task를 실행하는 프로세스 |
| **Metadata Database** | DAG, DAG Run, Task Instance, Variables, Connections 등 여러 컴포넌트가 공유하는 데이터를 저장하는 곳 |
| **Web UI (Webserver)** | Metadata Database와 통신하며 DAG·Task 상태를 웹에서 보여주고 사용자와 상호작용할 수 있게 함 |

전체 흐름은 다음과 같다.

```
DAG Directory(파이썬 파일) → Scheduler(스캔·파싱·실행 판단) → Executor(실행 전략 수립)
→ Worker(실제 실행) → Metadata Database(결과 기록) → Webserver(UI로 표시)
```

### DAG Directory

- 파이썬으로 작성된 DAG 파일을 저장하는 공간으로, `dag_folder` 또는 `dags_folder`라고 불린다.
- 기본적으로 `$AIRFLOW_HOME/dags/`로 설정되어 있다.
- DAG 파일을 작성해 이 디렉토리에 저장해두면, Scheduler가 **주기적으로 스캔**해서 파싱한다. 즉 파일을 올린다고 바로 UI에 반영되는 것은 아니고, Scheduler의 스캔 주기만큼 기다려야 반영된다.

### Scheduler

- DAG 파일을 파싱하고, Task 및 DAG Run과 Task Instance의 상태를 관리하며 Executor에게 실행을 요청하는 역할을 한다.
- **DAG 파일 처리 과정**
  1. DAG 파일 검색 및 로드
  2. DAG 파일 파싱 및 해석
  3. DAG 등록 및 실행 준비
- 여기서 등장하는 개념 정리:
  - **DAG Run**: 하나의 DAG가 한 번 실행되는 사이클(예: 오늘 새벽 1시 실행 = DAG Run 1, 내일 새벽 1시 실행 = DAG Run 2)
  - **Task Instance**: 특정 DAG Run 안에서 개별 Task가 가지는 실행 상태

### Executor

- Executor는 **Scheduler가 생성하는 서브 프로세스**로, Queue에 들어온 Task Instance를 실제로 실행하는 역할을 담당한다.

| 분류 | Executor | 설명 |
|---|---|---|
| 단일 프로세스형 (Single-Process) | Sequential Executor | 한 번에 하나의 Task만 순차적으로 실행. 개발·테스트용. 실무에서는 거의 사용되지 않음 |
| 로컬 병렬형 (Local Multi-Process) | Local Executor | Scheduler 내부에서 여러 Task를 병렬로 실행 (멀티프로세싱 기반) |
| 분산형 (Distributed) | Celery Executor | 여러 워커 노드에 Task를 분산 실행. 대규모 분산 환경에 적합하며 워커 수를 늘려 수평 확장 가능 |
| 분산형 (Distributed) | Kubernetes Executor | 각 Task를 독립된 Pod으로 실행하여 완전한 격리와 자동 확장(스케일 인/아웃)을 지원 |

> 이번 실습에서 사용하는 Docker Compose 환경은 **Celery Executor**로 기본 세팅되어 있어, 어느 정도 분산 처리가 가능한 구조를 갖추고 있다.

### Metadata Database

- Airflow의 DAG, DAG Run, Task Instance, Variables, Connections 등 여러 컴포넌트에서 공유해야 하는 데이터를 저장한다.
- Task 실행 결과, 재시도 횟수, 성공/실패 이력 등도 여기에 기록·추적되어 문제 발생 시 원인 분석에 활용된다.
- Connections나 Variables(여러 DAG에서 공용으로 쓸 수 있는 전역 변수 개념)도 Metadata Database를 통해 중앙 관리된다.

### Webserver

- Metadata Database와 통신하며 DAG, DAG Run, Task Instance, Variables, Connections 등의 데이터를 가져와 웹 UI로 보여주고 사용자와 상호작용할 수 있게 한다.
- 실무에서 가장 자주 다루게 되는 것은 결국 **DAG Directory(코드 작성)**와 **Webserver(실행 확인)** 두 곳이며, Metadata Database는 문제가 생기지 않는 이상 직접 들여다볼 일이 적다.

## 4. Deployment 구조

### Basic Deployment (기본 배포)

- **Airflow User**가 DAG 파일을 작성(author)해서 **DAG files** 디렉토리에 올리고, 플러그인·패키지를 설치(install)한다.
- **Scheduler**가 DAG files를 읽어(read) 파싱·스케줄링·실행을 담당한다.
- **Webserver**가 UI를 제공하며, 사용자가 조작(operate)한다.
- Scheduler와 Webserver 모두 **Metadata DB**를 공유하며 상태를 주고받는다.
- 기본 배포 형태에서는 Scheduler와 Task 실행 환경(Executor/Worker)이 같은 서버 안에 있는 구조로 표현되며, 이번 Docker Compose 실습 환경도 결국 하나의 서버(로컬 환경) 안에 모든 컨테이너가 함께 뜨는 형태다.

### Distributed Airflow architecture (분산 배포)

규모가 커지면 역할별로 컴포넌트를 분리해서 운영한다.

- **DAG Author**가 DAG files를 작성해 동기화(sync)한다.
- **Scheduler(s)**는 언제 작업을 실행할지 판단하고, **Worker(s)**는 실제 Task를 실행한다. Worker를 여러 개 두면 여러 Task를 병렬로 처리할 수 있다.
- **Triggerer(s)**는 외부 조건(이벤트)을 기다리는 작업을 담당한다. 예를 들어 특정 이벤트가 실제로 트리거할 만한 조건인지 판단하는 역할이다.
- **Deployment Manager**는 플러그인·패키지를 모든 서버에 설치하는 등 배포 관리를 담당한다.
- 모든 컴포넌트는 동일한 **Metadata DB**를 바라보고 상태를 공유하며, **API Server**를 통해 Operations User(운영자)가 접근한다.

> 즉 소규모로는 Basic Deployment처럼 한 서버에 다 몰아서 구성할 수 있고, 규모가 커지면 Scheduler·Worker·Triggerer를 역할별로 분리한 Distributed 구조로 확장할 수 있다.

## 💡 한 줄 요약
> Airflow는 DAG(방향성·비순환 그래프)로 여러 Task의 실행 흐름을 정의하며, Scheduler가 DAG를 파싱해 Executor에게 실행을 지시하고 Worker가 실제로 처리한 결과를 Metadata Database에 저장해 Webserver(UI)로 보여주는 구조로 동작한다.

## ❓ 더 찾아볼 것
- Kubernetes Executor의 Pod 생성·격리 방식
- Triggerer 컴포넌트가 Deferrable Operator와 함께 동작하는 원리
- Airflow의 Connections와 Variables 설정 방법
