# Networking Foundations Deep Dive

SAP-C02のネットワーク問題は、サービス名の暗記ではなく、**通信がどこからどこへ、どの経路で、どの境界を通り、どこで許可/拒否されるか**を読む試験。SG、NACL、Route Table、NAT Gateway、PrivateLinkなどは、名前が似ていても担当レイヤーが違う。

## まず全体像

```text
Client / AWS service / On-prem
  → DNS resolves name
  → Route Table chooses next hop
  → Gateway / Endpoint / Peering / TGW carries traffic
  → NACL checks subnet boundary
  → Security Group checks ENI/resource boundary
  → OS / application / IAM / resource policy may still deny
```

この順番は「毎回この順にAWSが評価する」という丸暗記ではなく、問題文を読むための地図。たとえば「S3に到達できない」なら、ネットワーク経路、S3 bucket policy、KMS key policyのどれで詰まっているかを分ける。

## 読むときの4点セット

| 観点 | 質問 | 例 |
|---|---|---|
| 誰が | 通信の起点は誰か | Internet user, EC2, Lambda, on-prem router |
| 何に | 宛先は何か | ALB, RDS, S3, another VPC, SaaS |
| どの権限で | IAM/Resource policy/KMSは関係するか | S3/KMS/AWS APIなら関係する |
| どの経路で | Packetはどこを通るか | IGW, NAT GW, TGW, VPC endpoint, DX/VPN |

ネットワーク制御は「道」と「門番」を分けると理解しやすい。

| 役割 | 代表 | これは何をするか | これは何ではないか |
|---|---|---|---|
| 道案内 | Route Table | 宛先CIDRに対する次ホップを選ぶ | 許可/拒否の判定ではない |
| 門番 | Security Group | ENI単位でAllowする | 明示Denyはできない |
| 改札 | Network ACL | Subnet単位でAllow/Denyする | アプリ単位の認可ではない |
| 出入口 | IGW/NAT/VGW/TGW/Endpoint | ネットワーク同士をつなぐ | IAM権限を付与しない |
| 検査所 | Network Firewall/GWLB/WAF | 通信内容やパターンを検査する | ルーティングだけの部品ではない |

## SG / NACL / Route Table

### Security Group

- 一言でいうと: ENIやRDS、ALBなどの「リソースの門番」。
- 何のためにあるか: リソース単位で必要な通信だけを許可するため。
- 何が嬉しいか: App SGからDB SGへ、のようにCIDRではなく役割で通信元を指定できる。
- 何と混同しやすいか: NACL。SGはstatefulでAllowのみ、NACLはstatelessでAllow/Deny。
- 試験問題ではどう出るか: `allow only ALB to reach application`, `DB accepts traffic only from app tier`。
- 間違えやすい選択肢: SGで特定IPを明示Denyする、戻り通信のephemeral portをSGに追加する。
- 小さな構成図:

```text
Internet
  → ALB SG: inbound 443 from 0.0.0.0/0
  → App SG: inbound 8080 from ALB SG
  → DB  SG: inbound 5432 from App SG
```

- 暗記のコツ、語源、語呂: **SG = Stateful Gate**。許した通信の帰り道は覚えている。

### Network ACL

- 一言でいうと: Subnetの「番号つき改札」。
- 何のためにあるか: サブネット境界で広めのAllow/Denyをかけるため。
- 何が嬉しいか: 特定CIDRを明示的に拒否できる。SGではできないDenyができる。
- 何と混同しやすいか: SG。NACLは戻り通信を覚えない。
- 試験問題ではどう出るか: `block known malicious IP at subnet level`, `stateless rules`, `rule number order`。
- 間違えやすい選択肢: Inboundだけ許可してOutboundの戻り通信を忘れる。
- 小さな構成図:

```text
Public subnet NACL
  inbound  100 ALLOW 443 from 0.0.0.0/0
  inbound  110 ALLOW ephemeral from 0.0.0.0/0
  outbound 100 ALLOW 443 to   0.0.0.0/0
  outbound 110 ALLOW ephemeral to 0.0.0.0/0
```

