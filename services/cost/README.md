# Cloud Financial Management / Cost Optimization Services

SAP-C02では、コスト最適化は単なる値引き選択ではなく、**需要の変動、性能要件、可用性、運用負荷、長期コミットのリスク**を踏まえた設計判断として問われる。

## Tier 1

| サービス/概念 | 詳細 | 主な用途 |
|---|---|---|
| AWS Cost Explorer | [cost-explorer.md](cost-explorer.md) | コスト分析、予測、RI/SPレポート |
| AWS Budgets | [budgets.md](budgets.md) | 予算、しきい値通知、利用率/カバレッジ監視 |
| Savings Plans | [savings-plans.md](savings-plans.md) | Compute使用量への柔軟な長期コミット割引 |
| EC2 Reserved Instances | [reserved-instances.md](reserved-instances.md) | EC2の属性一致型割引、キャパシティ予約との区別 |
| AWS Compute Optimizer | [compute-optimizer.md](compute-optimizer.md) | Rightsizing、性能/コスト推奨 |

## 選定の基本

| 要件 | 選ぶもの |
|---|---|
| 現状コストを分析したい | Cost Explorer |
| 予算超過前に通知/制御したい | AWS Budgets |
| EC2/Fargate/Lambdaの利用が安定している | Compute Savings Plans |
| 特定リージョン・特定ファミリーのEC2が安定 | EC2 Instance Savings Plans / Reserved Instances |
| 実際のメトリクスからサイズを下げたい | Compute Optimizer |
| 一時的で中断可能な処理 | Spot Instances |
| 確実な容量確保 | Capacity Reservation / Zonal RI |

## 試験での注意

Savings PlansやRIはコストを下げるが、アーキテクチャ上の過剰プロビジョニングを直すわけではない。まずRightsizing、Auto Scaling、Serverless、ストレージ階層化、データ転送料を見直す。
