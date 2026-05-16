# AWS Glue

## 何をするサービスか

AWS Glue は、サーバーレスのデータ統合サービス。SAP-C02では、**Data Catalog、Crawler、ETL Job、S3データレイク、Athena/Redshift/Lake Formation連携**で出る。

## 主要コンポーネント

| コンポーネント | 役割 |
|---|---|
| Glue Data Catalog | データベース/テーブル/スキーマのメタデータ管理 |
| Crawler | S3やDBをクロールしてテーブル定義を作成/更新 |
| Glue Jobs | ETL/ELT処理を実行 |
| Triggers / Workflows | ジョブ実行の自動化・依存関係管理 |
| Glue Studio | 視覚的なETL作成 |

## Athena / Lake Formationとの関係

```text
S3 Data Lake
   ↓ Crawler
Glue Data Catalog
   ├─ Athena がSQLで参照
   ├─ Redshift Spectrum が外部テーブル参照
   └─ Lake Formation が権限制御
```

## 試験での判断軸

| 要件 | Glueの使い方 |
|---|---|
| S3上のデータ構造を自動発見 | Crawler |
| Athenaで参照するメタデータ管理 | Data Catalog |
| CSVをParquetへ変換 | Glue Job |
| DBからS3データレイクへ整形連携 | Glue ETL |
| テーブル単位の権限を中央管理 | Lake Formation + Glue Catalog |

## よくある誤答

- **GlueをBIとして選ぶ**: BIはQuickSight。
- **Glueをクエリエンジンとして選ぶ**: SQLクエリはAthenaやRedshift。
- **Crawlerだけでデータ品質が保証されると思う**: Crawlerはメタデータ検出。品質管理は別途設計が必要。

## SAP-C02 Focus

- Glue Data Catalogはデータレイクのメタデータ中核。
- Crawlerはスキーマ発見、JobはETL、AthenaはSQL分析、Lake Formationは権限制御。
