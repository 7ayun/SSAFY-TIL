# [데이터 엔지니어링] Kafka Consumer

---

## 1. Consumer란? — 구독과 후속 처리

**컨슈머(Consumer)**는 Kafka 토픽의 데이터를 읽는(구독, Subscribe) 역할을 한다. Kafka가 데이터 수집·전달까지 처리해주면, 그 다음 "이 데이터를 어디로 가져가 어떻게 쓸 것인가"를 담당하는 것이 컨슈머다.

보통 컨슈머가 읽은 데이터는 ETL(Extract, Transform, Load) 과정을 거쳐 다른 시스템으로 넘어가거나, Apache Flink · Apache Spark · Apache Airflow 같은 데이터 파이프라인 처리 프레임워크를 통해 후속 처리가 이루어진다. 즉 컨슈머는 고정된 하나의 형태가 아니라, 목적에 따라 다양한 형태(모니터링 시스템, 분석 시스템, ML 학습 파이프라인 등)로 존재할 수 있다.

## 2. Consumer Lag — 컨슈머가 얼마나 밀렸는가

**컨슈머 랙(Consumer Lag)**은 프로듀서가 넣은 최신 메시지의 오프셋과, 컨슈머가 현재 읽고 있는 오프셋의 차이다. 즉 "아직 읽지 못한 메시지가 얼마나 쌓였는가"를 나타내는 지표다.

**`record-lag-max`**는 여러 파티션 중 가장 큰 랙 값을 의미한다. 예를 들어 파티션 0~3의 랙이 각각 3, 1, 2, 3이라면 `record-lag-max`는 3이 된다. 이 지표는 시스템이 데이터를 실시간에 가깝게 잘 처리하고 있는지, 아니면 병목이 발생하고 있는지를 판단하는 대표적인 모니터링 지표로 활용된다.

## 3. Consumer 핵심 특징 (Polling · 멀티 컨슈밍 · Consumer Group)

| 특징 | 설명 |
|---|---|
| Polling | 브로커가 데이터를 밀어주는(Push) 게 아니라, 컨슈머가 스스로 요청(Pull)해서 가져오는 구조. 컨슈머 자신이 처리 능력·시스템 부하 상황을 가장 잘 알기 때문에, 스스로 필요한 만큼 가져가는 방식이 더 효율적이다. |
| 멀티 컨슈밍 | 다른 메시징 시스템은 한 번 소비되면 데이터가 사라지는 경우가 많지만, Kafka는 로그 기반 구조라 그렇지 않다. 하나의 토픽 데이터를 여러 컨슈머가 동시에, 각자 독립적인 목적(예: 모니터링용 / ML 학습용)으로 소비할 수 있다. |
| Consumer Group | 여러 컨슈머를 묶어 논리적으로 하나의 소비자처럼 동작시키는 집단. 그룹으로 묶으면 파티션이 컨슈머들에게 자동으로 적절히 분배되어 수평 확장이 유연해진다. 컨슈머 수가 파티션 수보다 많으면 남는 컨슈머는 유휴(Idle) 상태가 된다. |

## 4. Fetch와 Commit

컨슈머 동작에서 중요한 두 가지 개념은 다음과 같다.

- **Fetch(페치)**: 컨슈머가 브로커로부터 레코드를 읽어오는 행위. 예를 들어 "파티션 1의 오프셋 100~109번 데이터를 주세요"라고 요청하고 데이터를 받는 것.
- **Commit(커밋)**: 특정 오프셋까지 실제 처리(DB 저장, 분석 시스템 전달 등)를 완료했다고 브로커에 선언하는 행위.

Fetch(읽어오기)와 Commit(처리 완료 선언)은 서로 다른 단계다. 커밋 정보를 저장해두는 이유는, 컨슈머가 장애로 멈췄다가 재시작할 때 **커밋된 오프셋 이후부터** 다시 읽을 수 있게 하기 위해서다. 커밋을 하지 않으면 이미 처리한 데이터를 중복으로 다시 읽거나, 반대로 처리하지 못한 데이터를 누락할 위험이 있다.

## 5. Group Coordinator와 Heartbeat

- **코디네이터(Coordinator)**: 컨슈머 그룹을 관리하는 주체로, 여러 브로커 중 하나가 그룹별로 지정된다. 코디네이터 프로세스는 해당 브로커 내부에서 항상 백그라운드로 동작한다. 컨슈머 그룹이 여러 개면, 그룹마다 담당 코디네이터 브로커가 각각 따로 존재한다.
- **하트비트(Heartbeat)**: 컨슈머 그룹들이 정상 동작 중인지 확인하는 신호로, Polling·Commit 시점마다 전송된다. 코디네이터가 하트비트를 감지하지 못하면 해당 컨슈머가 죽었다고 판단하고 그 순간부터 **리밸런싱**을 시작한다.

## 6. Consumer Rebalance — 발생 조건과 진행 과정

**리밸런싱(Rebalancing)**은 컨슈머 그룹의 변경(추가/제거/장애, 파티션 개수 변경)이 있을 때 파티션을 다시 배분하는 과정이다.

**발생 조건**
1. 새로운 컨슈머 추가 → 기존 파티션 일부를 새 컨슈머에 재할당
2. 컨슈머 제거(장애 포함) → 남은 컨슈머들이 기존 파티션을 나눠 담당
3. 파티션 개수 변경 → 전체 컨슈머에 대한 리밸런스 발생

