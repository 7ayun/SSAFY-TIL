# [데이터 엔지니어링] RAG 적용

---

## 1. LangChain이란

**LangChain**은 언어 모델을 활용한 애플리케이션을 구성하기 위한 프레임워크다. LLM 호출뿐만 아니라 모델에 입력하는 것부터 외부 데이터를 검색하는 것까지, 나만의 챗봇을 만들 때 필요한 요소들을 체인(Chain)처럼 엮어서 쉽고 빠르게 처리할 수 있게 해준다.

- **데이터 인식**: 언어 모델을 다른 데이터 소스(DB, 파일 등)에 연결
- **에이전트 기능**: 언어 모델이 환경과 상호작용할 수 있도록 함 (이번 강의에서는 에이전트보다 RAG 파이프라인 구현에 초점)

## 2. RAG 파이프라인 전체 흐름

LangChain으로 RAG를 만들 때의 기본 흐름은 다음과 같다.

```
1. 문서 업로드   → Document Loader
2. 문서 분할     → Text Splitter
3. 문서 임베딩   → Embed to Vectorstore
4. 벡터 저장     → Save to VectorDB
5. 임베딩 검색   → Vector Store Retriever
6. 답변 생성     → QA Chain (Prompt → LLM → Answer)
```

- **Document Loader**: 다양한 형태의 문서(PDF, Word, txt, CSV 등)를 RAG 전용 객체(Document)로 불러오는 모듈. `page_content`(문서 본문)와 `metadata`(문서 위치, 제목, 페이지 번호 등)로 구성된다.
- **Text Splitter**: 토큰 제한이 있는 LLM이 문서를 참고할 수 있도록 문서를 적절한 크기의 청크(Chunk)로 분할
- **Vector Embeddings**: 각 청크를 숫자 벡터로 변환
- **Retrievers**: 사용자 질문과 유사한 문장을 벡터 저장소에서 찾아 반환

검색기(Retriever)는 보통 1개가 아니라 여러 개의 청크(Top-K)를 반환하는데, 이는 정확도에 대한 일종의 "보험"이다. 상위 3~5개 정도를 함께 넘겨주면 그 안에 정답 근거가 포함될 확률이 높아지기 때문이다.

## 3. 데이터 구조화(Splitting)

### 왜 분할이 필요한가
- LLM에는 `max_token_length` 제한이 있어(예: 4096), 긴 문서를 그대로 넣을 수 없다.
- 구조화(Splitting)를 하지 않으면 Interpreter를 거쳐도 결국 LLM의 입력 크기(Input Size)를 넘거나, 검색 자체가 이상하게 동작할 수 있다.
- 단순한 길이 조절이 아니라, **검색 가능한 문맥 단위로 설계**하는 작업이 중요하다. 문장 중간을 끊어버리면 원래 이어져야 할 정보가 분리되어 손실될 수 있다.

### 짧은 Passage vs 긴 Passage

| | 문서를 짧은 Passage로 구조화 | 문서를 긴 Passage로 구조화 |
|---|---|---|
| 장점 | 개별 정보 손실이 적고 환각 현상이 적어짐 | 맥락 등 종합적인 정보를 담을 수 있어 여러 내용을 종합하는 질문에 답변 가능 |
| 단점 | 종합적인 정보를 담지 못해 여러 내용을 종합해야 하는 질문에 약함 | 개별 정보 손실 가능, LLM이 활용하는 범위가 넓어져 환각 현상 증가 |

### 문서 단위 구조화 vs 특성에 맞는 구조화

| | 문서를 단위로 구조화(자동) | 문서를 특성에 맞게 구조화(수동 규칙) |
|---|---|---|
| 방식 | 문서 종류별 개별 작업 없이 자동으로 구조화 | 문서 종류별로 개별 구조화 규칙을 생성 |
| 장점 | 리소스 부담이 적음 | 연관성 높은 정보끼리 잘 묶임 |
| 단점 | 실제 필요한 정보끼리 묶이지 않을 수 있음 | 리소스(전처리 인력·시간) 투입이 큼 |

