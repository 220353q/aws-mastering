# SAP-C02 Domain 2: Design for New Solutions

## Key Topics
- Serverless architectures (Lambda, API Gateway, Step Functions, DynamoDB, EventBridge)
- Container strategies (ECS, EKS, Fargate, ECR)
- Event-driven and asynchronous integration (EventBridge, SNS, SQS)
- Data & analytics platforms (S3 + Glue + Athena + Redshift + QuickSight + Lake Formation)
- ML/AI integration (SageMaker, Bedrock, Rekognition, Comprehend)
- High availability & disaster recovery (Multi-AZ, global services, Route 53, Global Accelerator)
- Security by design (KMS, Cognito, GuardDuty, WAF, Shield, PrivateLink)

## Common Scenarios
- Design serverless web app with auth and data store
- Build data lake for analytics + ML
- Design global low-latency application
- Modernize monolith to microservices
- Build event-driven order processing with fanout, queues, and workflow orchestration
- Allow mobile/web users to authenticate and access AWS resources with least privilege

## Recommended Services
Lambda + API Gateway + DynamoDB + Cognito + CloudFront + Route 53 + Step Functions + EventBridge + SNS + SQS + S3 + Glue + Athena + SageMaker + Bedrock + GuardDuty + WAF + KMS + PrivateLink

## High-Risk Exam Traps
- User Poolは認証、Identity PoolはAWS一時認証情報。
- SNSはPub/Sub、SQSはキュー、EventBridgeはイベントルーター、Step Functionsは状態管理。
- SSE-KMSの暗号化リソースは、データ権限とKMS権限の両方が必要。
- Gateway EndpointはS3/DynamoDB向け。多くのAWSサービスはInterface Endpointを使う。

---

## Global Delivery and Edge Security Design Notes

- グローバル固定IPと高速リージョン切替はGlobal Accelerator、HTTPキャッシュはCloudFront。
- ALB/NLB/GWLBはレイヤーと用途で選ぶ。パスルーティングはALB、TCP/固定IPはNLB、アプライアンス挿入はGWLB。
- WAFはL7 Web保護、ShieldはDDoS対策、Network FirewallはVPC境界検査。

---

## Data Platform and Standardization Design Notes

- データレイクはS3 + Glue Data Catalog + Athenaを基本形とし、権限制御はLake Formation、BIはQuickSight、データ発見・共有はDataZoneと切り分ける。
- 新規ソリューションでもCloudFormation/StackSetsを使って、環境の再現性と標準化を設計する。

---

## Practice Links

- [Scenario Set 04](../practice/scenario-set-04.md): Event-driven / Data Lake / ECS-EKS-Fargate / API + Cognito
