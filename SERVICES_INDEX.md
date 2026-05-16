# AWS Services Index (SAP-C02 Oriented)

このインデックスは、SAP-C02 の学習優先度に合わせて AWS サービスを整理したもの。公式の in-scope services は非網羅かつ変更されるため、ここでは **試験で設計判断に直結しやすい順** に Tier 分類する。

凡例: `✅` = 詳細ページあり / `◻` = 今後作成予定

---

## Tier 1 - SAP-C02 コアサービス

### Compute / Containers

| 状態 | サービス | 主な試験論点 |
|---|---|---|
| ✅ | [Amazon EC2](services/compute/ec2.md) | インスタンスタイプ、Auto Scaling、購入オプション、移行先 |
| ✅ | [AWS Lambda](services/compute/lambda.md) | サーバーレス、イベント駆動、同時実行、VPC接続 |
| ✅ | [Amazon ECS](services/compute/ecs.md) | コンテナ実行、ALB連携、Blue/Green、Task Role |
| ✅ | [Amazon EKS](services/compute/eks.md) | Kubernetes移行、IRSA、マルチクラウド/既存K8s |
| ✅ | [AWS Fargate](services/compute/fargate.md) | サーバーレスコンテナ、運用負荷削減、Fargate Spot |
| ◻ | AWS Batch | 大規模バッチ、ジョブキュー、Spot活用 |
| ◻ | AWS App Runner | コンテナWebアプリの簡易デプロイ |
| ✅ | [Elastic Load Balancing](services/networking/elb.md) | ALB/NLB/GWLBの使い分け |

### Storage

| 状態 | サービス | 主な試験論点 |
|---|---|---|
| ✅ | [Amazon S3](services/storage/s3.md) | ストレージクラス、CRR、ライフサイクル、暗号化、OAC |
| ◻ | Amazon EBS | EC2ブロックストレージ、スナップショット、性能設計 |
| ✅ | [Amazon EFS](services/storage/efs.md) | 共有ファイル、NFS、Linux、マルチAZ |
| ✅ | [Amazon FSx](services/storage/fsx.md) | Windows / Lustre / ONTAP / OpenZFS の使い分け |
| ✅ | [AWS Storage Gateway](services/storage/storage-gateway.md) | ハイブリッドストレージ、File/Volume/Tape Gateway |
| ◻ | Amazon S3 Glacier | アーカイブ、復元時間、コスト |
| ✅ | [AWS Backup](services/storage/backup.md) | 一元バックアップ、クロスリージョン/クロスアカウント |

### Database

| 状態 | サービス | 主な試験論点 |
|---|---|---|
| ✅ | [Amazon RDS](services/database/rds.md) | Multi-AZ、Read Replica、移行、運用負荷削減 |
| ✅ | [Amazon Aurora](services/database/aurora.md) | Global Database、Serverless v2、可用性/性能 |
| ✅ | [Amazon DynamoDB](services/database/dynamodb.md) | Global Tables、オンデマンド、TTL、DAX |
| ✅ | [Amazon ElastiCache](services/database/elasticache.md) | Redis/Memcached、キャッシュ、セッション管理 |
| ✅ | [Amazon Redshift](services/analytics/redshift.md) | DWH、Spectrum、RA3、データ分析基盤 |
| ✅ | [Amazon DocumentDB](services/database/documentdb.md) | MongoDB互換、移行判断 |
| ✅ | [Amazon Neptune](services/database/neptune.md) | グラフDB、関係性探索 |

### Networking & Content Delivery

