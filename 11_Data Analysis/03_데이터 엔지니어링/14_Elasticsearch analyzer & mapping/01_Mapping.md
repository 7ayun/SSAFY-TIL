# [데이터 엔지니어링] Mapping

---

## 1. Mapping이란?

Mapping은 관계형 데이터베이스의 **스키마(Schema)** 와 유사한 개념으로, Elasticsearch에서 문서(Document)가 어떤 필드 구조와 데이터 타입을 가지는지 정의하는 설정이다.

예를 들어 상품 데이터에 `name`(문자열), `price`(숫자) 필드가 있다면, 이 필드들이 각각 어떤 타입으로 저장·검색될지를 Mapping에서 결정한다. 완전히 동일한 개념은 아니지만, "어떤 필드가 존재하고 그 타입이 무엇인가"를 미리 정의해준다는 점에서 스키마와 비슷한 역할을 한다.

Mapping은 크게 **동적 매핑(Dynamic Mapping)** 과 **정적 매핑(Explicit Mapping)** 두 가지 방식으로 나뉜다.

---

## 2. 동적 매핑 vs 정적 매핑

### 동적 매핑 (Dynamic Mapping)

- 필드 타입을 미리 지정하지 않아도, 문서를 색인(indexing)할 때 Elasticsearch가 값을 보고 **자동으로 타입을 추론**해 매핑을 생성한다.
- 인덱스가 없는 상태에서 바로 데이터를 저장(색인)해도, 인덱스와 매핑이 자연스럽게 함께 생성된다.
- **장점**: 별도 설계 없이 편리하게 사용 가능
- **단점**: 처음 들어온 값에 따라 타입이 결정되므로, 이후 다른 형태의 값이 들어오면 **타입 충돌**이 발생할 수 있다.
  - 예: `price`에 `5000`이 먼저 들어와 정수(integer)로 추론됐는데, 이후 `5000.5` 같은 float 값이 들어오면 충돌 발생
  - 날짜 데이터가 문자열로 잘못 인식되는 경우도 있음
- 그래서 운영 환경에서는 이런 오류 가능성을 인지하고 있어야 하며, 보통은 테스트나 간단한 확인 용도로만 사용한다.

### 정적 매핑 (Explicit Mapping)

- 인덱스를 생성하는 시점에 필드 이름과 타입을 **사전에 명확하게 정의**하는 방식.
- 데이터 구조를 안정적으로 관리할 수 있고, 검색·집계 시 성능 예측이 쉬워진다.
- 단점은 사전 설계가 필요해 초기 구성이 다소 복잡하다는 것.
- **한 번 설정된 매핑은 이후 변경이 제한**되는 것이 일반적이다. 특히 절대 바뀌면 안 되는 필드는 반드시 정적 매핑으로 명확히 생성해야 한다. 타입을 바꿔야 한다면 보통 새로운 인덱스를 만들어 **재색인(reindexing)** 하는 방식을 사용한다.
- 운영 환경에서는 안정성과 성능 면에서 유리한 정적 매핑을 사용하는 것이 일반적이다.

---

## 3. Dynamic Parameter (dynamic 설정값)

동적 매핑을 얼마나 엄격하게 적용할지 `dynamic` 파라미터로 조정할 수 있다.

```python
es = Elasticsearch("http://localhost:9200")

es.indices.create(index="products", body={"mappings": {"dynamic": "runtime"}})
```

| 설정값 | 동작 |
|---|---|
| `true` (기본값) | 새로운 필드가 자동으로 추가됨 |
| `runtime` | 새로운 필드를 미리 색인하지 않고, **검색(런타임) 시점에만** 동적으로 처리 |
| `false` | 새로운 필드는 무시됨 (저장은 안 되지만 오류도 안 남) |
| `strict` | 새로운 필드 추가 시 오류 발생 (아예 넣을 수 없음) |

- `runtime`은 저장 시점의 색인·디스크 사용량은 줄일 수 있지만, 검색 시점에 값을 읽어 처리해야 하므로 그만큼 처리 비용이 발생한다. 보조 필드처럼 가끔 쓰는 필드에 적합하다.

---

## 4. 정적 매핑(Explicit Mapping) 예시

인덱스 생성 시 `mappings.properties` 하위에 필드명과 타입을 key-value 형태로 선언한다.

```python
# 클라이언트 연결
es = Elasticsearch("http://localhost:9200")

# 인덱스 생성
es.indices.create(
    index="products",
    body={
        "mappings": {
            "properties": {
                "name": {"type": "text"},
                "brand": {"type": "keyword"},
                "price": {"type": "float"},
                "category": {"type": "keyword"},
                "rating": {"type": "float"}
            }
        }
    }
)
```

`name`처럼 단어 단위 검색이 필요한 필드는 `text`로, `brand`처럼 정확히 일치하는 값으로만 다뤄질 필드는 `keyword`로 지정한다. 주요 필드는 인덱스 생성 시점에 명확히 매핑해두는 것이 좋다.

---

## 5. Elasticsearch 필드 데이터 타입

필드 타입은 검색 방식과 성능을 결정하므로 잘 이해해두는 것이 중요하다.

- **지형 데이터 타입 (Geo Data Types)**: `geo_point`(위도·경도 좌표), `geo_shape`(점·선·다각형 등 지리적 영역) — 반경 검색, 지도 기반 검색 등에 사용
- **계층 구조 데이터 타입 (Hierarchical Data Types)**
  - `object`: 일반적인 JSON 객체 구조 표현 (예: 상품 안의 제조사 정보)
  - `nested`: 배열 안의 각 객체를 독립적으로 구분해서 관리해야 할 때 사용 (예: 상품의 여러 리뷰 — 리뷰마다 작성자·평점·내용이 서로 섞이지 않도록 구분)
