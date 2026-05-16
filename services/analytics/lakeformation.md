# AWS Lake Formation

## 何をするサービスか

AWS Lake Formationは、S3上のデータレイクに対して、Glue Data Catalogのテーブル/列/行などの単位で権限を管理するサービス。S3 bucket policyだけで頑張るのではなく、**データ利用者にどのデータを見せるか**をデータレイクの言葉で制御する。

```text
Data in S3
  → Glue Data Catalog
  → Lake Formation permissions
  → Athena / Redshift Spectrum / Glue / QuickSight
```

## 一言でいうと

S3データレイクの「閲覧席」と「入館権限」を管理するサービス。

## 何のためにあるか

- S3上の生データを、テーブル、列、行、データロケーション単位で制御する。
- 部門別/職務別に見えるデータを変える。
- Athena、Glue、Redshift Spectrum、QuickSightなどの分析サービスから一貫した権限管理を使う。
- クロスアカウントのデータ共有を安全に行う。

## 何が嬉しいか

| 嬉しい点 | 説明 |
|---|---|
| 細粒度権限 | テーブル/列/行レベルで見せ分け |
| 中央管理 | S3 policyを個別に複雑化しにくい |
| 分析サービス連携 | Athena/Glue/Redshift/QuickSightと組み合わせやすい |
| 監査 | 誰がどのデータへアクセスしたか追いやすい |

## 何と混同しやすいか

| 混同 | 正しい見方 |
|---|---|
| Glue | Glueはカタログ/ETL、Lake Formationは権限統制 |
| Athena | Athenaはクエリ実行、Lake Formationはデータアクセス認可 |
| S3 bucket policy | S3は保存場所、Lake Formationはデータレイク権限 |
| KMS | KMSは復号権限、Lake Formationは見える行/列の制御 |
| QuickSight | QuickSightはBI表示、Lake Formationは背後のデータ権限 |

## 権限の流れ

```text
Analyst / BI user
  → Athena query or QuickSight dashboard
  → IAM principal / execution role
  → Glue Data Catalog table
  → Lake Formation permissions
  → S3 data location
  → KMS decrypt if SSE-KMS
```

見るポイント:

| 層 | 確認すること |
|---|---|
| IAM | Athena/Glue/Lake Formationを呼べるか |
| Lake Formation | DB/table/column/row/data location権限があるか |
| S3 | 直接S3 Readで迂回できないか、結果出力先に書けるか |
| KMS | SSE-KMSなら`kms:Decrypt`や`GenerateDataKey`があるか |
| QuickSight | Dataset/dashboard/RLS/CLSの設定が正しいか |

## 試験問題ではどう出るか

| 問題文の制約語 | 読み替え | 候補 |
|---|---|---|
| governed data lake | データレイク権限の一元管理 | Lake Formation |
| column-level access | 列単位の参照制御 | Lake Formation |
| Athena users can query only approved tables | テーブル権限 | Lake Formation + Glue |
| BI users see only department rows | 表示/行制御 | QuickSight RLS または Lake Formation |
| encrypted data lake access denied | 復号権限不足 | KMS key policy/IAM |
| discover/request/share datasets | データ利用体験/ガバナンス | DataZone + Lake Formation |

## 間違えやすい選択肢

- KMS暗号化だけで列レベル制御ができると考える。
- Glue Data Catalogにテーブルがあるだけで、ユーザーがデータを読めると考える。
- QuickSightのダッシュボード共有だけで、背後のS3/KMS/Lake Formation権限が不要になると考える。
- S3 bucket policyを広く開けてLake Formationを迂回させる。
- DataZoneをLake Formationの細粒度権限そのものとして扱う。

## 小さな構成図

```text
S3 raw/curated data
  → Glue crawler / ETL
  → Glue Data Catalog
  → Lake Formation grants
      ├→ Athena analysts
      ├→ Glue jobs
      └→ QuickSight datasets
```

## 暗記のコツ、語源、語呂

- **Lake Formation = 湖を整備して入館管理する**。
- S3は湖の水、Glueは地図、Lake Formationは入館権限、Athenaは検索係、QuickSightは見せる画面。

## Connections

- **S3**: データ本体。
- **Glue Data Catalog / Glue ETL**: メタデータと変換。
- **Athena / Redshift Spectrum / EMR / Glue**: データを読む/処理する実行サービス。
- **QuickSight**: BI表示。
- **KMS**: SSE-KMSデータやクエリ結果の復号/暗号化。
- **DataZone**: データ発見、申請、共有の上位ガバナンス。

## SAP-C02での読み方

Lake Formationは「認証」ではなく「データレイクの認可」として読む。MFA済み、Cognitoログイン済み、IAM Allowあり、のどれか1つだけでは不十分なことが多い。Athena/QuickSight/Glueが実際にどのIAM principalやサービスロールでS3/KMSへアクセスするかまで追う。

## このページを読んだあとに戻るべき関連ページ

- [Access Control and Encryption Deep Dive](../../comparisons/access-control-and-encryption.md)
- [Analytics / Data Lake Service Selection](../../comparisons/analytics-data-lake.md)
- [Glue](glue.md)
- [Athena](athena.md)
- [QuickSight](quicksight.md)
- [KMS](../security/kms.md)
