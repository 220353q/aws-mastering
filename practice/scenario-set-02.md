# SAP-C02 Scenario Set 02

Identity / Security / Networking を中心にした長文シナリオ型問題集。選択肢は A〜G の7択。正解数は1〜3個。

---

## Question 1: SCP・Permission Boundary・委任管理

ある企業はAWS Organizationsで本番・検証・共通サービスのOUを分けており、各事業部のクラウド管理者にIAMロール作成を委任したい。ただし、事業部管理者がAdministratorAccess相当のロールや、KMSキー削除・CloudTrail停止・外部アカウントへの無制限AssumeRoleを作成できないようにしたい。監査部門は、全アカウントに共通の禁止操作を適用しつつ、事業部ごとに作成可能なロール権限の上限も管理したい。最も適切な設計はどれか。

A. OrganizationsのSCPでCloudTrail停止やKMSキー削除などの禁止操作を明示的にDenyし、事業部管理者が作成するIAMロールにはPermission Boundaryの付与を必須化する。

B. 各事業部管理者にAdministratorAccessを付与し、CloudTrailで操作を記録する。問題があれば後から手動で修正する。

C. Permission Boundaryだけを使い、OrganizationsのSCPは使わない。Permission Boundaryはアカウント全体のrootユーザーにも適用されるため、共通禁止操作も制御できる。

D. SCPで事業部管理者に必要な権限をAllowすれば、IAMポリシーがなくてもロール作成権限が付与される。

E. IAM Identity Centerのグループ名だけでKMSキー削除を禁止する。SCPやIAMポリシーは不要である。

F. CloudFormation StackSetsでIAMロールを配布し、事業部による追加ロール作成をすべて禁止する。委任要件は満たせないが最も運用負荷が低い。

G. Resource-based Policyを各IAMロールに設定すれば、Permission Boundaryと同じ上限管理ができる。

**正解: A**

### 判断軸

- SCPはOrganizations配下のアカウントに対して、IAM user/roleが利用できる最大権限を制限する。
- Permission Boundaryは、特定のIAM user/roleに対して、Identity-based Policyで付与できる権限の上限を定義する。
- SCPもPermission Boundaryも、それ単体では権限を付与しない。

### 構成図

```text
Organizations Root
   └─ OU: Production
        ├─ SCP: CloudTrail停止/KMS削除/外部AssumeRole無制限をDeny
        └─ Account
             └─ 事業部管理者Role
                  ├─ IAM Policy: ロール作成を許可
                  └─ Permission Boundary付与を必須化
                         └─ 作成されるRoleの最大権限を制限
```

### 誤答理由

Bは事後監査だけで予防統制になっていない。CはPermission Boundaryの適用範囲を広げすぎている。DはSCPを権限付与手段と誤解している。Fは安全だが、委任管理という要件を満たしていない。

関連: [IAM](../services/security/iam.md), [Domain 1](../sap-c02/domain1.md), [CloudTrail](../services/management/cloudtrail.md)

---

## Question 2: 外部SaaSからのプライベート接続と最小露出

ある企業はVPC内の決済APIを外部SaaS事業者から利用させたい。APIはNLBの背後にあり、インターネット公開は避けたい。VPC PeeringやTransit Gatewayで相手先ネットワーク全体と接続すると、不要な到達性が増えるため避けたい。SaaS事業者は自社AWSアカウントのVPCからプライベートに接続できればよく、通信先は決済APIに限定したい。最も適切な設計はどれか。

A. 自社VPCとSaaS事業者VPCをVPC Peeringで接続し、ルートテーブルで全サブネットを相互到達可能にする。

B. Transit Gatewayに両社VPCを接続し、すべてのルートを伝播する。

C. 自社側でNLBを背後にしたVPC Endpoint Serviceを作成し、SaaS事業者側はInterface Endpointで接続する。接続承認とEndpoint Policyで利用者と操作を制御する。

D. Direct Connect Gatewayを共有すれば、特定サービスだけを公開できるためPrivateLinkは不要である。

