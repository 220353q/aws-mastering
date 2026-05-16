# RDS / Aurora Connection Deep Dive

RDS/Auroraの問題は「どのDBサービスを選ぶか」だけでは解けない。SAP-C02では、**DB instance、cluster、writer endpoint、reader endpoint、read replica、RDS Proxy、connection pool、分析基盤**を区別できるかが問われる。

## まず全体像

```text
Client / Lambda / ECS / EC2
  → application code
  → DB driver / connection pool
  → RDS Proxy?          (接続数を吸収する層)
  → writer endpoint?    (書き込み)
  → reader endpoint?    (読み取り分散)
  → DB instance / Aurora cluster
```

「DBに接続する」と言っても、実際には以下が別々に存在する。

| 層 | 見ること | 例 |
|---|---|---|
| 認証 | DBユーザー/パスワード/IAM DB auth | Secrets Manager, IAM auth |
| ネットワーク | どこからDBポートへ行けるか | private subnet, SG |
| 接続管理 | DB接続数をどう守るか | app pool, RDS Proxy |
| 読み書き分離 | SELECTと更新を分けるか | writer/reader endpoint |
| 可用性 | AZ障害にどう耐えるか | Multi-AZ, Aurora |
| 分析分離 | OLTPに重い集計を投げないか | Redshift, Athena, Glue |

## DB Instance / Cluster

### DB Instance

- 一言でいうと: DBエンジンが動く個別の計算リソース。
- 何のためにあるか: MySQL/PostgreSQLなどのDB処理を実行するため。
- 何が嬉しいか: 既存RDBに近い考え方で、CPU/メモリ/接続数をサイズで設計できる。
- 何と混同しやすいか: Cluster。instanceは1台の実行体、clusterは複数instanceと共有ストレージを含むまとまり。
- 試験問題ではどう出るか: `DB instance class`, `instance endpoint`, `fail over to standby`。
- 間違えやすい選択肢: instance endpointをアプリの汎用接続先にして、フェイルオーバー時の追従を壊す。
- 小さな構成図:

```text
RDS DB instance
  engine: PostgreSQL
  storage: managed volume
  endpoint: db-1.xxxxxx.rds.amazonaws.com
```

- 暗記のコツ、語源、語呂: **instance = 1つの実体**。

### Aurora Cluster

- 一言でいうと: 共有ストレージと複数DB instanceを持つRDSファミリーのクラスタ型DB。
- 何のためにあるか: 高可用性、高スループット、reader拡張、グローバルDRを扱いやすくするため。
- 何が嬉しいか: writer/reader endpoint、Aurora Replica、Global Databaseなどで設計の選択肢が増える。
- 何と混同しやすいか: 通常RDS Multi-AZ DB instance。Auroraのreaderは通常読み取りに使えるが、通常RDS Multi-AZ standbyは読み取り先ではない。
- 試験問題ではどう出るか: `Aurora Replicas`, `reader endpoint`, `Global Database`, `high throughput OLTP`。
- 間違えやすい選択肢: Aurora reader endpointをDWHとして重い分析に使う。
- 小さな構成図:

```text
Aurora Cluster
  shared cluster storage
  ├─ writer instance
  ├─ reader instance
  └─ reader instance
```

- 暗記のコツ、語源、語呂: **cluster = 房**。複数の実行体が1つのまとまり。

## Endpoint

### Writer / Cluster Endpoint

- 一言でいうと: 現在のwriterへ向かう安定した接続名。
- 何のためにあるか: フェイルオーバー後もアプリが新writerへ接続しやすくするため。
- 何が嬉しいか: アプリは個別instance名ではなく、役割名に接続できる。
- 何と混同しやすいか: reader endpoint。writerは更新可能、readerはread-only用途。
- 試験問題ではどう出るか: `INSERT/UPDATE/DELETE`, `current primary`, `failover support`。
- 間違えやすい選択肢: SELECT負荷分散にwriter endpointだけを使い続ける。
- 小さな構成図:

```text
App write path
  → writer endpoint
  → current primary DB instance
```

- 暗記のコツ、語源、語呂: **WriteはWriterへ**。

### Reader Endpoint

- 一言でいうと: reader群へ接続を分散するread-only入口。
- 何のためにあるか: SELECTをwriterから逃がすため。
- 何が嬉しいか: 読み取りが多いアプリでwriterの負荷を下げられる。
- 何と混同しやすいか: クエリ単位ロードバランサー。reader endpointは接続を分散するので、1本の長い接続内の個別クエリを毎回分けるわけではない。
- 試験問題ではどう出るか: `read scaling`, `reporting read-only`, `offload read traffic`。
- 間違えやすい選択肢: reader endpointへ書き込む、強いread-after-write整合性を期待する。
- 小さな構成図:

```text
App read path
  → reader endpoint
  ├→ reader instance A
  └→ reader instance B
```

- 暗記のコツ、語源、語呂: **ReaderはSELECTの避難先**。

### Instance Endpoint

- 一言でいうと: 特定DB instanceへ直接向かう接続名。
- 何のためにあるか: 診断、チューニング、特定reader確認のため。
- 何が嬉しいか: どのinstanceに問題があるか切り分けられる。
- 何と混同しやすいか: writer/reader endpoint。instance endpointは役割追従ではなく個体指定。
- 試験問題ではどう出るか: `connect to a specific DB instance`, `troubleshooting`。
- 間違えやすい選択肢: 本番アプリの通常接続をinstance endpoint固定にする。
- 小さな構成図:

```text
DBA / monitoring
  → instance endpoint
  → reader-2 only
```

- 暗記のコツ、語源、語呂: **instance endpoint = 個体名**。

## Read Offload

