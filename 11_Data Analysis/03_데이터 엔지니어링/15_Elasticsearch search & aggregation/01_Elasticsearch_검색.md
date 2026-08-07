# [데이터 엔지니어링] Elasticsearch의 검색

---

## 1. 검색 요청 처리 흐름

- 클라이언트(애플리케이션)는 클러스터 내 **아무 노드**에나 검색 쿼리(Search Query)를 전송해도 된다. 요청을 받은 노드가 자동으로 **Coordinating Node** 역할을 수행한다.
- Coordinating Node는 쿼리를 해당 인덱스의 모든 관련 샤드(Primary 또는 Replica 상관없음)에 전달한다.
- 각 샤드가 **병렬로** 검색을 수행 → 결과를 Coordinating Node에 다시 전달한다. 샤드별로 병렬 수행되기 때문에 속도 면에서 이점이 있다.
- Coordinating Node는 여러 노드로부터 받은 결과를 **병합(merge) → 정렬(sort) → 후처리(필터링 등)**한다.
- 정렬 기준 상위 N개만 조회하거나 특정 필드만 반환해야 하는 등 상세 조회가 필요한 경우, 문서 상세 정보를 다시 요청하는 과정이 추가로 존재할 수 있다.
- 최종적으로 필요한 정보만 골라 클라이언트에 응답한다.

> 정리하면: **검색 요청을 받은 노드가 Coordinating Node 역할 → 샤드 단위 검색 수행 → 결과 병합·정렬 → 응답 전송**

---

## 2. URI 검색 (URI Query String Search)

- 간단한 검색을 수행할 때 사용하는 방식으로, URL 뒤에 검색 조건을 `"key=value"` 형태로 붙여서 요청한다.
- URL에 검색할 필드와 검색어를 지정하고, 검색 조건을 여러 개 추가로 붙일 수 있다.
- **Kibana의 Dev Tools**에서 간단히 데이터를 확인/테스트할 때 자주 사용하는 방식이다. 별도의 클라이언트 코드 없이 HTTP 메서드 기반으로 바로 요청·응답을 확인할 수 있다.
- 장점: URL만으로 빠르게 테스트 가능
- 한계: 여러 조건을 조합하거나 필터·정렬 등을 함께 사용해야 하는 경우 표현에 한계가 있음 → 이런 경우 **Query DSL**을 사용

```
GET /products/_search?q=brand:Samsung&default_operator=AND
```

### URI 검색 파라미터

| 파라미터 | 기본값 | 설명 |
|---|---|---|
| q | - | 검색을 수행할 쿼리 문자열 |
| df | - | 기본값으로 검색할 필드를 지정 |
| analyzer | - | 검색어에 적용할 형태소 분석기 지정 |
| analyze_wildcard | false | 접두어/와일드카드 검색 활성화 여부 |
| default_operator | OR | 여러 검색어에 대한 기본 연산자 설정 (AND 또는 OR) |

### URI 검색 결과 조정 파라미터

| 파라미터 | 기본값 | 설명 |
|---|---|---|
| _source | true | 본문 필드를 전체 표시할지 여부 |
| sort | - | 검색 결과 정렬 기준 지정 |
| from | - | 검색한 문서의 시작 문서 지정 (페이지네이션) |
| size | - | 검색 결과로 반환할 개수 지정 |

```
GET /products/_search?q=brand:Samsung&sort=price:asc&size=10&from=0
```

---

## 3. Query DSL

- Elasticsearch에서 검색을 수행하기 위한 **JSON 기반의 질의 언어(Domain Specific Language)**
- HTTP 요청 시 본문(Request Body)의 JSON 문서를 활용해 검색을 요청한다.
- 조건이 많아지거나 여러 쿼리를 조합하거나, 필터·정렬을 함께 사용해야 하는 경우 URI 검색으로는 한계가 있어 Query DSL을 사용한다.

### Query DSL 쿼리 형식

