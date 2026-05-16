# SAP-C02 Scenario Set 04

Data / Integration / Governance / Compute Modernization を中心にした長文シナリオ型問題集。選択肢は A〜G の7択。正解数は1〜3個。

---

## Question 1: 注文処理のイベント駆動化

あるEC企業は注文確定後に、決済、在庫引当、配送手配、メール通知、分析イベント送信を行っている。現在は単一Lambdaから全処理を同期呼び出ししており、どれか1つが遅延すると注文API全体が遅くなる。今後は処理を疎結合にし、メールや分析のような複数購読者へイベントを配り、在庫引当のような確実に処理したい非同期タスクは再試行とDLQで保護したい。複雑な補償処理も一部必要である。最も適切な設計はどれか。

A. 注文イベントをEventBridgeへ発行し、ルールで処理先を振り分ける。

B. 複数購読者への通知やfanoutにはSNSを使う。

C. 確実に処理したい非同期ワーカーにはSQSとDLQを使う。

D. 複数ステップの状態管理や補償処理にはStep Functionsを使う。

E. すべてを1つの同期Lambdaに残し、タイムアウト値だけを最大にする。

F. SQSだけを使えば、イベント内容に応じた高度なルーティング、fanout、状態管理がすべて自動で完了する。

G. CloudFrontのキャッシュを使えば、注文処理の補償トランザクションを実現できる。

**正解: A, C, D**

### 判断軸

- EventBridge: イベントルーティング。
- SQS: キュー、再試行、DLQ、ワーカー平準化。
- Step Functions: 複数ステップの状態管理、分岐、補償処理。
- SNSはfanout向きだが、確実なワーカー処理や状態管理はSQS/Step Functionsと組み合わせる。

### 構成図

```text
Order API ── PutEvents ── EventBridge
                         ├─ SQS ── Inventory Worker ── DLQ
                         ├─ Step Functions ── Payment / Compensation
                         └─ SNS ── Email / Analytics Subscribers
```

### 誤答理由

Eは疎結合要件を満たさない。FはSQSを万能化している。GはCDNとトランザクション制御の混同。Bも有効だが、設問の中心要件である非同期ワーカー保護と補償処理まで含めるとA/C/Dが中核。

関連: [EventBridge](../services/integration/eventbridge.md), [SQS](../services/integration/sqs.md), [SNS](../services/integration/sns.md), [Step Functions](../services/integration/stepfunctions.md)

---

## Question 2: データレイクの発見・申請・列制御

ある小売企業はS3にPOS、EC、在庫、広告、サポートのデータを集約している。データエンジニアはETLで形式を整え、アナリストはSQLで探索し、経営層はダッシュボードで確認したい。部門横断でデータセットを発見し、利用申請と承認の履歴を残し、顧客PII列は許可された利用者だけに見せたい。最も適切な組み合わせはどれか。

A. Glue Data CatalogとCrawlerでメタデータを管理し、Glue JobsでETLを実行する。

B. AthenaでS3上のデータをSQL分析する。

C. Lake Formationでテーブル/列レベル権限を管理する。

D. DataZoneでデータ発見、公開、申請、承認を支援する。

E. S3バケットを全員にRead許可し、列レベル権限は命名規則で運用する。

F. QuickSightだけでETL、Data Catalog、Lake Formation権限、承認ワークフローをすべて代替する。

G. KMSで暗号化すれば、部門別の列表示制御は不要になる。

**正解: A, C, D**

### 判断軸

- Glue: カタログ/ETL。
- Athena: S3上のSQL分析。
- Lake Formation: データレイク権限。
- DataZone: データ発見・共有・申請の上位ガバナンス。

### 構成図

```text
S3 Data Lake
   ├─ Glue Crawler / Data Catalog
   ├─ Glue ETL Jobs
   ├─ Lake Formation: table/column permissions
   ├─ Athena: SQL Query
   ├─ QuickSight: BI
   └─ DataZone: discover / request / approve / share
```

### 誤答理由

Eは最小権限を崩す。FはQuickSightの役割を広げすぎ。Gは暗号化と認可の混同。Bも分析要件には必要だが、設問の中心であるETL/権限制御/申請ガバナンスの組み合わせとしてA/C/Dを選ぶ。

関連: [Glue](../services/analytics/glue.md), [Athena](../services/analytics/athena.md), [Lake Formation](../services/analytics/lakeformation.md), [DataZone](../services/analytics/datazone.md), [QuickSight](../services/analytics/quicksight.md)

---

## Question 3: 継続的コンプライアンスと自動修復

ある医療系企業は複数アカウントにまたがるAWS環境で、S3パブリック公開、EBS未暗号化、Security Groupの0.0.0.0/0開放、CloudTrail停止を継続的に検出したい。監査証跡は中央に保存し、設定逸脱は標準ルールで評価し、重大所見はセキュリティ運用アカウントへ集約したうえで、自動修復ワークフローを起動したい。新規アカウントにも同じ設定を配布したい。最も適切な組み合わせはどれか。

A. Organization trailでAPI操作履歴を中央ログアーカイブへ集約する。

B. AWS Config Rules / Conformance Packsで設定準拠を継続評価する。

C. Security Hubを委任管理者アカウントで集約し、重大度に応じて優先度付けする。

D. EventBridgeからSystems Manager AutomationまたはLambdaへ連携し、修復処理を実行する。

E. CloudWatchメトリクスだけでAPI監査、設定履歴、所見集約、自動修復をすべて代替する。

