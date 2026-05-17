# Endpoint / ENI / VIF / PrivateLink 理解補完レジュメ

## 何のためのレジュメか

このレジュメは、Endpoint / ENI / VIF / PrivateLink 周りで混同しやすい論点を、SAP-C02で解ける形に再整理するための補助教材。

特に補強する対象は次の4点。

```text
1. Gateway Endpoint と Interface Endpoint の実体差
2. PrivateLink は VPC全体接続ではなく「サービス単位接続」であること
3. Direct Connect VIF の3種類
4. Endpoint / ENI / VIF のレイヤー差
```

---

## 0. ゴール

この一文を説明できるようになること。

```text
Endpointは接続先。
ENIはVPC内の仮想NIC。
VIFはDirect Connect上の論理インターフェース。
Gateway EndpointはRoute Table型。
Interface EndpointはENI型。
PrivateLinkはサービス単位のprivate接続。
```

---

## 1. Endpoint / ENI / VIF の違い

| 用語 | 一言 | どこにあるか | 例 |
|---|---|---|---|
| Endpoint | 接続先・接続口 | サービス、DB、VPCなど | Aurora endpoint, VPC endpoint |
| ENI | 仮想NIC | VPCのSubnet内 | EC2のNIC, Interface EndpointのNIC |
| VIF | DXの論理インターフェース | Direct Connect接続上 | Private VIF, Public VIF, Transit VIF |

覚え方。

```text
Endpoint = どこへ接続するか
ENI      = VPC内で何を使って接続するか
VIF      = Direct Connect上でどの論理口を使うか
```

---

## 2. Gateway Endpoint と Interface Endpoint

最重要の一文。

```text
Gateway Endpoint = Route Table型
Interface Endpoint = ENI型
```

### Gateway Endpoint

Gateway Endpointは、S3 / DynamoDB へVPC内からprivateに接続するためのVPC Endpoint。

```text
Private Subnet EC2
  → Route Table
  → Gateway Endpoint
  → S3 / DynamoDB
```

特徴。

```text
Gateway Endpoint:
  - S3 / DynamoDB向け
  - Route Tableに経路を追加する
  - ENIは作らない
  - Security Groupは付かない
  - NAT Gateway不要
```

誤解しやすい点。

```text
Gateway EndpointもENIを作るのでは？
```

これは誤り。正しくは次の通り。

```text
Gateway EndpointはRoute Table型。
ENIは作らない。
Security Groupも付かない。
```

### Interface Endpoint

Interface Endpointは、VPC内にENIを作り、そのPrivate IP経由でAWSサービスやPrivateLinkサービスへ接続するVPC Endpoint。

```text
Private Subnet EC2
  → Interface Endpoint ENI
  → KMS / SSM / Secrets Manager / ECR / SaaS
```

特徴。

```text
Interface Endpoint:
  - ENIを作る
  - Private IPを持つ
  - Security Groupを付けられる
  - Private DNSを使える
  - 多くのAWSサービスに対応
```

代表例。

```text
KMS
SSM
Secrets Manager
ECR
CloudWatch Logs
SNS
SQS
EventBridge
STS
```

### 見分け方

```text
S3 / DynamoDB
  → Gateway Endpoint

KMS / SSM / Secrets Manager / ECR など
  → Interface Endpoint
```

試験ではこう読む。

```text
Private subnetからS3へNATなし
  → Gateway Endpoint

Private subnetからKMS/SSM/Secrets ManagerへNATなし
  → Interface Endpoint
```

---

## 3. PrivateLink

最も大きな誤解は次の形。

```text
PrivateLink = VPC同士を丸ごとつなぐもの
```

これは誤り。

正しくは次の通り。

```text
PrivateLink = サービス単位でprivateに公開/利用する仕組み
```

VPC全体をつなぐものではない。

### PrivateLinkの構成

```text
Provider VPC
  Service
    ↓
  NLB
    ↓
  Endpoint Service

PrivateLink

Consumer VPC
  Interface Endpoint ENI
    ↓
  Client
```

### 提供側と利用側

| 立場 | 使うもの | 役割 |
|---|---|---|
| Provider側 | NLB + Endpoint Service | サービスをPrivateLinkで公開する |
| Consumer側 | Interface Endpoint | 公開されたサービスをprivateに利用する |

### API Gatewayは関係あるか

基本形では必須ではない。