| 필드명 | 설명 |
|---|---|
| size | 반환할 문서 개수 (기본값: 10) |
| from | 검색 결과에서 몇 번째 문서부터 표시할지 설정 (기본값: 0) |
| timeout | 검색 수행 시간 제한 (기본값 없음, 필요 시 "30s" 등 설정 가능) |
| _source | 검색 결과에서 포함할 필드 지정 (기본값: 전체 포함) |
| query | 검색 조건이 들어가는 공간 |
| aggs | 통계 및 집계 데이터 설정 공간 |
| sort | 문서 정렬 기준 설정 |

```json
{
  "size": 10,
  "from": 0,
  "timeout": "30s",
  "_source": ["field1", "field2"],
  "query": { ... },
  "aggs": { ... },
  "sort": [ ... ]
}
```

### Query DSL 쿼리 검색 예시

```json
GET /products/_search
{
  "size": 5,
  "query": {
    "match": { "description": "무선 마우스" }
  }
}
```
→ description 필드에서 "무선 마우스"와 유사한 문서를 검색 (최대 5개 반환)

```json
GET /products/_search
{
  "_source": ["name", "price"],
  "query": { "match": { "category": "전자기기" } },
  "sort": [ { "price": "desc" } ]
}
```
→ category가 "전자기기"인 문서를 검색해 price 기준 내림차순 정렬, name과 price 필드만 출력

### Query DSL 쿼리 결과 구조

```json
{
  "took": "쿼리 실행에 소요된 시간(ms)",
  "_shards": {
    "total": "검색 대상이 된 전체 샤드 개수",
    "successful": "정상적으로 검색이 수행된 샤드 개수",
    "failed": "검색 중 오류가 발생한 샤드 개수"
  },
  "hits": {
    "total": "검색된 문서의 총 개수",
    "max_score": "가장 높은 검색 점수",
    "hits": "검색된 문서 목록"
  }
}
```

---

## 4. Query Context와 Filter Context

- **Query Context**: 검색어와 문서 간의 유사도 점수(`_score`)를 계산해 문서의 중요도를 평가. 관련성이 높다고 판단된 문서를 상위에 노출한다.
- **Filter Context**: 문서가 검색 조건에 해당하는지 여부만 판단(True/False)하고 `_score`는 계산하지 않는다.
  - 점수 계산이 없어 **캐싱이 가능**하므로, 반복되는 조건 검색에서 훨씬 효율적으로 동작한다.
  - 날짜 범위나 키워드 기반 정확한 조건 검색에 특히 유용하다.

> 두 컨텍스트의 핵심 차이는 결국 **"관련도 점수가 필요한가(Query) / 조건 일치 여부만 필요한가(Filter)"** 이다.

---

## 5. 검색 결과 정렬

- **기본 정렬 (Default Sorting)**: 검색 쿼리가 실행되면 `_score` 값(유사도 점수)에 따라 결과가 정렬된다. `_score`는 기본적으로 **BM25 알고리즘**을 기반으로 계산되며(similarity 설정으로 변경 가능), 값이 높을수록 상위에 노출된다.
- **특정 필드를 기준으로 정렬**: price(가격), name(상품명), created_at(등록일) 등 특정 필드를 기준으로 오름차순/내림차순 정렬이 가능하다.
- 일반적으로 텍스트 타입은 스코어링 기반 정렬이, 나머지 타입(keyword, date 등)은 특정 필드 기준 정렬이 자연스럽다. 쿼리 없이 정렬만 지정하는 것도 가능하다.

```json
GET products/_search
{
  "sort": [
    { "price": { "order": "desc" } }
  ]
}
```

---

## 6. 검색 결과 페이징

- `from`: 페이지를 가져올 때의 시작점
- `size`: 검색 결과를 가져올 양
- 다음 페이지로 넘어갈 때는 `from = 이전 from + size` 방식으로 계산해서 처리한다.

```json
GET products/_search
{
  "from": 0,
  "size": 5
}
```

---

## 7. 멀티 인덱스 검색 (멀티테넌시)

