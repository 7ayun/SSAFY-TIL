# [데이터 기초] Regression

---

## 1. 회귀(Regression)란?

회귀는 **숫자(연속적인 값)를 예측**하는 문제다. 분류(Classification)가 이산적인 범주를 예측하는 것과 대비된다.

| 구분 | 예측 대상 | 예시 |
|---|---|---|
| 회귀 | 연속적인 숫자 | 주택 가격, 연비, 기온 |
| 분류 | 이산적인 범주 | 스팸 여부, 고양이/강아지 |

입력 변수는 **피처(Feature) / 독립변수**, 출력 변수는 **타겟(Target) / 종속변수**라고 부른다. 예를 들어 주택의 크기·방 개수·위치가 독립변수이고, 이를 기반으로 예측하는 주택 가격이 종속변수다. 회귀는 출력값의 추세를 학습해 새로운 데이터가 들어왔을 때 값을 예측하는 것이 목표다.

## 2. 회귀의 종류

| 종류 | 특징 |
|---|---|
| 단순 선형회귀 | 독립변수 1개, 직선으로 표현 |
| 다중 선형회귀 | 독립변수 2개 이상, 고차원 공간에서 평면 형태로 표현 |
| 다항 회귀 | 2차항 이상을 사용, 곡선 형태로 비선형 패턴을 표현 |
| 로지스틱 회귀 | 범주형 종속변수를 다루는 모델 (분류 파트에서 자세히 다룸) |

## 3. 단순 선형회귀 & 다중 선형회귀

단순 선형회귀는 다음과 같은 식으로 표현된다.

```
y = w·x + b   (w: 가중치/기울기, b: 편향/절편)
```

데이터에 가장 잘 맞는 직선(w, b)을 찾는 것이 학습 과정이다. 다중 선형회귀는 입력 변수가 여러 개(x1, x2, ...)로 늘어난 것뿐이며, 예측 방식(각 변수에 가중치를 곱하고 모두 더함)은 단순 선형회귀와 동일하다.

- **장점**: 계산이 빠르고 해석이 쉬우며, 단순한 문제에서 좋은 성능을 낸다.
- **단점**: 비선형 관계에는 성능이 떨어지고, 과적합·다중공선성 문제가 발생할 수 있다.

> **다중공선성(Multicollinearity)**: 피처들끼리 서로 강한 선형관계를 가지는 경우를 말한다. 예를 들어 '혈중알코올농도'와 '일평균 음주량'은 서로 독립적이라기보다 강하게 연관되어 있어, 다중공선성이 존재한다고 볼 수 있다.

**코드 예시 (scikit-learn, California Housing 데이터셋)**
```python
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_squared_error, r2_score
from sklearn.datasets import fetch_california_housing
import pandas as pd

data = fetch_california_housing()
df = pd.DataFrame(data.data, columns=data.feature_names)
df["MedHouseVal"] = data.target  # 목표 변수(주택 가격) 추가

# 단순 선형회귀: 피처 1개(가구당 평균 방 개수)만 사용
X = df[["AveRooms"]]
y = df["MedHouseVal"]
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)

model = LinearRegression()
model.fit(X_train, y_train)
y_pred = model.predict(X_test)

mse = mean_squared_error(y_test, y_pred)   # 결과: 약 1.29
r2 = r2_score(y_test, y_pred)              # 결과: 약 0.010
```

피처를 1개만 썼을 때는 MSE=1.29, R²=0.010으로 성능이 낮았지만, **모든 피처를 사용한 다중 선형회귀**로 바꾸자 MSE는 훨씬 낮아지고 R²는 훨씬 높아졌다. 유의미한 변수를 더 많이 고려할수록 예측 성능이 좋아질 수 있다는 것을 보여준다.

## 4. 다항 회귀 (Polynomial Regression)

다항 회귀는 x, x² 같은 고차항을 추가해 **비선형 패턴**을 학습할 수 있게 해주는 방법이다.

```python
from sklearn.preprocessing import PolynomialFeatures

poly = PolynomialFeatures(degree=2)          # X를 [1, X, X²] 형태로 변환
X_train_poly = poly.fit_transform(X_train)   # train: 변환 규칙을 학습하면서 적용
X_test_poly = poly.transform(X_test)         # test: 이미 학습된 규칙만 적용

model = LinearRegression()
model.fit(X_train_poly, y_train)
y_pred = model.predict(X_test_poly)
```

> train 데이터에는 `fit_transform`(규칙을 학습하면서 동시에 적용), test 데이터에는 `transform`(학습된 규칙만 적용)을 쓴다. test 데이터로 다시 규칙을 학습하면 안 되기 때문이다.

- **장점**: 비선형 관계를 반영할 수 있어 유연한 모델링이 가능하다.
- **단점**: 차수가 높아질수록 모델이 복잡해져 과적합 위험이 커지고, 학습 시간과 리소스도 더 많이 필요하다. 데이터가 충분하지 않으면 오히려 모델이 불안정해질 수 있다.

실제로 `HouseAge`라는 하나의 피처에 대해 단순 선형회귀와 다항 회귀(degree=2) 결과를 비교했을 때, 두 결과의 차이가 크지 않았다. 즉 해당 피처는 다항 변환으로 얻을 수 있는 이득이 크지 않은 피처였다는 것을 보여주는 사례였다. → 무조건 복잡한 모델이 좋은 것이 아니라, 데이터 특성에 맞는 모델을 선택하는 것이 중요하다.

## 💡 한 줄 요약
> 회귀는 연속적인 숫자를 예측하는 문제이며, 데이터가 선형적인지 비선형적인지에 따라 단순·다중·다항 회귀 중 적절한 모델을 선택해야 한다.

## ❓ 더 찾아볼 것
- 다중공선성을 진단하는 지표: VIF(Variance Inflation Factor)
- 정규화(Regularization) 기법: Ridge, Lasso
