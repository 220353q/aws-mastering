# Deployment and Rollback Strategies

> デプロイ戦略は「どう新バージョンを配るか」だけではない。**どこで異常を検知し、どの単位で停止し、どの状態へ戻すか**まで含めて設計する。

SAP-C02では、Blue/Green、Canary、Rollingなどの名前を覚えるだけでは不十分である。次を一続きで説明できる必要がある。

```text
変更を作る
  → 検証する
  → 配布する
  → 一部へ反映する
  → 監視する
  → 継続 / 停止 / Rollbackを判断する
  → 旧状態または修正版へ戻す
```

---

## 1. 最初に決める六つのこと

| 判断 | 質問 |
|---|---|
| 変更対象 | Infrastructure、Application、Configuration、Databaseのどれか |
| 影響単位 | 全台、AZ、Target Group、Task、Function version、Stackのどこまでか |
| 同時変更率 | 一度に何%を新しくするか |
| 検証方法 | Health check、Synthetic、Business KPI、Error rateのどれを見るか |
| 許容時間 | 配布完了まで何分・何時間を許容するか |
| 戻し方 | Trafficを戻す、旧Artifactを再配置する、Stackをrollbackする、DBを前進修正する |

デプロイ名は、この六つを決めた結果である。

---

# 2. 戦略の比較

| 戦略 | 新旧同時稼働 | 変更速度 | Blast radius | Rollback | 主な用途 |
|---|---:|---:|---:|---:|---|
| All-at-once | 短時間のみ / なし | 最速 | 最大 | 再配布が中心 | 開発、停止許容、単純処理 |
| Rolling | 部分的 | 中 | Batch単位 | 旧版を再Rolling | EC2 Fleet、ECS Service |
| Rolling with additional batch | 部分的 | 中 | Batch単位 | 比較的容易 | 容量を落としにくい環境 |
| Immutable | あり | 中〜遅 | 新環境へ隔離 | 旧環境維持 | Instance差し替え、再現性重視 |
| Blue/Green | あり | 中 | Traffic切替単位 | Trafficを旧系へ戻す | Web/API、ECS、RDS更新 |
| Canary | あり | 遅め | 最小 | 少量Trafficを戻す | Lambda/API、重要サービス |
| Linear | あり | 遅め | 段階単位 | 段階停止・旧系へ戻す | 安全性と所要時間の中間 |
| Feature flag | 同一環境内 | 即時 | User/機能単位 | Flagを戻す | 機能公開、実験、Kill switch |

## 重要な見方

- **Blue/Green**は環境を二つ用意し、Trafficの向きを切り替える。
- **Canary**は新環境へ送るTrafficを少量から増やす。
- **Rolling**は実行基盤を少しずつ置き換える。
- **Immutable**は既存Instanceを変更せず、新しいInstance群を作る。
- **Feature flag**はデプロイとリリースを分離する。

一つのシステムで組み合わせることもある。

```text
Infrastructure: Immutable
Application traffic: Canary
Feature exposure: Feature flag
Database change: Expand and Contract
```

---

# 3. All-at-once

全対象を一度に新バージョンへ変える。

```text
Old v1: 100%
  ↓ deploy
New v2: 100%
```

## 長所

- 最短時間
- 実装が単純
- 二重環境の費用が少ない

## 短所

- 障害時の影響が全体へ及ぶ
- 旧版と新版の比較時間がない
- Rollback中も全体影響が続く

## 適する条件

- 開発・検証環境
- 停止を許容できる内部システム
- Statelessで再配布が容易
- 処理対象を再実行できるBatch

## 誤答パターン

「最小コスト」だけを理由に、停止不可の本番システムへAll-at-onceを選ばない。

---

# 4. Rolling

一部のInstanceやTaskを順番に置き換える。

```text
Step 1: v1 v1 v1 v2
Step 2: v1 v1 v2 v2
Step 3: v1 v2 v2 v2
Step 4: v2 v2 v2 v2
```

## 設計ポイント

