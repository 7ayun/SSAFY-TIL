# [데이터 엔지니어링] RDD

---

## 1. RDD란?
- RDD(Resilient Distributed Dataset)는 대용량 데이터를 분산 처리하고 분석하기 위한 Spark의 기본 데이터 처리 단위다.
- Resilient(탄력적인) + Distributed(분산된) + Dataset(데이터셋)의 합성어로, 실제 데이터들이 모여 있는 집합 구조를 의미한다.
- RDD 안에는 여러 개의 오브젝트(Object)·레코드(Record)·아이템(Item)이라 불리는 요소가 존재한다.
- 이 데이터는 파티션(Partition) 단위로 나누어 보관되며, 파티션 단위로 병렬 처리가 이루어진다. 파티션이 많을수록 더 많은 Executor가 동시에 데이터를 처리할 수 있다. 이런 구조 덕분에 Spark는 대규모 데이터를 빠르게 병렬 처리할 수 있다.

## 2. RDD의 특징

### (1) 데이터 추상화 (Data Abstraction)
- 데이터를 다룰 때 세부적인 저장 방식이나 물리적 위치를 몰라도 하나의 집합처럼 다룰 수 있게 해주는 추상화 계층을 제공한다.
- 파티션은 논리적인 단위이면서 동시에 물리적인 실행 단위이기도 하다. 데이터를 나누는 기준이 곧 하나의 Task로 매핑되어 실제로 Executor에서 병렬 실행되고, 저장할 때는 파일 단위로도 대응될 수 있다.
- 사용자는 셔플을 어떻게 덜 발생시킬지, 캐시를 어떻게 활용할지 정도의 레벨에서만 신경 쓰면 되고, 어떤 데이터가 몇 번 파티션에 있는지 같은 물리적 세부사항은 신경 쓸 필요가 없다.

### (2) 탄력성(Resilient) & 불변성(Immutable)
- RDD는 한 번 생성되면 변경할 수 없다(immutable). "변경"하는 게 아니라, Transformation을 적용해 새로운 RDD를 계속 쌓아가는 방식으로 동작한다.
- 예를 들어 어떤 RDD가 있을 때 이를 Transformation하면 원본은 그대로 두고 새로운 RDD가 만들어진다.
- 어떤 노드(서버)가 장애로 중단되어 특정 파티션의 데이터가 손실되더라도, Lineage(계보) — 즉 원본 데이터와 그동안의 실행 계획 — 를 기반으로 해당 파티션만 다시 계산해서 복구할 수 있다.

### (3) 타입 안정성 (파이썬에서는 예외)
- RDD는 하나의 타입을 가지는 객체만 담을 수 있으며, 모든 요소가 동일한 자료형을 가진다.
- 이 타입은 컴파일 시점에 검사되며, 이를 통해 성능을 최적화하고 코드의 가독성과 유지보수성을 높일 수 있다.
- 다만 PySpark를 사용할 때는 Python이 동적 타입 언어이기 때문에 이러한 타입 안정성이 실제로 적용되지는 않는다는 점을 참고해야 한다.

### (4) 정형(구조화된) & 비정형 데이터를 모두 처리 가능
- 비정형(구조화되지 않은) 데이터, 즉 고정된 포맷이 없는 텍스트 데이터는 `sc.textFile()` 등으로 RDD로 로딩한 뒤 `map`, `filter`, `flatMap` 등으로 가공한다.
- 정형(구조화된) 데이터, 즉 컬럼과 스키마가 있는 테이블 형태의 데이터는 보통 DataFrame이나 `RDD.map()`으로 가공한다.
- 실무에서는 구조가 없는 데이터는 텍스트로 읽어 RDD로 가공하고, 구조가 있는 데이터는 DataFrame으로 직접 다루는 것이 더 편하기 때문에, RDD를 직접 사용하는 경우는 상대적으로 드물다.

### (5) 지연 평가 (Lazy Evaluation)
- RDD에서도 계보(Lineage)가 만들어지고, 이를 기반으로 실행 계획을 최적화할 수 있는 여지를 확보한 뒤 실제 연산을 수행한다.
- 불필요한 연산을 방지해 리소스를 절약할 수 있다는 이점이 있다.

## 3. map() Transformation 동작 방식
- 하나의 RDD `x`에 3개의 아이템이 있다고 할 때, 각 아이템이 어떤 파티션에 속해 있는지는 지금 단계에서 신경 쓰지 않아도 된다.
- 각 아이템에 특정 함수를 적용해서 변환시키고 싶을 때 `map()`을 사용하면, 함수가 각 아이템(오브젝트, 로우, 레코드)에 하나씩 적용되어 새로운 값으로 바뀐 결과가 들어간다.
- `map()`은 원본 RDD는 그대로 두고 새로운 RDD를 반환하는 Transformation이다. 즉, 하나·둘·셋 각각의 레코드에 대해 처리를 해주고, 처리 전(before)과 후(after)로 만들어지는 것이 새로운 RDD(New RDD)다. 이는 RDD가 불변성(immutable)을 지키는 방식으로 동작하기 때문이다.

```python
x = sc.parallelize(['b', 'a', 'c'])
y = x.map(lambda z: (z, 1))

print(x.collect())  # ['b', 'a', 'c']
print(y.collect())  # [('b', 1), ('a', 1), ('c', 1)]
```

- 위 예시에서 `map()`은 파티션 간 재분배가 딱히 이루어지지 않으므로 **Narrow Transformation**에 해당한다.

## 4. 전체 코드 흐름 정리
```python
from pyspark import SparkContext

sc = SparkContext("local", "LazyEvalExample")

# 1. 텍스트 데이터를 RDD로 변환 → Transformation
rdd = sc.parallelize(["apple", "banana", "spark", "data"])

# 2. 대문자로 바꾸기 → Transformation
upper_rdd = rdd.map(lambda x: x.upper())

# 3. 'SPARK'가 포함된 문자열만 남기기 → Transformation
filtered_rdd = upper_rdd.filter(lambda x: "SPARK" in x)

# 4. 결과 확인 (Action) — 여기서 실제로 실행됨!
result = filtered_rdd.collect()
```
- SparkContext(또는 SparkSession에서 뽑아낸 컨텍스트)를 먼저 선언한다.
- 메모리에 있는 리스트(파이썬 객체)를 `parallelize()`로 RDD 형태로 변환한다.
- 이 RDD를 기반으로 `map()`, `filter()` 같은 Transformation을 연달아 선언해도, 이 시점까지는 아무것도 실제로 실행되지 않는다. 이것이 앞서 다룬 Lazy Evaluation(지연 평가)이다.
- `collect()` 같은 Action 연산을 호출하는 순간, 그동안 쌓아둔 모든 변환 계획이 한 번에 실행되어 최종 결과가 반환된다. 이것이 스파크가 처리되는 전반적인 흐름이다.

## 💡 한 줄 요약
> RDD는 파티션 단위로 분산 저장·병렬 처리되는 Spark의 기본 데이터 단위로, 데이터 추상화·불변성과 Lineage 기반 복구·지연 평가라는 특징을 통해 안정적이고 최적화된 분산 연산을 가능하게 한다.

## ❓ 더 찾아볼 것
- RDD와 DataFrame의 API 및 성능 차이
- `persist()` / `cache()`를 활용한 RDD 재사용 전략
- `map()` 외에 `flatMap()`, `reduceByKey()` 등 다른 Transformation 함수들의 동작 방식
