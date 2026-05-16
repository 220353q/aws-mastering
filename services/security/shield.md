# AWS Shield / AWS Shield Advanced

## Positioning

AWS Shield は、AWSリソースをDDoS攻撃から保護するサービス。Shield Standardは自動で基本的な保護を提供し、Shield Advancedは追加の検知、緩和、可視化、サポート、コスト保護などを提供する。

SAP-C02では、WAFとの違い、CloudFront/Route 53/ALB/NLB/EIP保護、Advancedの明示的保護設定が問われる。

---

## Standard vs Advanced

| 項目 | Shield Standard | Shield Advanced |
|---|---|---|
| 提供 | AWS利用者に自動適用 | 有料サブスクリプション |
| 保護 | 一般的なL3/L4 DDoS緩和 | 高度なDDoS保護、可視化、対応支援 |
| 対象設定 | 基本的に自動 | 保護対象リソースを明示設定 |
| WAF連携 | 個別設定 | L7自動緩和など高度連携 |
| サポート | 標準 | DDoS Response Teamなど |
| コスト保護 | なし | DDoS関連スケーリングコスト保護の文脈で出る |

---

## Protected Resource Types

Shield Advancedは、主に次のようなリソースを保護対象として扱う。

- Amazon CloudFront distributions
- Amazon Route 53 hosted zones
- Application Load Balancers
- Network Load Balancers
- Elastic IP addresses
- AWS Global Accelerator accelerators

対象リソースを明示的に保護に追加する点が重要。

---

## Shield vs WAF

| 要件 | 選択 |
|---|---|
| DDoS攻撃の緩和 | Shield |
| SQLi/XSS/Bot/Rate limitなどWebリクエスト条件制御 | WAF |
| CloudFront配信をDDoS + L7で守る | Shield Advanced + WAF |
| TCP/UDPの大規模攻撃対策 | Shield Advanced + CloudFront/NLB/Global Accelerator等 |

---

## Typical Edge Protection

```text
Internet
  → Route 53
  → CloudFront + AWS WAF + Shield Advanced
  → ALB
  → Application
```

グローバルWebアプリでは、CloudFrontを前段に置くことでエッジで吸収し、WAFでL7条件制御、ShieldでDDoS緩和を組み合わせる。

---

## SAP-C02 Focus

Shield Advancedを選ぶ文脈:

- DDoS resilience for critical internet-facing application
- access to DDoS response experts
- advanced detection and mitigation
- cost protection from DDoS-related scaling
- protect CloudFront/Route53/ALB/NLB/EIP/Global Accelerator

---

## Exam Traps

- Shield Advancedは契約すれば全リソースが自動的に高度保護される、ではない。保護対象設定が必要。
- SQL Injection対策はShieldではなくWAF。
- Security GroupはDDoS対策サービスではない。
- CloudFront + WAF + Shield Advancedの組み合わせは、重要な公開Webサービスの定番構成。

---

## Related

- [AWS WAF](waf.md)
- [Amazon CloudFront](../networking/cloudfront.md)
- [AWS Global Accelerator](../networking/global-accelerator.md)
- [Edge Security Comparison](../../comparisons/edge-security.md)

## Official Docs

- https://docs.aws.amazon.com/waf/latest/developerguide/ddos-overview.html
- https://docs.aws.amazon.com/waf/latest/developerguide/ddos-advanced-summary-protected-resources.html
