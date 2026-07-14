# AWS Performance Design Reader

> 高性能なサービスを選ぶのではない。**どこが遅く、どの資源が飽和し、どのアクセスパターンを、どの方法で改善するか**を設計する。

SAP-C02では、EC2を大きくする、Cacheを置く、Replicaを増やす、といった単純な選択ではなく、症状と原因を対応付ける必要がある。

```text
Business objective
  → measurable metric
  → workload model
  → bottleneck
  → candidate design
  → load test
  → cost / reliability tradeoff
```

---

# 1. 性能用語を分ける

## Latency

一回の処理にかかる時間。

```text
Request start → Response complete
```

見る値:

- Average
- p50
- p90
- p95
- p99
- Maximum

AverageだけではTail latencyを隠す。

## Throughput

一定時間に処理できる量。

- requests/sec
- records/sec
- MB/sec
- transactions/sec

## IOPS

1秒あたりのI/O操作回数。小さなRandom I/Oで重要。

## Bandwidth

1秒あたりに運べるData量。大きなSequential I/Oで重要。

## Concurrency

同時に処理中のWork数。

## Utilization

Resourceがどの程度使われているか。

## Saturation

Resource上限へ近づき、待ち行列が増えている状態。

## Queueing

処理能力より到着量が多いと待ち時間が増える。

```text
Arrival rate > Processing rate
  → backlog grows
  → latency grows
```

---

# 2. 性能目標をBusiness要件へ結び付ける

悪い目標:

- 速くする
- 高性能にする

良い目標:

- Peak 2,000 req/sでp95 300ms未満
- 10TBの夜間Batchを4時間以内
- 動画Upload後15分以内に95%を処理完了
- 1秒以内にGlobal userへStatic contentを配信
- RPO 1秒未満を維持しつつRead latencyを50ms未満

性能は、LoadとPercentileと時間帯を含める。

---

# 3. Workload Model

## Traffic shape

- Constant
- Daily peak
- Seasonal
- Burst
- Unpredictable
- Batch window

## Request characteristic

- Read-heavy / Write-heavy
- Small random / Large sequential
- Short / Long running
- CPU-bound / Memory-bound / I/O-bound
- Stateful / Stateless
- Synchronous / Asynchronous

## Data characteristic

- Size
- Growth rate
- Access frequency
- Consistency
- Partition key
- Hot data / Cold data

Workloadを分類してからServiceを選ぶ。

---

# 4. BottleneckをLayerで切り分ける

```text
Client
  → DNS / Edge
  → Load Balancer
  → Compute
  → Queue / Stream
  → Cache
  → Database / Storage
  → External API
```

## Client / Network

- Geographic distance
- Packet loss
- TLS handshake
- DNS latency
- Large payload

## Edge

- Cache hit ratio
- Origin latency
- Connection reuse

## Load Balancer

- Target response time
- Unhealthy target
- Connection count
- Cross-zone traffic

## Compute

- CPU
- Memory
- GC
- Thread / worker pool
- Network
- Disk

## Integration

- Queue age
- Consumer lag
- Retry storm
- Throttle

## Data

- Query latency
- Lock
- Connection
- Cache hit
- Read / write IOPS
- Replication lag

---

# 5. Little's Lawの直感

安定した系では、概ね次の関係になる。

```text
Concurrency = Throughput × Latency
```

例:

```text
100 requests/sec × 0.2 sec = 20 concurrent requests
```

Latencyが2秒へ増えると、同じThroughputでもConcurrencyは200になる。Connection、Thread、Memoryを圧迫する。

性能問題では、遅延が原因で同時実行数が増え、さらに遅くなる循環を見る。

---

# 6. EC2 Instance Familyの選び方

| 特性 | 候補の考え方 | 例 |
|---|---|---|
| General purpose | CPU/Memoryの均衡 | Web、App server |
| Compute optimized | CPU比率が高い | Encoding、Simulation、HPC |
| Memory optimized | 大量Memory | In-memory DB、Analytics |
| Storage optimized | Local high I/O | Search、Distributed DB |
| Accelerated computing | GPU/Accelerator | ML、Rendering |

Instance名の暗記より、Bottleneckに合うFamilyを選ぶ。

## Vertical scaling

Instanceを大きくする。

長所:

- 変更が単純
- Stateful / legacyに使いやすい

短所:

- 上限
- Cost
- Restart
- Single node dependency

## Horizontal scaling

Instance数を増やす。

長所:

- Elastic
- Fault tolerance
- 段階拡張

短所:

- State externalization
- Distribution
- Coordination

---

# 7. Auto Scaling

## Target tracking

