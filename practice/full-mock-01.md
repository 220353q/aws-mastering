# SAP-C02 Full Mock 01

本番形式に近づけるため、各問は複数選択肢から1〜3個を選ぶ。まずは20問で、Domain横断の弱点を診断する。採点後は [weakness-diagnosis.md](weakness-diagnosis.md) に戻り、落とした判断軸をサービスページへ戻って復習する。

## Domain Mapping

| Domain | 問題 |
|---|---|
| Domain 1: Organizational Complexity | Q1, Q2, Q3, Q4, Q5 |
| Domain 2: New Solutions | Q6, Q7, Q8, Q9, Q10, Q11 |
| Domain 3: Continuous Improvement | Q12, Q13, Q14, Q15, Q16 |
| Domain 4: Migration and Modernization | Q17, Q18, Q19, Q20 |

---

## Q1: Organizations標準統制

複数OUに分かれた200アカウントで、CloudTrail停止、KMSキー削除、外部アカウントへの無制限AssumeRoleを禁止したい。各事業部には一定範囲でIAMロール作成を委任したい。最も適切な設計はどれか。

A. SCPで禁止操作をDenyし、委任管理者が作るロールにはPermission Boundaryを必須化する。  
B. SCPでAllowを定義すれば、IAMポリシーなしで権限が付与される。  
C. Permission Boundaryだけでrootユーザーを含む全アカウントの最大権限を制御する。  
D. CloudTrailで監査すれば予防統制は不要。  
E. IAM Identity Centerのグループ名だけでKMS削除を防止する。  

**正解: A**

---

## Q2: クロスアカウントSSE-KMS

中央データレイクアカウントのS3データを、分析アカウントのAthena/QuickSightから参照したい。データはSSE-KMSで暗号化され、列単位の権限制御も必要。重要な確認事項はどれか。

A. S3 bucket policy/IAM policyだけでなくKMS key policyも分析主体を許可する。  
B. Lake Formationでテーブル/列権限を分析主体へ付与する。  
C. KMSで暗号化していればLake Formationは不要。  
D. QuickSightの利用者がMFA済みなら列権限は自動適用される。  
E. S3バケット全体のReadを広く付け、Lake Formationで後から絞る。  

**正解: A, B**

---

## Q3: 共有ネットワーク

ネットワーク専用アカウントがVPCサブネットとTransit Gatewayを管理し、複数ワークロードアカウントへ共有したい。どのサービスを使うか。

A. AWS RAM  
B. AWS STS  
C. AWS Firewall Manager  
D. AWS Service Catalog  
E. Amazon Cognito  

**正解: A**

---

## Q4: 社員SSO

企業IdPと連携し、社員が複数AWSアカウントへSSOし、アカウントごとに標準権限を割り当てたい。最適なサービスはどれか。

A. IAM Identity Center + Permission Set  
B. Cognito User Pool  
C. Cognito Identity Poolのみ  
D. Amazon Macie  
E. AWS WAF  

**正解: A**

---

## Q5: 組織全体のWeb防御標準化

Organizations配下の新規/既存アカウントに対して、CloudFrontやALBへ共通WAFルールを強制適用したい。最も適切なサービスはどれか。

A. AWS Firewall Manager  
B. Amazon Inspector  
C. AWS Configのみ  
D. AWS RAM  
E. AWS DRS  

**正解: A**

---

## Q6: Public Web + S3非公開

世界中のユーザーに静的コンテンツとAPIを同一ドメインで配信し、S3直接アクセスは禁止したい。SQLiと高頻度Botアクセスも緩和したい。適切な組み合わせはどれか。

A. CloudFront + S3 OAC + ALB Origin + AWS WAF  
B. S3 Static Website HostingをPublic Readにする  
C. Global AcceleratorだけでS3をキャッシュする  
D. ACMでTLS証明書を管理する  
E. Shield StandardだけでWAFルールも証明書も不要になる  

**正解: A, D**

---

## Q7: GraphQL API

モバイルアプリが必要なフィールドだけ取得し、DynamoDBとLambdaを単一GraphQL APIで統合したい。リアルタイム更新も必要。最適なサービスはどれか。

A. AWS AppSync  
B. Amazon MQ  
C. AWS Batch  
D. AWS Glue  
E. AWS DataSync  

**正解: A**

---

## Q8: イベント駆動注文処理

注文イベントを複数サービスへルーティングし、支払いと在庫処理は状態管理、失敗時の補償処理が必要。適切な組み合わせはどれか。

A. EventBridgeでイベントルーティングする。  
B. Step Functionsで状態、分岐、補償処理を管理する。  
C. CloudTrailで注文処理の分岐を実装する。  
D. SQSを使えば複雑な補償ワークフローは不要。  
E. SNS/SQSを購読者ごとの非同期処理に使う。  

**正解: A, B, E**

---

## Q9: Kafka互換要件