- Minimum healthy percentage
- Maximum percentage
- Batch size
- Deregistration delay
- Connection draining
- Startup / warm-up time
- Session state

## 危険な状態

新旧が同時稼働する期間があるため、次の互換性が必要である。

- API request / response
- Message schema
- Database schema
- Session format
- Cache key

## Rollback

旧版Artifactを同じ方式で再配布する。Blue/Greenのような瞬時Traffic切替ではないため、戻し完了まで時間がかかる。

---

# 5. Immutable

既存環境を直接変更せず、新しいImageまたはArtifactから新環境を作る。

```text
Existing v1 instances: 変更しない
New v2 instances: 新規作成
  → health verification
  → traffic移行
  → old削除
```

## 長所

- Configuration driftを減らせる
- Build済みArtifactをそのまま再現できる
- 旧環境をRollback先として残せる

## 短所

- 一時的に二重Capacityが必要
- 起動と検証に時間がかかる
- Stateful workloadはデータ移行を別途設計する

## AWSでの読み替え

- AMI差し替え
- Auto Scaling Group Instance Refresh
- Elastic Beanstalk Immutable deployment
- ECS Task Definition revision
- Lambda Version
- CloudFormation replacement

---

# 6. Blue/Green

Blueを現行、Greenを新環境として並行稼働させる。

```text
Client
  → Router / Load Balancer
      ├─ Blue v1: current
      └─ Green v2: candidate
```

検証後、TrafficをGreenへ切り替える。

## Rollbackが速い理由

旧環境Blueが残っているため、TrafficをBlueへ戻せる。ただし、次の条件が必要である。

- Blueがまだ稼働している
- DB schemaが旧版と互換
- 新版で書かれたデータを旧版が読める
- External side effectを取り消せる

## よくある誤解

Blue/Greenなら必ず安全ではない。Shared Databaseを変更して旧版が動けなくなれば、Trafficだけ戻してもRollbackできない。

## AWSでの代表

- CodeDeploy + EC2 / ECS
- ECS two target groups
- Lambda Alias
- RDS Blue/Green Deployments
- Route 53 weighted routing
- ALB listener切替

---

# 7. Canary

最初に少量のTrafficだけを新バージョンへ送る。

```text
v1: 95%
v2:  5%
  → metrics正常
v1: 75%
v2: 25%
  → metrics正常
v2: 100%
```

## 見るべき指標

### Technical metrics

- 5xx rate
- Latency p95 / p99
- Timeout
- CPU / Memory
- Throttle
- DLQ count

### Business metrics

- Login success rate
- Checkout completion
- Payment authorization success
- Conversion rate
- Search zero-result rate

Technical metricだけ正常でも、業務機能が壊れている可能性がある。

## Canaryの単位

- Request percentage
- User cohort
- Region
- Tenant
- API path
- Function Alias weight

## 注意

少量Trafficでは、低頻度バグや長時間処理を検出できないことがある。観測期間を負荷周期に合わせる。

---

# 8. Linear

一定間隔で一定割合ずつTrafficを新バージョンへ移す。

```text
10% → 20% → 30% → ... → 100%
```

Canaryより規則的で、All-at-onceより安全。配布時間は長くなる。

## 適する条件

- 段階的に影響範囲を広げたい
- 観測に一定時間が必要
- Traffic比率を自動制御したい

---

# 9. Feature Flag

コードをデプロイしても、機能をすぐ公開しない。

```text
Deploy code v2
  → feature disabled
  → internal users only
  → 5% customers
  → all customers
```

## デプロイとリリースの分離

- Deployment: Codeを環境へ置く
- Release: Userへ機能を見せる

## 長所

- 機能単位で停止できる
- User cohortごとに公開できる
- A/B testに使える

## 短所

- Flagが増えると分岐が複雑になる
- 古いFlagの削除が必要
- Security controlの代替にはならない

---

# 10. Database変更とRollback

Applicationは戻せても、Database変更は簡単に戻せない。

## 危険な変更

```sql
ALTER TABLE orders DROP COLUMN legacy_status;
```

