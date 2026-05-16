# Amazon RDS & Amazon Aurora

Amazon Relational Database Service (RDS) は、MySQL、PostgreSQL、MariaDB、Oracle、SQL ServerなどのリレーショナルDBをマネージドに運用するサービス。Amazon AuroraはRDSファミリーの一部だが、クラスタ構造、ストレージ、Reader endpoint、Global Databaseなどで通常のRDS DB instanceとは設計判断が変わる。

SAP-C02では、単に「RDSを使う」ではなく、**単一DB instance、Multi-AZ DB instance、Multi-AZ DB cluster、Aurora cluster、Read Replica、RDS Proxy** を切り分ける。DB接続、endpoint、connection pooling、分析用途の切り分けは [RDS / Aurora Connection Deep Dive](../../comparisons/rds-aurora-connection-deep-dive.md) で深掘りする。

## まず一言で

| 要件 | 選ぶ候補 |
|---|---|
| 一般的なマネージドRDB | RDS DB instance |
| 高可用性、障害時自動フェイルオーバー | RDS Multi-AZ / Aurora |
| 高スループットOLTP、read scaling | Aurora |
| 読み取り負荷をwriterから逃がす | Read Replica / Aurora Replica / Reader endpoint |
| Lambdaなどの接続急増を吸収 | RDS Proxy |
| グローバル低レイテンシ読み取り/DR | Aurora Global Database |
| 大量分析/DWH | Redshift / Athena / データレイク |

## RDSとAuroraの構造差

| 項目 | RDS DB instance | Aurora DB cluster |
|---|---|---|
| 基本単位 | DB instance | DB cluster + 複数DB instances |
| 書き込み | Primary instance | Primary instance |
| 読み取り分散 | Read Replicaを追加 | Aurora Replicas + Reader endpoint |
| ストレージ | インスタンスに紐づく管理ストレージ | クラスタ共有ストレージ |
| エンドポイント | Instance endpoint中心 | Cluster / Reader / Instance / Custom endpoint |
| グローバルDR | Cross-Region replica等 | Aurora Global Database |

## Multi-AZの混同注意

| 構成 | 読み取りに使えるか | 試験での注意 |
|---|---:|---|
| RDS Multi-AZ DB instance | 基本的にNo | standbyは高可用性用。read offload目的ではない |
| RDS Multi-AZ DB cluster | Yes | writer endpoint / reader endpoint / instance endpointを使い分ける |
| Aurora cluster | Yes | primary + replicas。reader endpointでread-onlyを分散 |

**重要**: 「Multi-AZだから読み取り分散できる」と短絡しない。通常のRDS Multi-AZ DB instanceのstandbyは、障害時昇格用であり、普段の読み取り先ではない。

## Endpointの使い分け

### Aurora / RDS Multi-AZ DB cluster

| Endpoint | 接続先 | 用途 |
|---|---|---|
| Cluster endpoint / Writer endpoint | 現在のwriter | INSERT/UPDATE/DELETE、DDL、通常のread/write |
| Reader endpoint | reader群 | SELECT、レポート、read-only処理 |
| Instance endpoint | 特定DB instance | 診断、チューニング、特定readerへの接続 |
| Custom endpoint | Auroraの特定replica subset | 高性能reader群、低優先reader群などを分ける |

```text
Application write path
  → Cluster endpoint / Writer endpoint
  → Current writer DB instance

Application read path
  → Reader endpoint
  → Reader instances / Aurora Replicas

Troubleshooting
  → Instance endpoint
  → Specific DB instance
```

## Read Offload

Read offloadは、読み取り処理をwriterから逃がして、書き込み性能やアプリ応答を守る設計。

```text
Web/API
  ├─ Write: writer endpoint
  └─ Read: reader endpoint / read replica
```

向いている処理:
- 商品一覧、検索補助、レポート表示
- 管理画面の参照系
- BIの軽い参照
- 読み取りが多いWebアプリ