| 状態 | サービス | 主な試験論点 |
|---|---|---|
| ✅ | [Amazon VPC](services/networking/vpc.md) | サブネット、ルート、NAT、Endpoint、設計基盤 |
| ✅ | [Amazon Route 53](services/networking/route53.md) | ルーティングポリシー、フェイルオーバー、ヘルスチェック |
| ✅ | [Amazon CloudFront](services/networking/cloudfront.md) | キャッシュ、OAC、カスタムオリジン、TLS |
| ✅ | [AWS Transit Gateway](services/networking/transitgateway.md) | ハブ&スポーク、マルチアカウント接続 |
| ✅ | [AWS Direct Connect](services/networking/direct-connect.md) | 専用線、DX Gateway、Transit VIF、冗長化 |
| ✅ | [AWS Site-to-Site VPN](services/networking/site-to-site-vpn.md) | ハイブリッド接続、バックアップ回線 |
| ✅ | [AWS PrivateLink](services/networking/privatelink.md) | プライベートサービス公開、Interface/Gateway Endpoint |
| ✅ | [AWS Global Accelerator](services/networking/global-accelerator.md) | Anycast IP、リージョン切替、低レイテンシ |
| ✅ | [Elastic Load Balancing](services/networking/elb.md) | ALB/NLB/GWLB、L7/L4/アプライアンス連携 |

### Security, Identity & Compliance

| 状態 | サービス | 主な試験論点 |
|---|---|---|
| ✅ | [AWS IAM & Organizations](services/security/iam.md) | 権限評価、SCP、Permission Boundary、クロスアカウント |
| ✅ | [Amazon GuardDuty](services/security/guardduty.md) | 脅威検出、Organizations集約 |
| ✅ | [AWS KMS](services/security/kms.md) | Key Policy、Grants、SSE-KMS、クロスアカウント |
| ◻ | AWS IAM Identity Center | 社員向けSSO、Permission Set、Organizations |
| ✅ | [Amazon Cognito](services/security/cognito.md) | User Pool / Identity Pool、アプリ認証 |
| ✅ | [AWS Secrets Manager](services/security/secrets-manager.md) | シークレット管理、自動ローテーション |
| ✅ | [AWS Certificate Manager](services/security/acm.md) | TLS証明書、CloudFront/ALB連携 |
| ✅ | [AWS WAF](services/security/waf.md) | L7保護、CloudFront/ALB/API Gateway |
| ✅ | [AWS Shield](services/security/shield.md) | DDoS対策、Standard/Advanced |
| ✅ | [AWS Network Firewall](services/security/network-firewall.md) | VPC境界のL3-L7制御 |
| ✅ | [AWS Security Hub](services/management/security-hub.md) | セキュリティ集約、標準準拠チェック |
| ✅ | [AWS Config](services/management/config.md) | リソース設定評価、コンプライアンス |
| ✅ | [AWS CloudTrail](services/management/cloudtrail.md) | API監査、証跡、Lake |

### Management & Governance

| 状態 | サービス | 主な試験論点 |
|---|---|---|
| ✅ | [Amazon CloudWatch](services/management/cloudwatch.md) | メトリクス、Logs、Alarms、Synthetics |
| ✅ | [AWS Systems Manager](services/management/ssm.md) | Session Manager、Patch、Automation、Parameter Store |
| ✅ | [AWS Control Tower](services/management/controltower.md) | Landing Zone、ガードレール、Account Factory |
| ✅ | [AWS Fault Injection Simulator](services/management/fis.md) | 障害注入、DRテスト、信頼性検証 |
| ✅ | [AWS X-Ray](services/management/xray.md) | 分散トレーシング |
| ✅ | [AWS CloudFormation](services/management/cloudformation.md) | IaC、StackSets、Change Set、Drift Detection |
| ◻ | AWS Trusted Advisor | コスト/セキュリティ/信頼性チェック |
| ◻ | AWS Service Catalog | 標準化されたプロビジョニング |
| ◻ | AWS License Manager | ライセンス管理、移行時の最適化 |
| ✅ | [AWS Compute Optimizer](services/cost/compute-optimizer.md) | Rightsizing、コスト最適化 |
| ◻ | AWS Resilience Hub | 回復性評価、RTO/RPO検証 |