Target metricを一定に保つ。

例:

- CPU 50%
- ALB request count per target

## Step scaling

Alarm程度に応じて段階的に増減。

## Scheduled scaling

予測できる時間帯へ事前にCapacityを用意。

## Predictive scaling

過去Patternを使い、需要前にScale。

## Warm-up

新Instanceが正常処理できるまでの時間を考慮する。起動が遅いWorkloadでReactive scalingだけでは間に合わない。

## Scale metric

CPUがWork量と相関しない場合は、業務Metricを使う。

```text
SQS backlog per worker
ALB request per target
Jobs pending
```

---

# 8. Placement Groups

## Cluster

Instanceを近く配置し、低Latency・高帯域通信へ。

- HPC
- tightly coupled workload

Risk:

- 同一Hardware障害領域への集中
- Capacity確保

## Spread

少数の重要Instanceを異なるHardwareへ分散。

- Failure isolation

## Partition

Large distributed systemをPartition単位でHardware分離。

- Hadoop
- Cassandra系

Placementは性能と障害分離のTradeoffを見る。

---

# 9. Network Performance

確認:

- Instance network bandwidth
- PPS
- ENA
- MTU
- Cross-AZ
- Cross-Region
- Internet path

## CloudFront

HTTP(S) contentをEdge cache。Cache hitでOrigin latencyとLoadを減らす。

## Global Accelerator

Anycast IPからAWS global networkへ入り、TCP/UDPをRegional endpointへ転送。Cacheはしない。

## Direct Connect

On-premとの一貫したNetwork path。Application performanceはEnd-to-endで確認する。

---

# 10. EBS性能

EBSでは三つを分ける。

- Volume IOPS
- Volume throughput
- EC2 instance EBS bandwidth

Volumeを高性能化しても、Instance側帯域が上限なら性能は増えない。

## gp3

容量とIOPS・Throughputを独立設定でき、一般用途の第一候補になりやすい。

## io2系

高IOPS、低Latency、高耐久性要件。

## st1

大きなSequential throughput向け。Boot volumeには使わない。

## sc1

低頻度、低CostのThroughput用途。

## Snapshot restore

Snapshotからの復元後はBlockが初回Access時に読み込まれる場合がある。Fast Snapshot Restoreや事前初期化を検討する。

---

# 11. EFS性能

確認:

- General Purpose / Max I/OなどのPerformance特性
- Bursting / provisioned / elastic throughput
- File count
- Small file
- Metadata operation
- Mount targetとAZ

大量Small fileとMetadata-heavy処理では、単純な容量比較で決めない。

---

# 12. FSx

## FSx for Lustre

High-throughput parallel file system。HPC、ML、S3 dataset処理。

## FSx for Windows File Server

SMB、Windows、Active Directory。

## FSx for ONTAP

NFS / SMB / iSCSI、NetApp機能。

## FSx for OpenZFS

OpenZFS互換。

性能だけでなくProtocolと既存運用を選定軸にする。

---

# 13. S3性能

S3はObject storageであり、Block/File systemと同じI/O modelではない。

改善:

- Parallel request
- Multipart upload
- Byte-range fetch
- Transfer Acceleration
- CloudFront
- Appropriate prefix / key distribution

Small object大量処理ではRequest costとOverheadを見る。

---

# 14. Database選定

## Relational

- Join
- Transaction
- Referential integrity
- Existing SQL

候補: RDS / Aurora。

## Key-value / Document

- Known access pattern
- Massive scale
- Low latency

候補: DynamoDB。

## Cache

- Repeated read
- Session
- Ranking

候補: ElastiCache。

## Graph

Relationship traversal。Neptune。

## Search

Full text、Log analytics。OpenSearch。

## Analytics DWH

Large aggregation。Redshift。

Purpose-built databaseは、Storage engineの特性へWorkloadを合わせる。

---

# 15. RDS / Aurora Performance

## Vertical scale

DB instance classを変更。

## Read scale

Read Replica / Aurora Reader。

## Connection management

RDS Proxy / application connection pool。

## Query improvement

- Index
- Query plan
- Lock reduction
- N+1 elimination
- Partitioning

## Cache

ElastiCache。Stale dataとInvalidationを設計。

## Aurora Serverless v2

変動Capacityへ細粒度に対応。Minimum capacity、Connection、Scale behaviorを確認する。

DBを大きくする前に、Slow queryとLockを見る。

---

# 16. DynamoDB Performance

DynamoDBの性能はPartition key設計が中心。

## Hot partition

一部KeyへTraffic集中。

悪い例:

```text
Partition key = current_date
```

