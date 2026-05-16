# AWS Endpoint / ENI / VIF 完全整理

## 何のためのページか

AWSでは **endpoint** という言葉が複数のレイヤーで使われる。Aurora endpoint、VPC endpoint、API endpoint、Interface endpoint、Endpoint Service、Direct Connect VIF は、名前が似ていても指しているものが違う。

このページは、SAP-C02で混同しやすい **Endpoint / ENI / VIF / Gateway / Interface** を横断して整理する。

---

## まず一言でいうと

```text
Endpoint = 接続先、または接続口
ENI      = VPC内の仮想ネットワークインターフェース
VIF      = Direct Connect上の論理インターフェース
```

ただし、AWSでは endpoint が次のように多義的に使われる。

```text
Endpoint
├─ DB接続先DNS
│   ├─ Aurora cluster/writer endpoint
│   ├─ Aurora reader endpoint
│   ├─ Aurora instance endpoint
│   └─ RDS endpoint
│
├─ VPC Endpoint
│   ├─ Gateway Endpoint
│   ├─ Interface Endpoint
│   └─ Gateway Load Balancer Endpoint
│
├─ PrivateLink
│   ├─ Interface Endpoint
│   └─ Endpoint Service
│
├─ API / Service endpoint
│   ├─ API Gateway endpoint
│   ├─ S3 endpoint
│   ├─ AWS service public endpoint
│   └─ CloudFront / ALB / NLB DNS name
│
└─ Hybrid connectivity周辺
    ├─ Direct Connect VIF
    ├─ VPN endpoint
    ├─ VGW
    └─ Transit Gateway attachment
```

---

## 1. Endpointとは何か

一般的にendpointとは、通信の終端、つまりクライアントやアプリケーションが接続する相手先を指す。

```text
Application → https://api.example.com
Application → aurora-cluster.cluster-xxxxx.ap-northeast-1.rds.amazonaws.com
EC2         → com.amazonaws.ap-northeast-1.ssm
On-prem     → Direct Connect VIF → AWS
```

重要なのは、**endpoint = VPC Endpoint ではない** という点。

AWSの問題文で endpoint と出たら、まず「何のendpointか」を確認する。

---

## 2. VPC Endpoint

VPC Endpointは、VPC内のリソースからAWSサービス、他アカウントのサービス、SaaSサービスへ、インターネットを通らずに接続するための入口。

```text
Private subnet resource
  → VPC Endpoint
  → AWS service / PrivateLink service
```

VPC Endpointには主に3種類ある。

| 種類 | 対象 | 実体 | 代表例 | 試験の罠 |
|---|---|---|---|---|
| Gateway Endpoint | S3 / DynamoDB | Route Table + Prefix List | EC2 → S3 | ENIではない。SGも付かない |
| Interface Endpoint | 多くのAWSサービス / PrivateLink | ENI + Private IP | KMS, SSM, Secrets Manager, ECR | ENIができる。SGで制御する |
| Gateway Load Balancer Endpoint | FW/IDS/IPS挿入 | GWLB経由のEndpoint | 中央検査VPC | 通常のInterface Endpointではない |

---

## 3. Gateway Endpoint

Gateway Endpointは、S3またはDynamoDBへVPC内からプライベートに接続するためのVPC Endpoint。

```text
Private Subnet EC2
  → Route Table
  → S3 Gateway Endpoint
  → Amazon S3
```

### 特徴

```text
Gateway Endpoint = Route Table型のVPC Endpoint
```

- S3 / DynamoDB向け
- ルートテーブルに宛先を追加する
- ENIは作らない
- Security Groupは付かない
- 追加の時間課金はない
- Endpoint Policyで、そのendpoint経由の操作を制限できる

### SAP-C02での読み方

```text
VPC内のEC2からS3/DynamoDBへNAT Gatewayなしでアクセスしたい
→ Gateway Endpoint
```

ただし、オンプレミスからS3へプライベート接続したい場合は、Gateway Endpointではなく、S3 Interface EndpointやDirect Connect Public VIFなど別の選択肢を検討する。

---

## 4. Interface Endpoint

Interface Endpointは、VPC内にENIを作り、そのPrivate IP経由でAWSサービスやPrivateLinkサービスに接続するVPC Endpoint。

```text
Private Subnet EC2
  → Interface Endpoint ENI
  → AWS service / Endpoint Service
```

### 特徴

```text
Interface Endpoint = ENI型のVPC Endpoint
```

