# AWS Cost Optimization Comparison

> Cost optimizationは、Discountを買うことではない。**Visibility → Waste elimination → Demand matching → Architecture optimization → Commitment**の順で進める。

## 基本フロー

```text
See the cost
  → Attribute the cost
  → Remove idle waste
  → Match capacity to demand
  → Reduce expensive paths
  → Buy commitments for stable baseline
  → Verify unit economics
```

---

## Service / Tool比較

| Tool | 主目的 | 得意 | 苦手 / 代替ではない |
|---|---|---|---|
| Cost Explorer | Interactive cost analysis | Trend、filter、forecast | Raw granular line-item分析 |
| Cost and Usage Report | Detailed usage dataset | Resource / usage type分析 | 即時Dashboard |
| AWS Budgets | Threshold alert / action | Actual、forecast、usage | Root-cause分析 |
| Cost Anomaly Detection | Unexpected pattern検出 | 異常支出 | Budget planning |
| Compute Optimizer | Rightsizing recommendation | Compute / volume等 | Business peak判断の代替 |
| Trusted Advisor | Best-practice checks | Idle / risk発見 | Full cost allocation |
| Pricing Calculator | Future architecture estimate | What-if model | Actual bill analysis |
| Savings Plans | Compute commitment discount | Flexible compute baseline | Rightsizing |
| Reserved Instances | Service-specific reservation discount | Stable service usage | Burst / unknown demand |
| Spot | Interruptible capacity | Batch、stateless | Stateful primary |
| S3 Storage Lens | S3 visibility | Usage、activity、optimization | Non-S3 cost |

---

# 1. Cost Explorer vs CUR

## Cost Explorer

- 月次 / 日次傾向
- Account / Service / Region / Tag filter
- Forecast
- RI / Savings Plans report

## CUR

- Detailed line items
- Usage type
- Operation
- Resource ID
- Amortized commitment
- Athena / BI分析

## 判断

```text
「どのServiceが増えた？」 → Cost Explorer
「どのResource・Usage type・Transfer path？」 → CUR
```

---

# 2. Budgets vs Anomaly Detection

## Budgets

Expected limitを超えるか監視。

例:

- 月100万円
- EC2 usage 10,000 hours
- Savings Plans coverage 80%未満

## Anomaly Detection

Historical patternから外れた急増を検出。

例:

- 通常1万円/日のWorkloadが突然8万円

## 組み合わせ

Budgetは計画超過、Anomalyは予期しない変化。

---

# 3. Rightsizing vs Commitment

## Rightsizing

必要CapacityへResourceを合わせる。

## Commitment

安定Baselineの利用額を期間契約してDiscountを得る。

正しい順序:

```text
Remove idle
  → Rightsize
  → Measure baseline
  → Commit
```

誤った順序:

```text
Oversized resource
  → Reserved purchase
  → waste locked for years
```

---

# 4. Savings Plans vs Reserved Instances

| 観点 | Savings Plans | Reserved Instances |
|---|---|---|
| Discount basis | Spend commitment | Service / instance attributes |
| Flexibility | Typeにより高い | Standard / Convertible等 |
| 主対象 | Compute usage | EC2 / RDS等Service別 |
| Capacity reservation | 別途確認 | Zonal EC2 RI等で特性あり |
| 選定 | Stable compute baseline | Stable specific usage |

問題文では、Flexibility、Capacity reservation、Service、属性変更可能性を見る。

---

# 5. Spot vs Auto Scaling

Spotは購入Option、Auto ScalingはCapacity制御。

組み合わせ:

```text
On-Demand baseline
  + Spot burst
  + Auto Scaling
  + SQS buffer
```

Auto Scalingだけでは単価は下がらず、SpotだけではDemand matchingにならない。

---

# 6. Schedule vs Auto Scaling vs Serverless

| Workload | Strategy |
|---|---|
| 平日9-18時のみ | Scheduled start/stop |
| Trafficに応じて連続変動 | Auto Scaling |
| Idleが多い短時間Event | Serverless |
| Constant high utilization | Reserved / Savings Plansを比較 |

