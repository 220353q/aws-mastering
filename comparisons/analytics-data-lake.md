# Analytics / Data Lake Service Selection

## 一言で切り分ける

| 要件 | サービス |
|---|---|
| S3上のデータをSQLで分析 | Athena |
| メタデータカタログ/ETL | Glue |
| データレイク権限制御 | Lake Formation |
| DWH/高頻度分析 | Redshift |
| BI/ダッシュボード | QuickSight / Quick Sight |
| データ発見・共有・ガバナンス | DataZone |
| ストリーミング取り込み | Kinesis Data Streams / Firehose |
| ログ検索/全文検索 | OpenSearch |

## 典型データレイク構成

```text
Data Sources
  ├─ App logs
  ├─ RDS/DMS
  ├─ SaaS
  └─ Streams
       ↓
S3 Data Lake
       ↓ Crawler / ETL
Glue Data Catalog
       ├─ Athena
       ├─ Redshift Spectrum
       ├─ Lake Formation Permissions
       └─ QuickSight Dashboards
```

## SAP-C02での罠

- Athenaはクエリ、Glueはカタログ/ETL、Lake Formationは権限。
- QuickSightはBIであり、ETLやDWHではない。
- DataZoneはデータ利用体験とガバナンスの上位レイヤー。Lake Formationと同義ではない。
