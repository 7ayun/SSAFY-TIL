# [데이터 기초] Bias, Variance, Regularization

---

## 1. 예측 오차의 구성

머신러닝 모델의 총 예측 오차는 세 가지 요소로 나눌 수 있다.

```
총 오차 = 편향(Bias)² + 분산(Variance) + 불확실성(Noise)
```

| 요소 | 의미 |
|---|---|
| 편향 (Bias) | 모델이 문제를 지나치게 단순화해서 발생하는 오류. 중요한 패턴을 놓치게 됨 |
| 분산 (Variance) | 모델이 데이터의 작은 변화에도 민감하게 반응해 발생하는 오류 |
| 불확실성 (Noise) | 데이터 자체가 가진 본질적인 변동성. 모델과 무관하게 항상 존재하는 오차 |

## 2. Bias-Variance Trade-off

| 구분 | 편향 | 분산 | 결과 |
|---|---|---|---|
| 단순한 모델 | 높음 | 낮음 | 언더피팅(Underfitting) — 학습·테스트 데이터 모두에서 성능 낮음 |
| 복잡한 모델 | 낮음 | 높음 | 오버피팅(Overfitting) — 학습 데이터 성능은 좋지만 테스트 데이터 성능 저하 |

모델의 복잡도를 조절하면서 편향과 분산 사이의 균형을 맞추는 것이 관건이다. 둘 다 낮은 모델이 이상적이지만 쉽지 않으며, 대표적인 해결 방법은 다음과 같다.

- 교차 검증(Cross Validation)
- 정규화(Regularization)
- 모델 튜닝

## 3. Regularization (정규화 = 규제)

정규화는 모델이 학습 데이터에 과도하게 맞춰지는 것(오버피팅)을 방지하고, 일반화 성능을 높이기 위한 기법이다. 손실 함수에 **규제항(Regularization term)**을 추가해, 가중치가 너무 커지거나 모델이 복잡해지지 않도록 제약을 건다.

```
L(W) = (1/N) × Σ Li(f(xi, W), yi)  +  λ × R(W)
        └────── Data loss ──────┘    └ Regularization ┘
        (예측이 학습 데이터와            (모델이 과도하게 맞춰지지
         얼마나 일치하는가)                않도록 하는 페널티)
```

- **λ(람다)**: 정규화 강도를 조절하는 하이퍼파라미터. λ가 크면 규제가 강해져 모델이 단순해지고, λ가 작으면 규제가 약해져 데이터에 더 민감하게 반응한다.

> 오컴의 면도날(Occam's Razor, 1285-1347): "여러 경쟁 가설들 중 가장 단순한 것이 최선이다." 학습 데이터에 너무 딱 맞는 복잡한 곡선(노이즈까지 학습)보다, 전체 경향을 잘 잡는 단순한 직선이 새로운 데이터에는 더 잘 맞을 수 있다는 것이 정규화의 철학적 배경이다.

## 4. L1 vs L2 정규화

정규화는 가중치 벡터가 너무 커지지 않도록 거리 개념(Norm)으로 제약을 거는 것이라고 볼 수 있다.

| 구분 | 계산 방식 | 특징 |
|---|---|---|
| L2 정규화 (Ridge) | 가중치의 제곱합: Σ W² | 값이 클수록 페널티가 크게 늘어나 특정 가중치가 튀는 것을 강하게 억제. 가중치를 0으로 만들진 않고 전체적으로 고르게 작아지게 함 |
| L1 정규화 (Lasso) | 가중치의 절댓값 합: Σ \|W\| | L2만큼 강하게 억제하진 않지만, 불필요한 가중치를 아예 **0으로 만들어** 특성을 제거하는 효과(변수 선택) |
| Elastic Net | L1 + L2 혼합 | 두 방식의 단점을 서로 보완 |

## 5. Standardization & Normalization (복습)

| 구분 | 의미 |
|---|---|
| Normalization (정규화) | 데이터 값의 범위를 0~1 등 공통 척도로 맞추는 것 (범위 조정) |
| Standardization (표준화) | 평균 0, 분산 1인 표준정규분포 형태로 데이터를 재조정하는 것 (분포 조정) |

## 6. 코드 예시 (Ridge / Lasso, 스케일링)

```python
from sklearn.preprocessing import StandardScaler, MinMaxScaler
from sklearn.linear_model import LogisticRegression

# 표준화: train에는 fit_transform, test에는 transform만 적용
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# 정규화(규제) 적용 - L1(Lasso) / L2(Ridge)
model_l1 = LogisticRegression(penalty='l1', solver='liblinear')
model_l2 = LogisticRegression(penalty='l2')
```

손글씨 데이터 분류 실습에서는 규제를 적용하지 않았을 때 정확도 97.78%, L1 적용 시 96.11%, L2 적용 시 97.78%가 나왔다. (데이터셋을 깊게 튜닝한 것이 아니라 수치 자체보다는 규제를 코드로 어떻게 적용하는지에 의미가 있다.) `StandardScaler`(표준화) 외에 `MinMaxScaler`로 값을 0~1 사이로 변환하는 정규화도 함께 사용할 수 있다.

## 💡 한 줄 요약
> 총 오차는 편향·분산·노이즈로 구성되며, 정규화(L1/L2)는 손실 함수에 규제항을 더해 모델이 학습 데이터에 과도하게 맞춰지지 않도록 함으로써 편향-분산 균형을 맞추는 기법이다.

## ❓ 더 찾아볼 것
- Elastic Net의 혼합 비율(l1_ratio) 조정 방법
- 교차 검증(K-Fold Cross Validation)의 구체적인 구현 방법
