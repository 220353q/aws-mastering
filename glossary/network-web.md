# Network / Web / Security Terms

## Network Basics

| 用語 | 意味 | SAP-C02での判断 |
|---|---|---|
| CIDR | IPアドレス範囲の表記。例: `10.0.0.0/16` | VPC Peering/TGWではCIDR重複に注意。重複があるサービス公開はPrivateLinkが有利 |
| Subnet | VPC内のIP範囲。通常AZ単位で作る | Multi-AZ設計では複数AZにサブネットを作る |
| Route Table | 宛先CIDRごとの次ホップを定義 | IGW/NAT/TGW/VPC Endpointの経路を読む |
| Internet Gateway | VPCからインターネットへ出入りする入口 | Public subnetにはIGWへのルートが必要 |
| NAT Gateway | Private subnetから外向きインターネット通信を出す | S3/DynamoDBやAWS API宛はVPC EndpointでNATコスト削減可能 |
| Security Group | ENI単位のstateful firewall | 戻り通信は自動許可。許可ルールのみ |
| Network ACL | Subnet単位のstateless firewall | 戻り通信も明示許可が必要 |
| Inbound | リソース/サブネットへ入る方向の通信 | Server側ではrequestがInbound、Client側では戻り通信がInboundになる |
| Outbound | リソース/サブネットから出る方向の通信 | Private subnetからInternetへ出るならNAT GatewayやEndpointを読む |
| Explicit Deny | 明示的な拒否 | SGにはない。NACL/IAM/SCP/WAF/Network Firewallで使う |
| Stateful | 通信状態を覚えて戻り通信を扱う | SG/Network Firewall stateful rules |
| Stateless | 通信状態を覚えない | NACL/Network Firewall stateless rules |
| Ephemeral Port | クライアント側の一時ポート | NACLでは戻り通信用に高番ポート許可を考える |

## Security Group / NACL Memory Aids

### 語源

| 用語 | 語源・分解 | 覚え方 |
|---|---|---|
| Security Group | security + group | 同じ守り方をするENIのグループ |
| ACL | Access Control List | アクセス制御リスト。Allow/Denyの行リスト |
| NACL | Network ACL | ネットワーク境界、つまりSubnetに効くACL |
| Inbound | in + bound | 内側へ向かう通信 |
| Outbound | out + bound | 外側へ向かう通信 |
| Stateful | state + ful | 通信状態を覚える |
| Stateless | state + less | 通信状態を覚えない |

### ゴロと判断

```text
SG = Stateful Gate
  Sが2つ: Security Group / Stateful
  「許す門番」。Allowだけ、帰りは顔パス。

NACL = Numbered ACL
  NはNumberのN
  「番号つき改札」。番号順、Denyあり、帰りも切符が必要。

Route Table = Road Table
  道案内。通行許可証ではない。
```

### Deny早見表

| Denyしたい場所 | 使う候補 |
|---|---|
| 特定IP/CIDRをサブネット境界で拒否 | NACL |
| HTTPリクエストを条件で拒否 | WAF |
| VPC境界で通信を検査/拒否 | Network Firewall |
| AWS API操作を組織単位で禁止 | SCP |
| IAM主体の操作を明示拒否 | IAM explicit Deny |
| Security Groupで拒否 | できない。Allowしないことで閉じる |

## Routing and Connectivity

| 用語 | 意味 | SAP-C02での判断 |
|---|---|---|
| BGP | 動的ルーティングプロトコル | Direct Connect/VPNで冗長経路や経路広告を扱う |
| Transit Routing | あるネットワークを経由して別ネットワークへ到達すること | VPC Peeringは推移的ルーティング不可。大規模はTGW |
| Anycast | 同じIPを複数拠点から広告し近い入口へ誘導 | Global Acceleratorの固定Anycast IP |
| PrivateLink | Interface Endpoint経由のサービス単位接続 | VPC全体接続を避け、最小露出にしたいとき |
| Gateway Endpoint | S3/DynamoDB向けVPC Endpoint | NAT Gatewayを通さず安価にS3/DynamoDBへ |
| Interface Endpoint | ENIを使うPrivateLink型Endpoint | KMS/Secrets Manager/ECR/SSMなどAWS APIへprivate接続 |
| Internet Gateway | VPCとインターネットの出入口 | Public subnetにはIGW route、public IP、SG/NACL許可が必要 |
| NAT Gateway | Private subnetから外向き通信する出口 | 外部からPrivate subnetへ入る入口ではない |
| Transit Gateway | 多数VPC/オンプレ接続のハブ | 大規模ネットワーク集約。検査はNetwork Firewall/GWLB等と組み合わせる |
| Virtual Private Gateway | VPC側のVPN/DX終端 | 単一VPC接続で出やすい。大規模ハブはTGW |
| Customer Gateway | 顧客側VPN装置/設定 | オンプレ側の対向装置情報 |
| Direct Connect Gateway | Direct Connectを複数VPC/TGWに関連付ける中継 | VPC間トランジットルーターではない |
| Direct Connect | 専用線接続 | 低遅延/安定帯域。暗号化は別途VPN/MACsecなど |
| VPN | IPsec暗号化トンネル | 迅速/低コストなハイブリッド接続、DXのバックアップ |

