# Amazon EMR

Amazon EMR は、Apache Spark、Hadoop、Hive、Prestoなどのビッグデータ処理基盤をAWS上で実行するサービス。SAP-C02では、Athena/Glue/Redshiftとの違いで問われる。

## 一言で

Spark/Hadoopエコシステムの大規模分散処理を細かく制御したいならEMR。

## 試験で選ぶ条件

- 既存Spark/HadoopジョブをAWSへ移行したい
- 大規模ETL/機械学習前処理を分散処理したい
- クラスタ構成、インスタンス、Spot活用を制御したい
- S3上のデータを分散処理したい

## Glue / Athena / Redshiftとの違い

| 要件 | サービス |
|---|---|
| サーバーレスSQLでS3をアドホック分析 | Athena |
| マネージドETLとData Catalog | Glue |
| DWH、BI向け高性能分析 | Redshift |
| Spark/Hadoopクラスタ制御 | EMR |

## High-Risk Exam Traps

- SQLだけの簡易ログ分析ならAthenaが軽い。
- フルマネージドETL中心ならGlueが自然。
- BI/DWHの低レイテンシ分析はRedshiftを検討する。

## Related

- [Athena](athena.md)
- [Glue](glue.md)
- [Redshift](redshift.md)
