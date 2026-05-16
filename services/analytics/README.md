# Analytics Services

## Tier 1

| サービス | 詳細 | 主な用途 |
|---|---|---|
| Amazon Athena | [athena.md](athena.md) | S3上のSQL分析、ログ分析、アドホッククエリ |
| AWS Glue | [glue.md](glue.md) | Data Catalog、Crawler、ETL、データ統合 |
| AWS Lake Formation | [lakeformation.md](lakeformation.md) | データレイク権限、行/列レベル制御 |
| Amazon Redshift | [redshift.md](redshift.md) | DWH、Spectrum、RA3、分析基盤 |
| Amazon QuickSight / Quick Sight | [quicksight.md](quicksight.md) | BI、ダッシュボード、埋め込み分析 |
| Amazon DataZone | [datazone.md](datazone.md) | データカタログ、発見、共有、ガバナンス |
| Amazon Kinesis | [kinesis.md](kinesis.md) | ストリーミング取り込み/処理 |
| Amazon OpenSearch Service | [opensearch.md](opensearch.md) | ログ検索、全文検索、可視化 |

## Focus

Serverless analyticsは **S3 + Glue Data Catalog + Athena**、権限制御は **Lake Formation**、DWHは **Redshift**、BIは **QuickSight/Quick Sight**、組織横断の発見・共有・ガバナンスは **DataZone** と切り分ける。

詳しくは [Analytics / Data Lake Comparison](../../comparisons/analytics-data-lake.md) を参照。
