# Management / Governance / Audit Comparison

## 一言で切り分ける

| 要件 | サービス |
|---|---|
| APIを誰が呼んだか | CloudTrail |
| リソース設定がどう変わったか | AWS Config |
| メトリクス/ログ/アラーム | CloudWatch |
| IaCで展開 | CloudFormation |
| 複数アカウントにIaC展開 | CloudFormation StackSets |
| セキュリティ所見を集約 | Security Hub |
| 脅威を検出 | GuardDuty |
| 標準Landing Zone | Control Tower |
| EC2へ安全に接続/自動運用 | Systems Manager |

## 代表パターン: 検知から自動修復

```text
CloudTrail / Config / GuardDuty
        ↓
Security Hub / EventBridge
        ↓
Systems Manager Automation / Lambda
        ↓
修復・通知・チケット化
```

## SAP-C02での罠

- CloudTrailは「誰がやったか」。Configは「何がどうなっているか」。
- Security Hubは「集約と優先度付け」。脅威検出そのものはGuardDutyなど。
- CloudFormationは「望ましい状態の展開」。Configは「現在/過去状態の評価」。
