# Amazon QuickSight / Amazon Quick Sight

## 何をするサービスか

Amazon QuickSight は、AWSのフルマネージドBIサービス。2026年時点のAWSドキュメントではAmazon Quickエコシステム内の **Amazon Quick Sight** として説明されるが、SAP-C02学習では従来名のQuickSightでも理解しておく。

## 主な用途

| 要件 | QuickSightの役割 |
|---|---|
| ダッシュボード作成 | 分析/可視化 |
| S3/Athena/Redshift/RDS等を可視化 | データソース連携 |
| 高速なインメモリ分析 | SPICE |
| SaaSや社内アプリへBIを埋め込み | Embedded Analytics |
| ユーザー別に見えるデータを制御 | Row-Level Security / Column-Level Security |

## 典型構成

```text
S3 Data Lake / Redshift / RDS / Athena
        ↓
QuickSight Dataset
        ↓ SPICE or Direct Query
Analysis / Dashboard
        ↓
Business Users / Embedded App
```

## Lake Formation連携の注意

QuickSightでLake Formationによる権限制御を効かせる場合、QuickSight側のデータソース、ユーザー/グループ、Lake Formation権限、Glue Data Catalog権限の対応関係を確認する。単にCognitoでMFA済みだからLake Formationの行/列制御が自動的に効く、という理解は危険。

## よくある誤答

- **QuickSightをETLとして選ぶ**: ETLはGlue。
- **QuickSightをDWHとして選ぶ**: DWHはRedshift。
- **QuickSightだけでデータレイク権限が完結すると思う**: Glue/Lake Formation/IAMとの連携が必要。

## SAP-C02 Focus

- 可視化/BIならQuickSight。
- 高速表示が必要ならSPICE、常に最新が必要ならDirect Queryを検討。
- 埋め込みBIではRegistered Embedding、IAM、Cognito、Lake Formation権限のつながりが重要。
