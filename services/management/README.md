# Management & Governance Services

SAP-C02では、Management & Governance は単体暗記ではなく、**大規模組織でどう統制し、監査し、継続改善するか**として問われる。

## Tier 1

| サービス | 詳細 | 主な用途 |
|---|---|---|
| Amazon CloudWatch | [cloudwatch.md](cloudwatch.md) | メトリクス、ログ、アラーム、可観測性 |
| AWS CloudTrail | [cloudtrail.md](cloudtrail.md) | API監査、証跡、データイベント、Lake |
| AWS Config | [config.md](config.md) | リソース設定履歴、準拠評価、Conformance Pack |
| AWS CloudFormation | [cloudformation.md](cloudformation.md) | IaC、Stack、StackSets、Change Set、Drift Detection |
| AWS Systems Manager | [ssm.md](ssm.md) | Session Manager、Patch、Automation、Parameter Store |
| AWS Control Tower | [controltower.md](controltower.md) | Landing Zone、Account Factory、ガードレール |
| AWS Security Hub | [security-hub.md](security-hub.md) | セキュリティ所見集約、標準準拠チェック |
| AWS Compute Optimizer | [../cost/compute-optimizer.md](../cost/compute-optimizer.md) | Rightsizing、性能/コスト最適化 |

## 試験での見分け方

| 要件 | 選ぶもの |
|---|---|
| APIを誰がいつ呼んだか確認 | CloudTrail |
| リソース設定が過去どう変わったか確認 | AWS Config |
| 設定がルールに準拠しているか継続評価 | AWS Config Rules / Conformance Packs |
| メトリクス・ログ・アラーム | CloudWatch |
| 複数アカウント/複数リージョンへ同じIaCを展開 | CloudFormation StackSets |
| 管理コンソールを使わずEC2へ接続 | Systems Manager Session Manager |
| セキュリティ検出結果を集約・優先度付け | Security Hub |
| マルチアカウントの初期統制 | Control Tower |

## SAP-C02の重要原則

CloudTrail、Config、Security Hub、GuardDuty、EventBridge、Systems Manager Automation を組み合わせると、**検知 → 集約 → ルーティング → 自動修復** の設計が作れる。
