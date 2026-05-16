# Amazon Aurora

## 何をするサービスか

Amazon Auroraは、MySQL/PostgreSQL互換のマネージドリレーショナルDB。RDSファミリーの一部だが、通常のRDS DB instanceと違い、**cluster、共有ストレージ、writer endpoint、reader endpoint、Aurora Replica、Global Database**を前提に設計する。

```text
Aurora DB cluster
  shared cluster storage
  ├─ writer instance
  ├─ reader instance
  └─ reader instance
```

## 一言でいうと

高スループットOLTP、読み取り分散、グローバルDRを扱いやすいクラスタ型RDB。

## 何のためにあるか

- 既存RDBの互換性を保ちつつ、性能/可用性/運用性を上げる。
- writerとreaderを分けて、読み取り負荷を逃がす。
- Global Databaseでリージョン障害やグローバル低レイテンシreadに備える。
- Serverless v2で変動負荷に合わせた容量調整をしやすくする。

## 何が嬉しいか

| 嬉しい点 | 説明 |
|---|---|
| Reader endpoint | read-only接続をreader群へ分散 |
| Cluster endpoint | writerの切替後もアプリ接続先を安定化 |
| Aurora Replicas | 読み取りスケールと可用性 |
| Global Database | 低遅延グローバルreadとDR |
| Managed backups/PITR | 運用負荷を削減 |

## 何と混同しやすいか

| 混同 | 正しい見方 |
|---|---|
| RDS Multi-AZ DB instance | 従来型Multi-AZのstandbyは基本的に読み取り先ではない。読み取り分散が目的ならRead ReplicaやAurora Replicaを検討する |
| RDS Multi-AZ DB cluster | RDS側にもクラスタ型構成があるが、Auroraとはストレージ、レプリカ、Global Databaseの設計が異なる |
| Read Replica | Aurora Replicaはcluster内readerとしてreader endpointと組み合わせる |
| Redshift | AuroraはOLTP中心。大量分析/DWHはRedshift/Athena |
| RDS Proxy | Auroraのreader endpointは読み取り分散、RDS Proxyは接続プール。クエリキャッシュや自動read routingではない |

## Endpointの読み方

```text
Application
  ├─ write path → cluster/writer endpoint → current writer
  └─ read path  → reader endpoint         → Aurora Replicas

DBA / monitoring
  → instance endpoint → specific DB instance
```

| Endpoint | 用途 | 試験の罠 |
|---|---|---|
| Cluster / Writer endpoint | 書き込み、通常のread/write | SELECT負荷を全部ここへ寄せる |
| Reader endpoint | read-only接続の分散 | 書き込みや強いread-after-writeを期待する |
| Instance endpoint | 特定instanceへの接続 | 本番アプリの通常接続先に固定する |
| Custom endpoint | reader subset分離 | 何でも自動最適化する魔法ではない |

Reader endpointは読み取りをAurora Replica群へ分散するが、writer直後の最新データを常に即座に読めることを保証するものではない。強いread-after-writeが必要な処理では、writer endpointを使うか、アプリ側で読み取り経路を分ける。

## Global Databaseの読み方

Aurora Global Databaseは、基本的に1つのprimary Regionでwriteし、secondary Regionは低レイテンシreadとDRに使う。リージョン障害時はsecondaryを昇格させて復旧する。DynamoDB Global Tablesのような双方向multi-active writeを簡単に実現するサービスではない。

```text
Primary Region
  App writes → Aurora writer
               └─ storage replication
                    ↓
Secondary Region
  App reads  → Aurora secondary cluster
  DR event   → promote secondary to primary
```

write forwardingは存在するが、DynamoDB Global Tablesのような双方向multi-active writeとは別物として扱う。SAP-C02では、Global Databaseを「global read + low RPO DR」として読み、multi-writer要件がある場合は本当にAuroraでよいかを確認する。

## 試験問題ではどう出るか

| 問題文の制約語 | 読み替え | 選ぶ候補 |
|---|---|---|
| high-throughput relational OLTP | RDB互換の高性能 | Aurora |
| read-heavy workload | 読み取り逃がし | Aurora Replicas + Reader endpoint |
| global users need low-latency reads | グローバルread/DR | Aurora Global Database |
| unpredictable relational workload | 変動負荷 | Aurora Serverless v2 |
| too many DB connections from Lambda | 接続数問題 | RDS Proxy |
| large analytics / BI / DWH | OLAP | Redshift / Athena |

## 小さな構成図

```text
Users
  → ALB / API
  → App tier
      ├─ writes → Aurora writer endpoint
      └─ reads  → Aurora reader endpoint
                     ├─ reader AZ-a
                     └─ reader AZ-c
```

## 暗記のコツ、語源、語呂

- **Aurora = RDSのクラスタ型RDB**。
- **WriteはWriter、ReadはReader**。
- **Global Databaseはデータの地域展開**。KMS Multi-Region keyとは別物。

## 間違えやすい選択肢

- Aurora reader endpointをDWHとして選ぶ。
- Reader endpointへ更新処理を投げる。
- Reader endpointで常に強いread-after-writeが得られると考える。
- Global Databaseだけでアプリの完全なmulti-active書き込みが簡単にできると考える。
- Aurora Global Databaseのsecondary Regionを通常時のwrite先として扱う。
- RDS Proxyをread scalingやクエリキャッシュとして選ぶ。
- KMS暗号化スナップショット共有でKMS key policyを忘れる。

## SAP-C02での読み方

Auroraは、RDB互換を保ちたいが、可用性、読み取り性能、グローバルDR、運用負荷削減が強く求められるときに読む。設問に「分析」「DWH」「大量集計」が出たら、Aurora readerだけでなくRedshift/Athenaとの役割分担を確認する。

## このページを読んだあとに戻るべき関連ページ

- [Amazon RDS & Amazon Aurora](rds.md)
- [RDS / Aurora Connection Deep Dive](../../comparisons/rds-aurora-connection-deep-dive.md)
- [DynamoDB vs Aurora](../../comparisons/dynamodb-vs-aurora.md)
- [Analytics / Data Lake Service Selection](../../comparisons/analytics-data-lake.md)
- [KMS](../security/kms.md)

