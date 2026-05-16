# SAP-C02 Scenario Set 01

Domain横断の重要論点をまとめた長文シナリオ型問題集。

---

## Question 1: マルチアカウント監査と自動修復

ある金融系企業はAWS Organizationsで120以上のアカウントを管理しており、各事業部が個別にVPC、S3、RDS、Lambdaを利用している。監査部門は、すべてのアカウントでAPI操作履歴を中央のログアーカイブアカウントへ集約し、S3バケットのパブリック公開やEBS未暗号化などの設定逸脱を継続的に検出し、重大な逸脱はセキュリティ運用アカウントで優先度付けしたうえで自動修復ワークフローへ流したい。各アカウントの開発チームには過度な管理権限を与えず、将来アカウントが増えても標準設定を一括展開したい。最も適切な組み合わせはどれか。

A. 各アカウントでCloudWatch Logsを有効化し、Logs InsightsでAPI操作を検索する。S3公開設定はCloudWatch Alarmで監視し、SNSで通知する。

B. Organizationsの管理アカウントでOrganization trailを作成し、ログアーカイブアカウントのS3へ配信する。AWS ConfigのConformance Packsを組織へ展開し、Security Hubを委任管理者アカウントで集約し、EventBridgeからSystems Manager Automationへ修復を流す。

C. すべてのアカウントにIAM AdministratorAccessを持つ監査ロールを作成し、監査担当者が定期的にコンソールで確認する。逸脱は手動で修正する。

D. GuardDutyだけをOrganizations全体で有効化し、すべてのコンプライアンス違反とAPI監査をGuardDuty findingsとして集約する。

E. CloudFormation StackSetsでアプリケーションリソースを展開し、CloudTrailとConfigは不要にする。CloudFormationのDrift DetectionでAPI操作履歴を確認する。

**正解: B**

### 解説

CloudTrailはAPI監査、Configは設定履歴/準拠評価、Security Hubは所見集約、EventBridgeは自動修復へのルーティング、Systems Manager Automationは修復実行に向く。複数アカウントへの標準配布にはCloudFormation StackSetsやOrganizations連携のConformance Packsを使う。

AはCloudWatch Logsを監査の一次情報として扱っている点が弱い。DはGuardDutyをコンプライアンス評価やAPI監査の代替にしている。EはCloudFormationの役割を広げすぎている。

関連: [CloudTrail](../services/management/cloudtrail.md), [Config](../services/management/config.md), [Security Hub](../services/management/security-hub.md), [CloudFormation](../services/management/cloudformation.md)

---

## Question 2: KMSクロスアカウントとS3データレイク

ある企業は中央データレイクアカウントのS3バケットに、各事業部アカウントからログと取引データを集約している。すべてのオブジェクトはSSE-KMSで暗号化し、分析アカウントのAthenaとQuickSightから一部のテーブルだけを参照させたい。データ所有部門は列単位で機密項目を制御したいが、分析者にはS3バケット全体の直接読み取り権限を与えたくない。最も適切な設計はどれか。

A. 分析者にS3FullAccessを付与し、KMS key policyではrootを許可する。AthenaはS3を直接読み取るためLake Formationは不要である。

B. S3バケットポリシー、IAMポリシー、KMS key policyを整合させ、Glue Data CatalogとLake Formationでテーブル/列レベル権限を管理する。Athena/QuickSight用の実行ロールに必要なS3/KMS/Lake Formation権限だけを付与する。

C. KMS key policyだけでS3オブジェクトとAthenaテーブルの権限をすべて管理する。IAMとLake Formationの設定は不要である。

D. Cognito User PoolでMFAを必須化すれば、Lake Formationの列レベル制御は自動的にすべてのQuickSight利用者へ適用される。

E. CloudFront OACをS3バケットの前段に置き、AthenaとQuickSightからCloudFront経由でクエリさせる。

**正解: B**

### 解説

SSE-KMSで暗号化されたS3データをクロスアカウントで読むには、S3/IAMだけでなくKMS key policy側も許可が必要。テーブル/列レベルのデータレイク権限はLake Formationの領域。QuickSightやAthenaの実行ロールに対して、S3、KMS、Glue/Lake Formation権限を整合させる必要がある。

Dはよくある罠。MFA済み認証とLake Formationのデータ権限は自動的に同義ではない。

関連: [KMS](../services/security/kms.md), [Athena](../services/analytics/athena.md), [Glue](../services/analytics/glue.md), [Lake Formation](../services/analytics/lakeformation.md), [QuickSight](../services/analytics/quicksight.md)

---

## Question 3: 移行後のコスト最適化

ある企業はオンプレミスの業務システムをMGNでEC2へリホストした。移行直後は安定稼働を優先したため、移行元サーバーと同等以上のサイズでEC2を起動している。3か月後、月額コストが想定より高く、経営層から30%の削減を求められた。ただし本番処理は日中にピークがあり、夜間は低負荷で、バッチ処理の一部は中断されても再実行できる。今後も基幹アプリは少なくとも3年間AWS上で稼働予定だが、インスタンスタイプは見直す可能性がある。最も適切な改善順序はどれか。

