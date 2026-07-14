# Messaging and Eventing Comparison

> SQS、SNS、EventBridge、Step Functions、Kinesis、Amazon MQは、すべて「何かを次へ渡す」ように見える。しかし、**保存するのか、配るのか、振り分けるのか、順序を管理するのか、Streamingするのか、既存Protocolを維持するのか**が違う。

## 最初の判断

```text
一対一で処理を待たせる         → SQS
一対多へPushする              → SNS
Event内容で複数Targetへ振分け  → EventBridge
処理順序・分岐・状態を管理      → Step Functions
連続Data streamを保持・再生     → Kinesis Data Streams / MSK
既存JMS/AMQP/MQTT broker互換    → Amazon MQ
```

---

## 比較表

| Service | 主目的 | Data保持 | Consumer model | 順序 | Replay | 状態管理 |
|---|---|---|---|---|---|---|
| SQS Standard | Buffer / work queue | Message削除まで | Pull | Best effort | DLQ / redrive中心 | なし |
| SQS FIFO | Ordered work queue | Message削除まで | Pull | Message group内 | DLQ / redrive | なし |
| SNS | Pub/Sub fanout | 配信中心 | Push | Standardでは保証しない | 原則Subscriber側 | なし |
| EventBridge | Event routing | Archive optional | Push to targets | Event順序を主目的にしない | Archive / Replay | なし |
| Step Functions | Workflow orchestration | Execution history | State transition | 明示的 | Retry / execution rerun | あり |
| Kinesis Data Streams | Streaming log | Retention期間 | Pull / enhanced fan-out | Shard内 | Retention内で再読込 | Consumer側 |
| Data Firehose | Delivery pipeline | Buffer後配送 | Managed delivery | 分析用途の厳密順序ではない | Source次第 | なし |
| MSK | Kafka streaming | Topic retention | Consumer group | Partition内 | Offsetで再生 | Consumer側 |
| Amazon MQ | Broker互換 | Queue / topic | Protocol依存 | Broker機能 | Broker機能 | なし |

---

# 1. SQS

## 役割

ProducerとConsumerの速度差を吸収し、Consumerが停止してもMessageを保持する。

```text
Producer
  → SQS Queue
      → Consumer polls
          → process
          → delete
```

## 選ぶ条件

- Work queue
- Burst absorption
- Retry
- DLQ
- Consumerを独立Scale
- SenderがReceiver完了を待たない

## Standard

- 高Throughput
- At-least-onceを前提
- Duplicateを考慮
- 厳密順序を前提にしない

## FIFO

- Message Group内順序
- Deduplication
- Throughput制約を確認

## 重要設計

- Visibility Timeout
- Long polling
- DLQ
- Redrive
- Idempotency
- Age of oldest message

---

# 2. SNS

## 役割

一つのMessageを複数SubscriberへPushする。

```text
Publisher
  → SNS Topic
      ├─ SQS A
      ├─ SQS B
      ├─ Lambda
      └─ HTTP / email等
```

## 選ぶ条件

- Fanout
- Multiple subscribers
- Notification
- Topic-based pub/sub

## SNS + SQS

各Subscriberの前にSQSを置くと、Subscriber停止中のBuffer、Retry、独立Scaleが可能。

```text
SNS
  ├─ SQS for Billing
  ├─ SQS for Analytics
  └─ SQS for Email
```

SNSだけで長期BufferとConsumer pullを代替しない。

---

# 3. EventBridge

## 役割

Event patternを評価してTargetへRouteする。

```text
Event source
  → Event bus
      → rule pattern
          → target
```

## 選ぶ条件

- AWS service events
- SaaS integration
- Custom application events
- Content-based routing
- Cross-account event bus
- Schedule / Scheduler
- Archive / Replay

## EventBridgeとSNS

| 観点 | EventBridge | SNS |
|---|---|---|
| Routing | Event content pattern | Topic subscription / filter |
| Event source | AWS / SaaS / custom | PublisherがTopicへ送信 |
| Archive replay | 対応 | Subscriber側で設計 |
| Fanout | Ruleで複数Target | Topicで自然なfanout |
| 主用途 | Event backbone | Notification pub/sub |