- Subnet内にENIが作成される
- Private IPを持つ
- Security Groupを設定できる
- Private DNSを有効化できる
- 対応サービスでEndpoint Policyを設定できる
- 時間課金とデータ処理課金がある

### 代表サービス

```text
KMS
Secrets Manager
SSM
CloudWatch Logs
ECR
STS
SNS
SQS
EventBridge
```

### SAP-C02での読み方

```text
Private subnetからAWS APIへNAT Gatewayなしで接続したい
→ Interface Endpoint
```

例えば、Private subnetのEC2やLambdaがKMS、SSM、Secrets Manager、ECRにアクセスするなら、Interface Endpointが候補になる。

---

## 5. PrivateLink / Endpoint Service

PrivateLinkは、VPC全体をつなぐ仕組みではなく、**サービス単位でプライベート接続を提供/利用する仕組み**。

```text
Provider VPC
  NLB
   ↓
Endpoint Service
   ↓ PrivateLink
Consumer VPC
  Interface Endpoint ENI
   ↓
Client
```

### 用語の分解

| 用語 | 立場 | 意味 |
|---|---|---|
| PrivateLink | 仕組み | サービス単位のプライベート接続モデル |
| Endpoint Service | 提供側 | NLB背後のサービスをPrivateLinkで公開する設定 |
| Interface Endpoint | 利用側 | Consumer VPCに作る接続口 |
| ENI | 利用側VPC内の実体 | Private IPを持つ仮想NIC |

### PrivateLinkが向く要件

- SaaSを顧客VPCへプライベート公開したい
- 共有サービスを複数VPCから利用させたい
- VPC PeeringやTransit Gatewayのような広い到達性を与えたくない
- CIDR重複がある環境でサービス単位に接続したい

### PrivateLinkが向かない要件

- VPC全体を双方向に通信させたい
- 多数VPCをハブ&スポークで接続したい
- ルーティング制御や広いネットワーク到達性が必要

その場合は、VPC PeeringやTransit Gatewayを検討する。

---

## 6. ENIとは何か

ENIは **Elastic Network Interface**。VPC内の仮想ネットワークインターフェース。

```text
ENI = VPC内の仮想NIC
```

ENIはendpointそのものとは限らない。EC2、RDS、Lambda in VPC、NAT Gateway、Interface Endpointなど、多くのVPCリソースがENIを使ってネットワークに接続する。

### ENIが持つもの

- Private IP
- Security Group
- MAC address
- Subnet
- AZ
- 必要に応じてPublic IP / Elastic IP

### ENIが関係する代表例

```text
EC2 instance
Lambda in VPC
RDS / Aurora
EFS mount target
NAT Gateway
Interface Endpoint
Transit Gateway attachment周辺
```

### Interface EndpointとENIの関係

```text
Interface Endpointを作る
  → 指定SubnetにENIが作られる
  → ENIがPrivate IPを持つ
  → Security Groupで到達制御する
  → DNS名がPrivate IPへ解決される
```

つまり、**Interface EndpointはENIを使ったサービス接続口**。

---

## 7. Endpoint Policy

VPC EndpointにはEndpoint Policyを設定できる場合がある。

Endpoint Policyは、そのendpoint経由で許可する操作を制限する追加ガード。

```text
IAM Policy
Resource Policy
Endpoint Policy
KMS Key Policy
SCP
Permission Boundary
  → これらを総合して最終的な許可/拒否が決まる
```

Endpoint Policyだけで権限が決まるわけではない。

### S3でよく出る条件

```json
"Condition": {
  "StringEquals": {
    "aws:SourceVpce": "vpce-xxxxxxxx"
  }
}
```

これは、特定のVPC Endpoint経由のアクセスだけを許可する条件。

### 試験の罠

- Endpoint Policyで許可していても、IAMやS3 bucket policyで拒否されれば失敗する
- KMS暗号化オブジェクトなら、KMS key policy / IAM / grantsも必要になる
- S3 bucket policyの `aws:SourceVpce` は、特定endpoint経由制限に使う

---

## 8. Aurora / RDS Endpoint

DB系のendpointは、VPC Endpointとは別物。これはDBに接続するためのDNS名。

```text
Application
  → Aurora / RDS endpoint DNS
  → DB cluster / DB instance
```

### Aurora endpointの種類

