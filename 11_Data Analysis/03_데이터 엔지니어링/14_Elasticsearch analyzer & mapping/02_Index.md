# [데이터 엔지니어링] Index

---

## 1. Index란?

Elasticsearch에서 Index는 데이터를 저장하는 공간이며, 저장의 목적은 결국 **검색을 빠르게 하기 위함**이다. 단순 저장 도구가 많음에도 Elasticsearch를 사용하는 이유가 바로 여기에 있다.

Index는 RDB의 테이블·데이터베이스와 비슷해 보이지만 완전히 동일하지는 않다. 하나의 인덱스(예: `products`) 안에 각 문서(document)들이 저장되며, 여러 인덱스를 함께 묶어 한 번에 검색하는 것도 가능하다 (예: `products1`, `products2` 동시 검색).

---

## 2. Forward Index vs Inverted Index

문서를 어떤 구조로 색인해야 검색어가 들어왔을 때 빠르게 문서를 찾아낼 수 있는지가 중요하다.

### Forward Index (정방향 인덱스)

- **문서 중심**으로 인덱스를 구축 — 각 문서가 어떤 단어들을 포함하는지 목록을 저장
- 인덱스 구축은 단순하지만, 특정 단어를 포함한 문서를 찾으려면 모든 문서를 순회해야 하므로 **검색 속도가 느림**
- 특정 문서 하나의 내용을 확인할 때는 유용함

```text
문서 1 → ["Elasticsearch", "검색", "엔진"]
문서 2 → ["Kibana", "시각화", "도구"]
```

### Inverted Index (역방향 인덱스)

- **단어 중심**으로 인덱스를 구축 — 각 단어가 어떤 문서들에 포함되어 있는지 목록을 저장
- Elasticsearch가 기본적으로 사용하는 방식이며, 특정 단어를 검색했을 때 해당 단어를 포함한 문서 목록을 바로 읽어올 수 있어 **검색 속도가 훨씬 빠름**
- 텍스트(text) 타입 기반의 전문 검색(Full-Text Search) 처리에 이 구조가 사용됨

```text
"검색" → [문서 1]
"엔진" → [문서 1]
"Kibana" → [문서 2]
```

---

## 3. text 필드의 정렬/집계를 위한 fielddata

Inverted Index는 검색에는 유리하지만, **정렬이나 집계**를 하기엔 적합하지 않다. keyword 타입이 아닌 text 필드에서 정렬·집계를 쓰고 싶다면 `fielddata`를 활용할 수 있다.

- text 필드를 정렬·집계(aggregation)에 사용할 때 값을 **메모리에 로드**하는 방식
- 다만 메모리 사용량이 커질 수 있어서, 실무에서는 fielddata 대신 **keyword 타입 서브필드를 추가로 만드는 것을 권장** (예: `field.text`, `field.keyword`)

```python
# 인덱스 생성
es.indices.put_mapping(
    index="my_index",
    body={
        "properties": {
            "my_field": {
                "type": "text",
                "fielddata": True
            }
        }
    }
)
```

### doc_values (권장 방식)

fielddata의 메모리 부담을 보완하기 위한 방식으로, 필드 값을 **디스크 기반의 컬럼(Column) 저장 방식**으로 저장한다.

- 기본적으로 비(非) 텍스트 필드(keyword, 숫자, 날짜 등)에서 활성화됨
- text 타입이어도 keyword 서브필드를 만들어 doc_values를 활용 가능

```python
es.indices.put_mapping(
    index="my_index",
    body={
        "properties": {
            "user_name": {
                "type": "text",
                "fields": {
                    "keyword": {
                        "type": "keyword",
                        "ignore_above": 256
                    }
                }
            }
        }
    }
)
```

- `ignore_above: 256`처럼 설정하면 256자 이상인 값은 keyword 서브필드에 색인되지 않도록 해서, 불필요한 리소스 낭비를 줄일 수 있다.

---

