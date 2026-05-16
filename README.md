# AWS Mastering Repository

**AWS SAP-C02に必要なサービス選定力を体系的・実践的に鍛えるマスタリングノートブック**

SAP-C02合格 + 実務アーキテクト力向上を目指す人のためのガイド。

## 🎯 戦略 (Tiered Approach)

- **Tier 1 (最重要サービス)**: 詳細解説、豊富なユースケース、対比、Well-Architected紐付け
- **Tier 2**: 標準テンプレート (概要、ユースケース 3-5個、接続)
- **Tier 3**: コンパクト参照用

## 📁 フォルダ構成

- `services/` - 各AWSサービス詳細
- `comparisons/` - サービス対比表
- `patterns/` - 実践アーキテクチャパターン
- `architecture-diagrams/` - SAP-C02頻出構成図
- `sap-c02/` - 試験ドメイン別の論点整理
- `glossary/` - ネットワーク/Web/認証/DRなど周辺用語
- `well-architected/` - Well-Architected柱別ノート
- `practice/` - SAP-C02長文シナリオ型問題集
- `LEARNING_PATH.md` - 体系的学習ロードマップ
- `SERVICES_INDEX.md` - 全サービス一覧 + Tier分類

## 🚀 使い方

1. **体系的に学ぶ**: `LEARNING_PATH.md` を順番に読む
2. **辞書的に調べる**: `SERVICES_INDEX.md` からサービスページへ移動
3. **設計力を鍛える**: `architecture-diagrams/`, `patterns/`, `comparisons/` を使い、サービス選定理由を言語化する
4. **試験対策する**: `sap-c02/` でドメイン別に頻出論点を確認する
5. **ネットワークで詰まったら**: `comparisons/networking-foundations-deep-dive.md` で経路/SG/NACL/Gateway/Endpointを確認する
6. **権限・暗号化で混乱したら**: `comparisons/access-control-and-encryption.md` でIAM/KMS/Lake Formation/QuickSight/TLSを整理する
7. **DB接続で詰まったら**: `comparisons/rds-aurora-connection-deep-dive.md` でendpoint、read offload、RDS Proxy、分析分離を確認する
8. **AWS外の前提語で詰まったら**: `glossary/web-runtime.md` と `glossary/pool-terms.md` に戻る

## 📊 現在の進捗

- 基本構造完成
- IAM / KMS / Cognito / Secrets Manager / ACM / WAF / Shield / Network Firewall などセキュリティ重要論点を追加
- Direct Connect / Site-to-Site VPN / PrivateLink / Global Accelerator / ELB を追加し、ハイブリッド・グローバル接続を強化
- EFS / FSx / Storage Gateway / AWS Backup を追加し、ストレージ選定を強化
- Migration & Transfer フォルダを追加済み
- Messaging/Eventing比較表、Storage比較表、Edge Security比較表を追加
- Management/Governance、Cost Optimization、Analytics/Data Lake比較表を追加
- CloudFormation / CloudTrail / Config / Security Hub を追加し、監査・統制を強化
- Cost Explorer / Budgets / Savings Plans / Reserved Instances / Compute Optimizer を追加し、Cloud Financial Managementを強化
- Athena / Glue / QuickSight / DataZone の個別ページを追加し、データレイク/BI/ガバナンスを強化
- Practiceフォルダを追加し、Domain横断の長文シナリオ問題セットを作成
- Practiceを拡充し、Identity/Security/Networking、Migration/DR/Cost、Data/Integration/Governance/Computeの3セットを追加
- `practice/exam-techniques.md` を追加し、SAP-C02長文シナリオの読解フレームを整理
- 公式スコープ差分表、構成図集、用語集、本番形式模試を追加
- ネットワーク、RDS/Aurora接続、認証認可/暗号化、Web/API、Pool系の「説明の説明」深掘りページを追加

## 次の拡張候補

- Practice: 本番形式問題を65問以上へ拡張し、難易度別・ドメイン別に整理
- Database深掘り: Aurora / DynamoDB / RDS の高難度比較
- Hybrid/Edge: Outposts / Local Zones / Wavelength
- Developer Tools: CodePipeline / CodeDeploy / CDK
- Final Review: 模擬試験形式、弱点診断表、頻出誤答パターン集