## EventBridgeからECS

定刻またはEventでECS RunTaskを直接Targetにできる。外部HTTP API公開要件がなければAPI Gatewayは不要。

---

# 4. Step Functions

## 役割

複数Stepの順序、分岐、並列、待機、Retry、Catch、補償を管理する。

```text
Start
  → Validate
  → Parallel
      ├─ Reserve inventory
      └─ Authorize payment
  → Choice
  → Complete / Compensate
```

## 選ぶ条件

- Workflow state
- Human approval / wait
- Saga
- Parallel processing
- Visual execution history
- Complex retry policy

## EventBridgeとの違い

- EventBridge: Eventをどこへ送るか
- Step Functions: 処理をどの順番で完了させるか

EventBridgeが複数Targetへ送っても、全処理の完了順序や補償を自動管理しない。

---

# 5. Kinesis Data Streams

## 役割

高頻度の連続RecordをShardへ保存し、複数ConsumerがRetention内で読む。

```text
Producers
  → Stream / shards
      → Consumer A
      → Consumer B
```

## 選ぶ条件

- Streaming ingestion
- Ordered records within shard
- Multiple independent consumers
- Replay
- Near-real-time analytics

## SQSとの違い

| 観点 | SQS | Kinesis Data Streams |
|---|---|---|
| 処理Model | Work queue | Shared stream log |
| 一件の処理 | 通常一Consumerが処理 | 複数Consumerが同じRecordを読める |
| 削除 | Consumerがdelete | Retentionで消える |
| 順序 | FIFO group | Shard内 |
| Replay | 主目的ではない | Retention内で可能 |

---

# 6. Data Firehose

## 役割

Streaming DataをBufferし、S3、Redshift、OpenSearchなどへManaged deliveryする。

## 選ぶ条件

- Consumer codeを持ちたくない
- DestinationへBatch delivery
- Transformation
- Compression / format conversion

Custom low-latency consumerや任意Replayが中心ならKinesis Data Streamsと比較する。

---

# 7. MSK

## 役割

Managed Apache Kafka。Topic、Partition、Consumer Group、Offsetを利用する。

## 選ぶ条件

- Existing Kafka ecosystem
- Kafka API / connector
- Long retention streaming
- Cross-platform portability
- High throughput event log

## EventBridgeとの違い

EventBridgeはManaged event routing。MSKはKafka broker platformであり、Cluster capacity、Partition、Consumer lagなどを設計する。

---

# 8. Amazon MQ

## 役割

ActiveMQ / RabbitMQ等の既存Broker互換をManagedで提供する。

## 選ぶ条件

- Existing applicationがJMS / AMQP等に依存
- Code changeを最小化
- Broker semanticsが必要

Cloud-native新規設計ではSQS/SNS/EventBridgeを先に検討するが、互換性が強い制約ならAmazon MQが適する。

---

# 9. 同期APIとの比較

## 同期

```text
A → B
A waits for B
```

長所:

- 即時Result
- Simple transaction flow

短所:

- B障害がAへ伝播
- Timeout
- Burstを吸収しにくい

## 非同期

```text
A → Queue/Event → B later
```

長所:

- Decoupling
- Buffer
- Retry
- Independent scaling

短所:

- Eventual completion
- Duplicate
- Ordering
- Observability
- Compensation

---

# 10. Delivery semantics

## At-most-once

Duplicateを避けるが、Failure時にLossの可能性。

## At-least-once

Lossを減らす代わりにDuplicateの可能性。Idempotencyが必要。

## Exactly-once

End-to-endで実現するには、BrokerだけでなくConsumerのSide effectとData storeまで設計する。Service marketing用語だけで判断しない。

---

# 11. Idempotency

同じRequestを複数回処理しても結果が重複しない性質。

```text
Message ID
  → processed table / conditional write
      ├─ new → execute
      └─ exists → skip / return previous result
```

Payment、Order、Inventoryでは特に重要。

---

# 12. RetryとDLQ

```text
Process failed
  → retry with backoff
      → max attempts
          → DLQ
```

DLQは墓場ではない。