## 4. 인덱스 엘리아스 (Index Alias)

- 인덱스 명을 대신하는 **가상의 이름(별칭)** 을 부여할 수 있는 기능
- 여러 개의 인덱스를 하나의 인덱스처럼 연결해서 동시에 검색 대상으로 사용 가능 (예: `products1`, `products2`, `products3`를 하나의 alias로 묶어 한 번에 검색)
- `_alias` API를 사용하여 설정

```python
es.indices.create(
    index="products",
    body={
        "aliases": {
            "products_alias": {}
        }
    }
)
```

Alias는 실제 인덱스 이름을 감싸는 **가상의 대상**으로, 애플리케이션과 인덱스 사이에 하나의 추상화 계층을 넣어주는 역할을 한다. 클라이언트 코드는 항상 alias만 바라보면 되므로, 내부적으로 인덱스가 교체되어도 클라이언트 코드를 변경할 필요가 없다.

---

## 5. 무중단 인덱스 변경 (Zero-downtime Reindexing)

`products_alias`가 기존 `products` 인덱스를 가리키고 있다고 가정할 때, 새로운 인덱스(`products_v2`)를 생성한 뒤 alias를 새 인덱스로 옮기면 **다운타임 없이** 인덱스를 교체할 수 있다.

**1) 새 인덱스 생성 (쓰기 전용 alias 지정)**

```python
es.indices.create(
    index="products_v2",
    body={
        "aliases": {
            "products_write": {}
        },
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

**2) `_aliases` API로 alias를 기존 인덱스 → 새 인덱스로 교체**

```python
es.indices.update_aliases(
    body={
        "actions": [
            { "remove": { "index": "products",    "alias": "products_alias" } },
            { "add":    { "index": "products_v2", "alias": "products_alias" } }
        ]
    }
)
```

- `action` 키를 통해 `remove`(제거)와 `add`(추가)를 조합해서 alias 대상을 자유롭게 전환할 수 있다.
- 클라이언트는 `products_alias`만 바라보고 있으므로, 내부적으로 어떤 실제 인덱스를 가리키는지 바뀌어도 클라이언트 코드는 변경할 필요가 없다.

### Write 전용 alias 지정 (`is_write_index`)

여러 인덱스가 동일한 alias를 공유하되, **쓰기(write)는 그중 하나에만** 하도록 지정할 수 있다.

```python
es.indices.update_aliases(
    body={
        "actions": [
            { "add": {
                "index": "products",
                "alias": "products_alias",
                "is_write_index": False
            }},
            { "add": {
                "index": "products_v2",
                "alias": "products_alias",
                "is_write_index": True
            }}
        ]
    }
)
```

- 검색(읽기) 관점에서는 alias에 연결된 모든 인덱스를 동시에 대상으로 삼지만, 새 문서를 삽입(쓰기)할 때는 `is_write_index: True`로 지정된 인덱스에만 저장된다.
- 예: 로그 인덱스가 날짜별(`log_0101`, `log_0102`...)로 나뉘는 경우, 날짜가 바뀔 때 write 대상 인덱스만 교체해주는 방식으로 활용 가능

인덱스 엘리아스는 실무 운영 단계에서 인덱스 교체, 버전 관리, 멀티 인덱스 검색을 유연하게 처리할 수 있게 해주는 중요한 기능이다.

---

## 💡 한 줄 요약
> Elasticsearch는 단어 중심으로 문서를 찾는 Inverted Index 구조로 빠른 검색을 제공하며, doc_values로 정렬·집계를, Index Alias로 무중단 인덱스 교체와 유연한 운영을 가능하게 한다.

## ❓ 더 찾아볼 것
- Elasticsearch의 Shard와 Replica 개념이 인덱스 알리아스와 어떻게 함께 동작하는지
- `_reindex` API를 활용한 대규모 데이터 재색인 방법
- ILM(Index Lifecycle Management)을 활용한 로그성 인덱스 자동 관리
