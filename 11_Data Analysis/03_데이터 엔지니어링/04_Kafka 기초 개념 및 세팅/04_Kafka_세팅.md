# [데이터 엔지니어링] Kafka 세팅

---

## 1. 환경 설정 전 필수 요소

Kafka는 JVM(Java Virtual Machine) 기반으로 동작하기 때문에, 설치 전에 아래 요소들을 확인해야 한다.

- **운영체제**: Linux 환경이 가장 안정적이며 추천됨. Windows는 **WSL** 환경 사용을 권장
- **Java**: JVM 실행을 위해 필요 (Java 8 이상이면 되지만, 실습에서는 버전 통일을 위해 **Java 17** 사용, JRE보다는 **JDK** 설치 권장)
- **Zookeeper**: Kafka를 다운로드하면 함께 포함되어 있으므로 별도 설치가 필요 없음

이번 실습은 **로컬에 직접 다운로드해서 세팅하는 방법**과 **Docker Compose로 띄우는 방법** 두 가지를 모두 다룬다. 로컬 설치를 함께 보는 이유는, 온프레미스 환경에서 직접 설치했을 때 디렉토리 구조나 내부 동작 방식을 간략하게라도 이해할 수 있기 때문이다.

---

## 2. Java 설치

### 방법 1) 공식 홈페이지를 통한 설치