E. APIをInternet-facing ALBで公開し、Security GroupでSaaS事業者の送信元IPだけを許可する。

F. Gateway Endpointを作成し、SaaS事業者にエンドポイントIDを共有する。

G. CloudFrontをNLBの前段に配置し、キャッシュすればプライベート接続になる。

**正解: C**

### 判断軸

- PrivateLinkはサービス単位のプライベート接続に向く。
- VPC Peering/TGWはネットワーク間接続であり、到達範囲が広くなりやすい。
- Gateway EndpointはS3/DynamoDB向けであり、任意のNLB背後サービス公開には使わない。

### 構成図

```text
Provider Account                         Consumer Account
VPC-A                                    VPC-B
  NLB ── Endpoint Service  <──────>  Interface Endpoint
   │                                      │
Payment API                         SaaS Workload
```

### 誤答理由

A/Bは到達性が広すぎる。Eはインターネット公開になる。FはGateway Endpointの用途違い。GはCDNであり、VPC間のプライベートサービス公開ではない。

関連: [PrivateLink](../services/networking/privatelink.md), [ELB](../services/networking/elb.md), [Domain 1](../sap-c02/domain1.md)

---

## Question 3: CloudFront・OAC・WAF・Shieldの組み合わせ

あるメディア企業はS3上の動画サムネイルとALB背後のAPIを同一ドメインで配信している。世界中の利用者に低レイテンシで配信し、S3オブジェクトへの直接アクセスは禁止したい。APIにはSQLインジェクションやBot由来の高頻度アクセス対策を入れ、DDoS時にも運用チームが優先支援を受けられるようにしたい。TLS証明書はマネージドに更新したい。最も適切な組み合わせはどれか。

A. CloudFrontを前段に置き、S3 OriginにはOACを使う。API OriginはALBにし、AWS WAFをCloudFrontに関連付け、必要に応じてShield Advancedを有効化し、証明書はACMで管理する。

B. S3をStatic Website Hostingで公開し、バケットポリシーをPublic Readにする。WAFはS3バケットに直接関連付ける。

C. ALBとS3の両方を直接インターネット公開し、Route 53のWeighted RoutingだけでDDoSを吸収する。

D. Shield Standardを有効化すればWAFのルール設定やACM証明書管理は不要になる。

E. CloudFrontを使わず、Global AcceleratorだけでS3のオブジェクトキャッシュを行う。

F. S3 OriginにOACを使い、バケットポリシーでCloudFrontサービスプリンシパルからのアクセスのみ許可する。

G. WAFのRate-based RuleやManaged Rulesを使い、SQLインジェクションや高頻度アクセスを緩和する。

**正解: A, F, G**

### 判断軸

- CloudFrontは静的コンテンツ配信と複数Originルーティングに向く。
- OACはS3 Originへの直接アクセス抑止に使う。
- WAFはL7のWeb攻撃対策、ShieldはDDoS対策、ACMは証明書管理。

### 構成図

```text
Users
  │ HTTPS / ACM
CloudFront
  ├─ Behavior: /static/* ── S3 Origin + OAC
  └─ Behavior: /api/*    ── ALB ── API
       │
 AWS WAF: Managed Rules + Rate-based Rule
 Shield Advanced: 重要リソース保護
```

### 誤答理由

BはS3を公開してしまう。DはShieldとWAF/ACMの役割を混同している。EはGlobal AcceleratorをCDNキャッシュとして誤用している。

関連: [CloudFront](../services/networking/cloudfront.md), [WAF](../services/security/waf.md), [Shield](../services/security/shield.md), [ACM](../services/security/acm.md)

---

## Question 4: 中央集約型のVPC検査

ある企業は50以上のVPCをTransit Gatewayで接続しており、インターネット向け通信とVPC間通信の一部を中央の検査VPCへ集約したい。セキュリティ部門は、L3/L4だけでなく一部のstateful inspectionを行い、必要に応じてIDS/IPS系アプライアンスも挿入したい。各アプリケーションVPCに個別の検査装置を置くと運用負荷が高い。最も適切な設計はどれか。

