# [데이터 기초] Pandas

---

## 1. Pandas란?

Pandas는 파이썬 기반의 데이터 분석 라이브러리로, 대용량보다는 **소~중규모의 정형 데이터**(행과 열로 구성된 표 형태 데이터)를 다루는 데 최적화되어 있다.

- **NumPy 기반**으로 만들어져 있어, 각 컬럼이 동일한 타입을 가질 때 브로드캐스트 연산을 통해 대량의 데이터(수만~수십만 건)를 효율적으로 처리할 수 있다.
- `describe()`, `info()` 처럼 함수 하나로 통계 정보나 데이터 특성을 바로 확인할 수 있는 **고수준 API**를 제공한다.

## 2. Series와 DataFrame

Pandas가 다루는 정형 데이터 구조는 크게 두 가지로 나뉜다.

| 구분 | Series | DataFrame |
|---|---|---|
| 차원 | 1차원 | 2차원 |
| 형태 | 컬럼 하나짜리 데이터 목록 | 여러 컬럼이 모인 표 (행 × 열) |
| 타입 | 모든 값이 동일한 타입 | 컬럼마다 다른 타입 가능 |

두 구조 모두 값에 접근할 수 있는 **인덱스**가 함께 붙으며, 따로 지정하지 않으면 0부터 자동으로 생성된다.

```python
import pandas as pd

# Series 생성
ages = pd.Series(data=[22, 35, 58], name="Age")
print(ages)
# 0    22
# 1    35
# 2    58
# Name: Age, dtype: int64

# DataFrame 생성 (딕셔너리 형태)
df = pd.DataFrame({
    "Name": ["Braund, Mr. Owen Harris", "Allen, Mr. William Henry", "Bonnell, Miss. Elizabeth"],
    "Age": [22, 35, 58],
    "Sex": ["male", "male", "female"],
})
```

> **단일 컬럼 선택 시 주의할 점** — 대괄호 안에 컬럼명을 문자열로만 넣으면 Series가, 리스트로 감싸서 넣으면 DataFrame이 반환된다.
> ```python
> df['Age']     # Series 반환
> df[['Age']]   # DataFrame 반환 (리스트로 감쌈)
> ```

## 3. Pandas를 쓰는 이유 — 고수준 API가 제공하는 기능들

| 기능 | 설명 |
|---|---|
| 결측치 처리 | 비어있는 값을 쉽게 채우거나 제거 |
| 행/열 조작 | 추가·삭제가 간단함 |
| 정렬 | 원하는 컬럼 기준으로 손쉽게 정렬 |
| 집계 (groupby) | 특정 컬럼 기준으로 그룹을 만들어 평균·합계 등 계산 |
| 자료구조 변환 | list, dict, numpy 배열 등으로 자유롭게 변환 |
| 병합 (merge/join) | 여러 데이터프레임을 결합 |
| 파일 입출력 | CSV, Excel, DB, JSON 등 다양한 포맷을 읽고 쓸 수 있음 |

---

## 💡 한 줄 요약
> Pandas는 NumPy 기반의 정형 데이터 처리 라이브러리이며, 1차원 Series와 2차원 DataFrame이라는 두 자료구조를 통해 결측치 처리·정렬·집계·병합 등을 고수준 API로 간편하게 수행할 수 있다.

## ❓ 더 찾아볼 것
- NumPy의 브로드캐스팅(broadcasting) 개념과 동작 원리
- Series에 커스텀 인덱스를 직접 지정하는 방법
- 정형 데이터와 비정형 데이터(텍스트, 이미지 등)의 차이
