# [데이터 사이언스] MLFlow Tracking 실험 관리

---

## 1. 실험 관리 전략이란

단순히 실험을 기록하는 것을 넘어, **실험을 체계적으로 수립·계획·수행하고 그 결과를 비교·분석하는 전체 과정**을 전략적으로 설계하는 것을 의미한다.

기본 흐름은 다음과 같다.

```
목표 수립 → 비교 단위 정의 → 비교 가능한 실험 설계 → 자동화하여 반복 실행
```

이렇게 실험의 목적과 범위를 명확히 구분해두면, 실험이 많아져도 결과를 혼동하지 않고 정리된 상태로 비교·분석·공유할 수 있다. 무작위로 실험을 던지는 것이 아니라, **맥락이 있는 관리 가능한 단위**로 만드는 것이 전략의 핵심이다.

## 2. 저장 구조: mlruns와 Experiment / Run 계층

MLflow는 기본적으로 `mlruns`라는 디렉토리에 실험 결과를 저장한다.

- 각 실험(**Experiment**)은 고유 ID를 가지며, 그 하위에 실행(**Run**)마다 Run ID가 하나씩 생성된다.
- 하나의 Run 디렉토리 아래에는 **metric, parameter, artifact, meta 파일** 등이 저장된다.
- 로컬 디렉토리 저장이 기본이지만, 설정을 바꾸면 백엔드 저장소를 **DB(RDB)** 나 **S3 같은 외부 클라우드 저장소**로 변경할 수 있다.
  - 예: 메타데이터는 SQLite 등 RDB에, 모델 파일 자체는 S3에 저장
- 저장된 내용을 직접 디렉토리에서 확인할 필요 없이, **MLflow UI**에서 실험(Run)을 클릭하면 파라미터·메트릭·아티팩트·메타 정보를 한눈에 확인할 수 있다.

## 3. 실험 / 런 네이밍 전략

정해진 정답은 없지만, 이름만 보고도 어떤 목적의 실험인지 알 수 있도록 구성하는 것이 협업과 기록 모두에 유리하다.

- **Experiment 명**: 실험의 큰 목적 단위 (예: `Customer_Segmentation`, `baseline_randomforest`)
- **Run 명**: Experiment 안에서 세부적으로 변경되는 조건 (예: `XGBoost_tuned_001`, `XGBoost_tuned_002`)

```python
with mlflow.start_run(run_name="XGBoost_tuned_001"):
    mlflow.log_param("max_depth", 5)
    ...
```

어떤 단위로 나눌지는 팀에서 사용하는 방식을 따르는 것이 일반적이다.

## 4. 로깅 항목: Parameter, Metric, Tag, Artifact

MLflow는 실험 정보를 4가지로 구조화해서 기록한다.

| 항목 | 의미 | 예시 |
|---|---|---|
| Parameter | 실험 조건 | `max_depth`, `learning_rate` |
| Metric | 성능 결과 | `accuracy`, `rmse` |
| Tag | Run에 대한 설명 (메타 정보) | 작성자, 실험 목적, 데이터 버전 |
| Artifact | 실험 결과물 | ROC curve 이미지, 모델 파일 등 |

이렇게 조건과 결과를 구분해서 기록하면 필터링·검색이 쉬워지고, 협업자와 공유할 때도 구조적으로 정리된 정보를 전달할 수 있다.

### 태그 활용 예시

태그는 Key-Value 형태로 자유롭게 기록하는 메타 정보로, UI/API에서 태그 기반 검색·필터링에 활용된다.

| 태그 키 | 용도 |
|---|---|
| Author | 작성자 이름 |
| Description | 실험 목적 |
| Notes | 참고 사항 |
| Data_version | 사용한 데이터 버전 |

```python
mlflow.set_tag("author", "kim")
mlflow.set_tag("description", "XGBoost with SMOTE")
mlflow.set_tag("data_version", "v2.1")
```

## 5. 자동 로깅(Autolog) vs 수동 로깅

`mlflow.autolog()` 한 줄만 추가하면 학습 과정에서 하이퍼파라미터, 성능 지표, 모델 정보 등을 자동으로 기록해준다. 사이킷런, PyTorch, TensorFlow, XGBoost, Spark 등 다양한 프레임워크를 지원한다.

```python
import mlflow

mlflow.autolog()

with mlflow.start_run():
    model.fit(X_train, y_train)
```

다만 자동 로깅은 완벽하지 않다. 팀에서 정의한 **커스텀 지표**나 중간 결과 등은 여전히 `log_param`, `log_metric`, `log_artifact`로 **수동 로깅**이 필요하다.

## 6. 실험 비교와 검색

- **UI 기반**: 필터링(Filter Experiment), 지표 정렬, 그래프 비교(아티팩트 이미지), 결과 다운로드가 가능
- **코드 기반**: `search_runs()`로 특정 Experiment 내 조건에 맞는 Run을 검색·정렬하여 베스트 Run을 코드상에서 바로 추출할 수 있다.

```python
runs = mlflow.search_runs(
    experiment_ids=["1"],
    filter_string="params.max_depth = '5'",
    order_by=["metrics.rmse ASC"],
)
best_run_id = runs.iloc[0]["run_id"]
```

이렇게 하면 UI를 직접 보지 않고도 코드에서 실험 결과를 활용할 수 있다.

