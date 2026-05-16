# AWS CloudFormation

## 何をするサービスか

AWS CloudFormation は、テンプレートでAWSリソースを定義し、Stackとして作成・更新・削除するIaCサービス。SAP-C02では、**再現性のある環境構築、複数アカウント展開、変更前確認、ドリフト検出**の文脈で出る。

## 試験での判断軸

| 要件 | CloudFormationの使い方 |
|---|---|
| 同じ構成を何度も再現 | Template + Stack |
| 変更の影響を事前確認 | Change Set |
| 手動変更でテンプレートと実環境がズレたか確認 | Drift Detection |
| 複数アカウント/複数リージョンへ標準構成を展開 | StackSets |
| Organizations配下へ一括展開 | StackSets service-managed permissions |
| アプリケーション一式をまとめて管理 | Nested Stack / Stack分割 |

## StackSetsが問われる場面

大企業のSAP-C02問題では、単一アカウントのIaCではなく、**100以上のアカウントに同じIAM Role、Config Rule、CloudTrail設定、VPC基盤を展開する**場面が出やすい。この場合は通常のStackではなくStackSetsが候補になる。

```text
Administrator Account
  └─ StackSet
       ├─ Account A / ap-northeast-1
       ├─ Account B / us-east-1
       └─ Account C / eu-west-1
```

## Change Set

Change Setは、Stack更新前に「どのリソースが作成・変更・置換・削除されるか」を確認する仕組み。RDSやELBなど、置換されると影響が大きいリソースを含む更新で重要。

## Drift Detection

Drift Detectionは、CloudFormationテンプレートの期待状態と実リソースの現在状態のズレを検出する。手動変更を完全に防ぐ機能ではない。ズレを見つけるための機能。

## よくある誤答

- **CloudTrailと混同**: CloudTrailはAPI監査。CloudFormationはリソース作成・変更の宣言的管理。
- **Configと混同**: Configは設定履歴・準拠評価。CloudFormationは望ましい構成の展開。
- **CodeDeployと混同**: CodeDeployはアプリケーションデプロイ。CloudFormationはインフラ定義。

## SAP-C02 Focus

- 複数アカウント展開ならStackSets。
- 本番変更前の影響確認ならChange Set。
- 実リソースが手動変更されたか確認するならDrift Detection。
- Control TowerのAccount Factoryやガードレールと組み合わせて、標準環境の自動展開を考える。
