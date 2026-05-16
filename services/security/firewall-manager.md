# AWS Firewall Manager

AWS Firewall Manager は、Organizations配下の複数アカウント/リソースに対して、WAF、Shield Advanced、Security Group、Network Firewallなどのセキュリティポリシーを一元適用するサービス。

## 一言で

複数アカウントへファイアウォール系ポリシーを標準展開するならFirewall Manager。

## 試験で選ぶ条件

- Organizations全体にWAFルールを強制したい
- ALB/API Gateway/CloudFrontなどへ共通Web保護を適用したい
- Shield AdvancedやSecurity Groupポリシーを中央管理したい
- 新規アカウント/新規リソースにも自動適用したい

## 役割比較

| 要件 | サービス |
|---|---|
| L7 Web攻撃対策 | WAF |
| DDoS保護 | Shield |
| VPC境界のネットワーク検査 | Network Firewall |
| それらを組織全体へ中央適用 | Firewall Manager |

## High-Risk Exam Traps

- Firewall Managerは単体のWAFルールエンジンではなく、中央管理の仕組み。
- Organizations前提のマルチアカウント設計で強い。
- Security Hubの所見集約とは役割が違う。

## Related

- [WAF](waf.md)
- [Shield](shield.md)
- [Network Firewall](network-firewall.md)
- [IAM / Organizations](iam.md)
