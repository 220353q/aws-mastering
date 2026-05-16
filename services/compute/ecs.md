# Amazon ECS (Elastic Container Service)

## Overview
AWS ネイティブのフルマネージドコンテナオーケストレーションサービス。
EC2 または Fargate 上でコンテナを実行する。Kubernetes より学習コストが低く、AWS サービスとの統合が深い。

---

## 主要コンセプト

| コンセプト | 説明 |
|---|---|
| **Task Definition** | コンテナの設計図（イメージ, CPU/メモリ, 環境変数, IAM Role） |
| **Task** | Task Definition の実行インスタンス（1回限りのバッチ処理など） |
| **Service** | Task を指定数維持・ALB と統合・ローリングデプロイを管理 |
| **Cluster** | Task/Service を実行する論理グループ（EC2 or Fargate） |
| **Task Role** | コンテナが AWS サービスにアクセスするための IAM Role |
| **Execution Role** | ECS エージェントが ECR からイメージを Pull するための IAM Role |

---

## 起動タイプ: EC2 vs Fargate

| | EC2 起動タイプ | Fargate 起動タイプ |
|---|---|---|
| **インフラ管理** | EC2 インスタンスを自分で管理 | サーバーレス（管理不要） |
| **コスト** | 長時間稼働なら安い | 短時間・断続的なら安い |
| **カスタマイズ** | GPU インスタンス、カスタム AMI 可 | 制限あり（GPU 非対応） |
| **スケーリング** | EC2 ASG + ECS Service Auto Scaling | ECS Service Auto Scaling のみ |
| **SAP 用途** | 大規模常時稼働 / GPU ワークロード | マイクロサービス / バッチ |

---

## ECS vs EKS の選択基準

| 観点 | ECS | EKS |
|---|---|---|
| **学習コスト** | 低（AWS ネイティブ） | 高（Kubernetes 知識が必要） |
| **AWS 統合** | 深い（ALB, IAM, CloudWatch がシンプル） | 可能だが設定が複雑 |
| **マルチクラウド / ポータビリティ** | 低（AWS 依存） | 高（K8s 標準） |
| **エコシステム** | AWS 中心 | Helm, Istio 等 K8s エコシステム |
| **SAP 用途** | AWS 完結の新規開発 | オンプレ K8s からの移行 / マルチクラウド |

**選択の鉄則**: AWS 完結の新規開発 → ECS。オンプレ Kubernetes からの移行 / マルチクラウド → EKS。

---

## ネットワーキングモード

| モード | 説明 | 用途 |
|---|---|---|
| **awsvpc** | 各タスクに ENI を割り当て（独立した IP） | 推奨。セキュリティグループをタスク単位で設定可 |
| **bridge** | Docker の NAT ブリッジ（EC2 のみ） | レガシー |
| **host** | EC2 のネットワークを直接使用 | 高パフォーマンス（ポートの競合に注意） |

---

## Blue/Green デプロイ with CodeDeploy

```
ALB
 ├── Target Group Blue (現行: v1.0)  ← 100% トラフィック
 └── Target Group Green (新規: v2.0) ← 0%

CodeDeploy が徐々にトラフィックを Green に移行
→ 問題なければ Blue を削除
→ 問題があれば即時 Blue に戻す（ロールバック）
```

---

## Use Cases (Tier 1)

1. **マイクロサービス API** — Fargate + ALB + ECS Service。サーバーレスでスケーラブル。
2. **バッチ処理** — Fargate Task を EventBridge でスケジュール実行。完了後リソース解放。
3. **Blue/Green デプロイ** — CodeDeploy + ECS でゼロダウンタイムデプロイ。
4. **レガシーコンテナ移行** — EC2 起動タイプで特殊要件（GPU, カスタム OS）に対応。
5. **CI/CD パイプライン** — CodePipeline → CodeBuild (Build) → ECS (Deploy)。

---

## Connections

- **ECR**: コンテナイメージのレジストリ
- **ALB**: トラフィック分散 + ヘルスチェック + Blue/Green
- **IAM**: Task Role（コンテナの権限）+ Execution Role（ECS エージェントの権限）
- **CloudWatch Logs**: コンテナログを自動収集（awslogs ドライバー）
- **X-Ray**: コンテナ間の分散トレーシング（サイドカーパターン）
- **Secrets Manager / SSM Parameter Store**: 環境変数の安全な注入

---

## Well-Architected

- **信頼性**: Multi-AZ デプロイ + Service Auto Scaling でタスク数を維持
- **セキュリティ**: Task Role で最小権限 / awsvpc でタスク単位のセキュリティグループ
- **コスト**: Fargate Spot でバッチ処理コスト 70% 削減可能
- **運用**: CloudWatch Container Insights でコンテナ単位のメトリクス収集

---

## SAP-C02 頻出問題パターン

| キーワード | 正解アプローチ |
|---|---|
| "containerize", "no server management" | Fargate |
| "GPU workload", "custom AMI" | ECS EC2 起動タイプ |
| "zero downtime deployment", "containers" | ECS + CodeDeploy Blue/Green |
| "migrate from on-premises Kubernetes" | EKS（ECS ではない） |
| "container logs to CloudWatch" | awslogs ドライバー設定 |
