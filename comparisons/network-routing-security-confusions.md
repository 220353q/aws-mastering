# Network / Routing / Security Boundary 混同整理 — SAP-C02横断

## 何のためのページか

AWSネットワーク問題では、通信できない理由を「経路がない」のか「許可されていない」のか「検査で落ちている」のかに分ける必要がある。

このページでは、SAP-C02で混同しやすいNetwork / Routing / Security Boundary系の概念を整理する。

---

## まず結論

```text
Route Table = どこへ流すか
Security Group = ENI単位で何を許可するか
NACL = Subnet単位で何を許可/拒否するか
WAF = HTTP/HTTPSのL7 Web保護
Network Firewall = VPC内のネットワーク検査
Transit Gateway = 多数VPC/オンプレのハブ
PrivateLink = サービス単位のprivate接続
CloudFront = HTTP/HTTPS CDN/cache
Global Accelerator = TCP/UDPのグローバル入口最適化
```

---

## 1. Route Table vs Security Group

| 概念 | 役割 | 例 |
|---|---|---|
| Route Table | 宛先CIDRに対して次ホップを決める | 0.0.0.0/0 → NAT Gateway |
| Security Group | ENIに対して許可する通信を決める | inbound 443 from ALB SG |

### 覚え方

```text
Route Table = 道案内
Security Group = 入場許可
```

Route Tableが正しくても、Security Groupが閉じていれば通信できない。逆にSecurity Groupが開いていても、Route Tableに経路がなければ通信できない。

---

## 2. Security Group vs NACL

| 項目 | Security Group | NACL |
|---|---|---|
| 適用単位 | ENI / Resource | Subnet |
| 状態 | Stateful | Stateless |
| ルール | Allowのみ | Allow / Deny |
| 戻り通信 | 自動許可 | 明示的に許可が必要 |
| 評価 | 全ルール | 番号順 |

### 覚え方

```text
SG = リソースのドア
NACL = サブネットの門
```

### よくある罠

```text
NACLはstatelessなので、戻り通信のephemeral portも考える。
SGはstatefulなので、許可した通信の戻りは自動許可される。
```

---

## 3. Internet Gateway vs NAT Gateway

| サービス | 目的 | 典型配置 |
|---|---|---|
| Internet Gateway | VPCとインターネットの出入口 | VPCにattach |
| NAT Gateway | Private subnetから外向きインターネット通信 | Public subnet |

### 覚え方

```text
IGW = VPCをインターネットに出す門
NAT Gateway = Private subnetから外へ出る代理
```

NAT Gatewayは外向き通信には使えるが、インターネットからprivate subnetへ直接入る入口ではない。

---

## 4. NAT Gateway vs VPC Endpoint

| 要件 | 選ぶ候補 |
|---|---|
| Private subnetから一般インターネットへ出たい | NAT Gateway |
| Private subnetからS3/DynamoDBへprivate接続 | Gateway Endpoint |
| Private subnetからKMS/SSM/Secrets Manager等へprivate接続 | Interface Endpoint |

### コスト問題の罠

NAT Gateway経由で大量にS3へアクセスしている場合、Gateway Endpointへ置き換えるとコストとセキュリティの両面で改善できることがある。

---

## 5. Transit Gateway vs VPC Peering vs PrivateLink

| サービス | 目的 | 向くケース |
|---|---|---|
| VPC Peering | 2つのVPCを直接接続 | 少数VPC、シンプル接続 |
| Transit Gateway | 多数VPC/オンプレのハブ | 大規模hub-and-spoke |
| PrivateLink | サービス単位でprivate接続 | SaaS公開、共有サービス、CIDR重複 |

### 覚え方

```text
VPC Peering = 1対1接続
Transit Gateway = ネットワークのハブ
PrivateLink = サービスだけ公開
```

### 誤答パターン

| 誤答 | なぜ危険か |
|---|---|
| PrivateLinkでVPC全体を相互接続する | PrivateLinkはサービス単位 |
| VPC Peeringで大規模多数VPCを管理する | 接続数・ルート管理が複雑化する |
| Transit GatewayでSaaSの特定サービスだけ公開する | それはPrivateLinkが自然 |

---

## 6. WAF vs Shield vs Network Firewall

