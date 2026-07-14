# Continuous Improvement Playbook

> 既存システムの改善は、思いついたサービスを追加することではない。**観測し、症状を定義し、原因を切り分け、改善し、効果を検証する反復プロセス**である。

SAP-C02 Domain 3では、完成済みアーキテクチャを見て「どこが悪いか」「何から直すか」「改善が成功したとどう判断するか」を問われる。

```text
Observe
  → Define
  → Diagnose
  → Prioritize
  → Change
  → Verify
  → Standardize
  → Repeat
```

---

# 1. 改善前に作る現状モデル

最低限、次を一枚にする。

```text
User / System
  → Entry
  → Compute
  → Integration
  → Data
  → External dependency

Control plane:
  IAM / Deployment / Configuration / Monitoring

Failure path:
  Retry / DLQ / Failover / Backup / Restore
```

## 収集する情報

- Business critical path
- Dependency map
- Data ownerと正本
- Traffic pattern
- Peak / average / seasonal variation
- RTO / RPO
- Current SLO / SLA
- Deployment frequency
- Incident history
- Cost by account / service / workload
- Manual operation
- Service quotas

構成図だけでなく、**時間変化と障害時動作**を確認する。

---

# 2. 症状を測定可能な言葉へ変える

悪い例:

- 遅い
- 不安定
- 高い
- 運用が大変

良い例:

- 平日12時にCheckout APIのp99が4秒を超える
- 月に2回、DB connectionが上限へ達して5xxが発生する
- NAT Gateway経由のS3転送が月額の18%を占める
- 毎週6時間、担当者がPatch適用と確認を手動実施する
- Region障害時の復旧手順が未検証で、RTOを証明できない

改善は、測定可能な症状から始める。

---

# 3. SLI・SLO・SLA・KPI

## SLI

実測する指標。

例:

- Request success rate
- p95 latency
- Queue age
- Replication lag
- Restore time

## SLO

目標値。

例:

- 30日間で99.9%のRequestを成功させる
- p95を300ms未満にする

## SLA

顧客との契約上の保証。SLOと同じとは限らない。

## KPI

事業成果を測る。

- Checkout completion
- Payment success
- Active users
- Processing completion time

## Error Budget

SLOが100%でない場合に許容される失敗量。

```text
SLO 99.9%
  → 0.1%の失敗を許容
  → Budgetを使い切ったらRelease速度よりReliabilityを優先
```

---

# 4. 観測可能性の三本柱

## Metrics

集約された数値。傾向とAlarmに向く。

- CPUUtilization
- RequestCount
- Error rate
- Queue depth
- Latency

## Logs

個別EventとContext。原因調査に向く。

- Request ID
- User / Tenant
- Error message
- Input metadata
- Decision path

## Traces

分散処理の呼び出し関係と所要時間。

```text
Client
  → API Gateway 20ms
  → Lambda 80ms
  → RDS 900ms  ← bottleneck
```

CloudWatchだけ、Logsだけ、Traceだけで全てを解決しない。

---

# 5. Monitoring設計

## Golden Signals

- Latency
- Traffic
- Errors
- Saturation

## Workload固有指標

### Web/API

- Request count
- p50 / p95 / p99
- 4xx / 5xx
- Target response time
- Active connection

### Queue worker

- Queue depth
- Age of oldest message
- Arrival rate
- Processing rate
- DLQ count

### Database

- Connection count
- CPU / Memory
- Read / write latency
- Lock wait
- Replication lag
- Buffer cache hit ratio

### Batch

- Job duration
- Success / failure
- Pending jobs
- Cost per completed unit

### Streaming

- Incoming records
- Consumer lag
- Iterator age
- Throttle
- Delivery failure

---

# 6. Alarm設計

Alarmは「値が高い」ではなく、Actionにつなげる。

```text
Signal
  → Threshold / anomaly
  → Alarm state
  → Notification / Automation
  → Verification
```

## Alarmに必要な情報

- 何が起きたか
- User impact
- First action
- Dashboard / Runbook
- Owner
- Escalation

