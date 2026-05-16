# SAP-C02 Scenario Set 03

Migration / Disaster Recovery / Cost Optimization を中心にした長文シナリオ型問題集。選択肢は A〜G の7択。正解数は1〜3個。

---

## Question 1: 大規模サーバー移行の移行波設計

ある製造業はオンプレミスに600台以上のVMと物理サーバーを持ち、依存関係が十分に文書化されていない。経営層は12か月以内に大半をAWSへ移行したいが、まずアプリケーション単位の依存関係を把握し、移行波を計画したい。移行方式は当面リホスト中心で、カットオーバー時の停止時間を短くしたい。進捗は複数チームで共有し、移行後に一部アプリを段階的にモダナイズする予定である。最も適切な組み合わせはどれか。

A. Application Discovery Serviceでサーバー情報と依存関係を収集し、Migration Hubで移行進捗をアプリケーション単位に追跡し、MGNでリホスト移行を行う。

B. Snowball Edgeだけを使い、全サーバーの依存関係分析と継続レプリケーションを自動的に完了する。

C. DMSでVMイメージを継続レプリケーションし、Migration Hubでデータベーススキーマ変換を行う。

D. CloudFormation StackSetsだけでオンプレミスサーバーをAWSへ複製し、依存関係を自動検出する。

E. MGNはサーバーのリホストに使えるが、移行波設計にはDiscovery/Migration Hubの情報が役立つ。

F. 移行後の段階的モダナイゼーションでは、Strangler FigやBlue/Greenなどのパターンを検討する。

G. Transfer FamilyはVMレベルのリホスト移行に最適なサービスである。

**正解: A, E, F**

### 判断軸

- Application Discovery Serviceは移行計画のための発見・依存関係把握に使う。
- Migration Hubは複数移行ツールの進捗をアプリケーション単位で追跡する。
- MGNはサーバーのリホスト移行に向く。

### 構成図

```text
On-prem Servers
   ├─ Discovery Agent/Collector ── Application Discovery Service
   │                                      │
   │                                      ▼
   └─ MGN Agent ── Replication ── AWS Staging Area ── Cutover EC2
                                          │
                                     Migration Hub
```

### 誤答理由

Bはオフラインデータ転送と移行計画/継続レプリケーションを混同している。CはDMSの対象を広げすぎ。DはIaCと移行レプリケーションの混同。GはTransfer Familyの用途違い。

関連: [MGN](../services/migration/mgn.md), [Migration Hub](../services/migration/migration-hub.md), [Application Discovery Service](../services/migration/application-discovery-service.md), [Strangler Fig](../patterns/strangler-fig.md)

---

## Question 2: OracleからAurora PostgreSQLへの移行

ある企業はオンプレミスOracle Databaseで基幹システムを運用している。ライセンスコスト削減のためAurora PostgreSQLへ移行したい。データベース容量は数TBで、移行期間中も本番更新は継続されるため停止時間は最小化したい。ストアドプロシージャや独自データ型が多く、スキーマ互換性の確認と変換が必要である。最も適切な移行方針はどれか。

A. DMS Schema ConversionまたはSCTでスキーマ変換評価と変換を行い、AWS DMSのFull Load + CDCでデータを移行する。

B. MGNでOracle DatabaseをAurora PostgreSQLへ直接変換する。

C. DataSyncでOracleのデータファイルをS3へコピーすれば、Aurora PostgreSQLが自動的に読み込む。

D. 同種DB移行なのでスキーマ変換は不要であり、DMSのCDCだけでアプリケーション互換性も保証される。

E. 変換できないSQLやストアドプロシージャは手動修正・アプリケーション修正が必要になる可能性を見込む。

F. 最小停止時間のため、本番稼働中の変更をCDCで追跡し、カットオーバー直前に差分を同期する。

G. Transfer FamilyでOracleのSQLをSFTP送信すれば、Aurora PostgreSQLへ自動適用される。

**正解: A, E, F**

