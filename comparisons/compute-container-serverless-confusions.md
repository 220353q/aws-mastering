# Compute / Container / Serverless 混同整理 — SAP-C02横断

## 何のためのページか

AWSの実行基盤は、EC2、ECS、EKS、Fargate、Lambda、App Runner、Elastic Beanstalkなどが混ざりやすい。

SAP-C02では「どこで実行するか」だけでなく、運用責任、スケール単位、権限、デプロイ方式を分けて読む必要がある。

---

## まず結論

```text
EC2 = 仮想サーバーを自分で管理
ECS = AWSネイティブなコンテナオーケストレーション
EKS = Kubernetesマネージド基盤
Fargate = サーバーレスなコンテナ実行基盤
Lambda = 関数単位のサーバーレス実行
App Runner = コンテナ/Webアプリの簡易PaaS
Elastic Beanstalk = アプリデプロイのPaaS的ラッパー
```

---

## 1. EC2 vs ECS vs EKS vs Lambda

| サービス | 実行単位 | 運用責任 | 向く用途 |
|---|---|---|---|
| EC2 | VM/OS | OS/patch/runtime管理が必要 | 最大自由度、レガシー、特殊要件 |
| ECS | Container task/service | orchestratorはAWS管理 | AWSネイティブコンテナ |
| EKS | Kubernetes Pod | Kubernetes運用知識が必要 | K8s標準、移植性、エコシステム |
| Lambda | Function | サーバー管理不要 | イベント駆動、短時間処理 |

### 覚え方

```text
EC2 = サーバーを借りる
ECS = AWS流コンテナを動かす
EKS = KubernetesをAWSで動かす
Lambda = 関数だけ置く
```

---

## 2. ECS on EC2 vs ECS on Fargate

| 起動タイプ | 何を管理するか | 向くケース |
|---|---|---|
| ECS on EC2 | EC2 cluster容量、AMI、patch、instance type | EC2制御、特殊要件、コスト最適化 |
| ECS on Fargate | task定義とコンテナ | サーバー管理削減、シンプル運用 |

### 覚え方

```text
ECS on EC2 = コンテナを載せるEC2も管理する
ECS on Fargate = コンテナだけ考える
```

Fargateはコンテナ実行基盤であり、ECSそのものではない。ECS/EKSの実行基盤としてFargateを選べる。

---

## 3. ECS vs EKS

| 観点 | ECS | EKS |
|---|---|---|
| 標準 | AWSネイティブ | Kubernetes標準 |
| 学習コスト | 比較的低い | 高い |
| 移植性 | AWS寄り | Kubernetesエコシステム |
| 運用 | シンプル | K8s設計/運用が必要 |
| 試験でのキーワード | 最小運用、AWS統合 | Kubernetes、既存K8s、CNCF |

### 判断

```text
Kubernetes要件が明示される
  → EKS

AWS上でシンプルにコンテナ運用したい
  → ECS
```

---

## 4. Lambda vs Fargate

| 観点 | Lambda | Fargate |
|---|---|---|
| 実行単位 | Function | Container task/pod |
| 起動 | イベント駆動 | サービス/タスクとして起動 |
| 実行時間 | 制限あり | 長時間処理に向く |
| コンテナ | コンテナイメージ対応あり | コンテナ前提 |
| 向く用途 | 短時間・イベント駆動 | Web service, worker, batch container |

### 覚え方

```text
Lambda = 関数をイベントで起動
Fargate = コンテナをサーバーレスに実行
```

---

## 5. ECS Task Role vs Execution Role

ECSで非常に混同しやすい。

| Role | 誰が使うか | 用途 |
|---|---|---|
| Task Role | コンテナ内アプリ | DynamoDB/S3/KMS等へのアクセス |
| Execution Role | ECS agent/Fargate基盤 | ECR pull、CloudWatch Logs出力、Secrets取得など起動処理 |

### 覚え方

```text
Task Role = アプリの権限
Execution Role = 起動するための権限
```

### 誤答パターン