- 하나의 요청으로 **여러 인덱스를 동시에 검색**할 수 있는 방식 (예: `GET products,services/_search`)
- 스키마가 달라도 검색은 가능하지만, 보통 스키마가 유사한 인덱스끼리 사용하는 것이 일반적이다.
- **활용 예시**: 인덱스를 날짜별로 생성해서 사용하면(예: `products_0807`, `products_0808` …) 필요한 검색 범위만 최소화할 수 있다. 로그 데이터를 날짜 단위로 쌓는 형태에서 특히 유용하다.
- 다만 운영 관점에서는 검색 범위를 너무 넓게 잡으면 여러 샤드를 동시에 조회하게 되어 부담이 커질 수 있다. 운영 초반 Primary Shard 개수를 산정하기 애매한 경우, 인덱스를 나눠서 활용하는 방식이 대안이 될 수 있다.

```json
GET products,services/_search
{
  "_source": ["name", "category"],
  "query": { "match_all": {} },
  "sort": [ { "price": { "order": "desc" } } ]
}
```

---

## 8. Term-level Query

### Term Query
- 분석기(Analyzer)를 거치지 않은 **정확한 term 값**을 검색할 때 사용
- keyword, numeric, date, boolean 같은 구조화된 필드에 적합 (SQL의 `=` 조건과 유사)
- text 필드에는 보통 term query보다 match query를 사용
- Query Context에 넣으면 `_score`가 계산될 수 있지만, 정확한 조건 검색은 보통 Bool Query의 `filter` 절에서 사용

```json
GET products/_search
{
  "query": {
    "term": { "brand": "Samsung" }
  }
}
```

> 실습 확인: Term Query를 Query Context에서 그냥 사용하면, 매칭된 문서들의 `_score`가 모두 동일한 값(예: 0.89)으로 나온다. 이는 사실상 의미 없는 스코어이므로, 이럴 때는 **Bool Query의 filter 절 안에 Term Query를 넣어** 스코어 계산 없이 조건 일치 여부만 판단하도록 처리하는 것이 적절하다.

### Terms Query
- 특정 필드가 **여러 값 중 하나**와 정확히 일치하는 문서를 검색 (SQL의 `IN` 조건과 유사)
- 문자열뿐 아니라 숫자, 날짜, boolean 등 구조화 데이터에도 사용 가능
- 분석된 text 검색이 아니라 정확한 term 매칭에 가깝다
- 대량 조건 필터링은 filter context에서 사용하는 것이 일반적

```json
GET products/_search
{
  "query": {
    "terms": { "brand": ["Samsung", "Apple"] }
  }
}
```

### Range Query
- 숫자, 가격, 날짜 등을 **범위 조건**으로 검색할 때 사용
- 구조화 데이터 조건이므로 `bool.filter`에서 자주 사용

| 파라미터 | 설명 |
|---|---|
| gt | A보다 큼 (> A) |
| gte | A보다 크거나 같음 (>= A) |
| lt | A보다 작음 (< A) |
| lte | A보다 작거나 같음 (<= A) |

```json
GET products/_search
{
  "query": {
    "range": {
      "price": { "gte": 2000, "lte": 3000 }
    }
  }
}
```

---

## 9. Match 계열 쿼리 (Query Context)

### Match Query
- Elasticsearch에서 가장 기본적인 **전체 텍스트 검색(full-text search)** 방식
- 분석기(Analyzer)를 적용해 검색어를 토큰화·변환한 뒤 검색
- SQL로 표현하면 `LIKE '%검색어%'`와 비슷한 기능이지만, 역색인 기반으로 동작하기 때문에 훨씬 효율적
- 엄밀히는 analyzer 기반 토큰 검색이지, 임의의 substring을 찾는 LIKE 검색과는 동작 방식이 다름

```json
GET products/_search
{
  "query": {
    "match": { "description": "AI" }
  }
}
```

### Match Query with operator
- `operator`는 검색어가 여러 개일 때 AND 또는 OR 조건을 적용할지 결정 (기본값 OR)

```json
GET products/_search
{
  "query": {
    "match": {
      "name": { "query": "Samsung Ultra", "operator": "AND" }
    }
  }
}
```
→ "Samsung"과 "Ultra"가 (순서 상관없이) 둘 다 포함된 문서만 반환

