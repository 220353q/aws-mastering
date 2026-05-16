# AWS Fargate

## Overview
コンテナ用のサーバーレスコンピューティングエンジン。**EC2 インスタンスの管理が不要**。
ECS と EKS の両方で起動タイプとして使用できる。CPU/メモリ単位での従量課金。

---

## Fargate の価値提案

```
従来: EC2 クラスター管理 → AMI 更新 → キャパシティ計画 → スケーリング設定 → パッチ適用
Fargate: Task/Pod の定義だけ → あとは AWS が管理
```

**採用基準:**
- サーバー管理チームがない / 小規模チーム
- ワークロードが断続的・バースト型
- セキュリティ要件: ホスト OS に触らせたくない

---

## Fargate のリソース割り当て

タスク定義で CPU と メモリ を指定する（組み合わせに制約あり）。

| vCPU | メモリ範囲 |
|---|---|
| 0.25 | 0.5 GB 〜 2 GB |
| 0.5 | 1 GB 〜 4 GB |
| 1 | 2 GB 〜 8 GB |
| 2 | 4 GB 〜 16 GB |
| 4 | 8 GB 〜 30 GB |
| 8 | 16 GB 〜 60 GB |
| 16 | 32 GB 〜 120 GB |

**GPU は非対応**（GPU ワークロードは ECS/EKS の EC2 起動タイプを使用）。

---

## Fargate Spot

- **最大 70% コスト削減** が可能なスポット型 Fargate。
- AWS の余剰キャパシティを使用するため、中断の可能性あり。
- **適合するワークロード**: バッチ処理・ETL・テスト・CI/CD ビルド（中断耐性あり）
- **不適合**: API サービス・リアルタイム処理（中断が許容されない）

```
コスト最適化パターン:
  ECS Service: 通常 Fargate（本番 API → 中断不可）
  ECS Task (バッチ): Fargate Spot（コスト 70% 削減）
```

---

## Fargate のネットワーク

- **必ず awsvpc モード**: 各タスクに独立した ENI と IP アドレス
- セキュリティグループをタスク単位で設定可能
- **VPC エンドポイント推奨**: ECR からのイメージ Pull を Private に（NAT Gateway コスト削減）

---

## ECS on Fargate vs Lambda の選択

| | Fargate | Lambda |
|---|---|---|
| **最大実行時間** | 無制限 | 15 分 |
| **コンテナ** | Docker コンテナを直接実行 | コンテナイメージ対応（最大 10GB） |
| **起動時間** | 数秒〜数十秒（コールドスタート大） | ミリ秒〜数秒 |
| **状態** | ファイルシステム一時利用可 | ステートレス |
| **コスト** | 秒単位（CPU+メモリ） | リクエスト数 + 実行時間 |
| **適合** | 長時間処理・既存コンテナ移行 | 短時間イベント処理 |

---

## Use Cases (Tier 1)

1. **サーバーレスマイクロサービス** — ECS on Fargate + ALB。EC2 管理ゼロで本番 API を運用。
2. **スケジュールバッチ** — EventBridge → Fargate Task。完了後リソース自動解放。
3. **CI/CD ビルド環境** — CodeBuild + Fargate Spot で並列ビルドをコスト最小化。
4. **コンテナ移行 PoC** — オンプレのコンテナアプリをそのまま Fargate に移植して検証。
5. **マルチテナント処理** — テナントごとに独立した Fargate Task を起動しセキュリティを分離。

---

## Connections

- **ECS / EKS**: Fargate は起動タイプとして機能する（オーケストレーターと分離）
- **ALB / NLB**: ロードバランサー経由でトラフィックを受信
- **ECR**: イメージを Pull（VPC エンドポイント経由推奨）
- **CloudWatch Logs**: awslogs ドライバーでコンテナログを自動送信
- **EventBridge**: バッチ Task のスケジュール起動

---

## SAP-C02 頻出問題パターン

| キーワード | 正解アプローチ |
|---|---|
| "no server management", "containers" | Fargate |
| "cost optimization", "batch jobs" | Fargate Spot |
| "GPU containers" | ECS EC2 起動タイプ（Fargate 非対応） |
| "long-running container > 15 min" | Fargate（Lambda は 15 分上限） |
| "existing Docker workload, serverless" | ECS on Fargate |