- 暗記のコツ、語源、語呂: **NACL = Numbered ACL**。番号順、No memory、Denyあり。

### Inbound / Outbound / Ephemeral Port

Inbound/Outboundは「世界全体から見て入る/出る」ではなく、**そのリソースやサブネットから見て入る/出る**という意味。

```text
Client:49152  →  Server:443
Client:49152  ←  Server:443

Server側:
  inbound  = Clientから443へ来る通信
  outbound = Clientの49152へ戻る通信

Client側:
  outbound = Serverの443へ出る通信
  inbound  = Serverから49152へ戻る通信
```

Ephemeral portは、クライアントが一時的に使う戻り用の高番ポート。SGはstatefulなので戻りを自動許可する。NACLはstatelessなので、戻り方向のephemeral portも許可する。

### Route Table

- 一言でいうと: 宛先CIDRに対する「次の行き先表」。
- 何のためにあるか: `0.0.0.0/0 → IGW`、`10.0.0.0/8 → TGW`のように道を選ぶため。
- 何が嬉しいか: Public/private subnet、VPC endpoint経由、TGW経由などを分離できる。
- 何と混同しやすいか: Firewall。Route Tableは拒否制御ではない。
- 試験問題ではどう出るか: `private subnet cannot reach internet`, `route to NAT gateway`, `route propagation`。
- 間違えやすい選択肢: Route Tableだけで通信を許可できる、または拒否できると考える。
- 小さな構成図:

```text
Public subnet route:
  0.0.0.0/0 → Internet Gateway

Private subnet route:
  0.0.0.0/0 → NAT Gateway
  pl-s3      → Gateway Endpoint
```

- 暗記のコツ、語源、語呂: **Routeは地図**。地図は通行許可証ではない。

## Gateways and Endpoints

### Internet Gateway

- 一言でいうと: VPCをインターネットへ出入りさせる門。
- 何のためにあるか: Public IPを持つリソースがインターネットと通信するため。
- 何が嬉しいか: ALB、public EC2、NAT Gatewayを外部へ接続できる。
- 何と混同しやすいか: NAT Gateway。IGWはpublic subnetの双方向、NATはprivate subnetの外向き。
- 試験問題ではどう出るか: `internet-facing ALB`, `public subnet`, `public IP`, `0.0.0.0/0 to IGW`。
- 間違えやすい選択肢: IGWを付けただけでEC2が公開される。Public IP、route、SG/NACLも必要。
- 小さな構成図:

```text
Internet
  ↔ IGW
    ↔ Public subnet route 0.0.0.0/0
      ↔ ALB / EC2 with public IP
```

- 暗記のコツ、語源、語呂: **Internet Gate Way**。外の道に出る門。

### NAT Gateway

- 一言でいうと: Private subnetから外へ出るための代理出口。
- 何のためにあるか: Private EC2がパッチ取得や外部API呼び出しをするため。
- 何が嬉しいか: Private resourceにpublic IPを付けずに外向き通信ができる。
- 何と混同しやすいか: IGW、PrivateLink。NATは外向きインターネット、PrivateLinkはAWS内private接続。
- 試験問題ではどう出るか: `private instances need outbound internet access`, `software updates`, `no inbound from internet`。
- 間違えやすい選択肢: インターネットからPrivate EC2へ入る入口としてNAT Gatewayを選ぶ。
- 小さな構成図:

```text
Private EC2
  → private route 0.0.0.0/0
  → NAT Gateway in public subnet
  → IGW
  → Internet
```

- 暗記のコツ、語源、語呂: **NATは中から外**。外から中の新規接続は入れない。

### Transit Gateway

- 一言でいうと: 多数VPCとオンプレを集約するネットワークハブ。
- 何のためにあるか: VPC Peeringのメッシュ地獄を避けるため。
- 何が嬉しいか: ルートテーブルでドメイン別/環境別に接続を分離できる。
- 何と混同しやすいか: VPC Peering、DX Gateway。TGWはVPC/オンプレ接続の中心ルータ。
- 試験問題ではどう出るか: `hundreds of VPCs`, `hub-and-spoke`, `centralized inspection`, `multi-account`。
- 間違えやすい選択肢: 大規模環境でVPC Peeringを大量に張る。
- 小さな構成図:

