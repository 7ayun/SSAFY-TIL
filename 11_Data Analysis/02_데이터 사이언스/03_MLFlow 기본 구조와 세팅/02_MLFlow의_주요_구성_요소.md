# [데이터 사이언스] MLFlow의 주요 구성 요소

---

## 1. MLFlow Tracking

Tracking은 머신러닝 실험·실행을 체계적으로 관리하기 위한 API와 UI를 제공하는 컴포넌트다. 모델 학습을 수행할 때마다 하나의 **run(실행)** 단위가 생성된다. 예를 들어 하이퍼파라미터 조합을 4가지로 바꿔가며 학습했다면, 그 각각이 하나의 run이 된다.

### 주요 기록 항목

| 항목 | 설명 |
|---|---|
| Parameter (파라미터) | 학습에 사용된 입력 값을 키-값 쌍으로 저장·추적 |
| Metric (지표) | 정확도, 손실 함수 값 등 모델 성능 지표를 숫자로 저장·추적 |
| Artifact (산출물) | 모델 파일, 이미지, 데이터 파일 등 실행 결과로 생성된 모든 형식의 출력 파일 |
| 코드 버전 | 실행에 사용된 Git 커밋 해시 등 소스 코드 버전 |
| Tags | 실험을 설명하는 커스텀 라벨(메모, 설명 등 자유롭게 부여) |

기본 동작 방식은 ML 코드가 Tracking API에 요청을 보내면, 산출물(Artifact)은 로컬 파일에 저장되고 메타데이터는 Tracking Server가 추적하는 구조다.

### Tracking 환경 구성 3가지

실험 기록을 어디에 저장하고 어떻게 접근할지에 따라 세 가지 방식으로 구성할 수 있다.

1. **로컬 호스트 사용 (기본값)**
   - 메타데이터와 모델 파일이 모두 현재 폴더(PC)에 저장
   - 별다른 설정 없이 바로 실험 기록 가능. 혼자 간단히 쓸 때 적합

2. **로컬 + 데이터베이스 연결**
   - 산출물은 로컬에 저장하되, 실험 정보(메타데이터)는 SQLite 같은 DB에서 관리
   - Tracking Server 실행 시 `--backend-store-uri` 옵션으로 DB 위치 지정
   - `mlruns` 폴더는 그대로 유지되지만, 메타데이터 조회는 DB를 통해 이루어짐

3. **원격 추적 서버 구성 (Remote Tracking Server)**
   - `mlflow server` 명령어로 서버를 하나 띄워 여러 사람이 공용으로 사용
   - Artifact는 S3, GCS 등 클라우드 스토리지나 서버 디렉토리에 저장하고, 메타데이터는 별도 DB에 저장하는 분리된 구조
   - 서버를 통해 접근 권한 관리도 가능해 실제 협업 환경에 적합

> 혼자 하거나 간단한 실험이면 로컬(1번)로 충분하지만, 반복이 많고 여러 사람이 함께 다루는 프로젝트라면 원격 서버 구성(3번)을 고려할 수 있다.

## 2. MLFlow Projects

Projects는 재현성을 위한 컴포넌트로, 머신러닝 코드를 재사용 가능한 형태로 패키징하기 위한 표준 형식을 제공한다. 즉, **일관된 환경에서 코드를 실행하고 공유**할 수 있게 해준다.

- 각 프로젝트는 코드, 데이터, 환경 설정 등을 포함하는 디렉토리(혹은 Git 저장소)로 구성
- `MLproject` 파일 안에 실행할 스크립트 경로와 하이퍼파라미터를 어떻게 받을지 명시
- 의존성 등 실행 환경 정보를 표준화된 패키지로 만들어 어디서든 동일하게 실행 가능
- 다양한 실행 환경(conda, 로컬 가상환경 등)을 지원

## 3. MLFlow Models

Models는 머신러닝 모델을 다양한 배포 환경에서 일관되게 패키징하고 배포하기 위한 표준 형식을 제공하는 컴포넌트다. 이를 통해 모델의 **재현성과 호환성**을 보장하고 다양한 라이브러리·프레임워크와 호환된다.

- **Flavor 개념**: 모델을 여러 형식(Flavor)으로 저장할 수 있다. Scikit-learn, TensorFlow, PyTorch 등 어떤 라이브러리로 학습했는지를 명시해두면, 그에 맞는 방식으로 모델을 불러올 수 있다. 대부분 `python_function` 형태도 함께 만들어져 범용적으로 로드 가능하다.
- **저장 구조**: 모델은 하나의 디렉토리로 저장되며, 구조는 다음과 같다.

```
my_model/
├── MLmodel        ← 모델 메타데이터 정의 (필수)
├── model.pkl      ← 실제 모델 파일
├── conda.yaml     ← 실행 환경 정의 (선택)
└── ...            (추가 파일들)
```

`MLmodel` 파일 예시(일부):
```yaml
artifact_path: model
flavors:
  sklearn:
    sklearn_version: 1.2.2
    pickled_model: model.pkl
  python_function:
    loader_module: mlflow.sklearn
```

