# Comparison: Edge and Network Security Services

## Purpose

SAP-C02では、WAF / Shield / Network Firewall / Security Group / NACL / CloudFront / GWLB の役割を混同しやすい。どのレイヤーで、何を守るのかで判断する。

---

## Service Selection Table

| 要件 | 選ぶサービス |
|---|---|
| SQLi/XSS/Bot/Rate limitなどHTTPリクエスト制御 | AWS WAF |
| DDoS緩和 | AWS Shield Standard / Advanced |
| 重要な公開Webサービスの高度DDoS対策 | Shield Advanced + CloudFront + WAF |
| VPC境界でネットワークトラフィック検査 | AWS Network Firewall |
| EC2/ENI単位のポート許可 | Security Group |
| サブネット単位のステートレスACL | Network ACL |
| サードパーティFirewall/IDS/IPS挿入 | Gateway Load Balancer |
| エッジキャッシュとOAC | CloudFront |
| 固定Anycast IPとグローバルフェイルオーバー | Global Accelerator |

---

## Layer View

```text
L7 Web request        → AWS WAF
DDoS L3/L4/L7         → AWS Shield / Shield Advanced
VPC network inspection→ AWS Network Firewall
ENI-level filtering   → Security Group
Subnet stateless ACL  → NACL
Appliance insertion   → Gateway Load Balancer
Edge cache/TLS/OAC    → CloudFront
Global anycast entry  → Global Accelerator
```

---

## Typical Secure Public Web Pattern

```text
Users
  → Route 53
  → CloudFront
      + AWS WAF
      + Shield Advanced
  → ALB
      + ACM certificate
  → Private subnets
      + Security Groups
  → App / DB
      + KMS encryption
```

---

## Common Exam Traps

- WAFはDDoS専門ではない。L7条件制御が中心。
- Shield Advancedは重要公開リソースのDDoS対策。SQLiならWAF。
- Network FirewallはVPC境界の検査。CloudFrontのWeb ACLとは役割が違う。
- GWLBはアプライアンス挿入。普通のWeb L7ルーティングならALB。
- ACMはTLS証明書管理。攻撃検知やDDoS緩和はしない。

---

## Related Services

- [AWS WAF](../services/security/waf.md)
- [AWS Shield](../services/security/shield.md)
- [AWS Network Firewall](../services/security/network-firewall.md)
- [AWS Certificate Manager](../services/security/acm.md)
- [Elastic Load Balancing](../services/networking/elb.md)
- [AWS Global Accelerator](../services/networking/global-accelerator.md)