PrivateLinkの基本は次の形。

```text
Provider側:
  NLB + Endpoint Service

Consumer側:
  Interface Endpoint
```

API Gatewayを使うパターンもあるが、PrivateLinkの中心概念ではない。試験でまず覚えるべきは **NLB + Endpoint Service + Interface Endpoint**。

### PrivateLinkを使う理由

PrivateLinkはこういうときに選ぶ。

```text
特定サービスだけ使わせたい
VPC全体の到達性は与えたくない
CIDR重複がある
SaaSを顧客VPCへprivateに提供したい
VPC PeeringやTransit Gatewayより狭く公開したい
```

逆に、VPC全体を相互接続したいなら次を考える。

```text
VPC Peering
Transit Gateway
```

---

## 4. ENI

ENIはVPC内の仮想NIC。

```text
ENI = Elastic Network Interface
ENI = VPC内の仮想NIC
```

ENIは、VPC内のリソースがネットワークに出るための部品。

代表例。

```text
EC2
Lambda in VPC
RDS / Aurora
NAT Gateway
Interface Endpoint
EFS mount target
```

### Endpointとの違い

```text
ENIはネットワーク部品。
Endpointは接続先・接続口。
```

ただし、Interface EndpointはENIを使う。

```text
Interface Endpoint
  → ENIを作る
  → Private IPを持つ
  → Security Groupで制御する
```

つまり次の関係。

```text
ENI = 部品
Interface Endpoint = ENIを使った接続口
```

---

## 5. Direct Connect VIF

VIFはDirect Connectで最も混同しやすい。

```text
VIF = Virtual Interface
VIF = Direct Connect上の論理インターフェース
```

構成イメージ。

```text
On-premises router
  → Direct Connect connection
  → VIF
  → AWS側の接続先
```

ENIとは違う。

```text
ENI = VPC内の仮想NIC
VIF = Direct Connect回線上の論理口
```

### VIFの3種類

| VIF | 一言 | 使う場面 |
|---|---|---|
| Private VIF | VPCへprivate接続 | オンプレから特定VPCへ接続 |
| Public VIF | AWS public servicesへDX経由接続 | オンプレからS3などへDX経由で接続 |
| Transit VIF | TGW経由で多数VPCへ接続 | 複数アカウント・複数VPC・大規模接続 |

### Public VIFはインターネットではない

誤解しやすい形。

```text
Public VIF = インターネット接続
```

これは誤り。

正しくは次の通り。

```text
Public VIF = AWS public servicesへDirect Connect経由で到達するためのVIF
```

例。

```text
On-prem
  → Direct Connect
  → Public VIF
  → S3 public endpoint
```

「public」とついているが、一般的なインターネット接続ではない。

### 試験での読み方

```text
オンプレから1つのVPCへprivate接続
  → Private VIF

オンプレからS3などAWS public servicesへDX経由接続
  → Public VIF

オンプレから多数VPC・多数アカウントへ拡張
  → Transit VIF + Direct Connect Gateway + Transit Gateway
```

---

## 6. Aurora Endpoint

Aurora endpointはVPC Endpointとは別物。DB接続先DNSとして読む。

### Aurora Endpointの種類

| Endpoint | 用途 |
|---|---|
| Cluster / Writer endpoint | 書き込み、通常のread/write |
| Reader endpoint | Aurora Replica群へのread-only分散 |
| Instance endpoint | 特定DB instanceへの接続 |
| Custom endpoint | 特定reader群への接続 |

### Reader endpointの注意

不正確な理解。

```text
Reader endpoint = レプリケーションへの接続点
```

より正確には次の通り。

```text
Reader endpoint = Aurora Replica群へのread-only接続を分散するDNS
```

重要な注意。

```text
Reader endpointは、writer直後の最新データを必ず即座に読める保証ではない。
```

したがって、次のような処理ではreader endpointへ逃がすと危険な場合がある。

```text
書き込み直後の確認
決済直後の状態確認
在庫更新直後の参照
```

---

## 7. RDS Proxy

RDS Proxyは接続プール。

```text
RDS Proxy = DB接続をプールして再利用するマネージドプロキシ
```

向いている場面。

```text
LambdaからDB接続が大量に発生する
DB接続数が枯渇しそう
failover時の接続管理を改善したい
```

### Reader endpointとの違い

```text
Reader endpoint = 読み取り分散
RDS Proxy       = 接続プール
```