| サービス | 守るレイヤー | 主な用途 |
|---|---|---|
| AWS WAF | L7 HTTP/HTTPS | SQLi/XSS/rate-based rule/bot対策 |
| AWS Shield | DDoS | L3/L4/L7 DDoS緩和 |
| AWS Network Firewall | VPC内ネットワーク | stateful/stateless filtering, domain filtering |

### 覚え方

```text
WAF = Webの中身を見る
Shield = DDoSに備える
Network Firewall = VPC内通信を検査する
```

---

## 7. Gateway Load Balancer vs Network Firewall

| サービス | 役割 |
|---|---|
| Gateway Load Balancer | サードパーティアプライアンスを透過的に挿入/分散 |
| AWS Network Firewall | AWSマネージドのネットワークファイアウォール |

GWLBはFirewallそのものではなく、アプライアンス挿入のためのロードバランサーとして読む。

---

## 8. CloudFront vs Global Accelerator

| サービス | 主対象 | 特徴 |
|---|---|---|
| CloudFront | HTTP/HTTPS | CDN、キャッシュ、WAF、オリジン保護 |
| Global Accelerator | TCP/UDP | Anycast IP、AWS global network、リージョン切替 |

### 覚え方

```text
CloudFront = HTTPコンテンツを近くに置く
Global Accelerator = TCP/UDP入口を速く安定させる
```

キャッシュが必要ならCloudFront。固定Anycast IPやTCP/UDPの入口最適化ならGlobal Accelerator。

---

## 9. ALB vs NLB

| LB | レイヤー | 主な用途 |
|---|---|---|
| ALB | L7 | HTTP/HTTPS/gRPC、パス/ホストベースルーティング |
| NLB | L4 | TCP/TLS/UDP、高性能、固定IP、PrivateLink提供側 |

### 覚え方

```text
ALB = URLやHostを見る
NLB = IP/Portレベルで速く流す
```

PrivateLinkのProvider側ではNLBが頻出。

---

## 10. SAP-C02判断フロー

```text
通信できない？
  → Route Tableに経路はある？
  → SGは許可している？
  → NACLは戻り通信も許可している？
  → DNSは正しく解決している？
  → VPC Endpoint / NAT / IGW / TGWなど入口出口はある？
  → WAF / Firewall / Endpoint Policy / IAMで拒否されていない？
```

```text
多数VPCをつなぎたい？
  → Transit Gateway

特定サービスだけprivate公開？
  → PrivateLink

S3/DynamoDBへNATなし？
  → Gateway Endpoint

KMS/SSM等へNATなし？
  → Interface Endpoint

HTTP/HTTPSをglobal配信/cache？
  → CloudFront

TCP/UDPをglobal最適化？
  → Global Accelerator
```

---

## よくある誤答パターン

| 誤答 | なぜ危険か |
|---|---|
| Route Tableで許可/拒否を制御する | Route Tableは経路。Firewallではない |
| SGでDenyを書く | SGはAllowのみ |
| NACLで戻り通信を忘れる | NACLはstateless |
| NAT Gatewayを外部からprivate subnetへ入る入口にする | NATは外向き代理 |
| PrivateLinkでVPC全体を接続する | PrivateLinkはサービス単位 |
| GWLBをFirewallそのものと考える | GWLBはアプライアンス挿入/分散 |
| CloudFrontとGlobal Acceleratorを同じものと考える | CDN/cacheかTCP/UDP入口最適化かが違う |
| WAFでVPC内の全TCP通信を検査する | WAFはHTTP/HTTPS L7保護 |

---

## 最短暗記

```text
Route = 道
SG = ENIの許可
NACL = Subnetの門
IGW = Internetの門
NAT = Private subnetの外向き代理
TGW = 多数VPCのハブ
PrivateLink = サービスだけprivate公開
WAF = Web L7保護
Network Firewall = VPC内検査
CloudFront = CDN/cache
Global Accelerator = TCP/UDP入口最適化
```

---

## 関連ページ

- [AWS 混同しやすい概念インデックス](aws-confusing-concepts-index.md)
- [Networking Foundations Deep Dive](networking-foundations-deep-dive.md)
- [Security Group / NACL / Firewall Decision Guide](network-security-boundaries.md)
- [AWS Gateway Services and Terms](aws-gateways.md)
- [HTTP / HTTPS / WebSocket / MQTT / gRPC / TCP / UDP とAWS入口サービス](protocols-and-aws-entrypoints.md)
- [AWS Endpoint / ENI / VIF 完全整理](endpoints-eni-vif.md)