## DNS and Content Delivery

| 用語 | 意味 | SAP-C02での判断 |
|---|---|---|
| DNS | 名前をIPなどへ解決する仕組み | Route 53、Private Hosted Zone、Resolver |
| TTL | DNSキャッシュの有効時間 | DR切替ではTTLとクライアントキャッシュを考慮 |
| Hosted Zone | DNSレコードを管理する単位 | Public/Privateの違いを読む |
| Health Check | エンドポイントの正常性確認 | Route 53 failover routingで利用 |
| CDN | エッジにキャッシュして配信する仕組み | AWSではCloudFront |
| Origin | CloudFrontの配信元 | S3、ALB、API Gateway、カスタムオリジン |
| OAC | CloudFrontからS3へ安全にアクセスする仕組み | S3直接公開を避けたいとき |

## Web / API

| 用語 | 意味 | SAP-C02での判断 |
|---|---|---|
| HTTP | Web通信プロトコル | L7。ALB/WAF/CloudFront/API Gatewayの領域 |
| HTTPS | TLSで暗号化したHTTP | ACMで証明書管理 |
| TLS | 通信暗号化と証明書検証 | CloudFront/ALB/API Gateway/ACM、mTLS |
| REST API | リソース指向API | API Gateway REST/HTTP APIで出やすい |
| WebSocket | 双方向通信 | API Gateway WebSocket、AppSync subscriptions |
| GraphQL | クライアントが必要なデータ形を指定するAPI | AppSyncのキーワード |
| Rate limiting | 一定時間あたりのリクエスト制限 | WAF rate-based rule、API Gateway throttling |
| SQL injection | 入力を悪用したSQL攻撃 | WAF Managed Rulesで緩和 |
| Bot traffic | 自動化アクセス | WAF Bot Control、rate-based rule、Shieldとの組み合わせ |

## Authentication and Authorization

| 用語 | 意味 | SAP-C02での判断 |
|---|---|---|
| Authentication | 誰かを確認すること | Cognito User Pool、IAM Identity Center |
| Authorization | 何を許可するか | IAM policy、Lake Formation、resource policy |
| SAML | 企業IdP連携でよく使うフェデレーション規格 | IAM Identity Center、エンタープライズSSO |
| OIDC | OAuth 2.0上の認証レイヤー | Cognito、ALB認証、EKS IRSA、外部IdP |
| JWT | 署名付きトークン | Cognito User PoolやOIDCでAPI認証に使う |
| MFA | 多要素認証 | 認証強度。データ権限そのものではない |
| STS | 一時認証情報の発行 | AssumeRole、フェデレーション、Cognito Identity Pool |
| External ID | サードパーティAssumeRoleの安全対策 | 混同代理問題を防ぐ |

## Performance Terms

| 用語 | 意味 | SAP-C02での判断 |
|---|---|---|
| Latency | 1リクエストの遅延 | CloudFront/Global Accelerator/read replica/cache |
| Throughput | 単位時間あたり処理量 | EBS throughput、Kinesis shard、FSx/EFS性能 |
| IOPS | 1秒あたりI/O回数 | EBS io2/gp3、DB性能 |
| Caching | 結果を一時保存すること | CloudFront、ElastiCache、DAX |
| Backpressure | 下流処理が詰まること | SQSでバッファ、Kinesisでスケール |

## Related

- [Networking Options](../comparisons/networking-options.md)
- [Security Group / NACL / Firewall](../comparisons/network-security-boundaries.md)
- [AWS Gateway Services and Terms](../comparisons/aws-gateways.md)
- [Edge Security](../comparisons/edge-security.md)
- [IAM](../services/security/iam.md)
- [Cognito](../services/security/cognito.md)
