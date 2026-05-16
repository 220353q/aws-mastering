# SAP-C02 Architecture Diagrams

SAP-C02では、文章中の制約を構成図に変換してから選択肢を切る力が重要。このページは、頻出アーキテクチャを「どのサービスがどの責務か」で確認するための図集。

## 1. Multi-Account Landing Zone

```text
AWS Organizations
  ├─ Security OU
  │    ├─ Log Archive Account
  │    │    └─ S3 + CloudTrail Organization Trail
  │    └─ Security Tooling Account
  │         ├─ GuardDuty delegated admin
  │         ├─ Security Hub delegated admin
  │         └─ EventBridge → SSM Automation / Lambda
  ├─ Infrastructure OU
  │    └─ Network Account
  │         ├─ Transit Gateway
  │         ├─ Shared VPC / RAM
  │         └─ Route 53 Resolver
  └─ Workloads OU
       ├─ Prod Account
       └─ Dev Account

Control Tower: landing zone / guardrails
SCP: maximum permission boundary for accounts
IAM Identity Center: workforce SSO
CloudFormation StackSets: baseline deployment
```

確認ポイント:
- SCPは権限を付与しない。最大権限を制限する。
- CloudTrailはAPI監査、Configは設定準拠、Security Hubは所見集約。
- 新規アカウントにも標準設定を配るならControl Tower/StackSets。

## 2. Secure Public Web Application

```text
Users
  │ HTTPS
Route 53
  │
CloudFront + ACM + WAF + Shield
  ├─ /static/* → S3 private bucket + OAC
  └─ /api/*    → ALB → ECS/EC2/Lambda
                        │
                        ├─ Secrets Manager / KMS
                        └─ RDS/Aurora/DynamoDB

CloudTrail / CloudWatch / Config / Security Hub
```

確認ポイント:
- HTTPキャッシュとS3直接アクセス禁止はCloudFront + OAC。
- SQLi/XSS/Bot/Rate limitはWAF、DDoS支援はShield Advanced。
- Global Acceleratorはキャッシュしない。

## 3. Hybrid Network Hub

```text
On-Premises
  ├─ Direct Connect
  └─ Site-to-Site VPN backup
        │
Direct Connect Gateway / Transit Gateway
        │
Inspection VPC
  ├─ AWS Network Firewall
  └─ GWLB → third-party IDS/IPS
        │
Transit Gateway route tables
  ├─ Shared Services VPC
  ├─ Prod VPC
  └─ Dev VPC

Route 53 Resolver: hybrid DNS
PrivateLink: service-level private exposure
```

確認ポイント:
- 多数VPC/オンプレ集約はTransit Gateway。
- 専用線はDirect Connect。暗号化が必要ならVPN over DXやMACsecを検討。
- PrivateLinkはVPC全体接続ではなくサービス単位。

## 4. Data Lake Governance

```text
Producer Accounts
  └─ Logs / Events / Data
        │
        ▼
Central Data Lake Account
  ├─ S3 raw / curated buckets + SSE-KMS
  ├─ Glue Data Catalog + Crawlers
  ├─ Lake Formation permissions
  ├─ Athena queries
  ├─ Redshift / Spectrum
  ├─ QuickSight dashboards
  └─ DataZone discovery / subscription / governance

Security:
  IAM + S3 bucket policy + KMS key policy + Lake Formation grants
```

確認ポイント:
- S3/IAM/KMSに広すぎる直接権限があるとLake Formationを迂回しうる。
- Glueはカタログ/ETL、AthenaはSQL、QuickSightはBI、DataZoneは発見/共有。
- SSE-KMSのクロスアカウントはKMS key policyも必要。

## 5. Disaster Recovery Patterns

```text
Backup & Restore
Primary Region ── Snapshot / AWS Backup ──> DR Region restore on failure

Pilot Light
Primary App + DB ── replication ──> DR data layer running, app layer stopped/minimal

Warm Standby
Primary full stack ── replication ──> DR scaled-down full stack

Multi-Site Active/Active
Route 53 / Global Accelerator
  ├─ Region A active
  └─ Region B active
```

確認ポイント:
- RTO/RPOが緩くコスト最優先ならBackup & Restore。
- RTO/RPOが短くなるほどPilot Light、Warm Standby、Active/Activeへ進む。
- Aurora Global Databaseは基本single-writer。multi-region writesならDynamoDB Global Tablesを検討。

## 6. Migration and Modernization

```text
Discover
  Application Discovery Service → Migration Hub

Move
  Servers      → MGN / Elastic Disaster Recovery
  Databases    → DMS + SCT / DMS Schema Conversion
  Files        → DataSync
  Offline data → Snow Family
  SFTP/FTPS    → Transfer Family

Modernize
  Strangler Fig → API Gateway / ALB → new services
  Containers    → ECS / EKS / Fargate
  Serverless    → Lambda / Step Functions / EventBridge
```

確認ポイント:
- サーバーのリホストはMGN、DR待機はElastic Disaster Recovery。
- 異種DBはSCT/Schema Conversion + DMS。
- ファイル移行はDataSync、回線不足の大容量はSnow Family。

## 7. Event-Driven Order Processing

```text
API Gateway / ALB
  │
Order Service
  │ emits event
  ▼
EventBridge Bus
  ├─ Rule → Step Functions workflow
  │          ├─ Payment Lambda
  │          ├─ Inventory Lambda
  │          └─ Compensation path
  ├─ Rule → SNS fanout
  │          ├─ Email
  │          └─ SQS per subscriber
  └─ Rule → SQS buffer → Worker
```

確認ポイント:
- イベントルーティングはEventBridge。
- 複数購読者への通知はSNS。
- 処理のバッファ/平準化はSQS。
- 状態、分岐、補償処理はStep Functions。

## 8. Continuous Compliance and Remediation

```text
All Accounts
  ├─ CloudTrail organization trail → Log Archive S3
  ├─ AWS Config rules / conformance packs
  ├─ GuardDuty findings
  └─ Inspector / Macie findings
        │
        ▼
Security Hub delegated admin
        │
        ▼
EventBridge rule
        │
        ├─ Systems Manager Automation
        ├─ Lambda remediation
        └─ SNS / Incident workflow

CloudFormation StackSets: deploy baseline rules
```

確認ポイント:
- 検出、集約、ルーティング、修復を分けて考える。
- CloudTrailは操作証跡、Configは設定準拠、Security Hubは所見集約。
- 自動修復はEventBridge + SSM Automation/Lambda。
