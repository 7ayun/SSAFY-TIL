# [데이터 엔지니어링] Analyzer

---

## 1. Analyzer(분석기)란?

Analyzer는 문서를 색인하거나 검색할 때 **텍스트를 처리하는 방식**이다. 문서 내용을 토큰(token)으로 변환해서 색인 및 검색을 진행하며, Analyzer를 거친 단어들만 검색 대상이 된다.

- 어떤 Analyzer를 쓰고, 어떤 순서로 실행하느냐에 따라 **검색 결과가 달라질 수 있다** — 설정 방식이 곧 검색 품질에 영향을 준다.
- 너무 많은 분석(전처리)을 적용하면 색인 속도가 느려지는 등 **성능이 저하**될 수 있다. 단순히 공백 기준으로 자를지, 형태소까지 세밀하게 분석할지는 트레이드오프 관계다.

---

## 2. 색인과 검색에서의 Analyzer 차이

- 문서를 저장(색인)할 때와 검색할 때 Analyzer가 다를 수 있지만, 일반적으로는 **색인 시점과 검색 시점에 동일한 tokenizer(같은 방식으로 토큰을 나누는 것)를 사용하는 것이 안전**하다.
- 색인과 검색에 서로 다른 분석기를 적용하는 대표적인 경우:
  - **검색어 필터링이 필요한 경우**: 의미 없는 단어(불용어)를 제거해야 하거나, 특정 단어를 제외하고 검색해야 하는 경우
  - **동의어·맞춤법 교정을 적용하는 경우**: "car"를 검색하면 "automobile"도 함께 검색되게 하거나, "color"와 "colour"를 동일하게 처리하는 경우

---

## 3. Analyzer 구성요소: Character Filter → Tokenizer → Token Filter

Analyzer는 검색 성능 향상을 위해 문서를 토큰화(tokenization)하고 텍스트를 변환하는 기능을 수행하며, **3단계**로 구성된다.

```
Input String
   ↓
[Character Filter] → [Tokenizer] → [Token Filter]
   ↓
A list of unique terms
```

### ① Character Filter (문자 필터)

- 원본 텍스트를 토큰으로 나누기 **전** 전처리하는 단계. 특정 문자나 패턴을 변환·제거함
- `html_strip`: HTML 태그 제거 (HTML 문서를 그대로 분석하면 태그 때문에 검색 품질이 떨어지므로 사용)
- `mapping`: 특정 문자열을 다른 문자열로 매핑
- `pattern_replace`: 정규식을 이용한 텍스트 변경 (예: 전화번호·개인정보 마스킹 처리)

### ② Tokenizer (토크나이저)

- Character Filter를 거친 텍스트를 특정 규칙에 따라 **토큰(단어) 단위로 분리**
- Analyzer 구성 시 **한 개의 Tokenizer만 사용 가능** (Character Filter와 Token Filter는 여러 개 사용 가능)
- `whitespace`: 공백 기준으로 단어 분리
- `standard`: 일반적인 텍스트 토큰화 (영어 기반이라 한국어와는 잘 맞지 않음)
- `ngram`: 문자열을 부분 문자열(서브스트링) 단위로 분리
  - 예: "안녕하세요"를 bigram으로 나누면 → `안녕`, `녕하`, `하세`, `세요` (자동완성 구현 시 활용)

### ③ Token Filter (토큰 필터)

- Tokenizer로 분리된 토큰을 **추가, 수정, 삭제**하는 필터. 여러 개를 배열로 조합 가능
- `lowercase`: 모든 단어를 소문자로 변환 (대소문자 표기가 달라도 같은 단어로 인식되게 함)
- `stop`: 불용어(예: "the", "is") 제거 — 검색 의미가 크지 않은 단어를 걸러냄
- `synonym`: 동의어 처리 (예: "car" ↔ "automobile")
- 필터를 여러 개 순서대로 적용할 수 있으며, **적용 순서에 따라 최종 색인되는 토큰이 달라질 수 있다.**

---

## 4. `_analyze` API로 확인하기

