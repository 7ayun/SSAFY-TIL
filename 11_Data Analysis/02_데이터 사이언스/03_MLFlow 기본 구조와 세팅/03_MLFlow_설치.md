# [데이터 사이언스] MLFlow 설치

---

## 1. 가상환경 생성 및 MLFlow 설치

MLFlow는 파이썬 라이브러리 기반으로 바로 설치해 사용할 수 있다. 기존에 쓰던 가상환경을 그대로 사용해도 되지만, MLFlow용으로 별도 관리하고 싶다면 새 가상환경을 만든다.

```powershell
# 가상환경 생성 및 활성화
PS D:\Project\ssafy\mlflow> python -m venv mlflow-env
PS D:\Project\ssafy\mlflow> .\mlflow-env\Scripts\Activate
(mlflow-env) PS D:\Project\ssafy\mlflow>
```

필요한 패키지들은 `requirements.txt` 기반으로 설치한다. 버전이 다르면 실습 화면과 다르게 동작할 수 있으므로, 반드시 제공된 파일 기준으로 설치하는 것이 좋다.

```
tensorflow==2.15.0
numpy==1.26.0
pandas==2.2.3
typing-extensions==4.13.0
scikit-learn==1.6.1
mlflow==2.21.3
hyperopt==0.2.7
```

## 2. MLFlow 버전 확인 및 UI 실행

설치가 끝나면 버전을 확인하고, MLFlow UI가 정상적으로 뜨는지 간단히 체크한다.

```powershell
(mlflow-env) PS D:\Project\ssafy\mlflow> mlflow --version
mlflow, version 2.21.0

(mlflow-env) PS D:\Project\ssafy\mlflow> mlflow ui
INFO:waitress:Serving on http://127.0.0.1:5000
```

- `mlflow --version` : 설치된 버전 확인
- `mlflow ui` : UI 실행 → `http://127.0.0.1:5000` 접속하면 확인 가능 (Ctrl+좌클릭으로 링크 이동)

> **주의**: `mlflow ui`는 단순히 UI 화면만 띄우는 명령으로, 실제로 사용할 Tracking Server와는 연동되지 않는다. "UI가 잘 뜨는지"를 확인하는 용도로만 쓰고, 실제 실습에서는 아래의 `mlflow server` 명령을 사용한다.

## 3. MLFlow Tracking Server 실행

실제로 사용할 서버는 `mlflow server` 명령으로 옵션을 지정해 실행한다. 실행 전, 반드시 `mlflow ui`로 띄워둔 프로세스는 꺼둔다.

```powershell
(mlflow-env) PS D:\Project\ssafy\mlflow> mlflow server `
>> --backend-store-uri sqlite:///mlflow.db `
>> --default-artifact-root ./mlruns `
>> --host 127.0.0.1 `
>> --port 5000
2025/07/24 14:33:47 INFO mlflow.store.db.utils: Creating initial MLflow database tables...
2025/07/24 14:33:47 INFO mlflow.store.db.utils: Updating database tables
```

### 옵션 설명

| 옵션 | 설명 |
|---|---|
| `--backend-store-uri` | 실험 메타데이터를 저장할 DB 지정 (예: SQLite 파일 `mlflow.db`) |
| `--default-artifact-root` | 모델·플롯 등 Artifact를 저장할 기본 경로 |
| `--host` | 서버가 수신할 IP 주소 |
| `--port` | 서버가 수신할 포트 번호 (기본값: 5000) |

**실행 절차**: ① 가상환경 활성화 → ② 로컬 환경에서 Tracking 서버 실행 → ③ `http://localhost:5000` 접속

> **⚠️ 보안 주의사항**: `--host`를 `0.0.0.0`으로 설정하면 외부에서도 서버에 접근이 가능해지므로 보안에 유의해야 한다.

서버를 실행하면 `mlflow.db`(DB 파일)와 `mlruns`(Artifact 저장 폴더) 두 가지가 생성되고, 이 안에서 실험이 추적된다.

## 4. MLflow UI 구조 살펴보기

MLFlow UI는 크게 **Experiments(실험 목록)** 와 **Runs(실행 테이블)** 두 부분으로 구성된다.

- **Experiments**: 실험명을 기준으로 구분해서 관리 (기본값은 `Default`)
- **Runs**: 하나의 실험(Experiment) 안에는 여러 번의 실행(Run)이 존재하며, 각 run은 모델·메타데이터·의존성·메트릭·파라미터 등의 정보를 담고 있음

