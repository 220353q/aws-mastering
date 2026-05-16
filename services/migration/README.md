# Migration & Transfer Services

SAP-C02 Domain 4（Accelerate Workload Migration and Modernization）で重要な移行系サービスをまとめる。移行問題は「どのサービス名か」ではなく、**何を移行するのか、オンラインかオフラインか、同種か異種か、継続レプリケーションか一括転送か、カットオーバー時間をどこまで短くするか**で判断する。

---

## サービス選択マップ

| 移行対象 | 代表サービス | 判断軸 |
|---|---|---|
| サーバー / VM / 物理マシン | [AWS Application Migration Service](mgn.md) | リホスト、継続レプリケーション、短いカットオーバー |
| データベースのデータ | [AWS Database Migration Service](dms.md) | 同種/異種、CDC、最小停止時間 |
| 異種DBのスキーマ | [AWS SCT / DMS Schema Conversion](sct.md) | Oracle→Aurora PostgreSQL などの変換 |
| ファイル / オブジェクト | [AWS DataSync](datasync.md) | オンライン転送、NFS/SMB/S3/EFS/FSx |
| 大容量オフラインデータ | [AWS Snow Family](snow-family.md) | 回線が遅い、PB級、物理搬送 |
| SFTP/FTPS/FTP/AS2 ワークフロー | [AWS Transfer Family](transfer-family.md) | 既存転送方式を維持してS3/EFSへ |
| 移行計画 / 追跡 | [AWS Migration Hub](migration-hub.md) | アプリ単位で進捗を集約 |
| 現行環境の発見 | [AWS Application Discovery Service](application-discovery-service.md) | サーバー構成、使用率、依存関係の収集 |

---

## 7R とサービス対応

| 7R | 意味 | 代表サービス / パターン |
|---|---|---|
| **Rehost** | そのまま移す | MGN, VM Import/Export |
| **Relocate** | プラットフォーム単位で移す | VMware Cloud on AWS |
| **Replatform** | 少し変更して移す | RDS, Elastic Beanstalk, ECS |
| **Repurchase** | SaaSへ置換 | Marketplace, SaaS |
| **Refactor / Re-architect** | アプリ構造を作り替える | ECS/EKS, Lambda, Strangler Fig |
| **Retain** | 当面残す | Hybrid connectivity, Direct Connect |
| **Retire** | 廃止 | Migration Hub + Discoveryで棚卸し |

---

## SAP-C02の読み方

- `minimal downtime`, `ongoing replication`, `servers` → MGN。
- `database migration`, `CDC`, `heterogeneous` → DMS + Schema Conversion。
- `schema conversion`, `stored procedures`, `Oracle to Aurora PostgreSQL` → SCT / DMS Schema Conversion。
- `NFS`, `SMB`, `file server`, `S3/EFS/FSx` → DataSync。
- `petabytes`, `limited bandwidth`, `offline transfer` → Snow Family。
- `existing SFTP clients`, `partners`, `no client-side changes` → Transfer Family。
- `track many migration waves`, `application grouping` → Migration Hub。
- `discover dependencies`, `right-size before migration` → Application Discovery Service。