### 判断軸

- 異種DB移行は「スキーマ/コード変換」と「データ移行」を分ける。
- SCT/DMS Schema Conversionは変換評価とスキーマ変換の役割。
- DMSはFull Load + CDCで移行停止時間を短縮する。

### 構成図

```text
Oracle On-prem
   ├─ Schema/Code Assessment ── SCT / DMS Schema Conversion ── Aurora PostgreSQL Schema
   └─ Data Replication ──────── DMS Full Load + CDC ────────── Aurora PostgreSQL Data
```

### 誤答理由

Bはサーバー移行とDBエンジン変換を混同している。Cはファイル転送とDB移行の混同。DはOracle→PostgreSQLを同種DBと誤認している。GはTransfer Familyの用途違い。

関連: [DMS](../services/migration/dms.md), [SCT / DMS Schema Conversion](../services/migration/sct.md), [Aurora](../services/database/aurora.md)

---

## Question 3: ファイル移行・大容量オフライン移行・SFTP維持

ある研究機関はオンプレミスに500TBのNFSファイルサーバーを持ち、日々更新されるデータをAWSへ段階的に移したい。通常時は専用線の帯域に余裕があるため、差分同期を自動化したい。一方、別拠点にはネットワーク帯域が極端に細いPB級アーカイブがあり、初回移行は物理デバイスを使いたい。また、外部共同研究先は既存のSFTPクライアントを変えずにAWS上のデータ受け渡しを続けたい。最も適切な組み合わせはどれか。

A. 日々更新されるNFS/SMBデータのオンライン転送にはDataSyncを使う。

B. 帯域不足のPB級初回移行にはSnow Familyを検討する。

C. 既存SFTP/FTPS/FTP/AS2クライアントを維持する受け渡しにはTransfer Familyを使う。

D. DMSでNFSファイルの差分同期とSFTPサーバー運用をまとめて実現する。

E. Storage GatewayはオンプレからAWSストレージを継続利用するハイブリッドアクセスに向くが、大量ファイルの移行ジョブ自動化ではDataSyncと役割が異なる。

F. Snow Familyを使えば、移行後の日次差分同期もネットワークなしで自動的に継続される。

G. CloudFrontを使えば、オンプレNFSからS3への一括移行が自動実行される。

**正解: A, B, C**

### 判断軸

- DataSync: オンラインのファイル/オブジェクト転送・差分同期。
- Snow Family: 帯域制約が強い大容量オフライン移行。
- Transfer Family: 既存SFTP/FTPS/FTP/AS2クライアント維持。

### 構成図

```text
NFS/SMB File Server ── DataSync ── S3 / EFS / FSx
PB Archive Site ──── Snow Family ── AWS Import
Partners SFTP ───── Transfer Family ── S3 / EFS
```

### 誤答理由

DはDMSの対象を誤っている。Eは正しい補足だが、設問の3要件の直接回答としてはA/B/C。FはSnowの役割を継続同期まで広げすぎ。GはCDNの用途違い。

関連: [DataSync](../services/migration/datasync.md), [Snow Family](../services/migration/snow-family.md), [Transfer Family](../services/migration/transfer-family.md), [Storage Gateway](../services/storage/storage-gateway.md)

---

## Question 4: RTO/RPOとDR戦略

あるEC企業は注文API、在庫DB、画像配信基盤をAWS上で運用している。現行は単一リージョンMulti-AZだが、リージョン障害時にも主要注文機能を1時間以内に復旧し、データ損失は数分以内に抑えたい。ただし常時二重稼働するMulti-Site Active/Activeほどのコストは許容できない。読み取り専用コンポーネントやIaCは別リージョンに準備できるが、通常時は最小構成に抑えたい。最も適切な戦略はどれか。

A. Backup & Restoreだけを採用し、復旧時にAMIとDBスナップショットからすべて手動復旧する。

B. Pilot Lightとして最小限のコアデータ基盤とIaCを別リージョンに準備し、必要時にアプリ層をスケールアウトする。