注意:
- readerはread-only。書き込みはwriterへ。
- replica lagがあるため、書いた直後にreaderで読むと古い値が見える可能性がある。
- 重い分析を本番readerへ投げると、本番read性能を壊すことがある。DWHやデータレイクも検討する。

## 分析用途との切り分け

| 要件 | 選ぶ |
|---|---|
| アプリの読み取り負荷を逃がす | Reader endpoint / Read Replica |
| 本番DBに近いデータで軽いレポート | Read Replica / Aurora Replica |
| 大量集計、BI、DWH | Redshift |
| S3上のログ/データレイクをSQL分析 | Athena |
| ETL、カタログ、変換 | Glue |

**補足**: 「分析用だからreader endpoint」と決め打ちしない。reader endpointはread-only queryの分散先であり、重い分析基盤そのものではない。

## RDS Proxy

RDS Proxyは、アプリケーションとDBの間でDB接続をプールし、再利用するマネージドプロキシ。

```text
Lambda / ECS / App
  → RDS Proxy
     → pooled DB connections
       → RDS / Aurora
```

選ぶ条件:
- Lambdaなどで同時実行が急増し、DB接続数が枯渇しやすい
- DB接続確立のCPU/メモリ負荷を減らしたい
- Secrets ManagerやIAM認証と組み合わせたい
- フェイルオーバー時にアプリ接続の影響を減らしたい

注意:
- RDS Proxyはクエリ結果をキャッシュしない。キャッシュならElastiCache/DAX。
- SQLの状態によってはconnection pinningが起き、プール効率が落ちる。
- 読み取り分散そのものはReader endpoint/Replicaの役割。

## RDS / Aurora / ElastiCache

| 要件 | 選ぶ |
|---|---|
| 永続的なリレーショナルデータ | RDS / Aurora |
| 読み取りを水平分散 | Read Replica / Aurora Replica |
| DB接続数を抑える | RDS Proxy |
| 頻繁に読む結果を高速化 | ElastiCache |
| セッション保存 | ElastiCache / DynamoDBなど |

## Security

- 保存時暗号化: KMS。スナップショットやリードレプリカの暗号化も確認する。
- 通信経路暗号化: TLS/SSL接続を使う。
- 認証情報: Secrets Managerで管理し、ローテーションを検討する。
- IAM DB Authentication: 対応エンジンで、DBパスワードの代わりにIAM認証を使える。
- ネットワーク: DBは通常private subnetに置き、SGでApp SGからのDBポートのみ許可する。

## High-Risk Exam Traps

| 罠 | 正しい判断 |
|---|---|
| RDS Multi-AZ standbyを読み取りに使う | 通常のMulti-AZ DB instanceのstandbyは読み取り用ではない |
| Reader endpointへ書き込む | Reader endpointはread-only |
| Reader endpointをDWHとして扱う | 大量分析はRedshift/Athena/データレイクも検討 |
| RDS Proxyをキャッシュとして選ぶ | RDS Proxyはconnection pooling。結果キャッシュではない |
| Read Replicaで強整合なread-after-writeを期待する | replica lagに注意 |
| KMS暗号化スナップショットを共有し、KMS key policyを忘れる | Snapshot共有 + KMS許可が必要 |

## SAP-C02での読み方

RDS/Aurora問題は、**HA、read scaling、connection scaling、DR、分析分離** のどれを求めているかで解く。

```text
障害に強くしたい → Multi-AZ / Aurora
読み取りを逃がしたい → Read Replica / Reader endpoint
接続数を守りたい → RDS Proxy
グローバルDR/低遅延read → Aurora Global Database
大量分析 → Redshift / Athena
```

## このページを読んだあとに戻るべき関連ページ

- [RDS / Aurora Connection Deep Dive](../../comparisons/rds-aurora-connection-deep-dive.md)
- [Aurora](aurora.md)
- [DynamoDB vs Aurora](../../comparisons/dynamodb-vs-aurora.md)
- [RDS vs DynamoDB](../../comparisons/rds-vs-dynamodb.md)
- [ElastiCache](elasticache.md)
- [KMS](../security/kms.md)