A. 各VPCにNAT Gatewayだけを置き、Security Groupで全通信検査を実現する。

B. Transit Gatewayのルーティングで検査VPCを経由させ、AWS Network FirewallまたはGateway Load Balancer配下の検査アプライアンスへトラフィックを誘導する。

C. ALBを中央に置き、全TCP/UDP通信をALBで中継する。

D. CloudFrontを検査VPCに配置し、VPC間通信をすべてCloudFront経由にする。

E. VPC Flow Logsだけを有効化すれば、通信は自動的に遮断・修復される。

F. Network FirewallはVPC境界のstateless/stateful検査に使える。

G. Gateway Load Balancerはサードパーティ仮想アプライアンスを透過的に挿入する用途に向く。

**正解: B, F, G**

### 判断軸

- Transit Gatewayは多VPC接続とルート制御の中心になる。
- Network FirewallはAWSマネージドのネットワークファイアウォール。
- Gateway Load Balancerは仮想アプライアンス挿入のためのロードバランサ。

### 構成図

```text
App VPCs ── Transit Gateway ── Inspection VPC ── Internet / Shared Services
                                ├─ AWS Network Firewall
                                └─ GWLB ── IDS/IPS Appliances
```

### 誤答理由

AはNATとSGを過大評価している。CはALBのレイヤー違い。DはCloudFrontの用途違い。Eはログ取得と制御を混同している。

関連: [Transit Gateway](../services/networking/transitgateway.md), [Network Firewall](../services/security/network-firewall.md), [ELB](../services/networking/elb.md)

---

## Question 5: Cognito MFAとデータレイク権限制御

ある企業はCognito User Poolで顧客ポータルの認証を行い、MFA済みユーザーだけにQuickSightの埋め込みダッシュボードを表示している。データはS3データレイクにあり、Glue Data CatalogとLake Formationで部門別の列レベル権限を設定している。しかし一部のMFA済みユーザーが、本来見えない列まで表示できてしまう。設計を見直すうえで最も重要な確認事項はどれか。

A. Cognito User PoolでMFAを必須化すれば、Lake Formationの列レベル権限は自動的にすべての分析クエリへ伝播する。

B. QuickSightやAthenaが実際に使用する実行ロール/ユーザーに対して、Lake Formationのテーブル/列権限が期待どおり対応しているか確認する。

C. S3バケットに対して広いRead権限が付与され、Lake Formationを迂回できる経路が残っていないか確認する。

D. KMSで暗号化していれば列レベル制御は不要である。

E. Cognito Identity Poolが発行するAWS一時認証情報を使う場合、IAMロールとLake Formation権限の対応を確認する。

F. WAFのManaged Rulesを追加すれば、列レベル権限の誤設定は自動修正される。

G. QuickSightのデータセット、SPICE取り込み、権限更新タイミングを確認する。

**正解: B, C, E**

### 判断軸

- MFAは認証強度の条件であり、Lake Formation権限そのものではない。
- Lake Formationは、クエリを実行する主体に対して権限が正しく紐づいている必要がある。
- S3/IAM/KMS側に広すぎる直接アクセスがあると、意図したデータガバナンスを迂回する可能性がある。

### 構成図

```text
User ── Cognito User Pool(MFA) ── App
                                  │
                                  ▼
                           QuickSight / Athena
                                  │ 実行主体の権限が重要
                                  ▼
                    Lake Formation + Glue Data Catalog
                                  │
                             S3 + SSE-KMS
```

### 誤答理由

Aは認証とデータ認可を混同している。Dは暗号化と列レベル認可を混同している。FはWAFの用途違い。Gも重要な調査点だが、最も基本となる権限モデル確認としてはB/C/Eが中心。

関連: [Cognito](../services/security/cognito.md), [QuickSight](../services/analytics/quicksight.md), [Lake Formation](../services/analytics/lakeformation.md), [KMS](../services/security/kms.md)
