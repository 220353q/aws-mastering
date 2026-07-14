# SAP-C02 公式タスク・カバレッジマトリクス

> 公式試験ガイド Version 1.2 の4ドメイン・20タスクを、リポジトリ内の通読資料、比較表、サービス辞書、演習へ対応付ける。

このページの目的は、ファイル数を増やすことではない。**公式タスクごとに、理解・比較・演習のどこが不足しているかを見えるようにすること**である。

## 状態の定義

| 状態 | 意味 |
|---|---|
| A | 通読、比較、個別サービス、演習が揃っている |
| B | 基礎資料はあるが、比較または演習が薄い |
| C | 個別サービスはあるが、タスク横断の判断材料が不足 |
| D | 主要資料が未整備 |

## 全体サマリー

| Domain | 比重 | タスク数 | 更新後の重点 |
|---|---:|---:|---|
| 1. Organizational Complexity | 26% | 5 | Hybrid DNS、マルチアカウント運用、コスト可視化 |
| 2. New Solutions | 29% | 6 | デプロイ、性能、データ転送コスト |
| 3. Continuous Improvement | 25% | 5 | 観測→診断→改善→検証の運用サイクル |
| 4. Migration and Modernization | 20% | 4 | 評価、Wave、Cutover、Rollback、モダナイズ |

---

# Domain 1: Design Solutions for Organizational Complexity

## Task 1.1 Architect network connectivity strategies

**公式論点**

- 複数VPC、オンプレミス、コロケーション接続
- Region/AZ選択とレイテンシ
- Hybrid DNS
- Network segmentation
- Traffic flow troubleshooting
- Service endpoints

**対応資料**

