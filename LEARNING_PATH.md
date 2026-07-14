# AWS Mastering Learning Path

## SAP-C02向け推奨学習順序

このリポジトリは、サービスページを順番に暗記するよりも、説明文の読み方、設計判断、公式Taskとの対応、横断テーマ、個別サービス、演習の順で進む使い方を推奨する。

```text
説明を読む
  → 設計を選ぶ
  → 公式Taskと照合する
  → 横断テーマを深める
  → 個別Serviceへ戻る
  → 問題を解く
  → 弱点を更新する
```

---

# Phase -1: AWSの説明を分解できるようにする

最初に [AWS「説明の説明」](EXPLANATION_OF_EXPLANATIONS.md) を読む。

AWSの説明文には、`route`、`proxy`、`endpoint`、`replication`、`cache`、`stateful`、`managed`、`failover` など、一般IT用語が圧縮されている。次の形へ展開できる状態を作る。

```text
誰が
  → 何を
  → どこへ
  → いつ
  → どうやって
  → なぜ
  → どんな代償で
```

## Phase -1の完了条件

- 「前段」「後段」が何を基準にした位置関係か説明できる
- client/serverとproducer/consumerを分けられる
- route、forward、proxy、replicate、cache、bufferの違いを説明できる
- public/private、sync/async、stateless/statefulを二択暗記せず説明できる
- availability、replica、backup、DRを分けられる
- latency、throughput、IOPS、concurrencyを分けられる
- authentication、authorization、credential、roleを分けられる
- retryが起きる構成でidempotencyが必要な理由を説明できる

---

# Phase 0: 通読して設計の型を作る

次に [AWS SAP設計読本](SAP_DESIGN_READER.md) を読む。

読了時の目標:

> この要件ではAを選ぶ。Bでも実現可能だが、運用負荷・RTO・コストの制約に合わないため採用しない。

## 通読メモ

```text
目的:
前提:
制約:
候補:
採用:
不採用理由:
```

説明そのものが理解できない場合:

```text
主体:
入力:
処理:
出力 / 状態変化:
通信経路:
データの正本:
障害時:
代償:
```

---

# Phase 0.5: 公式20タスクと照合する

[SAP-C02カバレッジマトリクス](SAP_C02_COVERAGE_MATRIX.md) を読み、Domain 1〜4のどこが弱いかを把握する。

## 記録する

```text
Task:
現在の理解:
読んだ資料:
説明できない点:
落とした問題:
次に読む資料:
```

ページ数ではなく、Taskごとに次を判定する。

- 説明できるか
- 比較できるか
- 構成図を描けるか
- 誤答理由を言えるか
- Scenario問題を解けるか

---

# Phase 1: 基盤固め（Tier 1）

1. **IAM、Organizations、KMS、Well-Architected Framework**
2. **VPC、Route 53、CloudFront、Direct Connect、PrivateLink、Transit Gateway**
3. **EC2、Auto Scaling、ELB、EBS**
4. **S3、EFS、FSx、Storage Gateway、AWS Backup**
5. **RDS、Aurora、DynamoDB、ElastiCache**

## 各Serviceで残すメモ

```text
解決する問題:
入力 / 出力:
Control plane:
Data plane:
何と比較するか:
選ぶ制約語:
選ばない条件:
障害時:
Cost driver:
```

---

# Phase 2: 横断テーマを専門読本で固める

## 2.1 DeploymentとRollback

[Deployment and Rollback Strategies](comparisons/deployment-and-rollback-strategies.md)

説明できる状態:

- All-at-once、Rolling、Immutable、Blue/Green、Canary、Linearの違い
- Traffic rollbackとArtifact rollbackの違い
- Database変更で旧版互換が必要な理由
- Health checkとBusiness validationの違い
- IaC、Pipeline、Configuration managementの責任分担

## 2.2 Hybrid DNS

[Hybrid DNS Deep Dive](comparisons/hybrid-dns-deep-dive.md)

説明できる状態:

- Inbound EndpointとOutbound EndpointのQuery方向
- Private Hosted ZoneとNetwork connectivityの違い
- Conditional Forwarding
- DNS TTLとCutover
- TGWを作ってもDNS統合が自動化されない理由
- DNS Query成功後にIP到達性を別確認する理由

## 2.3 Performance

[Performance Design Reader](PERFORMANCE_DESIGN_READER.md)

説明できる状態:

- Latency、Throughput、IOPS、Bandwidth、Concurrency
- CPUが低いのに遅い原因
- Caching、Buffering、Replicaの目的差
- EC2、EBS、DB、QueueのBottleneck分析
- Auto Scaling指標をWork量へ合わせる方法
- RightsizingをDownsizeだけで考えない理由

## 2.4 CostとData Transfer

[Cost Modeling and Data Transfer](comparisons/cost-modeling-and-data-transfer.md)

説明できる状態:

- NAT、AZ間、Region間、Internet transferの費用境界
- Gateway EndpointとInterface Endpointの費用構造
- Rightsizing後にCommitmentを買う理由
- CURとCost Explorerの使い分け
- Unit cost、Showback、Chargeback
- Managed serviceのTCO

## 2.5 Continuous Improvement

[Continuous Improvement Playbook](CONTINUOUS_IMPROVEMENT_PLAYBOOK.md)

説明できる状態:

- Observe → Diagnose → Change → Verify
- SLI、SLO、SLA、KPI
- Metrics、Logs、Traces
- Alarmから自動修復までの経路
- Game DayとRestore test
- 改善を標準化する方法

## 2.6 Migration and Modernization

[Migration and Modernization Reader](MIGRATION_AND_MODERNIZATION_READER.md)

説明できる状態:

- Portfolio、Workload、Executionの三層
- 7Rs
- Wave Planning
- MGN、DMS、DataSync、Snow、Transfer Familyの役割
- CutoverとRollback
- Data divergence
- Rehost後のModernization

---

# Phase 3: Serverless、Containers、Loose Coupling

6. **Lambda、API Gateway、Step Functions**
7. **ECS、EKS、Fargate**
8. **EventBridge、SQS、SNS**
9. **Cognito**

## 重点課題

- EventBridgeからECS Taskを定刻起動するとき、API Gatewayが不要な理由
- SQSとEventBridgeを、BufferとRoutingの役割で分ける
- ECSとEKSを、Kubernetes要件と運用負荷から比較する
- 「疎結合」を停止時・急増時・再試行時の動作へ展開する
- SQS StandardでIdempotencyが必要な理由
- Step FunctionsとEventBridgeの違い

---

# Phase 4: Migration、Analytics、Management

10. **MGN、DMS、Schema Conversion、DataSync、Migration Hub、Snow Family**
11. **Athena、Glue、Redshift、QuickSight、Lake Formation、DataZone**
12. **SageMaker、Bedrock、Rekognition**
13. **CloudWatch、CloudTrail、Config、CloudFormation、Systems Manager、Security Hub**

Serviceページを読むときは、Phase 2の横断読本へ戻ってProgram全体の中での位置を確認する。

---

# Phase 5: 比較表で誤答を落とす

- [Networking Foundations](comparisons/networking-foundations-deep-dive.md)
- [Hybrid DNS](comparisons/hybrid-dns-deep-dive.md)
- [RDS / Aurora Connection](comparisons/rds-aurora-connection-deep-dive.md)
- [Access Control and Encryption](comparisons/access-control-and-encryption.md)
- [IAM Boundaries / SCP / Condition](comparisons/iam-boundaries-scp-condition-deep-dive.md)
- [Messaging and Eventing](comparisons/messaging-eventing-comparison.md)
- [Storage Comparison](comparisons/storage-comparison.md)
- [Deployment and Rollback](comparisons/deployment-and-rollback-strategies.md)
- [Cost and Data Transfer](comparisons/cost-modeling-and-data-transfer.md)
- [Web Runtime](glossary/web-runtime.md)
- [Pool Terms](glossary/pool-terms.md)

## 比較表の読み方

```text
共通点:
決定的な違い:
選ぶ制約語:
誤答になる条件:
組み合わせる場合:
```

---

# Phase 6: Scenario問題

1. [長文問題の読解フレーム](practice/exam-techniques.md)
2. `practice/scenario-set-01〜04`
3. Domain別問題
4. 本番形式模試

## 解答後に分類する

- 知識不足
- 用語の誤読
- 制約語の見落とし
- Serviceの役割混同
- Cost / Performance / Reliabilityの優先順位誤り
- 複数正解の選び過ぎ・不足
- 変更前後のData flowを描けなかった

## 誤答記録

```text
Question:
Selected:
Correct:
Missed constraint:
Confused concepts:
Why wrong:
Return page:
Re-test date:
```

---

# Phase 7: 口頭説明と白紙設計

次をService名の列挙ではなく、設計判断として説明する。

- Shared VPCとTransit Gateway
- VPC PeeringとPrivateLink
- ALB、NLB、GWLB
- CloudFront、Route 53、Global Accelerator
- ECS、EKS、Lambda
- RDS Multi-AZ、Read Replica、Aurora Global Database
- SQS、SNS、EventBridge、Step Functions
- Backup & Restore、Pilot Light、Warm Standby、Active/Active
- MGN、DMS、DataSync、Snow Family
- SCP、Permission Boundary、Resource Policy、Session Policy
- Blue/Green、Canary、Rolling
- Resolver Inbound、Outbound
- Cache、Buffer、Replica
- Savings Plans、Reserved Instances、Spot

さらに各説明へ次を追加する。

- 誰が動作主体か
- 何が通信・複製・保存されるか
- 状態はどこにあるか
- 障害時に何が起きるか
- 何ではないか
- どんな代償があるか

---

# Phase 8: 学習完了の判定

- AWSの短い説明を通信・Data・State・責任へ展開できる
- 公式20Taskごとに対応資料と弱点を言える
- 問題文から強い制約語を抜き出せる
- 通信経路とData flowを描ける
- 矢印がHTTP、Event、Message、Replication、DNS、Role assumptionのどれか説明できる
- Correct answerだけでなく、Distractorが失格になる理由を説明できる
- RTO/RPO、SLO、Performance、Migration downtime、Costを数値で比較できる
- DeploymentのRollbackをData互換性まで説明できる
- MigrationのCutover後Dataをどう扱うか説明できる
- 改善前後を同じMetricで検証できる

**Tips**: Service単体暗記ではなく、「誰が何をしているか」「どのConstraintで選ぶか」「何を失うか」「どう戻すか」を必ず言語化する。
