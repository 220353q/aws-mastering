# Cost Modeling and Data Transfer

> AWSコスト最適化は、単価の安いサービスへ置き換えることではない。**利用量を分解し、費用が発生する境界を特定し、要件を保ったまま無駄と不整合を減らすこと**である。

SAP-C02では、EC2料金だけでなく、Data transfer、NAT Gateway、AZ間通信、Storage request、Backup、Log retention、Discount commitmentまで含めて判断する。

```text
Business unit
  → Workload
  → Resource
  → Usage quantity
  → Rate
  → Discount
  → Shared cost allocation
  → Unit economics
```

---

# 1. コストを五種類に分ける

| 種類 | 例 | 改善方法 |
|---|---|---|
| Provisioned capacity | EC2、RDS、EBS容量 | Rightsizing、Schedule、Auto Scaling |
| Consumption | Lambda request、Athena scan | Request削減、Data layout、Batch化 |
| Data transfer | Internet、AZ間、Region間、NAT | 経路変更、Cache、Endpoint、配置 |
| Operations | 人の作業、Patch、障害対応 | Managed service、自動化、標準化 |
| Risk | Downtime、Data loss、Compliance | 適切な冗長化とControl |

料金だけ下げてOperationsやRiskを増やすと、Total costは下がらない場合がある。

---

# 2. 基本式

```text
Monthly cost
  = Fixed cost
  + Usage quantity × Unit price
  + Data transfer
  + Request / operation
  + Support / license
  - Discounts
```

## Unit cost

```text
Cost per order
Cost per active user
Cost per processed GB
Cost per inference
```

総額だけでなく、Business単位あたりCostを見る。利用者増加で総額が増えてもUnit costが下がっていれば効率は改善している可能性がある。

---

# 3. Visibilityの順序

```text
Consolidated billing
  → Account
  → Cost Category
  → Tag
  → Service
  → Usage type
  → Resource
  → Business metric
```

## Account

強い配賦境界。Production、Development、Security、Shared Servicesなど。

## Tag

柔軟な分類。

例:

- application
- environment
- owner
- cost-center
- product

## Cost Category

AccountやTag、Charge typeなどをBusinessルールでまとめる。

## CUR

最も詳細なUsage record。Athena等で分析する。

---

# 4. Cost and Usage Report

CURでは、次のような切り口を見る。

- line item usage type
- operation
- resource ID
- linked account
- region
- pricing term
- reservation / savings plan benefit
- credits / refunds
- amortized cost

## CURが必要になる場面

- NAT Gateway processingの送信元を絞る
- Savings Plans適用状況を詳細分析
- Shared service費用を配賦
- Resource単位の月次推移
- Data transfer chargeをUsage type別に調査

Cost Explorerは素早い探索、CURは詳細分析に向く。

---

# 5. Amortized cost

前払い費用を利用期間へ配分して見る。

```text
Upfront payment
  → commitment periodへ配分
```

Cash支出と利用原価を分ける。Reserved InstanceやSavings Plansの比較ではAmortized viewが役立つ。

---

# 6. ShowbackとChargeback

## Showback

各部門へ費用を見せるが、実際の請求移転はしない。

## Chargeback

各部門へ費用を配賦・請求する。

## Shared cost

Network、Security、Support、Loggingなどは共有される。

配賦方法:

- Equal split
- Revenue ratio
- Headcount
- Actual usage
- Hybrid

技術的に正確でも、Businessが理解できない配賦は運用されない。

---

# 7. Rightsizing

## Step 1: Idleを消す

- 停止中だが課金されるVolume
- Unattached EBS
- Idle Load Balancer
- Unused Elastic IP
- Old snapshot
- Forgotten development environment

## Step 2: Demandに合わせる

- Instance family変更
- Size変更
- Auto Scaling
- Schedule
- Serverless
- Storage class

## Step 3: Commitment

- Savings Plans
- Reserved Instances

過剰Resourceに先にCommitすると無駄を固定化する。

---

# 8. Savings Plans

一定の計算利用額を期間Commitし、対象Compute利用へDiscountを適用する。

判断:

