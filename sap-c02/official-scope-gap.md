# SAP-C02 Official Scope Gap Check

この表は、AWS公式の SAP-C02 Exam Guide と In-scope AWS services を基準に、この教材だけで合格余裕を作るための対応状況を管理するもの。

公式参照:
- AWS Certified Solutions Architect - Professional (SAP-C02): https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-professional-02/solutions-architect-professional-02.html
- In-scope AWS services and features: https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-professional-02/sap-02-in-scope-services.html
- Technologies and Concepts: https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-professional-02/sap-technologies-concepts.html

## Exam Facts

| 項目 | 内容 | 教材での扱い |
|---|---|---|
| 採点対象 | 65問 | `practice/` を65問以上へ拡張する |
| 未採点問題 | 10問 | 本番では見分けられない前提で読む |
| 合格ライン | 750 / 1000 | 弱点診断でDomain別に穴を潰す |
| Domain 1 | Design Solutions for Organizational Complexity: 26% | `sap-c02/domain1.md` + multi-account/security問題 |
| Domain 2 | Design for New Solutions: 29% | `sap-c02/domain2.md` + new architecture問題 |
| Domain 3 | Continuous Improvement for Existing Solutions: 25% | `sap-c02/domain3.md` + cost/ops/compliance問題 |
| Domain 4 | Accelerate Workload Migration and Modernization: 20% | `sap-c02/domain4.md` + migration/modernization問題 |

## Coverage Legend

| 状態 | 意味 |
|---|---|
| Strong | サービスページ、比較、練習問題のいずれも揃っている |
| Good | サービスページはあるが、比較や練習問題を増やしたい |
| Basic | 関連ページ内で触れているが、単独ページが薄い/ない |
| Gap | 試験で選択肢に出たとき切り分けに不安が残る |

## High-Priority Coverage

| 分類 | サービス/概念 | 状態 | 追加する教材 |
|---|---|---|---|
| Storage | Amazon EBS | Basic | `services/storage/ebs.md` |
| Storage | Amazon S3 Glacier | Basic | `services/storage/s3-glacier.md` |
| Storage / DR | AWS Elastic Disaster Recovery | Basic | `services/migration/elastic-disaster-recovery.md` |
| Security | AWS IAM Identity Center | Good | `services/security/iam-identity-center.md` |
| Security | Amazon Inspector | Basic | `services/security/inspector.md` |
| Security | Amazon Macie | Basic | `services/security/macie.md` |
| Security | AWS Firewall Manager | Basic | `services/security/firewall-manager.md` |
| Security | AWS RAM / AWS STS | Basic | `services/security/ram.md`, `services/security/sts.md` |
| Management | AWS Trusted Advisor | Basic | `services/management/trusted-advisor.md` |
| Management | AWS Resilience Hub | Basic | `services/management/resilience-hub.md` |
| Management | AWS Service Catalog | Basic | `services/management/service-catalog.md` |
| Application Integration | AWS AppSync | Basic | `services/integration/appsync.md` |
| Analytics | Amazon EMR | Basic | `services/analytics/emr.md` |
| Analytics | Amazon MSK / Managed Service for Apache Flink | Basic | `services/analytics/msk.md`, `services/analytics/flink.md` |

## Category Coverage

| 公式カテゴリ | 教材の状態 | コメント |
|---|---|---|
| Analytics | Good | Athena/Glue/Lake Formation/Redshift/Kinesis/OpenSearchはある。EMR/MSK/FlinkはBasic、Data Exchangeは未対応。 |
| Application Integration | Good | EventBridge/SNS/SQS/Step Functions/MQは強い。AppSyncはBasic、AppFlowは未対応。 |
| Cloud Financial Management | Good | Cost Explorer/Budgets/Savings Plans/RI/Compute Optimizerはある。CURはCost Explorer内に留まる。 |
| Compute | Basic | EC2/Lambda/Fargateはあるが、EBS/Auto Scaling/Batch/App Runner/Beanstalkの切り分けが不足。 |
| Containers | Good | ECS/EKS/Fargateはある。ECR/EKS Anywhere/ECS Anywhereは補足程度でよい。 |
| Database | Good | RDS/Aurora/DynamoDB/ElastiCache/DocumentDB/Neptuneはある。Keyspaces/Timestreamは低優先で追加候補。 |
| Developer Tools | Gap | CodePipeline/CodeBuild/CodeDeploy/CDK/SAM/Protonが未整理。Domain 3対策で追加したい。 |
| Management and Governance | Good | CloudTrail/Config/CloudWatch/Control Tower/SSM/Security Hubは強い。Trusted Advisor/Service Catalog/Resilience HubはBasic。 |
| Migration and Transfer | Strong | MGN/DMS/SCT/DataSync/Snow/Transfer Family/Discovery/Migration Hubが揃う。Elastic Disaster Recoveryを明確化する。 |
| Networking and Content Delivery | Strong | VPC/TGW/DX/VPN/PrivateLink/Route 53/CloudFront/GA/ELBが揃う。Route 53 ResolverとVPC Flow Logsを補足したい。 |
| Security, Identity, and Compliance | Good | IAM/KMS/Cognito/Secrets/WAF/Shield/Network Firewall/GuardDuty/Security Hubは強い。Inspector/Macie/Firewall Manager/RAM/STSはBasic。 |
| Storage | Good | S3/EBS/EFS/FSx/Storage Gateway/Backup/S3 Glacier/Elastic Disaster Recoveryを押さえた。EBS/Glacier/DRSは練習問題を追加したい。 |

## Technologies and Concepts Coverage

| 公式概念 | 教材で強化する場所 |
|---|---|
| Compute | `services/compute/`, `comparisons/ec2-vs-lambda-fargate.md` |
| Cost management | `services/cost/`, `comparisons/cost-optimization.md` |
| Database | `services/database/`, `comparisons/rds-vs-dynamodb.md`, `comparisons/dynamodb-vs-aurora.md` |
| Disaster recovery | `patterns/disaster-recovery.md`, `services/management/resilience-hub.md` |
| High availability | `patterns/multi-region.md`, `architecture-diagrams/README.md` |
| Management and governance | `services/management/`, `sap-c02/domain1.md`, `sap-c02/domain3.md` |
| Microservices and decoupling | `patterns/microservices.md`, `comparisons/messaging-eventing.md` |
| Migration and data transfer | `services/migration/`, `sap-c02/domain4.md` |
| Networking, connectivity, and content delivery | `services/networking/`, `glossary/network-web.md` |
| Security | `services/security/`, `comparisons/edge-security.md` |
| Serverless design principles | `patterns/serverless-web-app.md`, `services/compute/lambda.md` |
| Storage | `services/storage/`, `comparisons/storage-options.md` |

## Maintenance Rule

- 新しいサービスページを作ったら、この表の `Gap` を `Basic` 以上に更新する。
- 練習問題でそのサービスが正解/誤答の両方に出たら `Good` にする。
- 比較表、構成図、練習問題が揃ったら `Strong` にする。
