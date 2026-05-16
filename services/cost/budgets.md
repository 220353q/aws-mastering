# AWS Budgets

## 何をするサービスか

AWS Budgets は、コスト・使用量・RI/Savings Plans利用率/カバレッジに対して予算を設定し、しきい値に応じて通知やアクションを行うサービス。

## 代表的な予算タイプ

| 予算タイプ | 用途 |
|---|---|
| Cost budget | 月次コストがしきい値を超えた/超えそうな場合に通知 |
| Usage budget | 特定サービスの使用量を監視 |
| RI utilization / coverage | RIの利用率・カバレッジを監視 |
| Savings Plans utilization / coverage | Savings Plansの利用率・カバレッジを監視 |

## Cost Explorerとの違い

| サービス | 役割 |
|---|---|
| Cost Explorer | 分析・予測・傾向把握 |
| Budgets | しきい値監視・通知・予算運用 |

## Budget Actions

Budget Actionsを使うと、しきい値超過時にIAM/SCPの適用やEC2/RDS停止などのアクションを実行できる。ただし、本番停止を伴う自動制御は慎重に設計する。

## SAP-C02 Focus

- 「予算を超えたら通知」だけならBudgets。
- 「予算超過時に自動制御」ならBudget Actionsも検討。
- Organizations環境では、アカウント/OU/タグ/Cost Category単位で予算を切る。