「開発環境を夜間停止」はAuto ScalingのTraffic responseとは目的が違う。

---

# 7. Storage Optimization

## S3 Lifecycle

Access patternでClass移行。

## Intelligent-Tiering

Access patternが未知・変化する場合。

## EBS

- Unattached volume削除
- Snapshot retention
- gp3移行
- Provisioned IOPS見直し

## EFS / FSx

Storage classとThroughput capacityを確認。

Storage単価だけでなくRetrieval、Request、Minimum durationを見る。

---

# 8. Data Transfer Optimization

代表:

- S3 / DynamoDB Gateway Endpoint
- Interface EndpointとNATのBreak-even比較
- CloudFront cache
- Same-AZ pathの検討
- Cross-Region replication範囲
- Payload compression
- Batch / aggregation

詳しくは [Cost Modeling and Data Transfer](cost-modeling-and-data-transfer.md)。

---

# 9. Managed Service vs Self-managed

料金表上のComputeだけでなく次を含める。

- Patch
- Backup
- HA
- Monitoring
- Upgrade
- On-call
- License
- Failure recovery

## 強い制約

「最小の運用負荷」なら、単価が少し高いManaged serviceがTotal costで有利になり得る。

---

# 10. Unit Economics

```text
Cost per request
Cost per customer
Cost per order
Cost per GB processed
```

総額が増えても、Business volume以上に増えているかを見る。

例:

```text
Cost +20%
Orders +50%
→ cost per order improved
```

---

# 11. Cost Allocation

## Account

Strong boundary。

## Tag

Flexible metadata。

## Cost Category

Business rulesでAccount / Tagをまとめる。

## Shared cost

Network、Security、Logging、SupportをDriverで配賦。

- Processed GB
- Account count
- Actual usage
- Revenue ratio

---

# 12. SAP-C02の選択

| 問題文 | 選択 |
|---|---|
| 詳細なResource / Usage type分析 | CUR + Athena |
| 月額超過を通知 | Budgets |
| 予期しない急増 | Cost Anomaly Detection |
| EC2/EBSサイズ候補 | Compute Optimizer |
| Stable compute baseline | Savings Plans |
| Interruptible batch | Spot |
| S3 access pattern不明 | Intelligent-Tiering |
| Private subnetからS3大量転送 | Gateway Endpoint |
| Static content global配信 | CloudFront |

---

# 13. よくある誤答

- Savings PlansをRightsizingの代替にする
- BudgetをCost原因分析Toolとして使う
- Spotを中断不可DBへ使う
- VPC Endpointは必ずNATより安いと断定
- Single AZ化をCost optimizationとする
- Logを全削除してComplianceを壊す
- Backupを減らしてRPOを満たさない
- Compute Optimizerを無検証で適用
- Gross monthly billだけで評価する

---

# 14. 判断テンプレート

```text
Current cost:
Business volume:
Unit cost:
Top services:
Top usage types:
Idle waste:
Rightsizing opportunity:
Data transfer path:
Storage lifecycle:
Stable baseline:
Discount option:
Risk / availability constraint:
Expected saving:
Validation date:
```

## 完成した説明

> EC2は平均CPUが低いだけでなくPeak時も30%未満で、Memoryにも余裕があるためDownsize候補である。まずLoad testでSLOを確認してSizeを下げ、その後の安定BaselineにSavings Plansを適用する。現在のOversized利用額へ先にCommitするとWasteを固定化するため採用しない。

## 関連資料

- [Cost Modeling and Data Transfer](cost-modeling-and-data-transfer.md)
- [Cost Explorer](../services/cost/cost-explorer.md)
- [Budgets](../services/cost/budgets.md)
- [Savings Plans](../services/cost/savings-plans.md)
- [Reserved Instances](../services/cost/reserved-instances.md)
- [Compute Optimizer](../services/cost/compute-optimizer.md)