```text
コンテナアプリがS3へアクセスできない
  → Task Roleを見る

ECRからimageをpullできない
  → Execution Roleを見る
```

---

## 6. Auto Scaling vs Load Balancing

| 概念 | 役割 |
|---|---|
| Auto Scaling | 台数やtask数を増減する |
| Load Balancing | 通信を複数ターゲットへ振り分ける |

### 覚え方

```text
Auto Scaling = 何台にするか
Load Balancer = どこへ流すか
```

両方必要なことが多いが、役割は違う。

---

## 7. ALB vs API Gateway for Lambda

| 入口 | 向く用途 |
|---|---|
| API Gateway + Lambda | API管理、認証、スロットリング、Usage Plan |
| ALB + Lambda | L7 Load Balancer配下でLambdaをターゲットにする |
| Lambda Function URL | 簡易HTTPS公開 |

### 判断

```text
本格API管理
  → API Gateway

既存ALB配下にLambdaを混ぜたい
  → ALB Lambda target

簡易公開
  → Lambda Function URL
```

---

## 8. Elastic Beanstalk vs ECS/App Runner

| サービス | 向く用途 |
|---|---|
| Elastic Beanstalk | アプリを簡単にデプロイし、下回りはある程度AWSに任せる |
| ECS | コンテナ構成を明示的に設計する |
| App Runner | Webアプリ/APIコンテナをよりPaaS的に簡単公開する |

SAP-C02では、細かい制御・マイクロサービス・複数コンテナ設計ならECS/EKS、シンプルWebアプリ公開ならApp Runner/Beanstalkが選択肢になる。

---

## 9. Batch vs Lambda vs Step Functions

| サービス | 役割 |
|---|---|
| AWS Batch | 大量バッチジョブ、キュー、計算環境 |
| Lambda | 短時間イベント処理 |
| Step Functions | ワークフロー制御、分岐、リトライ、補償処理 |

### 覚え方

```text
Batch = ジョブを大量にさばく
Lambda = 小さな処理をイベントで動かす
Step Functions = 処理の流れを制御する
```

---

## 10. SAP-C02判断フロー

```text
OSや特殊設定を細かく管理したい？
  → EC2

コンテナをAWSネイティブに動かしたい？
  → ECS

Kubernetesが明示要件？
  → EKS

コンテナ基盤のEC2管理を避けたい？
  → Fargate

短時間イベント駆動？
  → Lambda

長時間/大量バッチジョブ？
  → AWS Batch

処理の順序、分岐、リトライ、補償が重要？
  → Step Functions
```

---

## よくある誤答パターン

| 誤答 | なぜ危険か |
|---|---|
| Fargateをオーケストレーターとして扱う | Fargateは実行基盤。ECS/EKSと組み合わせる |
| ECSとEKSを単に同じコンテナサービスと扱う | Kubernetes要件の有無が大きい |
| Lambdaを長時間常駐処理に使う | 実行時間や接続維持の制限を考える |
| Task RoleとExecution Roleを逆にする | アプリ権限と起動権限が違う |
| Auto Scalingで通信を振り分ける | 振り分けはLoad Balancer |
| ALBをAPI管理サービスとして使う | 認証/usage plan/API管理はAPI Gatewayが得意 |

---

## 最短暗記

```text
EC2 = 自分でサーバー管理
ECS = AWSネイティブコンテナ
EKS = Kubernetes
Fargate = サーバーレスコンテナ実行基盤
Lambda = 関数
Task Role = アプリ権限
Execution Role = 起動権限
Auto Scaling = 増減
Load Balancer = 振り分け
```

---

## 関連ページ

- [AWS 混同しやすい概念インデックス](aws-confusing-concepts-index.md)
- [HTTP / HTTPS / WebSocket / MQTT / gRPC / TCP / UDP とAWS入口サービス](protocols-and-aws-entrypoints.md)
- [EventBridge vs SNS vs SQS vs Step Functions vs Amazon MQ](messaging-eventing.md)
- [Elastic Load Balancing](../services/networking/elb.md)
