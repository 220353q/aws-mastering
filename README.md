# AWS Mastering Repository

**AWS SAP-C02に必要なサービス選定力を体系的・実践的に鍛えるマスタリングノートブック**

SAP-C02合格と、実務で「なぜその設計なのか」を説明できるアーキテクト力の獲得を目指す。

## 📖 最初に読む

AWSの説明文に出てくる `endpoint`、`proxy`、`replication`、`stateful`、`managed`、`failover` などの前提語が曖昧な場合は、最初に **[AWS「説明の説明」](EXPLANATION_OF_EXPLANATIONS.md)** を読む。

その後、**[AWS SAP設計読本](SAP_DESIGN_READER.md)** を最初から読み、**[SAP-C02カバレッジマトリクス](SAP_C02_COVERAGE_MATRIX.md)** で公式20タスクとリポジトリの対応状況を確認する。

### 主要読本の役割

| 読本 | 身につけること |
|---|---|
| [AWS「説明の説明」](EXPLANATION_OF_EXPLANATIONS.md) | AWS解説文を「誰が・何を・どこへ・なぜ・どの条件で」へ分解する力 |
| [AWS SAP設計読本](SAP_DESIGN_READER.md) | 要件、制約、候補、比較、不採用理由から設計を選ぶ力 |
| [SAP-C02カバレッジマトリクス](SAP_C02_COVERAGE_MATRIX.md) | 公式4ドメイン・20タスクの整備状況と次の弱点 |
| [Performance Design Reader](PERFORMANCE_DESIGN_READER.md) | 症状、Metric、Bottleneck、改善、検証をつなぐ力 |
| [Continuous Improvement Playbook](CONTINUOUS_IMPROVEMENT_PLAYBOOK.md) | 既存環境を観測し、改善を反復する力 |
| [Migration and Modernization Reader](MIGRATION_AND_MODERNIZATION_READER.md) | Portfolio評価、7R、Wave、Cutover、Modernizationを設計する力 |

## 🎯 戦略（Tiered Approach）

- **Tier 1（最重要サービス）**: 詳細解説、豊富なユースケース、対比、Well-Architected紐付け
- **Tier 2**: 標準テンプレート（概要、ユースケース、接続、判断基準）
- **Tier 3**: コンパクトな参照用ページ

サービス数ではなく、公式Taskごとに**説明・比較・演習が揃っているか**で完成度を判断する。

## 📁 フォルダ構成

- `EXPLANATION_OF_EXPLANATIONS.md` - AWSの説明文を読み解くための基礎読本
- `SAP_DESIGN_READER.md` - SAP設計判断を通読で学ぶ本編
- `SAP_C02_COVERAGE_MATRIX.md` - 公式20タスクと教材の対応表
- `PERFORMANCE_DESIGN_READER.md` - 性能設計とBottleneck分析
- `CONTINUOUS_IMPROVEMENT_PLAYBOOK.md` - 既存環境の改善サイクル
- `MIGRATION_AND_MODERNIZATION_READER.md` - 移行ProgramとModernization
- `services/` - 各AWSサービス詳細
- `comparisons/` - サービス対比表・混同しやすい概念の深掘り
  - `deployment-and-rollback-strategies.md`
  - `hybrid-dns-deep-dive.md`
  - `cost-modeling-and-data-transfer.md`
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

1. **説明を分解する**: [AWS「説明の説明」](EXPLANATION_OF_EXPLANATIONS.md)
2. **設計を通読する**: [AWS SAP設計読本](SAP_DESIGN_READER.md)
3. **公式範囲と照合する**: [SAP-C02カバレッジマトリクス](SAP_C02_COVERAGE_MATRIX.md)
4. **横断テーマを深める**:
   - [Deployment and Rollback](comparisons/deployment-and-rollback-strategies.md)
   - [Hybrid DNS](comparisons/hybrid-dns-deep-dive.md)
   - [Performance Design](PERFORMANCE_DESIGN_READER.md)
   - [Cost Modeling and Data Transfer](comparisons/cost-modeling-and-data-transfer.md)
   - [Continuous Improvement](CONTINUOUS_IMPROVEMENT_PLAYBOOK.md)
   - [Migration and Modernization](MIGRATION_AND_MODERNIZATION_READER.md)
5. **体系的に深める**: [LEARNING_PATH.md](LEARNING_PATH.md)
6. **辞書的に調べる**: [SERVICES_INDEX.md](SERVICES_INDEX.md)
7. **設計力を鍛える**: `architecture-diagrams/`、`patterns/`、`comparisons/`
8. **試験対策する**: `sap-c02/` と `practice/`

## 🧭 説明を読むフレーム

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

```text
目的 → 前提 → 制約 → 候補 → 比較 → 決定 → 不採用理由
```

特に「最小コスト」「最小の運用負荷」「停止時間を最小化」「データ損失不可」などの強い制約語を先に見つける。サービス名を見つけて飛びつくのではなく、制約を満たさない選択肢から落とす。

## 🧭 既存環境を改善するフレーム

```text
Observe
  → Define
  → Diagnose
  → Prioritize
  → Change
  → Verify
  → Standardize
```

新しいServiceを追加したことではなく、SLO、Business KPI、Cost、復旧時間が改善したことを成果とする。

## 📊 現在の進捗

- AWS解説文の前提・動詞・因果関係を展開する `EXPLANATION_OF_EXPLANATIONS.md`
- SAP-C02向けの通読本編 `SAP_DESIGN_READER.md`
- 公式4ドメイン・20タスクを対応付ける `SAP_C02_COVERAGE_MATRIX.md`
- Deployment / Rollback、Hybrid DNS、Cost / Data Transferの横断比較
- Performance、Continuous Improvement、Migration / Modernizationの専門読本
- IAM / KMS / Cognito / Secrets Manager / ACM / WAF / Shield / Network FirewallなどのSecurity重要論点
- Direct Connect / VPN / PrivateLink / Global Accelerator / ELBなどHybrid・Global接続
- EFS / FSx / Storage Gateway / AWS BackupなどStorage選定
- Migration & Transferサービスページ
- Messaging/Eventing、Storage、Edge Security、Governance、Cost、Analytics比較
- CloudFormation / CloudTrail / Config / Security Hubによる監査・統制
- Cost Explorer / Budgets / Savings Plans / Reserved Instances / Compute Optimizer
- Athena / Glue / QuickSight / DataZone
- Domain横断の長文Scenarioと読解フレーム

## 次の拡張候補

- Practice: 65問以上の本番形式模試を公式Domain比率へ整合
- Multi-account operating model: AWS RAM、Delegated Administrator、組織通知
- Developer Tools: CodePipeline / CodeBuild / CodeDeploy / CDK / SAMの個別ページ
- Calculation exercises: Kinesis shard、DynamoDB partition、EBS帯域、Data transfer
- Purpose-built database比較
- Hybrid/Edge: Outposts / Local Zones / Wavelength
- Final Review: 弱点診断表、頻出誤答パターン集
