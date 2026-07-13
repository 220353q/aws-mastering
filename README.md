# AWS Mastering Repository

**AWS SAP-C02に必要なサービス選定力を体系的・実践的に鍛えるマスタリングノートブック**

SAP-C02合格と、実務で「なぜその設計なのか」を説明できるアーキテクト力の獲得を目指す。

## 📖 最初に読む

AWSの説明文に出てくる `endpoint`、`proxy`、`replication`、`stateful`、`managed`、`failover` などの前提語が曖昧な場合は、最初に **[AWS「説明の説明」](EXPLANATION_OF_EXPLANATIONS.md)** を読む。

その後、**[AWS SAP設計読本](SAP_DESIGN_READER.md)** を最初から読む。

### 二つの読本の役割

| 読本 | 身につけること |
|---|---|
| [AWS「説明の説明」](EXPLANATION_OF_EXPLANATIONS.md) | AWS解説文を「誰が・何を・どこへ・なぜ・どの条件で」へ分解する力 |
| [AWS SAP設計読本](SAP_DESIGN_READER.md) | 要件、制約、候補、比較、不採用理由から設計を選ぶ力 |

設計読本は、個別サービスの暗記ではなく、次の流れを一冊にまとめている。

1. 問題文から目的・前提・制約を抜き出す
2. 複数の候補を比較する
3. 可用性、性能、セキュリティ、運用、コストのトレードオフを整理する
4. 採用理由と不採用理由を言語化する
5. 詳細が必要な箇所だけ、本リポジトリ内のサービス辞書・比較表・演習へ戻る

## 🎯 戦略（Tiered Approach）

- **Tier 1（最重要サービス）**: 詳細解説、豊富なユースケース、対比、Well-Architected紐付け
- **Tier 2**: 標準テンプレート（概要、ユースケース、接続、判断基準）
- **Tier 3**: コンパクトな参照用ページ

## 📁 フォルダ構成

- `EXPLANATION_OF_EXPLANATIONS.md` - AWSの説明文を読み解くための基礎読本
- `SAP_DESIGN_READER.md` - SAP設計判断を通読で学ぶ本編
- `services/` - 各AWSサービス詳細
- `comparisons/` - サービス対比表・混同しやすい概念の深掘り
- `patterns/` - 実践アーキテクチャパターン
- `architecture-diagrams/` - SAP-C02頻出構成図
- `architecture/` - 横断的なアーキテクチャ解説
- `sap-c02/` - 試験ドメイン別の論点整理
- `glossary/` - ネットワーク、Web、認証、DRなど周辺用語
- `well-architected/` - Well-Architected柱別ノート
- `practice/` - SAP-C02長文シナリオ型問題集
- `LEARNING_PATH.md` - 体系的学習ロードマップ
- `SERVICES_INDEX.md` - 全サービス一覧とTier分類

## 🚀 使い方

1. **説明を分解する**: [AWS「説明の説明」](EXPLANATION_OF_EXPLANATIONS.md) で、用語・通信・状態・責任・因果関係の読み方を覚える
2. **設計を通読する**: [AWS SAP設計読本](SAP_DESIGN_READER.md) で、要件から設計を導く考え方をつかむ
3. **体系的に深める**: `LEARNING_PATH.md` を順番に読む
4. **辞書的に調べる**: `SERVICES_INDEX.md` からサービスページへ移動する
5. **設計力を鍛える**: `architecture-diagrams/`、`patterns/`、`comparisons/` を使い、採用理由と不採用理由を言語化する
6. **試験対策する**: `sap-c02/` でドメイン別に頻出論点を確認する
7. **ネットワークで詰まったら**: `comparisons/networking-foundations-deep-dive.md` で経路、SG、NACL、Gateway、Endpointを確認する
8. **権限・暗号化で混乱したら**: `comparisons/access-control-and-encryption.md` でIAM、KMS、Lake Formation、QuickSight、TLSを整理する
9. **権限制御の境界で混乱したら**: `comparisons/iam-boundaries-scp-condition-deep-dive.md` でPermission Boundary、SCP、Session Policy、Resource-based Policy、Conditionを整理する
10. **DB接続で詰まったら**: `comparisons/rds-aurora-connection-deep-dive.md` でendpoint、read offload、RDS Proxy、分析分離を確認する
11. **AWS外の前提語で詰まったら**: `glossary/web-runtime.md` と `glossary/pool-terms.md` に戻る

## 🧭 説明を読むフレーム

個別サービスの説明は、次へ展開する。

```text
誰が
  → 何を
  → どこへ
  → いつ
  → どうやって
  → なぜ
  → どんな代償で
```

さらに、通信経路、データの正本、状態の保持者、障害時の動作、運用責任、比較対象を確認する。

## 🧭 SAPで使う判断フレーム

各問題・設計は次の順序で整理する。

```text
目的 → 前提 → 制約 → 候補 → 比較 → 決定 → 不採用理由
```

特に「最小コスト」「最小の運用負荷」「停止時間を最小化」「データ損失不可」などの強い制約語を先に見つける。サービス名を見つけて飛びつくのではなく、制約を満たさない選択肢から落とす。

## 📊 現在の進捗

- AWS解説文の前提・動詞・因果関係を展開する `EXPLANATION_OF_EXPLANATIONS.md` を追加
- SAP-C02向けの通読本編 `SAP_DESIGN_READER.md` を追加
- 基本構造完成
- IAM / KMS / Cognito / Secrets Manager / ACM / WAF / Shield / Network Firewall などセキュリティ重要論点を追加
- Direct Connect / Site-to-Site VPN / PrivateLink / Global Accelerator / ELB を追加し、ハイブリッド・グローバル接続を強化
- EFS / FSx / Storage Gateway / AWS Backup を追加し、ストレージ選定を強化
- Migration & Transfer フォルダを追加済み
- Messaging/Eventing、Storage、Edge Security、Management/Governance、Cost Optimization、Analytics/Data Lakeの比較表を追加
- CloudFormation / CloudTrail / Config / Security Hub を追加し、監査・統制を強化
- Cost Explorer / Budgets / Savings Plans / Reserved Instances / Compute Optimizer を追加
- Athena / Glue / QuickSight / DataZone の個別ページを追加
- PracticeフォルダとDomain横断の長文シナリオ問題セットを追加
- `practice/exam-techniques.md` にSAP-C02長文シナリオの読解フレームを整理
- 公式スコープ差分表、構成図集、用語集、本番形式模試を追加
- ネットワーク、RDS/Aurora接続、認証認可/暗号化、Web/API、Pool系の深掘りページを追加

## 次の拡張候補

- Practice: 本番形式問題を65問以上へ拡張し、難易度別・ドメイン別に整理
- Database深掘り: Aurora / DynamoDB / RDS の高難度比較
- Hybrid/Edge: Outposts / Local Zones / Wavelength
- Developer Tools: CodePipeline / CodeDeploy / CDK
- Final Review: 模擬試験形式、弱点診断表、頻出誤答パターン集