F. CloudFormation StackSetsやControl Towerで標準設定を新規/既存アカウントへ展開する。

G. WAFを有効化すれば、EBS未暗号化やCloudTrail停止も自動的に検出される。

**正解: A, B, C**

### 判断軸

- CloudTrail: API監査。
- Config: リソース設定履歴と準拠評価。
- Security Hub: セキュリティ所見集約。
- EventBridge + Automation/Lambda: 自動修復連携。
- StackSets/Control Tower: 標準設定展開。

### 構成図

```text
Member Accounts
  ├─ CloudTrail Org Trail ──> Log Archive S3
  ├─ AWS Config Rules / Conformance Packs
  └─ Security Hub Findings ──> Delegated Admin
                                  │
                                  ▼
                         EventBridge ── SSM Automation / Lambda
```

### 誤答理由

EはCloudWatchを万能化している。GはWAFの用途違い。D/Fも重要な拡張要素だが、設問の「検出・集約」の中核としてA/B/Cを選ぶ。

関連: [CloudTrail](../services/management/cloudtrail.md), [Config](../services/management/config.md), [Security Hub](../services/management/security-hub.md), [CloudFormation](../services/management/cloudformation.md)

---

## Question 4: ECS・EKS・Fargateの選択

ある企業は複数のマイクロサービスをAWSへ移行する。大半のチームはKubernetes運用経験がなく、コンテナをできるだけ少ない運用負荷で動かしたい。一方、一部のプラットフォームチームは既存のKubernetesマニフェストやエコシステムを使いたい。セキュリティ部門はノード管理を減らし、タスク/Pod単位で最小権限を付与したい。どの判断が適切か。

A. Kubernetes APIや既存Kubernetesエコシステムが強い要件ならEKSを検討する。

B. AWSネイティブでシンプルにコンテナを動かしたい場合はECSを検討する。

C. サーバー/ノード管理を減らしたい場合は、ECS on FargateまたはEKS on Fargateを検討する。

D. Fargateを使うと、すべてのワークロードでDaemonSetや特権コンテナが自由に使える。

E. ECSではタスクロールを使い、EKSではIRSAなどでPod単位のAWS権限分離を検討する。

F. EKSを選べばKubernetesの運用設計、アップグレード、アドオン管理は一切不要になる。

G. ALB/NLBとの連携やAuto Scalingはコンテナでは使えない。

**正解: A, B, C**

### 判断軸

- ECS: AWSネイティブで比較的シンプル。
- EKS: Kubernetes互換性・エコシステム。
- Fargate: サーバーレスコンテナ実行基盤。ノード管理を減らす。
- 権限分離はECS Task Role / EKS IRSAなどを使う。

### 構成図

```text
Requirement
  ├─ Simple AWS-native containers ── ECS
  ├─ Kubernetes API/ecosystem ───── EKS
  └─ Reduce node management ─────── Fargate
       ├─ ECS on Fargate
       └─ EKS on Fargate
```

### 誤答理由

DはFargateの制約を無視している。FはEKSの運用責任をゼロにしている。Gは誤り。Eは正しい補足だが、設問の選択軸としてはA/B/Cが中核。

関連: [ECS](../services/compute/ecs.md), [EKS](../services/compute/eks.md), [Fargate](../services/compute/fargate.md), [ELB](../services/networking/elb.md)

---

## Question 5: API公開方式と認証・認可

あるスタートアップはモバイルアプリ向けAPIを新規構築する。ユーザー認証はメール/パスワードと外部IdP連携を使い、MFAも導入したい。APIはLambda中心で実装し、一部のユーザーにはS3へ直接アップロードさせたいが、長期認証情報は配布したくない。APIの前段ではレート制御や認可を行い、静的コンテンツは世界中へキャッシュ配信したい。最も適切な組み合わせはどれか。

A. Cognito User Poolでユーザー認証を行う。

B. Cognito Identity Poolで必要に応じてAWS一時認証情報を発行し、S3アップロードなどを最小権限ロールで許可する。

C. API Gateway + LambdaでAPIを実装し、AuthorizerやUsage Plan/Throttleを使う。

D. 静的コンテンツはCloudFront + S3 Originで配信する。

E. IAM Userのアクセスキーをアプリに埋め込み、S3アップロードに使う。

F. CloudFrontだけでユーザー登録、MFA、AWS一時認証情報発行をすべて実装する。

G. S3バケットをPublic Writeにして、アプリから直接アップロードさせる。

**正解: A, B, C**

### 判断軸

- User Pool: アプリユーザー認証。
- Identity Pool: AWS一時認証情報。
- API Gateway + Lambda: API公開とサーバーレス実装。
- CloudFront: 静的コンテンツ配信。

### 構成図

```text
Mobile App
  ├─ Sign-in ── Cognito User Pool
  ├─ Temporary AWS Credentials ── Cognito Identity Pool ── IAM Role ── S3 Upload
  └─ API Call ── API Gateway ── Lambda
Static Assets ── CloudFront ── S3 Origin
```

### 誤答理由

E/Gは長期認証情報や公開書き込みで危険。FはCloudFrontの役割を超えている。Dも要件に合うが、設問の認証・認可・API公開の中核はA/B/C。

関連: [Cognito](../services/security/cognito.md), [API Gateway](../services/integration/apigateway.md), [Lambda](../services/compute/lambda.md), [CloudFront](../services/networking/cloudfront.md)