- [AWS SAP設計読本](SAP_DESIGN_READER.md#4-ネットワークを経路として理解する)
- [Networking Foundations Deep Dive](comparisons/networking-foundations-deep-dive.md)
- [Hybrid DNS Deep Dive](comparisons/hybrid-dns-deep-dive.md)
- [Amazon VPC](services/networking/vpc.md)
- [Transit Gateway](services/networking/transitgateway.md)
- [Direct Connect](services/networking/direct-connect.md)
- [Site-to-Site VPN](services/networking/site-to-site-vpn.md)
- [PrivateLink](services/networking/privatelink.md)

**状態: A-**

残課題は、BGP・MTU・非対称ルーティングの計算型演習を増やすこと。

## Task 1.2 Prescribe security controls

**公式論点**

- IAM / IAM Identity Center
- Cross-account access
- Third-party IdP
- KMS / ACM
- Central security notifications and auditing
- Access Analyzer / Security Hub / Inspector

**対応資料**

- [Access Control and Encryption](comparisons/access-control-and-encryption.md)
- [IAM Boundaries / SCP / Condition](comparisons/iam-boundaries-scp-condition-deep-dive.md)
- [IAM](services/security/iam.md)
- [IAM Identity Center](services/security/iam-identity-center.md)
- [KMS](services/security/kms.md)
- [Security Hub](services/management/security-hub.md)
- [CloudTrail](services/management/cloudtrail.md)

**状態: A-**

残課題は、Access Analyzer、Inspector、委任管理者を横断したケース問題。

## Task 1.3 Design reliable and resilient architectures

**公式論点**

- RTO / RPO
- DR戦略
- Data backup and restoration
- Automatic recovery
- Scale-up / Scale-out

**対応資料**

- [AWS SAP設計読本](SAP_DESIGN_READER.md#9-可用性バックアップdrを分ける)
- [Continuous Improvement Playbook](CONTINUOUS_IMPROVEMENT_PLAYBOOK.md)
- [AWS Backup](services/storage/backup.md)
- [Resilience Hub](services/management/resilience-hub.md)
- [Fault Injection Simulator](services/management/fis.md)
- [Elastic Disaster Recovery](services/migration/elastic-disaster-recovery.md)

**状態: A-**

残課題は、RTO/RPOから構成と費用を選ぶ計算問題。

## Task 1.4 Design a multi-account AWS environment

**公式論点**

- Organizations / Control Tower
- Account structure
- Resource sharing
- Central logging
- Multi-account event notifications
- Governance model

**対応資料**

- [AWS SAP設計読本](SAP_DESIGN_READER.md#3-マルチアカウントとガバナンス)
- [IAM](services/security/iam.md)
- [Control Tower](services/management/controltower.md)
- [CloudTrail](services/management/cloudtrail.md)
- [CloudFormation](services/management/cloudformation.md)

**状態: B+**

追加候補:

- AWS RAM
- Delegated Administrator
- Organization-wide EventBridge
- Tag Policies / Backup Policies
- Log Archive / Security Tooling / Networkアカウントの責任分界

## Task 1.5 Determine cost optimization and visibility strategies

**公式論点**

- Trusted Advisor / Pricing Calculator / Cost Explorer / Budgets
- Reserved Instances / Savings Plans / Spot
- Compute Optimizer / S3 Storage Lens
- Tagging and business-unit allocation

**対応資料**

- [Cost Modeling and Data Transfer](comparisons/cost-modeling-and-data-transfer.md)
- [Cost Explorer](services/cost/cost-explorer.md)
- [Budgets](services/cost/budgets.md)
- [Savings Plans](services/cost/savings-plans.md)
- [Reserved Instances](services/cost/reserved-instances.md)
- [Compute Optimizer](services/cost/compute-optimizer.md)

**状態: A-**

残課題は、CUR + Athenaの実践クエリとタグ欠損時の配賦設計。

---

# Domain 2: Design for New Solutions

## Task 2.1 Design a deployment strategy to meet business requirements

**公式論点**

- IaC
- CI/CD
- Change management
- Configuration management
- Upgrade path
- Rollback mechanisms
- Managed service adoption

**対応資料**

- [Deployment and Rollback Strategies](comparisons/deployment-and-rollback-strategies.md)
- [CloudFormation](services/management/cloudformation.md)
- [Systems Manager](services/management/ssm.md)

**状態: A-**

残課題は、CodePipeline、CodeBuild、CodeDeploy、CDK、SAMの個別ページ。

## Task 2.2 Design a solution to ensure business continuity

**公式論点**

- Multi-AZ / Multi-Region
- Route 53 routing
- Replication
- DR testing
- Automated backup
- Centralized monitoring and proactive recovery

**対応資料**

- [AWS SAP設計読本](SAP_DESIGN_READER.md#9-可用性バックアップdrを分ける)
- [Continuous Improvement Playbook](CONTINUOUS_IMPROVEMENT_PLAYBOOK.md)
- [Route 53](services/networking/route53.md)
- [AWS Backup](services/storage/backup.md)
- [Aurora](services/database/aurora.md)
- [DynamoDB](services/database/dynamodb.md)

**状態: A**

## Task 2.3 Determine security controls based on requirements

**公式論点**

- Least privilege
- Inbound / outbound flows
- WAF / Shield / GuardDuty / Security Hub
- Encryption
- Endpoints
- Patch management
- Credential management

**対応資料**

- [Access Control and Encryption](comparisons/access-control-and-encryption.md)
- [Edge Security Comparison](comparisons/edge-security-comparison.md)
- [IAM](services/security/iam.md)
- [KMS](services/security/kms.md)
- [Secrets Manager](services/security/secrets-manager.md)
- [Systems Manager](services/management/ssm.md)

**状態: A-**

残課題は、Patch ManagerとMaintenance Windowの実践ケース。

## Task 2.4 Design a strategy to meet reliability requirements

**公式論点**

- Storage and replication
- Auto Scaling
- Loose coupling
- Failover operation
- DNS policies
- Service quotas

**対応資料**

- [AWS SAP設計読本](SAP_DESIGN_READER.md)
- [Messaging and Eventing Comparison](comparisons/messaging-eventing-comparison.md)
- [Storage Comparison](comparisons/storage-comparison.md)
- [Route 53](services/networking/route53.md)
- [SQS](services/integration/sqs.md)
- [Step Functions](services/integration/stepfunctions.md)

**状態: A-**

残課題は、Service Quotasを含む成長予測問題。

## Task 2.5 Design a solution to meet performance objectives

**公式論点**

- Performance monitoring
- Storage options
- Instance families
- Purpose-built databases
- Elastic architecture
- Caching / buffering / replicas
- Rightsizing

**対応資料**

- [Performance Design Reader](PERFORMANCE_DESIGN_READER.md)
- [RDS / Aurora Connection Deep Dive](comparisons/rds-aurora-connection-deep-dive.md)
- [EC2](services/compute/ec2.md)
- [EBS](services/storage/ebs.md)
- [ElastiCache](services/database/elasticache.md)
- [DynamoDB](services/database/dynamodb.md)

**状態: A-**

残課題は、Kinesis shard、DynamoDB partition、EBS帯域の計算演習。

## Task 2.6 Determine a cost optimization strategy

**公式論点**

- Rightsizing
- Pricing models
- Storage tiering
- Data transfer modeling
- Managed service tradeoffs
- Expenditure controls

**対応資料**

- [Cost Modeling and Data Transfer](comparisons/cost-modeling-and-data-transfer.md)
- [Cost Optimization Comparison](comparisons/cost-optimization-comparison.md)
- `services/cost/`

**状態: A**

---

# Domain 3: Continuous Improvement for Existing Solutions

## Task 3.1 Improve operational excellence

**公式論点**

- Monitoring / logging
- Alerting / automated remediation
- CI/CD and deployment improvement
- Configuration management
- Failure scenario exercises

**対応資料**

- [Continuous Improvement Playbook](CONTINUOUS_IMPROVEMENT_PLAYBOOK.md)
- [Deployment and Rollback Strategies](comparisons/deployment-and-rollback-strategies.md)
- [CloudWatch](services/management/cloudwatch.md)
- [Systems Manager](services/management/ssm.md)
- [Fault Injection Simulator](services/management/fis.md)

**状態: A**

## Task 3.2 Improve security

**公式論点**

- Retention / sensitivity / regulation
- Config rules and automated remediation
- Secrets
- Least privilege auditing
- Traceability
- Vulnerability response
- Patch / backup process

**対応資料**

- [Continuous Improvement Playbook](CONTINUOUS_IMPROVEMENT_PLAYBOOK.md)
- [Access Control and Encryption](comparisons/access-control-and-encryption.md)
- [Config](services/management/config.md)
- [CloudTrail](services/management/cloudtrail.md)
- [Secrets Manager](services/security/secrets-manager.md)
- [AWS Backup](services/storage/backup.md)

**状態: A-**

残課題は、Macie / Inspector / GuardDutyの所見から修復へつなぐ演習。

## Task 3.3 Improve performance

**公式論点**

- Auto Scaling / Instance Fleets / Placement Groups
- Global Accelerator / CloudFront / Edge
- SLA / KPI
- Bottleneck analysis
- Testing remediation
- Rightsizing

**対応資料**

- [Performance Design Reader](PERFORMANCE_DESIGN_READER.md)
- [Continuous Improvement Playbook](CONTINUOUS_IMPROVEMENT_PLAYBOOK.md)
- [Global Accelerator](services/networking/global-accelerator.md)
- [CloudFront](services/networking/cloudfront.md)
- [CloudWatch](services/management/cloudwatch.md)

**状態: A**

## Task 3.4 Improve reliability

**公式論点**

- Growth and usage trends
- Single points of failure
- Data replication
- Self-healing
- Elasticity
- Service quotas

**対応資料**

- [Continuous Improvement Playbook](CONTINUOUS_IMPROVEMENT_PLAYBOOK.md)
- [AWS SAP設計読本](SAP_DESIGN_READER.md)
- [Resilience Hub](services/management/resilience-hub.md)
- [Fault Injection Simulator](services/management/fis.md)

**状態: A-**

残課題は、依存サービス障害と部分劣化のケーススタディ。

## Task 3.5 Identify opportunities for cost optimizations

**公式論点**

- Underused / overused resources
- Unused resources
- Billing alarms
- CUR granular analysis
- Cost allocation tagging
- Network transfer cost

**対応資料**

- [Cost Modeling and Data Transfer](comparisons/cost-modeling-and-data-transfer.md)
- `services/cost/`

**状態: A-**

残課題は、CURサンプルデータを使う分析演習。

---

# Domain 4: Accelerate Workload Migration and Modernization

## Task 4.1 Select workloads and processes for potential migration

**公式論点**

- Portfolio assessment
- Asset planning
- Wave planning
- 7Rs
- TCO

**対応資料**

- [Migration and Modernization Reader](MIGRATION_AND_MODERNIZATION_READER.md)
- [Migration Hub](services/migration/migration-hub.md)
- [Application Discovery Service](services/migration/application-discovery-service.md)

**状態: A**

## Task 4.2 Determine the optimal migration approach

**公式論点**

- Data / application / database transfer
- DX / VPN / DNS
- Identity and Directory Service
- Governance
- Security for migration tools

**対応資料**

- [Migration and Modernization Reader](MIGRATION_AND_MODERNIZATION_READER.md)
- [MGN](services/migration/mgn.md)
- [DMS](services/migration/dms.md)
- [Schema Conversion](services/migration/sct.md)
- [DataSync](services/migration/datasync.md)
- [Snow Family](services/migration/snow-family.md)
- [Transfer Family](services/migration/transfer-family.md)

**状態: A**

## Task 4.3 Determine a new architecture for existing workloads

**公式論点**

- Compute platform
- Container hosting
- Storage platform
- Database platform

**対応資料**

- [AWS SAP設計読本](SAP_DESIGN_READER.md)
- [Migration and Modernization Reader](MIGRATION_AND_MODERNIZATION_READER.md)
- [Services Index](SERVICES_INDEX.md)

**状態: A-**

残課題は、Elastic Beanstalk / App Runner / Batchを含む比較表。

## Task 4.4 Determine opportunities for modernization and enhancements

**公式論点**

- Decoupling
- Serverless
- Containers
- Purpose-built databases
- Application integration

**対応資料**

- [Migration and Modernization Reader](MIGRATION_AND_MODERNIZATION_READER.md)
- [Messaging and Eventing Comparison](comparisons/messaging-eventing-comparison.md)
- [AWS SAP設計読本](SAP_DESIGN_READER.md#8-疎結合とイベント駆動)

**状態: A-**

残課題は、段階的モダナイゼーションとStrangler Figの総合演習。

---

# 次のバックログ

## P0: 得点と設計力に直結

1. 65問模試を公式Domain比率へ合わせる
2. Performance / Cost / Migrationの計算・判断演習
3. Multi-account operating model
4. CodePipeline / CodeBuild / CodeDeploy / CDK / SAM

## P1: 比較の穴を埋める

1. Purpose-built database comparison
2. Streaming comparison
3. Compute platform comparison
4. AWS RAM / Directory Service / Service Quotas

## P2: 参照用

1. Hybrid / Edge個別ページ
2. MLサービス選択マップ
3. IoTサービス選択マップ

# 更新ルール

新しいページを追加したときは、必ず次を更新する。

1. このマトリクスの対応資料と状態
2. [README](README.md) の読書導線
3. [LEARNING_PATH](LEARNING_PATH.md) の学習順序
4. 必要に応じて [SERVICES_INDEX](SERVICES_INDEX.md)

ページ数ではなく、**公式タスクを説明・比較・演習できるか**で完成度を判断する。
