# Elastic Load Balancing - ALB / NLB / GWLB

## Positioning

Elastic Load Balancing は、複数ターゲットへトラフィックを分散し、ヘルスチェックに基づいて正常なターゲットへルーティングする基盤サービス。SAP-C02では **ALB / NLB / Gateway Load Balancer の使い分け** が重要。

ALB/NLBがどのサービスの前段・後段に来るか、プロトコル別の入口サービスと具体ユースケースは [Protocols and Load Balancer Positioning](../../comparisons/protocols-load-balancers-positioning.md) を参照。

---

## Load Balancer Types

| 種類 | レイヤー | 主な用途 | キーワード |
|---|---|---|---|
| Application Load Balancer | L7 | HTTP/HTTPS、パス/ホストベースルーティング、コンテナ | Web, microservices, path routing |
| Network Load Balancer | L4 | TCP/UDP/TLS、超高性能、固定IP、低レイテンシ | TCP, static IP, extreme performance |
| Gateway Load Balancer | L3/L4 appliance insertion | IDS/IPS/Firewallなどアプライアンス挿入 | third-party firewall, GENEVE, appliance |
| Classic Load Balancer | Legacy | 旧世代 | 新規では基本選ばない |

---

## ALB

### Use when

- HTTP/HTTPSアプリケーション
- host-based / path-based routing
- ECS/EKSコンテナサービスのターゲット分散
- OIDC/Cognito認証連携
- WebSocket / HTTP/2 / gRPC などL7機能

### Typical Pattern

```text
Internet / CloudFront
  → ALB
    → Target Group A: /api/*
    → Target Group B: /admin/*
    → Target Group C: /static/*
```

---

## NLB

### Use when

- TCP/UDP/TLSレベルのロードバランシング
- 非HTTPプロトコル
- 低レイテンシ・高スループット
- 固定IPが必要
- PrivateLinkのEndpoint Serviceとして公開したい

### Trap

NLBはL7のパスベースルーティングをしない。HTTPのルーティング条件で分けたいならALB。

---

## Gateway Load Balancer

### Use when

- サードパーティFirewall/IDS/IPSを透過的に挿入したい
- 複数VPCの通信を中央検査したい
- アプライアンスのスケール/可用性をELBに任せたい

```text
Spoke VPC
  → GWLB Endpoint
    → Gateway Load Balancer
      → Security Appliances
```

GWLBはGENEVEプロトコルを使ってアプライアンスと通信する。

---

## ALB vs NLB vs GWLB

| 要件 | 選ぶもの |
|---|---|
| URLパス/ホスト名でルーティング | ALB |
| HTTPヘッダーや認証連携 | ALB |
| TCP/UDP、固定IP、高性能 | NLB |
| PrivateLinkでサービス公開 | NLB + Endpoint Service |
| セキュリティアプライアンス挿入 | GWLB |
| WAFを直接関連付けたい | ALB / CloudFront / API Gateway |

---

## SAP-C02での読み方

ELB選択問題では、次の順に読む。

1. プロトコルはHTTP/HTTPSか、TCP/UDPか
2. L7ルーティング条件が必要か
3. 固定IPやPrivateLink公開が必要か
4. セキュリティアプライアンスを挿入するのか
5. CloudFront/WAF/ACMとの連携が必要か

---

## Exam Traps

- 固定IP要件だけでALBを選ばない。NLBまたはGlobal Acceleratorを検討。
- WAFはNLBに直接関連付けるものではない。L7保護はALB/CloudFront/API Gateway側。
- GWLBは通常のWebロードバランサーではなく、アプライアンス挿入用。
- ALBはL7、NLBはL4という基本を外すと誤答になりやすい。

---

## Related

- [Protocols and Load Balancer Positioning](../../comparisons/protocols-load-balancers-positioning.md)
- [Networking Foundations Deep Dive](../../comparisons/networking-foundations-deep-dive.md)
- [Amazon VPC](vpc.md)
- [AWS PrivateLink](privatelink.md)
- [AWS Global Accelerator](global-accelerator.md)
- [AWS WAF](../security/waf.md)
- [Networking Options Comparison](../../comparisons/networking-options.md)

## Official Docs

- https://docs.aws.amazon.com/elasticloadbalancing/
- https://docs.aws.amazon.com/elasticloadbalancing/latest/gateway/introduction.html

## このページを読んだあとに戻るべき関連ページ

- [Protocols and Load Balancer Positioning](../../comparisons/protocols-load-balancers-positioning.md)
- [Networking Foundations Deep Dive](../../comparisons/networking-foundations-deep-dive.md)
- [AWS Gateway Services and Terms](../../comparisons/aws-gateways.md)
- [Security Group / NACL / Firewall](../../comparisons/network-security-boundaries.md)
- [AWS PrivateLink](privatelink.md)