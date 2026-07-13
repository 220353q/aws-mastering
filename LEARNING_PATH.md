# AWS Mastering Learning Path

## SAP-C02向け推奨学習順序

このリポジトリは、サービスページを順番に暗記するよりも、最初に設計判断の全体像をつかみ、必要な資料へ戻る使い方を推奨する。

## Phase 0: 通読して設計の型を作る

最初に [AWS SAP設計読本](SAP_DESIGN_READER.md) を読む。

読了時の目標は、次の一文を自分で作れることである。

> この要件ではAを選ぶ。Bでも実現可能だが、運用負荷・RTO・コストの制約に合わないため採用しない。

読書中に理解が浅い項目があっても、すぐ全サービスを暗記しない。リンク先を一度確認し、本編へ戻って読み進める。

### 通読時に使うメモ形式

```text
目的:
前提:
制約:
候補:
採用:
不採用理由:
```

## Phase 1: 基盤固め（Tier 1重点）

1. **IAM、Organizations、KMS、Well-Architected Framework** - セキュリティ・ガバナンスの基盤
2. **VPC、Route 53、CloudFront、Direct Connect、PrivateLink、Transit Gateway** - ネットワーク・CDN・ハイブリッド接続
3. **EC2、Auto Scaling、ELB、EBS** - コンピュート、ブロックストレージ、ロードバランシング
4. **S3、EFS、FSx、Storage Gateway、AWS Backup** - オブジェクト、ファイル、ハイブリッドストレージ、バックアップ
5. **RDS、Aurora、DynamoDB、ElastiCache** - データモデル、可用性、読み取り拡張、キャッシュ

各項目で、概要だけでなく「何と迷うか」「どの制約語で選択が変わるか」を記録する。

## Phase 2: サーバーレス、コンテナ、疎結合

6. **Lambda、API Gateway、Step Functions** - サーバーレスとワークフロー
7. **ECS、EKS、Fargate** - コンテナと運用責任の違い
8. **EventBridge、SQS、SNS** - イベント配送、バッファ、Pub/Sub
9. **Cognito** - アプリケーション認証とAWS一時認証情報

重点課題：

- EventBridgeからECSタスクを定刻起動するとき、API Gatewayが不要な理由を説明する
- SQSとEventBridgeを「どちらが上か」ではなく、バッファとルーティングの役割で分ける
- ECSとEKSを、Kubernetes要件と運用負荷から比較する

## Phase 3: 移行、分析、AI、管理

10. **MGN、DMS、Schema Conversion、DataSync、Migration Hub、Snow Family** - 移行戦略と停止許容時間
11. **Athena、Glue、Redshift、QuickSight、Lake Formation、DataZone** - 分析、BI、データガバナンス
12. **SageMaker、Bedrock、Rekognition** - ML/AI
13. **CloudWatch、CloudTrail、Config、CloudFormation、Systems Manager、Security Hub** - 管理、監視、監査、IaC、所見集約

## Phase 4: 実践デザインと試験対策

14. **Well-Architectedレビュー** と `patterns/` を読み、構成ごとの弱点を探す
15. **comparisons/** で最適サービス選択力を確立する
    - ネットワーク: [Networking Foundations Deep Dive](comparisons/networking-foundations-deep-dive.md)
    - RDS/Aurora接続: [RDS / Aurora Connection Deep Dive](comparisons/rds-aurora-connection-deep-dive.md)
    - 認証、認可、暗号化: [Access Control and Encryption Deep Dive](comparisons/access-control-and-encryption.md)
    - 権限制御境界: [IAM Boundaries / SCP / Condition Deep Dive](comparisons/iam-boundaries-scp-condition-deep-dive.md)
    - Web/API前提語: [Web Runtime and Proxy Terms](glossary/web-runtime.md)
    - Pool系用語: [Pool Terms](glossary/pool-terms.md)
16. **sap-c02/** でドメイン別に弱点を洗い出す
17. **practice/exam-techniques.md** で長文問題の制約語・誤答パターンを確認する
18. **practice/scenario-set-01〜04** を解き、各設問の誤答理由をサービスページへ戻って復習する
19. Domain 1〜4ごとに「自分が落ちた誤答パターン」を一覧化する

## Phase 5: 説明できるかを確認する

次のテーマを、サービス名の列挙ではなく設計判断として説明する。

- Shared VPCとTransit Gatewayの使い分け
- VPC PeeringとPrivateLinkの使い分け
- ALB、NLB、GWLBの使い分け
- CloudFront、Route 53、Global Acceleratorの使い分け
- ECS、EKS、Lambdaの使い分け
- RDS Multi-AZ、Read Replica、Aurora Global Databaseの目的の違い
- SQS、SNS、EventBridge、Step Functionsの役割の違い
- Backup & Restore、Pilot Light、Warm Standby、Active/Activeの選択
- MGN、DMS、DataSync、Snow Familyの選択
- SCP、Permission Boundary、Resource-based Policy、Session Policyの境界

## 学習完了の判定

次の状態になれば、単体暗記からSAP型の学習へ移行できている。

- 問題文から強い制約語を抜き出せる
- 通信経路とデータフローを簡単な図にできる
- 正解の理由だけでなく、誤答が失格になる理由を説明できる
- RTO/RPO、運用負荷、移行停止時間、コストを比較軸にできる
- 不明点が出たとき、どのサービスページや比較表へ戻るべきか分かる

**Tips**: Tier 1を完了後にTier 2へ進む。サービス単体暗記ではなく、「なぜ他サービスではないのか」を必ず比較する。
