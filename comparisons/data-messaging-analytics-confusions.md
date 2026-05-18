# Data / Messaging / Analytics 混同整理 — SAP-C02横断

## 何のためのページか

AWSのデータ系サービスは、保存、配信、ストリーム処理、分析、移行、ガバナンスの役割が混ざりやすい。

このページでは、SAP-C02で混同しやすいData / Messaging / Analytics系の概念を整理する。

---

## まず結論

```text
SQS = キュー。処理をためて1つずつ/並列にさばく
SNS = Pub/Sub通知。複数購読者へfanout
EventBridge = イベントルーティング。ルールでサービス間連携
Step Functions = ワークフロー制御。順序、分岐、リトライ
Kinesis = ストリーム。連続データを順序付きで処理
Amazon MQ = 既存MQ互換。ActiveMQ/RabbitMQ系
Aurora = RDB/OLTP
DynamoDB = NoSQL/key-value/document
Redshift = DWH
Athena = S3上のサーバーレスSQL
Glue = ETL/Data Catalog
Lake Formation = Data Lake権限管理
DataZone = データ発見/共有/ガバナンス
DMS = DB移行
DataSync = ファイル/オブジェクト転送
MGN = サーバー移行
```

---

## 1. SQS vs SNS vs EventBridge

| サービス | 一言 | 向く用途 |
|---|---|---|
| SQS | キュー | 非同期処理、バッファ、ワーカー分散 |
| SNS | Pub/Sub通知 | 複数購読者へfanout、通知配信 |
| EventBridge | イベントバス/ルーティング | SaaS/AWS/アプリ間イベント連携 |

### 覚え方

```text
SQS = ためる
SNS = 配る
EventBridge = ルールで振り分ける
```

### 誤答パターン

| 誤答 | なぜ危険か |
|---|---|
| SQSで複数購読者へ通知fanout | SNSが自然 |
| SNSで処理を安全にキューイング | SQSが必要になることが多い |
| EventBridgeを単なるキューとして使う | EventBridgeはイベントルーティング |

---

## 2. SQS Standard vs FIFO

| 種類 | 特徴 | 向く用途 |
|---|---|---|
| Standard | 高スループット、少なくとも1回配信、順序はベストエフォート | 一般的な非同期処理 |
| FIFO | 順序保証、重複排除 | 順序が重要な処理 |

### 覚え方

```text
順序が不要ならStandard
順序と重複排除が重要ならFIFO
```

---

## 3. Kinesis vs SQS

| 観点 | Kinesis | SQS |
|---|---|---|
| データモデル | Stream | Queue |
| 消費 | 複数consumerが同じstreamを読む | messageは処理後に削除 |
| 順序 | shard単位 | FIFOなら順序保証 |
| 向く用途 | ログ、クリックストリーム、IoT、大量時系列 | 非同期ジョブ、疎結合、バックプレッシャー |

### 覚え方

```text
Kinesis = 流れ続けるデータを読む
SQS = 処理待ちの仕事をためる
```

---

## 4. Step Functions vs EventBridge

| サービス | 役割 |
|---|---|
| Step Functions | ワークフロー制御、順序、分岐、リトライ、補償処理 |
| EventBridge | イベントを条件に応じてターゲットへルーティング |

### 判断

```text
処理の順序や状態管理が重要
  → Step Functions

イベント発生に応じてサービスへ配る
  → EventBridge
```

---

## 5. Amazon MQ vs SQS/SNS/EventBridge

| サービス | 向く用途 |
|---|---|
| Amazon MQ | 既存アプリがActiveMQ/RabbitMQ互換を必要とする |
| SQS/SNS/EventBridge | AWSネイティブな疎結合設計 |

### 覚え方

```text
既存MQ互換が必要 → Amazon MQ
新規AWSネイティブ設計 → SQS/SNS/EventBridge
```

---

## 6. Aurora vs DynamoDB

| 観点 | Aurora | DynamoDB |
|---|---|---|
| モデル | Relational | Key-value / Document |
| クエリ | SQL | Key-based access / PartiQL等 |
| トランザクション | RDBトランザクション | NoSQLトランザクション機能あり |
| スケール | Reader/cluster設計 | パーティション/キー設計 |
| 向く用途 | 複雑な関係、SQL、OLTP | 高スケール、低レイテンシ、キーアクセス |

### 覚え方

```text
関係とSQLが主役 → Aurora
キーアクセスと大規模スケール → DynamoDB
```

---

## 7. Redshift vs Athena

| サービス | 一言 | 向く用途 |
|---|---|---|
| Redshift | DWH | 大規模BI、定常分析、パフォーマンス最適化 |
| Athena | S3上のサーバーレスSQL | ad hoc分析、ログ分析、データレイククエリ |

### 覚え方

```text
Redshift = DWHを持つ
Athena = S3を直接読む
```

### 誤答パターン