- 一言でいうと: 読み取り処理をwriterから逃がす設計。
- 何のためにあるか: writerを更新処理に集中させ、アプリ全体の応答を守るため。
- 何が嬉しいか: 商品一覧、検索補助、管理画面、軽いレポートをreaderへ逃がせる。
- 何と混同しやすいか: 分析基盤。read offloadはOLTPの読み取り逃がしであり、DWHではない。
- 試験問題ではどう出るか: `read-heavy workload`, `scale read queries`, `offload reporting queries`。
- 間違えやすい選択肢: 大量集計や全表スキャンを本番readerへ流す。
- 小さな構成図:

```text
Application
  ├─ write: writer endpoint
  └─ read:  reader endpoint / read replica
```

- 暗記のコツ、語源、語呂: **Offload = 荷物を下ろす**。writerの荷物をreaderへ。

注意点:

| 注意 | 理由 |
|---|---|
| replica lag | readerはwriterより遅れる可能性がある |
| read-only | 更新はwriterへ向ける |
| heavy analytics | OLTP readerを壊す可能性がある |

## RDS Proxy / Connection Pool

### Connection Pool

- 一言でいうと: DB接続を作り捨てせず使い回す仕組み。
- 何のためにあるか: 接続確立のCPU/メモリ/TLS/認証コストを減らすため。
- 何が嬉しいか: DBの最大接続数を守り、アプリの急増に耐えやすくなる。
- 何と混同しやすいか: キャッシュ。Connection Poolは結果を保存しない。
- 試験問題ではどう出るか: `too many database connections`, `connection storms`, `Lambda concurrency`。
- 間違えやすい選択肢: 読み取り結果を高速化するためにRDS Proxyを選ぶ。
- 小さな構成図:

```text
Without pool:
  Request → open connection → query → close

With pool:
  Request → borrow connection → query → return connection
```

- 暗記のコツ、語源、語呂: **pool = 使い回す貯め場**。

### RDS Proxy

- 一言でいうと: AWSマネージドのDB接続プール兼プロキシ。
- 何のためにあるか: Lambda/ECS/EC2からの接続急増をDBへ直撃させないため。
- 何が嬉しいか: 接続の再利用、接続数の制御、Secrets Manager/IAM認証、フェイルオーバー時の回復性向上。
- 何と混同しやすいか: ElastiCache、reader endpoint。RDS Proxyは接続管理、ElastiCacheはデータキャッシュ、reader endpointは読み取り分散。
- 試験問題ではどう出るか: `serverless application exhausts RDS connections`, `unpredictable surges`, `pool and share connections`。
- 間違えやすい選択肢: 複雑なSQL集計を速くする目的でRDS Proxyを選ぶ。
- 小さな構成図:

```text
Lambda functions
  → RDS Proxy endpoint
      → smaller pool of DB connections
          → RDS / Aurora
```

- 暗記のコツ、語源、語呂: **Proxyは間に立つ代理人**。DBの前で接続をさばく。

## 分析用途との切り分け

| 要件 | 正しい読み替え | 候補 |
|---|---|---|
| アプリのSELECTを逃がしたい | OLTP read scaling | Reader endpoint / Read Replica |
| 直近の本番データで軽い参照 | 本番影響を抑えた読み取り | Read Replica |
| 大量集計/BI/履歴分析 | OLAP/DWH | Redshift |
| S3ログをSQL分析 | データレイクSQL | Athena + Glue |
| ETL/カタログ/変換 | データ準備 | Glue |
| 表示ダッシュボード | BI | QuickSight |

### OLTPとOLAPの説明の説明

OLTPは、注文登録や在庫更新のような「小さな更新をたくさん正確に処理する」世界。OLAPは、月次売上や行動ログ集計のような「大量データをまとめて読む」世界。

```text
OLTP: App → RDS/Aurora → small indexed reads/writes
OLAP: BI  → Redshift/Athena → large scans/aggregations
```

reader endpointはOLTP側の読み取り逃がし。大量分析の土台そのものではない。

## 試験で選択肢を切る判断軸

| 問題文の制約語 | 読み替え | 選ぶ候補 |
|---|---|---|
| automatic failover / AZ failure | 高可用性 | Multi-AZ / Aurora |
| read-heavy / SELECT scaling | 読み取り逃がし | Read Replica / Reader endpoint |
| Lambda overwhelms DB | 接続数問題 | RDS Proxy |
| low-latency global reads / DR | グローバルDB | Aurora Global Database |
| complex analytics / BI / DWH | 分析基盤 | Redshift / Athena |
| cache repeated query results | 結果キャッシュ | ElastiCache / DAX |
| secret rotation | 認証情報管理 | Secrets Manager + RDS integration |

## SAP-C02での読み方

1. まず要件がHA、読み取り分散、接続数、DR、分析分離のどれかを決める。
2. Multi-AZは高可用性、Read Replica/Reader endpointは読み取り分散、と分ける。
3. RDS Proxyは「接続」を守る。クエリ結果や分析性能を上げるサービスではない。
4. 分析という語が出たら、OLTP readerで済む軽い参照か、Redshift/Athenaが必要な大量分析かを見極める。
5. 誰がDBに接続するかを明示する。Lambda実行ロール、EC2アプリ、DBユーザー、KMS権限は別々。

## このページを読んだあとに戻るべき関連ページ

- [Amazon RDS & Amazon Aurora](../services/database/rds.md)
- [Amazon Aurora](../services/database/aurora.md)
- [RDS vs DynamoDB](rds-vs-dynamodb.md)
- [DynamoDB vs Aurora](dynamodb-vs-aurora.md)
- [Analytics / Data Lake Service Selection](analytics-data-lake.md)
- [Pool Terms](../glossary/pool-terms.md)
- [KMS](../services/security/kms.md)