全Writeが同じ日付へ集中する。

改善:

- High-cardinality key
- Write sharding
- Composite key
- Adaptive capacityを理解

## Capacity mode

### On-demand

予測困難・変動Workload。

### Provisioned

予測可能でAuto Scalingや予約Capacityを活用。

## GSI

新しいAccess patternを提供するが、追加Storage・Write cost・Propagationを伴う。

## DAX

Microsecond級Read cache用途。Write-heavyや複雑Queryの解決策ではない。

---

# 17. Cache Pattern

## Cache-aside

```text
App
  → cache lookup
      ├─ hit → return
      └─ miss → DB
                 → cache store
                 → return
```

## Write-through

Write時にCacheも更新。

## TTL

古いDataを許容できる時間。

## Invalidation

Cacheで最も難しい部分。

確認:

- Stale tolerance
- Stampede
- Hot key
- Eviction
- Failure behavior

## Cache stampede

TTL切れで多数Requestが同時にDBへ行く。

対策:

- Jittered TTL
- Request coalescing
- Lock
- Stale-while-revalidate

---

# 18. Buffering

SQSやKinesisで急増を後段から分離する。

```text
Burst input
  → buffer
  → steady consumers
```

Bufferは処理を速くするのではなく、**到着速度と処理速度の差を吸収する**。

## Tradeoff

- End-to-end latency増加
- Eventual completion
- Retry / duplicate
- Monitoring backlog

---

# 19. SQS Capacity Design

概算:

```text
Required consumers
  ≈ Arrival rate × Processing time per message ÷ Parallelism per consumer
```

例:

```text
100 msg/s × 0.5s = 50 concurrent processing slots
```

見るMetric:

- ApproximateNumberOfMessagesVisible
- ApproximateAgeOfOldestMessage
- MessagesSent / Deleted
- DLQ

Queue depthだけでは、Message sizeと処理時間を反映しない。

---

# 20. Kinesis Shardの考え方

必要Shard数は、WriteとReadの両方の制約から大きい方を選ぶ。

```text
Write shards = max(records/sec制約, MB/sec制約)
Read shards  = consumer modelとread throughputから算定
```

正確なQuotaは最新Documentationで確認する。試験では、Record rate、Data rate、Consumer lagを分ける。

Hot shardを避けるためPartition keyの分散を確認する。

---

# 21. Lambda Performance

確認:

- Duration
- Memory
- CPU allocation
- Cold start
- Concurrency
- Throttle
- Downstream capacity

LambdaはMemory設定に伴ってCPU能力も変化するため、Memoryを増やすとDurationが短縮し、総Costが下がる場合がある。Power tuningで比較する。

## Concurrency control

- Reserved concurrency
- Provisioned concurrency

Provisioned concurrencyはCold start低減。Reserved concurrencyは上限確保・制限。

Downstream DBが処理できないConcurrencyを許可しない。

---

# 22. ECS / EKS Performance

- Task / Pod CPU and memory
- Request / limit
- Node capacity
- Horizontal scaling
- Placement
- Image pull
- Startup time
- ENI / IP
- Load balancer target

EKSではNode、Pod、Cluster Autoscaler/Karpenter等の複数Scale layerを分ける。

FargateではNode管理を減らせるが、Workload制約とCostを確認する。

---

# 23. Batch Performance

AWS Batch等で見る。

- Queue priority
- Compute Environment
- Job definition
- vCPU / memory
- Array job
- Multi-node
- Spot interruption

Batch window内完了が目標なら、1Jobの速さだけでなく並列数とStartup overheadを見る。

---

# 24. Tail Latency

p99悪化の原因:

- GC pause
- Slow query
- Cold start
- Retry
- Network packet loss
- Noisy neighbor
- Lock contention
- Large payload

平均値が正常でも一部Userに大きな影響がある。

## Hedged request

非常に重要なReadで、一定時間を超えたら別Replicaへ同じRequestを送る設計もある。ただしLoad増加と重複処理に注意する。

---

# 25. Retryと性能

Retryは成功率を上げる一方、障害中のLoadを増やす。

```text
Failure
  → immediate retry by many clients
  → dependency overloaded
  → more failure
```

対策:

- Exponential backoff
- Jitter
- Maximum attempts
- Timeout
- Circuit breaker
- Retry budget

Write操作はIdempotencyを設計する。

---

# 26. Load Test

## 種類

- Baseline
- Load
- Stress
- Spike
- Soak
- Failover

## Production-like

- Data size
- Access distribution
- Cache warm/cold
- Connection
- External dependencies

Small test dataではIndexやCache behaviorが本番と異なる。