### Analytics / Data Lake

| 状態 | サービス | 主な試験論点 |
|---|---|---|
| ✅ | [Amazon Athena](services/analytics/athena.md) | S3上のクエリ、サーバーレス分析 |
| ✅ | [AWS Glue](services/analytics/glue.md) | ETL、Data Catalog、Crawler |
| ✅ | [AWS Lake Formation](services/analytics/lakeformation.md) | データレイク権限、行/列レベル制御 |
| ✅ | [Amazon Kinesis](services/analytics/kinesis.md) | ストリーミング、Data Streams / Firehose |
| ✅ | [Amazon OpenSearch Service](services/analytics/opensearch.md) | ログ検索、全文検索、可視化 |
| ◻ | Amazon EMR | Spark/Hadoop、ビッグデータ処理 |
| ✅ | [Amazon QuickSight / Amazon Quick Sight](services/analytics/quicksight.md) | BI、埋め込み、可視化 |
| ✅ | [Amazon DataZone](services/analytics/datazone.md) | データガバナンス、カタログ |
| ◻ | AWS Data Exchange | 外部データセット利用 |

### Application Integration

| 状態 | サービス | 主な試験論点 |
|---|---|---|
| ✅ | [Amazon API Gateway](services/integration/apigateway.md) | REST/HTTP/WebSocket API、認証、スロットリング |
| ✅ | [AWS Step Functions](services/integration/stepfunctions.md) | ワークフロー、Saga、Standard/Express |
| ✅ | [Amazon MQ](services/integration/mq.md) | 既存MQ移行、ActiveMQ/RabbitMQ |
| ✅ | [AWS App Mesh](services/integration/appmesh.md) | サービスメッシュ、mTLS、トラフィック制御 |
| ✅ | [Amazon EventBridge](services/integration/eventbridge.md) | イベントバス、疎結合、SaaS連携、Archive/Replay |
| ✅ | [Amazon SQS](services/integration/sqs.md) | キュー、バッファ、DLQ、FIFO |
| ✅ | [Amazon SNS](services/integration/sns.md) | Pub/Sub、ファンアウト、通知 |
| ◻ | AWS AppSync | GraphQL、リアルタイムAPI |

### Migration & Transfer

| 状態 | サービス | 主な試験論点 |
|---|---|---|
| ✅ | [AWS Application Migration Service](services/migration/mgn.md) | リホスト、継続レプリケーション、カットオーバー |
| ✅ | [AWS Database Migration Service](services/migration/dms.md) | DB移行、CDC、同種/異種移行 |
| ✅ | [AWS Schema Conversion Tool / DMS Schema Conversion](services/migration/sct.md) | 異種DBのスキーマ変換、評価レポート |
| ✅ | [AWS DataSync](services/migration/datasync.md) | ファイル/オブジェクト転送、オンライン移行 |
| ✅ | [AWS Migration Hub](services/migration/migration-hub.md) | 移行追跡、アプリ単位管理 |
| ✅ | [AWS Application Discovery Service](services/migration/application-discovery-service.md) | サーバー発見、依存関係、移行計画 |
| ✅ | [AWS Snow Family](services/migration/snow-family.md) | オフライン大容量移行、エッジ処理 |
| ✅ | [AWS Transfer Family](services/migration/transfer-family.md) | SFTP/FTPS/FTP/AS2 のマネージド転送 |

---

## Tier 2 - 頻出ではないが設計判断で効くサービス

### Cloud Financial Management

- [AWS Cost Explorer](services/cost/cost-explorer.md)
- [AWS Budgets](services/cost/budgets.md)
- AWS Cost and Usage Report
- [Savings Plans](services/cost/savings-plans.md)
- [Reserved Instances](services/cost/reserved-instances.md)
- AWS Billing Conductor
- AWS Marketplace

### Developer Tools / DevOps

