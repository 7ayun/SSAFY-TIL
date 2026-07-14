# [데이터 기초] Classification

---

## 1. 분류(Classification)란?

분류는 입력 데이터가 **여러 범주(카테고리) 중 어디에 속하는지** 판단하는 문제다. 스팸 메일 여부, 강아지/고양이 사진 구분, 이미지 속 객체 인식 등 많은 문제가 분류를 기반으로 풀린다.

```
f(x) → y   (x: 피처 벡터, y: 클래스)
```

회귀와 분류는 다음과 같이 비교하면 가장 빠르게 이해할 수 있다.

| 구분 | 회귀 | 분류 |
|---|---|---|
| 출력값 | 연속적인 숫자 | 이산적인 범주 |
| 예시 (공부시간 → 결과) | 공부시간 → 시험 점수 | 공부시간 → 합격/불합격 |
| 학습 결과 | 데이터를 잘 설명하는 선 | 데이터를 나누는 경계 |

- **이진 분류 (Binary Classification)**: 출력이 음성(0)/양성(1) 두 가지
- **다중 클래스 분류 (Multi-class)**: 여러 클래스 중 하나를 선택

## 2. 결정경계 (Decision Boundary)

모델이 클래스를 구분하기 위해 공간을 나누는 기준선(또는 기준면)을 **결정경계**라고 한다. 새로운 데이터가 주어지면, 그 데이터가 결정경계의 어느 쪽에 위치하는지에 따라 클래스가 결정된다.

- 선형 경계: 데이터가 직선/평면으로 깔끔하게 나뉘는 경우
- 비선형 경계: 더 복잡하고 휘어진 경계가 필요한 경우

(2·3차원까지는 시각화가 가능하지만, 4차원 이상부터는 시각적으로 이해하기 어려우므로 개념적으로 받아들이는 것으로 충분하다.)

## 3. 로지스틱 회귀 (Logistic Regression)

선형회귀는 이론상 예측값이 -∞ ~ +∞까지 나올 수 있어, 결과가 특정 범위로 제한되어야 하는 분류 문제에는 그대로 쓰기 어렵다. 예를 들어 풍속(윈드스피드)으로 비행기 지연 여부를 예측할 때, 회귀식의 결과값이 그대로는 "지연/정상"으로 해석되기 어렵다.

이 문제를 해결하는 것이 **시그모이드 함수(Sigmoid Function)**다.

```
g(x) = 1 / (1 + e^(-x))
```

- x가 커질수록 g(x) → 1
- x가 작아질수록 g(x) → 0
- 출력값을 확률로 해석 (예: 0.9 → 지연될 확률 90%)
- 특정 threshold를 기준으로 최종 분류(Yes/No)를 결정

> 시그모이드 함수는 딥러닝에서 활성화 함수(Activation Function)로도 사용된다.

로지스틱 회귀는 **이진 분류에 주로 활용되는 대표적인 선형 모델**이며, 입력을 기반으로 특정 클래스에 속할 확률을 예측한다. 이름에 "회귀"가 들어가지만 실제로는 분류 문제에 사용되는 모델이다.

- **장점**: 해석이 간단하고 계산이 빠르다.
- **단점**: 비선형 데이터에는 적합하지 않고, 복잡한 결정경계를 만들기 어렵다.

**코드 예시 (Iris 데이터셋 - 붓꽃 품종 분류)**
```python
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score, confusion_matrix, classification_report

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)   # 표준화: train은 fit_transform
X_test_scaled = scaler.transform(X_test)         # test는 transform

model = LogisticRegression(max_iter=200)          # max_iter: 반복 횟수(보통 100~300 사이 값 사용)
model.fit(X_train_scaled, y_train)
y_pred = model.predict(X_test_scaled)

acc = accuracy_score(y_test, y_pred)   # 결과: 약 0.9111 (91%)
```

## 4. 결정 트리 (Decision Tree)

결정 트리는 **트리 구조로 데이터를 분할**하며 분류(혹은 회귀)하는 알고리즘이다. 각 피처를 기준으로 데이터를 나누는 최적의 조건을 찾고, 최종 리프 노드(leaf node)에 도달하면 그것이 예측 결과가 된다.

분할 기준은 데이터가 얼마나 **순수(pure)**해졌는가로 판단한다. 하나의 노드 안에 하나의 클래스만 있으면 순수한 상태다.

| 지표 | 의미 | 좋은 분할일수록 |
|---|---|---|
| 엔트로피 (Entropy) | 데이터가 얼마나 뒤섞여 있는가(불확실성) | 값이 낮음 (한 클래스로 순수할 때 0에 수렴) |
| 지니 불순도 (Gini Impurity) | 노드에서 데이터를 하나 뽑았을 때 잘못 분류될 확률 | 값이 낮음 (잘못 분류될 확률이 낮음) |

- 엔트로피는 클래스 비율이 정확히 반반(50:50)일 때 가장 크다(가장 불확실한 상태).
- 두 지표 모두 값이 낮을수록 더 좋은 분할 기준이 된다.

- **장점**: 트리 구조로 시각화가 가능해 해석이 쉽고, 분기를 반복하며 복잡한(비선형) 결정경계도 만들어낼 수 있다.
- **단점**: 트리가 너무 깊어지면 과적합 위험이 커지고, 작은 데이터 변화에도 민감하게 반응할 수 있다. 데이터가 많아질수록 정해둔 깊이로는 성능이 떨어질 수도 있다.

**코드 예시**
```python
from sklearn.tree import DecisionTreeClassifier

model = DecisionTreeClassifier(random_state=42)   # random_state: 랜덤하게 분할되는 방식을 고정하는 시드(seed)
model.fit(X_train, y_train)
y_pred = model.predict(X_test)
```

동일한 Iris 데이터셋에 적용했을 때 로지스틱 회귀보다 **정확도는 더 높았지만, 틀리는 데이터의 패턴은 서로 달랐다**. 결정 트리의 장점 중 하나는 어떤 조건 때문에 오분류가 발생했는지 트리 구조를 직접 들여다볼 수 있다는 점이다.

## 💡 한 줄 요약
> 분류는 입력을 범주로 나누는 문제이며, 로지스틱 회귀는 확률 기반의 선형 경계를, 결정 트리는 순도(불순도) 기반의 비선형 경계를 만들어 분류한다.

## ❓ 더 찾아볼 것
- 결정 트리의 과적합을 막는 가지치기(Pruning) 기법
- 랜덤 포레스트(Random Forest) 등 트리 기반 앙상블 모델