```text
VPC-A ─┐
VPC-B ─┼→ Transit Gateway → VPN / Direct Connect → On-prem
VPC-C ─┘
```

- 暗記のコツ、語源、語呂: **Transit = 乗り換え**。多数の経路が集まる駅。

### VGW / CGW / DXGW

| 用語 | 一言でいうと | 誰側か | 試験の読み方 |
|---|---|---|---|
| VGW | VPC側のVPN/DX終端 | AWS側 | 1つのVPCをオンプレにつなぐ昔ながらの終端 |
| CGW | オンプレ側VPN装置の定義 | 顧客側 | 物理/仮想ルータのIPやBGP情報 |
| DXGW | Direct Connectを複数VPC/TGWに広げる中継 | AWS側 | DXをリージョン/VPC横断で使う |

```text
On-prem router
  = Customer Gateway
      │ IPsec VPN / Direct Connect
AWS side
  = VGW for single VPC
  = TGW for many VPCs
  = DXGW for Direct Connect association
```

罠は、DXGWをVPC同士のルータだと思うこと。VPC間のハブはTGW、Direct Connectの関連付けを広げるのがDXGW。

### Gateway Endpoint

- 一言でいうと: S3/DynamoDBへprivateに到達するroute tableの近道。
- 何のためにあるか: NAT GatewayやIGWを通さず、S3/DynamoDBへ到達するため。
- 何が嬉しいか: NATコスト削減、private subnetからのAWSサービスアクセス。
- 何と混同しやすいか: Interface Endpoint。Gateway EndpointはS3/DynamoDB向け、route tableで使う。
- 試験問題ではどう出るか: `private subnet access to S3 without NAT`, `reduce NAT Gateway cost`。
- 間違えやすい選択肢: KMS、SSM、Secrets Manager用にGateway Endpointを選ぶ。
- 小さな構成図:

```text
Private subnet route table
  Destination: S3 prefix list
  Target: Gateway Endpoint
```

- 暗記のコツ、語源、語呂: **Gateway EndpointはS3とDynamoDBの2枚切符**。

### Interface Endpoint / AWS PrivateLink

- 一言でいうと: VPC内にprivate IP付きENIを作り、AWSサービスや自社サービスへprivate接続する仕組み。
- 何のためにあるか: NAT/IGW/public IPなしでサービス単位に接続するため。
- 何が嬉しいか: KMS、SSM、Secrets Manager、ECR、CloudWatch Logsなどへprivateに到達できる。自社NLBサービスも他VPC/他アカウントへprivate公開できる。
- 何と混同しやすいか: VPC Peering/TGW。PrivateLinkはVPC全体接続ではなく、サービス単位接続。
- 試験問題ではどう出るか: `privately expose service to other accounts`, `no overlapping CIDR issue`, `private access to AWS APIs`。
- 間違えやすい選択肢: 2つのVPCをフルに相互通信させる目的でPrivateLinkを選ぶ。
- 小さな構成図:

```text
Consumer VPC
  App → Interface Endpoint ENI
           │ PrivateLink
Provider VPC
         Endpoint Service → NLB → Service
```

- 暗記のコツ、語源、語呂: **Interface = ENIのI**。private IPの入口ができる。

### Gateway Load Balancer

- 一言でいうと: Firewall/IDS/IPSなどの仮想アプライアンスを透過的に挿入するロードバランサー。
- 何のためにあるか: 中央検査VPCで大量通信をセキュリティ装置へ分散するため。
- 何が嬉しいか: アプライアンスのスケール、冗長化、経路挿入を設計しやすい。
- 何と混同しやすいか: ALB/NLB。GWLBはWebアプリのL7ルーティングではない。
- 試験問題ではどう出るか: `third-party firewall appliances`, `centralized inspection`, `bump-in-the-wire`。
- 間違えやすい選択肢: ALBの代わりにGWLBでpath-based routingをする。
- 小さな構成図:

```text
Workload VPC route
  → GWLB Endpoint
  → Gateway Load Balancer
  → Firewall appliances
  → destination
```

- 暗記のコツ、語源、語呂: **GWLBはアプライアンスを挟むLB**。

### Global Accelerator Static Anycast IP

- 一言でいうと: 世界中のAWSエッジから広告される固定IP入口。
- 何のためにあるか: クライアントに固定IPを見せたまま、最寄りエッジからAWSグローバルネットワークへ入れるため。
- 何が嬉しいか: IP許可リスト、リージョン切替、高速フェイルオーバー、TCP/UDPアプリで使いやすい。
- 何と混同しやすいか: Route 53 DNS failover、CloudFront。Global Acceleratorはキャッシュしない。DNS TTLに依存した切替でもない。
- 試験問題ではどう出るか: `static IP`, `Anycast`, `low latency global users`, `fast regional failover`, `TCP/UDP`。
- 間違えやすい選択肢: 静的コンテンツ配信やHTTPキャッシュ目的でGlobal Acceleratorを選ぶ。
- 小さな構成図:

```text
Users
  → same static Anycast IPs
  → nearest AWS edge
  → AWS global network
  → healthy ALB/NLB/EC2 endpoint in Region
```

- 暗記のコツ、語源、語呂: **Anycast = 同じ宛先を多地点から広告**。DNS名を返し分けるのではなく、IP入口自体がグローバル。

DNS failoverとの違い:

| 観点 | Global Accelerator | Route 53 DNS failover |
|---|---|---|
| 固定IP | あり | 基本はDNS名 |
| 切替単位 | エッジ/アクセラレータが健康状態で誘導 | DNS応答 |
| TTL影響 | 小さい | クライアント/Resolverのキャッシュ影響あり |
| キャッシュ | しない | しない |
| 向く要件 | 固定IP、TCP/UDP、速いリージョン退避 | DNSベースの柔軟な名前解決 |

## 試験で選択肢を切る判断軸

| 問題文の制約語 | 読み替え | 選びやすい候補 |
|---|---|---|
| resource-level allow only | ENI単位の許可 | Security Group |
| explicit deny / subnet level | サブネット境界の拒否 | NACL |
| route / next hop / prefix list | 道案内 | Route Table / Gateway Endpoint |
| private subnet outbound internet | 外向き代理 | NAT Gateway |
| many VPCs / hub-and-spoke | 集約ルータ | Transit Gateway |
| private service exposure | サービス単位の公開 | PrivateLink |
| private access to S3/DynamoDB | NAT不要の近道 | Gateway Endpoint |
| private access to KMS/SSM/Secrets | ENI型endpoint | Interface Endpoint |
| third-party firewall insertion | アプライアンス挿入 | GWLB |
| static anycast IP / TCP UDP global | 固定グローバル入口 | Global Accelerator |

## SAP-C02での読み方

1. まず「通信の起点」と「宛先」を固定する。
2. 次に「AWS APIやS3/KMSの権限問題」か「IPパケットの経路問題」かを分ける。
3. 経路問題なら、Route Table、Gateway/Endpoint、NACL、SGの順に境界を洗う。
4. 選択肢にDenyが出たらSGを疑う。SGはAllowだけ。
5. 「Private」と書かれていても、Private subnet、PrivateLink、private hosted zone、private IPは別概念として読む。

## このページを読んだあとに戻るべき関連ページ

- [Security Group / NACL / Firewall Decision Guide](network-security-boundaries.md)
- [AWS Gateway Services and Terms](aws-gateways.md)
- [Networking Connectivity Options](networking-options.md)
- [Amazon VPC](../services/networking/vpc.md)
- [AWS PrivateLink](../services/networking/privatelink.md)
- [AWS Global Accelerator](../services/networking/global-accelerator.md)
- [Access Control and Encryption Map](access-control-and-encryption.md)