- Baseline usage
- Coverage
- Utilization
- Term
- Payment option
- Flexibility

## Coverage

Eligible usageのうちDiscount対象になった割合。

## Utilization

購入Commitmentのうち実際に利用した割合。

高CoverageでもUtilizationが低ければ買い過ぎの可能性がある。

---

# 9. Reserved Instances

Serviceと属性に応じたDiscount / capacity reservation特性を持つ。

確認:

- Standard / Convertible
- Regional / Zonal scope
- Instance attributes
- Term
- Payment
- Modification / exchange

RDS等のReserved optionもEC2と完全に同じではない。対象Serviceの条件を確認する。

---

# 10. Spot

余剰Capacityを低Costで使うが中断され得る。

適する:

- Stateless worker
- Batch
- Queue consumer
- Distributed processing
- Checkpoint可能

適さない:

- 中断不可Single node
- Long transaction without recovery
- Stateful primary

## 設計

- Multiple instance types
- Multiple AZs
- Capacity-optimized allocation
- Checkpoint
- SQS
- Graceful interruption handling
- On-Demand baseline

---

# 11. Data Transferの境界

Data transfer costは、「誰から誰へ」「どの方向」「どの境界を越えるか」で決まる。

```text
Source
  → same AZ?
  → another AZ?
  → another Region?
  → Internet?
  → AWS service through NAT?
```

## 調査テンプレート

```text
Source service:
Source AZ / Region:
Destination service:
Destination AZ / Region:
Direction:
Monthly GB:
Path:
Intermediary:
Current charge type:
Alternative path:
```

---

# 12. Same-AZ / Cross-AZ

Cross-AZ trafficは可用性のため必要な場合があるが、Data transfer chargeとLatencyが増える可能性がある。

例:

```text
EC2 AZ-a
  → NAT Gateway AZ-b
  → Internet / AWS service
```

この構成では、Cross-AZとNAT処理が重なる可能性がある。

## 原則

- NAT Gatewayは各AZに配置し、同AZ private subnetから利用
- AZ affinityだけを優先してReliabilityを落とさない
- Managed serviceの課金モデルを確認

Cost削減のためにSingle AZへ集約し、Availability要件を壊さない。

---

# 13. NAT Gateway Cost

NAT Gatewayでは主に次を考える。

- 時間料金
- Data processing
- Cross-AZ path
- Destination側Data transfer

## 典型的な無駄

Private subnetからS3やDynamoDBへの大量TrafficをNAT経由。

改善:

- S3 Gateway Endpoint
- DynamoDB Gateway Endpoint

## Interface Endpointとの比較

Interface EndpointにはEndpoint時間料金とData処理がある。NATより常に安いとは限らない。

比較:

```text
NAT hourly + NAT processing + transfer
vs
Endpoint hourly × AZ count + endpoint processing
```

Traffic量、AZ数、利用Service数で変わる。

---

# 14. VPC Endpoint

## Gateway Endpoint

- S3
- DynamoDB
- Route table連携
- Endpoint自体の時間料金がない代表的方式

## Interface Endpoint

- PrivateLink
- ENI / private IP
- Security Group
- AZごとのEndpoint
- Hourly + processing

EndpointはSecurityとNetwork path改善にもなる。Costだけで判断しない。

---

# 15. CloudFront

CloudFrontでCache hitが増えると、次を減らせる。

- Origin request
- Origin compute
- Origin data transfer
- Global latency

## Cost要素

- Edge data transfer
- Request
- Invalidation
- Origin shield等

Dynamic / uncacheable trafficではCache効果が限定される。Cache keyを増やし過ぎるとHit ratioが下がる。

---

# 16. Region間転送

Cross-Region replicationやGlobal architectureでは継続転送費用が発生する。

例:

- S3 CRR
- Aurora Global Database
- DynamoDB Global Tables
- Cross-Region backup copy
- Inter-Region VPC connectivity

## 判断

- RPO / RTOに必要か
- 全Dataか重要Dataだけか
- Compression / batching可能か
- Read trafficをLocal化できるか

DR要件がないのに全Dataを複製しない。一方、Cost削減のためRPOを満たさない設計にしない。