[https://www.oracle.com/java/technologies/downloads](https://www.oracle.com/java/technologies/downloads) 에서 운영체제(Linux/macOS/Windows)와 시스템 환경(x64 등)에 맞는 JDK를 선택해 다운로드한다.

### 방법 2) Linux 패키지를 통한 설치 (WSL 등 Linux 환경)

```bash
# 1. 패키지 업데이트
sudo apt update

# 2. 자바 JDK 설치 (8 이상, 여기서는 17 사용)
sudo apt install openjdk-17-jdk
# 설치 중간에 확인 메시지가 나오면 Y 입력 후 엔터
```

아래와 유사한 메시지가 출력되면 설치가 완료된 것이다.

```
done.
done.
Processing triggers for libglib2.0-0:amd64 (2.72.4-0ubuntu2.2) ...
Processing triggers for libc-bin (2.35-0ubuntu3.6) ...
```

### 설치 확인

```bash
java -version
```

다음과 같이 출력되면 정상이다.

```
openjdk version "17.0.15" 2025-04-15
OpenJDK Runtime Environment (build 17.0.15+6-Ubuntu-0ubuntu122.04)
OpenJDK 64-Bit Server VM (build 17.0.15+6-Ubuntu-0ubuntu122.04, mixed mode, sharing)
```

---

## 3. Kafka 브로커 설치 (로컬 세팅)

### 다운로드

공식 다운로드 페이지([https://kafka.apache.org/downloads](https://kafka.apache.org/downloads))에서 직접 받거나, 터미널에서 `wget`으로 받을 수 있다.

```bash
wget https://dlcdn.apache.org/kafka/3.9.0/kafka_2.12-3.9.0.tgz
```

왼쪽과 같은 진행 로그가 출력되고 `saved` 메시지가 뜨면 다운로드 성공이다.

```
Resolving dlcdn.apache.org (dlcdn.apache.org)... 151.101.2.132, ...
Connecting to dlcdn.apache.org (dlcdn.apache.org)|151.101.2.132|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 122204110 (117M) [application/x-gzip]
Saving to: 'kafka_2.12-3.9.0.tgz'

kafka_2.12-3.9.0.tgz   100%[===================>] 116.54M  97.4MB/s   in 1.2s

'kafka_2.12-3.9.0.tgz' saved [122204110/122204110]
```

### 압축 해제 및 폴더 이동

```bash
# 압축 해제
tar -zxf kafka_2.12-3.9.0.tgz

# ls 했을 때 압축 풀린 폴더가 보이면 성공
ls
```

관리 편의를 위해 압축 해제한 폴더를 원하는 경로로 이동시켜 정리한다.

```bash
sudo mv kafka_2.12-3.9.0 /home/ssafy/kafka
```

---

## 4. Zookeeper 실행하기

```bash
# 1. Kafka 폴더로 이동
cd /home/ssafy/kafka

# 2. Zookeeper 실행 (zookeeper.properties 설정 파일 기반으로 실행)
./bin/zookeeper-server-start.sh config/zookeeper.properties
```

아래와 유사한 로그가 출력되며 계속 떠 있는 상태가 되면 성공이다.

```
[2025-03-06 12:02:33,691] INFO Reading configuration from: config/zookeeper.properties (org.apache.zookeeper.server.quorum.QuorumPeerConfig)
[2025-03-06 12:02:33,704] INFO clientPortAddress is 0.0.0.0:2181 (org.apache.zookeeper.server.quorum.QuorumPeerConfig)
...
```

---

## 5. Kafka 브로커 실행하기

Zookeeper를 켜둔 터미널은 그대로 두고, **별도의 터미널**을 새로 열어 진행한다.

```bash
# 1. Kafka 폴더로 이동
cd /home/ssafy/kafka

# 2. Kafka 브로커 실행 (server.properties 설정 파일 기반으로 실행)
./bin/kafka-server-start.sh config/server.properties
```

```
[2025-03-06 12:08:34,014] INFO Registered kafka:type=kafka.Log4jController MBean ...
[2025-03-06 12:08:34,764] INFO starting (kafka.server.KafkaServer)
[2025-03-06 12:08:34,765] INFO Connecting to zookeeper on localhost:2181 (kafka.server.KafkaServer)
...
```

---

## 6. Kafka 테스트 (토픽 생성 및 메시지 송수신)

Zookeeper, Kafka 브로커가 모두 떠 있는 상태에서 **또 다른 터미널**을 열어 진행한다.

```bash
cd /home/ssafy/kafka

# 1. 토픽 생성 (파티션 3개, 복제 개수 1개)
./bin/kafka-topics.sh --create --topic lecture-test-topic \
  --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1
# → Created topic lecture-test-topic.
```

> 복제 개수(replication-factor)는 지금 브로커가 1개만 떠 있는 상태이기 때문에 1로 설정한다. 브로커가 1개뿐이면 복제 자체가 불가능하다.

```bash
# 2. 메시지 전송 (Producer)
./bin/kafka-console-producer.sh --topic lecture-test-topic --bootstrap-server localhost:9092
>Hello!
```

```bash
# 3. 메시지 수신 (Consumer)
./bin/kafka-console-consumer.sh --topic lecture-test-topic \
  --from-beginning --bootstrap-server localhost:9092
Hello!
```

`--from-beginning` 옵션은 컨슈머가 붙은 시점 이후의 메시지만 읽는 게 아니라 **토픽의 맨 처음부터** 전부 읽어오겠다는 의미다. 컨슈머는 연결되어 있는 동안 계속 데이터를 요청하며 새로 들어오는 메시지도 실시간으로 가져온다.

---

## 7. Kafka를 Docker Compose로 구축하기

로컬 설치 대신, 아래처럼 `docker-compose.yml`을 작성해 Zookeeper와 Kafka를 한 번에 띄우는 것도 가능하다.

```yaml
services:
  zookeeper:
    container_name: zookeeper
    image: confluentinc/cp-zookeeper:7.5.0
    ports:
      - "2181:2181"
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

  kafka:
    container_name: kafka
    image: confluentinc/cp-kafka:7.5.0
    ports:
      - "9092:9092"
      - "29092:29092"
      - "9094:9094"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092,PLAINTEXT_HOST://0.0.0.0:29092
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT_HOST://localhost:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    depends_on:
      - zookeeper
```

### 포트가 두 개(9092 / 29092)인 이유

| 리스너 | 용도 |
|---|---|
| `PLAINTEXT://kafka:9092` | 같은 Docker 네트워크 안의 다른 컨테이너가 Kafka에 접근할 때 사용 (컨테이너 DNS 이름 `kafka`로 접근) |
| `PLAINTEXT_HOST://localhost:29092` | 호스트 PC(로컬)에서 직접 접근하거나, 클라이언트 코드/브라우저에서 접근할 때 사용 |

즉, 도커 내부 통신은 `kafka:9092`, 로컬(호스트)에서의 접근은 `localhost:29092`로 구분해서 사용한다. `KAFKA_LISTENERS`, `KAFKA_ADVERTISED_LISTENERS` 같은 환경 변수 설정은 Confluent에서 제공하는 이미지 문서를 기반으로 세팅된 값이므로, 더 자세히 알고 싶다면 Confluent의 이미지 문서를 참고하면 된다.

### 실행 및 테스트

```bash
# 1. Docker Compose 실행
docker compose up -d

# 2. 정상적으로 떠 있는지 확인
docker ps
```

로컬 PC에서 실행하되 대상이 Docker 컨테이너인 경우, `--bootstrap-server`를 호스트용 포트인 **29092**로 지정한다.

```bash
./bin/kafka-topics.sh --create --topic lecture-test-topic \
  --bootstrap-server localhost:29092 --partitions 3 --replication-factor 1
```

또는 컨테이너 내부로 직접 들어가 실행할 수도 있다. 이 경우 Confluent 이미지에 미리 세팅되어 있어 `.sh` 확장자 없이 바로 명령어를 실행할 수 있다.

```bash
docker exec -it kafka kafka-topics --create --topic lecture-test-topic \
  --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1
```

> 이미 존재하는 토픽명으로 다시 생성을 시도하면 `already exists` 오류가 발생한다. 이 경우 토픽명을 바꿔서 실행하면 된다.

### 참고: 토픽 삭제가 막혀 있는 경우

토픽 삭제는 `kafka-topics.sh`의 `--delete` 옵션으로 처리할 수 있는데(실무에서는 잘 사용하지 않지만, 실습 중 반복 테스트를 위해 필요할 수 있다), 만약 삭제가 되지 않는다면 `config/server.properties`에서 아래 옵션이 `false`로 되어 있지 않은지 확인한다.

```
delete.topic.enable=true
```

기본적으로는 막혀 있지 않지만, 혹시 삭제가 안 될 경우를 대비해 참고로 알아두면 좋다. 또한 실습 중 같은 토픽명을 삭제 후 재생성하다 보면, 메타데이터는 남아있는데 실제 데이터는 지워진 상태로 충돌이 나며 오류가 발생하는 경우가 있으니, 이럴 땐 토픽명을 바꿔서 실습하는 것을 권장한다.

---

## 💡 한 줄 요약
> Kafka는 로컬에 Java·Zookeeper·Kafka를 직접 설치해 띄우거나, Docker Compose로 Zookeeper와 Kafka 컨테이너를 함께 띄워 실행할 수 있으며, 두 방식 모두 토픽 생성 → Producer 전송 → Consumer 수신의 흐름으로 동작을 테스트한다.

## ❓ 더 찾아볼 것
- KRaft 모드로 Zookeeper 없이 Kafka 단독 구성하는 방법
- server.properties의 주요 설정값 (log.dirs, num.partitions, log.retention.hours 등)
- Kafka 클라이언트 라이브러리(Python confluent-kafka, kafka-python 등)로 Producer/Consumer 구현하기