커스텀 Analyzer가 실제로 어떻게 동작하는지 테스트할 수 있는 API.

```python
response = es.indices.analyze(body={
    "analyzer": "whitespace",
    "text": "삼성 청년 SW 아카데미"
})
```

응답 결과:
```json
{
  "tokens": [
    { "token": "삼성", "start_offset": 0, "end_offset": 2, "type": "word", "position": 0 },
    { "token": "청년", "start_offset": 3, "end_offset": 5, "type": "word", "position": 1 },
    { "token": "SW", "start_offset": 6, "end_offset": 8, "type": "word", "position": 2 },
    { "token": "아카데미", "start_offset": 9, "end_offset": 13, "type": "word", "position": 3 }
  ]
}
```

- `token`: 분리된 단어
- `start_offset` / `end_offset`: 원본 텍스트(전문) 기준 시작/끝 위치
- `position`: 토큰의 순서

---

## 5. Custom Analyzer 조합하기

Character Filter, Tokenizer, Token Filter를 조합해 직접 Analyzer를 구성할 수 있다. tokenizer는 하나만 지정 가능하며, char_filter와 filter는 여러 개 적용 가능하다.

```python
response = es.indices.analyze(body={
    "char_filter": ["html_strip"],
    "tokenizer": "whitespace",
    "filter": ["stop", "lowercase"],
    "text": ["<b>삼성 갤럭시</b> S25 Ultra"]
})
```

- 위 예시는 HTML 태그를 제거(`html_strip`)한 뒤, 공백 기준(`whitespace`)으로 토큰화하고, 불용어 제거(`stop`)와 소문자 변환(`lowercase`)을 적용한다.
- `start_offset`, `end_offset`은 원본 문서(전문) 전체 기준이므로, HTML 태그가 제거되기 전 위치를 반영해 계산된다.

실무에서는 보통 기본 Analyzer를 그대로 쓰지 않고, 처리하려는 언어·산업 특성에 맞게 이렇게 **커스터마이징**해서 사용한다. 특히 한국어는 이런 커스터마이징이 거의 필수적이다.

---

## 6. 한국어 Analyzer (Nori Analyzer)

한국어는 조사·어미 등 문장 구조가 복잡해서 `whitespace`, `standard` 같은 기본 분석기로는 적절히 분해하기 어렵다. (예: "삼성전자의 새로운 제품"을 공백 기준으로만 나누면 "삼성전자의"까지 하나의 토큰이 되어, "삼성전자"로 검색해도 매칭되지 않는 문제 발생)

- 한국어 처리를 위해 **Nori 분석기**를 제공하며, **형태소 기반 분석기**로 한국어의 복잡한 문장 구조를 효과적으로 분석한다.
- Elasticsearch 기본 패키지가 아니라 **플러그인 설치가 필요**하다.

### Nori 구성요소

- **Tokenizer**: `nori_tokenizer` — 형태소 분석을 수행하여 단어를 분리
- **Token Filter**:
  - `nori_part_of_speech`: 품사 기반 필터링 (예: 명사만 남기고 조사·어미 등 제외)
  - `nori_readingform`: 한자·외래어 등을 한글 발음으로 변환
  - `nori_number`: 숫자를 표준화 (예: "일" → "1")

```python
es.indices.create(
    index="products",
    body={
        "settings": {
            "index": {
                "analysis": {
                    "tokenizer": {
                        "nori_tokenizer": {
                            "type": "nori_tokenizer",
                            "decompound_mode": "mixed",
                            "discard_punctuation": "false"
                        }
                    },
                    "analyzer": {
                        "custom_nori_analyzer": {
                            "type": "custom",
                            "tokenizer": "nori_tokenizer"
                        }
                    }
                }
            }
        }
    }
)
```

- `decompound_mode`: 복합명사 처리 방식 지정
  - `discard`: 분해된 단어만 사용 (원형은 버림)
  - `none`: 분해하지 않은 원형만 사용
  - `mixed`: 분해된 단어 + 원형을 모두 사용
