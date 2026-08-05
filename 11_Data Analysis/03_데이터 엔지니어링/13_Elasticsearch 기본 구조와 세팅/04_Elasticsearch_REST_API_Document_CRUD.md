# [데이터 엔지니어링] Elasticsearch REST API & Document CRUD

---

## 1. Elasticsearch의 RESTful API

Elasticsearch는 완전한 **RESTful API** 기반으로 동작하기 때문에, HTTP 요청 방식(GET/POST/PUT/DELETE)만 잘 숙지하면 데이터를 쉽게 다룰 수 있다.

| 특징 | 설명 |
|---|---|
| HTTP 기반 동작 | 모든 요청·응답이 HTTP 프로토콜 위에서 이루어짐 |
| 자원을 URL로 표현 | `/users/1`, `/products/10` 처럼 자원의 위치를 URL 경로로 표현 |
| HTTP 메서드로 CRUD 수행 | GET(조회), POST(생성/수정), PUT, DELETE(삭제) |
| JSON 데이터 형식 사용 | 요청 본문(body)과 응답 모두 기본적으로 JSON 형식 |

예시:

```http
POST /products/_doc/1
Content-Type: application/json

{
  "name": "Samsung Galaxy S25 Ultra",
  "brand": "Samsung",
  "price": 1099,
  "category": "smartphone"
}
```

- `_doc`은 Document API를 사용할 때 쓰는 경로로, 문서를 저장·조회할 때 기본적으로 사용된다.

---

## 2. 인덱스(Index) 생성

인덱스는 여러 문서를 저장하는 **논리적 공간**이다. 생성 방식은 크게 두 가지다.

**① Dynamic Mapping (자동 매핑)**
문서를 바로 색인하면, 명시적으로 인덱스를 선언하지 않아도 Elasticsearch가 **필드 타입을 자동으로 추론**해 매핑을 설정해준다. 단, 처음 들어온 데이터를 기준으로 타입이 고정되기 때문에 이후 다른 타입의 값이 들어오면 문제가 될 수 있다.

```python
doc = {
    "name": "Samsung Galaxy S24 Ultra",
    "brand": "Samsung",
    "price": 1199.99,
    "category": "smartphone",
    "rating": 4.8
}
response = es.index(index="products", id=1001, document=doc)
```

**② Explicit Mapping (명시적 매핑)**
인덱스 설정(샤드 수, 레플리카 수, 필드 타입 등)을 미리 정의해 명시적으로 생성하는 방식. 안정적인 운영을 위해서는 이 방식이 권장된다(Mapping 상세는 다음 시간에 다룰 예정).

```python
es.indices.create(
    index="products",
    body={
        "settings": {
            "index": {
                "number_of_shards": 3,
                "number_of_replicas": 1
            }
        }
    }
)
```

---

## 3. Document CRUD

### Create — POST로 문서 생성

```http
POST /products/_doc/1001
{
  "name": "Samsung Galaxy S24 Ultra",
  "brand": "Samsung",
  "price": 1199.99,
  "category": "smartphone",
  "rating": 4.8
}
```

```python
# 클라이언트 생성
es = Elasticsearch("http://localhost:9200")

# 문서 삽입
doc = {
    "name": "Samsung Galaxy S24 Ultra",
    "brand": "Samsung",
    "price": 1199.99,
    "category": "smartphone",
    "rating": 4.8
}
response = es.index(index="products", id=1001, document=doc)
```

응답 예시:

```json
{
  "_index": "products",
  "_id": "1001",
  "_version": 1,
  "result": "created",
  "_shards": { "total": 2, "successful": 1, "failed": 0 },
  "_seq_no": 0,
  "_primary_term": 1
}
```

- `_version`: 문서 변경 이력을 나타내는 값. 문서가 변경될 때마다 1씩 증가
- `result`: `created`(신규 생성) / `updated`(수정) 등 처리 결과

### Read — GET으로 문서 조회

```http
GET /products/_doc/1001
```

```python
response = es.get(index="products", id=1001)
```

응답의 `found: true`는 문서를 정상적으로 찾았음을 의미하며, `_source`에 실제 저장된 문서 내용이 담겨 있다.

### Update — POST로 부분 업데이트

문서 전체를 다시 보내는 대신, `doc` 키를 사용해 **변경할 필드만** 전달한다.

```http
POST /products/_update/1001
{
  "doc": {
    "price": 1099
  }
}
```

```python
update_body = {"doc": {"price": 1099}}
response = es.update(index="products", id=1001, body=update_body)
```

- 실행 결과 `_version`이 1 → 2로 증가, `result: "updated"` 확인 가능

새로운 필드 추가도 동일한 방식으로 가능하다 (스키마리스 특성 확인):