| Endpoint | 用途 | 実体 | 試験の罠 |
|---|---|---|---|
| Cluster / Writer endpoint | 書き込み、通常のread/write | 現在のwriterを指すDNS | SELECT負荷まで全部writerに寄せる |
| Reader endpoint | read-only負荷分散 | Aurora Replica群を指すDNS | 更新直後の強い整合性を期待する |
| Instance endpoint | 特定DB instanceへ接続 | 個別インスタンスDNS | 本番アプリを固定接続してfailover耐性を下げる |
| Custom endpoint | 特定reader群へ接続 | reader subset | 何でも自動最適化する魔法ではない |

### 覚え方

```text
Aurora endpoint = DB接続先DNS
VPC endpoint    = AWSサービスへのプライベート接続口
```

### Reader endpointの注意

Reader endpointは読み取りをAurora Replica群へ分散するが、writer直後の最新データを常に即座に読めることを保証するものではない。強いread-after-writeが必要なら、writer endpointを使うか、アプリ側で読み取り経路を分ける。

---

## 9. API / Service Endpoint

API endpointやservice endpointは、サービスの公開URLやDNS名を指す。

```text
Client → API Gateway endpoint
Client → CloudFront distribution domain
Client → ALB DNS name
Client → S3 regional endpoint
Client → AWS public service endpoint
```

### 代表例

| Endpoint | 意味 |
|---|---|
| API Gateway endpoint | APIの公開URL |
| CloudFront domain | CDN配信の公開入口 |
| ALB/NLB DNS name | Load Balancerの接続先 |
| S3 REST endpoint | S3 API操作用のendpoint |
| S3 website endpoint | 静的Webサイトホスティング用endpoint |
| AWS service public endpoint | KMS/STS/S3などAWS APIのpublic endpoint |

### S3 REST endpointとWebsite endpointの違い

| 種類 | 用途 | CloudFront OAC | HTTPS | 認証 |
|---|---|---|---|---|
| S3 REST endpoint | S3 API / private bucket配信 | 使える | CloudFront側で対応 | IAM / bucket policy |
| S3 website endpoint | 静的Webサイトホスティング | OAC不可 | S3 website endpoint自体はHTTP | 公開Web用途中心 |

SAP-C02では、CloudFrontでprivate S3 originを守るなら、S3 REST endpoint + OACを読む。S3 website endpointは公開Webサイトホスティング寄りで、OACとは組み合わせない。

---

## 10. Direct Connect VIF

VIFは **Virtual Interface**。Direct Connect接続上に作る論理インターフェース。

```text
On-premises router
  → Direct Connect connection
  → VIF
  → DX Gateway / VGW / TGW / AWS public services
```

VIFはendpointというより、Direct Connectでどこへ到達するかを決める論理的な接続口。

### VIFの種類

| VIF | 接続先 | 用途 |
|---|---|---|
| Private VIF | VPC / VGW / DX Gateway | オンプレからVPCへPrivate IPで接続 |
| Public VIF | AWS public services | S3などAWS public endpointへ専用線経由で接続 |
| Transit VIF | DX Gateway + Transit Gateway | 多数VPC/多数アカウントへの大規模接続 |

### VIFとEndpointの違い

```text
Endpoint = 接続先、または接続口
VIF      = Direct Connect上の論理インターフェース
```

Public VIFはインターネット接続ではない。AWS public servicesへDirect Connect経由で到達するためのVIF。

---

## 11. Gateway / Endpoint / Interfaceの違い

| 用語 | 基本意味 | AWSでの代表例 |
|---|---|---|
| Gateway | 経路の次ホップ、中継点 | IGW, NAT Gateway, Transit Gateway, DX Gateway |
| Endpoint | 接続先、接続口 | VPC Endpoint, Aurora endpoint, API endpoint |
| Interface | ネットワークインターフェース | ENI, Direct Connect VIF |

### GatewayとEndpointの違い

```text
Gateway  = どこへ流すかを決める中継点
Endpoint = どこへ接続するかの入口/宛先
```

ただし、Gateway Endpointのように名前が混ざるサービスがある。Gateway EndpointはGatewayという名前だが、VPC Endpointの一種。

---

## 12. 用語対応表