## Static threshold

明確な上限がある場合。

- Connection > 90%
- DLQ > 0
- Free storage < 10%

## Anomaly detection

日周期・週周期があり、固定値では誤Alarmが多い場合。

## Composite alarm

複数Signalを組み合わせる。

```text
High latency AND high error rate
```

Deploy直後の一時的CPU上昇だけでIncidentにしない。

---

# 7. 自動修復

```text
Detection
  → EventBridge
  → Lambda / Systems Manager Automation
  → Remediation
  → Verification
  → Notification
```

## 自動化しやすい処理

- Unhealthy Instanceの置換
- Service restart
- Security Group ruleの除去
- Public S3設定の修復
- Patch適用
- Snapshot取得
- Failed jobの再実行

## 自動化しにくい処理

- Data corruption判断
- Unknown root cause
- Irreversible change
- Large-scale failover
- Legal / compliance判断

## Safety

- Idempotency
- Maximum retry
- Scope limit
- Dry-run
- Approval gate
- Audit log
- Rollback
- Post-check

「自動化できる」と「自動実行して安全」は別である。

---

# 8. RunbookとPlaybook

## Runbook

特定Operationの具体手順。

```text
RDS storage full対応
1. Alarm確認
2. Growth確認
3. Emergency expansion
4. Application確認
5. Root cause analysis
```

## Playbook

複数状況に使う判断体系。

```text
Latency incident
  → entry / compute / dependency / dataの順に切り分け
```

Runbookは「操作」、Playbookは「判断」に強い。

---

# 9. Operational Excellence改善

## 手動作業を分類する

| 種類 | 例 | 改善 |
|---|---|---|
| Repeatable | Patch、定期停止 | Automation |
| Error-prone | 手入力、Copy | IaC / validation |
| Slow approval | 定型変更 | Risk-based approval |
| Hidden knowledge | 個人メモ | Runbook |
| Reactive | 障害後確認 | Proactive monitoring |

## Systems Manager

- Run Command
- Automation
- Patch Manager
- State Manager
- Session Manager
- Parameter Store
- Maintenance Windows

SSH接続を前提にせず、中央管理とAuditを設計する。

## IaC

Manual resource creationを減らし、Desired stateとReview可能な変更へ移す。

---

# 10. Deployment改善

確認する。

- Release frequency
- Change failure rate
- Lead time
- Mean time to restore
- Rollback成功率
- Manual step数

改善候補:

- Automated test
- Artifact immutability
- Blue/Green
- Canary
- Change Set
- Automatic rollback
- Feature flag

詳しくは [Deployment and Rollback Strategies](comparisons/deployment-and-rollback-strategies.md) を参照する。

---

# 11. Security改善

## 11.1 Identity

- Long-term access key
- Shared account
- Excessive administrator
- Unused permission
- Cross-account trust
- MFA

改善:

- IAM Identity Center
- Role assumption
- Access Analyzer
- Permission review
- Short-lived credentials

## 11.2 Data

- Classification
- Retention
- Encryption
- Backup
- Access log
- Deletion protection

## 11.3 Detection

- CloudTrail
- Config
- GuardDuty
- Inspector
- Macie
- Security Hub

## 11.4 Remediation

```text
Finding
  → Security Hub
  → EventBridge
  → Automation
  → Ticket / notification
```

## 11.5 Patch

- Asset inventory
- Patch baseline
- Maintenance Window
- Staging
- Rollback / replacement
- Compliance report

Immutable replacementとIn-place patchを、Workloadに応じて選ぶ。

---

# 12. Performance改善

```text
Business symptom
  → Metric
  → Layer isolation
  → Bottleneck
  → Candidate
  → Load test
  → Compare
```

## Layer isolation

1. Client / network
2. Edge / load balancer
3. Compute
4. Queue / stream
5. Database / cache
6. External dependency

## Bottleneckの例

