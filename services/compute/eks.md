# Amazon EKS (Elastic Kubernetes Service)

## Overview
AWS マネージドの Kubernetes サービス。Kubernetes コントロールプレーン（etcd, API Server 等）を AWS が管理する。
オンプレミス Kubernetes との互換性・マルチクラウド移植性が強み。

---

## EKS の主要コンポーネント

| コンポーネント | 説明 |
|---|---|
| **Control Plane** | AWS マネージド。etcd, API Server, Scheduler。Multi-AZ で高可用性 |
| **Node Group (EC2)** | ワーカーノード。マネージドノードグループ or 自己管理 |
| **Fargate Profile** | サーバーレスノード。EC2 管理不要 |
| **Pod** | コンテナの最小デプロイ単位 |
| **Namespace** | クラスター内の論理的な分離 |

---

## ノードタイプ比較

| | マネージドノードグループ | Fargate Profile | 自己管理ノード |
|---|---|---|---|
| **OS 管理** | AWS が AMI 更新 | 不要（サーバーレス） | 自分で管理 |
| **スケーリング** | Cluster Autoscaler | 自動 | 自分で設定 |
| **GPU 対応** | 対応 | 非対応 | 対応 |
| **コスト** | EC2 料金 | Pod 単位の料金 | EC2 料金（最安だが管理コスト高） |
| **用途** | 一般的なワークロード | バースト / バッチ | 特殊要件 |

---

## AWS サービスとの統合

| AWS サービス | 統合方法 | 用途 |
|---|---|---|
| **IAM** | IRSA (IAM Roles for Service Accounts) | Pod 単位の IAM 権限付与 |
| **ALB** | AWS Load Balancer Controller | Ingress → ALB 自動プロビジョニング |
| **ECR** | 標準で統合 | プライベートイメージレジストリ |
| **CloudWatch** | Container Insights | メトリクス・ログ収集 |
| **Secrets Manager** | External Secrets Operator | シークレットを Pod に安全に注入 |
| **VPC CNI Plugin** | ネイティブ統合 | Pod に VPC の IP を直接割り当て |

---

## IRSA (IAM Roles for Service Accounts)

EKS の最重要セキュリティ概念。Pod 単位で IAM Role を付与する仕組み。

```
1. IAM Role を作成（信頼ポリシーに EKS OIDC を指定）
2. Kubernetes ServiceAccount に IAM Role ARN をアノテーション
3. Pod が ServiceAccount を使用すると、自動的に IAM 一時クレデンシャルを取得
→ EC2 インスタンスプロファイルと異なり、Pod 単位で最小権限を実現
```

---

## EKS Anywhere / EKS Distro

| | EKS | EKS Anywhere | EKS Distro |
|---|---|---|---|
| **実行場所** | AWS クラウド | オンプレ / VMware | 任意（OSS） |
| **管理** | AWS マネージド | ユーザー管理（AWS ツール付き） | ユーザー管理 |
| **用途** | クラウドネイティブ | ハイブリッドクラウド | 自社 K8s 構築 |

---

## Use Cases (Tier 1)

1. **オンプレ Kubernetes からの移行** — EKS で K8s 互換性を保ちつつ AWS に移行。
2. **マルチクラウド / ポータブル設計** — Helm Chart を使い他クラウドにも展開可能。
3. **大規模マイクロサービス** — Namespace で複数チームの環境を論理分離。
4. **ML ワークロード** — GPU ノードグループ + Kubeflow でトレーニング基盤。
5. **ハイブリッドクラウド** — EKS Anywhere でオンプレと AWS を統一 K8s で管理。

---

## Connections

- **ECR**: コンテナイメージの格納
- **ALB + AWS Load Balancer Controller**: Ingress リソースから ALB を自動生成
- **IAM (IRSA)**: Pod 単位の最小権限 IAM 制御
- **VPC CNI**: Pod に VPC ネイティブ IP を割り当て（セキュリティグループ適用可）
- **Karpenter**: 次世代ノードオートスケーラー（Cluster Autoscaler より高速・効率的）

---

## SAP-C02 頻出問題パターン

| キーワード | 正解アプローチ |
|---|---|
| "migrate from on-premises Kubernetes" | EKS |
| "multi-cloud portability" | EKS（ECS は AWS 依存） |
| "IAM permissions per pod" | IRSA |
| "hybrid cloud, same K8s" | EKS Anywhere |
| "AWS native, simpler management" | ECS（EKS より簡単） |
