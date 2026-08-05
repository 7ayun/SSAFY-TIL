# [데이터 엔지니어링] Elasticsearch 개요

---

## 1. 지금까지의 데이터 파이프라인과 Elasticsearch의 위치

지금까지 다룬 데이터 파이프라인 기술 스택을 역할 기준으로 정리하면 아래와 같다.

| 기술 | 역할 |
|---|---|
| Kafka | 지속적으로 발생하는 데이터(메시지)를 안정적으로 수집·전달 |
| Spark | 대량 데이터의 배치 처리, Structured Streaming을 통한 준실시간 처리 |
| Airflow | 배치/주기적 데이터 흐름을 자동화하고 스케줄링·실패 로그 관리 |

이렇게 수집·처리된 데이터는 결국 어딘가에 **저장**되어야 한다. 저장소는 용도에 따라 다음과 같이 구분된다.

- **RDB**: 구조화된 데이터를 안정적으로 저장하는 운영계(OLTP) DB
- **데이터 웨어하우스**(Snowflake, BigQuery 등): 구조화된 대량 데이터를 클라우드 기반으로 저장·분석
- **데이터 레이크**(S3, HDFS 등): 원시 데이터를 대량으로 저장하는 분산 파일 시스템 기반 저장소
- **NoSQL**(MongoDB 등): SQL을 사용하지 않는 저장소 전반. 비정형 데이터나 대규모 저장에 사용

**Elasticsearch**는 이 중 어디에도 딱 들어맞지 않는다. 데이터를 저장할 수는 있지만 단순 저장소라기보다 **검색과 분석에 특화된 엔진**이다. OLTP도 OLAP도 아닌, 메인 저장소에 부가적으로 붙어 검색/분석 기능을 제공하는 형태로 동작한다. 즉, **검색 엔진을 내장한 저장소**라고 볼 수 있다.

> 참고: RAG(Retrieval-Augmented Generation)에서 LLM의 할루시네이션을 보완하기 위한 Retriever 단계에서도, 벡터 검색과 함께 텍스트 기반 검색(하이브리드 검색)이 자주 활용되는데 이때도 검색 기술의 중요성이 강조된다.

---

## 2. 정보 검색(Information Retrieval)이란?

**정보 검색**은 대규모 데이터 속에서 사용자가 원하는 정보를 찾아 제공하는 기술이다. 단순히 웹사이트 검색만을 의미하지 않으며, 웹 문서·이미지·동영상·연구 논문 등 다양한 데이터 유형을 대상으로 한다. Google, Microsoft(Bing), Naver 등 글로벌 기업이 핵심 기술로 활용하고 있다.

검색 시스템에 요구되는 사항:
- 빠르고 정확한 검색 결과
- 좋은 사용자 경험
- 보안 및 개인정보 보호 준수
- 대량 요청에도 안정적인 확장성·유지보수

### 정보 검색의 핵심 기술

**① 데이터 수집(Data Collection)**
- **웹 크롤링(Crawling)**: 링크를 타고 이동하며 웹 문서를 자동으로 수집·색인
- **스크래핑(Scraping)**: 특정 사이트/페이지에 접근해 필요한 데이터만 추출·가공

**② 데이터 저장(Data Storage) — 역색인(Inverted Index)**
일반적인 저장 방식이 "문서"를 중심으로 한다면, 역색인은 **단어를 중심으로** 저장한다. 키워드와 그 키워드가 포함된 문서 정보를 저장해두고, 검색 시 키워드 기반으로 관련 문서를 빠르게 찾아낸다. Elasticsearch는 이 역색인 구조를 **Lucene** 라이브러리 기반으로 구현하고 있다.

**③ 검색 알고리즘(Search Algorithm)**
| 알고리즘 | 설명 |
|---|---|
| TF-IDF | 특정 키워드가 문서 내에서 갖는 상대적 중요도(빈도-역문서빈도)를 계산 |
| BM25 | TF-IDF를 보완한 가중치 기반 알고리즘. 문서 길이·단어 빈도를 더 현실적으로 반영해 검색 정확도를 높임 |