**진행 과정**
1. 그룹 코디네이터가 모든 컨슈머의 파티션 소유권을 박탈하고 일시정지시킨다.
2. 컨슈머들의 `JoinGroup` 요청을 기다렸다가, 가장 빠르게 응답한 컨슈머를 **리더**로 선정한다.
3. 리더 컨슈머가 파티션 재분배 전략을 계산하고, 그 결과를 코디네이터에게 전달한다.
4. 코디네이터가 새로운 파티션 할당 정보를 각 컨슈머에게 전파한다.

## 7. 파티션 재할당 전략 (Assignor)

리밸런싱 시 파티션을 컨슈머에게 어떻게 나눠줄지 결정하는 전략(Assignor)에는 다음과 같은 방식들이 있다.

| 전략 | 방식 | 비고 |
|---|---|---|
| RangeAssignor | 토픽별로 파티션을 순서대로 나눔 | 과거 기본값 |
| RoundRobinAssignor | 모든 파티션을 훑으며 하나씩 고르게 분배 | |
| **StickyAssignor (현재 기본값)** | 이전 할당 정보를 최대한 유지하면서 최소한만 재배치 | 불필요한 재분배를 줄여, 리밸런싱 시 데이터 전송(파티션 이동)을 최소화 |

## 8. Kafka Transaction — Exactly Once를 위한 트랜잭션

Kafka에서 정합성이 중요한 경우(예: 주문 처리) 사용하는 **트랜잭션 프로듀서(Transaction Producer)** 개념이다.

동작 흐름:
1. 프로듀서가 **트랜잭션 시작**을 선언 — "지금부터 보내는 메시지는 한 묶음이다."
2. 여러 메시지를 브로커에 전송 (아직 미확정 상태)
3. 프로듀서가 **커밋 메시지**를 전송 → 브로커가 해당 묶음을 트랜잭션 완료 상태로 표시
4. 컨슈머는 **완료된 트랜잭션만** 읽는다 (완료되지 않은 메시지는 보이지 않음)

만약 일정 시간 안에 커밋 메시지가 도착하지 않으면 트랜잭션은 실패로 처리되고, 컨슈머는 해당 메시지를 처음부터 다시 받아 처리한다. 이를 통해 프로듀서-컨슈머가 연계된 EOS(Exactly Once Semantics)를 지킬 수 있다.

## 9. 코드로 살펴보는 Kafka Consumer

**kafka-python 방식**

```python
from kafka import KafkaConsumer

consumer = KafkaConsumer(
    'test-topic-1',
    bootstrap_servers='localhost:9092',
    auto_offset_reset='earliest',        # earliest: 처음부터 읽기, latest: 최신 메시지부터 읽기
    enable_auto_commit=True,             # 오프셋 자동 커밋 여부
    value_deserializer=lambda v: v.decode('utf-8')  # 저장된 바이트를 다시 문자열로 역직렬화
)

for message in consumer:
    print(message.value)
```

> kafka-python에서는 `group_id`가 필수는 아니며, 지정하지 않아도 개별적으로 동작할 수 있다(사실상 자기 자신만 속한 하나의 그룹으로 취급됨).

**confluent-kafka 방식** (더 세부적인 고급 설정이 필요할 때 사용)

```python
from confluent_kafka import Producer, Consumer

# --- Producer ---
def delivery_report(err, msg):
    if err is not None:
        print(f'Delivery failed: {err}')
    else:
        print(f'Delivered to {msg.topic()} [{msg.partition()}]')

p = Producer({'bootstrap.servers': 'localhost:29092'})
p.produce('test-topic-2', value='hello'.encode('utf-8'), callback=delivery_report)
p.poll(0)     # 콜백 처리를 위한 폴링
p.flush()     # 버퍼에 남은 메시지 모두 전송

# --- Consumer ---
c = Consumer({
    'bootstrap.servers': 'localhost:29092',
    'group.id': 'my-group',          # confluent-kafka는 group.id가 필수
    'auto.offset.reset': 'earliest'
})
c.subscribe(['test-topic-2'])

while True:
    msg = c.poll(1.0)
    if msg is None:
        continue
    print(msg.value().decode('utf-8'))
```

- confluent-kafka는 kafka-python과 달리 `group.id`를 반드시 지정해야 한다.
- Producer의 `produce()` 호출 시 등록한 콜백(`delivery_report`)이 내부 전송 큐 처리(polling) 시점마다 호출되어 성공/실패, 전송된 토픽·파티션 정보를 확인할 수 있다.
- 실습 수준에서는 kafka-python이 더 간단해 우선 권장되고, confluent-kafka는 더 다양한 고급 설정이 필요할 때 사용한다.

---

## 💡 한 줄 요약
> Kafka Consumer는 Polling 방식으로 브로커에서 데이터를 능동적으로 가져오며, Consumer Group과 Group Coordinator를 통해 파티션을 자동 분배·재분배(리밸런싱)하고, Fetch/Commit 단계 분리와 트랜잭션 기능으로 안정적인 메시지 소비를 보장한다.

## ❓ 더 찾아볼 것
- Cooperative Sticky Assignor(Incremental Rebalancing)와 기존 Eager Rebalancing의 차이
- `read_committed` vs `read_uncommitted` isolation level
- kafka-python과 confluent-kafka의 실제 성능·기능 비교
