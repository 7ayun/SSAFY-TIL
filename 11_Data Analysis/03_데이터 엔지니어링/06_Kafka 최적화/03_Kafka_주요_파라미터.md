# [데이터 엔지니어링] Kafka 주요 파라미터

---

## 1. Topic 및 Replica 관련 설정

| 파라미터 | 설명 | 기본값 |
|---|---|---|
| `num.partitions` | 토픽의 파티션 수. **한번 늘리면 다시 줄일 수 없음** | 1 (실무에서는 거의 사용하지 않음) |
| `replication.factor` | 파티션별 복제본(Replica) 수. 직접 변경 불가 → 재할당 방식으로만 변경 가능 | 1 (실무에서는 거의 사용하지 않음) |
| `min.sync.replicas` | 정상 동작(ISR)해야 하는 최소 replica 수 | - |

- 파티션 수는 늘릴 수는 있지만 줄일 수는 없다. 파티션이 많아지면 병렬 처리에는 유리하지만, 이를 관리하는 메타데이터 비용과 컨슈머 운영 비용도 함께 늘어나므로 무작정 늘리는 것은 지양해야 한다.
- `replication.factor`도 처음부터 신중하게 설정하는 것이 이상적이며, 이후 변경이 필요하면 `kafka-reassign-partitions.sh`로 재할당해야 한다.
- `min.sync.replicas=2`로 설정하면, replica 3개 중 2개가 in-sync 상태여야 메시지 전송이 성공으로 처리된다 (acks=all과 함께 사용).

```bash
# 토픽 생성 (파티션 3, 복제본 2)
kafka-topics.sh --create --topic my-topic \
  --partitions 3 --replication-factor 2 \
  --bootstrap-server localhost:9092

# 파티션 수 증가는 가능 (6개로 확장)
kafka-topics.sh --alter --topic my-topic \
  --partitions 6 --bootstrap-server localhost:9092

# 파티션 수 감소는 불가능 (에러 발생)
kafka-topics.sh --alter --topic my-topic \
  --partitions 1 --bootstrap-server localhost:9092   # ❌
```

```json
// replication.factor 변경을 위한 재할당 파일 예시 (reassign.json)
{
  "version": 1,
  "partitions": [
    {"topic": "my-topic", "partition": 0, "replicas": [1, 2, 3]},
    {"topic": "my-topic", "partition": 1, "replicas": [2, 3, 1]},
    {"topic": "my-topic", "partition": 2, "replicas": [3, 1, 2]}
  ]
}
```

```bash
kafka-reassign-partitions.sh --execute \
  --bootstrap-server localhost:9092 \
  --reassignment-json-file reassign.json
```

## 2. 네트워크 및 메모리 설정 최적화

| 파라미터 | 설명 | 기본값 |
|---|---|---|
| `socket.send.buffer.bytes` / `socket.receive.buffer.bytes` | 네트워크 송수신 버퍼 크기. `0`으로 설정하면 자동 조정(추천) | 100KB |
| `log.flush.interval.messages` / `log.flush.interval.ms` | 디스크로 로그를 플러시하는 주기 | Long.MAX_VALUE(사실상 특별한 요구가 없으면 기본값 사용 권장) |
| `message.max.bytes` | 브로커가 수용하는 메시지 최대 크기. 너무 크면 브로커 부하 발생, 10MB 이내 권장 | 1MB |
| `num.network.threads` | 네트워크 요청을 처리하는 스레드 수. 보통 CPU 코어 수와 비슷하게 설정 | 3 |

```properties
# config/server.properties
socket.send.buffer.bytes=512000
socket.receive.buffer.bytes=512000
num.network.threads=8

message.max.bytes=5242880   # 5MB까지 허용

log.flush.interval.messages=10000
log.flush.interval.ms=1000
```

## 3. 브로커 리소스 최적화

| 파라미터 | 설명 | 기본값 / 권장값 |
|---|---|---|
| `KAFKA_HEAP_OPTS` | Kafka가 JVM 기반으로 동작하기 때문에 필요한 힙 메모리 크기 설정 | 기본 1GB, 운영 시 4~8GB 권장 (16GB 이상은 GC 부담을 함께 고려해야 함) |
| `num.io.threads` | Disk I/O를 처리하는 스레드 수. CPU 코어 수에 맞춰 조정 | 기본 8 |
| `replica.fetch.min.bytes` | 팔로워가 리더로부터 복제해 오는 최소 데이터 크기. 1MB 정도 권장 | 기본 1 Byte |

```bash
# terminal
export KAFKA_HEAP_OPTS="-Xmx8G -Xms8G"   # 8GB 메모리 사용
```