- **일반 데이터 타입**: 문자열(`keyword`, `text`), `date`, `long`, `double`, `integer`, `boolean` 등

이 중에서도 실무에서 가장 자주 다루는 것이 문자열 타입인 **keyword**와 **text**다.

---

## 6. 문자열 타입 ① keyword

- 값을 **변형 없이 그대로 저장**하며, 분석기(Analyzer)를 적용하지 않는다.
- 공백과 대소문자까지 구분해서 **정확히 일치하는지(exact matching)** 만 비교한다.
- 집계(Aggregation), 정렬(Sort), 필터링(Filtering)에 적합하다.
- 사용 예: 태그(tags), 카테고리(category), 브랜드명(brand), 사용자 ID(user_id), 로그 수준(log_level), 국가 코드(country_code)

```json
{
  "mappings": {
    "properties": {
      "name": { "type": "keyword" }
    }
  }
}
```

### constant_keyword

- 인덱스 내 **모든 문서가 동일한 값**을 가지는 필드에 사용 (예: 특정 인덱스의 브랜드 값이 항상 "Samsung"인 경우)
- 색인 크기를 줄이고 필터링 성능을 최적화하는 데 적합
- 한 번 지정하면 **수정 불가능**

```python
es.indices.create(
    index="products",
    body={
        "mappings": {
            "properties": {
                "brand": {
                    "type": "constant_keyword",
                    "value": "Samsung"
                }
            }
        }
    }
)
```

### wildcard

- 정확한 값 일치가 아니라, 문자열 안의 **특정 패턴을 검색**할 때 사용 (`*`, `?` 같은 와일드카드 문자, 정규식(regexp) 쿼리 활용)
- 로그 메시지처럼 구조화되지 않은 긴 문자열에서 패턴 검색이 필요할 때 사용
- 일반적인 정확 일치·정렬·집계 목적이라면 keyword가 더 적합
- 참고: 자동완성처럼 앞부분만 입력해도 결과를 찾는 기능도 이론상 wildcard로 가능하지만, 이런 prefix 검색에는 보통 **n-gram** 방식(토크나이저의 일종)이 더 적합하다.

```json
{
  "mappings": {
    "properties": {
      "name": { "type": "wildcard" }
    }
  }
}
```

---

## 7. 문자열 타입 ② text

- 입력된 텍스트를 **분석기(Analyzer)를 통해 토큰(token) 단위로 분리**하여 색인·검색한다.
- 정확히 같은 문자열을 찾는 것이 아니라, 검색어와 문서 내용이 **얼마나 연관성이 있는지**를 기준으로 검색된다 (BM25, TF-IDF 같은 통계적 방식으로 관련도 점수를 계산).
- 공백, 대소문자, 형태소 분석 등 다양한 전처리가 가능하다.
- 텍스트 타입으로 저장하면 토큰화 및 역인덱스(inverted index) 구축 과정이 추가로 필요해 keyword보다 저장 공간과 색인 비용이 더 든다. 전문 검색이 필요 없는 필드라면 keyword를 쓰는 것이 공간·성능 면에서 유리하다.

```json
{
  "mappings": {
    "properties": {
      "name": { "type": "text" }
    }
  }
}
```

### match_only_text

- 로그 분석 등에 주로 사용되는, keyword와 text의 **중간 단계** 타입
- 전체 텍스트 쿼리는 실행하지만 관련도 **점수 계산을 하지 않는다** (단순히 단어 포함 여부만 확인)
- 정렬·집계가 필요 없고, 단어의 포함 여부만 빠르게 확인하면 되는 경우에 적합

```json
{
  "mappings": {
    "properties": {
      "name": { "type": "match_only_text" }
    }
  }
}
```

### search_as_you_type

- **자동완성(Autocomplete)** 검색을 지원하는 타입
- 입력 중인 검색어에 대해 prefix(앞부분) 기반 검색을 쉽게 구성할 수 있도록, 내부적으로 shingle 및 prefix용 서브필드를 활용
- 검색어가 점진적으로 확장되는 형태의 검색에 적합하며, 중간 문자열 임의 검색이 필요하다면 n-gram 등 별도 설계를 고려해야 한다.

```json
{
  "mappings": {
    "properties": {
      "name": { "type": "search_as_you_type" }
    }
  }
}
```

---

## 8. 하나의 필드에 여러 타입 동시 선언

하나의 필드에 대해 text와 keyword를 동시에 선언해 두 가지 방식(전문 검색 + 정확 일치 검색)을 모두 활용하는 것도 가능하다. (예: `field.text`, `field.keyword`)

---

## 💡 한 줄 요약
> Mapping은 문서의 필드 구조와 타입을 정의하는 설정으로, 자동 추론에 의존하는 동적 매핑보다는 필드 특성(전문 검색 여부, 정렬·집계 필요 여부)에 맞춰 keyword·text 등 타입을 명확히 지정하는 정적 매핑이 안정성과 성능 면에서 유리하다.

## ❓ 더 찾아볼 것
- Elasticsearch reindexing(재색인) 절차와 downtime 없이 진행하는 방법
- nested 타입과 object 타입의 쿼리 방식 차이 (`nested` 쿼리)
- BM25, TF-IDF의 관련도 점수 계산 방식