예를 들어 "은행 업무로 인해 발생하는 비용이 대출금리를 결정하는데 영향을 끼칠까?" 같은 질문에는, 산출 근거·참고자료가 함께 붙어 있도록 특성에 맞게 구조화된 문서가 더 유리하게 검색된다. 무엇이 정답이라기보다 **검색 시스템의 로직, 기존 문서 형식의 일관성, 전처리에 투입 가능한 리소스**를 고려해서 선택해야 하는 트레이드오프다.

### RecursiveCharacterTextSplitter와 chunk_overlap
- `CharacterTextSplitter`: 구분자 1개를 기준으로 분할 → max_token을 못 지키는 경우 발생
- `RecursiveCharacterTextSplitter`: 줄바꿈 → 마침표 → 쉼표 순으로 재귀적으로 분할 → max_token을 지켜서 분할 (실무에서 대부분 이 방식 사용)
- `chunk_overlap`: 청크 경계에서 문맥이 끊기는 문제를 줄이기 위해, 청크 사이에 일부 구간을 겹치게 잘라주는 설정 (예: chunk_size 250에 overlap 50)

## 4. 실습: RAG 파이프라인 코드 흐름

> 아래는 강의에서 설명한 흐름을 정리한 개념 코드다. GMS(사내 API 키) 기반 ChatOpenAI 객체를 사용했다.

### (1) LLM 호출 기본 형태
```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(base_url=GMS_BASE_URL, api_key=GMS_API_KEY)

response = llm.invoke([
    {"role": "system", "content": "너는 교통사고 데이터 분석가야. 3개 항목으로 요약해줘."},
    {"role": "user", "content": "<분석할 데이터>"}
])
```
- System Message: 모델의 역할·답변 방식을 지정
- Human(User) Message: 실제 사용자의 질문이나 분석할 데이터

### (2) 문서 로드 (Document Loader)
```python
from langchain_community.document_loaders import TextLoader, CSVLoader, PyPDFLoader

# txt 파일 로드 (예: 금융상품 비교 안내서)
docs = TextLoader("financial_product_guide.txt").load()
# docs[0].page_content, docs[0].metadata 확인 가능

# CSV(FAQ)는 행 단위로 검색기를 따로 구성하기도 함
faq_docs = CSVLoader("faq.csv").load()
```
- CSV 로더는 FAQ처럼 "질문-답변-키워드"가 행 단위로 정리된 데이터를 룰베이스에 가까운 챗봇 답변에 활용할 때 유용하다.

### (3) 문서 분할 (Text Splitter)
```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(chunk_size=250, chunk_overlap=50)
chunks = splitter.split_documents(docs)  # 메타데이터는 유지된 채 여러 청크로 분리
```

### (4) 임베딩 및 벡터스토어 저장
```python
from langchain_openai import OpenAIEmbeddings
from langchain_community.vectorstores import FAISS

embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
vectorstore = FAISS.from_documents(chunks, embeddings)  # 인메모리 벡터 DB
```
- 임베딩 모델은 한 번 정하면 웬만하면 계속 같은 모델을 써야 한다. 모델(아키텍처·차원)이 바뀌면 벡터 위치가 달라져 같은 벡터 공간에서 비교할 수 없기 때문이다.

### (5) Retriever 생성 및 검색
```python
retriever = vectorstore.as_retriever(search_kwargs={"k": 3})  # k = 반환할 검색 결과 수
relevant_docs = retriever.invoke("주문한 뒤 배송지 주소를 변경하고 싶어요")
```

### (6) 프롬프트 템플릿 + Runnable 체인 (LCEL)
```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_core.output_parsers import StrOutputParser

def format_docs(docs):
    return "\n\n".join(d.page_content for d in docs)

prompt = ChatPromptTemplate.from_template(
    "다음 컨텍스트를 참고해서 답변해줘. 컨텍스트 안에서 답을 찾을 수 없으면 "
    "'찾을 수 없습니다'라고 답해줘.\n\n컨텍스트: {context}\n\n질문: {question}"
)

rag_chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | llm
    | StrOutputParser()
)

rag_chain.invoke("청년도약적금 우대금리 조건 알려줘")
```

**핵심 개념 두 가지**
- **PromptTemplate**: 프롬프트의 빈칸을 실행 시점 값으로 채워주는 객체 (F-string처럼 매번 새로 만들지 않고 일관되게 재사용)
- **Runnable / LCEL**: A의 출력을 B의 입력으로 넘기는 파이프라인 형태로 구성 요소들을 체인화해주는 것. `RunnableLambda`로 일반 함수도 Runnable 객체로 바꿔 체인에 넣을 수 있다.

