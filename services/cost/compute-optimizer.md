# AWS Compute Optimizer

## 何をするサービスか

AWS Compute Optimizer は、CloudWatchメトリクスなどを分析して、EC2、Auto Scaling groups、EBS、Lambda、ECS on Fargate、RDS/Auroraなどのリソースに対して最適化推奨を出すサービス。

## 主な推奨

| 対象 | 推奨例 |
|---|---|
| EC2 | インスタンスタイプ変更、過剰/不足プロビジョニングの指摘 |
| Auto Scaling groups | インスタンスタイプやグループ設定の最適化 |
| EBS | ボリュームタイプ/サイズ/IOPSの見直し |
| Lambda | メモリサイズ調整 |
| ECS on Fargate | CPU/メモリ設定の見直し |
| RDS/Aurora | DBインスタンスクラス、Aurora I/O-Optimized等の推奨 |

## Trusted Advisorとの違い

| サービス | 主な役割 |
|---|---|
| Compute Optimizer | 実メトリクスに基づくリソースサイズ最適化 |
| Trusted Advisor | コスト、セキュリティ、信頼性、性能、制限などのベストプラクティスチェック |

## SAP-C02 Focus

- 「実際の使用率からサイズを下げる/上げる」ならCompute Optimizer。
- 「Savings PlansやRI購入前」にまずRightsizingを実施する。
- Organizations環境では、管理アカウントまたは委任管理者で複数アカウントの推奨を集約する。