| 用語 | 何か | どこにあるか | 何をするか |
|---|---|---|---|
| Endpoint | 接続先/接続口 | サービス、VPC、DB、APIなど | クライアントが通信する相手 |
| VPC Endpoint | AWSサービスへのprivate接続口 | VPC内 | NAT/Internetを避けてAWSサービスへ接続 |
| Gateway Endpoint | Route Table型VPC Endpoint | Route Table | S3/DynamoDBへの経路を追加 |
| Interface Endpoint | ENI型VPC Endpoint | Subnet内 | Private IPでAWSサービス/PrivateLinkへ接続 |
| Endpoint Service | PrivateLinkの提供側 | Provider VPC | NLB背後のサービスを公開 |
| ENI | 仮想NIC | Subnet/AZ内 | IP/SGを持つネットワーク接続点 |
| VIF | DXの論理IF | Direct Connect接続上 | オンプレとAWSの経路を張る |
| VGW | VPC側のVPN/DX終端 | VPC側 | Site-to-Site VPN / DXの接続先 |
| TGW attachment | TGWへの接続 | TGW側 | VPC/VPN/DXをTGWへ接続 |

---

## 13. SAP-C02判断フロー

```text
そのendpointは何の接続先か?

DBに接続するDNS名?
  → RDS / Aurora endpoint

AWS APIへPrivate subnetから接続?
  → VPC Endpoint

S3 / DynamoDBだけ?
  → Gateway Endpoint

KMS / SSM / Secrets Manager / ECRなど?
  → Interface Endpoint

SaaSや別アカウントのサービスをPrivateに使う?
  → PrivateLink + Interface Endpoint

オンプレからAWSへ専用線?
  → Direct Connect + VIF

多数VPCをハブ接続?
  → Transit Gateway

サービス単位で公開?
  → PrivateLink / Endpoint Service

VPC全体を相互接続?
  → Transit Gateway / VPC Peering
```

---

## 14. よくある誤答パターン

| 誤答 | なぜ危険か |
|---|---|
| Aurora endpointとVPC endpointを同じものとして扱う | Aurora endpointはDB接続DNS、VPC endpointはAWSサービスへのprivate接続口 |
| Interface EndpointとENIを同義にする | Interface EndpointはENIを作るが、ENIはより一般的な仮想NIC |
| Gateway EndpointにSecurity Groupを付けようとする | Gateway EndpointはENI型ではないためSGは付かない |
| S3 Gateway Endpointをオンプレから使う | Gateway EndpointはVPC内ルートテーブル向け。オンプレ用途は別設計 |
| PrivateLinkでVPC全体を相互接続する | PrivateLinkはサービス単位。広い到達性はTGW/Peering |
| Direct Connectだけで暗号化されると考える | DX自体はIPsec暗号化を提供しない。要件次第でVPN over DXやMACsec等を検討 |
| Public VIFを一般インターネット接続と考える | Public VIFはAWS public servicesへのDX経由接続 |
| Reader endpointを強整合read用に使う | Reader endpointはread分散。更新直後の強いread-after-writeは注意 |
| RDS Proxyをread scalingやクエリキャッシュとして使う | RDS Proxyは接続プール。read routingやcacheではない |
| Endpoint Policyだけで全権限を制御できると考える | IAM/resource policy/KMS/SCP等も評価される |

---

## 15. 最短暗記

```text
Endpoint = 接続先
ENI = VPC内の仮想NIC
Interface Endpoint = ENIを使うVPC Endpoint
Gateway Endpoint = Route Tableを使うS3/DynamoDB用Endpoint
Endpoint Service = PrivateLinkの提供側
VIF = Direct Connect上の論理インターフェース
Aurora Endpoint = DB接続用DNS
```

特にこの3つを分ける。

```text
Aurora endpoint       = DB接続先DNS
VPC endpoint          = AWSサービスへのprivate接続口
Direct Connect VIF    = オンプレ接続の論理インターフェース
```

---

## 関連ページ

- [Amazon VPC](../services/networking/vpc.md)
- [AWS PrivateLink / VPC Endpoints](../services/networking/privatelink.md)
- [AWS Direct Connect](../services/networking/direct-connect.md)
- [Amazon Aurora](../services/database/aurora.md)
- [Networking Foundations Deep Dive](networking-foundations-deep-dive.md)
- [AWS Gateway Services and Terms](aws-gateways.md)
- [Networking Connectivity Options](networking-options.md)

## Official Docs

- VPC endpoints: https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html
- Gateway endpoints: https://docs.aws.amazon.com/vpc/latest/privatelink/gateway-endpoints.html
- Interface endpoints: https://docs.aws.amazon.com/vpc/latest/privatelink/interface-endpoints.html
- AWS PrivateLink concepts: https://docs.aws.amazon.com/vpc/latest/privatelink/concepts.html
- Direct Connect virtual interfaces: https://docs.aws.amazon.com/directconnect/latest/UserGuide/WorkingWithVirtualInterfaces.html
- Aurora endpoints: https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Overview.Endpoints.html