### (7) 대화 이력(Chat History) 반영
```python
from langchain_core.prompts import MessagesPlaceholder

prompt = ChatPromptTemplate.from_messages([
    ("system", "너는 이 컨텍스트를 참고해서 답변하는 AI 어시스턴트야..."),
    MessagesPlaceholder("chat_history"),
    ("human", "{question}")
])
```
- `chat_history`는 실행 중인 노트북(런타임) 동안 메모리에 유지되는 리스트로, 최근 N턴까지만 프롬프트에 포함시켜 길이를 제한할 수 있다. 커널을 재시작하면 초기화된다.
- 이전 대화를 참고해 후속 질문(예: "그 상품의 우대금리 조건도 알려줘")에 답할 수 있게 해준다.

## 5. 하이브리드 서치 (Keyword + Semantic)

Semantic Search만으로는 한계가 있어, 실제로는 **BM25 Retriever + Vector Retriever**를 함께 쓰는 하이브리드 서치가 일반적이다.

```python
from langchain_community.retrievers import BM25Retriever
from langchain.retrievers import EnsembleRetriever

bm25_retriever = BM25Retriever.from_documents(chunks)
bm25_retriever.k = 3

vector_retriever = vectorstore.as_retriever(search_kwargs={"k": 3})

ensemble_retriever = EnsembleRetriever(
    retrievers=[bm25_retriever, vector_retriever],
    weights=[0.5, 0.5]
)
```
- 두 Retriever가 각각 top-k(예: 3개씩)를 뽑아오고, 합쳐서 중복을 제거한 뒤 사용 (최종 3~6개 사이)
- 더 정확도를 높이고 싶다면 단순 합집합 대신 **RRF(Reciprocal Rank Fusion)**를 사용: 중복 제거가 아니라, 여러 검색 결과에서 반복적으로 상위권에 등장한 문서에 더 높은 점수를 부여해 재정렬하는 방식. BM25 결과와 벡터 결과, 단순 하이브리드와 RRF 기반 결과의 순위가 서로 다르게 나타날 수 있다.

## 6. 고급 RAG 기법

| 분류 | 기법 | 설명 |
|---|---|---|
| 검색 품질 향상 | Query Transformation (질의 변환) | 사용자 질문을 검색에 유리한 형태로 변환 (예: 여러 하위 질문으로 분리, 가상의 답변을 생성 후 검색) |
| 검색 품질 향상 | Re-ranking (재순위화) | 1차로 검색된 문서들(예: 10개)을 더 정교한 모델(Re-ranker)로 순위를 다시 매겨 정확도를 높임 |
| 답변 품질 향상 | Small-to-Large (Chunk-to-Document) | 작은 Chunk로 먼저 검색해 관련 문서를 찾고, 해당 문서 전체(또는 더 큰 청크 뭉치)를 LLM에 제공해 넓은 문맥으로 답변 생성 |
| 능동적 처리 | Agentic RAG / Self-Corrective RAG | LLM이 스스로 판단해 검색된 정보가 충분하지 않으면 추가 검색을 하거나 쿼리를 다시 만들어 재검색하는 등 능동적으로 행동 |
| 데이터 확장 | Multi-modal RAG | 텍스트뿐 아니라 이미지, 표, 차트 등 다양한 형태의 데이터를 검색·활용 |

이 외에도 그래프 기반 RAG(GraphDB, Neo4j 등을 활용) 등 더 심화된 방식도 존재한다.

## 💡 한 줄 요약
> LangChain은 Document Loader → Text Splitter → Embedding → VectorDB → Retriever → LLM으로 이어지는 체인을 통해 RAG 파이프라인을 쉽게 구성하게 해주며, 실무에서는 BM25(Keyword)와 Vector(Semantic) 검색을 결합한 하이브리드 서치가 널리 쓰인다.

## ❓ 더 찾아볼 것
- LangGraph와 LangChain(LCEL)의 차이점
- Reciprocal Rank Fusion(RRF) 알고리즘 상세 수식
- Agentic RAG / Self-Corrective RAG 구현 사례
- Multi-modal RAG, Graph 기반 RAG(Neo4j 등)