### Match Phrase Query
- **단어 순서를 유지한 채** 검색을 수행
- 예: "Samsung Neo"로 검색 시 → "Samsung Neo QLED 8K 75-inch"는 매칭, "Samsung Odyssey Neo G9"는 단어 사이에 다른 단어가 끼어 있어 제외됨

```json
GET products/_search
{
  "query": {
    "match_phrase": { "name": "Samsung Neo" }
  }
}
```

### Match Phrase Prefix Query
- 단어 순서는 유지하되, **마지막 term을 prefix(접두어)로 처리**
- 예: "Samsung Ne"만 입력해도 "Samsung Neo…" 같은 문구가 검색됨 → **자동완성** 형태의 검색에 활용 가능
- 단, prefix 확장이 많아지면 성능 비용이 발생할 수 있음

```json
GET products/_search
{
  "query": {
    "match_phrase_prefix": { "name": "Samsung N" }
  }
}
```

---

## 10. Multi Match Query

- **여러 필드를 동시에 검색**할 때 사용하는 쿼리로, match와 다르게 여러 필드에서 일치하는 문서를 찾을 수 있다.
- `type`을 설정해 검색 방식을 조정할 수 있다.

| type | 설명 |
|---|---|
| **best_fields** (기본값) | 여러 필드에서 가장 높은 점수를 가진 필드만 반영. "Samsung"이 name 또는 description 중 하나라도 포함되면 반환되며, 점수는 가장 높은 필드를 기준으로 결정 |
| **most_fields** | 여러 필드에서 모든 일치 항목의 점수를 **합산**. 여러 필드에서 매칭될수록(예: name과 description 양쪽에 있을수록) 점수가 더 높아짐 |
| **cross_fields** | 여러 필드를 **하나의 가상 필드**처럼 보고 검색. 검색어의 각 term이 서로 다른 필드에 나뉘어 있어도 매칭 가능 (예: "Samsung"은 brand 필드, "Ultra"는 name 필드에 있어도 검색됨). 사람 이름, 주소처럼 여러 필드가 하나의 의미를 구성할 때 유용 |

```json
GET products/_search
{
  "query": {
    "multi_match": {
      "query": "Samsung",
      "fields": ["name", "description"],
      "type": "best_fields",
      "operator": "or"
    }
  }
}
```

---

## 11. Query String / Exist Query

### Query String
- **Lucene Query Syntax**를 사용해 복합적인 쿼리 구문을 분석하는 검색 방식
- AND, OR, 범위 검색, 와일드카드, 정규식 검색 등을 지원하며 SQL의 LIKE, IN, BETWEEN, REGEXP와 유사한 표현이 가능
- 문법이 엄격한 편이라 자주 사용되는 방식은 아님
- 사용자 입력을 그대로 넣으면 구문 오류나 의도치 않은 검색이 발생할 수 있음 → 사용자 검색창에는 `simple_query_string`을 고려하는 것이 안전

```json
GET products/_search
{
  "query": {
    "query_string": { "query": "name:Apple" }
  }
}
```

### Exist Query
- 필드가 **존재하는 문서만** 검색
- Elasticsearch는 필드가 존재하는 문서와 존재하지 않는 문서가 공존할 수 있음 (예: 어떤 문서에는 description 필드가 있고 어떤 문서에는 없을 수 있음)
- 데이터가 누락된 문서를 필터링하거나, 필드 유무 자체가 중요한 조건일 때 사용
- 존재 여부만 판단하므로 상대적으로 빠르게 동작

```json
GET products/_search
{
  "query": {
    "exists": { "field": "description" }
  }
}
```

---

## 12. Boolean Query

- `must` / `must_not` / `should` / `filter`를 조합하여 검색 조건을 구성

| 유형 | 설명 | 점수 계산 | 예제 |
|---|---|---|---|
| must | 모든 조건을 만족하는 문서 검색 (AND) | O | brand = Samsung AND description = AI |
| must_not | 특정 조건을 포함하지 않는 문서 검색 | X | brand != Google |
| should | 하나라도 조건을 만족하면 검색 (OR, 가산점) | O | brand = Samsung OR brand = Apple |
| filter | 조건 판단만 수행 (빠른 검색, 점수 미계산) | X | brand.keyword = Samsung AND price >= 100 |

