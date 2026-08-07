# [데이터 엔지니어링] Aggregation

---

## 1. Aggregation 개요

- Elasticsearch에서 데이터를 분석하고 통계를 계산하는 기능
- 분산된 문서에서 검색 조건에 맞는 데이터만 모아 통계를 수행
- SQL의 `GROUP BY`, `COUNT`, `SUM`, `AVG`, 혹은 Pandas의 `describe()`가 하는 역할과 유사 — 데이터를 다룰 때 대부분 이런 집계 기능이 포함되어 있다고 볼 수 있음
- 크게 두 가지로 나눌 수 있음
  - **Metric Aggregations (메트릭 집계)**: 개별 필드의 통계 값을 계산. sum, average, min, max, stats 등의 연산 수행
  - **Bucket Aggregations (버킷 집계)**: 특정 조건에 따라 데이터를 그룹화. SQL의 GROUP BY와 유사
  - 이 외에도 다양한 집계 유형이 존재하지만, 이번 학습에서는 이 두 가지를 중심으로 다룸

---

## 2. Metric Aggregations (메트릭 집계)

- 개별 필드의 통계 값을 계산 (sum, average, min, max, stats 등)
- 쿼리 바디에서 `aggs`(aggregations) 필드에 집계 이름(임의 네이밍)과 연산을 선언
- 예시: 제품 가격의 평균 계산
- `size: 0`으로 설정하면 **검색된 문서 자체는 반환하지 않고 집계 결과만 조회**한다. 문서가 많을 때 불필요한 응답 크기를 줄이기 위한 설정

```json
GET products/_search
{
  "size": 0,
  "aggs": {
    "average_price": {
      "avg": { "field": "price" }
    }
  }
}
```

응답 예시:
```json
{
  "hits": { "total": { "value": 10, "relation": "eq" }, "hits": [] },
  "aggregations": {
    "average_price": { "value": 1409.9900177001953 }
  }
}
```

> **주의**: 여기서 `size`는 이 aggregation이 계산할 값의 개수를 조정하는 파라미터가 아니라, **쿼리 결과로 반환할 문서 개수**를 조정하는 값이다. 즉 `size`를 바꿔도 aggregation 결과 값(예: 평균 가격) 자체는 영향을 받지 않는다. (`size: 0` → 문서 자체는 반환 안 함 / `hits.total`은 검색된 문서 총 개수를 그대로 보여줌)

---

## 3. Bucket Aggregations (버킷 집계)

- 특정 조건에 따라 데이터를 그룹화 (SQL의 GROUP BY와 유사)
- 예시: 브랜드별 제품 개수 집계
- `terms`가 `aggs`에 들어가면 필드의 **고유값을 기준으로 그룹(버킷)**을 나눈다.
- 집계 이름(예: `by_brand`)은 Aggregation 결과를 식별하기 위한 임의의 네이밍일 뿐, **필드명이 아니다.**
- 여기서의 `size`는 **반환할 버킷(그룹)의 개수**를 의미한다. 예를 들어 브랜드 종류가 매우 많을 때 상위 몇 개의 브랜드 그룹만 받아올지를 정하는 값이며, Metric Aggregation에서의 `size`(쿼리 결과 문서 개수 제한)와는 **동작 방식이 다르므로 혼동하지 않아야 한다.**

```json
GET products/_search
{
  "size": 0,
  "aggs": {
    "by_brand": {
      "terms": { "field": "brand", "size": 10 }
    }
  }
}
```

응답 예시:
```json
{
  "aggregations": {
    "by_brand": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        { "key": "Samsung", "doc_count": 8 },
        { "key": "Apple", "doc_count": 1 },
        { "key": "Sony", "doc_count": 1 }
      ]
    }
  }
}
```

### Bucket Aggregations 유형

| 유형 | 설명 |
|---|---|
| terms | 특정 필드 값으로 그룹화 (GROUP BY 역할) |
| range | 숫자 범위별로 그룹화 (예: 가격 구간별) |
| histogram | 고정 간격으로 그룹화 (예: 가격을 100 단위로 그룹화) — histogram 자체가 연속적인 수를 일정 구간으로 나눠 다루는 것과 같은 개념 |
| date_histogram | 날짜 단위로 그룹화 (예: 월별 판매량) |

---

## 4. 집계와 필터(쿼리) 결합

- 검색 결과에 대해 집계를 수행하는 것도 가능하고, 집계만 별도로 수행하는 것도 가능하다.
- 즉, 특정 조건으로 먼저 **필터링(query)**한 뒤, 그 결과 문서들에 대해서만 **집계(aggs)**를 수행할 수 있다.
- 예: 브랜드 필드가 "Samsung"인 문서만 미리 필터링한 뒤, 그 문서들의 price 필드에 대한 `stats`(요약 통계: count, min, max, avg, sum)를 계산

```json
GET products/_search
{
  "size": 0,
  "query": {
    "term": { "brand": "Samsung" }
  },
  "aggs": {
    "price_stats": {
      "stats": { "field": "price" }
    }
  }
}
```

응답 예시:
```json
{
  "hits": { "total": { "value": 8, "relation": "eq" }, "hits": [] },
  "aggregations": {
    "price_stats": {
      "count": 8,
      "min": 149.99000549316406,
      "max": 4999.990234375,
      "avg": 1537.4900245666504,
      "sum": 12299.920196533203
    }
  }
}
```

> `stats` 집계는 Pandas의 `describe()`처럼 count, min, max, avg, sum 등의 요약 통계를 한 번에 계산해준다. 위 예시처럼 `query`로 먼저 조건을 걸어 검색 범위를 좁힌 뒤, 그 결과를 기준으로 aggregation을 수행하는 방식이 자주 사용된다.

---

## 💡 한 줄 요약
> Aggregation은 Metric(개별 필드 통계 계산)과 Bucket(조건별 그룹화) 집계를 조합해 검색 결과를 요약·분석하는 기능이며, `size` 파라미터가 Metric에서는 반환 문서 개수를, Bucket에서는 반환 버킷 개수를 의미한다는 점을 구분해서 이해해야 한다.

## ❓ 더 찾아볼 것
- Sub Aggregation(중첩 집계)을 활용한 다단계 그룹 분석 방법
- `date_histogram`의 interval / calendar_interval 설정
- Bucket Aggregation의 `size`와 `shard_size` 차이, 그리고 `doc_count_error_upper_bound`가 의미하는 정확도 이슈
