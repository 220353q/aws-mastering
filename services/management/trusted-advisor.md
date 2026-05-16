# AWS Trusted Advisor

AWS Trusted Advisor は、AWS環境をベストプラクティス観点でチェックするサービス。SAP-C02では、コスト、セキュリティ、信頼性、性能、サービス制限の改善候補を見つける文脈で出る。

## 一言で

既存環境のベストプラクティス逸脱を広く見つけるならTrusted Advisor。

## Compute Optimizerとの違い

| 要件 | 選ぶ |
|---|---|
| EC2/EBS/Lambda/ECS/RDSなどのRightsizing | Compute Optimizer |
| コスト/セキュリティ/耐障害性/サービス制限の幅広いチェック | Trusted Advisor |
| 継続的なリソース設定準拠評価 | AWS Config |

## 試験で選ぶ条件

- 未使用リソースや過剰なリソースを見つけたい
- セキュリティグループの過剰開放などを検出したい
- サービスクォータや耐障害性リスクを確認したい
- 既存環境の改善候補を広く把握したい

## High-Risk Exam Traps

- Trusted Advisorは詳細な請求分析ツールではない。支出分析はCost Explorer/CUR。
- 設定変更履歴の記録はConfig、API監査はCloudTrail。
- Rightsizing専用の深い推奨はCompute Optimizerを優先する。

## Related

- [Compute Optimizer](../cost/compute-optimizer.md)
- [Cost Explorer](../cost/cost-explorer.md)
- [Config](config.md)