## 7. 예외 처리와 실패 실험 관리

실험이 중간에 에러로 종료되면 로그가 누락되거나 Run 상태가 `FINISHED`가 아닌 `FAILED`로 남을 수 있다. `with mlflow.start_run()` 블록을 명시적으로 종료해야 로그가 깔끔하게 정리된다.

```python
try:
    with mlflow.start_run():
        # 학습 및 로깅
        model.fit(X_train, y_train)
        mlflow.log_metric("rmse", rmse)
except Exception as e:
    print(f"실험 실패: {e}")
```

실패한 실험의 로그를 남겨두면 이후 `search_runs()`로 다시 불러와 재실행·복구하는 데 활용할 수 있다.

## 8. config.yaml 기반 반복 실험 자동화

반복되는 실험 조건을 하나하나 코드에 입력하는 것은 비효율적이므로, `config.yaml`에 실험 조건 조합을 정의해두고 읽어서 자동으로 반복 실행한다.

```yaml
learning_rate: [0.1, 0.01]
batch_size: [16, 32]
optimizer: [adam, sgd]
```

```python
import yaml
from itertools import product

with open("config.yaml") as f:
    config = yaml.safe_load(f)

keys, values = config.keys(), config.values()

for combo in product(*values):
    params = dict(zip(keys, combo))
    with mlflow.start_run():
        mlflow.log_params(params)
        # 학습 및 평가 수행
```

파라미터 3개에 각 2개의 값이 있으면 총 8개 조합이 자동으로 생성되어 반복 실행된다. 이렇게 하면 실험 조건을 체계적으로 구성하고 재사용성을 높일 수 있다.

## 9. MLflow 계층 구조와 Nested Run

MLflow Tracking은 **Experiment → Run**의 계층 구조로 이루어져 있다.

- **Experiment**: 실험의 상위 그룹 (예: `Customer_Segmentation`)
- **Run**: Experiment 하위의 개별 실행 단위 (하이퍼파라미터 조합별 실행 등)

여러 Run을 하나의 묶음으로 관리하고 싶을 때는 **Nested Run**을 사용한다. 예를 들어 학습률을 여러 값으로 바꿔가며 실험할 때, 이를 독립된 Run들로 흩어놓기보다 하나의 부모 실험 아래 세부 실험(Child Run)으로 묶을 수 있다.

```python
with mlflow.start_run(run_name="parent_experiment") as parent_run:
    for lr in [0.001, 0.01, 0.1]:
        with mlflow.start_run(run_name=f"lr_{lr}", nested=True):
            mlflow.log_param("learning_rate", lr)
            # 학습 및 로깅
```

`nested=True`로 선언하면 Parent Run 아래 Child Run으로 연결되고, 이를 더 반복하면 Grandchild Run까지도 만들 수 있다. 이렇게 트리 구조로 실험을 묶어 관리할 수 있다는 것이 장점이다.

## 10. Run ID 활용

MLflow는 실험을 실행할 때마다 각 Run에 고유한 **Run ID**를 부여한다. 이는 다음과 같은 용도로 활용된다.

- 어떤 설정에서 나온 모델인지 식별
- 저장된 모델을 다시 로드
- 실험 로그와 저장된 모델을 연결
- Run ID 기반으로 모델 레지스트리에 등록

```python
model_uri = f"runs:/{run_id}/model"
model = mlflow.pyfunc.load_model(model_uri)
```

## 11. 하이퍼파라미터 튜닝 기법

| 기법 | 방식 | 장점 | 단점 |
|---|---|---|---|
| Grid Search | 지정된 조합을 완전 탐색 | 최적 조합을 놓치지 않음, 직관적 | 조합 수가 많아지면 비효율적 |
| Random Search | 전체 공간 중 일부를 무작위 탐색 | 빠르고 효율적, 고차원에 유리 | 최적값을 놓칠 수 있음 |
| Hyperopt | 베이지안 기법(TPE 등) 기반 탐색 | 이전 결과를 바탕으로 더 나은 지점을 탐색해 효율적으로 수렴 | 그 자체로 완벽한 최적화를 보장하지는 않음 |

- Grid Search는 파라미터가 적고 조합 수가 크지 않을 때 적합하다.
- Random Search는 조합 수가 많거나 어떤 파라미터가 중요한지 미리 알기 어려울 때 적합하다.
- Hyperopt는 이전 시도 결과를 바탕으로 더 나은 후보를 탐색하는 베이지안 최적화 방식으로, 리소스와 시간이 한정된 상황에서 Grid/Random Search보다 효율적으로 좋은 조합에 수렴할 수 있다. (참고: Optuna, Ray Tune 등도 유사한 자동화 튜닝 도구)

---

## 💡 한 줄 요약
> MLflow Tracking은 Experiment-Run 계층 구조 위에서 Parameter/Metric/Tag/Artifact를 체계적으로 기록·비교하고, config.yaml과 Nested Run, 하이퍼파라미터 튜닝 기법을 결합해 반복 실험을 자동화·효율화하는 도구다.

## ❓ 더 찾아볼 것
- MLflow 백엔드 저장소를 S3 + RDB로 구성하는 실제 설정 방법
- Hyperopt의 TPE(Tree-structured Parzen Estimator) 알고리즘 원리
- Optuna, Ray Tune 등 다른 자동화 하이퍼파라미터 튜닝 도구와의 비교