- `discard_punctuation`: 구두점(마침표, 쉼표, 하이픈 등) 제거 여부. 기본적으로 구두점은 제거됨(`true`)
- `user_dictionary`: 사용자 정의 사전 적용 (아래 8번 참고)

품사 필터 예시 (`nori_part_of_speech`)는 조사, 어미 등 검색에 불필요한 품사 태그(`E`, `IC`, `J`, `MAG`, `MAJ`, `MM`, `SP`, `SSC`, `SSO`, `SC`, `SE`, `XPN`, `XSA`, `XSN`, `XSV`, `UNA`, `NA`, `VSV` 등)를 제외하는 방식으로 사용한다.

---

## 7. 동의어(Synonym) 처리

동의어는 완전히 같은 단어는 아니어도 같은 의미로 검색되게 해주는 기능으로, **색인 시** 처리하는 방식과 **검색 시** 처리하는 방식 두 가지가 있다.

### 색인 시 동의어 처리

- 문서를 저장(색인)하는 시점에 동의어를 미리 확장해서 저장하는 방식
- **장점**: 검색 시 추가 처리가 필요 없어 검색 속도가 빠름
- **단점**: 색인된 데이터 크기가 증가할 수 있고, 동의어 사전이 바뀌면 전체 데이터를 **재색인(reindexing)** 해야 함

```python
"analyzer": {
    "synonym_analyzer": {
        "type": "custom",
        "tokenizer": "standard",
        "filter": ["lowercase", "synonym_filter"]
    }
}
```
```python
"synonym_filter": {
    "type": "synonym",
    "synonyms": [
        "notebook, laptop",
        "smartphone, mobile"
    ]
}
```

이렇게 만든 analyzer는 매핑에서 필드에 지정해야 실제로 적용된다.

```python
"mappings": {
    "properties": {
        "description": {
            "type": "text",
            "analyzer": "synonym_analyzer"
        }
    }
}
```

### 검색 시 동의어 처리

- 검색어(query)를 입력하는 시점에 동의어를 확장해서 질의하는 방식
- **장점**: 색인된 문서를 변경하지 않고 동의어 규칙만 바꿀 수 있어, 동의어 사전을 수정해도 전체 데이터를 재색인할 필요가 없음
- **단점**: 검색 시 추가적인 처리 비용이 발생하고 (질의를 여러 번 수행하는 셈), 동의어를 잘못 등록하면 예상치 못한 단어가 검색 결과에 포함되어 품질이 저하될 수 있음
- 파일 기반 search analyzer 동의어는 `_reload_search_analyzers` API와 캐시 정리가 필요할 수 있음

```python
"mappings": {
    "properties": {
        "description": {
            "type": "text",
            "analyzer": "standard",
            "search_analyzer": "synonym_analyzer"
        }
    }
}
```

### 동의어 사전 구성

- **단어 동등 관계 (A, B)**: A와 B를 같은 의미로 처리 — `"notebook, laptop"`
- **단어 치환 관계 (A → B)**: A를 대표 표현 B로 치환 (표준 용어로 통일하고 싶을 때 사용) — `"notebook => laptop"`
- 적용 시점(색인/검색)에 따라 재색인 필요 여부가 달라진다는 점을 유의해야 한다.

### 동의어 사전 불러오기

- Elasticsearch 노드의 `config` 폴더 하위, 보통 `config/dictionary/` 디렉토리에 `synonyms.txt` 같은 파일로 저장
- 검색 시점에 동의어를 적용하는 경우, 사전을 수정해도 즉시 반영되지 않으므로 **리로드 API**를 실행해야 함

```
POST /products/_reload_search_analyzers
POST /products/_cache/clear?request=true
```

---

## 8. 불용어(Stopword) 처리

- 검색에 큰 의미가 없는 단어(예: "the", "is", "was", "a", "an", "of" 등)를 제거하는 사전
- `config/dictionary/` 폴더에 `stopwords.txt` 같은 파일을 만들어 한 줄에 하나씩 등록
- 색인 또는 검색 과정에서 필터로 적용 가능하며, 검색 시점에 적용하는 경우 사전 변경 후 리로드 절차가 필요할 수 있음