```properties
# config/server.properties
num.io.threads=16                    # I/O 스레드 개수 증가 (HDD는 코어 수보다 적게, NVMe 등 고성능 장치는 그 이상)
replica.fetch.min.bytes=1048576      # 1MB 이상 모아서 복제 (값이 클수록 네트워크 효율은 좋아지지만 지연 시간은 늘어남)
```

## 4. ZooKeeper 관련 설정

| 파라미터 | 설명 | 기본값 |
|---|---|---|
| `maxClientCnxns` | ZooKeeper에 동시 연결 가능한 클라이언트 수. 브로커 1개당 약 20 정도 필요 | 60 |
| `syncLimit` | 리더-팔로워 간 최대 동기화 지연 허용 시간 | 10초 |
| `autopurge.snapRetainCount` | 보관할 snapshot 개수 | 3 |
| `autopurge.purgeInterval` | 오래된 로그/snapshot을 삭제하는 주기 | 24시간 |

- ZooKeeper도 무한정 많은 연결을 감당할 수 있는 것은 아니므로, 브로커 수에 맞춰 `maxClientCnxns`를 넉넉하게 잡아야 한다.
- 최근에는 ZooKeeper 대신 Kafka 자체 관리 방식인 KRaft로 전환하는 추세이며, 이 설정은 참고 수준으로 가볍게 다룬다.

```properties
# 단독 설치 시: zookeeper/conf/zoo.cfg
# 카프카 내장 주키퍼 사용 시: config/zookeeper.properties
syncLimit=5
maxClientCnxns=200
autopurge.snapRetainCount=3
autopurge.purgeInterval=24
```

## 5. Kafka 데이터 저장(Log) 방식 최적화

Kafka는 로그(메시지)를 Segment 단위로 저장한다. Segment 크기가 너무 크면 삭제·정리 시 불편하고, 너무 작으면 파일 수가 많아져 관리 비용(IO 부담)이 커지므로 운영 규모에 맞게 조정해야 한다.

| 파라미터 | 설명 | 기본값 |
|---|---|---|
| `log.retention.ms` | 로그 보관 시간. 장기 보관이 필요하면 별도 저장장치로 백업하는 것을 추천 | 7일 |
| `log.segment.bytes` | Segment 하나의 크기 | 1GB |
| `log.cleanup.policy` | 오래된 데이터 처리 방식: `delete`(삭제) 또는 `compact`(압축). 동시 설정도 가능 | `delete` |
| `log.cleaner.enable` | 키별로 최신 데이터만 남길지 여부(Log Compaction 활성화) | false |

```properties
# config/server.properties
log.retention.ms=604800000        # 7일
log.segment.bytes=1073741824      # 1GB
log.cleaner.enable=true
log.cleanup.policy=delete,compact
```

### Segment 압축 (Log Compaction)
- 앞서 다룬 프로듀서 단의 메시지 압축(Gzip/LZ4/Snappy)과는 **다른 개념**이다.
- 목표는 **동일한 key를 가진 메시지 중 가장 최신 값만 남기고 나머지는 삭제**하는 것이다.
- 사용자 상태 값처럼, 보관 기간(예: 7일)이 지나도 최신 상태는 계속 유지해야 하는 토픽에 적합하다. (예: 7일 이상 접속하지 않은 사용자라도 마지막 상태 정보는 남아 있어야 하는 경우)
- 상태 관리가 필요한 토픽에만 선택적으로 적용하는 것이 일반적이다.

```text
[압축 전]                                [압축 후]
offset=0 key=user123 value=로그인 성공
offset=1 key=user456 value=결제 완료
offset=2 key=user123 value=로그아웃
offset=3 key=user456 value=상품 조회      →   offset=3 key=user456 value=상품 조회
offset=4 key=user123 value=회원 탈퇴      →   offset=4 key=user123 value=회원 탈퇴
```
(동일 key의 과거 값은 사라지고, 각 key의 가장 최신 값만 남는다)

## 💡 한 줄 요약
> Topic/Replica, 네트워크·메모리, 브로커 리소스, ZooKeeper, Log 저장 방식까지 각 영역의 기본값과 트레이드오프(예: 처리 속도 vs 지연 시간, 파일 관리 비용 vs 세밀한 삭제)를 이해하고 운영 환경 규모에 맞춰 조정하는 것이 Kafka 파라미터 튜닝의 핵심이다.

## ❓ 더 찾아볼 것
- Log Compaction의 내부 동작(Tombstone, cleaner 스레드 동작 방식)
- JVM Garbage Collection과 KAFKA_HEAP_OPTS 크기의 관계