### Run 상세 정보에서 확인 가능한 항목

| 항목 | 설명 |
|---|---|
| Run Name / ID | 실행의 이름 / 고유 식별자 (실행마다 하나씩 생성) |
| Created | 실행이 시작된 시각 |
| Duration | 실행이 완료되기까지 걸린 시간 |
| Source | 실행을 수행한 소스 코드나 환경 정보 |
| User | 해당 실행을 기록한 사용자 이름 |
| Metric | 로그된 평가 지표들 |
| Parameters | 로그된 하이퍼파라미터 값들 |
| Models | 해당 실행에서 모델 Artifact를 로그한 경우 그 모델 유형 |

run 이름을 별도로 지정하지 않으면 임의의 이름(예: `zealous-owl-166`)이 자동 생성된다. 여러 run을 선택해 Compare 기능으로 파라미터와 메트릭을 한 번에 비교할 수도 있다.

## 5. CLI를 통한 실험 생성

Tracking 서버가 켜져 있는 상태에서 새 터미널을 열어 실험(Experiment)을 생성할 수 있다. 이때 Tracking Server의 위치를 환경변수로 명확히 지정해줘야, 엉뚱한 위치에 새로운 DB/폴더가 생기는 것을 방지할 수 있다.

```powershell
(mlflow-env) PS D:\Project\ssafy\mlflow> $env:MLFLOW_TRACKING_URI = "http://127.0.0.1:5000"
(mlflow-env) PS D:\Project\ssafy\mlflow> mlflow experiments create --experiment-name "local_experiment"
Created experiment 'local_experiment' with id 1
```

- `Created experiment 'local_experiment' with id ~` 메시지가 뜨면 성공
- 환경변수 미지정 시 실행 위치에 새로운 `mlflow.db`/`mlruns`가 별도로 생성될 수 있으므로 주의
- `http://localhost:5000` 접속 후 좌측 Experiments 목록에서 `local_experiment`가 생성된 것을 확인 가능

## 6. 간단한 분류 모델 로깅 실습 (Iris 데이터셋)

### 목표 및 실험 내용

- **목표**: 머신러닝 실험의 구조를 이해하고, MLFlow로 실험을 기록·비교
- **데이터셋**: Iris(붓꽃) 데이터, Scikit-learn 내장 — Petal(꽃잎)·Sepal(꽃받침)의 길이·너비로 3개 품종을 분류
- **모델**: Logistic Regression
- **기록 방법**: MLFlow로 각 실험 정보를 자동 추적

**실험 시나리오**: 데이터 불러오기(Iris) → 데이터 분할(Train/Test) → 모델 정의(Logistic Regression) → 하이퍼파라미터 설정 → 학습 및 평가 → MLFlow 기록 → 성능 비교 및 분석

### 모델 학습 코드

```python
import mlflow
from mlflow.models import infer_signature

import pandas as pd
from sklearn import datasets
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score, precision_score, recall_score, f1_score

# Iris dataset 불러오기
X, y = datasets.load_iris(return_X_y=True)

# Train, Test 데이터셋 분할
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# 모델 하이퍼파라미터 정의
params = {
    "solver": "lbfgs",
    "max_iter": 1000,
    "multi_class": "auto",
    "random_state": 8888,
}

# 모델 학습
lr = LogisticRegression(**params)
lr.fit(X_train, y_train)

# test 데이터셋 Predict
y_pred = lr.predict(X_test)

# Metrics 계산
accuracy = accuracy_score(y_test, y_pred)
```

### MLFlow Tracking 연동 코드

```python
# Tracking Server 설정
mlflow.set_tracking_uri(uri='http://127.0.0.1:5000')  # 환경 변수로 설정했지만, 확실하게 재선언

# 새로운 MLFlow 실험을 생성 (생성하지 않으면 Default 실험에 기록됨)
mlflow.set_experiment("MLflow Quickstart")

# MLFlow 실행 시작
with mlflow.start_run():
    # 하이퍼파라미터 로그
    mlflow.log_params(params)

    # metric 로그
    mlflow.log_metric("accuracy", accuracy)

    # 태그 설정 (실행에 메모 남기기)
    mlflow.set_tag("Training Info", "Basic LR model for iris data")

    # 입출력 형태 추론
    signature = infer_signature(X_train, lr.predict(X_train))

    # 모델 로그
    model_info = mlflow.sklearn.log_model(
        sk_model=lr,
        artifact_path="iris_model",
        signature=signature,
        input_example=X_train,
        registered_model_name="tracking-quickstart",
    )
```