RDS Proxyは次のものではない。

```text
read scaling
query cache
自動read routing
```

---

## 8. 試験での判断フロー

```text
そのendpointは何の接続先か？

DBに接続するDNS名？
  → Aurora / RDS endpoint

Private subnetからAWSサービスへNATなし？
  → VPC Endpoint

S3 / DynamoDB？
  → Gateway Endpoint

KMS / SSM / Secrets Manager / ECR？
  → Interface Endpoint

SaaSを顧客VPCへprivate公開？
  → PrivateLink
  → Provider: NLB + Endpoint Service
  → Consumer: Interface Endpoint

オンプレからAWSへ専用線？
  → Direct Connect

オンプレから多数VPCへ拡張？
  → Direct Connect + Transit VIF + DX Gateway + TGW
```

---

## 9. 誤答を潰すための対応表

| 問題文 | 選ぶもの | 選ばないもの |
|---|---|---|
| EC2からS3へNATなし | Gateway Endpoint | Interface Endpointではない場合が多い |
| EC2からKMSへNATなし | Interface Endpoint | Gateway Endpointではない |
| SaaSをprivate公開 | PrivateLink | VPC Peeringでは広すぎる |
| VPC全体を相互接続 | TGW / Peering | PrivateLinkではない |
| オンプレから多数VPCへ | DX + Transit VIF + TGW | Interface Endpointではない |
| Aurora読み取り分散 | Reader endpoint | RDS Proxyではない |
| LambdaのDB接続数対策 | RDS Proxy | Reader endpointではない |
| S3/DynamoDB用Endpoint | Gateway Endpoint | ENIは作らない |
| 多くのAWS API用Endpoint | Interface Endpoint | ENIを作る |

---

## 10. 暗記カード

### Card 1

```text
Gateway Endpoint = Route Table型
Interface Endpoint = ENI型
```

### Card 2

```text
Gateway Endpoint:
  S3 / DynamoDB
  ENIなし
  SGなし

Interface Endpoint:
  多くのAWSサービス
  ENIあり
  SGあり
```

### Card 3

```text
PrivateLink:
  VPC全体接続ではない
  サービス単位のprivate接続

Provider:
  NLB + Endpoint Service

Consumer:
  Interface Endpoint
```

### Card 4

```text
Private VIF:
  VPCへprivate接続

Public VIF:
  AWS public servicesへDX経由接続

Transit VIF:
  TGW経由で多数VPCへ接続
```

### Card 5

```text
Aurora endpoint:
  DB接続先DNS

VPC endpoint:
  AWSサービスへのprivate接続口

Direct Connect VIF:
  DX上の論理インターフェース
```

---

## 11. 一枚まとめ

```text
Endpoint = 接続先

Aurora endpoint
  → DB接続先DNS

VPC Endpoint
  → AWSサービスへのprivate接続口

Gateway Endpoint
  → Route Table型
  → S3 / DynamoDB
  → ENIなし

Interface Endpoint
  → ENI型
  → KMS / SSM / Secrets Managerなど
  → SGあり

PrivateLink
  → サービス単位のprivate接続
  → Provider: NLB + Endpoint Service
  → Consumer: Interface Endpoint

ENI
  → VPC内の仮想NIC

VIF
  → Direct Connect上の論理インターフェース

Transit VIF
  → 多数VPC接続に使う
```

---

## 12. 確認問題

### Q1

```text
Gateway Endpoint = 何型？
Interface Endpoint = 何型？
```

### Q2

```text
Gateway EndpointにENIは作られる？
Security Groupは付く？
```

### Q3

```text
PrivateLinkのProvider側とConsumer側に必要なものは？
```

### Q4

```text
Private VIF / Public VIF / Transit VIF の違いは？
```

### Q5

```text
PrivateLinkとTransit Gatewayの違いは？
```

---

## 関連ページ

- [AWS Endpoint / ENI / VIF 完全整理](endpoints-eni-vif.md)
- [Networking Foundations Deep Dive](networking-foundations-deep-dive.md)
- [AWS Gateway Services and Terms](aws-gateways.md)
- [Networking Connectivity Options](networking-options.md)
- [AWS PrivateLink / VPC Endpoints](../services/networking/privatelink.md)
- [AWS Direct Connect](../services/networking/direct-connect.md)
- [Amazon Aurora](../services/database/aurora.md)