| 誤答 | なぜ危険か |
|---|---|
| Aurora readerでDWHを代替 | OLTPとOLAPを混同している |
| Athenaを高頻度BI基盤として常に選ぶ | Redshiftや集計設計が必要な場合がある |
| RedshiftをS3ログの簡単な一回分析に使う | Athenaの方が軽い場合が多い |

---

## 8. Glue vs Lake Formation vs DataZone

| サービス | 役割 |
|---|---|
| Glue | ETL、Crawler、Data Catalog |
| Lake Formation | Data Lake権限管理、table/column/rowレベル制御 |
| DataZone | データ発見、カタログ、共有、申請、ガバナンス体験 |

### 覚え方

```text
Glue = データを加工/カタログ化
Lake Formation = データレイク権限を管理
DataZone = データを見つけて共有/利用する場
```

---

## 9. QuickSight vs Redshift/Athena

| サービス | 役割 |
|---|---|
| QuickSight | BI可視化、ダッシュボード |
| Redshift | DWH/分析エンジン |
| Athena | S3上のSQLクエリエンジン |

QuickSightは分析結果を可視化する側。データを保存するDWHそのものではない。

---

## 10. DMS vs DataSync vs MGN vs Transfer Family

| サービス | 何を移すか | 向く用途 |
|---|---|---|
| DMS | Database | DB移行、CDC、異種DB移行 |
| DataSync | File/Object | NFS/SMB/S3/EFS/FSx間転送 |
| MGN | Server | lift-and-shiftサーバー移行 |
| Transfer Family | File transfer endpoint | SFTP/FTPS/FTP/AS2のマネージド受け口 |

### 覚え方

```text
DBを移す → DMS
ファイルを移す → DataSync
サーバーを移す → MGN
SFTP等の入口を提供 → Transfer Family
```

---

## 11. S3 vs EFS vs FSx

| サービス | ストレージ型 | 向く用途 |
|---|---|---|
| S3 | Object | オブジェクト、データレイク、静的配信 |
| EFS | File / NFS | Linux共有ファイル、複数AZ共有 |
| FSx | File | Windows/Lustre/ONTAP/OpenZFSなど特化ファイル |

### 覚え方

```text
ObjectならS3
Linux NFS共有ならEFS
Windows/HPC/ONTAP系ならFSx
```

---

## 12. SAP-C02判断フロー

```text
非同期に処理をためたい？
  → SQS

複数購読者へ通知したい？
  → SNS

イベントをルールで振り分けたい？
  → EventBridge

順序/分岐/リトライ/補償を制御したい？
  → Step Functions

連続ログ/IoT/クリックストリーム？
  → Kinesis

既存MQ互換が必要？
  → Amazon MQ

SQL/RDB/OLTP？
  → Aurora/RDS

Key-value大規模低レイテンシ？
  → DynamoDB

DWH/BI基盤？
  → Redshift

S3上のad hoc SQL？
  → Athena

Data Lake権限制御？
  → Lake Formation

データ発見/共有/申請？
  → DataZone

DB移行？
  → DMS

ファイル転送？
  → DataSync

サーバー移行？
  → MGN
```

---

## よくある誤答パターン

| 誤答 | なぜ危険か |
|---|---|
| SQSとSNSを同じ非同期サービスとして扱う | QueueとPub/Subは違う |
| EventBridgeをキューとして使う | イベントルーティングであり、処理待ちqueueではない |
| Step Functionsをイベントバスとして使う | workflow制御が主役 |
| KinesisとSQSを同じメッセージングとして扱う | streamとqueueは消費モデルが違う |
| Auroraを分析/DWH用途に使う | OLTPとOLAPを混同 |
| QuickSightをデータ保存先として扱う | QuickSightはBI/可視化 |
| GlueとLake Formationを同じものとする | ETL/catalogと権限制御が違う |
| DMSでサーバー全体を移行する | サーバー移行はMGN |
| DataSyncでDBスキーマ変換をする | DB移行/変換はDMS/SCT |

---

## 最短暗記

```text
SQS = ためる
SNS = 配る
EventBridge = イベントで振り分ける
Step Functions = 流れを制御する
Kinesis = 流れ続けるデータ
Amazon MQ = 既存MQ互換
Aurora = RDB
DynamoDB = NoSQL
Redshift = DWH
Athena = S3をSQLで読む
Glue = ETL/Catalog
Lake Formation = Data Lake権限
DataZone = 発見/共有/申請
DMS = DB移行
DataSync = ファイル転送
MGN = サーバー移行
```

---

## 関連ページ

- [AWS 混同しやすい概念インデックス](aws-confusing-concepts-index.md)
- [EventBridge vs SNS vs SQS vs Step Functions vs Amazon MQ](messaging-eventing.md)
- [Analytics / Data Lake Service Selection](analytics-data-lake.md)
- [Amazon Aurora](../services/database/aurora.md)
- [AWS Endpoint / ENI / VIF 完全整理](endpoints-eni-vif.md)
