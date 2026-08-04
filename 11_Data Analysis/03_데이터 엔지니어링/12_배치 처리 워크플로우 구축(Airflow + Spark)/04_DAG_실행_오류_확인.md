# [데이터 엔지니어링] DAG 실행 오류 확인

---

## 1. Airflow Web UI에서 로그 확인하기

Airflow는 각 Task의 실행 상태를 세밀하게 기록하기 때문에, 실패 원인을 파악하는 데 유용하게 활용할 수 있다.

로그 확인 절차:
1. DAG 목록에서 확인하려는 DAG ID 클릭
2. 특정 실행(Run)의 그래프(Graph) 화면에서 실패한 Task(빨간색으로 표시) 클릭
3. **Logs** 탭 클릭 → 해당 Task 하나 단위의 상세 로그 확인

실습에서 사용한 `dag_logs_processing` DAG는 `download_data → process_data → store_data` 순서로 구성되어 있다. `store_data` 단계에서 로컬에 띄워두지 않은(존재하지 않는) 데이터베이스에 의도적으로 연결을 시도해 오류를 발생시키는 예제다.

- `download_data`: 정상 성공, 로그에 다운로드 완료 및 데이터 형태 출력
- `process_data`: 정상 성공. 다만 반환값이 없어 로그에 `Return value was None`이 출력됨 — 이는 XCom에 전달할 반환값이 없다는 것을 알려주는 정상적인 안내 메시지다.
- `store_data`: **실패**. 로그의 트레이스백(Traceback)을 보면 `ConnectionError`가 발생했고, 최종적으로 `raise ConnectionError(f"데이터베이스 연결 실패: {e}")` 형태로 원인이 명확히 출력된다.

파이썬 기반 오류라면 이런 방식으로 트레이스백을 통해 정확한 실패 지점과 원인(여기서는 "데이터베이스 연결 실패")을 로그에서 바로 파악할 수 있다.

## 2. 로그 파일이 저장되는 위치

Airflow Web UI에서 확인하는 로그는 실제로는 파일 시스템에 저장되어 있으며, 메타데이터 DB에는 로그의 저장 위치 정보만 기록되고 로그 내용 자체는 저장되지 않는다.

로그는 `logs` 디렉토리 하위에 **DAG별로** 폴더가 나뉘어 저장된다.

```
logs/
├── dag_id=csv_transform_dag/
├── dag_id=dag_logs_processing/
├── dag_id=bash_processing_dag/
└── ...
```

특정 DAG의 특정 실행(Run)에 대한 실패 Task 로그를 직접 확인하고 싶다면, `dag_id → run_id → task_id` 순서로 디렉토리를 따라가면 된다.

```
logs/dag_id=dag_logs_processing/
  └── run_id=manual__2025-03-18T03:53:17.091912+00:00/
        └── task_id=store_data/
              └── attempt=1.log
```

DAG 이름과 Task 이름만 알면 Web UI를 거치지 않고도 실제 로그 파일을 직접 열어 확인할 수 있다는 것이 핵심이다. 이 방식은 Web UI 접속이 어려운 상황이거나, 여러 로그를 스크립트로 일괄 분석해야 할 때 유용하다.

## 💡 한 줄 요약
> Airflow의 Task 로그는 Web UI의 Logs 탭에서 확인할 수 있고, 실제로는 DAG ID·Run ID·Task ID 기준으로 `logs` 디렉토리에 파일 형태로 저장되며 메타데이터 DB에는 저장 위치만 기록된다.

## ❓ 더 찾아볼 것
- Airflow 로그를 S3, GCS 등 원격 스토리지로 보내는 Remote Logging 설정
- XCom의 동작 원리와 `Return value was None` 메시지가 발생하는 조건
- Task 실패 시 알림(Slack, Email)을 자동으로 보내는 방법