---

# 17. Internet Egress

外向きData transferは大きなCostになり得る。

改善候補:

- CloudFront
- Compression
- Image / video optimization
- Cache-Control
- Smaller payload
- Client-side pagination
- Regional architecture
- Direct Connect pricing comparison for on-prem transfer

Dataを減らすことが最も根本的な改善になる場合がある。

---

# 18. Direct ConnectとCost

DXにはPort hour、Data transfer、Provider費用などがある。

選定はCostだけでなく:

- Bandwidth
- Predictability
- Latency consistency
- Availability
- Lead time
- Encryption

大量継続転送ではInternetと比較する価値があるが、少量Trafficでは固定費が上回る場合がある。

---

# 19. Storage Cost

## S3

- Stored GB
- Request
- Retrieval
- Minimum duration
- Data transfer
- Lifecycle transition

## EBS

- Provisioned GB
- Provisioned IOPS / throughput
- Snapshot
- Fast Snapshot Restore等

## EFS

- Storage class
- Throughput mode
- IA access

## FSx

- Storage
- Throughput capacity
- Backup
- File system type

容量単価だけで比較せず、Request、Retrieval、Performanceを含める。

---

# 20. S3 Lifecycle

```text
Frequent
  → Infrequent
  → Archive
  → Delete
```

確認:

- Access pattern
- Minimum storage duration
- Retrieval time
- Retrieval charge
- Object size
- Compliance retention

小さなObjectを大量にArchiveすると、Metadata / request / minimum chargeの影響を確認する。

---

# 21. Database Cost

## RDS / Aurora

- Instance / ACU
- Storage
- I/O model
- Backup
- Data transfer
- Replica

## DynamoDB

- Read / write capacity or on-demand
- Storage
- Backup
- Streams
- Global Tables replication
- GSI

## ElastiCache

- Node
- Data transfer
- Backup

Database比較ではCostだけでなく、運用、Data model、Performanceを含める。

---

# 22. Serverless Cost

## Lambda

- Request
- Duration
- Memory
- Provisioned concurrency

## API Gateway

- Request
- Data transfer
- Cache等

## Step Functions

- State transitionまたは実行方式に応じた課金

## Serverlessが有利になりやすい

- Idleが多い
- Burst
- 短時間
- 運用人員が少ない

## 比較が必要

- Constant high utilization
- Long running
- High request volume
- Large data transfer

「Serverless = 常に安い」ではない。

---

# 23. LogsとObservability Cost

- Log ingestion
- Storage
- Query scan
- Metric cardinality
- Trace sampling
- Retention

## 改善

- Log level
- Sampling
- Retention by environment
- Archive to S3
- Structured logs
- Query partition

必要なAudit logをCostだけで削除しない。用途とRetention policyを分類する。

---

# 24. Backup Cost

- Backup storage
- Incremental behavior
- Cross-Region copy
- Cross-account copy
- Retention
- Restore test environment

Backup policyを「毎日永久保存」にしない。Legal、RPO、Retentionを分ける。

---

# 25. Shared Service Cost

例:

- Transit Gateway
- Network Firewall
- Central log
- Security tooling
- Direct Connect
- DNS Resolver Endpoint

Shared serviceは利用Accountへ配賦しないと、WorkloadのUnit costが見えない。

配賦Driver:

- Attachment count
- Processed GB
- Query count
- Account count
- Actual finding volume

---

# 26. Managed ServiceのTCO

Self-managed EC2が料金表では安く見えても、次を含める。

- Patch
- Upgrade
- Backup
- Monitoring
- On-call
- Capacity planning
- License
- Failure recovery

```text
TCO = AWS bill + labor + license + downtime risk + migration cost
```

SAP-C02では「最小の運用負荷」が強い制約なら、多少単価が高くてもManaged serviceが正解になる。

---

# 27. Cost Guardrail

## Budgets

Actual / forecast cost、Usage、Commitment coverage等のAlert。

## Cost Anomaly Detection

通常Patternから外れた支出を検出。

## SCP

特定RegionやResource typeを制限できるが、Cost toolではない。Business要件を阻害しないよう設計する。