| 症状 | 可能性 | 確認 |
|---|---|---|
| CPU高騰 | Compute不足 | CPU、run queue |
| CPU低いが遅い | I/O待ち、lock、external | latency内訳 |
| Queue age増加 | Consumer不足 | arrival vs processing |
| DB connection枯渇 | connection storm | connection、RDS Proxy |
| p99のみ悪化 | tail latency | trace、GC、slow query |
| Region遠方だけ遅い | network distance | CloudFront / GA |

詳しくは [Performance Design Reader](PERFORMANCE_DESIGN_READER.md) を参照する。

---

# 13. Reliability改善

## Single Point of Failure

- Single AZ
- Single NAT instance
- One database without failover
- One network path
- One DNS resolver
- One manual operator
- One shared secret

技術Resourceだけでなく、人とProcessもSPOFになる。

## Self-healing

- Auto Scaling health replacement
- ECS desired count
- Route 53 failover
- RDS failover
- SQS retry
- Lambda retry / DLQ

Self-healingには検知と正常状態の定義が必要である。

## Quota

GrowthでQuotaに達すると、Resourceが正常でもScaleできない。

- EC2 On-Demand limits
- ENI
- Lambda concurrency
- API Gateway throttle
- VPC / TGW limits

Quota usageを監視し、Increase lead timeを考慮する。

## Dependency failure

- Timeout
- Retry with backoff and jitter
- Circuit breaker
- Fallback
- Bulkhead
- Queue buffering

Retryだけを増やすと、障害中の依存先へ負荷を集中させる。

---

# 14. BackupとRestore改善

Backupの存在だけでなく、復元可能性を確認する。

```text
Backup policy
  → successful copy
  → retention
  → protected account / vault
  → restore test
  → application validation
```

## 確認項目

- RPOを満たす頻度
- RTOを満たす復元時間
- Cross-account / Cross-Region
- Immutability
- Encryption key
- Restore permission
- Dependency順序
- DNS / connection切替

Restore testなしではRTOを証明できない。

---

# 15. DR演習

## Tabletop

文書と役割を確認する。安価だが、技術動作は検証できない。

## Component test

DB restore、DNS切替など一部を試す。

## Simulation

Productionに近い環境でFailoverを検証する。

## Full interruption

実際にPrimaryを停止する。最も現実的だがRiskが高い。

## Game Day

- Hypothesis
- Scope
- Guardrail
- Observation
- Abort condition
- Learning
- Action item

FISは障害注入に使えるが、Application全体のBusiness validationは別途必要である。

---

# 16. Cost改善

## 四段階

```text
Visibility
  → Eliminate waste
  → Match demand
  → Discount
```

### Visibility

- Account
- Tag
- Cost Category
- CUR
- Unit cost

### Eliminate waste

- Unattached EBS
- Old snapshot
- Idle LB
- Unused EIP
- Over-retained logs

### Match demand

- Rightsizing
- Auto Scaling
- Schedule
- Storage tiering
- Serverless

### Discount

- Savings Plans
- Reserved Instances
- Spot

割引購入前に無駄を減らす。過剰Resourceを予約すると無駄を固定化する。

詳しくは [Cost Modeling and Data Transfer](comparisons/cost-modeling-and-data-transfer.md) を参照する。

---

# 17. Prioritization

全て同時に直さない。

## Risk score例

```text
Priority = Impact × Likelihood × Exposure ÷ Effort
```

## 優先しやすい改善

- Security critical finding
- Data loss risk
- RTO/RPO violation
- Frequent customer impact
- Repeated manual incident
- High-cost low-effort waste

## 後回しにしやすい改善

- 見た目だけの統一
- Root cause不明の大規模Rewrite
- Measurable outcomeがない新Service導入

---

# 18. 改善案の比較表

| Candidate | Impact | Risk | Cost | Time | Reversibility | Evidence |
|---|---:|---:|---:|---:|---:|---|
| Add cache | 高 | 中 | 中 | 短 | 高 | DB read latency |
| DB scale-up | 中 | 低 | 高 | 短 | 高 | CPU / memory |
| Schema redesign | 高 | 高 | 中 | 長 | 低 | Query plan |
| Queue decoupling | 高 | 中 | 中 | 中 | 中 | Burst / timeout |

