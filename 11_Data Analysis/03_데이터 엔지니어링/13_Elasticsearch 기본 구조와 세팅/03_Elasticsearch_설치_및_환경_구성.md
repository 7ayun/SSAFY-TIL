# [데이터 엔지니어링] Elasticsearch 설치 및 환경 구성

---

## 1. Docker를 통한 설치

로컬 환경에 직접 설치하는 대신, **OS 환경에 영향받지 않도록 Docker/Docker Compose를 통한 설치**를 권장한다.

- Windows에서는 **WSL 상의 Docker Desktop**을 이용
- 파일명이 `docker-compose.yml`이 아니라 다르게 지정된 경우(`docker-compose-elastic.yml` 등), `-f` 옵션을 붙여 실행

```bash
docker compose -f docker-compose-elastic.yml up
```

---

## 2. docker-compose-elastic.yml 구성 요소

Elasticsearch 공식 이미지를 기반으로, 한국어 형태소 분석기인 **Nori Analyzer** 플러그인을 미리 설치하기 위해 Dockerfile을 커스텀하여 사용한다.

### Image / container_name

```yaml
services:
  es01:
    build:
      context: .
      dockerfile: Dockerfile.elastic
    image: custom-elasticsearch-nori:8.17.1
    container_name: es01
```

- `dockerfile: Dockerfile.elastic` → 현재 위치의 `Dockerfile.elastic`을 읽어 이미지를 빌드
- `image`: 공식 Elasticsearch 8.17.1 이미지를 Nori 분석기가 포함되도록 커스터마이징한 이미지로 네이밍
- `container_name`: 컨테이너끼리 통신할 때 사용하는 이름을 `es01`로 고정 (노드 3개를 띄우므로 es01/es02/es03로 구분)

### node.name / cluster.name

```yaml
environment:
  - node.name=es01
  - cluster.name=elastic-docker-cluster
```

- `node.name`: 클러스터 내에서 노드를 구분하는 **고유 이름** (컨테이너 이름과는 별개로, Elasticsearch 자체에서 사용하는 노드 식별자)
- `cluster.name`: 같은 이름을 가진 노드끼리 하나의 클러스터로 묶임

### discovery.seed_hosts / cluster.initial_master_nodes

```yaml
  ## 3개의 노드 실행 시
  - discovery.seed_hosts=es02,es03
  - cluster.initial_master_nodes=es01,es02,es03

  ## 노드 하나만 실행 시 (위 두 줄은 주석 처리)
  # - discovery.seed_hosts=es02,es03
  # - cluster.initial_master_nodes=es01,es02,es03
  - discovery.type=single-node
```

- `discovery.seed_hosts`: 클러스터에 참여할 **다른 노드들의 주소 목록**. 예: es01 입장에서는 es02, es03을 seed host로 지정하여 3개 노드를 연결
- `cluster.initial_master_nodes`: 클러스터 **최초 부트스트랩 시점**에 마스터 후보가 될 수 있는 노드 목록을 지정 (예: es01, es02, es03 중 아무나 마스터가 될 수 있음을 의미). 장애 시 대체할 노드를 지정하는 개념과는 다르며, 클러스터가 정상 구성된 이후에는 값을 변경하지 않음
- 리소스 문제 등으로 노드 하나만 실행하고 싶다면 위 두 설정을 주석 처리하고 `discovery.type=single-node`로 설정

### node.roles / jvm.options

```yaml
  - node.roles=master,data,ingest
  - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
  - xpack.security.enabled=false
  - network.host=0.0.0.0
```

- `node.roles`: 이 노드가 수행할 역할 지정 (`master`, `data`, `ingest` 등). 실습 환경에서는 3개 노드 모두 모든 역할을 수행하도록 동일하게 설정했지만, 실제 대규모 운영에서는 역할을 분리하는 것이 리소스 낭비를 줄이는 데 유리하다
- `jvm.options` (`ES_JAVA_OPTS`): Elasticsearch가 사용할 Java 힙 메모리 크기 지정
  - Elasticsearch는 Lucene과 마찬가지로 **Java 기반**으로 만들어져 있어 JVM 힙 크기 설정이 필요
  - `-Xms`: 최소 힙 크기, `-Xmx`: 최대 힙 크기 (실습용으로 512m 정도면 충분)

### volumes / ports / networks

```yaml
volumes:
  # Elasticsearch 데이터 저장용 볼륨
  - es01-data:/usr/share/elasticsearch/data
  # Nori 사용자 사전 파일 공유용 디렉터리
  - ./config/dictionary:/usr/share/elasticsearch/config/dictionary
ports:
  - 9200:9200
networks:
  - elastic

volumes:
  es01-data:
  es02-data:
  es03-data:

networks:
  elastic:
    driver: bridge
```

| 설정 | 설명 |
|---|---|
| `volumes` | 컨테이너 내부의 Elasticsearch 데이터 디렉터리를 Docker 볼륨(`es01-data` 등)에 연결. 클러스터의 각 노드는 자신이 맡은 데이터를 자신의 데이터 디렉터리에 저장하며, 컨테이너가 중단·삭제되어도 데이터가 유지되도록 함 |
| `ports` | 호스트의 포트를 컨테이너의 9200번 포트에 매핑. es01은 `9200:9200`, es02는 `9201:9200`처럼 호스트 쪽 포트만 다르게 설정해 로컬에서 각각 접속 가능 (컨테이너 내부적으로는 모두 9200 사용) |
| `networks` | `elastic`이라는 Docker 브리지 네트워크로 연결되어 컨테이너 이름 기반(DNS)으로 서로 통신 가능. Kibana 등 다른 서비스와의 통신에도 사용 |
| `config/dictionary` | Nori 분석기의 사용자 사전 파일 공유용 디렉터리 (다음 시간 한국어 형태소 분석에서 다룰 예정) |

---

## 3. 실행 및 확인

```bash
docker compose -f docker-compose-elastic.yml up
```

위 명령으로 총 3개의 Elasticsearch 인스턴스(es01, es02, es03)와 Kibana(선택적으로 시각 확인용) 컨테이너가 함께 실행된다.

브라우저에서 각 노드가 정상적으로 떠 있는지 GET 요청으로 확인 가능:

- `http://localhost:9200` → `es01`
- `http://localhost:9201` → `es02`
- `http://localhost:9202` → `es03`

각 주소에 접속하면 해당 노드의 이름(`name` 필드에 es01/es02/es03)이 응답으로 표시되는 것을 브라우저에서 바로 확인할 수 있다(GET 명령어를 실행한 것과 동일한 효과).

---

## 💡 한 줄 요약
> Elasticsearch는 Docker Compose로 노드별 이름·클러스터명·역할(node.roles)·JVM 힙 크기·볼륨·포트를 설정해 멀티 노드 클러스터를 손쉽게 구성할 수 있다.

## ❓ 더 찾아볼 것
- Nori 형태소 분석기 플러그인 설치 및 사용자 사전 설정 방법
- xpack.security를 활성화했을 때의 인증 설정 방법
- 운영 환경에서 권장되는 JVM 힙 크기 산정 기준
