# Cost Optimization Comparison

## 基本フロー

```text
1. Cost Explorerで現状分析
2. タグ/Cost Categoriesで配賦単位を整える
3. Compute OptimizerでRightsizing
4. Auto Scaling / Serverless / Storage Lifecycleで構造改善
5. 安定ベースラインにSavings Plans / RI
6. Budgetsで監視・通知
```

## Savings Plans / RI / Spot / Capacity Reservation

| 選択肢 | 主目的 | 向くワークロード | 注意点 |
|---|---|---|---|
| Compute Savings Plans | 柔軟な長期割引 | EC2/Fargate/Lambdaの安定ベースライン | 容量確保ではない |
| EC2 Instance Savings Plans | EC2の高割引 | リージョン/ファミリーが安定 | Compute SPより柔軟性は低い |
| Standard RI | EC2属性一致型割引 | 長期間変わらないEC2 | 変更に弱い |
| Convertible RI | 変更可能なRI | 将来変更があり得るEC2 | 割引率はStandardより低め |
| Spot Instances | 大幅割引 | 中断可能バッチ、分散処理 | 中断前提設計が必要 |
| Capacity Reservation | 容量確保 | 必ず起動したい重要処理 | 単体では割引ではない |

## SAP-C02での罠

- コスト削減問題で、いきなりRI/Savings Plansを選ばない。まず過剰プロビジョニングや停止漏れ、ストレージ階層化、データ転送を確認する。
- 容量確保と割引を混同しない。
- Spotは「安い本番基盤」ではなく「中断に強い処理」に使う。