Elasticsearch는 기본적으로 **BM25**를 기반으로 검색 결과에 스코어링(점수 계산)을 적용한다.

---

## 3. 기존 RDB 검색의 한계

"삼성 블루투스 이어폰"이라는 타이틀을 가진 상품을 검색한다고 가정하면, RDB에서는 다음과 같은 쿼리를 사용한다.

```sql
SELECT title FROM product WHERE title LIKE '%삼성 블루투스 이어폰%'
```

이 방식은 **문자열이 정확히 포함되어 있어야만** 매칭된다는 한계가 있다.

- **쿼리가 복잡해짐**: 띄어쓰기·단어 순서가 다르면(예: "삼성 이어폰 블루투스") 매칭되지 않음
- **성능 문제**: `LIKE` 연산은 인덱스를 무력화시켜 풀스캔을 유발 → 데이터가 많을수록 성능이 급격히 저하
- **스펠링 오류·유사 검색 불가**: "버즈"처럼 의미상 동일한 단어나 오타가 있는 경우 검색이 불가능

이런 한계를 보완하기 위해 등장한 것이 Elasticsearch와 같은 전문 검색 엔진이다.

---

## 4. 검색 엔진 시장에서의 Elasticsearch

DB-Engines Ranking(검색 엔진 부문, 2026년 4월 기준)에서 Elasticsearch는 **1위**를 차지하고 있다. Splunk, Apache Solr, OpenSearch 등이 뒤를 잇는다.

- Solr, OpenSearch 등도 내부적으로는 모두 **Lucene**을 기반으로 만들어져 있다.
- **OpenSearch**는 AWS가 Elasticsearch를 포크(fork)하여 만든 검색엔진이다(라이선스 이슈로 분쟁이 있었음). AWS 클라우드 환경이 성장하면서 OpenSearch 사용도 늘고 있지만, 두 엔진은 구조적으로 매우 유사하여 하나를 알아두면 다른 것도 무리 없이 다룰 수 있다.

---

## 5. Elasticsearch란?

- 강력한 **오픈소스 검색 및 분석 엔진**
- **Apache Lucene** 기반
- **Elastic Stack**(Logstash, Beats, Kibana 포함)의 핵심 구성 요소
- 수평적 확장성, 안정성, 쉬운 관리를 위해 **분산 환경**을 고려해 설계됨 → 데이터가 많아지면 노드를 여러 개 띄워 데이터를 나눠 저장하고, 검색 요청도 여러 노드가 함께 처리

### Elastic Stack (ELK Stack)

과거에는 Elasticsearch + Logstash + Kibana를 묶어 **ELK 스택**이라 불렀으나, 이후 Beats가 추가되며 **Elastic Stack**으로 확장되었다.

```
[기본 흐름]
Logstash(수집·가공) → Elasticsearch(색인·저장) → Kibana(분석·시각화)

[대규모 환경]
Beats(수집) → Kafka(버퍼링) → Logstash(가공) → Elasticsearch(색인·저장) → Kibana(시각화)
```

서버 규모가 커지면 각 백엔드 서버에 Beats를 붙여 데이터를 Kafka로 버퍼링하고, Logstash가 중간에서 데이터를 정리한 뒤 Elasticsearch에 전달하는 방식으로 **저장 부하와 검색 부하를 분리**할 수 있다.

---

## 6. Elasticsearch와 Lucene의 관계

| 구분 | 설명 |
|---|---|
| **Lucene** | Java로 작성된 검색 라이브러리. 검색·색인(Indexing) 핵심 기능을 담당. 단독 사용 시 애플리케이션을 직접 개발해야 함 |
| **Elasticsearch** | Lucene을 감싼(wrapping) 분산 검색 엔진. REST API, 분산 환경 지원 등을 추가해 사용자 친화적으로 확장한 검색 플랫폼 |

