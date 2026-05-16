# Security & Identity Services

## Tier 1

| サービス | 詳細 | 主な用途 |
|---|---|---|
| AWS IAM / Organizations | [iam.md](iam.md) | 権限評価、SCP、Permission Boundary、クロスアカウント |
| AWS KMS | [kms.md](kms.md) | Key policy、IAM policy、grants、SSE-KMS、クロスアカウント暗号化 |
| Amazon Cognito | [cognito.md](cognito.md) | User Pool / Identity Pool、アプリ認証、AWS一時認証情報 |
| AWS Secrets Manager | [secrets-manager.md](secrets-manager.md) | シークレット管理、自動ローテーション、マルチリージョン |
| AWS Certificate Manager | [acm.md](acm.md) | TLS証明書、CloudFront/ALB/API Gateway連携 |
| AWS WAF | [waf.md](waf.md) | L7 Web保護、Managed Rules、Rate-based rules |
| AWS Shield | [shield.md](shield.md) | DDoS対策、Standard/Advanced |
| AWS Network Firewall | [network-firewall.md](network-firewall.md) | VPC境界のネットワーク検査、stateful/stateless rules |
| Amazon GuardDuty | [guardduty.md](guardduty.md) | 脅威検出、Organizations集約 |

## Tier 2 / Related

- AWS IAM Identity Center
- [AWS Security Hub](../management/security-hub.md)
- [AWS Config](../management/config.md)
- [AWS CloudTrail](../management/cloudtrail.md)
- AWS Firewall Manager
- Amazon Detective
- Amazon Inspector

## Key Principle

Defense in depth: Identity (IAM/Cognito) + Network (VPC/SG/Endpoint/Network Firewall) + Data (KMS/Secrets Manager) + Edge (WAF/Shield/CloudFront) + Detection (GuardDuty/Security Hub) + Audit (CloudTrail/Config)。

詳しくは [Edge and Network Security Comparison](../../comparisons/edge-security.md) を参照。
