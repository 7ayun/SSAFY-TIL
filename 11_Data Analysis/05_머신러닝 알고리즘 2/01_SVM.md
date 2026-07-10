# [데이터 기초] SVM

---

## 1. SVM(Support Vector Machine)이란?

서포트 벡터 머신(SVM)은 두 클래스 사이의 **여백(Margin)을 최대화하는 초평면(Hyperplane)**을 찾는 지도학습 기반 알고리즘이다. 분류와 회귀 문제 모두에 적용할 수 있으며, 딥러닝이 본격적으로 대두되기 전까지 널리 사용되던 강력한 모델이다.

- 데이터를 고차원 공간에 매핑하여 두 클래스를 최대한 잘 구분하는 최적의 초평면을 찾는다.
- 이 초평면은 전체 데이터가 아니라, 초평면과 가장 가까운 데이터 포인트인 **서포트 벡터(Support Vector)**에 의해서만 결정된다.
- 따라서 데이터가 많지 않아도 서포트 벡터만 잘 찾으면 좋은 모델을 만들 수 있다는 장점이 있다.

## 2. 마진(Margin)의 의미

마진은 **데이터가 오분류되지 않으면서 움직일 수 있는 최대 공간**을 의미한다. 마진이 넓을수록 새로운 데이터가 들어왔을 때 잘못 분류될 가능성이 줄어들기 때문에, SVM은 이 마진을 최대화하는 결정 경계를 찾는 데 집중한다.

| 구분 | 마진의 의미 |
|---|---|
| 분류(Classification) | 두 클래스 사이의 거리. 결정 경계에서 가장 가까운 데이터(서포트 벡터)까지의 거리를 최대화 |
| 회귀(Regression, SVR) | 허용 오차 범위(ε, epsilon). 이 범위 안에 들어오는 데이터는 오차로 보지 않고, 벗어나면 페널티를 부여 |

> 예: 라벨이 파란색·빨간색 두 클래스로 나뉘어 있고 초평면이 직선이라고 할 때, 어떤 데이터가 이 직선을 넘어가는 순간 반대 클래스로 잘못 예측되어 오류가 발생한다. 이때 데이터와 직선 사이의 거리가 곧 마진이다.

## 3. Soft Margin vs Hard Margin

| 구분 | 특징 |
|---|---|
| Hard Margin | 오차를 전혀 허용하지 않고 완벽하게 분리하려는 결정 경계. 이상치·노이즈에 매우 민감해 과적합되기 쉬움 |
| Soft Margin | 결정 경계를 조금씩 넘는 데이터를 어느 정도 허용해 더 유연한 결정 경계를 만듦. 실무에서는 대부분 이 방식을 사용 |

이 허용 정도는 하이퍼파라미터 **C(Cost)**로 조절한다.

- C가 크다 → 오차를 허용하지 않으려는 경향이 강해짐 → Hard Margin에 가까워짐 (과적합 위험)
- C가 작다 → 오차를 어느 정도 허용 → Soft Margin에 가까워짐 (더 유연한 결정 경계)

## 4. Kernel Trick

SVM은 기본적으로 선형 분리를 위한 결정 경계를 만들지만, 실제 데이터는 저차원에서 선형 분리가 불가능한 경우가 많다(예: 원 모양으로 겹쳐 있는 두 클래스). 이때 사용하는 것이 **커널 트릭(Kernel Trick)**이다.

- 저차원에서는 분리가 안 되는 데이터도, 고차원 공간으로 보내면 선형 분리가 가능해질 수 있다.
- 다만 데이터를 실제로 고차원으로 변환하는 것이 아니라, **커널 함수를 통해 고차원에서의 내적(연산)만 계산**한다. 그래서 '트릭'이라고 부른다.
- 이 방식 덕분에 연산량을 크게 늘리지 않고도 고차원 매핑의 장점을 활용할 수 있다.

**대표적인 커널 함수**: Linear(선형), Poly(다항식), RBF(방사 기저 함수 — 일명 가우시안 커널, 가장 널리 사용됨), Hyperbolic Tangent(쌍곡선 탄젠트)

## 5. 장점과 단점

| 장점 | 단점 |
|---|---|
| 피처(feature)가 많고 샘플이 적을 때 유리 | C(에러 허용 가중치)를 사용자가 직접 결정해야 함 |
| 서포트 벡터만 사용하므로 데이터가 적어도 좋은 성능 | 모델 구축과 파라미터 튜닝에 시간이 오래 걸림 |
| 커널 트릭 덕분에 예측 정확도가 통상적으로 높음 | 데이터가 많아질수록 모든 데이터 간 거리를 계산해야 해 연산량·학습 속도 부담 증가 |

> 데이터가 충분히 많다면 SVM보다 트리 계열이나 다른 앙상블 모델이 더 적합할 수 있다. 반대로 샘플이 적을 때는 SVM이 상대적으로 효과적이다.

## 6. 코드 예시 (손글씨 숫자 분류)

```python
from sklearn.model_selection import train_test_split
from sklearn.svm import SVC
from sklearn.metrics import accuracy_score, confusion_matrix, classification_report

# 데이터 분할 (stratify로 클래스 비율 유지)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.3, random_state=42, stratify=y
)

# SVM 분류기 (RBF 커널 사용)
model = SVC(kernel='rbf', random_state=42)
model.fit(X_train, y_train)
y_pred = model.predict(X_test)

print(accuracy_score(y_test, y_pred))
print(confusion_matrix(y_test, y_pred))
print(classification_report(y_test, y_pred))
```

- 분류(SVC)와 회귀(SVR) 모두 사용 가능하며, 커널 종류를 바꿔가며 실험할 수 있다.
- `stratify=y`는 train/test 분할 시 클래스 비율을 원본과 동일하게 유지해, 한쪽 클래스에 데이터가 쏠려 학습되는 것을 방지한다.
- Confusion Matrix로 어떤 클래스를 어떤 클래스로 잘못 예측했는지 확인할 수 있고, Classification Report로 클래스별 정밀도·재현율·F1 점수를 확인할 수 있다.

## 💡 한 줄 요약
> SVM은 마진을 최대화하는 초평면을 찾는 알고리즘으로, Soft/Hard Margin으로 오차 허용 정도를, Kernel Trick으로 비선형 데이터까지 다룰 수 있다.

## ❓ 더 찾아볼 것
- 커널 함수의 수학적 정의와 내적 연산 원리
- SVM과 로지스틱 회귀의 결정 경계 차이
