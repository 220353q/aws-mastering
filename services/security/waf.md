# AWS WAF

## Positioning

AWS WAF は、CloudFront、ALB、API Gateway、AppSyncなどの前段でHTTP/HTTPSリクエストを検査し、L7レベルで許可/ブロック/レート制御するWeb Application Firewall。

SAP-C02では、Shield、Security Group、Network Firewall、CloudFrontとの違いが問われる。

---

## Core Concepts

| 概念 | 内容 |
|---|---|
| Web ACL | 保護対象に関連付けるルール集合 |
| Rule | 条件に一致したリクエストへAllow/Block/Count等を適用 |
| Managed Rule Group | AWSやベンダーが管理するルール群 |
| Rate-based Rule | IPやヘッダー等の集約キーごとにリクエスト数を制限 |
| Label | ルール評価結果を後続ルールで利用 |
| WCU | Web ACL capacity unit。ルール複雑性の容量指標 |

---

## Protected Resources

| リソース | 使い方 |
|---|---|
| CloudFront | グローバルWeb保護、エッジでのL7制御 |
| Application Load Balancer | リージョナルWebアプリ保護 |
| API Gateway | APIへのL7保護 |
| AppSync | GraphQL API保護 |
| Cognito User Pool | 一部の認証エンドポイント保護に関係 |

---

## Common Rules

- AWS Managed Rules Common Rule Set
- SQL injection対策
- XSS対策
- IP set / Geo match
- Bot Control
- Rate-based rules
- Header / URI / query string / body inspection

---

## WAF vs Shield vs Network Firewall

| 要件 | 選択 |
|---|---|
| SQLi/XSS/Bot/Rate limitなどL7 Web保護 | WAF |
| DDoS対策 | Shield Standard / Shield Advanced |
| VPC境界でL3-L7ネットワークトラフィック検査 | Network Firewall |
| EC2単位の許可/拒否 | Security Group / NACL |
| CloudFrontで国別制限 | CloudFront Geo Restriction または WAF Geo Match |

---

## Rate-based Rule

```text
If requests from same IP exceed threshold in evaluation window
  → Block / Count / CAPTCHA / Challenge
```

大量リクエスト、簡易DoS、スクレイピング、ブルートフォース緩和で使われる。

---

## SAP-C02 Focus

WAFを選ぶキーワード:

- SQL injection / XSS
- HTTP flood at application layer
- block by URI/header/country/IP reputation
- rate limiting
- protect CloudFront / ALB / API Gateway

WAFを選ばない文脈:

- TCP/UDPレベルのフィルタ → Network Firewall / NACL / SG
- 大規模DDoS緩和・コスト保護 → Shield Advanced
- EC2インスタンスへの単純なポート制御 → Security Group

---

## Exam Traps

- WAFはL7。NLBやTCPアプリの直接保護には向かない。
- Shield Standardは自動で基本DDoS保護を提供するが、Advancedは明示的な保護対象設定や追加機能がある。
- Geo制限はCloudFront単体でも可能だが、複雑な条件ならWAF。
- Managed Rulesは便利だが、誤検知に備えてCountモードで検証する設計が問われることがある。

---

## Related

- [AWS Shield](shield.md)
- [AWS Network Firewall](network-firewall.md)
- [Amazon CloudFront](../networking/cloudfront.md)
- [Elastic Load Balancing](../networking/elb.md)
- [Edge Security Comparison](../../comparisons/edge-security.md)

## Official Docs

- https://docs.aws.amazon.com/waf/latest/developerguide/web-acl.html
- https://docs.aws.amazon.com/waf/latest/developerguide/aws-managed-rule-groups.html
- https://docs.aws.amazon.com/waf/latest/developerguide/waf-rule-statement-type-rate-based.html