- Alarm
- Reason analysis
- Fix
- Redrive
- Poison message isolation

Visibility Timeoutは最大処理時間より短過ぎても長過ぎても問題になる。

---

# 13. Ordering

Global strict orderingはThroughputとAvailabilityを制約する。

必要な範囲だけ順序を求める。

例:

```text
Order IDごとの順序は必要
全顧客のOrderを一列に並べる必要はない
```

SQS FIFO Message GroupやKinesis Partition KeyをBusiness keyへ合わせる。

---

# 14. Fanout Pattern

## SNS fanout

Notification中心。

## EventBridge fanout

Content routing、AWS/SaaS event backbone。

## Kinesis fanout

同じStreaming recordを複数Consumerが独立処理。

用途を区別する。

---

# 15. Saga

複数ServiceのTransactionをLocal transactionと補償処理へ分ける。

```text
Reserve inventory
  → Charge payment
      → Create shipment

Failure:
  → Refund payment
  → Release inventory
```

Step FunctionsはOrchestration型Sagaに向く。Event choreographyではEventBridge/SNS等を使うが、Flow全体の追跡が難しくなる場合がある。

---

# 16. Observability

追跡する。

- Correlation ID
- Event ID
- Causation ID
- Timestamp
- Producer version
- Schema version
- Retry count
- DLQ reason

非同期では一つのHTTP request logだけで全体を追えない。

---

# 17. Schema evolution

Safe change:

- Optional field追加
- Consumerが未知Fieldを無視
- Versionを明示

危険:

- Required field突然追加
- Field意味変更
- Type変更
- Old consumerを考慮せず削除

ProducerとConsumerを同時Deployできない前提で互換性を保つ。

---

# 18. SAP-C02選択表

| 問題文 | 第一候補 | 理由 |
|---|---|---|
| Worker停止中も依頼を保持 | SQS | Durable work queue |
| 複数Subscriberへ通知 | SNS | Pub/Sub fanout |
| Event contentでTarget振分け | EventBridge | Pattern routing |
| 複数Step、分岐、補償 | Step Functions | Workflow state |
| Continuous stream、Replay | Kinesis / MSK | Retained stream log |
| Existing JMS broker migration | Amazon MQ | Protocol compatibility |
| 定刻にECS Task起動 | EventBridge Scheduler | Direct target |
| S3へManaged delivery | Data Firehose | Delivery pipeline |

---

# 19. よくある誤答

- SQSだけで複雑なWorkflow stateを管理する
- EventBridgeを長期Work queueとして扱う
- SNSだけでSubscriber停止中の独立Bufferを保証する
- Step Functionsを高Throughput event brokerとして扱う
- Firehoseを任意ConsumerがReplayするStreamとして扱う
- FIFOを使えばApplication側Idempotencyが一切不要
- DLQへ送ればFailure対応完了
- Retryを無制限に増やす
- Strict global orderingを不要なWorkloadへ要求する

---

# 20. 設計テンプレート

```text
Producer:
Consumer:
Payload:
Expected rate:
Burst:
Retention:
One or multiple consumers:
Push or pull:
Ordering scope:
Duplicate tolerance:
Replay:
Retry:
DLQ:
Workflow state:
Schema evolution:
SLO:
```

## 完成した説明

> 注文APIは決済Workerの完了を待つ必要がなく、Peak時の到着率が処理率を超えるため、SQSで要求を保持する。Standard Queueの重複配信を前提にOrder IDでIdempotencyを実装し、処理不能MessageはDLQへ送る。メールや分析への複数配信はEventBridgeまたはSNSへ分離し、補償処理を含む決済FlowはStep Functionsで管理する。

## 関連資料

- [EventBridge](../services/integration/eventbridge.md)
- [SQS](../services/integration/sqs.md)
- [SNS](../services/integration/sns.md)
- [Step Functions](../services/integration/stepfunctions.md)
- [Amazon MQ](../services/integration/mq.md)
- [Kinesis](../services/analytics/kinesis.md)
- [MSK](../services/analytics/msk.md)
- [Managed Service for Apache Flink](../services/analytics/flink.md)