## Service Quotas

Quotaは予期しない大量利用の完全なCost controlではない。

## Tag Policy

Tag標準を促進するが、Cost allocation tag activationや実際の入力品質も必要。

---

# 28. Cost Optimization Cycle

```text
Measure
  → Attribute
  → Find waste
  → Model alternatives
  → Change
  → Verify
  → Commit discount
```

## Verify

- Total cost
- Unit cost
- Performance
- Availability
- Operator hours

請求額だけで成功判定しない。

---

# 29. ケース: NATからS3

## Current

```text
Private EC2
  → NAT Gateway
  → S3
```

月20TB。

## Candidate

S3 Gateway Endpoint。

## 効果

- NAT processing削減
- Internet path不要
- Private route

## 確認

- Route table
- Endpoint policy
- S3 bucket policy
- DNS / region
- Cross-Region S3 access

---

# 30. ケース: Multi-AZ NAT

## Current

```text
Subnets AZ-a / AZ-b / AZ-c
  → one NAT Gateway in AZ-a
```

## 問題

- Cross-AZ charge
- AZ-a障害
- Central bottleneck

## Candidate

AZごとにNAT Gateway。

## Tradeoff

- Hourly cost増加
- Cross-AZ減少
- Availability改善

Traffic量が少ない小規模環境では、費用比較が必要。Reliability要件も含める。

---

# 31. ケース: Savings Plans

## Current

Compute利用が毎時変動するが、最低50ドル/時を下回らない。

## 判断

BaselineへCommitし、PeakはOn-Demand / Spot等で対応する。

## 避ける

Peak 100ドル/時に合わせて全額Commit。低負荷時間にUtilizationが下がる。

---

# 32. ケース: CloudFront

## Current

Global userがALBから同じ画像を繰り返しDownload。

## Candidate

CloudFrontでCache。

## Model

```text
Current:
Origin requests + origin compute + internet egress

After:
Edge requests + edge egress + cache miss origin cost
```

Cache hit ratioを仮定し、Request / Transferを比較する。

---

# 33. ケース: Cross-Region DR

## Requirement

RPO 15分、RTO 2時間。

## Candidate

- Continuous full active-active
- Warm standby
- Backup copy + restore

Active-activeは要件を超えてCost過剰の可能性がある。RPO/RTOを満たす最小構成を選ぶ。

---

# 34. SAP-C02誤答

- Cost ExplorerだけでResource-level詳細分析が常に完了する
- Savings PlansをRightsizingの代替にする
- SpotをStateful primaryへ単独利用
- NAT Gatewayを一つに集約すれば常に安い
- VPC Endpointは全Traffic量で必ずNATより安い
- CloudFrontは全Dynamic trafficをCacheできる
- Single AZ化をCost optimizationとする
- Backup retentionを要件確認せず削る
- Managed serviceの人件費削減を無視する
- Cross-Region replicationを無料の可用性機能と考える

---

# 35. モデリングテンプレート

```text
Business metric:
Current monthly usage:
Peak / baseline:
Fixed cost:
Variable cost:
Data transfer path:
Shared cost:
Discount coverage:
Operational effort:
Risk requirement:
Candidate A:
Candidate B:
Break-even point:
Selected option:
Validation:
```

# 36. 一文の完成形

> Private subnetからS3へ月20TBを送るTrafficがNAT Gateway Data processingの主要因であるため、S3 Gateway Endpointへ経路を変更する。Endpoint policyとBucket policyでAccessを制限し、NAT processed bytesとApplication成功率を比較する。Interface EndpointはAZごとの時間料金があるため、このS3用途では同じものとして選ばない。

## 関連資料

- [Cost Explorer](../services/cost/cost-explorer.md)
- [Budgets](../services/cost/budgets.md)
- [Savings Plans](../services/cost/savings-plans.md)
- [Reserved Instances](../services/cost/reserved-instances.md)
- [Compute Optimizer](../services/cost/compute-optimizer.md)
- [Amazon VPC](../services/networking/vpc.md)
- [CloudFront](../services/networking/cloudfront.md)
- [S3](../services/storage/s3.md)