- `must`는 반드시 만족해야 하는 조건이면서 **관련도 점수가 필요한 경우**(주로 텍스트 타입)에 사용
- `filter`는 반드시 만족해야 하지만 **점수 계산이 필요 없는 경우**(주로 keyword, range 같은 구조화된 조건)에 사용
- Range Query, Terms Query 등을 Bool Query 안에 조합하면 "브랜드가 Samsung 또는 Sony이면서 가격이 2000~3000" 같은 복합 조건을 스코어 계산 없이(filter) 처리할 수 있다.

```json
GET products_/_search
{
  "_source": ["name", "brand", "description"],
  "query": {
    "bool": {
      "must": [
        { "match": { "brand": "Samsung" } },
        { "match": { "description": "AI" } }
      ]
    }
  }
}
```

```json
GET products_/_search
{
  "_source": ["name", "brand", "description"],
  "query": {
    "bool": {
      "must_not": [
        { "match": { "brand": "Google" } }
      ]
    }
  }
}
```

---

## 13. Nested Query

- Elasticsearch는 기본적으로 문서를 **평탄화(flat)**하여 저장한다. 배열 형태의 JSON 객체(예: `features: [{feature_name, feature_value}, ...]`)를 일반 Object 타입으로 저장하면, 배열 내부 **서로 다른 원소의 필드 값끼리 잘못 매칭**되는 문제가 발생할 수 있다.
  - 예: 어떤 제품이 RAM 16GB, Storage 512GB를 가지고 있을 때, "feature_name=RAM AND feature_value=8GB"처럼 **다른 객체의 값끼리 섞여서** 검색될 수 있음
- 이런 관계를 정확히 유지하려면 **nested 타입**을 사용해야 한다. nested로 정의하면 각 객체가 Elasticsearch 내부에서 숨겨진 별도의 문서처럼 저장되어 **독립적으로 검색**할 수 있다.

### 매핑

```json
"mappings": {
  "properties": {
    "product_id": { "type": "integer" },
    "name": { "type": "text" },
    "brand": { "type": "keyword" },
    "features": {
      "type": "nested",
      "properties": {
        "feature_name": { "type": "keyword" },
        "feature_value": { "type": "keyword" }
      }
    }
  }
}
```

### Nested Query 사용

```json
GET products_/_search
{
  "query": {
    "nested": {
      "path": "features",
      "query": {
        "bool": {
          "must": [
            { "term": { "features.feature_name": "RAM" } },
            { "term": { "features.feature_value": "16GB" } }
          ]
        }
      }
    }
  }
}
```
→ nested 쿼리를 사용하면 **같은 객체 내에서만** feature_name과 feature_value가 일치하는지 확인하도록 검색된다.

### Nested Query vs Object Query
- Object Query(nested 미사용)로 `bool + term/match`를 통해 nested 필드 값을 검색하는 것 자체는 가능하지만, **하나의 객체 내부 관계를 보장하지 못한다.** 서로 다른 배열 요소의 값이 결합되어 매칭될 수 있음
- path를 지정하지 않고 일반 Bool Query만으로 nested 필드 값을 검색하면 애초에 **아무것도 검색되지 않는다** (nested 필드는 nested 쿼리로만 검색 가능)

> 정리: **관계를 유지하고 싶다면 nested를 사용**해야 하며, nested를 사용하면 배열 안의 객체가 하나의 쌍(pair)으로 묶여 검색되므로 명확하게 검색이 가능해진다.

---

## 💡 한 줄 요약
> Elasticsearch는 URI 검색과 JSON 기반 Query DSL로 검색을 수행하며, 점수 계산이 필요한 Query Context와 조건 판단만 하는 Filter Context를 구분해 Term/Match/Bool/Nested 등 다양한 쿼리를 목적에 맞게 조합하는 것이 핵심이다.

## ❓ 더 찾아볼 것
- BM25 유사도 알고리즘의 구체적인 계산 방식
- `query_string`과 `simple_query_string`의 차이
- 인덱스 Alias를 활용한 멀티 인덱스 관리 전략
- Kibana Dev Tools 활용법