### 코드 상세 설명

| 함수 / 요소 | 역할 |
|---|---|
| `mlflow.set_experiment()` | 실험 이름을 지정. 지정하지 않으면 `Default` 실험에 기록됨 |
| `mlflow.start_run()` | 하나의 run(실행) 시작. `with` 구문 안의 내용이 하나의 run으로 묶여 기록됨 |
| `mlflow.log_params()` | 하이퍼파라미터 전체를 딕셔너리 형태로 한 번에 기록 (단일 파라미터는 `log_param()`) |
| `mlflow.log_metric()` | 평가지표(metric) 기록 (여러 개는 `log_metrics()`) |
| `mlflow.set_tag()` | 실험에 대한 메모·설명을 커스텀 라벨로 남김 |
| `infer_signature()` | 모델의 입력·출력 형식(타입)을 추론해 기록. 추론 요청 시 데이터 타입 불일치를 방지하는 스키마 역할 |
| `mlflow.sklearn.log_model()` | 모델 객체 저장, 저장 위치(`artifact_path`) 지정, `registered_model_name`으로 Model Registry에 등록까지 수행 |

run 이름을 따로 지정하지 않으면 랜덤하게 생성된다.

### 저장된 모델 불러와서 예측 수행

```python
# 예측을 위해 일반 Python 함수 모델(pyfunc)로 모델을 다시 불러온다
loaded_model = mlflow.pyfunc.load_model(model_info.model_uri)

predictions = loaded_model.predict(X_test)

iris_feature_names = datasets.load_iris().feature_names

result = pd.DataFrame(X_test, columns=iris_feature_names)
result["actual_class"] = y_test
result["predicted_class"] = predictions

result[:4]
```

Scikit-learn 기반(`mlflow.sklearn`)으로 모델을 저장했더라도, 대부분 `python_function` 형태로도 함께 저장되기 때문에 `mlflow.pyfunc.load_model()`로 불러오는 것이 더 범용적이고 안정적으로 동작한다.

### UI에서 결과 확인

- 실행 후 `http://127.0.0.1:5000` 접속 → `MLflow Quickstart` 실험이 생성되고, 그 안에 하나의 run이 생성됨
- Run 상세 화면의 **Parameters**, **Metrics** 탭에서 로그된 하이퍼파라미터(`solver`, `max_iter`, `multi_class`, `random_state`)와 `accuracy` 값을 확인 가능
- **Registered models** 항목에서 `tracking-quickstart` 이름으로 Model Registry에 등록된 것을 확인 가능
- **Artifacts 탭**에서 다음 파일들을 확인할 수 있음

| 파일 | 설명 |
|---|---|
| `MLmodel` | 모델의 메타정보 |
| `conda.yaml` | Python 패키지 버전 (의존성) |
| `input_example.json` | 모델에 들어가는 입력 예시 |
| `model.pkl` | 실제 모델 객체 |

- Artifact 안의 `flavors`를 보면 `sklearn`과 `python_function` 두 가지가 동시에 만들어져 있는 것을 확인할 수 있다.
- Model Registry에서 등록된 모델(Version 1)을 클릭하면 `infer_signature()`로 추론한 입력(Inputs)·출력(Outputs) 스키마를 확인할 수 있다. 이 스키마는 추후 인퍼런스 요청 시 입력 형식이 맞지 않으면 에러를 내는 등, 운영 환경에서 데이터 타입 불일치 문제를 방지하는 역할을 한다.

## 7. ML 프로젝트 생성 및 실행 실습

### 목표 및 실험 내용

- **목표**: 앞서 작성한 Iris 실험 코드를 기반으로 MLFlow Project 구성. `MLproject`와 `conda.yaml` 파일을 작성하여 재현 가능한(reproducible) 실험 환경을 구축하고, `mlflow run` 명령어로 실험을 자동화
- **데이터셋**: Iris, **모델**: Logistic Regression
- **파라미터**: `C` (정규화 강도) — 값을 바꿔가며 실험을 반복하고 정확도를 추적

### 실습 디렉토리 구조

```
~/mlflow/
├── mlflow.db          # 실험 기록용 DB (Tracking 서버용)
├── mlruns/            # 실험 기록 자동 저장 디렉토리
└── src/
    ├── iris_train.ipynb        # 기존 노트북 코드
    └── iris_project/
        ├── train.py            # 실행 코드 (.ipynb에서 변환)
        ├── MLproject           # 프로젝트 실행 정의 파일
        └── conda.yaml          # 실행 환경 정의 (선택)
```