- AWS CodePipeline
- AWS CodeBuild
- AWS CodeDeploy
- AWS CodeArtifact
- AWS CodeCommit
- AWS CodeGuru
- AWS CDK
- AWS SAM
- AWS Amplify
- AWS Proton

### End User Computing / Business Applications

- Amazon WorkSpaces
- Amazon AppStream 2.0
- Amazon WorkSpaces Web
- Amazon Connect
- Amazon SES
- Amazon Pinpoint
- Amazon WorkMail

### Machine Learning

| 状態 | サービス | 主な用途 |
|---|---|---|
| ✅ | [Amazon SageMaker](services/ml/sagemaker.md) | ML開発/学習/推論基盤 |
| ✅ | [Amazon Bedrock](services/ml/bedrock.md) | 生成AI、基盤モデル、RAG |
| ◻ | Amazon Rekognition | 画像/動画認識 |
| ◻ | Amazon Comprehend | 自然言語処理 |
| ◻ | Amazon Textract | OCR / 文書抽出 |
| ◻ | Amazon Transcribe | 音声文字起こし |
| ◻ | Amazon Translate | 機械翻訳 |
| ◻ | Amazon Kendra | エンタープライズ検索 |
| ◻ | Amazon Lex | チャットボット |
| ◻ | Amazon Polly | 音声合成 |
| ◻ | Amazon Personalize | レコメンド |
| ◻ | Amazon Forecast | 時系列予測 |
| ◻ | Amazon Fraud Detector | 不正検知 |

### Hybrid / Edge

- AWS Outposts
- AWS Local Zones
- AWS Wavelength
- VMware Cloud on AWS
- Amazon ECS Anywhere
- Amazon EKS Anywhere
- Amazon EKS Distro

### IoT / Industry

- AWS IoT Core
- AWS IoT Greengrass
- AWS IoT SiteWise
- AWS IoT Device Defender
- AWS IoT Device Management
- AWS IoT Events
- AWS IoT TwinMaker
- AWS IoT Analytics

### Additional Data / Specialized Databases

- Amazon Managed Streaming for Apache Kafka (MSK)
- Amazon Managed Service for Apache Flink
- Amazon Timestream
- Amazon Keyspaces
- Amazon QLDB
- Amazon Managed Blockchain
- Amazon MemoryDB for Redis

---

## Tier 3 - ニッチ / 低優先 / 参照用

- AWS ParallelCluster
- AWS Thinkbox Deadline
- AWS RoboMaker
- Amazon Lookout for Equipment
- Amazon Lookout for Metrics
- Amazon Lookout for Vision
- Amazon Monitron
- Amazon Panorama / AWS Panorama
- AWS Ground Station
- AWS Braket
- Amazon Chime SDK
- AWS Device Farm
- Amazon Honeycode
- Amazon Q Business
- Amazon Q Developer
- Amazon Q in QuickSight
- Amazon Q in Connect
- Amazon Bedrock Agents
- Amazon Bedrock Knowledge Bases
- Amazon Bedrock Guardrails
- SageMaker Ground Truth
- SageMaker Pipelines
- SageMaker Feature Store
- SageMaker Clarify
- SageMaker Debugger
- SageMaker Model Monitor
- SageMaker Autopilot
- SageMaker JumpStart
- SageMaker Canvas

---

## 次に詳細ページ化すべき優先順位

1. `practice/` — 長文シナリオ型問題集。読む教材から解く教材へ移行する。
2. `services/database/aurora.md` / `dynamodb.md` 深掘り — Global Database、Global Tables、整合性、DR、コスト。
3. `services/hybrid/` — Outposts / Local Zones / Wavelength / VMware Cloud on AWS。
4. `services/devops/` — CodePipeline / CodeDeploy / CodeBuild / CDK / SAM。
5. `services/security/iam-identity-center.md` — 社員SSOとPermission SetをIAM/Cognitoと比較。