旧Applicationが`legacy_status`を読む場合、旧版へ戻せない。

## Expand and Contract

### Phase 1: Expand

新旧両方が動ける変更を追加する。

```text
Old columnを残す
New columnを追加
Applicationは両方へ対応
```

### Phase 2: Migrate

既存データを移し、読み書きを新形式へ寄せる。

### Phase 3: Contract

旧版が不要になった後で旧Columnを削除する。

## 原則

- Additive changeを先に行う
- Destructive changeを最後に行う
- Schema versionとApplication versionを独立管理する
- Dual writeは失敗時の整合性を設計する
- RollbackよりRoll-forwardが安全な変更もある

---

# 11. Infrastructure as Code

IaCの利点は「自動化」だけではない。

- Review可能なDiff
- Repeatability
- Environment consistency
- Change history
- Automated validation
- Rollback / recovery

## CloudFormation Change Set

実行前に、作成・変更・削除・置換されるResourceを確認する。

特に`Replacement`は、Resource ID変更、停止、データ移行につながる可能性がある。

## Stack rollback

Resource作成や更新が失敗した場合、CloudFormationはStackを以前の安定状態へ戻そうとする。ただし、外部処理やData plane上の副作用をすべて自動で戻すわけではない。

## Drift

Consoleや別経路で変更すると、Templateと実環境がずれる。

```text
Desired state: Template
Actual state: AWS resources
Difference: Drift
```

Drift detection後は、どちらを正とするか判断する。

---

# 12. CI/CDの責任分解

```text
Source
  → Build
  → Test
  → Package
  → Deploy
  → Observe
  → Approve / Rollback
```

| 段階 | 代表的な責任 | AWS候補 |
|---|---|---|
| Source | Version管理、Review | GitHub等 |
| Orchestration | StageとApproval | CodePipeline |
| Build | Compile、Unit test、Image作成 | CodeBuild |
| Artifact | Package保存 | S3、ECR、CodeArtifact |
| Deploy | EC2/ECS/Lambdaへ配布 | CodeDeploy、CloudFormation |
| Configuration | Patch、Run Command、State | Systems Manager |
| Observe | Metrics、Logs、Trace | CloudWatch、X-Ray |

Pipelineは「DeployするService」ではなく、工程をつなぐOrchestratorとして読む。

---

# 13. Rollbackの種類

## 13.1 Traffic rollback

Trafficを旧環境へ戻す。

- ALB listener
- Route 53 weight
- Lambda Alias
- ECS target group

速いが、Data互換性が必要。

## 13.2 Artifact rollback

旧Artifactを再配置する。

- Previous container image
- Previous Lambda version
- Previous AMI

完了まで時間がかかる。

## 13.3 Infrastructure rollback

TemplateやStackを以前の状態へ戻す。

Resource replacementやData resourceの扱いに注意する。

## 13.4 Configuration rollback

Feature flag、Parameter、AppConfigなどを以前の値へ戻す。

Codeを戻すより速い場合がある。

## 13.5 Data rollback

Snapshot / PITRから復元する。復元後の書き込みをどう扱うかが難しい。通常のApplication rollbackと同じ感覚で実施しない。

## 13.6 Roll-forward

原因を修正した新Versionを配布する。Database migrationやExternal side effectがある場合、旧版へ戻すより安全なことがある。

---

# 14. Automatic rollbackの設計

```text
Deploy v2
  → CloudWatch Alarm
      ├─ normal → continue
      └─ breach → stop deployment
                    → shift traffic to v1
                    → notify
```

## Alarm設計の注意

- 1分だけのSpikeで戻すか
- Missing dataをどう扱うか
- Low traffic時にError rateが有意か
- Deployment由来のAlarmか、外部障害か
- Rollback先も正常か

複数指標をComposite Alarmで組み合わせる場合もある。

---

# 15. Health Checkの階層

## Process health

Processが起動している。

## Dependency health

DB、Cache、External APIへ接続できる。

## Functional health

実際の主要機能が成功する。