既存Kafkaクライアント、トピック、コンシューマグループを維持しながらAWSへ移行したい。最適なサービスはどれか。

A. Amazon MSK  
B. EventBridge  
C. Amazon SQS FIFO  
D. AWS AppSync  
E. AWS Transfer Family  

**正解: A**

---

## Q10: ストリーム処理

Kinesis Data Streamsのデータに対し、イベント時間ベースのウィンドウ集計と状態管理を行いたい。最適なサービスはどれか。

A. Managed Service for Apache Flink  
B. Amazon S3 Glacier  
C. AWS Backup  
D. AWS Firewall Manager  
E. IAM Identity Center  

**正解: A**

---

## Q11: ブロック/ファイル/オブジェクト

EC2上のデータベースに低レイテンシのブロックデバイスが必要。複数ホスト共有は不要。最適なストレージはどれか。

A. Amazon EBS  
B. Amazon EFS  
C. Amazon S3  
D. FSx for Windows File Server  
E. Amazon S3 Glacier Deep Archive  

**正解: A**

---

## Q12: Rightsizing

既存EC2/EBS/Lambda/ECS/RDSの過大/過小リソースを推奨に基づいて調整したい。最適なサービスはどれか。

A. AWS Compute Optimizer  
B. Amazon Macie  
C. AWS DMS  
D. AWS AppSync  
E. AWS Snow Family  

**正解: A**

---

## Q13: 既存環境の広い改善候補

コスト、セキュリティ、信頼性、パフォーマンス、サービス制限の観点で既存AWS環境を広くチェックしたい。最適なサービスはどれか。

A. AWS Trusted Advisor  
B. AWS SCT  
C. Amazon EMR  
D. AWS Transfer Family  
E. Amazon Cognito  

**正解: A**

---

## Q14: RTO/RPO評価

既存アプリが定義済みRTO/RPOを満たせるか評価し、改善案を得たい。障害注入テストも別途検討している。最適な評価サービスはどれか。

A. AWS Resilience Hub  
B. AWS FIS  
C. AWS Backupのみ  
D. Amazon Inspector  
E. AWS RAM  

**正解: A**

---

## Q15: 脆弱性/機密データ

EC2とECRイメージのCVEを継続検出し、S3内の個人情報も検出したい。適切な組み合わせはどれか。

A. Amazon Inspector  
B. Amazon Macie  
C. AWS WAFだけ  
D. AWS CloudTrailだけ  
E. AWS Shieldだけ  

**正解: A, B**

---

## Q16: 継続的コンプライアンス

CloudTrail停止、S3公開、EBS未暗号化を検出し、重大所見を集約して自動修復へ流したい。適切な組み合わせはどれか。

A. CloudTrailで操作証跡を取得する。  
B. AWS Config Rules/Conformance Packsで設定準拠を評価する。  
C. Security Hubで所見を集約する。  
D. EventBridge + Systems Manager Automation/Lambdaで修復する。  
E. Cost Explorerだけで修復する。  

**正解: A, B, C, D**

---

## Q17: サーバー移行

オンプレの多数のVMを大きく変更せずEC2へリホストしたい。継続レプリケーションと短いカットオーバーが必要。最適なサービスはどれか。

A. AWS Application Migration Service  
B. AWS DataSync  
C. AWS SCTのみ  
D. Amazon Macie  
E. AWS AppSync  

**正解: A**

---

## Q18: サーバーDR

オンプレサーバーをAWSへ継続レプリケーションし、障害時にEC2として起動したい。移行ではなくDRが主目的。最適なサービスはどれか。

A. AWS Elastic Disaster Recovery  
B. AWS Transfer Family  
C. Amazon EMR  
D. AWS Glue Crawler  
E. AWS RAM  

**正解: A**

---

## Q19: 異種DB移行

OracleからAurora PostgreSQLへ移行したい。スキーマ変換と最小停止時間のデータ移行が必要。適切な組み合わせはどれか。

A. AWS SCT / DMS Schema Conversion  
B. AWS DMS Full Load + CDC  
C. Amazon S3 Glacier  
D. AWS WAF  
E. Amazon QuickSightだけ  

**正解: A, B**

---

## Q20: 大容量データ移行

PB級データをオンプレからAWSへ移したいが、ネットワーク帯域が不足し、オンライン転送では現実的な期間に終わらない。最適なサービスはどれか。

A. AWS Snow Family  
B. AWS DataSyncのみ  
C. Amazon AppSync  
D. AWS Firewall Manager  
E. AWS IAM Identity Center  

**正解: A**

---

## Score Guide

| 正答率 | 目安 |
|---|---|
| 90%+ | 合格余裕ライン。ただし本番65問の持久力演習を継続 |
| 80-89% | 合格圏。誤答分野をDomain別に潰す |
| 70-79% | 知識はあるがサービス区別に穴あり |
| <70% | `LEARNING_PATH.md` と `comparisons/` に戻る |
