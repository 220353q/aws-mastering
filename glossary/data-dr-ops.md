# Data / DR / Operations Terms

## Data and Analytics

| 用語 | 意味 | SAP-C02での判断 |
|---|---|---|
| Data lake | 生データ/加工データをS3などに集約する基盤 | S3 + Glue + Athena + Lake Formation |
| Data warehouse | 分析用に構造化されたDWH | Redshift |
| ETL | Extract, Transform, Load | Glue、EMR、Step Functions連携 |
| ELT | Extract, Load, Transform | S3/Redshiftに入れてから変換 |
| Data Catalog | データのメタデータ管理 | Glue Data Catalog |
| Schema | データ構造 | Glue/Athena/DMS/SCTで重要 |
| CDC | Change Data Capture | DMSで最小停止DB移行 |
| Column-level permission | 列単位の権限制御 | Lake Formation |
| PII | 個人識別情報 | S3内検出はMacie |
| Streaming | 継続的に流れるデータ処理 | Kinesis/MSK/Flink |

## DR and Availability

| 用語 | 意味 | SAP-C02での判断 |
|---|---|---|
| RTO | 復旧までに許容される時間 | 短いほどWarm Standby/Active-Active寄り |
| RPO | 許容されるデータ損失時間 | 短いほど継続レプリケーション/同期寄り |
| Backup & Restore | バックアップから復元 | 最低コスト、RTO/RPO長め |
| Pilot Light | 最小限のコアだけDR側で常時稼働 | コストを抑えつつ短めRTO |
| Warm Standby | 縮小版フルスタックをDR側で常時稼働 | より短いRTO |
| Active/Active | 複数リージョンで常時処理 | 最高コスト/複雑性、低RTO/RPO |
| Failover | 障害時に待機系へ切替 | Route 53、Global Accelerator、DB昇格 |
| Switchover | 計画的な切替 | Aurora Global Databaseなど |
| Replication lag | レプリカの遅延 | RPOや読み取り整合性に影響 |

## Operations and Governance

| 用語 | 意味 | SAP-C02での判断 |
|---|---|---|
| Observability | メトリクス/ログ/トレースで状態を把握すること | CloudWatch + X-Ray + Logs |
| Audit trail | 操作証跡 | CloudTrail |
| Compliance | ルールや基準への準拠 | Config Rules / Conformance Packs |
| Finding | セキュリティ/準拠の検出結果 | Security Hubで集約 |
| Remediation | 修復 | EventBridge + SSM Automation/Lambda |
| Runbook | 手順書 | Systems Manager Automationで自動化可能 |
| Drift | IaC定義と実リソースの差分 | CloudFormation Drift Detection |
| Quota | サービス上限 | Service Quotas / Trusted Advisor |
| Tagging | リソースへのラベル付け | Cost Allocation Tags、運用/所有者管理 |

## Cost Terms

| 用語 | 意味 | SAP-C02での判断 |
|---|---|---|
| Rightsizing | 過大/過小リソースの適正化 | Compute Optimizer |
| Savings Plans | 使用量コミット割引 | EC2/Fargate/Lambdaなどの継続利用 |
| Reserved Instances | インスタンス予約割引 | RDS/Redshift/EC2など |
| Spot | 余剰キャパシティの割引利用 | 中断許容バッチ/ワーカー |
| Chargeback | 部門別に費用配賦 | Cost Allocation Tags、Cost Categories |
| CUR | 詳細な請求データ | S3に出力しAthena/QuickSightで分析 |

## Related

- [Disaster Recovery](../patterns/disaster-recovery.md)
- [Cost Optimization](../comparisons/cost-optimization.md)
- [Analytics Data Lake](../comparisons/analytics-data-lake.md)
- [Management Governance](../comparisons/management-governance.md)