- 저장한 모델은 명령어 한 줄로 서빙(serving)할 수 있다.

## 4. MLFlow Model Registry

Model Registry는 머신러닝 모델의 전 생애주기를 체계적으로 관리하기 위한 **중앙 집중화된 저장소**다. 모델을 코드처럼 버전 관리하고, 승인 워크플로우까지 적용할 수 있게 해준다.

### 주요 기능

- **모델 계보 추적**: 각 모델이 어떤 실험·실행에서 생성되었는지 추적
- **버전 관리**: 모델의 각 버전을 체계적으로 관리해 특정 버전을 재현하거나 비교
- **단계 전환**: 모델 상태를 `Staging`(검증 단계) → `Production`(실제 운영) → `Archived`(더 이상 쓰이지 않는 이전 모델)로 전환하며 배포 단계를 관리
- **주석 및 태그 추가**: 각 모델 버전에 설명이나 태그 부여

**리뷰 및 배포 연동**: 모델 승격은 리뷰어(관리자)의 승인을 거쳐 CI/CD 툴이나 자동화 스크립트로 배포까지 자연스럽게 연결될 수 있다.

## 5. MLFlow 아키텍처 (전체 흐름)

MLFlow 아키텍처는 실험 기록, 모델 저장, 프로젝트 실행, 모델 배포 등 MLOps 전 과정을 지원하도록 구성되어 있다.

```
[MLFlow Tracking Server]           [MLFlow Model Registry]
 실험 실행 시 parameter,     →      실험에서 나온 모델을 등록
 metric, model 등 자동 저장         Staging / Production / Archived로 구분
                                          ↓
                              리뷰어 승인 + CI/CD 툴
                                          ↓
                    Downstream / Automated Jobs / REST Serving
```

1. Tracking Server가 실험(파라미터, 메트릭, 모델)을 기록
2. 실험에서 생성된 모델이 Model Registry에 등록되어 상태별로 관리됨 (실험에서 나온 모델 → 테스트 → 검증 후 Staging → 운영 승인 후 Production)
3. 모델 승격은 리뷰어의 승인과 CI/CD 자동화를 거쳐 배포로 연결
4. 배포된 모델은 REST API로 실시간 서빙되거나 다운스트림 시스템/자동화된 작업에 연동되어 활용

**요약 흐름**: `실험 → 모델 관리 → 배포`

## 6. MLFlow vs 다른 MLOps 도구

| 항목 | MLFlow | Kubeflow | Metaflow | W&B |
|---|---|---|---|---|
| 출시 기업 | Databricks | Google | Netflix | Weights & Biases |
| 주요 기능 | 실험 추적, 모델 관리, 배포 | 파이프라인, 서빙, 전체 워크플로우 | 파이프라인, 스토리 관리, 재현성 | 실험 추적, 시각화, 협업 |
| 설치 난이도 | 쉬움 (로컬도 가능) | 복잡 (Kubernetes 필요) | 중간 (로컬 & 클라우드 지원) | 쉬움 (클라우드 기반) |
| 파이프라인 관리 | 제한적 (MLFlow Projects) | 강력 | 강력 | 없음 |
| 모델 배포 기능 | 기본 제공 (mlflow serve, REST API 등) | 가능 (Kserve, KFServing 등과 통합) | 없음 (별도 연동 필요) | 없음 (서빙 없음) |
| 프레임워크 종속성 | 없음 (Scikit-learn, PyTorch 등 자유) | TensorFlow 중심 (현재는 확장됨) | Python 중심 | 없음 (다양한 언어 지원) |
| 장점 | 가볍고 직관적 | 강력한 전체 파이프라인 자동화 | 데이터 사이언티스트 친화적 | 깔끔한 UI와 협업 중심 |
| 단점 요약 | 파이프라인 자동화는 약함 | 설정 복잡, 진입장벽 높음 | 서빙 기능 부족 | 서빙/파이프라인 없음 |

MLFlow는 설치가 가장 쉽고 로컬에서도 바로 사용할 수 있어, 온프레미스 환경에서 MLOps 전 과정을 가볍게 훑어보기에 적합한 도구다. 실제로 클라우드 기반 MLOps 서비스들도 내부적으로 MLFlow를 많이 활용한다.

---

## 💡 한 줄 요약
> MLFlow는 Tracking(실험 기록), Projects(재현 가능한 패키징), Models(일관된 모델 저장·배포 형식), Model Registry(중앙집중식 버전·단계 관리) 4대 컴포넌트가 유기적으로 연결되어 실험부터 배포까지의 전 과정을 지원한다.

## ❓ 더 찾아볼 것
- MLFlow Tracking Server의 REST API 구조
- MLFlow Model Serving (`mlflow models serve`) 실습
- Model Registry의 Alias 기능 (최신 버전에서 Stage를 대체)
- Kubeflow Pipelines와 MLFlow 연동 사례
