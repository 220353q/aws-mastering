# AWS Gateway Services and Terms

AWSには `Gateway` と名のつくものが多い。SAP-C02では「入口/出口/中継/変換/接続終端」のどれかを読むと混同しにくい。

## Gatewayの語源

`gate` は門、`way` は道。つまり `gateway` は **別のネットワーク、別のプロトコル、別の管理領域へ出入りする門**。

AWSでは次の意味で使われる。

| 意味 | 例 |
|---|---|
| インターネットへの門 | Internet Gateway, NAT Gateway |
| VPC/オンプレ接続の終端 | Virtual Private Gateway, Customer Gateway, Direct Connect Gateway |
| 多数ネットワークの中継 | Transit Gateway |
| サービス/APIへの入口 | API Gateway, Gateway Endpoint |
| アプライアンス挿入の入口 | Gateway Load Balancer |
| オンプレとAWSストレージの変換口 | Storage Gateway |

## Gateway系まとめ

| 名前 | 略称 | 一言 | よくある誤解 |
|---|---|---|---|
| Internet Gateway | IGW | VPCをインターネットへ接続する門 | 付けるだけでEC2が公開されるわけではない |
| NAT Gateway | NAT GW | Private subnetから外向き通信を出す | 外部からPrivate subnetへ入る入口ではない |
| Transit Gateway | TGW | 多数VPC/オンプレを集約するハブ | セキュリティ検査そのものではない |
| Virtual Private Gateway | VGW | Site-to-Site VPN/DXのVPC側終端 | 大規模ハブ用途はTGWが中心 |
| Customer Gateway | CGW | VPN接続のオンプレ側装置/定義 | AWS側リソースではあるが実体は顧客側装置情報 |
| Direct Connect Gateway | DXGW | DXを複数VPC/TGWへ接続するための中継 | VPC間トランジットルーターではない |
| API Gateway | APIGW | API公開、認証、スロットリング | VPCネットワークのGWではない |
| Gateway Endpoint | - | S3/DynamoDBへprivate接続するVPC Endpoint | 任意サービス公開には使えない |
| Gateway Load Balancer | GWLB | 仮想アプライアンスを透過挿入 | Web用ロードバランサではない |
| Storage Gateway | - | オンプレからAWSストレージを使う変換口 | データ移行専用ならDataSyncも読む |
| Route 53 Resolver Inbound/Outbound Endpoint | - | ハイブリッドDNSの入口/出口 | 通信経路ではなく名前解決の経路 |

## Internet Gateway

```text
Public Subnet
  EC2 public IP
    │
Route Table: 0.0.0.0/0 → IGW
    │
Internet
```

成立条件:
- VPCにIGWがアタッチされている
- Subnet route tableにIGWへのデフォルトルートがある
- リソースにPublic IPまたはElastic IPがある
- SG/NACLが許可している

## NAT Gateway

```text
Private Subnet EC2
  │ outbound only
  ▼
NAT Gateway in Public Subnet
  │
Internet Gateway
  │
Internet
```

ポイント:
- Private subnetからインターネットへ出るためのもの。
- インターネットからPrivate subnetへ新規Inbound接続する入口ではない。
- S3/DynamoDBやAWS API宛の通信はVPC EndpointでNATコストを下げられることがある。

## Transit Gateway / Direct Connect Gateway / VGW

```text
On-Prem
  │ DX / VPN
  ▼
DXGW / VGW / TGW
  │
VPCs
```

| 要件 | 選ぶ |
|---|---|
| 単一VPCとVPN/DX接続 | VGW |
| 多数VPC/オンプレのハブ | TGW |
| DXを複数リージョン/VPC/TGWへ関連付け | DXGW |

## Gateway Endpoint vs Interface Endpoint

| 項目 | Gateway Endpoint | Interface Endpoint |
|---|---|---|
| 対象 | S3 / DynamoDB | 多数のAWSサービス、PrivateLink |
| 仕組み | Route Tableにprefix list宛ルート | ENI + private IP |
| 料金 | 追加料金なしが基本 | 時間/データ処理課金 |
| SG | なし | Endpoint ENIにSG |
| よくある用途 | NAT Gatewayコスト削減 | KMS/SSM/Secrets/ECR/CloudWatch Logs等へprivate接続 |

覚え方:
- **Gateway Endpointは2つだけ**: S3 と DynamoDB。
- **InterfaceはENIのI**: ENIができる、SGが付く。

## API Gateway vs ALB vs CloudFront

