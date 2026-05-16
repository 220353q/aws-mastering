# Amazon QuickSight

## 何をするサービスか

Amazon QuickSight は、AWSのフルマネージドBIサービス。SAP-C02では、S3/Athena/Glue/Lake Formation/Redshiftなどで整えたデータを、ダッシュボードや埋め込み分析として提供する役割で問われる。

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

```text
Viewer
  → QuickSight dashboard
  → Dataset: SPICE or Direct Query
  → Athena / Redshift / RDS
  → Lake Formation / IAM / KMS as needed
```

判断ポイント:
- QuickSightはBI表示のサービス。ETLはGlue、DWHはRedshift、データレイク権限はLake Formation。
- SPICEに取り込んだデータは、更新タイミングと権限変更の反映を考える。
- Direct Queryでは背後のデータソース権限、実行主体、KMS権限を確認する。

## よくある誤答

- **QuickSightをETLとして選ぶ**: ETLはGlue。
- **QuickSightをDWHとして選ぶ**: DWHはRedshift。
- **QuickSightだけでデータレイク権限が完結すると思う**: Glue/Lake Formation/IAMとの連携が必要。

## SAP-C02での読み方

- 可視化/BIならQuickSight。
- 高速表示が必要ならSPICE、常に最新が必要ならDirect Queryを検討。
- 埋め込みBIではRegistered Embedding、IAM、Cognito、Lake Formation権限のつながりが重要。

## このページを読んだあとに戻るべき関連ページ

- [Access Control and Encryption Deep Dive](../../comparisons/access-control-and-encryption.md)
- [Analytics / Data Lake Service Selection](../../comparisons/analytics-data-lake.md)
- [Lake Formation](lakeformation.md)
- [Athena](athena.md)
- [Glue](glue.md)
- [KMS](../security/kms.md)
