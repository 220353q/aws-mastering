# AWS Cost Explorer

## 何をするサービスか

AWS Cost Explorer は、AWSのコストと使用量を可視化・分析・予測するサービス。SAP-C02では、**現状把握、コスト配賦、RI/Savings Plans判断、異常な増加の原因分析**で出る。

## できること

| 機能 | 用途 |
|---|---|
| コスト/使用量の可視化 | サービス別、アカウント別、タグ別に分析 |
| Forecast | 月末までの予測コスト確認 |
| Group by / Filter | 環境、チーム、サービス単位の深掘り |
| RI / Savings Plans reports | 利用率・カバレッジ分析 |
| Recommendations | コミットメント購入判断の材料 |

## Cost ExplorerとCURの違い

| 項目 | Cost Explorer | Cost and Usage Report |
|---|---|---|
| 主な用途 | 画面で分析・予測 | S3出力された詳細明細をAthena/BIで分析 |
| 粒度 | 標準的な分析向け | 最も詳細 |
| 向く場面 | まず状況を見る | FinOps基盤・高度な配賦 |

## SAP-C02 Focus

- 「コストを分析したい」「予測したい」ならCost Explorer。
- 「予算を超えそうなら通知」ならAWS Budgets。
- 「詳細な請求データをデータレイクへ出したい」ならCUR/Data Exports。
- コスト配賦にはCost Allocation TagsとCost Categoriesを組み合わせる。