Evidenceがない改善案は仮説として扱う。

---

# 19. 検証

## Before / After

- 同じ負荷
- 同じ時間帯
- 同じData volume
- 同じSuccess criteria

## Technical result

- latency
- error
- saturation
- recovery time

## Business result

- completion rate
- support ticket
- operator hours
- unit cost

## Negative result

改善しなかった場合も記録する。仮説が外れたことは学習である。

---

# 20. Standardize

改善が有効なら、一回限りで終わらせない。

- IaC module
- Organization policy
- Service Catalog
- StackSets
- Runbook
- Dashboard template
- Alarm standard
- Golden path

標準化により、次のAccount / Workloadへ同じ改善を適用する。

---

# 21. ケーススタディ: LambdaからRDSへの接続障害

## 症状

Traffic急増時に5xx。Lambda concurrencyとRDS connectionが同時に増える。

## 観測

- Lambda concurrent execution
- RDS connection count
- DB CPU
- Connection error
- Query latency

## 仮説

短時間に大量Connectionが作られ、DBの接続上限と接続確立処理がBottleneck。

## 候補

1. RDSをScale-up
2. Lambda concurrencyを制限
3. RDS Proxy
4. Query cache

## 判断

Connection stormが主因ならRDS Proxyを導入し、Concurrency上限もDB処理能力に合わせる。Query latency自体が問題ならIndexやQuery改善を別途行う。

## 検証

- Peak connection
- Connection error
- p95 latency
- Failover時のRecovery

---

# 22. ケーススタディ: SQS backlog

## 症状

Queue depthが増え続け、処理完了がSLOを超える。

## 見る式

```text
Backlog growth = Arrival rate - Processing rate
```

## 対応

- Consumer countを増やす
- 1件あたり処理時間を減らす
- Batch receive
- Downstream bottleneckを解消
- Poison messageをDLQへ分離

Queue depthだけでなくAge of oldest messageを見る。

---

# 23. ケーススタディ: 高いNAT Gateway費用

## 症状

Cost ExplorerでData processing chargeが増加。

## 調査

- VPC Flow Logs
- CUR
- Destination service
- AZ配置

## 典型原因

Private subnetからS3 / DynamoDBへNAT経由で大量転送。

## 改善

Gateway Endpointを使い、Route tableを更新する。

## 検証

- NAT processed bytes
- Endpoint bytes
- Application connectivity
- Cost

---

# 24. SAP-C02の誤答パターン

- Monitoringを追加するだけで自動復旧したと考える
- AlarmにActionとOwnerがない
- Retry回数を増やしてReliabilityを改善する
- Backup取得だけでDR完了とする
- Rightsizing前に長期割引を購入する
- CPUだけで全Performanceを判断する
- ConfigをAPI監査Serviceとして扱う
- CloudTrailをResource configuration complianceとして扱う
- FISを無GuardrailでProductionへ実行する
- 新Service導入自体を改善成果とする

---

# 25. 改善提案テンプレート

```text
Business symptom:
Current architecture:
Evidence:
SLO / constraint:
Root-cause hypothesis:
Candidate options:
Selected change:
Why not alternatives:
Risk:
Rollback:
Validation metrics:
Owner:
Standardization:
```

## 一文の完成形

> Checkout遅延はEC2 CPUではなくRDS lock waitとconnection増加に相関しているため、Instance追加では解消しない。QueryとIndexを改善し、RDS ProxyでConnection burstを吸収する。p95、lock wait、connection errorを同じ負荷条件で比較し、改善しなければ変更を戻す。

この説明ができれば、既存システム改善をService選択問題ではなく、観測と仮説検証として扱えている。

## 関連資料

- [CloudWatch](services/management/cloudwatch.md)
- [CloudTrail](services/management/cloudtrail.md)
- [Config](services/management/config.md)
- [Systems Manager](services/management/ssm.md)
- [Security Hub](services/management/security-hub.md)
- [Fault Injection Simulator](services/management/fis.md)
- [Resilience Hub](services/management/resilience-hub.md)