Elasticsearch에서의 검색 관련 API는 대부분 Lucene 기반 검색 API에서 출발하며, 여기에 분산 처리·캐싱·샤드 기반 검색 등의 기능을 얹어 대규모 데이터 검색을 최적화한다.

```
Elasticsearch Index
 ├─ primary: Elasticsearch shard (Lucene index)
 │            └─ Lucene segment, Lucene segment
 ├─ replica: Elasticsearch shard (Lucene index)
 │            └─ Lucene segment, Lucene segment
 └─ replica: Elasticsearch shard (Lucene index)
              └─ Lucene segment, Lucene segment
```

- **Index**(뉴스 인덱스 등)는 RDB의 테이블과 비슷한 논리적 저장 단위
- Index는 여러 개의 **Shard**(primary/replica)로 나뉘어 저장
- 각 Shard는 내부적으로 하나의 **Lucene Index**이며, 여러 개의 **Segment**로 구성됨

### Lucene의 동작 구조 (색인 · 검색)

```
[색인]  Document → Text analysis chains(토큰화) → IndexWriter → Segment → Directory(파일 저장)
[검색]  Query → Query parser → Text analysis chains → Scoring API → Collector → 순위화된 결과
```

- 문서가 들어오면 텍스트 분석 체인을 거쳐 문장을 단어 단위(토큰)로 쪼갠 뒤 역색인 형태로 Segment에 저장
- 검색어도 동일한 분석 체인을 거쳐 토큰화된 뒤, 여러 Segment 조합에서 적합한 문서를 찾아 점수(Scoring)를 계산해 정렬된 결과(Ranked Results)를 반환

### Segment의 특징

- Lucene에서 색인된 문서들을 저장하는 **최소 단위**
- 하나의 샤드는 여러 개의 세그먼트로 구성
- **한 번 생성되면 수정되지 않음(Immutable)**
  - 문서가 업데이트되면 → 기존 세그먼트를 고치는 게 아니라 **새로운 세그먼트가 생성**됨
  - 삭제된 문서는 즉시 지워지지 않고 **삭제 플래그**로만 표시됨
- 장점
  - **동시성 확보**: 여러 세그먼트에서 동시에 검색 가능
  - **빠른 색인 처리**: 기존 세그먼트를 수정하지 않고 새 세그먼트만 추가
  - **안정적인 검색**: 검색 중단 없이 색인이 가능

> 세그먼트가 계속 늘어나면 검색 시 확인해야 할 대상이 많아져 성능에 부담이 되므로, 주기적으로 여러 세그먼트를 하나의 큰 세그먼트로 **병합(Merge)**하는 과정을 거친다.

---

## 7. Elasticsearch의 특징

| 특징 | 설명 |
|---|---|
| 분산 구조(Distributed Nature) | 클러스터 내 모든 노드에 데이터를 자동 분산 → 준실시간으로 대량 데이터 처리 |
| 전문 검색(Full-Text Search) | HTTP 웹 인터페이스와 스키마리스(schema-less) JSON 문서 기반의 고급 전문 검색 지원 |
| 확장성(Scalability) | 수백~수천 대 서버로 확장 가능, 페타바이트급 데이터 처리 가능 |
| 유연성(Flexibility) | 다양한 소스의 이질적 데이터 유형을 색인, 복잡한 검색 기능 제공 |

그 외 특징:
- **유연한 JSON 데이터 관리**: 스키마리스 방식(단, `Mapping`이라는 스키마와 유사한 개념이 존재 — 다음 시간에 다룰 예정)
- **정밀한 검색 및 필터링**, **다양한 검색 쿼리 지원**(정렬, 그룹화 등)
- **다양한 클라이언트 지원**: Python, Java, .NET, PHP 등 SDK 제공
- **확장성과 안정성**: 오토스케일링(자동으로 서버가 늘어난다기보다, 인프라 작업으로 새 노드를 추가하면 클러스터에 자동으로 참여하는 방식), 데이터 백업·복원 기능
- **Kibana 시각화**: 대시보드·리포팅을 통해 로그 분석/모니터링에 활용 가능

---