## Business health

注文、決済、登録などが期待率で完了する。

Load Balancer health checkが成功していても、Business機能が正常とは限らない。

---

# 16. SessionとDeployment

SessionをInstance内に保持すると、RollingやBlue/GreenでUser体験が壊れやすい。

改善候補:

- ElastiCache
- DynamoDB
- Signed cookie / token
- Sticky sessionは一時策

Stateを外へ出すと、Target差し替えが容易になる。

---

# 17. Queue ConsumerのDeployment

非同期Workerでは、HTTP health checkだけでなくMessage処理の互換性を見る。

## 確認項目

- Old producer → New consumer
- New producer → Old consumer
- Message schema version
- Visibility Timeout
- In-flight message
- Idempotency
- DLQ

## Safe change

MessageへFieldを追加するとき、Consumerが未知Fieldを無視できる設計にする。Required fieldの突然の追加は避ける。

---

# 18. Multi-Region Deployment

同時に全Regionへ配布すると、Global outageの原因になり得る。

```text
Wave 1: Test Region
  → observe
Wave 2: Low traffic Region
  → observe
Wave 3: Primary Regions
```

## 確認項目

- RegionごとのConfiguration
- Data replication lag
- Route 53 / Global Accelerator
- Cross-Region dependency
- Artifact availability
- Rollback順序

---

# 19. Change Management

技術的に安全でも、変更管理がなければ運用上危険である。

最低限必要な記録:

- 変更目的
- 対象Resource
- Risk
- Expected impact
- Validation
- Rollback condition
- Rollback procedure
- Owner
- Execution window
- Result

Manual approvalは常に必要ではない。高Risk変更だけ承認し、定型変更は自動化する。

---

# 20. SAP-C02での判断

## 問題文: 最小の停止時間、即時Rollback

Blue/GreenまたはCanaryを優先検討する。

## 問題文: 一時的な二重費用を避けたい

Rollingを検討する。ただしCapacity低下とRollback時間を確認する。

## 問題文: 新Versionを少数利用者で検証

CanaryまたはFeature flag。

## 問題文: Configuration driftを防ぎたい

Immutable image + IaC。

## 問題文: Database schema変更を伴う

Expand and Contract、Backward compatibility、Roll-forwardを確認する。

## 問題文: 複数Accountへ標準Resourceを配布

CloudFormation StackSetsを検討する。

## 問題文: Patchと定型運用を集中自動化

Systems Managerを検討する。

---

# 21. よくある誤答

- Blue/GreenならDatabaseも自動で元に戻る
- CanaryならMonitoringは不要
- Rollingは常に無停止
- Feature flagをIAM認可の代わりにする
- CloudFormation rollbackがExternal APIの副作用も取り消す
- Pipelineを作ればTest品質が自動的に高くなる
- Health check成功だけで業務機能正常と判断する
- Latest tagのContainer imageを使い、同じVersionを再現できなくする

---

# 22. 設計テンプレート

```text
変更対象:
Deployment単位:
選択戦略:
同時変更率:
新旧互換性:
監視指標:
観測時間:
停止条件:
Rollback方式:
Database変更:
External side effect:
承認者:
復旧確認:
```

---

# 23. 一文で説明する

> この変更では、旧環境を維持したまま新環境へ少量Trafficを送り、Error rateと業務成功率を観測するCanaryを採用する。異常時はAliasを旧Versionへ戻す。ただしDatabase schemaは旧版互換を保ち、破壊的変更は全Traffic移行後に行う。

ここまで説明できれば、単なる戦略名の暗記ではなく、DeploymentとRollbackを設計できている。

## 関連資料

- [CloudFormation](../services/management/cloudformation.md)
- [Systems Manager](../services/management/ssm.md)
- [CloudWatch](../services/management/cloudwatch.md)
- [ECS](../services/compute/ecs.md)
- [Lambda](../services/compute/lambda.md)
- [Continuous Improvement Playbook](../CONTINUOUS_IMPROVEMENT_PLAYBOOK.md)