A. ただちに全EC2に3年Standard Reserved Instancesを購入し、その後Compute Optimizerを確認する。

B. Cost Explorerでサービス/タグ/アカウント別の支出を分析し、Compute OptimizerでRightsizingを行い、Auto Scalingやスケジュール停止、Spot適用可能なバッチを整理したうえで、安定ベースラインにCompute Savings Plansを適用する。

C. すべての本番EC2をSpot Instancesに変更し、安定ベースラインのコストを最大限下げる。

D. Savings Plansを購入すれば過剰プロビジョニングも自動的に解消されるため、利用率分析は不要である。

E. AWS Budgetsだけを設定し、予算超過通知によってコスト最適化を完了する。

**正解: B**

### 解説

コスト最適化は、まず可視化、次にRightsizing、その後に構造改善、最後に安定ベースラインへのコミットメント割引が安全。RIやSavings Plansを過剰サイズのまま購入すると、非効率を固定化する。Spotは中断可能処理には有効だが、本番基幹の全面置き換えには不適切。

関連: [Cost Explorer](../services/cost/cost-explorer.md), [Compute Optimizer](../services/cost/compute-optimizer.md), [Savings Plans](../services/cost/savings-plans.md), [Reserved Instances](../services/cost/reserved-instances.md)

---

## Question 4: データレイクと部門横断ガバナンス

ある小売企業は、POS、ECサイト、在庫、マーケティング、カスタマーサポートのデータをS3に集約している。データエンジニアはETLで形式を整え、アナリストはSQLで探索し、経営層はダッシュボードで確認したい。一方で、部門ごとに利用可能なデータセットを明確にし、利用申請と承認の履歴を残し、顧客個人情報の列は許可された利用者にしか見せたくない。最も適切な役割分担はどれか。

A. S3に全データを置き、全員にS3 Read権限を与える。SQL分析、ETL、BI、利用申請はすべてS3 Selectで実現する。

B. Glue Data CatalogとCrawlerでメタデータを管理し、Glue JobsでETLを行い、AthenaでS3上のデータをSQL分析する。Lake Formationでテーブル/列レベル権限を管理し、QuickSightでBIを提供する。部門横断の発見・申請・共有にはDataZoneを使う。

C. Redshiftだけに全データをロードし、ETL、権限申請、BI、カタログをすべてRedshiftで完結させる。

D. QuickSightだけでETL、データカタログ、列レベル権限、利用申請、承認ワークフローを管理する。

E. OpenSearchにすべてのデータを投入し、SQL分析とBIをOpenSearch Dashboardsだけで実行する。

**正解: B**

### 解説

S3データレイクでは、Glueがカタログ/ETL、AthenaがSQLクエリ、Lake Formationがデータレイク権限、QuickSightがBI、DataZoneがデータ発見・共有・申請の上位ガバナンスを担う。各サービスの役割を混同しないことが重要。

関連: [Athena](../services/analytics/athena.md), [Glue](../services/analytics/glue.md), [DataZone](../services/analytics/datazone.md), [QuickSight](../services/analytics/quicksight.md)

---

## Question 5: Aurora Global DatabaseとDR

あるSaaS企業はAurora PostgreSQLで主要トランザクションを処理しており、現在は東京リージョンのMulti-AZ構成で稼働している。アジアと北米の利用者が増え、読み取り遅延を下げたい。また、リージョン障害時には短いRPOで別リージョンへ切り替えたい。ただしアプリケーションは単一ライター前提で作られており、複数リージョンから同時に競合する書き込みを行う設計にはなっていない。最も適切な設計はどれか。

A. Aurora Global Databaseを構成し、プライマリリージョンで書き込み、セカンダリリージョンを読み取りに使う。障害時は計画的switchoverまたはfailoverでプライマリを移す。

B. Aurora Global Databaseを構成すれば、すべてのセカンダリリージョンが自動的に完全なmulti-writerになるため、アプリケーション変更なしでActive/Active書き込みが実現できる。

C. DynamoDB Global Tablesに変更せず、Auroraのまま全リージョンで同時書き込みを行えば競合解決は自動で完全に処理される。

D. S3 Cross-Region Replicationを使ってAuroraのトランザクションログを複製し、障害時にS3からRDSへ自動復旧する。

E. CloudFrontをAuroraの前段に置けば、データベースの読み取り遅延とリージョンDRを同時に解決できる。

**正解: A**

### 解説

Aurora Global Databaseは、通常1つのプライマリリージョンに書き込み、セカンダリリージョンは低レイテンシ読み取りとDRに使う。セカンダリを自動的に一般的なmulti-writerとして扱うのは危険。multi-region active-active書き込みが中心要件ならDynamoDB Global Tablesなど別の設計も検討する。

関連: [Disaster Recovery](../patterns/disaster-recovery.md), [Aurora](../services/database/aurora.md)