```python
update_body = {"doc": {"stock": 200}}
response = es.update(index="products", id=1001, body=update_body)
```
→ 기존에 없던 `stock` 필드가 문서에 정상적으로 추가되며 `_version`이 3으로 증가한다.

### Upsert — 있으면 Update, 없으면 Insert

`doc_as_upsert: true` 옵션을 사용하면, 해당 ID의 문서가 **존재하면 업데이트**, **존재하지 않으면 새로 생성**한다.

```http
POST /products/_update/1001
{
  "doc": {
    "price": 1099,
    "stock": 150
  },
  "doc_as_upsert": true
}
```

```python
update_body = {
    "doc": {"price": 1099, "stock": 150},
    "doc_as_upsert": True
}
response = es.update(index="products", id=1001, body=update_body)
```

- 문서가 이미 있으면 `result: "updated"`
- 문서가 없으면(삭제된 이후 등) `result: "created"`

### Delete — DELETE로 문서 삭제

```http
DELETE /products/_doc/1001
```

```python
response = es.delete(index="products", id=1001)
print(response)
```

DELETE 요청 시 문서는 **즉시 물리적으로 제거되지 않고 삭제 표시(flag)만 된다.** 물리적 제거는 이후 Segment Merge 과정에서 이루어진다.

---

## 4. 문서 업데이트의 내부 동작 — 왜 세그먼트가 새로 생기는가

Elasticsearch의 문서는 **불변(Immutable)**이므로 직접 수정되지 않는다. `update`/`upsert`도 실제로는 기존 세그먼트 내부 값을 바꾸는 것이 아니라, **새로운 세그먼트를 생성하는 방식**으로 동작한다.

업데이트 처리 과정:
1. **기존 문서 조회** → 현재 색인된 문서를 가져옴
2. **변경 사항 적용** → 가져온 문서에 수정 내용을 반영
3. **새 문서 색인** → 변경된 문서를 새로운 버전으로 다시 저장
4. **이전 문서 삭제 처리** → 기존 문서는 논리적 삭제(Logical Deletion)로 표시되어 검색에서 제외
5. **세그먼트 병합** → Segment Merging을 통해 삭제된 문서를 물리적으로 제거하고 저장 공간 확보

### Refresh / Flush / Merge

| 개념 | 설명 |
|---|---|
| **Refresh** | 새로 색인된 데이터를 검색 가능하게 만드는 과정. 문서가 추가·수정되어도 즉시 검색 결과에 반영되지는 않으며, Refresh가 일어나야 새 세그먼트 형태로 검색 대상에 포함됨 |
| **Flush** | 세그먼트를 디스크에 기록해 안정적인 상태로 확정하는 과정. 커밋 포인트·트랜스로그를 생성해, 장애 발생 시 이 지점 기준으로 복구 가능 |
| **Merge** | 세그먼트가 많아지면 검색 시 확인 대상이 늘어나 성능 부담이 됨 → 여러 세그먼트를 하나의 큰 세그먼트로 주기적으로 병합해 검색 성능을 최적화. 삭제 플래그가 표시된 문서의 **물리적 제거**도 이 과정에서 수행됨 |

강제로 Flush를 수행하는 예시:

```python
response = es.indices.flush(index="products")
print(response)
```

---

## 5. Bulk API — 여러 문서를 한 번에 삽입

지금까지는 문서를 한 건씩 넣는 방식만 다뤘지만, 하나의 인덱스에 **여러 건의 데이터를 한 번에** 넣고 싶을 때는 **Bulk API**를 사용한다.

```python
from elasticsearch import helpers

# bulk_data: 여러 문서를 담은 리스트(제너레이터)
helpers.bulk(es, bulk_data)
```

- `elasticsearch` 파이썬 클라이언트 라이브러리의 `helpers` 모듈에서 `bulk` 함수를 제공
- 벌크로 삽입할 데이터와 Elasticsearch 클라이언트를 함께 전달하면 여러 문서가 한 번에 색인됨
- 내부적으로 요청 본문은 JSON이 아닌 **JSON Lines(NDJSON)** 형태로, 여러 개의 JSON 구조가 줄 단위로 이어져 전달되는 구조를 가진다

---

## 💡 한 줄 요약
> Elasticsearch는 HTTP + JSON 기반의 RESTful API로 Document CRUD를 수행하며, 내부적으로는 불변 세그먼트 구조 때문에 Update/Upsert도 결국 "새 세그먼트 생성 + 기존 문서 논리적 삭제 + Merge를 통한 물리적 제거"의 흐름으로 동작한다.

## ❓ 더 찾아볼 것
- Search Query DSL (match, term, bool 쿼리 등 대표적인 검색 쿼리)
- Mapping을 명시적으로 정의하는 방법과 필드 타입 종류
- Bulk API의 JSON Lines(NDJSON) 포맷 구조
- 트랜스로그(Translog)를 활용한 장애 복구 원리
