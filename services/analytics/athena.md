# Amazon Athena

## 何をするサービスか

Amazon Athena は、S3上のデータを標準SQLで直接クエリできるサーバーレス分析サービス。SAP-C02では、**S3データレイクへのアドホック分析、ログ分析、Glue Data Catalogとの連携**で出る。

## 典型構成

```text
S3 Data Lake
  ├─ raw/
  ├─ curated/ Parquet
  └─ query-results/
        ↑
AWS Glue Data Catalog
        ↑
Amazon Athena
        ↓
QuickSight / Analyst / BI
```

## 試験での判断軸

| 要件 | Athenaが向く理由 |
|---|---|
| S3にあるCSV/JSON/ParquetをすぐSQL分析 | データをDWHへロード不要 |
| サーバー管理を避けたい | サーバーレス |
| ログを必要な時だけ分析 | クエリ実行分の課金 |
| Lake Formationで表/列/行レベル制御したい | Glue Data Catalog / Lake Formation連携 |

## パフォーマンス/コスト最適化

- CSV/JSONよりParquet/ORCなどの列指向形式を使う。
- パーティションを設計する。
- 不要な列・期間をスキャンしない。
- クエリ結果用S3バケットと暗号化を設計する。

## よくある誤答

- **Redshiftと混同**: RedshiftはDWH。AthenaはS3上のデータを直接クエリする。
- **OpenSearchと混同**: OpenSearchは検索/ログ探索。AthenaはSQL分析。
- **Glueと混同**: Glueはカタログ/ETL。Athenaはクエリエンジン。

## SAP-C02 Focus

- S3データレイクを最小運用でSQL分析するならAthena。
- 高頻度・低レイテンシ・複雑なDWH用途ならRedshiftも検討。
- 権限制御はIAMだけでなくLake Formationを絡めて考える。
