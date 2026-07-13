# AWS SAP設計読本

> **暗記から設計判断へ。** AWS Certified Solutions Architect – Professional（SAP-C02）で問われるのは、サービス名を知っていることではない。要件・制約・トレードオフを読み取り、なぜその構成を選ぶのか、なぜ別案を捨てるのかを説明できることである。

この読本は、リポジトリ内のサービス辞書、比較表、設計パターン、長文問題を一本の物語としてつなぐための本編である。最初から順番に読み、詳しく確認したい箇所だけ既存資料へ移動する。

---

## 目次

1. [SAPで問われる設計力](#1-sapで問われる設計力)
2. [問題文を要件定義書として読む](#2-問題文を要件定義書として読む)
3. [マルチアカウントとガバナンス](#3-マルチアカウントとガバナンス)
4. [ネットワークを経路として理解する](#4-ネットワークを経路として理解する)
5. [コンピュートを運用責任で選ぶ](#5-コンピュートを運用責任で選ぶ)
6. [ストレージをアクセス方式から選ぶ](#6-ストレージをアクセス方式から選ぶ)
7. [データベースをデータ特性から選ぶ](#7-データベースをデータ特性から選ぶ)
8. [疎結合とイベント駆動](#8-疎結合とイベント駆動)
9. [可用性・バックアップ・DRを分ける](#9-可用性バックアップdrを分ける)
10. [移行は7Rと停止許容時間で考える](#10-移行は7rと停止許容時間で考える)
11. [セキュリティを多層防御として設計する](#11-セキュリティを多層防御として設計する)
12. [コスト最適化は要件を落とすことではない](#12-コスト最適化は要件を落とすことではない)
13. [長文問題の解法](#13-長文問題の解法)
14. [総合ケーススタディ](#14-総合ケーススタディ)
15. [最終チェックリスト](#15-最終チェックリスト)

---

# 1. SAPで問われる設計力

SAAでは「この用途に合うサービスは何か」が中心になる。SAPでは、候補が複数成立する状況で、制約に最も合う案を選ぶ。

たとえば、世界向けWebサービスを高可用化するだけなら複数案がある。

- Route 53のフェイルオーバールーティング
- CloudFrontと複数オリジン
- Global Acceleratorと複数リージョンのALB/NLB
- Aurora Global Database
- DynamoDB Global Tables

どれも「グローバル」「高可用性」という語に関係する。しかし、同じ役割ではない。

- **DNSレベルで切り替える**のか
- **静的・HTTPコンテンツをエッジ配信する**のか
- **固定Anycast IPとAWSグローバルネットワークを使う**のか
- **リレーショナルデータを別リージョンへ複製する**のか
- **複数リージョンで同時書き込みする**のか

SAPの正解は、サービスの格ではなく、要件との一致度で決まる。

## 設計判断の基本式

設計は次の順序で考える。

1. **目的**：何を実現したいか
2. **前提**：既存環境、組織、技術、データ量
3. **制約**：停止時間、RPO/RTO、予算、運用人数、規制
4. **候補**：要件を満たせる複数案
5. **比較**：可用性、性能、コスト、運用、移行難易度
6. **決定**：最重要制約に最も合う案
7. **不採用理由**：他案が劣る理由

試験でも実務でも、最後の「不採用理由」が重要である。

> 関連資料: [試験テクニック](practice/exam-techniques.md)、[Well-Architected](well-architected/)

---

# 2. 問題文を要件定義書として読む

長文を最初から均等に読んではいけない。文章を次の4種類に分ける。

## 2.1 現状

- オンプレミスで稼働
- 複数AWSアカウントを個別管理
- 単一リージョン、単一AZ
- 手動デプロイ
- 夜間バッチが長時間化

現状は、変更対象と移行難易度を示す。

## 2.2 目標

- 可用性を上げたい
- グローバル展開したい
- 運用負荷を減らしたい
- セキュリティ統制を強化したい
- 移行中の停止を最小化したい

目標だけでは候補を一つに絞れない。

## 2.3 強い制約語

正解を決めるのは次のような言葉である。

| 制約語 | 設計への影響 |
|---|---|
| 最小の運用負荷 | マネージド、サーバーレス、自動化を優先 |
| 最小コスト | 常時冗長化より段階的復旧、従量課金、既存契約活用 |
| 数秒以内 | DNS切替だけでは不足する可能性。ヘルスチェック、接続、データ層まで確認 |
| データ損失不可 | 同期性、トランザクション、RPOを優先 |
| IPアドレスを変更不可 | 固定IP、NLB、Global Accelerator、BYOIPなどを検討 |
| 既存コードを変更不可 | リホスト、互換サービス、プロキシ、既存プロトコルを優先 |
| コンプライアンス | ログ集約、暗号化、職務分離、組織単位の予防統制 |
| 一時的な大量処理 | Auto Scaling、Spot、Batch、サーバーレスを検討 |

## 2.4 ノイズ

問題文には正解へ直結しない情報もある。サービス名を見つけた瞬間に飛びつかず、「その情報がどの制約に効くか」を確認する。

---

# 3. マルチアカウントとガバナンス

SAPでは、単一アカウント内のIAM設計だけでなく、企業全体をどう統制するかが問われる。

## 3.1 なぜアカウントを分けるのか

AWSアカウントは強い分離境界である。VPC、IAM、請求、APIクォータ、障害影響範囲を分離できる。

代表的な分割軸は次の通り。

- 本番 / 開発 / 検証
- 部門 / プロダクト
- セキュリティログ / 監査
- ネットワーク共有
- サンドボックス

「リソースを整理したい」だけならタグも使える。アカウント分割は、**権限・請求・障害影響を強く分離したいとき**に使う。

## 3.2 Organizations、SCP、IAMの関係

- **Organizations**：アカウントを組織化する
- **OU**：似た統制を適用する単位
- **SCP**：利用可能な権限の最大範囲を制限する
- **IAMポリシー**：実際の許可を与える

SCPは権限を付与しない。IAMで許可されていても、SCPで禁止されていれば実行できない。

### よくある誤答

「全アカウントのユーザーへ権限を配るためSCPを使う」は誤り。SCPは許可の配布ではなく、上限の設定である。

## 3.3 Control Tower

Control Towerは、Organizations、IAM Identity Center、Configなどを組み合わせ、Landing Zoneを標準化する。新規アカウント発行、予防的・発見的統制、ログ集約を一貫させたい場合に有効である。

既存の大規模Organizationsへ導入する場合は、既存統制との競合や登録対象を確認する。

## 3.4 IAM Identity Center

従業員が複数アカウントへアクセスする場合、各アカウントにIAMユーザーを作らない。外部IdPまたはIdentity Centerを使い、Permission Setからアカウント別ロールを払い出す。

- 人のアクセス：Identity Center
- ワークロードのアクセス：IAMロール
- アプリ利用者の認証：Cognitoなど

対象者を混同しない。

> 深掘り: [IAM](services/security/iam.md)、[IAM Identity Center](services/security/iam-identity-center.md)、[権限制御境界](comparisons/iam-boundaries-scp-condition-deep-dive.md)、[Control Tower](services/management/controltower.md)

---

# 4. ネットワークを経路として理解する

ネットワーク問題はサービス名ではなく、通信経路を一筆書きする。

```text
送信元
  → 名前解決
  → インターネット / 専用線 / VPN
  → エッジ
  → VPC入口
  → ルート
  → セキュリティ制御
  → ロードバランサー / Endpoint
  → 宛先
  → 戻り経路
```

## 4.1 VPC Peering、Transit Gateway、PrivateLink

### VPC Peering

少数VPCを1対1で接続する。推移的ルーティングはできない。接続数が増えるとメッシュが複雑になる。

### Transit Gateway

多数VPC、VPN、Direct Connectをハブ&スポークで接続する。ルートテーブルを分け、環境や部門間の通信を制御できる。

### PrivateLink

ネットワーク全体を相互接続せず、特定サービスだけをプライベート公開する。利用側はInterface Endpointを作る。CIDR重複を避けたい、提供側VPC全体を見せたくない場合に強い。

### 判断

- 2〜3 VPCを単純接続：Peering
- 多数VPCとオンプレを集約：Transit Gateway
- 特定サービスだけ提供：PrivateLink

## 4.2 Direct ConnectとVPN

- Site-to-Site VPN：インターネット上の暗号化トンネル。短期間で構築しやすい
- Direct Connect：専用接続。帯域と経路の一貫性が必要な場合

Direct Connect自体は暗号化を自動的に保証するものではない。必要に応じてMACsecやPublic/Transit VIF上のVPNなどを検討する。

高可用性が必要なら、異なるロケーション・デバイス・接続を使って冗長化する。単一DX + VPNは現実的な段階案だが、要件によっては二重DXが必要になる。

## 4.3 ALB、NLB、GWLB

| サービス | 主な層 | 選ぶ理由 |
|---|---|---|
| ALB | L7 HTTP/HTTPS | Host/Path/Headerルーティング、WebSocket、OIDC、WAF連携 |
| NLB | L4 TCP/UDP/TLS | 高性能、低遅延、静的IP、送信元IP維持が重要 |
| GWLB | L3/L4 | FirewallやIDS/IPSなど仮想アプライアンスを透過的に挿入 |

「NLBはALBより上位」ではない。見る情報と提供機能が違う。

## 4.4 CloudFront、Route 53、Global Accelerator

- **CloudFront**：HTTP(S)コンテンツをエッジでキャッシュ・配信
- **Route 53**：DNS応答を制御
- **Global Accelerator**：固定Anycast IPからAWSグローバルネットワークへ取り込み、正常なリージョナルエンドポイントへ転送

静的配信やキャッシュが主目的ならCloudFront。TCP/UDP、固定IP、迅速なエンドポイント切替が重要ならGlobal Accelerator。DNSベースの地理・重み付け・フェイルオーバーならRoute 53。

> 深掘り: [Networking Foundations](comparisons/networking-foundations-deep-dive.md)、[VPC](services/networking/vpc.md)、[Transit Gateway](services/networking/transitgateway.md)、[PrivateLink](services/networking/privatelink.md)

---

# 5. コンピュートを運用責任で選ぶ

コンピュートの選定は「実行できるか」だけでなく、誰がどこまで管理するかで決める。

| 選択肢 | 運用責任 | 向く状況 |
|---|---|---|
| EC2 | OS、パッチ、容量、ランタイム | 特殊OS、既存ソフト、細かな制御 |
| ECS on EC2 | EC2 + コンテナ基盤 | コンテナとインスタンス最適化を両立 |
| ECS on Fargate | タスク定義中心 | Kubernetes不要、運用負荷を減らす |
| EKS | Kubernetes管理面 + ワーカー方式 | K8s標準、既存エコシステム、移植性 |
| Lambda | 関数とイベント中心 | 短時間、イベント駆動、急変動 |
| Batch | ジョブ、キュー、Compute Environment | 大量バッチ、依存関係、Spot活用 |

## 5.1 Lambdaを選ばない理由

Lambdaは万能ではない。

- 実行時間やランタイム制約
- 長時間接続
- 大きな一時ストレージや特殊デバイス
- 常時高負荷で別方式の方が安い
- 既存アプリの変更量が大きい

これらではECS、EKS、EC2、Batchが適する。

## 5.2 ECSとEKS

Kubernetes要件が明示されていないのにEKSを選ぶと、運用複雑性を増やすことがある。既存K8s資産、組織標準、K8s APIやOperatorが必要ならEKS。AWS中心でシンプルにコンテナを動かすならECSが有力である。

## 5.3 Auto Scaling

スケーリング対象と指標を一致させる。

- Web：リクエスト数、CPU、応答時間
- ワーカー：SQSキュー深度 / 処理速度
- ストリーム：IteratorAge、シャード、消費遅延

CPUだけを見てはいけない。ボトルネックを表す業務指標が重要である。

> 深掘り: [EC2](services/compute/ec2.md)、[ECS](services/compute/ecs.md)、[EKS](services/compute/eks.md)、[Fargate](services/compute/fargate.md)、[Lambda](services/compute/lambda.md)

---

# 6. ストレージをアクセス方式から選ぶ

最初に「オブジェクト、ブロック、ファイル」のどれかを判定する。

| 種類 | AWS代表 | 特徴 |
|---|---|---|
| オブジェクト | S3 | APIアクセス、高耐久、静的データ、データレイク |
| ブロック | EBS | EC2へディスクとして接続、DBやOS |
| ファイル | EFS / FSx | NFS/SMBなど既存ファイルアクセス |

## 6.1 S3

S3は共有ファイルシステムではない。オブジェクトAPIを前提とする。静的コンテンツ、ログ、バックアップ、データレイクに向く。

選定時は次を見る。

- アクセス頻度とストレージクラス
- ライフサイクル
- Versioning、Object Lock
- CRR/SRR
- SSE-S3、SSE-KMS
- CloudFront OAC

## 6.2 EBS

単一AZのブロックストレージである。スナップショットはS3基盤へ保存されるが、通常のS3オブジェクトとして直接扱わない。

性能は容量、IOPS、スループットを分けて考える。gp3は容量とIOPS/スループットを独立設定でき、一般用途の第一候補になりやすい。io2系は高IOPS・高耐久性要件で検討する。

## 6.3 EFSとFSx

- Linux NFS共有、伸縮性：EFS
- Windows SMB、Active Directory：FSx for Windows File Server
- HPCとS3連携：FSx for Lustre
- NetApp機能、NFS/SMB/iSCSI、移行：FSx for ONTAP
- OpenZFS互換：FSx for OpenZFS

「共有ファイル」だけでは決めず、プロトコルと既存製品互換性を見る。

> 深掘り: [Storage比較](comparisons/storage-comparison.md)、[S3](services/storage/s3.md)、[EBS](services/storage/ebs.md)、[EFS](services/storage/efs.md)、[FSx](services/storage/fsx.md)

---

# 7. データベースをデータ特性から選ぶ

データベース選定では、まずデータモデルとアクセスパターンを見る。

## 7.1 RDS / Aurora

リレーショナル、SQL、トランザクション、既存DB互換が重要なら候補になる。

- RDS Multi-AZ：可用性と自動フェイルオーバー
- Read Replica：読み取りスケール、場合によりDR補助
- Aurora Reader：読み取り分散
- Aurora Global Database：クロスリージョン読み取りとDR
- Aurora Serverless v2：細粒度に変動する容量
- RDS Proxy：接続数急増やフェイルオーバー時の接続管理

Multi-AZとRead Replicaの目的を混同しない。

## 7.2 DynamoDB

キー・バリュー / ドキュメント型で、アクセスパターンが明確、高スケール、低レイテンシが必要な場合に強い。

- Partition Key設計
- オンデマンド / プロビジョンド
- GSI / LSI
- TTL
- Streams
- Global Tables
- DAX

複雑なJOINや自由なアドホックSQLを主目的に選ばない。

## 7.3 ElastiCache / MemoryDB

キャッシュは正本データベースと役割が違う。

- ElastiCache：キャッシュ、セッション、ランキング、Pub/Sub
- MemoryDB for Redis：Redis互換の耐久性あるプライマリDB用途

キャッシュ戦略では、TTL、失効、キャッシュミス、整合性を設計する。

## 7.4 Redshift

OLTPではなく分析用DWHである。大量データの集計、列指向、BIに向く。AuroraとRedshiftは「DB」という言葉だけで比較しない。トランザクション処理か分析処理かで分ける。

> 深掘り: [RDS/Aurora接続](comparisons/rds-aurora-connection-deep-dive.md)、[RDS](services/database/rds.md)、[Aurora](services/database/aurora.md)、[DynamoDB](services/database/dynamodb.md)

---

# 8. 疎結合とイベント駆動

同期呼び出しだけでシステムをつなぐと、一つの障害や遅延が連鎖する。SAPでは、SQS、SNS、EventBridge、Step Functionsの役割を分ける。

## 8.1 SQS

処理要求をキューへ蓄積し、送信側と処理側の速度差を吸収する。

- Standard：高スループット、少なくとも1回の配信を前提に冪等化
- FIFO：順序と重複排除が必要
- Visibility Timeout
- DLQ
- Long Polling

ワーカーのスケーリングはキュー深度だけでなく、到着率と1件あたり処理時間を考える。

## 8.2 SNS

一つのメッセージを複数購読者へPushするPub/Sub。SNS→複数SQSで、各処理系が独立して再試行できるファンアウトを作れる。

## 8.3 EventBridge

AWSサービス、独自アプリ、SaaSのイベントをルールで振り分ける。スケジュール実行、イベントバス、Archive/Replay、Schemaなどを使える。

### EventBridgeからECSを日次起動する場合

EventBridge SchedulerまたはルールはECS RunTaskを直接ターゲットにできる。人や外部クライアントへHTTP APIを公開する必要がなければAPI Gatewayは不要である。

```text
時刻到来
  → EventBridge Scheduler
  → ECS RunTask
  → Fargate Task
```

API Gatewayを追加すると、公開API、認証、スロットリングが必要なケースには有効だが、内部スケジュール起動だけなら経路と運用対象を増やす。

## 8.4 Step Functions

複数処理の順序、分岐、並列、待機、再試行、補償処理を状態機械として管理する。単なるイベント配送ならEventBridge、バッファならSQS、手順のオーケストレーションならStep Functions。

> 深掘り: [Messaging比較](comparisons/messaging-eventing-comparison.md)、[EventBridge](services/integration/eventbridge.md)、[SQS](services/integration/sqs.md)、[SNS](services/integration/sns.md)、[Step Functions](services/integration/stepfunctions.md)

---

# 9. 可用性・バックアップ・DRを分ける

この3つは似ているが別物である。

- **高可用性**：障害中もサービスを継続する
- **バックアップ**：過去時点へデータを戻す
- **DR**：大規模障害後に別環境で業務を復旧する

Read Replicaはバックアップの代わりではない。誤削除や論理破損が複製される可能性がある。

## 9.1 RTOとRPO

- RTO：復旧まで許容できる時間
- RPO：失ってよいデータ時間

「高可用性が必要」だけでは設計できない。数分か数時間か、データ損失ゼロか数時間許容かで構成が変わる。

## 9.2 DR戦略

| 戦略 | コスト | RTO | 概要 |
|---|---:|---:|---|
| Backup & Restore | 低 | 長 | バックアップから再構築 |
| Pilot Light | 中低 | 中長 | 最小限の中核を常時稼働 |
| Warm Standby | 中高 | 中短 | 縮小構成を常時稼働 |
| Multi-site Active/Active | 高 | 最短 | 複数サイトで同時稼働 |

最も高価なActive/Activeが常に正解ではない。要件を超える設計は不正解になり得る。

## 9.3 AWS Backup

複数サービスのバックアップポリシー、Vault、クロスアカウント・クロスリージョンコピーを集中管理する。Organizationsと組み合わせ、組織全体へポリシーを適用できる。

バックアップアカウントを分離し、Vault Lockや論理的エアギャップなどを検討することで、削除・改ざん耐性を高める。

> 深掘り: [AWS Backup](services/storage/backup.md)、[Resilience Hub](services/management/resilience-hub.md)、[バックアップとネットワーク設計](architecture/aws-backup-and-network-patterns.md)

---

# 10. 移行は7Rと停止許容時間で考える

移行問題では「どの移行サービスか」より先に、移行戦略を決める。

- Retire：廃止
- Retain：維持
- Rehost：そのまま移す
- Relocate：基盤単位で移す
- Replatform：一部マネージド化
- Repurchase：SaaS等へ置換
- Refactor：アプリを再設計

## 10.1 サーバー移行

AWS Application Migration Service（MGN）は、ブロックレベルで継続複製し、リホストを支援する。コード変更を最小化し、停止時間を短くしたい場合の候補になる。

## 10.2 DB移行

- DMS：データ移行とCDC
- Schema Conversion：異種DBのスキーマ・コード変換支援
- ネイティブバックアップ：同種エンジン、大容量、要件次第

異種DB移行では、DMSだけでストアドプロシージャ等が自動的に完全変換されるとは限らない。

## 10.3 ファイル移行

- DataSync：オンラインでNFS/SMB/S3等を転送
- Snow Family：ネットワーク帯域が不足する大容量オフライン移行
- Storage Gateway：移行だけでなくハイブリッド運用

データ量、回線速度、変更率、移行期限を計算する。

> 深掘り: [Migrationサービス群](services/migration/)

---

# 11. セキュリティを多層防御として設計する

セキュリティは一つのサービスで完成しない。

## 11.1 予防・検知・対応

- 予防：SCP、IAM、Security Group、WAF、KMS、Network Firewall
- 検知：CloudTrail、Config、GuardDuty、Inspector、Macie
- 集約：Security Hub
- 対応：EventBridge、Lambda、Systems Manager Automation

## 11.2 暗号化

- 保存時：SSE-S3、SSE-KMS、EBS/RDS暗号化
- 転送時：TLS、VPN、MACsec等
- 鍵管理：KMS Key Policy、IAM Policy、Grant

KMSでは「IAMで許可したから使える」とは限らない。Key Policyとの関係、クロスアカウント許可を確認する。

## 11.3 ログ集約

専用ログアーカイブアカウントへCloudTrailやConfig、VPC Flow Logs、サービスログを集約する。運用者が証跡を削除できないよう職務分離する。

## 11.4 エッジ防御

```text
利用者
  → Route 53 / Global Accelerator
  → CloudFront
  → Shield
  → WAF
  → ALB / API Gateway
  → アプリケーション
```

すべてを必ず置くのではなく、攻撃面と通信方式に合わせる。WAFはHTTP(S)のL7保護であり、任意TCP/UDPの制御を担うものではない。

> 深掘り: [認証認可と暗号化](comparisons/access-control-and-encryption.md)、[Edge Security比較](comparisons/edge-security-comparison.md)

---

# 12. コスト最適化は要件を落とすことではない

コスト最適化は、必要な成果を保ちながら無駄を減らすことである。

## 12.1 料金モデル

- On-Demand：変動・短期・予測困難
- Savings Plans：一定利用をコミットし計算料金を削減
- Reserved Instances：対象サービス・属性に応じた予約割引
- Spot：中断許容ワークロード

本番だから全てReserved、バッチだから全てSpot、という決め方はしない。

## 12.2 データ転送

SAPではコンピュート料金だけでなく、NAT Gateway、AZ間、リージョン間、インターネット転送を確認する。

代表的な最適化例：

- S3/DynamoDB Gateway EndpointでNAT経由を避ける
- CloudFrontでオリジン負荷と転送を最適化
- 同一AZ配置と可用性のトレードオフを理解
- 不要なクロスリージョン複製を避ける

## 12.3 Rightsizingと自動停止

Compute Optimizer、Cost Explorer、Trusted Advisor、CloudWatchを使い、過剰スペックを見直す。開発環境はスケジュール停止、バッチは実行時のみ起動、サーバーレスはアイドルコストを減らせる。

> 深掘り: [Cost Optimization](services/cost/)、[コスト比較](comparisons/cost-optimization-comparison.md)

---

# 13. 長文問題の解法

## Step 1: 最後の質問を先に読む

「最小コスト」「最小運用」「最小停止」「最も回復性が高い」など、評価軸を確認する。

## Step 2: 制約語へ印を付ける

- MUST：絶対条件
- SHOULD：重要条件
- CURRENT：現状
- GOAL：目標

## Step 3: データフローを書く

```text
Client → Edge → Entry → Compute → Queue → Database → Backup/DR
```

## Step 4: 各選択肢を失格条件で落とす

正解を直接探すより、次を確認する。

- 要件を満たさない
- 余計な運用が増える
- RTO/RPOに届かない
- プロトコルが違う
- コスト条件に反する
- 移行中の停止条件に反する

## Step 5: 「なぜ他ではないか」を一文で言う

例：

> EventBridgeからECSタスクを定刻起動でき、外部HTTP APIを公開する要件がないためAPI Gatewayは不要である。

この一文が作れれば、サービス選択を理解している。

---

# 14. 総合ケーススタディ

## ケース：複数部門の動画処理基盤をAWSへ移行する

### 要件

- 複数部門が動画をアップロードする
- 処理量は日によって大きく変動
- 失敗した動画は再処理したい
- 処理基盤の運用人数は少ない
- 部門ごとにコストを把握したい
- 本番環境のログを開発者が削除できないようにする
- 将来は海外利用者へ配信する

### 設計

```text
利用者
  → CloudFront
  → S3（直接アップロード）
  → EventBridge
  → SQS
  → ECS/Fargate ワーカー
  → S3 処理済みバケット
  → CloudFront 配信

Organizations / Control Tower
  ├─ Workload Account
  ├─ Log Archive Account
  ├─ Security Account
  └─ Shared Services Account
```

### 判断理由

1. **S3へ直接アップロード**
   - 大容量動画をアプリサーバー経由にせず、Presigned URLで負荷を減らす。

2. **SQSでバッファリング**
   - アップロード速度と動画処理速度を分離する。失敗時はDLQへ送る。

3. **ECS/Fargate**
   - 動画処理はLambdaの実行特性に合わない可能性がある。コンテナ化しつつEC2管理を減らす。

4. **キュー深度でスケール**
   - CPUだけでなく、未処理件数と処理時間から必要タスク数を決める。

5. **専用ログアーカイブアカウント**
   - ワークロード管理者と証跡管理者を分離する。

6. **CloudFront**
   - 海外配信、キャッシュ、オリジン保護を行う。

### 不採用案

- API Gateway経由で動画本体をアップロード：大容量ファイルの中継として不要な制約とコストを増やす
- EventBridgeから全処理を同期実行：大量到着時のバッファと再試行分離が弱い
- 常時最大台数のEC2：変動負荷に対してアイドルコストが大きい
- Read Replicaをバックアップ扱い：誤削除や論理障害への復旧手段にならない

### SAPでの着眼点

このケースの本質はサービス名ではない。

- 大容量転送をデータプレーンへ直送する
- 非同期化で速度差を吸収する
- 運用責任をマネージドサービスへ移す
- アカウント境界で職務分離する
- 将来要件をエッジ配信で吸収する

---

# 15. 最終チェックリスト

設計案を選ぶ前に、次を確認する。

## 要件

- 最重要評価軸は何か
- RTO/RPOは数値化されているか
- 停止時間、データ量、変更率は何か
- 運用人数と既存スキルは何か

## アカウント・権限

- アカウントを分けるべき境界か
- SCPとIAMを混同していないか
- 人、ワークロード、アプリ利用者の認証を分けたか

## ネットワーク

- 往路と戻り経路が成立するか
- 名前解決を考えたか
- Peering/TGW/PrivateLinkの目的が合うか
- NAT、AZ間、リージョン間の転送料を見たか

## コンピュート

- 特殊要件がなければ運用責任を減らせるか
- スケーリング指標はボトルネックを表すか
- Spot中断を許容できるか

## データ

- データモデルとアクセスパターンが合うか
- 可用性、読み取り拡張、バックアップを混同していないか
- 整合性とレイテンシの優先順位は何か

## 連携

- 同期である必要があるか
- バッファ、再試行、DLQ、冪等性を設計したか
- EventBridge、SQS、SNS、Step Functionsを役割で分けたか

## 回復性

- AZ障害とリージョン障害を分けたか
- バックアップを実際に復元テストしたか
- 要件を超える高価なDRを選んでいないか

## セキュリティ

- 予防、検知、対応があるか
- ログを攻撃者や運用者から分離したか
- 暗号鍵のポリシーとローテーションを確認したか

## コスト

- 常時稼働が必要か
- データ転送とNATコストを含めたか
- 割引モデルが負荷特性に合うか

---

# 次に読む資料

1. [LEARNING_PATH.md](LEARNING_PATH.md) で学習順序を決める
2. [SERVICES_INDEX.md](SERVICES_INDEX.md) でサービス辞書へ移動する
3. [comparisons/](comparisons/) で迷いやすい選択を比較する
4. [patterns/](patterns/) と [architecture-diagrams/](architecture-diagrams/) で構成を読む
5. [practice/exam-techniques.md](practice/exam-techniques.md) とシナリオ問題で判断を練習する

この読本を読み終えた状態とは、AWSサービスをすべて暗記した状態ではない。要件を読み、候補を比較し、**「この制約ではこれを選ぶ。別案はこの理由で選ばない」**と言える状態である。