## 8. Elasticsearch의 기본 요소

- **Document(문서)**: 색인될 수 있는 기본 정보 단위. JSON 형식으로 표현되는 가벼운 데이터 교환 형식
- **Field(필드)**: Elasticsearch에서 가장 작은 데이터 단위, key-value 쌍을 의미

### RDBMS와의 대응 관계

| Elasticsearch | RDBMS |
|---|---|
| Index | Database (테이블과 데이터베이스의 중간 정도 개념) |
| Document | Row |
| Field | Column |
| Mapping | Schema |

RDBMS의 Schema는 타입을 강제하는 구조지만, Elasticsearch의 **Mapping**은 강제성이 상대적으로 약해 문서마다 필드가 존재하거나 존재하지 않을 수 있다. 이런 특성 때문에 "스키마리스(schema-less)"라고 표현한다.

---

## 9. 데이터 저장 및 관리

| 개념 | 설명 |
|---|---|
| **인덱싱(Indexing)** | 데이터를 Index 단위로 관리. 각 인덱스는 Database처럼 동작. 문서는 JSON으로 저장되며 검색에 최적화된 형태로 변환(분석기 적용 등) |
| **샤딩(Sharding)** | Index는 여러 개의 Shard로 나뉨. 데이터를 여러 노드에 분산 저장해 성능 향상 및 대용량 처리. 샤드를 과도하게 만들면 관리 비용 증가·성능 저하로 이어질 수 있어 데이터 크기·노드 수·검색 부하를 고려해 적정 수로 설정 필요 |
| **레플리카(Replica)** | 기본 샤드의 사본. 장애 발생 시 데이터 손실 방지, 검색 요청 분산 처리로 성능·안정성 향상 |

---

## 10. Elasticsearch 검색 동작 원리

**① 질의 처리(Query Processing)**
사용자가 입력한 질의는 구문 분석(Parsing) 및 변환(Transforming) 과정을 거쳐 Lucene 인덱스에서 검색 가능한 형태로 최적화된다. 변환된 질의는 모든 관련 샤드(기본 샤드 + 복제 샤드)에 병렬로 실행되어 빠른 검색이 가능하다.

**② 연관성 점수 계산(Relevance Scoring)**
TF-IDF, BM25 등의 알고리즘으로 각 문서가 질의와 얼마나 부합하는지 계산해 순위를 결정한다. 이 스코어링은 주로 텍스트 타입(전문 검색)에 적용되며, 랭킹 없이 전체 데이터를 그대로 가져오고 싶다면 스코어링 없이 조회하는 것도 가능하다.

**③ 준실시간 검색(Near Real-Time, NRT)**
- 데이터를 색인하자마자 100% 즉시 검색되는 것은 아니지만, 아주 짧은 시간 뒤 검색 가능한 상태가 된다.
- **메모리 버퍼**를 활용해 새로운 문서를 임시로 저장해두고, 일정 주기로 버퍼를 비워 세그먼트로 생성한다. 세그먼트에 완전히 기록되기 전에도 메모리 버퍼 덕분에 검색 결과에 반영될 수 있다.
- 준실시간 검색이 가능한 이유
  - 메모리 기반 버퍼링으로 색인 속도 향상
  - 비동기 색인 처리 → 검색과 색인을 동시에 수행 가능
  - Lucene 엔진 최적화를 통한 빠른 색인 적용

---

## 💡 한 줄 요약
> Elasticsearch는 Lucene 기반의 분산 검색·분석 엔진으로, 역색인과 불변(Immutable) 세그먼트 구조를 통해 대량의 데이터를 준실시간으로 빠르고 유연하게 검색할 수 있게 해준다.

## ❓ 더 찾아볼 것
- Mapping과 Dynamic Mapping / Explicit Mapping의 차이
- Analyzer(분석기)의 동작 원리 (토크나이저, 필터)
- OpenSearch와 Elasticsearch의 라이선스 분쟁 히스토리
- BM25 알고리즘의 수식과 TF-IDF 대비 개선점