```python
es.indices.create(
    index="products",
    body={
        "settings": {
            "index": {
                "analysis": {
                    "analyzer": {
                        "search_stop_analyzer": {
                            "tokenizer": "whitespace",
                            "filter": ["search_stop_filter"]
                        }
                    },
                    "filter": {
                        "search_stop_filter": {
                            "type": "stop",
                            "stopwords_path": "dictionary/stopwords.txt"
                        }
                    }
                }
            }
        }
    }
)
```

---

## 9. 사용자 사전 (User Dictionary)

- 불용어 사전이 "제외할 단어"를 관리하는 것과 반대로, 사용자 사전은 **복합명사나 단일명사를 원하는 형태로 검색되도록 강제 지정**하는 용도다.
- 한국어에는 복합명사·고유명사가 많아, Nori 분석기가 기본적으로 원하는 단위로 토큰을 나누지 못할 수 있다. (예: "삼성갤럭시"가 "삼성"과 "갤럭시"로 분리되길 원하거나, 반대로 하나로 유지되길 원하는 경우)
- `dictionary/` 폴더 하위에 `userdict_ko.txt` 같은 파일로 저장

```
아이폰
삼성갤럭시 삼성 갤럭시
```

```python
"tokenizer": {
    "nori_custom_tokenizer": {
        "type": "nori_tokenizer",
        "decompound_mode": "mixed",
        "discard_punctuation": "false",
        "user_dictionary": "dictionary/userdict_ko.txt"
    }
},
"analyzer": {
    "user_dic_analyzer": {
        "type": "custom",
        "tokenizer": "nori_custom_tokenizer"
    }
}
```

- `decompound_mode: mixed`로 복합어 처리를 적절히 수행하고, `discard_punctuation: false`로 구두점을 유지하며, `user_dictionary`로 사용자 사전을 적용한다.
- 향후 색인 및 검색 시점 모두에 적용 가능하다.

---

## 10. 실습 흐름 정리 (동의어·사용자 사전·불용어 종합 적용)

실습 코드에서는 `synonym`(동의어 사전) + `user_dictionary`(사용자 사전) + `stop`(불용어 필터)을 함께 적용한 인덱스를 만들고, bulk로 샘플 데이터를 넣은 뒤 검색 테스트를 진행했다.

- 데이터에 없는 값("모바일")으로 검색했지만 동의어 사전(`smartphone, mobile`)이 색인 시점에 적용되어 있어 카테고리가 "smartphone"인 문서가 정상적으로 검색됨을 확인
- "삼성 갤럭시"로 검색했을 때, 사용자 사전으로 하나의 단위로 유지된 복합어와 동의어로 등록된 영어 표현이 모두 매칭되어 검색 결과가 늘어나는 것을 확인
- 불용어 사전을 적용하면 "The Galaxy and iPhone is a smartphone" 같은 문장에서 "The", "and", "is", "a" 같은 불용어가 제거되고 의미 있는 단어만 토큰으로 남는 것을 `_analyze` API로 확인

이처럼 Mapping과 Analyzer를 적절히 조합하면, 단순 매칭을 넘어 사용자가 원하는 검색 품질을 세밀하게 제어할 수 있다.

---

## 💡 한 줄 요약
> Analyzer는 Character Filter → Tokenizer → Token Filter 순서로 텍스트를 토큰화하는 파이프라인이며, 한국어의 경우 Nori 분석기에 동의어·불용어·사용자 사전을 조합해 커스터마이징해야 원하는 검색 품질을 얻을 수 있다.

## ❓ 더 찾아볼 것
- n-gram / edge n-gram 방식과 search_as_you_type의 차이
- Nori 분석기의 `decompound_mode` 옵션별 실제 토큰화 결과 비교
- BM25 스코어링과 동의어 확장이 relevance score에 미치는 영향
