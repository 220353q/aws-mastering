# AWS Security Hub

## 何をするサービスか

AWS Security Hub は、複数のAWSサービスやサードパーティ製品からセキュリティ所見を集約し、標準に基づいて評価・優先度付けするサービス。SAP-C02では、**セキュリティ検出結果の集中管理、マルチアカウント運用、継続的な姿勢管理**で出る。

## 何を集約するか

| 入力元 | 役割 |
|---|---|
| GuardDuty | 脅威検出 |
| Inspector | 脆弱性管理 |
| Macie | 機密データ検出 |
| IAM Access Analyzer | 外部公開や信頼ポリシー分析 |
| AWS Config | 準拠評価の材料 |
| Partner products | 外部セキュリティ製品の所見 |

## セキュリティ標準

Security Hubは、AWS Foundational Security Best PracticesやCISなどの標準に基づいてコントロールを評価できる。単なるログ保管ではなく、**所見を正規化・集約・優先度付け**する点が重要。

## 典型アーキテクチャ

```text
Member Accounts
  ├─ GuardDuty
  ├─ Inspector
  ├─ Macie
  └─ Config
        ↓ findings
Delegated Security Account
  └─ Security Hub
        ↓ EventBridge
Automation / Ticket / SIEM
```

## GuardDuty / Security Hub / Detective の違い

| サービス | 役割 |
|---|---|
| GuardDuty | 脅威を検出する |
| Security Hub | 所見を集約・標準評価・優先度付けする |
| Detective | セキュリティ事象を調査する |

## よくある誤答

- **Security Hubを脅威検出そのものとして選ぶ**: 検出はGuardDutyやInspectorなど。Security Hubは集約・評価。
- **CloudTrailの代替として選ぶ**: CloudTrailは監査ログ。Security Hubは所見管理。
- **Configの代替として選ぶ**: Configはリソース設定準拠評価。Security Hubは複数ソースの所見集約。

## SAP-C02 Focus

- OrganizationsではDelegated Administratorを使って集約する。
- Security Hub findingsはEventBridgeで検知し、自動修復やチケット化へつなぐ。
- 継続的改善では、Security Hub + Config + GuardDuty + CloudTrail + Systems Manager Automationの組み合わせが強い。