| 要件 | 選ぶ |
|---|---|
| REST/HTTP/WebSocket API、認証、Usage Plan、Throttle | API Gateway |
| HTTP/HTTPSアプリのL7ロードバランス | ALB |
| エッジキャッシュ、OAC、グローバル配信 | CloudFront |

語源の罠:
- API GatewayのGatewayは「API利用者の入口」。
- Internet Gateway/NAT GatewayのGatewayは「ネットワークの出入口」。
- 同じGatewayでもレイヤーが違う。

## Gateway Load Balancer

```text
Traffic
  │
GWLB Endpoint
  │
Gateway Load Balancer
  │
Firewall / IDS / IPS appliances
```

選ぶ条件:
- サードパーティ仮想アプライアンスを透過的に挿入したい
- 中央検査VPCを作りたい
- 通常のWeb L7ロードバランサではない

## Storage Gateway

| タイプ | プロトコル/用途 | AWS側 |
|---|---|---|
| File Gateway | NFS/SMB | S3 / FSx |
| Volume Gateway | iSCSI block | EBS Snapshot |
| Tape Gateway | 仮想テープ | S3 Glacier系 |

選ぶ条件:
- オンプレアプリを変えずにAWSストレージを使いたい。
- ローカルキャッシュが必要。
- 単発移行/同期ならDataSyncと比較する。

## 暗記テクニック

```text
I-GW = Internetへ In/Out の門
NAT-GW = privateから外へ抜ける片道出口
T-GW = Transit、つまり乗り換えハブ
DX-GW = Direct Connectを広げる中継
VGW = VPC側VPN/DX終端
CGW = Customer側VPN装置
APIGW = APIの受付
GWLB = applianceを挟むロードバランサ
Storage GW = storage protocol変換
```

語呂:
- **NATは「中から外」**: 外から中の新規接続は入れない。
- **TGWは「たくさんGateway」**: 多数VPC/オンプレの乗換駅。
- **DXGWは「DXの分配器」**: VPC間ルーターではない。
- **Gateway Endpointは「S3とDynamoDBだけの近道」**。

試験テクニック:
1. `private subnet needs internet access for patching` → NAT Gateway。ただしAWSサービス宛ならEndpointも読む。
2. `many VPCs / hub-and-spoke / centralized inspection` → Transit Gateway + inspection VPC。
3. `private access to S3/DynamoDB without NAT` → Gateway Endpoint。
4. `private access to KMS/SSM/ECR/Secrets` → Interface Endpoint。
5. `expose own service privately to another VPC/account` → PrivateLink Endpoint Service。
6. `insert third-party firewall appliance` → Gateway Load Balancer。
7. `existing on-prem NFS/SMB/iSCSI/Tape continues` → Storage Gateway。

## Common Exam Traps

| 罠 | 正しい判断 |
|---|---|
| NAT Gatewayで外部からPrivate EC2へ接続 | NATはOutbound用。Inbound入口ではない |
| IGWを付けただけでEC2が公開される | Public IP、route、SG/NACLも必要 |
| DXGWをVPC間ルーターとして使う | VPC間接続はTGW/Peeringなど |
| Gateway EndpointでKMS/Secrets Managerへ接続 | KMS/SecretsはInterface Endpoint |
| Gateway Endpointで自社NLBサービスを公開 | 任意サービス公開はPrivateLink Endpoint Service |
| GWLBをALBの代わりにWebルーティングへ使う | GWLBはアプライアンス挿入 |
| Storage Gatewayを大規模一括移行専用に選ぶ | 一括/同期はDataSync、オフラインはSnow Familyも読む |

## Related

- [Networking Foundations Deep Dive](networking-foundations-deep-dive.md)
- [VPC](../services/networking/vpc.md)
- [PrivateLink](../services/networking/privatelink.md)
- [Transit Gateway](../services/networking/transitgateway.md)
- [Direct Connect](../services/networking/direct-connect.md)
- [ELB](../services/networking/elb.md)
- [Storage Gateway](../services/storage/storage-gateway.md)

## SAP-C02での読み方

Gatewayという単語だけで選ばない。IGWはインターネットの門、NAT Gatewayはprivate subnetの外向き出口、TGWは多数VPC/オンプレのハブ、Gateway EndpointはS3/DynamoDBのroute table近道、GWLBはアプライアンス挿入。KMS/SSM/Secrets ManagerなどはInterface Endpointとして読む。

## このページを読んだあとに戻るべき関連ページ

- [Networking Foundations Deep Dive](networking-foundations-deep-dive.md)
- [Networking Connectivity Options](networking-options.md)
- [Security Group / NACL / Firewall](network-security-boundaries.md)