C. Warm Standbyとして縮小版のアプリ/DB環境を別リージョンに常時稼働させ、障害時にスケールアップする。

D. Multi-Site Active/Activeを必ず採用しなければ、RPOを短くすることは一切できない。

E. Route 53のヘルスチェックやGlobal Acceleratorなどを使い、切替経路を設計する。

F. DR手順はドキュメント化すれば十分で、定期テストは不要である。

G. S3 CRR、Aurora Global Database、DynamoDB Global Tablesなど、データ特性に合う複製方式を検討する。

**正解: C, E, G**

### 判断軸

- RTO 1時間以内、RPO数分以内、常時Active/Activeほどのコスト不可ならWarm Standbyが有力。
- Pilot Lightはより低コストだが、アプリ層の立ち上げが必要でRTOが長くなりやすい。
- DRはデータ複製、DNS/ルーティング切替、手順テストがセット。

### 構成図

```text
Primary Region                         DR Region
ALB / ECS / Aurora Primary              縮小版ALB / ECS / Aurora Secondary
S3 Bucket ── CRR ────────────────────> S3 Bucket
Route 53 / Global Accelerator ───────> Failover Routing
```

### 誤答理由

AはRTO/RPO要件に対して弱い。Bは近いが1時間以内・数分RPO・主要機能復旧ではWarm Standbyの方が安全。Dは極端。FはDRテストを軽視している。

関連: [Disaster Recovery](../patterns/disaster-recovery.md), [Global Accelerator](../services/networking/global-accelerator.md), [Route 53](../services/networking/route53.md), [Aurora](../services/database/aurora.md)

---

## Question 5: 移行後のコスト最適化と購入オプション

ある企業はMGNで移行したEC2群のコスト削減を進めている。Web層はAuto Scalingで日中に増減し、基幹APIは24時間安定稼働する。バッチ処理は夜間に大量起動するが、失敗時は再実行できる。移行直後のインスタンスサイズは過大である可能性が高く、今後一部はGravitonへ変更する予定である。コスト削減の順序として最も適切なものはどれか。

A. まずCost Explorerとタグで支出を可視化し、Compute OptimizerでRightsizing候補を確認する。

B. 過大サイズのまま3年Standard RIを購入し、後でインスタンスタイプを変える。

C. 安定稼働する基幹APIのベースラインにはCompute Savings Plansなど柔軟性のあるコミットメント割引を検討する。

D. 中断可能な夜間バッチにはSpot InstancesやSpot Fleet/EC2 Auto Scalingの活用を検討する。

E. Budgetsを設定すれば、リソースサイズは自動的に最適化される。

F. Web層のスケールイン/スケールアウトを見直し、夜間低負荷時の過剰稼働を減らす。

G. Savings Plansを買う前にRightsizingすると割引額が減るため、必ず購入を先に行う。

**正解: A, C, D**

### 判断軸

- コスト最適化は「可視化 → Rightsizing → 構造改善 → コミットメント割引」の順が安全。
- Spotは中断可能ワークロード向け。
- Savings Plans/RIは過剰プロビジョニングを自動解消しない。

### 構成図

```text
Cost Explorer / Tags
        ▼
Compute Optimizer ── Rightsizing / Graviton検討
        ▼
Workload分類
  ├─ 安定API     → Savings Plans検討
  ├─ Web Auto Scaling → スケジュール/ターゲット追従調整
  └─ 再実行可能Batch → Spot活用
```

### 誤答理由

B/Gは非効率を固定化する危険がある。Eは予算通知と最適化実行を混同。Fは正しい改善策だが、設問の「最も適切なもの」は包括的なA/C/Dの組み合わせ。

関連: [Cost Explorer](../services/cost/cost-explorer.md), [Compute Optimizer](../services/cost/compute-optimizer.md), [Savings Plans](../services/cost/savings-plans.md), [Reserved Instances](../services/cost/reserved-instances.md)