## Guardrail

- Test account
- Cost limit
- Abort condition
- Quota
- Data safety

---

# 27. Performance Testの判定

```text
Objective:
Load:
Duration:
Success criteria:
Metrics:
Cost:
Failure behavior:
```

変更前後を同条件で比較する。

- p95 / p99
- Throughput
- Error
- Saturation
- Cost per transaction

速くなってもCostが10倍、Reliabilityが低下するなら採用しない場合がある。

---

# 28. Rightsizing

## Oversized

- 低Utilization
- High idle cost

## Undersized

- Saturation
- Throttle
- Queue
- Performance SLO violation

## 判断

Peakだけでなく周期を確認する。

- Downsize
- Schedule
- Auto Scale
- Family change
- Architecture change

Compute OptimizerのRecommendationをそのまま適用せず、Business peakとMemory visibilityを確認する。

---

# 29. Service Quotas

性能設計にはQuotaが含まれる。

- Lambda concurrency
- API rate
- ENI
- VPC endpoint
- EIP
- EC2 capacity
- RDS connection

ArchitectureがScale可能でもQuotaへ達すれば止まる。

```text
Expected peak
  < configured quota
  < tested capacity
```

Quota increaseのLead timeを計画する。

---

# 30. CostとのTradeoff

## Cache

DB Loadを減らすがCache node費用とInvalidationが増える。

## Replica

Read scaleとAvailabilityを改善するがReplication costとLagがある。

## Multi-AZ

Availability目的。Read performance改善と決めつけない。

## Provisioned capacity

Predictable performanceだがIdle cost。

## Serverless

Variable workloadで有利。Constant high usageでは比較が必要。

---

# 31. SAP-C02症状別判断

## Read-heavy DBが遅い

- Query / Index
- Read Replica / Aurora Reader
- ElastiCache
- DAX for DynamoDB

Write bottleneckにはRead Replicaを選ばない。

## BurstでAPIがTimeout

- Auto Scaling warm-up
- SQS buffer
- Lambda concurrency
- Downstream limit

## Global HTTP latency

- CloudFront
- Multi-Region origin
- Route 53 latency routing

Static / cacheableかを確認する。

## TCP/UDP global endpoint、固定IP

Global Accelerator。

## EC2 disk latency

- Volume IOPS / throughput
- Instance EBS bandwidth
- Queue length
- File system

## LambdaからRDS connection枯渇

RDS Proxy / Connection Pool / Concurrency control。

## DynamoDB throttle

Partition key、Hot key、Capacity mode、GSIを確認。

---

# 32. よくある誤答

- CPUが低いから性能問題ではない
- Average latencyだけを見る
- Multi-AZでRead性能が倍になる
- ReplicaをBackupとして使う
- Cacheを置けば整合性設計不要
- Queueを置けば処理自体が高速化する
- EBS Volumeだけ高性能化し、EC2側帯域を見ない
- Auto Scalingを設定すれば起動時間を無視できる
- Retryを増やせばReliabilityとPerformanceが両方改善する
- DynamoDBはPartition設計なしで無限に均等Scaleする
- Rightsizingを単純なDownsizeと考える

---

# 33. 設計テンプレート

```text
Business objective:
Traffic pattern:
Data size / growth:
Read / write ratio:
Latency target:
Throughput target:
Concurrency:
Current bottleneck:
Evidence:
Candidate options:
Selected design:
Scaling metric:
Quota:
Load test:
Cost per unit:
Failure behavior:
```

# 34. 完成した説明

> Peak時の遅延はEC2 CPUではなく、SQSの到着率100 msg/sに対してWorker処理率が60 msg/sしかないことが原因である。Age of oldest messageがSLOを超えているため、Backlog per taskを指標にECS TaskをScaleし、下流DBのConnection上限を超えない最大Task数を設定する。処理自体のLatencyも計測し、必要ならBatch writeとIndexを改善する。

この説明ができれば、「高性能Serviceを選ぶ」段階から「Systemの処理能力を設計する」段階へ進んでいる。

## 関連資料

- [EC2](services/compute/ec2.md)
- [Lambda](services/compute/lambda.md)
- [ECS](services/compute/ecs.md)
- [EKS](services/compute/eks.md)
- [EBS](services/storage/ebs.md)
- [EFS](services/storage/efs.md)
- [FSx](services/storage/fsx.md)
- [RDS](services/database/rds.md)
- [Aurora](services/database/aurora.md)
- [DynamoDB](services/database/dynamodb.md)
- [ElastiCache](services/database/elasticache.md)
- [CloudWatch](services/management/cloudwatch.md)
