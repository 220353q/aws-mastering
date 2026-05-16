# AWS Mastering Learning Path

## SAP-C02向け推奨学習順序

### Phase 1: 基盤固め (Tier 1 重点)
1. **IAM, Organizations, KMS, Well-Architected Framework** - セキュリティ・ガバナンスの基盤
2. **VPC, Route 53, CloudFront, Direct Connect, PrivateLink, Transit Gateway** - ネットワーク・CDN・ハイブリッド接続
3. **EC2, Auto Scaling, ELB, EBS** - コンピュート・ブロックストレージ・ロードバランシング
4. **S3, EFS, FSx, Storage Gateway, AWS Backup** - オブジェクト/ファイル/ハイブリッドストレージ/バックアップ
5. **RDS, Aurora, DynamoDB, ElastiCache** - データベース

### Phase 2: サーバーレス & コンテナ
6. **Lambda, API Gateway, Step Functions** - サーバーレスとワークフロー
7. **ECS, EKS, Fargate** - コンテナ
8. **EventBridge, SQS, SNS** - イベントドリブン・非同期設計
9. **Cognito** - アプリケーション認証・AWS一時認証情報

### Phase 3: 移行・分析・ML・管理
10. **MGN, DMS, SCT, DataSync, Migration Hub, Snow Family** - 移行・モダナイゼーション
11. **Athena, Glue, Redshift, QuickSight, Lake Formation, DataZone** - 分析・BI・データガバナンス
12. **SageMaker, Bedrock, Rekognition** - ML/AI
13. **CloudWatch, CloudTrail, Config, CloudFormation, Systems Manager, Security Hub** - 管理・監視・監査・IaC・所見集約

### Phase 4: 実践デザイン & 対策
14. **Well-Architected レビュー** + `patterns/`
15. **comparisons/** で最適サービス選択力を確立（networking / messaging / storage / edge-security / governance / cost / analytics）
    - ネットワークで混乱したら [Networking Foundations Deep Dive](comparisons/networking-foundations-deep-dive.md)
    - RDS/Aurora接続で混乱したら [RDS / Aurora Connection Deep Dive](comparisons/rds-aurora-connection-deep-dive.md)
    - 認証/認可/暗号化で混乱したら [Access Control and Encryption Deep Dive](comparisons/access-control-and-encryption.md)
    - Web/API前提語で混乱したら [Web Runtime and Proxy Terms](glossary/web-runtime.md) と [Pool Terms](glossary/pool-terms.md)
16. **sap-c02/** でドメイン別に弱点を洗い出す
17. **practice/exam-techniques.md** で、長文問題の制約語・誤答パターンを確認する
18. **practice/scenario-set-01〜04** を解き、各設問の誤答理由をサービスページへ戻って復習する
19. 仕上げでは、Domain 1〜4ごとに「自分が落ちた誤答パターン」を一覧化する

**Tips**: Tier 1を完了後にTier 2へ。サービス単体暗記ではなく、「なぜ他サービスではないのか」を必ず比較する。