### train.py (실험 실행 코드)

Iris 데이터셋을 로지스틱 회귀로 학습하며, `argparse`로 `-C` 파라미터를 커맨드 라인에서 입력받고, MLFlow로 파라미터·정확도·모델을 자동 기록한다.

```python
import argparse
import mlflow
import mlflow.sklearn
from sklearn.datasets import load_iris
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score

parser = argparse.ArgumentParser()
parser.add_argument("--C", type=float, default=1.0)
args = parser.parse_args()

with mlflow.start_run():
    iris = load_iris()
    X_train, X_test, y_train, y_test = train_test_split(
        iris.data, iris.target, test_size=0.2, random_state=42
    )

    model = LogisticRegression(C=args.C, max_iter=200)
    model.fit(X_train, y_train)
    preds = model.predict(X_test)

    acc = accuracy_score(y_test, preds)
    mlflow.log_param("C", args.C)
    mlflow.log_metric("accuracy", acc)
    mlflow.sklearn.log_model(model, "model")

    print(f"Accuracy: {acc:.4f}")
```

`--C` 인자를 지정하지 않으면 기본값 `1.0`이 사용된다.

### MLproject 파일

프로젝트를 어떻게 실행할지 정의하는 파일로, `mlflow run` 실행 시 이 파일을 기준으로 동작한다.

```yaml
name: IrisProject

conda_env: conda.yaml

entry_points:
  main:
    parameters:
      C: {type: float, default: 1.0}
    command: "python train.py --C {C}"
```

### conda.yaml (선택)

실행 환경을 자동으로 구성해준다. 프로젝트 실행 시 `--env-manager=local` 옵션을 사용하면 conda 없이 현재 가상환경을 그대로 쓸 수도 있다.

```yaml
name: iris_env

channels:
  - defaults

dependencies:
  - python=3.10
  - scikit-learn
  - pip
  - pip:
    - mlflow
```

### 프로젝트 실행

`--env-manager=local` 옵션을 쓰면 conda 없이 현재 가상환경을 그대로 사용한다. `-P C=값` 형태로 파라미터를 전달한다.

```bash
(mlflow-env) mlflow_user@DESKTOP:~/mlflow_project$ mlflow run src/iris_project --env-manager=local -P C=0.5

Accuracy: 1.0000
🏃 View run kindly-wolf-561 at: http://localhost:5000/#/experiments/0/runs/d174083ced894717977ba2f9ef33991d
🧪 View experiment at: http://localhost:5000/#/experiments/0
=== Run (ID 'd174083ced894717977ba2f9ef33991d') succeeded ===
```

실행 절차 요약:
1. `src/iris_project` 경로에서 `mlflow run` 실행 (MLproject 파일 기준으로 동작)
2. `--env-manager=local`로 conda 대신 현재 가상환경 사용
3. `-P C=0.5`처럼 파라미터 값 지정
4. `http://localhost:5000` 접속 후 실험 결과 확인 → Experiment ID와 실행 로그의 Run ID가 일치하면 로컬 파일에 정상 저장된 것

### 여러 C 값으로 비교 실험

C 값을 바꿔가며(예: 0.1, 0.2, 0.3, 0.4, 0.5 등) 여러 번 실행하면, 실행할 때마다 새로운 run이 쌓인다. UI에서 여러 run을 선택해 **Compare** 기능을 사용하면 파라미터(C)별 정확도 변화를 Parallel Coordinates Plot 등으로 한눈에 비교할 수 있다.

---

## 💡 한 줄 요약
> `mlflow server`로 Tracking Server를 띄우고 `mlflow.start_run()` 안에서 `log_params`·`log_metric`·`log_model`로 실험을 기록하면, MLFlow UI에서 실험을 추적·비교할 수 있고 `MLproject` + `conda.yaml`로 패키징하면 `mlflow run` 명령어 하나로 재현 가능한 실험을 반복 실행할 수 있다.

## ❓ 더 찾아볼 것
- `mlflow.autolog()`를 활용한 자동 로깅
- `MLFLOW_TRACKING_URI` 환경변수를 영구적으로 설정하는 방법
- Hyperopt와 MLFlow를 연동한 하이퍼파라미터 튜닝 자동화
- `mlflow models serve`로 REST API 서빙 실습
